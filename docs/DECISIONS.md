# Architecture Decisions

Record of technical decisions for Aurora OS. Each decision documents context, alternatives, choice, and rationale.

---

## Decision 001: C + Assembly as primary stack

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS targets tail latency (p99, p999) and predictability. The stack must be coherent with this goal: no runtime, no garbage collector, no abstraction layers that introduce unpredictable latency.

**Alternatives considered:**

1. C + Assembly (pure)
2. C + Assembly + Rust from the start
3. C + Assembly + high-level languages

**Decision:**

C + Assembly as primary stack. Rust reserved for critical modules in a future phase. High-level languages permanently discarded.

**Rationale:**

- **C**: full control over memory and hardware; minimal latency; predictability; portability; consolidated kernel ecosystem.
- **Assembly**: mandatory for boot, context switch, and architecture-specific instructions; zero overhead.
- **Rust**: evaluated for critical modules (allocator, scheduler) in a future phase. `no_std` eliminates runtime, but introduces interop complexity with C.
- **High-level (Python, Java, Go, Node.js)**: discarded. Runtime, GC, and abstraction layers introduce unpredictable latency, incompatible with the project's goal.

**Consequences:**

- Greater manual effort in memory management.
- Greater exposure to memory bugs (buffer overflow, use-after-free).
- Full control over latency and system behavior.
- Portability to architectures with scarce resources (limited RAM, no advanced MMU).
- Rust can be introduced incrementally in critical modules without rewriting the kernel.

**References:**

- Linux Kernel (C + Assembly)
- xv6 (C + Assembly)
- Redox OS (Rust, reference for future phase)
- SerenityOS (C++, architecture reference)

---

## Decision 002: Niche scope — tail latency

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its scope in the operating systems ecosystem. The choice is between competing in general use (against Linux, FreeBSD, Windows) or focusing on a specific problem not solved by general-purpose OSes.

**Alternatives considered:**

1. General-purpose OS
2. Niche OS (tail latency)
3. Own category (low latency + predictability)

**Decision:**

Niche OS focused on tail latency, positioned in its own category: low latency + predictability.

**Rationale:**

- General-purpose = infinite scope, direct competition with Linux, FreeBSD, Windows.
- Niche = controlled scope, real problem, measurable differential.
- Tail latency (p99, p999) is an unsolved problem in general-purpose OSes, which optimize average throughput and fairness.
- Applications: gaming, REST APIs, real-time networking, distributed systems, remote surgery, high-frequency trading, edge computing, critical embedded systems.

**Consequences:**

- Latency-oriented scheduler, not fairness.
- Priority for interactive workloads.
- Benchmarks focused on p99/p999, jitter, and predictability.
- Possibility to grow into general use in the future, maintaining latency focus.

**References:**

- QNX (real-time)
- Zephyr (embedded)
- PusOS (edge computing)
- LITMUS^RT (deterministic Linux)

---

## Decision 003: Category — low-latency, predictable OS

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS defines its category in the operating systems ecosystem. The choice is between competing directly with general-purpose OSes or occupying a space not served by them.

**Alternatives considered:**

1. General use (compete with Linux, FreeBSD, Windows)
2. Niche (tail latency)
3. Own category (low latency + predictability)

**Decision:**

Own category. Aurora OS is a low-latency, predictable OS, not a competitor to general-purpose OSes.

**Rationale:**

- Linux, FreeBSD, and Windows have solved general use for decades.
- None of them solves predictable tail latency well.
- Aurora OS occupies a space that general-purpose OSes do not.
- As the world becomes more interactive, more workloads need predictable latency.

**Consequences:**

- Does not compete with Linux, FreeBSD, or Windows.
- Focus on scheduler, memory, and latency.
- Possibility to grow into general use while maintaining latency focus.
- Own category: low latency + predictability.

**References:**

- QNX (real-time)
- Zephyr (embedded)
- PusOS (edge computing)
- LITMUS^RT (deterministic Linux)

---

## Decision 004: Architecture — monolithic modular

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its kernel architecture. The choice is between monolithic, microkernel, or hybrid.

**Alternatives considered:**

1. Pure monolithic
2. Microkernel
3. Hybrid
4. Monolithic modular

**Decision:**

Monolithic modular.

**Rationale:**

- **Pure monolithic**: simpler, but hard to maintain and evolve.
- **Microkernel**: safer and modular, but introduces IPC overhead (unpredictable latency), incompatible with the project's goal.
- **Hybrid**: compromise, but unnecessary complexity for the scope.
- **Monolithic modular**: single kernel in privileged space, but organized into modules with clear interfaces. Combines monolithic performance with microkernel modularity.

