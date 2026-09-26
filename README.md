
# Aurora OS

A portable, low-latency operating system focused on predictable performance for interactive workloads — gaming, REST APIs, real-time networking, and distributed systems.

## Status

Early development — boot and VGA.

## Stack

- **C** — drivers and interop
- **Rust** — kernel core, memory, scheduler
- **Assembly** — boot, context switch

## Focus

Aurora OS targets **tail latency** (p99, p999) rather than average throughput. The scheduler is designed for predictability in interactive workloads, not fairness or batch throughput.

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
