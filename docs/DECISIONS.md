# Architecture Decisions

Record of technical decisions for TailOS. Each decision documents context, alternatives, choice, and rationale.

---

## Decision 001: C + Assembly as primary stack

**Status:** Accepted

**Context:**
TailOS targets tail latency (p99, p999) and predictability. The stack must be coherent with this goal: no runtime, no garbage collector, no abstraction layers that introduce unpredictable latency.

**Alternatives considered:**
1. C + Assembly (pure)
2. C + Assembly + Rust from the start
3. C + Assembly + high-level languages

**Decision:**
C + Assembly as primary stack. Rust reserved for critical modules in a future phase. High-level languages permanently discarded.

**Rationale:**
- **C**: full control over memory and hardware; minimal latency; predictability; portability.
- **Assembly**: mandatory for boot, context switch, and architecture-specific instructions; zero overhead.
- **Rust**: evaluated for critical modules (allocator, scheduler) in a future phase. `no_std` eliminates runtime, but introduces interop complexity with C.
- **High-level (Python, Java, Go, Node.js)**: discarded. Runtime, GC, and abstraction layers introduce unpredictable latency.

**Consequences:**
- Greater manual effort in memory management.
- Greater exposure to memory bugs.
- Full control over latency and system behavior.
- Portability to architectures with scarce resources.
- Rust can be introduced incrementally without rewriting the kernel.

**References:**
- Linux Kernel (C + Assembly)
- xv6 (C + Assembly)
- Redox OS (Rust, reference for future phase)
- SerenityOS (C++, architecture reference)

---

## Decision 002: Scope — tail latency, own category

**Status:** Accepted

**Context:**
TailOS needs to define its scope in the operating systems ecosystem. The choice is between competing in general use (against Linux, FreeBSD, Windows) or focusing on a specific problem not solved by general-purpose OSes.

**Alternatives considered:**
1. General-purpose OS
2. Niche OS (tail latency)
3. Own category (low latency + predictability)

**Decision:**
Niche OS focused on tail latency, positioned in its own category: low latency + predictability. Not a competitor to general-purpose OSes.

**Rationale:**
- General-purpose = infinite scope, direct competition with Linux, FreeBSD, Windows.
- Niche = controlled scope, real problem, measurable differential.
- Tail latency (p99, p999) is an unsolved problem in general-purpose OSes.
- Applications: gaming, REST APIs, real-time networking, distributed systems, remote surgery, high-frequency trading, edge computing, critical embedded systems.
- As the world becomes more interactive, more workloads need predictable latency.

**Consequences:**
- Does not compete with Linux, FreeBSD, or Windows.
- Latency-oriented scheduler, not fairness.
- Priority for interactive workloads.
- Benchmarks focused on p99/p999, jitter, and predictability.

**References:**
- QNX (real-time)
- Zephyr (embedded)
- PusOS (edge computing)
- LITMUS^RT (deterministic Linux)

---

## Decision 003: Architecture — monolithic modular

**Status:** Accepted

**Context:**
TailOS needs to define its kernel architecture. The choice is between monolithic, microkernel, or hybrid.

**Alternatives considered:**
1. Pure monolithic
2. Microkernel
3. Hybrid
4. Monolithic modular

**Decision:**
Monolithic modular.

**Rationale:**
- **Pure monolithic**: simpler, but hard to maintain and evolve.
- **Microkernel**: safer and modular, but introduces IPC overhead (unpredictable latency).
- **Hybrid**: compromise, but unnecessary complexity.
- **Monolithic modular**: single kernel in privileged space, organized into modules with clear interfaces. Combines monolithic performance with microkernel modularity.

**Consequences:**
- Drivers and subsystems run in kernel space.
- Clear interfaces between modules.
- Facilitates incremental evolution without rewriting.
- Maintains minimal and predictable latency.

**References:**
- Linux (monolithic modular)
- FreeBSD (monolithic modular)
- SerenityOS (monolithic modular)

---

## Decision 004: Target architecture — x86_64

**Status:** Accepted

**Context:**
TailOS needs to define its initial target hardware architecture. The choice is between x86_64, ARM64, RISC-V, or multiple.

**Alternatives considered:**
1. x86_64
2. ARM64
3. RISC-V
4. Multiple from the start

**Decision:**
x86_64 as initial target architecture. Other architectures evaluated in a future phase.

**Rationale:**
- **x86_64**: dominant architecture; abundant documentation; mature QEMU; OSDev Wiki focused on it.
- **ARM64**: relevant, but documentation less accessible.
- **RISC-V**: promising, but ecosystem still maturing.
- **Multiple from the start**: infinite scope, unfeasible for a solo project.

**Consequences:**
- Boot, GDT, IDT, paging, and context switch specific to x86_64.
- Portability requires future refactoring.
- Focus on one architecture accelerates development.

