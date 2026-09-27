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
