# Osmium Operating System Kernel

> *"From yours truly, Captain I."*

## Developer's Note & Project Status

Welcome to **Osmium**, an experimental x86 operating system kernel built from scratch!

This project was—and continues to be—a personal side project born out of a desire to learn low-level systems programming. Because I am still expanding my operating system development knowledge, certain high-level features like a Graphical User Interface (GUI) and a full User Shell are not yet implemented (a robust shell will require an updated filesystem architecture and internet/network drivers).


## Directory Structure

* **`kernel.c`**: The main entry point for the kernel after the bootloader hands off execution. Initializes kernel subsystems (GDT, IDT, PMM, VMM, PIT, keyboard, disk drivers, filesystem) and starts initial kernel/user tasks.
* **`cpu/asm/`**: Assembly routines required for CPU table loading, context switching, interrupt stubs, and user mode transitions.
* **`cpu/lib/`**: Core kernel architecture routines (CPU tables, memory managers, scheduler, interrupts, system calls).
* **`cpu/drivers/`**: Hardware driver implementations and filesystem interfaces.

---

## CPU / LIB Directory Components

The files in `cpu/lib/` implement fundamental x86 hardware management and core operating system abstractions:

### 1. CPU Descriptors & Interrupts
* **`gdt.c`**: Sets up the Global Descriptor Table (GDT). Configures segment descriptors for kernel code/data, user code/data, and Task State Segment (TSS) entries required for ring switching.
* **`idt.c`**: Sets up the Interrupt Descriptor Table (IDT), assigning handler function pointers to specific gate offsets (0x00 through 0xFF).
* **`isr.c`**: Implements Interrupt Service Routine (ISR) handlers for standard CPU exceptions (e.g., Page Faults, Divide-by-Zero, Double Faults).
* **`irq.c`**: Manages hardware Interrupt Requests (IRQs) remaped via the PIC/APIC, dispatching IRQ lines to registered driver callbacks.
* **`pic.c`**: Configures the Programmable Interrupt Controller (8259 PIC), remapping hardware IRQs (IRQ 0-15) to vectors above standard CPU exceptions (vectors 32-47).
* **`apic.c`**: Provides support and initialization for modern Advanced Programmable Interrupt Controller (APIC) architectures for timer and interrupt management.

### 2. Memory Management
* **`pmm.c`**: Physical Memory Manager. Implements a bitmap-based allocator to track free and used physical 4KB memory frames across the RAM map provided by the bootloader.
* **`vmm.c`**: Virtual Memory Manager. Handles hardware page directory and page table operations for x86 paging (32-bit paging mode).
* **`paging.c`**: Interfaces physical frame allocation with virtual address mapping. Manages fault handling for unmapped memory pages and sets up kernel/user space address splits.
* **`kheap.c`**: Kernel Heap Allocator (`kmalloc` / `kfree`). Provides dynamic memory allocation capabilities within the kernel's virtual address space using a boundary-tag or node-based allocation strategy.
* **`memory_map.c`**: Parses and manages BIOS E820 memory maps passed down during boot to determine usable vs. reserved RAM regions.

### 3. Process & Thread Management
* **`process.c`**: High-level process lifecycle control. Manages process creation, address space allocation (page directories), state management, and termination.
* **`thread.c`**: Execution thread management and context scheduler. Maintains execution states, stacks, and thread control blocks (TCBs) for context switching.
* **`syscall.c`**: System Call dispatcher. Configures interrupt `0x80` or equivalent entry points to allow user-mode tasks to safely invoke kernel functionality (e.g., I/O, memory request, yield).

### 4. Utilities & Clocks
* **`pit.c`**: Configures the Programmable Interval Timer (8254 PIT) to generate regular clock tick IRQs (IRQ0) for preemption and timekeeping.
* **`acpi.c`**: Advanced Configuration and Power Interface parser. Extracts system information from ACPI tables (RSDP, MADT, FADT) for power control and APIC detection.

---

## CPU / DRIVERS Directory Components

The files in `cpu/drivers/` allow the kernel to interact with peripheral hardware devices and storage:

* **`terminal.c`**: Handles VGA text-mode display memory output (`0xB8000`). Provides screen clearing, character printing, scrolling, cursor positioning, and colored text rendering.
* **`keyboard.c`**: PS/2 Keyboard Controller driver. Catches IRQ1 interrupts, reads scancodes from port `0x60`, translates scancodes into ASCII characters, and feeds an input buffer.
* **`ata_pio.c`**: Hard disk controller driver using ATA Programmed Input/Output (PIO) mode. Reads and writes 512-byte sectors from IDE/ATA drives via standard I/O ports (`0x1F0`-`0x1F7`).
* **`osmium_fs.c`**: High-level driver for the custom Osmium File System. Provides APIs for mounting storage devices, creating, reading, writing, and deleting files on disk.
* **`fs_layout.c`**: Manages on-disk structures and metadata formatting (superblocks, inode tables, directory entry parsing) required by the filesystem.

---

## Build and Execution Flow

1. **Bootstrapping (`boot.asm`)**: Starts execution in 16-bit real mode, transitions the CPU into 32-bit protected mode, enables A20 line, loads kernel code into memory, and jumps to `kernel.c`.
2. **Initialization (`kernel.c`)**:
   * Initializes basic text output (`terminal.c`).
   * Loads GDT (`gdt.c`) and IDT (`idt.c`).
   * Remaps PIC (`pic.c`) and enables hardware interrupts.
   * Initializes physical (`pmm.c`) and virtual (`vmm.c`, `paging.c`) memory managers.
   * Enables the kernel heap (`kheap.c`).
   * Configures system timer (`pit.c`) and keyboard (`keyboard.c`).
   * Initializes disk access (`ata_pio.c`) and mounts the filesystem (`osmium_fs.c`).
   * Registers system call handlers (`syscall.c`).
3. **Execution Loop**: Spawns initial kernel threads or user-space tasks (`process.c`, `thread.c`) and initiates the scheduling loop.


## Building and Running

To build and run the kernel locally using QEMU:

```bash
make
qemu-system-i386 -drive format=raw,file=osmium-disk.img
```
or 

```bash
make run
```





### ⏸️ Temporary Project Pause & Resume Plan
I am currently putting active development on hold to focus fully on my studies. 

* **Current Status:** Paused (Development Break)
* **Planned Resume Date:** January 2027

### 🤝 Open for Collaboration & Feedback
Even though I am taking a hiatus, feedback and collaboration are very welcome! If you inspect the codebase, find any bugs/flaws, or want to contribute to its development, feel free to reach out.

* **Contact Email:** `subhadipm368@gmail.com`

---
