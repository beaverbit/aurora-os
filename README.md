<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/logo-dark.png">
    <source media="(prefers-color-scheme: light)" srcset="assets/logo-light.png">
    <img alt="KinetOS" src="assets/logo-light.png" width="600">
  </picture>
</p>

# KinetOS

A portable, low-latency operating system focused on predictable performance for interactive and critical workloads.

## Focus

KinetOS targets **tail latency** (p99, p999) and **predictability**, not average throughput. The goal is an operating system where latency is a requirement, not a consequence.

Intended for workloads where every microsecond matters: gaming, APIs, real-time networking, distributed systems, critical applications, and embedded systems.

KinetOS is not an operating system for everything. It is an operating system for when **every microsecond matters**.

## Stack

- **C** — kernel, drivers, interop
- **Assembly** — boot, context switch

## Documentation

- [Roadmap](docs/ROADMAP.md) — current status and milestones
- [Kernel Decisions](docs/DECISIONS_KERNEL.md) — architecture, scope, stack, memory, scheduling, drivers
- [Project Decisions](docs/DECISIONS_PROJECT.md) — development process, tooling, licensing, code structure, future phases

## License

GPLv2 — see [LICENSE](LICENSE).