**Consequences:**

- Drivers and subsystems run in kernel space.
- Clear interfaces between modules (scheduler, memory, drivers).
- Facilitates incremental evolution without rewriting.
- Maintains minimal and predictable latency.

**References:**

- Linux (monolithic modular)
- FreeBSD (monolithic modular)
- SerenityOS (monolithic modular)

---

## Decision 005: Target architecture — x86_64

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its initial target hardware architecture. The choice is between x86_64, ARM64, RISC-V, or multiple.

**Alternatives considered:**

1. x86_64
2. ARM64
3. RISC-V
4. Multiple from the start

**Decision:**

x86_64 as initial target architecture. Other architectures (ARM64, RISC-V) evaluated in a future phase.

**Rationale:**

- **x86_64**: dominant architecture in desktop, server, and datacenter; abundant documentation; mature QEMU; OSDev Wiki focused on it.
- **ARM64**: relevant in mobile and embedded, but documentation less accessible for hobby.
- **RISC-V**: promising, but ecosystem still maturing.
- **Multiple from the start**: infinite scope, unfeasible for a solo project.

**Consequences:**

- Boot, GDT, IDT, paging, and context switch specific to x86_64.
- Portability to other architectures requires future refactoring.
- Focus on one architecture accelerates initial development.

**References:**

- Linux (supports multiple)
- xv6 (x86)
- SerenityOS (x86_64)

---

## Decision 006: Bootloader — Limine

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs a bootloader to load the kernel. The choice is between writing a custom bootloader, using Multiboot2 + GRUB, or using Limine.

**Alternatives considered:**

1. Custom bootloader
2. Multiboot2 + GRUB
3. Limine

**Decision:**

Limine.

**Rationale:**

- **Custom bootloader**: unnecessary scope; does not add to the project's goal.
- **Multiboot2 + GRUB**: functional, but GRUB is complex and has configuration overhead.
- **Limine**: modern bootloader, simple, with its own protocol; clear documentation; QEMU-compatible; used by projects like SerenityOS.

**Consequences:**

- Fast and simple boot.
- Limine protocol defines the interface between bootloader and kernel.
- Less time spent on boot, more time on kernel.

**References:**

