# Aurora OS

A portable, low-latency operating system focused on predictable performance for interactive and critical workloads.

## Status

Early development — boot and VGA.

## Stack

- **C** — drivers and interop
- **Assembly** — boot, context switch

## Focus

Aurora OS targets **tail latency** (p99, p999) and **predictability**, not average throughput. The scheduler is designed for workloads where every microsecond matters — not for fairness or batch processing.

The goal is an operating system where latency is a requirement, not a consequence. That means:

- **Gaming** — consistent frame pacing, minimal input lag
- **APIs and services** — predictable response under load
- **Real-time networking** — controlled jitter
- **Distributed systems** — synchronization with deterministic latency
- **Critical applications** — remote surgery, industrial control, automation
- **Embedded systems** — from the datacenter to hardware with scarce RAM

Aurora OS is not an operating system for everything. It is an operating system for when **every microsecond matters**.

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
- [ ] GPU (long term)

## License

GPLv2 — see [LICENSE](LICENSE).