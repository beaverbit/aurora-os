# Roadmap

Current status of KinetOS development. Checkboxes are marked as milestones are reached.

## Phases

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

## Notes

- Each phase is a milestone. A tag is created when a phase is complete.
- Benchmarks are developed alongside each phase, not at the end.
- See `DECISIONS_KERNEL.md` and `DECISIONS_PROJECT.md` for architectural context.