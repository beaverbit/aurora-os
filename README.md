# TailOS

Um sistema operacional portátil e de baixa latência, focado em desempenho previsível para cargas de trabalho interativas e críticas.

## Status

Desenvolvimento inicial — ainda sem build funcional.

## Foco

O TailOS tem como alvo a **latência de cauda** (p99, p999) e a **previsibilidade**, não a vazão média. O objetivo é um sistema operacional onde a latência é um requisito, não uma consequência.

Destinado a cargas de trabalho onde cada microssegundo importa: jogos, APIs, redes em tempo real, sistemas distribuídos, aplicações críticas e sistemas embarcados.

O TailOS não é um sistema operacional para tudo. É um sistema operacional para quando **cada microssegundo importa**.

## Stack

- **C** — kernel, drivers, interop
- **Assembly** — boot, troca de contexto

## Roadmap

- [x] Estrutura inicial
- [ ] Boot (Limine/Multiboot2)
- [ ] Modo texto VGA
- [ ] GDT / IDT
- [ ] Memória física e virtual
- [ ] Escalonador (orientado a latência de cauda)
- [ ] Syscalls
- [ ] Espaço de usuário
- [ ] Drivers
- [ ] Benchmarks (latência, jitter, p99/p999)

## Documentação

- [Decisões de Arquitetura](docs/DECISIONS.md)

## Licença

GPLv2 — veja [LICENSE](LICENSE).