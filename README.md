# Aurora OS

A portable, low-latency operating system focused on predictable performance for interactive and critical workloads.

## Status

Early development — no working build yet.

## Focus

Aurora OS targets **tail latency** (p99, p999) and **predictability**, not average throughput. The goal is an operating system where latency is a requirement, not a consequence.

Intended for workloads where every microsecond matters: gaming, APIs, real-time networking, distributed systems, critical applications, and embedded systems.

Aurora OS is not an operating system for everything. It is an operating system for when **every microsecond matters**.

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

- [Architecture Decisions](docs/DECISIONS.md)

## License

GPLv2 — see [LICENSE](LICENSE).