- Limine (https://github.com/limine-bootloader/limine)
- SerenityOS (uses Limine)

---

## Decision 007: Memory model — 4-level paging

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its virtual memory model. The choice is between segmentation, 2-level paging, 4-level paging, or 5-level paging.

**Alternatives considered:**

1. Segmentation
2. 2-level paging
3. 4-level paging (x86_64 standard)
4. 5-level paging

**Decision:**

4-level paging (x86_64 standard).

**Rationale:**

- **Segmentation**: obsolete on x86_64; not used by modern OSes.
- **2-level paging**: insufficient for 64-bit addressing.
- **4-level paging**: x86_64 standard; supports 48-bit virtual addresses (256 TB); sufficient for the scope.
- **5-level paging**: supports 57 bits (128 PB), but hardware still rare; unnecessary complexity.

**Consequences:**

- Uses PML4, PDPT, PD, and PT.
- Supports 256 TB of virtual address space.
- Huge pages (2 MB, 1 GB) to reduce TLB misses.
- Alignment with x86_64 standard.

**References:**

- Intel SDM Volume 3A (paging)
- Linux (x86_64 uses 4 levels, with optional 5-level support)

---

## Decision 008: Scheduler — tail-latency oriented

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its scheduling policy. The choice is between fairness (CFS-like), real-time (fixed priority), or tail-latency oriented.

**Alternatives considered:**

1. Fairness (CFS-like)
2. Real-time (fixed priority)
3. Tail-latency oriented

**Decision:**

Tail-latency oriented scheduler (p99, p999), not fairness or average throughput.

**Rationale:**

- **Fairness (CFS-like)**: optimizes average throughput and fairness, but not tail latency.
- **Real-time (fixed priority)**: guarantees deadlines, but does not adapt to variable interactive workloads.
- **Tail-latency oriented**: prioritizes predictability in interactive workloads; reduces p99 and p999; adapts to variable loads.

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

## Decision 009: Driver model — in kernel space

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define where drivers run. The choice is between kernel space (monolithic) or user space (microkernel).

**Alternatives considered:**

1. Kernel space (monolithic)
2. User space (microkernel)
3. Hybrid

**Decision:**

Drivers in kernel space.

**Rationale:**

- **Kernel space**: lower latency (no IPC), simpler, aligned with monolithic modular architecture.
- **User space**: greater isolation, but IPC overhead introduces unpredictable latency.
- **Hybrid**: unnecessary complexity for the scope.

**Consequences:**

- Drivers have full access to hardware.
- Greater risk of kernel failure due to driver bug.
- Lower latency in I/O operations.
- Alignment with low-latency goal.

**References:**

- Linux (drivers in kernel)
- QNX (drivers in userspace, different trade-off)

---

## Decision 010: License — GPLv2

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its license. The choice is between permissive (MIT, BSD, Apache 2.0) and copyleft (GPLv2, GPLv3).

**Alternatives considered:**

1. MIT
2. Apache 2.0
3. GPLv2
4. GPLv3

**Decision:**

GPLv2.

**Rationale:**

- **MIT/Apache 2.0**: permissive; allow proprietary use without contribution back; incompatible with the philosophy of an open, community-driven project.
- **GPLv2**: copyleft; ensures modifications and distributions remain open; compatible with the C ecosystem; Linux's choice.
- **GPLv3**: more modern, but incompatible with GPLv2 and some libraries; unnecessary complexity for the scope.

**Consequences:**

- Code remains open and community-driven.
- Contributions back are mandatory.
- Incompatibility with Apache 2.0 code (evaluated case by case).
- Alignment with Linux and other open-source kernels.

**References:**

- Linux (GPLv2)
- FreeBSD (BSD)
- Redox OS (MIT)

---

## Decision 011: Philosophy — latency as a requirement, not a consequence

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its design philosophy. The choice is between optimizing for average throughput (like general-purpose OSes) or for predictable latency.

**Alternatives considered:**

1. Average throughput (general use)
2. Predictable latency (focus)
3. Both equally

**Decision:**

Latency as a requirement, not a consequence. Every design decision is evaluated by its impact on tail latency.

**Rationale:**

- General-purpose OSes optimize average throughput; tail latency is a consequence, not a goal.
- Aurora OS inverts the priority: tail latency is the goal; throughput is a consequence.
- This guides all decisions: scheduler, memory, drivers, I/O.
- Coherence with the project's goal.

**Consequences:**

- Every feature is evaluated by its impact on latency.
- Benchmarks focused on p99/p999, jitter, predictability.
- Explicit trade-offs: throughput may be lower than general-purpose OSes.
- Clear philosophy guides project evolution.

**References:**

- QNX (real-time philosophy)
- LITMUS^RT (determinism philosophy)
- Linux (general-use philosophy, for contrast)

---

## Decision 012: Roadmap — incremental, with benchmarks from the start

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its development roadmap. The choice is between developing everything and benchmarking at the end, or developing incrementally with benchmarks from the start.

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
- Each module (memory, scheduler, drivers) has an associated benchmark.
- Latency data guides design decisions.
- Roadmap includes benchmarks at each phase.

**References:**

- LITMUS^RT (real-time benchmarks)
- Linux (scheduler benchmarks)

---

## Decision 013: Code structure — modular, with clear interfaces

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its code structure. The choice is between a monolith of files or a modular structure with clear interfaces.

**Alternatives considered:**

1. Monolith of files
2. Modular structure with clear interfaces

**Decision:**

Modular structure with clear interfaces.

**Rationale:**

- **Monolith**: simple at first, but hard to maintain and evolve.
- **Modular**: each subsystem (memory, scheduler, drivers) has a clear interface; facilitates incremental evolution; aligned with monolithic modular architecture.

**Consequences:**

- Directories organized by subsystem.
- Documented interfaces between modules.
- Facilitates testing and benchmarks per module.
- Incremental evolution without rewriting.

**References:**

- Linux (modular)
- SerenityOS (modular)

---

## Decision 014: Versioning — semver

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its versioning scheme. The choice is between linear versioning, semver, or date-based.

**Alternatives considered:**

1. Linear (v1, v2, v3)
2. Semver (MAJOR.MINOR.PATCH)
3. Date-based (YYYY.MM.DD)

**Decision:**

Semver.

**Rationale:**

- **Linear**: does not communicate compatibility.
- **Semver**: communicates compatibility; industry standard; facilitates integration.
- **Date-based**: does not communicate compatibility.

**Consequences:**

- Versions: MAJOR.MINOR.PATCH.
- MAJOR: incompatible changes.
- MINOR: compatible new features.
- PATCH: compatible fixes.
- Alignment with industry standard.

**References:**

- Semver (https://semver.org)
- Linux (own versioning)

---

## Decision 015: Documentation — decisions recorded in DECISIONS.md

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define how to document technical decisions. The choice is between dispersed documentation or a centralized decision record.

**Alternatives considered:**

1. Dispersed documentation
2. Centralized record (DECISIONS.md)
3. ADR (Architecture Decision Records)

**Decision:**

Centralized record in `docs/DECISIONS.md`, inspired by ADR.

**Rationale:**

- **Dispersed**: hard to find; loses context.
- **Centralized**: easy to find; maintains context; guides evolution.
- **ADR**: industry standard; adapted to the project's scope.

**Consequences:**

- Each decision has context, alternatives, choice, rationale, and consequences.
- Decisions are versioned alongside code.
- Facilitates onboarding of future contributors.
- Guides project evolution.

**References:**

- ADR (https://adr.github.io)
- Linux (dispersed documentation, but with kernel-doc)

---

## Decision 016: Build system — Makefile

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its build system. The choice is between Makefile, CMake, Ninja, or a custom build system.

**Alternatives considered:**

1. Makefile
2. CMake
3. Ninja
4. Custom build system

**Decision:**

Makefile.

**Rationale:**

- **Makefile**: simple, universal, standard in kernel projects; full control over the process.
- **CMake**: complex for a kernel; unnecessary overhead.
- **Ninja**: fast, but generated by another system (CMake, Meson).
- **Custom build system**: unnecessary scope.

**Consequences:**

- Makefile defines targets: `build`, `run`, `debug`, `clean`.
- Integration with GCC, NASM, LD, and QEMU.
- Full control over compilation flags.
- Simplicity and portability.

**References:**

- Linux (Kbuild, Make-based)
- xv6 (Makefile)
- SerenityOS (Makefile + CMake)

---

## Decision 017: Emulator — QEMU

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its testing environment. The choice is between real hardware, QEMU, Bochs, or VirtualBox.

**Alternatives considered:**

1. Real hardware
2. QEMU
3. Bochs
4. VirtualBox

**Decision:**

QEMU.

**Rationale:**

- **Real hardware**: risky, hard to debug, slow to iterate.
- **QEMU**: mature emulator, x86_64 support, GDB debugging, fast, open source.
- **Bochs**: good for debugging, but slow.
- **VirtualBox**: focused on virtualization, not kernel development.

**Consequences:**

- Development and testing in QEMU.
- Debugging with GDB + QEMU.
- Tests on real hardware only at milestones.
- Fast iteration.

**References:**

- OSDev Wiki (QEMU)
- SerenityOS (QEMU)

---

## Decision 018: Debugging — GDB + QEMU

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its debugging strategy. The choice is between printf, GDB + QEMU, or specific tools.

**Alternatives considered:**

1. printf
2. GDB + QEMU
3. Specific tools

**Decision:**

GDB + QEMU.

**Rationale:**

- **printf**: useful, but limited; does not allow state inspection.
- **GDB + QEMU**: complete inspection of registers, memory, stack; breakpoints; step-by-step.
- **Specific tools**: unnecessary complexity.

**Consequences:**

- Debugging with GDB connected to QEMU.
- Breakpoints in kernel code.
- Inspection of registers, memory, and stack.
- Combination with printf for logs.

**References:**

- OSDev Wiki (GDB + QEMU)
- SerenityOS (GDB + QEMU)

---

## Decision 019: Version control — Git

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its version control system. The choice is between Git, Mercurial, or SVN.

**Alternatives considered:**

1. Git
2. Mercurial
3. SVN

**Decision:**

Git.

**Rationale:**

- **Git**: industry standard; GitHub; mature tools.
- **Mercurial**: good, but less popular.
- **SVN**: centralized, obsolete for new projects.

**Consequences:**

- Repository on GitHub.
- Frequent and descriptive commits.
- Branches for features.
- Future CI/CD integration.

**References:**

- Linux (Git)
- SerenityOS (Git)

---

## Decision 020: Hosting — GitHub

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define where to host its repository. The choice is between GitHub, GitLab, Codeberg, or self-hosted.

**Alternatives considered:**

1. GitHub
2. GitLab
3. Codeberg
4. Self-hosted

**Decision:**

GitHub.

**Rationale:**

- **GitHub**: largest community; mature tools; visibility.
- **GitLab**: good, but less visibility for open-source projects.
- **Codeberg**: ethical, but less visibility.
- **Self-hosted**: unnecessary complexity.

**Consequences:**

- Repository at `https://github.com/beaverbit/aurora-os`.
- Visibility for contributors.
- Future GitHub Actions integration.
- Ease of collaboration.

**References:**

- Linux (GitHub)
- SerenityOS (GitHub)

---

## Decision 021: Folder structure — modular by subsystem

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its folder structure. The choice is between a flat structure or modular by subsystem.

**Alternatives considered:**

1. Flat structure
2. Modular by subsystem

**Decision:**

Modular by subsystem.

**Rationale:**

- **Flat**: simple at first, but hard to maintain.
- **Modular**: each subsystem has its folder; facilitates navigation; aligned with monolithic modular architecture.

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

## Decision 022: Testing — benchmarks from the start

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its testing strategy. The choice is between unit tests, benchmarks, or both.

**Alternatives considered:**

1. Unit tests
2. Benchmarks
3. Both

**Decision:**

Both, with focus on benchmarks from the start.

**Rationale:**

- **Unit tests**: ensure correctness.
- **Benchmarks**: ensure predictable latency.
- **Both**: correctness + performance.

**Consequences:**

- Unit tests for critical functions.
- Benchmarks for latency, jitter, p99/p999.
- Data guides design decisions.
- Regressions detected early.

**References:**

- LITMUS^RT (benchmarks)
- Linux (tests + benchmarks)

---

## Decision 023: CI/CD — future

**Date:** 2026-09-27

**Status:** Deferred

**Context:**

Aurora OS needs to define its CI/CD strategy. The choice is between setting up now or deferring.

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
- CI/CD set up when there are contributors.
- GitHub Actions evaluated in the future.

**References:**

- Linux (mature CI/CD)
- SerenityOS (mature CI/CD)

---

## Decision 024: Contributors — future

**Date:** 2026-09-27

**Status:** Deferred

**Context:**

Aurora OS needs to define its contributor strategy. The choice is between opening to contributors now or focusing on solo development initially.

**Alternatives considered:**

1. Open now
2. Focus on solo initially

**Decision:**

Focus on solo initially.

**Rationale:**

- **Now**: coordination complexity; lack of solid foundation.
- **Solo initially**: solid foundation before opening; mature documentation; clear vision.

**Consequences:**

- Solo development at the start.
- Mature documentation before opening.
- Contributors evaluated in the future.
- Clear vision guides evolution.

**References:**

- Linux (started solo)
- SerenityOS (started solo)

---

## Decision 025: Language — English in code, Portuguese in internal documentation

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its language. The choice is between English, Portuguese, or both.

**Alternatives considered:**

1. English
2. Portuguese
3. Both

**Decision:**

English in code and public documentation; Portuguese in internal documentation.

**Rationale:**

- **English in code**: industry standard; facilitates international contributors.
- **Portuguese in internal documentation**: facilitates solo development; documents like DECISIONS.md can be in Portuguese.
- **Both**: balance.

**Consequences:**

- Code, comments, and README in English.
- DECISIONS.md and internal documentation in Portuguese (translated to English when public).
- Facilitates international contributors.
- Facilitates solo development.

**References:**

- Linux (English)
- SerenityOS (English)

---

## Decision 026: Code philosophy — simplicity and clarity

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its code philosophy. The choice is between aggressive optimization or simplicity and clarity.

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

## Decision 027: Project philosophy — latency as a requirement

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its project philosophy. The choice is between latency as a requirement or latency as a consequence.

**Alternatives considered:**

1. Latency as a requirement
2. Latency as a consequence

**Decision:**

Latency as a requirement.

**Rationale:**

- **Latency as a consequence**: general-purpose OS approach; latency is optimized later.
- **Latency as a requirement**: Aurora OS approach; latency guides all decisions from the start.

**Consequences:**

- Every design decision is evaluated by its impact on latency.
- Benchmarks from the start.
- Explicit trade-offs.
- Coherence with the project's goal.

**References:**

- QNX (latency as a requirement)
- LITMUS^RT (latency as a requirement)
- Linux (latency as a consequence, for contrast)

---

## Decision 028: Evolution philosophy — incremental

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its evolution philosophy. The choice is between rewriting or incremental evolution.

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

## Decision 029: Documentation philosophy — recorded decisions

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its documentation philosophy. The choice is between dispersed documentation or a centralized record.

**Alternatives considered:**

1. Dispersed documentation
2. Centralized record

**Decision:**

Centralized record in `docs/DECISIONS.md`.

**Rationale:**

- **Dispersed**: hard to find; loses context.
- **Centralized**: easy to find; maintains context; guides evolution.

**Consequences:**

- Each decision has context, alternatives, choice, rationale, and consequences.
- Decisions versioned alongside code.
- Facilitates onboarding of future contributors.
- Guides project evolution.

**References:**

- ADR (https://adr.github.io)

---

## Decision 030: Benchmark philosophy — from the start

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its benchmark philosophy. The choice is between benchmarks at the end or from the start.

**Alternatives considered:**

1. Benchmarks at the end
2. Benchmarks from the start

**Decision:**

Benchmarks from the start.

**Rationale:**

- **At the end**: risk of discovering problems too late.
- **From the start**: validates decisions continuously; detects regressions early; generates data.

**Consequences:**

- Benchmarks are part of development.
- Each module has an associated benchmark.
- Data guides design decisions.
- Roadmap includes benchmarks at each phase.

**References:**

- LITMUS^RT (benchmarks)
- Linux (benchmarks)

---

## Decision 031: Testing philosophy — unit tests + benchmarks

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its testing philosophy. The choice is between unit tests, benchmarks, or both.

**Alternatives considered:**

1. Unit tests
2. Benchmarks
3. Both

**Decision:**

Both.

**Rationale:**

- **Unit tests**: ensure correctness.
- **Benchmarks**: ensure predictable latency.
- **Both**: correctness + performance.

**Consequences:**

- Unit tests for critical functions.
- Benchmarks for latency, jitter, p99/p999.
- Data guides design decisions.
- Regressions detected early.

**References:**

- LITMUS^RT (benchmarks)
- Linux (tests + benchmarks)

---

## Decision 032: Contribution philosophy — future

**Date:** 2026-09-27

**Status:** Deferred

**Context:**

Aurora OS needs to define its contribution philosophy. The choice is between opening now or focusing on solo development initially.

**Alternatives considered:**

1. Open now
2. Focus on solo initially

**Decision:**

Focus on solo initially.

**Rationale:**

- **Now**: coordination complexity; lack of solid foundation.
- **Solo initially**: solid foundation before opening; mature documentation; clear vision.

**Consequences:**

- Solo development at the start.
- Mature documentation before opening.
- Contributors evaluated in the future.
- Clear vision guides evolution.

**References:**

- Linux (started solo)
- SerenityOS (started solo)

---

## Decision 033: License philosophy — GPLv2

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its license philosophy. The choice is between permissive and copyleft.

**Alternatives considered:**

1. MIT/Apache 2.0
2. GPLv2
3. GPLv3

**Decision:**

GPLv2.

**Rationale:**

- **MIT/Apache 2.0**: permissive; allow proprietary use without contribution back.
- **GPLv2**: copyleft; ensures modifications remain open; compatible with the C ecosystem.
- **GPLv3**: more modern, but incompatible with GPLv2 and some libraries.

**Consequences:**

- Code remains open and community-driven.
- Contributions back are mandatory.
- Incompatibility with Apache 2.0 code.
- Alignment with Linux.

**References:**

- Linux (GPLv2)
- FreeBSD (BSD)
- Redox OS (MIT)

---

## Decision 034: Versioning philosophy — semver

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its versioning philosophy. The choice is between linear, semver, or date-based.

**Alternatives considered:**

1. Linear
2. Semver
3. Date-based

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

## Decision 035: Build philosophy — Makefile

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its build philosophy. The choice is between Makefile, CMake, Ninja, or custom.

**Alternatives considered:**

1. Makefile
2. CMake
3. Ninja
4. Custom

**Decision:**

Makefile.

**Rationale:**

- **Makefile**: simple, universal, standard in kernel projects.
- **CMake**: complex for a kernel.
- **Ninja**: fast, but generated by another system.
- **Custom**: unnecessary scope.

**Consequences:**

- Makefile defines targets: `build`, `run`, `debug`, `clean`.
- Integration with GCC, NASM, LD, and QEMU.
- Full control over flags.

**References:**

- Linux (Kbuild)
- xv6 (Makefile)
- SerenityOS (Makefile + CMake)

---

## Decision 036: Debug philosophy — GDB + QEMU

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its debug philosophy. The choice is between printf, GDB + QEMU, or specific tools.

**Alternatives considered:**

1. printf
2. GDB + QEMU
3. Specific tools

**Decision:**

GDB + QEMU.

**Rationale:**

- **printf**: useful, but limited.
- **GDB + QEMU**: complete inspection; breakpoints; step-by-step.
- **Specific tools**: unnecessary complexity.

**Consequences:**

- Debugging with GDB connected to QEMU.
- Breakpoints in kernel code.
- Inspection of registers, memory, and stack.
- Combination with printf for logs.

**References:**

- OSDev Wiki (GDB + QEMU)
- SerenityOS (GDB + QEMU)

---

## Decision 037: Structure philosophy — modular by subsystem

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its structure philosophy. The choice is between flat or modular by subsystem.

**Alternatives considered:**

1. Flat
2. Modular by subsystem

**Decision:**

Modular by subsystem.

**Rationale:**

- **Flat**: simple at first, but hard to maintain.
- **Modular**: each subsystem has its folder; facilitates navigation.

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

## Decision 038: Memory philosophy — 4-level paging

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its memory philosophy. The choice is between segmentation, 2-level paging, 4-level paging, or 5-level paging.

**Alternatives considered:**

1. Segmentation
2. 2-level paging
3. 4-level paging
4. 5-level paging

**Decision:**

4-level paging.

**Rationale:**

- **Segmentation**: obsolete on x86_64.
- **2-level**: insufficient for 64-bit.
- **4-level**: x86_64 standard; supports 48 bits (256 TB).
- **5-level**: rare hardware; unnecessary complexity.

**Consequences:**

- Uses PML4, PDPT, PD, and PT.
- Supports 256 TB of virtual address space.
- Huge pages (2 MB, 1 GB).
- Alignment with x86_64.

**References:**

- Intel SDM Volume 3A
- Linux (x86_64 uses 4 levels)

---

## Decision 039: Scheduler philosophy — tail-latency oriented

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its scheduler philosophy. The choice is between fairness, real-time, or tail-latency oriented.

**Alternatives considered:**

1. Fairness (CFS-like)
2. Real-time (fixed priority)
3. Tail-latency oriented

**Decision:**

Tail-latency oriented.

**Rationale:**

- **Fairness**: optimizes average throughput; not tail latency.
- **Real-time**: guarantees deadlines; does not adapt to interactive workloads.
- **Tail latency**: prioritizes predictability; reduces p99 and p999.

**Consequences:**

- Priority for interactive tasks.
- CPU isolation.
- Fast preemption.
- Benchmarks focused on p99/p999.
- Trade-off: average throughput may be lower.

**References:**

- CFS (Linux)
- LITMUS^RT
- QNX

---

## Decision 040: Driver philosophy — in kernel space

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its driver philosophy. The choice is between kernel space or user space.

**Alternatives considered:**

1. Kernel space
2. User space
3. Hybrid

**Decision:**

Kernel space.

**Rationale:**

- **Kernel**: lower latency (no IPC); simpler.
- **User**: greater isolation; IPC overhead.
- **Hybrid**: unnecessary complexity.

**Consequences:**

- Drivers have full access to hardware.
- Greater risk of kernel failure.
- Lower latency in I/O.
- Alignment with low latency.

**References:**

- Linux (drivers in kernel)
- QNX (drivers in userspace)

---

## Decision 041: Bootloader philosophy — Limine

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its bootloader philosophy. The choice is between custom, Multiboot2 + GRUB, or Limine.

**Alternatives considered:**

1. Custom
2. Multiboot2 + GRUB
3. Limine

**Decision:**

Limine.

**Rationale:**

- **Custom**: unnecessary scope.
- **Multiboot2 + GRUB**: functional, but complex.
- **Limine**: modern, simple, clear documentation.

**Consequences:**

- Fast and simple boot.
- Limine protocol defines the interface.
- Less time on boot.

**References:**

- Limine (https://github.com/limine-bootloader/limine)
- SerenityOS (uses Limine)

---

## Decision 042: Architecture philosophy — monolithic modular

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its architecture philosophy. The choice is between pure monolithic, microkernel, hybrid, or monolithic modular.

**Alternatives considered:**

1. Pure monolithic
2. Microkernel
3. Hybrid
4. Monolithic modular

**Decision:**

Monolithic modular.

**Rationale:**

- **Pure monolithic**: simple, but hard to maintain.
- **Microkernel**: safe, but IPC overhead.
- **Hybrid**: unnecessary complexity.
- **Monolithic modular**: performance + modularity.

**Consequences:**

- Drivers and subsystems in kernel space.
- Clear interfaces between modules.
- Incremental evolution.
- Minimal latency.

**References:**

- Linux (monolithic modular)
- FreeBSD (monolithic modular)
- SerenityOS (monolithic modular)

---

## Decision 043: Target architecture philosophy — x86_64

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its target architecture philosophy. The choice is between x86_64, ARM64, RISC-V, or multiple.

**Alternatives considered:**

1. x86_64
2. ARM64
3. RISC-V
4. Multiple

**Decision:**

x86_64.

**Rationale:**

- **x86_64**: dominant; abundant documentation; mature QEMU.
- **ARM64**: relevant, but less accessible documentation.
- **RISC-V**: promising, but maturing ecosystem.
- **Multiple**: infinite scope.

**Consequences:**

- Boot, GDT, IDT, paging, and context switch specific to x86_64.
- Portability requires future refactoring.
- Focus accelerates development.

**References:**

- Linux (multiple)
- xv6 (x86)
- SerenityOS (x86_64)

---

## Decision 044: Emulator philosophy — QEMU

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its emulator philosophy. The choice is between real hardware, QEMU, Bochs, or VirtualBox.

**Alternatives considered:**

1. Real hardware
2. QEMU
3. Bochs
4. VirtualBox

**Decision:**

QEMU.

**Rationale:**

- **Real hardware**: risky, slow to iterate.
- **QEMU**: mature, GDB debugging, fast.
- **Bochs**: good for debugging, slow.
- **VirtualBox**: focused on virtualization.

**Consequences:**

- Development and testing in QEMU.
- Debugging with GDB + QEMU.
- Tests on real hardware at milestones.

**References:**

- OSDev Wiki (QEMU)
- SerenityOS (QEMU)

---

## Decision 045: Version control philosophy — Git

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its version control philosophy. The choice is between Git, Mercurial, or SVN.

**Alternatives considered:**

1. Git
2. Mercurial
3. SVN

**Decision:**

Git.

**Rationale:**

- **Git**: industry standard; GitHub; mature tools.
- **Mercurial**: good, but less popular.
- **SVN**: centralized, obsolete.

**Consequences:**

- Repository on GitHub.
- Frequent commits.
- Branches for features.
- Future CI/CD.

**References:**

- Linux (Git)
- SerenityOS (Git)

---

## Decision 046: Hosting philosophy — GitHub

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its hosting philosophy. The choice is between GitHub, GitLab, Codeberg, or self-hosted.

**Alternatives considered:**

1. GitHub
2. GitLab
3. Codeberg
4. Self-hosted

**Decision:**

GitHub.

**Rationale:**

- **GitHub**: largest community; visibility.
- **GitLab**: good, but less visibility.
- **Codeberg**: ethical, but less visibility.
- **Self-hosted**: unnecessary complexity.

**Consequences:**

- Repository at `https://github.com/beaverbit/aurora-os`.
- Visibility for contributors.
- Future GitHub Actions.
- Ease of collaboration.

**References:**

- Linux (GitHub)
- SerenityOS (GitHub)

---

## Decision 047: Language philosophy — English in code, Portuguese in internal documentation

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its language philosophy. The choice is between English, Portuguese, or both.

**Alternatives considered:**

1. English
2. Portuguese
3. Both

**Decision:**

English in code and public documentation; Portuguese in internal documentation.

**Rationale:**

- **English in code**: industry standard; international contributors.
- **Portuguese in internal documentation**: facilitates solo development.
- **Both**: balance.

**Consequences:**

- Code, comments, and README in English.
- DECISIONS.md in Portuguese (translated when public).
- Facilitates contributors.
- Facilitates solo development.

**References:**

- Linux (English)
- SerenityOS (English)

---

## Decision 048: Code philosophy — simplicity and clarity

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its code philosophy. The choice is between aggressive optimization or simplicity and clarity.

**Alternatives considered:**

1. Aggressive optimization
2. Simplicity and clarity
3. Both

**Decision:**

Simplicity and clarity, with optimization when necessary and measurable.

**Rationale:**

- **Aggressive optimization**: risk of bugs; hard to maintain.
- **Simplicity and clarity**: easy to understand; easy to maintain.
- **Both**: balance.

**Consequences:**

- Clear and readable code.
- Optimization guided by benchmarks.
- Comments explain the "why".
- Facilitates evolution.

**References:**

- Linux (simplicity + optimization)
- SerenityOS (clarity)

---

## Decision 049: Evolution philosophy — incremental

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its evolution philosophy. The choice is between rewriting or incremental evolution.

**Alternatives considered:**

1. Rewriting
2. Incremental evolution

**Decision:**

Incremental evolution.

**Rationale:**

- **Rewriting**: loss of knowledge; risk of regression.
- **Incremental**: maintains knowledge; adds features without breaking.

**Consequences:**

- Features added incrementally.
- Benchmarks ensure no regression.
- Solid foundation before expanding.
- Facilitates contributors.

**References:**

- Linux (incremental evolution)
- SerenityOS (incremental evolution)

---

## Decision 050: Documentation philosophy — recorded decisions

**Date:** 2026-09-27

**Status:** Accepted

**Context:**

Aurora OS needs to define its documentation philosophy. The choice is between dispersed documentation or a centralized record.

**Alternatives considered:**

1. Dispersed
2. Centralized

**Decision:**

Centralized record in `docs/DECISIONS.md`.

**Rationale:**

- **Dispersed**: hard to find; loses context.
- **Centralized**: easy to find; maintains context.

**Consequences:**

- Each decision has context, alternatives, choice, rationale, and consequences.
- Decisions versioned alongside code.
- Facilitates onboarding of contributors.
- Guides project evolution.

**References:**

- ADR (https://adr.github.io)

---

End of document.