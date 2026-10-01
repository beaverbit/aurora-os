<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.png">
    <source media="(prefers-color-scheme: light)" srcset="assets/logo-light.png">
    <img alt="TailOS" src="assets/logo-light.png" width="600">
  </picture>
</p>

# TailOS

A portable, low-latency operating system focused on predictable performance for interactive and critical workloads.

## Focus

TailOS targets **tail latency** (p99, p999) and **predictability**, not average throughput. The goal is an operating system where latency is a requirement, not a consequence.

Intended for workloads where every microsecond matters: gaming, APIs, real-time networking, distributed systems, critical applications, and embedded systems.

TailOS is not an operating system for everything. It is an operating system for when **every microsecond matters**.

## Stack

- **C** — kernel, drivers, interop
- **Assembly** — boot, context switch

## Roadmap

- [x] Initial structure
- [ ] Boot (Limine/Multiboot2)
- [ ] VGA text mode
- [ ] GDT / IDT
- [ ] Physical and virtual memory
- [ ] Scheduler (tail-latency oriented)
- [ ] Syscalls
- [ ] Userspace
- [ ] Drivers
- [ ] Benchmarks (latency, jitter, p99/p999)

## Documentation

Architecture Decision Records (ADRs) for TailOS:

- [Kernel Decisions](docs/DECISIONS_KERNEL.md) — architecture, scope, stack, memory, scheduling, drivers
- [Project Decisions](docs/DECISIONS_PROJECT.md) — development process, tooling, licensing, code structure, future phases
## License

GPLv2 — see [LICENSE](LICENSE).
