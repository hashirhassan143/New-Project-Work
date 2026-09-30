# xv6-riscv: Extended Kernel Modules

> A set of small kernel extensions for the **xv6 (RISC-V)** teaching operating system, split into an **Operating Systems track** and a **Computer Architecture track**, five modules each.


## What is in this repository?

xv6 is a simple Unix-like kernel written in C for RISC-V. In this project the kernel is extended with ten new features. Every feature is small on purpose, so that the changes are easy to read and explain in a viva.

```
Track A  ->  Operating Systems         (modules A1 - A5)
Track B  ->  Computer Architecture     (modules B1 - B5)
```

## Setup

**You need:** RISC-V GCC toolchain, QEMU (`qemu-system-riscv64`), `make`.

```bash
git clone <repository-url>
cd xv6-riscv
make clean
make qemu
```

Quit QEMU with `Ctrl+A`, then `X`.

---

# Track A: Operating Systems

## A1. `hello` System Call

**What it does:** Takes an integer from the user program, prints a greeting from inside the kernel and returns the same number plus one.

**How it works:** The user calls `hello(5)`. The call enters the kernel through `ecall`, `sys_hello()` reads the argument with `argint()`, prints a message with `printf`, and returns the result.

**Files modified:** `kernel/syscall.h`, `kernel/syscall.c`, `kernel/sysproc.c`, `user/user.h`, `user/usys.pl`, `Makefile`
**New file:** `user/hellotest.c`

**Test:** `$ hellotest`

---

## A2. `syscount` System Call

**What it does:** Counts how many system calls the current process has made.

**How it works:** A new integer field `syscalls` is added to `struct proc`. The central `syscall()` function increments it on every call. `sys_syscount()` returns the value.

**Files modified:** `kernel/proc.h`, `kernel/proc.c` (initialise to 0 in `allocproc`), `kernel/syscall.c`, `kernel/sysproc.c`, `kernel/syscall.h`, `user/user.h`, `user/usys.pl`
**New file:** `user/sctest.c`

**Test:** `$ sctest`

---

## A3. `touch` Utility

**What it does:** Creates an empty file, or does nothing if the file already exists.

**How it works:** A user-level program that calls `open()` with `O_CREATE | O_RDWR` for each argument and then `close()`.

**Files modified:** `Makefile` (add `$U/_touch` to `UPROGS`)
**New file:** `user/touch.c`

**Test:**
```
$ touch notes.txt
$ ls
```

---

## A4. `random` System Call

**What it does:** Returns a pseudo-random number.

**How it works:** A linear congruential generator (LCG) in the kernel keeps a static seed. The seed is mixed with the current `ticks` value so results differ between runs.

**Files modified:** `kernel/sysproc.c`, `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h`, `user/usys.pl`
**New file:** `user/randtest.c`

**Test:** `$ randtest`

---

## A5. `getprocname` System Call

**What it does:** Returns the name of the calling process (for example `sh` or `randtest`).

**How it works:** `sys_getprocname()` reads `myproc()->name` and copies it to a user buffer using `copyout()`.

**Files modified:** `kernel/sysproc.c`, `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h`, `user/usys.pl`
**New file:** `user/nametest.c`

**Test:** `$ nametest`

---

# Track B: Computer Architecture

## B1. Hart (CPU Core) Information

**What it does:** Shows which CPU core (hart) is running the current process and how many cores xv6 is configured for.

**How it works:** xv6 stores the hart ID in the `tp` register. A syscall calls `cpuid()` and returns it together with the `NCPU` constant from `kernel/param.h`. Run QEMU with `make qemu CPUS=4` to see different core IDs.

**Files modified:** `kernel/sysproc.c`, `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h`, `user/usys.pl`
**New file:** `user/hartinfo.c`

**Test:** `$ hartinfo`

---

## B2. Hardware Timer Reader (`rdtime`)

**What it does:** Returns the value of the RISC-V `time` register so the program can measure elapsed hardware time.

**How it works:** The `r_time()` helper in `kernel/riscv.h` reads the CSR with inline assembly. A syscall returns it. The test program reads the value before and after a loop and prints the difference.

**Files modified:** `kernel/riscv.h` (if the helper is missing), `kernel/sysproc.c`, `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h`, `user/usys.pl`
**New file:** `user/timetest.c`

**Test:** `$ timetest`

---

## B3. Memory Layout Display

**What it does:** Prints the important addresses and sizes used by the kernel.

**Values shown:** `KERNBASE`, `PHYSTOP`, `PGSIZE`, `MAXVA`, `UART0`, `VIRTIO0`, `PLIC`, `TRAMPOLINE`, `TRAPFRAME`.

**How it works:** The constants come from `kernel/memlayout.h` and `kernel/riscv.h`. A syscall prints them with `printf` from kernel space.

**Files modified:** `kernel/sysproc.c`, `kernel/syscall.h`, `kernel/syscall.c`, `user/user.h`, `user/usys.pl`
**New file:** `user/memlayout.c`

**Test:** `$ memlayout`

---

## B4. Readable Exception Names in `usertrap`

**What it does:** When a user program crashes, the kernel prints a human-readable reason instead of only a number.

**How it works:** A small lookup function converts `scause` values (for example 2 = illegal instruction, 13 = load page fault, 15 = store page fault) into text. `usertrap()` in `kernel/trap.c` prints this text next to `sepc` and `stval`.

**Files modified:** `kernel/trap.c`
**New file:** `user/crashtest.c` (dereferences a null pointer on purpose)

**Test:** `$ crashtest`

---

## B5. Data Representation Test

**What it does:** Shows how RISC-V stores data: the size of basic C types and the byte order (endianness).

**How it works:** A pure user program prints `sizeof(char)`, `sizeof(int)`, `sizeof(long)` and `sizeof(void *)`. It then stores `0x01020304` in an integer and prints each byte to show that RISC-V is little-endian.

**Files modified:** `Makefile`
**New file:** `user/datatest.c`

**Test:** `$ datatest`

---

## Module Overview

| ID | Track | Module | Type |
|----|-------|--------|------|
| A1 | OS | `hello` | System call |
| A2 | OS | `syscount` | System call + `proc` change |
| A3 | OS | `touch` | User program |
| A4 | OS | `random` | System call |
| A5 | OS | `getprocname` | System call |
| B1 | CA | Hart information | System call |
| B2 | CA | `rdtime` | System call + CSR |
| B3 | CA | Memory layout | System call |
| B4 | CA | Exception names | Kernel change (`trap.c`) |
| B5 | CA | Data representation | User program |

## Output Screenshots

Screenshots of every module running in QEMU are placed in the `docs/` folder, named by module ID (`A1.png` to `B5.png`).

## Acknowledgements

Based on **xv6-riscv** by MIT PDOS (https://github.com/mit-pdos/xv6-riscv), inspired by *Lions' Commentary on UNIX 6th Edition*. Course reference: https://pdos.csail.mit.edu/6.1810/

All modifications in this repository were made for academic coursework.# New-Project-Work