**References:**
- Linux (supports multiple)
- xv6 (x86)
- SerenityOS (x86_64)

---

## Decision 005: Bootloader — Limine

**Status:** Accepted

**Context:**
TailOS needs a bootloader to load the kernel. The choice is between writing a custom bootloader, using Multiboot2 + GRUB, or using Limine.

**Alternatives considered:**
1. Custom bootloader
2. Multiboot2 + GRUB
3. Limine

**Decision:**
Limine.

**Rationale:**
- **Custom bootloader**: unnecessary scope.
- **Multiboot2 + GRUB**: functional, but complex and has configuration overhead.
- **Limine**: modern, simple, with its own protocol; clear documentation; QEMU-compatible.

**Consequences:**
- Fast and simple boot.
- Less time spent on boot, more time on kernel.

**References:**
- Limine (https://github.com/limine-bootloader/limine)
- SerenityOS (uses Limine)

---

## Decision 006: Memory model — 4-level paging

**Status:** Accepted

**Context:**
TailOS needs to define its virtual memory model. The choice is between segmentation, 2-level paging, 4-level paging, or 5-level paging.

**Alternatives considered:**
1. Segmentation
2. 2-level paging
3. 4-level paging (x86_64 standard)
4. 5-level paging

**Decision:**
4-level paging (x86_64 standard).

**Rationale:**
- **Segmentation**: obsolete on x86_64.
- **2-level paging**: insufficient for 64-bit addressing.
- **4-level paging**: x86_64 standard; supports 48-bit virtual addresses (256 TB).
- **5-level paging**: rare hardware; unnecessary complexity.

**Consequences:**
- Uses PML4, PDPT, PD, and PT.
- Supports 256 TB of virtual address space.
- Huge pages (2 MB, 1 GB) to reduce TLB misses.
- Alignment with x86_64 standard.

**References:**
- Intel SDM Volume 3A (paging)
- Linux (x86_64 uses 4 levels)

---

## Decision 007: Scheduler — tail-latency oriented

**Status:** Accepted

**Context:**
TailOS needs to define its scheduling policy. The choice is between fairness (CFS-like), real-time (fixed priority), or tail-latency oriented.

**Alternatives considered:**
1. Fairness (CFS-like)
2. Real-time (fixed priority)
3. Tail-latency oriented

**Decision:**
Tail-latency oriented scheduler (p99, p999), not fairness or average throughput.

**Rationale:**
- **Fairness (CFS-like)**: optimizes average throughput and fairness, but not tail latency.
- **Real-time (fixed priority)**: guarantees deadlines, but does not adapt to variable interactive workloads.
- **Tail-latency oriented**: prioritizes predictability; reduces p99 and p999; adapts to variable loads.

**Consequences:**
- Priority for interactive tasks.
- CPU isolation for critical tasks.
- Fast preemption.
- Benchmarks focused on p99/p999, jitter, and predictability.
- Trade-off: average throughput may be lower than CFS on batch workloads.

**References:**
- CFS (Linux)
- LITMUS^RT (deterministic Linux)
- QNX (real-time)

---

## Decision 008: Drivers — in kernel space

**Status:** Accepted

**Context:**
TailOS needs to define where drivers run. The choice is between kernel space (monolithic) or user space (microkernel).

**Alternatives considered:**
1. Kernel space (monolithic)
2. User space (microkernel)
3. Hybrid

**Decision:**
Drivers in kernel space.

**Rationale:**
- **Kernel space**: lower latency (no IPC), simpler, aligned with monolithic modular architecture.
- **User space**: greater isolation, but IPC overhead introduces unpredictable latency.
- **Hybrid**: unnecessary complexity.

**Consequences:**
- Drivers have full access to hardware.
- Greater risk of kernel failure due to driver bug.
- Lower latency in I/O operations.
- Alignment with low-latency goal.

**References:**
- Linux (drivers in kernel)
- QNX (drivers in userspace, different trade-off)

---

## Decision 009: License — GPLv2

**Status:** Accepted

**Context:**
TailOS needs to define its license. The choice is between permissive (MIT, BSD, Apache 2.0) and copyleft (GPLv2, GPLv3).

**Alternatives considered:**
1. MIT
2. Apache 2.0
3. GPLv2
4. GPLv3

**Decision:**
GPLv2.

**Rationale:**
- **MIT/Apache 2.0**: permissive; allow proprietary use without contribution back; incompatible with the philosophy of an open, community-driven project.
- **GPLv2**: copyleft; ensures modifications remain open; compatible with the C ecosystem; Linux's choice.
- **GPLv3**: more modern, but incompatible with GPLv2 and some libraries.

**Consequences:**
- Code remains open and community-driven.
- Contributions back are mandatory.
- Incompatibility with Apache 2.0 code (evaluated case by case).
- Alignment with Linux.

**References:**
- Linux (GPLv2)
- FreeBSD (BSD)
- Redox OS (MIT)

---

## Decision 010: Development model — incremental, benchmarks from the start

**Status:** Accepted

**Context:**
TailOS needs to define its development model. The choice is between developing everything and benchmarking at the end, or developing incrementally with benchmarks from the start.

**Alternatives considered:**
1. Develop everything, benchmark at the end
2. Develop incrementally, benchmark from the start

**Decision:**
Incremental development, with benchmarks from the start.

**Rationale:**
- **Benchmark at the end**: risk of discovering latency problems too late.
- **Benchmark from the start**: validates design decisions continuously; detects regressions early; generates data for analysis.
- Alignment with the philosophy of latency as a requirement.

**Consequences:**
- Benchmarks are part of development, not a final step.
- Each module has an associated benchmark.
- Latency data guides design decisions.
- Roadmap includes benchmarks at each phase.

**References:**
- LITMUS^RT (real-time benchmarks)
- Linux (scheduler benchmarks)

---

## Decision 011: Code structure — modular by subsystem

**Status:** Accepted

**Context:**
TailOS needs to define its code structure. The choice is between a monolith of files or a modular structure with clear interfaces.

**Alternatives considered:**
1. Monolith of files
2. Modular structure with clear interfaces

**Decision:**
Modular structure with clear interfaces, organized by subsystem.

**Rationale:**
- **Monolith**: simple at first, but hard to maintain and evolve.
- **Modular**: each subsystem has a clear interface; facilitates incremental evolution; aligned with monolithic modular architecture.

**Consequences:**
- `boot/` — bootloader and linker script.
- `kernel/` — kernel code.
- `kernel/src/memory/` — memory management.
- `kernel/src/sched/` — scheduler.
- `kernel/src/drivers/` — drivers.
- `userspace/` — user space code.
- `docs/` — documentation.
- `scripts/` — build and run scripts.

**References:**
- Linux (modular)
- SerenityOS (modular)

---

## Decision 012: Build system — Makefile

**Status:** Accepted

**Context:**
TailOS needs to define its build system. The choice is between Makefile, CMake, Ninja, or a custom build system.

**Alternatives considered:**
1. Makefile
2. CMake
3. Ninja
4. Custom build system

**Decision:**
Makefile.

**Rationale:**
- **Makefile**: simple, universal, standard in kernel projects; full control.
- **CMake**: complex for a kernel; unnecessary overhead.
- **Ninja**: fast, but generated by another system.
- **Custom**: unnecessary scope.

**Consequences:**
- Makefile defines targets: `build`, `run`, `debug`, `clean`.
- Integration with GCC, NASM, LD, and QEMU.
- Full control over compilation flags.

**References:**
- Linux (Kbuild)
- xv6 (Makefile)
- SerenityOS (Makefile + CMake)

---

## Decision 013: Emulator and debug — QEMU + GDB

**Status:** Accepted

**Context:**
TailOS needs to define its testing and debugging environment. The choice is between real hardware, QEMU, Bochs, VirtualBox, and GDB.

**Alternatives considered:**
1. Real hardware
2. QEMU + GDB
3. Bochs
4. VirtualBox

**Decision:**
QEMU for emulation, GDB for debugging.

**Rationale:**
- **Real hardware**: risky, hard to debug, slow to iterate.
- **QEMU**: mature emulator, x86_64 support, GDB debugging, fast, open source.
- **GDB**: complete inspection of registers, memory, stack; breakpoints; step-by-step.
- **Bochs**: good for debugging, but slow.
- **VirtualBox**: focused on virtualization, not kernel development.

**Consequences:**
- Development and testing in QEMU.
- Debugging with GDB connected to QEMU.
- Tests on real hardware only at milestones.
- Fast iteration.

**References:**
- OSDev Wiki (QEMU + GDB)
- SerenityOS (QEMU + GDB)

---

## Decision 014: Version control and hosting — Git + GitHub

**Status:** Accepted

**Context:**
TailOS needs to define its version control and hosting. The choice is between Git, Mercurial, SVN, and GitHub, GitLab, Codeberg, self-hosted.

**Alternatives considered:**
1. Git + GitHub
2. Git + GitLab
3. Git + Codeberg
4. Mercurial / SVN
5. Self-hosted

**Decision:**
Git for version control, GitHub for hosting.

**Rationale:**
- **Git**: industry standard; mature tools.
- **GitHub**: largest community; visibility; GitHub Actions.
- **GitLab**: good, but less visibility for open source.
- **Codeberg**: ethical, but less visibility.
- **Self-hosted**: unnecessary complexity.

**Consequences:**
- Repository at `https://github.com/beaverbit/tail-os`.
- Frequent and descriptive commits.
- Branches for features.
- Future CI/CD integration.

**References:**
- Linux (Git + GitHub)
- SerenityOS (Git + GitHub)

---

## Decision 015: Versioning — semver

**Status:** Accepted

**Context:**
TailOS needs to define its versioning scheme. The choice is between linear versioning, semver, or date-based.

**Alternatives considered:**
1. Linear (v1, v2, v3)
2. Semver (MAJOR.MINOR.PATCH)
3. Date-based (YYYY.MM.DD)

**Decision:**
Semver.

**Rationale:**
- **Linear**: does not communicate compatibility.
- **Semver**: communicates compatibility; industry standard.
- **Date-based**: does not communicate compatibility.

**Consequences:**
- Versions: MAJOR.MINOR.PATCH.
- MAJOR: incompatible changes.
- MINOR: compatible new features.
- PATCH: compatible fixes.

**References:**
- Semver (https://semver.org)

---

## Decision 016: Language — English in code, Portuguese in internal documentation

**Status:** Accepted

**Context:**
TailOS needs to define its language. The choice is between English, Portuguese, or both.

**Alternatives considered:**
1. English
2. Portuguese
3. Both

**Decision:**
English in code and public documentation; Portuguese in internal documentation.

**Rationale:**
- **English in code**: industry standard; facilitates international contributors.
- **Portuguese in internal documentation**: facilitates solo development.
- **Both**: balance.

**Consequences:**
- Code, comments, and README in English.
- DECISIONS.md in English.
- Facilitates international contributors.
- Facilitates solo development.

**References:**
- Linux (English)
- SerenityOS (English)

---

## Decision 017: Code philosophy — simplicity and clarity

**Status:** Accepted

**Context:**
TailOS needs to define its code philosophy. The choice is between aggressive optimization or simplicity and clarity.

**Alternatives considered:**
1. Aggressive optimization
2. Simplicity and clarity
3. Both

**Decision:**
Simplicity and clarity, with optimization when necessary and measurable.

**Rationale:**
- **Aggressive optimization**: risk of bugs; hard to maintain.
- **Simplicity and clarity**: easy to understand; easy to maintain; foundation for future optimization.
- **Both**: balance.

**Consequences:**
- Clear and readable code.
- Optimization guided by benchmarks, not intuition.
- Comments explain the "why", not the "what".
- Facilitates evolution and contributors.

**References:**
- Linux (simplicity + optimization)
- SerenityOS (clarity)

---

## Decision 018: Project philosophy — latency as a requirement

**Status:** Accepted

**Context:**
TailOS needs to define its project philosophy. The choice is between latency as a requirement or latency as a consequence.

**Alternatives considered:**
1. Latency as a requirement
2. Latency as a consequence

**Decision:**
Latency as a requirement. Every design decision is evaluated by its impact on tail latency.

**Rationale:**
- **Latency as a consequence**: general-purpose OS approach; latency is optimized later.
- **Latency as a requirement**: TailOS approach; latency guides all decisions from the start.

**Consequences:**
- Every feature is evaluated by its impact on latency.
- Benchmarks focused on p99/p999, jitter, predictability.
- Explicit trade-offs: throughput may be lower than general-purpose OSes.
- Clear philosophy guides project evolution.

**References:**
- QNX (latency as a requirement)
- LITMUS^RT (latency as a requirement)
- Linux (latency as a consequence, for contrast)

---

## Decision 019: Evolution philosophy — incremental

**Status:** Accepted

**Context:**
TailOS needs to define its evolution philosophy. The choice is between rewriting or incremental evolution.

**Alternatives considered:**
1. Rewriting
2. Incremental evolution

**Decision:**
Incremental evolution.

**Rationale:**
- **Rewriting**: loss of knowledge; risk of regression.
- **Incremental evolution**: maintains knowledge; adds features without breaking.

**Consequences:**
- Features added incrementally.
- Benchmarks ensure no regression.
- Solid foundation before expanding.
- Facilitates contributors.

**References:**
- Linux (incremental evolution)
- SerenityOS (incremental evolution)

---

## Decision 020: Future phases — CI/CD and contributors

**Status:** Deferred

**Context:**
TailOS needs to define its strategy for CI/CD and contributors. The choice is between setting up now or deferring.

**Alternatives considered:**
1. Set up now
2. Defer to future phase

**Decision:**
Defer to future phase.

**Rationale:**
- **Now**: unnecessary overhead at the start; focus on code.
- **Future**: when the project has contributors and stability.

**Consequences:**
- Focus on code and benchmarks at the start.
- CI/CD and contributors evaluated in the future.
- GitHub Actions evaluated in the future.

**References:**
- Linux
- SerenityOS

---

- **Date:** 2026-09-27
- **End of document.**
