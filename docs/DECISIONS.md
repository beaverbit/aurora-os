# Decisões de Arquitetura

Registro de decisões técnicas para o TailOS. Cada decisão documenta contexto, alternativas, escolha e justificativa.

---

## Decisão 001: C + Assembly como stack principal

**Status:** Aceita

**Contexto:**
O TailOS tem como alvo a latência de cauda (p99, p999) e a previsibilidade. A stack deve ser coerente com esse objetivo: sem runtime, sem garbage collector, sem camadas de abstração que introduzam latência imprevisível.

**Alternativas consideradas:**
1. C + Assembly (puro)
2. C + Assembly + Rust desde o início
3. C + Assembly + linguagens de alto nível

**Decisão:**
C + Assembly como stack principal. Rust reservado para módulos críticos em uma fase futura. Linguagens de alto nível permanentemente descartadas.

**Justificativa:**
- **C**: controle total sobre memória e hardware; latência mínima; previsibilidade; portabilidade.
- **Assembly**: obrigatório para boot, troca de contexto e instruções específicas de arquitetura; zero overhead.
- **Rust**: avaliado para módulos críticos (alocador, escalonador) em uma fase futura. `no_std` elimina o runtime, mas introduz complexidade de interop com C.
- **Alto nível (Python, Java, Go, Node.js)**: descartadas. Runtime, GC e camadas de abstração introduzem latência imprevisível.

**Consequências:**
- Maior esforço manual no gerenciamento de memória.
- Maior exposição a bugs de memória.
- Controle total sobre latência e comportamento do sistema.
- Portabilidade para arquiteturas com recursos escassos.
- Rust pode ser introduzido incrementalmente sem reescrever o kernel.

**Referências:**
- Linux Kernel (C + Assembly)
- xv6 (C + Assembly)
- Redox OS (Rust, referência para fase futura)
- SerenityOS (C++, referência de arquitetura)

---

## Decisão 002: Escopo — latência de cauda, categoria própria

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu escopo no ecossistema de sistemas operacionais. A escolha é entre competir em uso geral (contra Linux, FreeBSD, Windows) ou focar em um problema específico não resolvido por SOs de propósito geral.

**Alternativas consideradas:**
1. SO de propósito geral
2. SO de nicho (latência de cauda)
3. Categoria própria (baixa latência + previsibilidade)

**Decisão:**
SO de nicho focado em latência de cauda, posicionado em sua própria categoria: baixa latência + previsibilidade. Não é concorrente de SOs de propósito geral.

**Justificativa:**
- Propósito geral = escopo infinito, concorrência direta com Linux, FreeBSD, Windows.
- Nicho = escopo controlado, problema real, diferencial mensurável.
- Latência de cauda (p99, p999) é um problema não resolvido em SOs de propósito geral.
- Aplicações: jogos, APIs REST, redes em tempo real, sistemas distribuídos, cirurgia remota, trading de alta frequência, edge computing, sistemas embarcados críticos.
- À medida que o mundo se torna mais interativo, mais cargas de trabalho precisam de latência previsível.

**Consequências:**
- Não compete com Linux, FreeBSD ou Windows.
- Escalonador orientado a latência, não a justiça.
- Prioridade para cargas de trabalho interativas.
- Benchmarks focados em p99/p999, jitter e previsibilidade.

**Referências:**
- QNX (tempo real)
- Zephyr (embarcados)
- PusOS (edge computing)
- LITMUS^RT (Linux determinístico)

---

## Decisão 003: Arquitetura — monolítica modular

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua arquitetura de kernel. A escolha é entre monolítica, microkernel ou híbrida.

**Alternativas consideradas:**
1. Monolítica pura
2. Microkernel
3. Híbrida
4. Monolítica modular

**Decisão:**
Monolítica modular.

**Justificativa:**
- **Monolítica pura**: mais simples, mas difícil de manter e evoluir.
- **Microkernel**: mais seguro e modular, mas introduz overhead de IPC (latência imprevisível).
- **Híbrida**: compromisso, mas complexidade desnecessária.
- **Monolítica modular**: kernel único em espaço privilegiado, organizado em módulos com interfaces claras. Combina performance monolítica com modularidade de microkernel.

**Consequências:**
- Drivers e subsistemas rodam em espaço de kernel.
- Interfaces claras entre módulos.
- Facilita evolução incremental sem reescrever.
- Mantém latência mínima e previsível.

**Referências:**
- Linux (monolítica modular)
- FreeBSD (monolítica modular)
- SerenityOS (monolítica modular)

---

## Decisão 004: Arquitetura alvo — x86_64

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua arquitetura de hardware alvo inicial. A escolha é entre x86_64, ARM64, RISC-V ou múltiplas.

**Alternativas consideradas:**
1. x86_64
2. ARM64
3. RISC-V
4. Múltiplas desde o início

**Decisão:**
x86_64 como arquitetura alvo inicial. Outras arquiteturas avaliadas em uma fase futura.

**Justificativa:**
- **x86_64**: arquitetura dominante; documentação abundante; QEMU maduro; OSDev Wiki focado nela.
- **ARM64**: relevante, mas documentação menos acessível.
- **RISC-V**: promissora, mas ecossistema ainda em maturação.
- **Múltiplas desde o início**: escopo infinito, inviável para um projeto solo.

**Consequências:**
- Boot, GDT, IDT, paginação e troca de contexto específicos para x86_64.
- Portabilidade exige refatoração futura.
- Foco em uma arquitetura acelera o desenvolvimento.

**Referências:**
- Linux (suporta múltiplas)
- xv6 (x86)
- SerenityOS (x86_64)

---

## Decisão 005: Bootloader — Limine

**Status:** Aceita

**Contexto:**
O TailOS precisa de um bootloader para carregar o kernel. A escolha é entre escrever um bootloader customizado, usar Multiboot2 + GRUB ou usar Limine.

**Alternativas consideradas:**
1. Bootloader customizado
2. Multiboot2 + GRUB
3. Limine

**Decisão:**
Limine.

**Justificativa:**
- **Bootloader customizado**: escopo desnecessário.
- **Multiboot2 + GRUB**: funcional, mas complexo e com overhead de configuração.
- **Limine**: moderno, simples, com protocolo próprio; documentação clara; compatível com QEMU.

**Consequências:**
- Boot rápido e simples.
- Menos tempo gasto no boot, mais tempo no kernel.

**Referências:**
- Limine (https://github.com/limine-bootloader/limine)
- SerenityOS (usa Limine)

---

## Decisão 006: Modelo de memória — paginação de 4 níveis

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu modelo de memória virtual. A escolha é entre segmentação, paginação de 2 níveis, paginação de 4 níveis ou paginação de 5 níveis.

**Alternativas consideradas:**
1. Segmentação
2. Paginação de 2 níveis
3. Paginação de 4 níveis (padrão x86_64)
4. Paginação de 5 níveis

**Decisão:**
Paginação de 4 níveis (padrão x86_64).

**Justificativa:**
- **Segmentação**: obsoleta no x86_64.
- **Paginação de 2 níveis**: insuficiente para endereçamento de 64 bits.
- **Paginação de 4 níveis**: padrão x86_64; suporta endereços virtuais de 48 bits (256 TB).
- **Paginação de 5 níveis**: hardware raro; complexidade desnecessária.

**Consequências:**
- Usa PML4, PDPT, PD e PT.
- Suporta 256 TB de espaço de endereço virtual.
- Páginas grandes (2 MB, 1 GB) para reduzir faltas de TLB.
- Alinhamento com o padrão x86_64.

**Referências:**
- Intel SDM Volume 3A (paginação)
- Linux (x86_64 usa 4 níveis)

---

## Decisão 007: Escalonador — orientado a latência de cauda

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua política de escalonamento. A escolha é entre justiça (similar ao CFS), tempo real (prioridade fixa) ou orientada a latência de cauda.

**Alternativas consideradas:**
1. Justiça (similar ao CFS)
2. Tempo real (prioridade fixa)
3. Orientada a latência de cauda

**Decisão:**
Escalonador orientado a latência de cauda (p99, p999), não a justiça ou vazão média.

**Justificativa:**
- **Justiça (similar ao CFS)**: otimiza vazão média e justiça, mas não a latência de cauda.
- **Tempo real (prioridade fixa)**: garante prazos, mas não se adapta a cargas de trabalho interativas variáveis.
- **Orientada a latência de cauda**: prioriza previsibilidade; reduz p99 e p999; adapta-se a cargas variáveis.

**Consequências:**
- Prioridade para tarefas interativas.
- Isolamento de CPU para tarefas críticas.
- Preempção rápida.
- Benchmarks focados em p99/p999, jitter e previsibilidade.
- Trade-off: vazão média pode ser menor que o CFS em cargas de trabalho em lote.

**Referências:**
- CFS (Linux)
- LITMUS^RT (Linux determinístico)
- QNX (tempo real)

---

## Decisão 008: Drivers — em espaço de kernel

**Status:** Aceita

**Contexto:**
O TailOS precisa definir onde os drivers rodam. A escolha é entre espaço de kernel (monolítico) ou espaço de usuário (microkernel).

**Alternativas consideradas:**
1. Espaço de kernel (monolítico)
2. Espaço de usuário (microkernel)
3. Híbrido

**Decisão:**
Drivers em espaço de kernel.

**Justificativa:**
- **Espaço de kernel**: menor latência (sem IPC), mais simples, alinhado com arquitetura monolítica modular.
- **Espaço de usuário**: maior isolamento, mas overhead de IPC introduz latência imprevisível.
- **Híbrido**: complexidade desnecessária.

**Consequências:**
- Drivers têm acesso total ao hardware.
- Maior risco de falha do kernel devido a bug em driver.
- Menor latência em operações de I/O.
- Alinhamento com o objetivo de baixa latência.

**Referências:**
- Linux (drivers no kernel)
- QNX (drivers em espaço de usuário, trade-off diferente)

---

## Decisão 009: Licença — GPLv2

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua licença. A escolha é entre permissiva (MIT, BSD, Apache 2.0) e copyleft (GPLv2, GPLv3).

**Alternativas consideradas:**
1. MIT
2. Apache 2.0
3. GPLv2
4. GPLv3

**Decisão:**
GPLv2.

**Justificativa:**
- **MIT/Apache 2.0**: permissivas; permitem uso proprietário sem contribuição de volta; incompatíveis com a filosofia de um projeto aberto e comunitário.
- **GPLv2**: copyleft; garante que modificações permaneçam abertas; compatível com o ecossistema C; escolha do Linux.
- **GPLv3**: mais moderna, mas incompatível com GPLv2 e algumas bibliotecas.

**Consequências:**
- Código permanece aberto e comunitário.
- Contribuições de volta são obrigatórias.
- Incompatibilidade com código Apache 2.0 (avaliado caso a caso).
- Alinhamento com o Linux.

**Referências:**
- Linux (GPLv2)
- FreeBSD (BSD)
- Redox OS (MIT)

---

## Decisão 010: Modelo de desenvolvimento — incremental, benchmarks desde o início

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu modelo de desenvolvimento. A escolha é entre desenvolver tudo e fazer benchmarks no final, ou desenvolver incrementalmente com benchmarks desde o início.

**Alternativas consideradas:**
1. Desenvolver tudo, benchmark no final
2. Desenvolver incrementalmente, benchmark desde o início

**Decisão:**
Desenvolvimento incremental, com benchmarks desde o início.

**Justificativa:**
- **Benchmark no final**: risco de descobrir problemas de latência tarde demais.
- **Benchmark desde o início**: valida decisões de design continuamente; detecta regressões cedo; gera dados para análise.
- Alinhamento com a filosofia de latência como requisito.

**Consequências:**
- Benchmarks são parte do desenvolvimento, não um passo final.
- Cada módulo tem um benchmark associado.
- Dados de latência guiam decisões de design.
- Roadmap inclui benchmarks em cada fase.

**Referências:**
- LITMUS^RT (benchmarks de tempo real)
- Linux (benchmarks de escalonador)

---

## Decisão 011: Estrutura de código — modular por subsistema

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua estrutura de código. A escolha é entre um monólito de arquivos ou uma estrutura modular com interfaces claras.

**Alternativas consideradas:**
1. Monólito de arquivos
2. Estrutura modular com interfaces claras

**Decisão:**
Estrutura modular com interfaces claras, organizada por subsistema.

**Justificativa:**
- **Monólito**: simples no início, mas difícil de manter e evoluir.
- **Modular**: cada subsistema tem uma interface clara; facilita evolução incremental; alinhado com arquitetura monolítica modular.

**Consequências:**
- `boot/` — bootloader e linker script.
- `kernel/` — código do kernel.
- `kernel/src/memory/` — gerenciamento de memória.
- `kernel/src/sched/` — escalonador.
- `kernel/src/drivers/` — drivers.
- `userspace/` — código de espaço de usuário.
- `docs/` — documentação.
- `scripts/` — scripts de build e execução.

**Referências:**
- Linux (modular)
- SerenityOS (modular)

---

## Decisão 012: Sistema de build — Makefile

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu sistema de build. A escolha é entre Makefile, CMake, Ninja ou um sistema customizado.

**Alternativas consideradas:**
1. Makefile
2. CMake
3. Ninja
4. Sistema customizado

**Decisão:**
Makefile.

**Justificativa:**
- **Makefile**: simples, universal, padrão em projetos de kernel; controle total.
- **CMake**: complexo para um kernel; overhead desnecessário.
- **Ninja**: rápido, mas gerado por outro sistema.
- **Customizado**: escopo desnecessário.

**Consequências:**
- Makefile define alvos: `build`, `run`, `debug`, `clean`.
- Integração com GCC, NASM, LD e QEMU.
- Controle total sobre flags de compilação.

**Referências:**
- Linux (Kbuild)
- xv6 (Makefile)
- SerenityOS (Makefile + CMake)

---

## Decisão 013: Emulador e debug — QEMU + GDB

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu ambiente de teste e depuração. A escolha é entre hardware real, QEMU, Bochs, VirtualBox e GDB.

**Alternativas consideradas:**
1. Hardware real
2. QEMU + GDB
3. Bochs
4. VirtualBox

**Decisão:**
QEMU para emulação, GDB para depuração.

**Justificativa:**
- **Hardware real**: arriscado, difícil de depurar, lento para iterar.
- **QEMU**: emulador maduro, suporte a x86_64, depuração com GDB, rápido, open source.
- **GDB**: inspeção completa de registradores, memória, pilha; breakpoints; passo a passo.
- **Bochs**: bom para depuração, mas lento.
- **VirtualBox**: focado em virtualização, não em desenvolvimento de kernel.

**Consequências:**
- Desenvolvimento e teste no QEMU.
- Depuração com GDB conectado ao QEMU.
- Testes em hardware real apenas em marcos.
- Iteração rápida.

**Referências:**
- OSDev Wiki (QEMU + GDB)
- SerenityOS (QEMU + GDB)

---

## Decisão 014: Controle de versão e hospedagem — Git + GitHub

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu controle de versão e hospedagem. A escolha é entre Git, Mercurial, SVN, e GitHub, GitLab, Codeberg, self-hosted.

**Alternativas consideradas:**
1. Git + GitHub
2. Git + GitLab
3. Git + Codeberg
4. Mercurial / SVN
5. Self-hosted

**Decisão:**
Git para controle de versão, GitHub para hospedagem.

**Justificativa:**
- **Git**: padrão da indústria; ferramentas maduras.
- **GitHub**: maior comunidade; visibilidade; GitHub Actions.
- **GitLab**: bom, mas menos visibilidade para open source.
- **Codeberg**: ético, mas menos visibilidade.
- **Self-hosted**: complexidade desnecessária.

**Consequências:**
- Repositório em `https://github.com/beaverbit/tail-os`.
- Commits frequentes e descritivos.
- Branches para features.
- Integração futura de CI/CD.

**Referências:**
- Linux (Git + GitHub)
- SerenityOS (Git + GitHub)

---

## Decisão 015: Versionamento — semver

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu esquema de versionamento. A escolha é entre versionamento linear, semver ou baseado em data.

**Alternativas consideradas:**
1. Linear (v1, v2, v3)
2. Semver (MAJOR.MINOR.PATCH)
3. Baseado em data (YYYY.MM.DD)

**Decisão:**
Semver.

**Justificativa:**
- **Linear**: não comunica compatibilidade.
- **Semver**: comunica compatibilidade; padrão da indústria.
- **Baseado em data**: não comunica compatibilidade.

**Consequências:**
- Versões: MAJOR.MINOR.PATCH.
- MAJOR: mudanças incompatíveis.
- MINOR: novas funcionalidades compatíveis.
- PATCH: correções compatíveis.

**Referências:**
- Semver (https://semver.org)

---

## Decisão 016: Idioma — Inglês no código, Português na documentação interna

**Status:** Aceita

**Contexto:**
O TailOS precisa definir seu idioma. A escolha é entre Inglês, Português ou ambos.

**Alternativas consideradas:**
1. Inglês
2. Português
3. Ambos

**Decisão:**
Inglês no código e documentação pública; Português na documentação interna.

**Justificativa:**
- **Inglês no código**: padrão da indústria; facilita contribuidores internacionais.
- **Português na documentação interna**: facilita o desenvolvimento solo.
- **Ambos**: equilíbrio.

**Consequências:**
- Código, comentários e README em Inglês.
- DECISIONS.md em Inglês.
- Facilita contribuidores internacionais.
- Facilita o desenvolvimento solo.

**Referências:**
- Linux (Inglês)
- SerenityOS (Inglês)

---

## Decisão 017: Filosofia de código — simplicidade e clareza

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua filosofia de código. A escolha é entre otimização agressiva ou simplicidade e clareza.

**Alternativas consideradas:**
1. Otimização agressiva
2. Simplicidade e clareza
3. Ambos

**Decisão:**
Simplicidade e clareza, com otimização quando necessária e mensurável.

**Justificativa:**
- **Otimização agressiva**: risco de bugs; difícil de manter.
- **Simplicidade e clareza**: fácil de entender; fácil de manter; base para otimização futura.
- **Ambos**: equilíbrio.

**Consequências:**
- Código claro e legível.
- Otimização guiada por benchmarks, não por intuição.
- Comentários explicam o "porquê", não o "o quê".
- Facilita evolução e contribuidores.

**Referências:**
- Linux (simplicidade + otimização)
- SerenityOS (clareza)

---

## Decisão 018: Filosofia do projeto — latência como requisito

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua filosofia de projeto. A escolha é entre latência como requisito ou latência como consequência.

**Alternativas consideradas:**
1. Latência como requisito
2. Latência como consequência

**Decisão:**
Latência como requisito. Toda decisão de design é avaliada pelo seu impacto na latência de cauda.

**Justificativa:**
- **Latência como consequência**: abordagem de SO de propósito geral; latência é otimizada depois.
- **Latência como requisito**: abordagem do TailOS; latência guia todas as decisões desde o início.

**Consequências:**
- Toda funcionalidade é avaliada pelo seu impacto na latência.
- Benchmarks focados em p99/p999, jitter, previsibilidade.
- Trade-offs explícitos: vazão pode ser menor que SOs de propósito geral.
- Filosofia clara guia a evolução do projeto.

**Referências:**
- QNX (latência como requisito)
- LITMUS^RT (latência como requisito)
- Linux (latência como consequência, para contraste)

---

## Decisão 019: Filosofia de evolução — incremental

**Status:** Aceita

**Contexto:**
O TailOS precisa definir sua filosofia de evolução. A escolha é entre reescrever ou evoluir incrementalmente.

**Alternativas consideradas:**
1. Reescrever
2. Evolução incremental

**Decisão:**
Evolução incremental.

**Justificativa:**
- **Reescrever**: perda de conhecimento; risco de regressão.
- **Evolução incremental**: mantém o conhecimento; adiciona funcionalidades sem quebrar.

**Consequências:**
- Funcionalidades adicionadas incrementalmente.
- Benchmarks garantem que não há regressão.
- Base sólida antes de expandir.
- Facilita contribuidores.

**Referências:**
- Linux (evolução incremental)
- SerenityOS (evolução incremental)

---

## Decisão 020: Fases futuras — CI/CD e contribuidores

**Status:** Adiada

**Contexto:**
O TailOS precisa definir sua estratégia para CI/CD e contribuidores. A escolha é entre configurar agora ou adiar.

**Alternativas consideradas:**
1. Configurar agora
2. Adiar para fase futura

**Decisão:**
Adiar para fase futura.

**Justificativa:**
- **Agora**: overhead desnecessário no início; foco no código.
- **Futuro**: quando o projeto tiver contribuidores e estabilidade.

**Consequências:**
- Foco no código e benchmarks no início.
- CI/CD e contribuidores avaliados no futuro.
- GitHub Actions avaliado no futuro.

**Referências:**
- Linux (começou solo, CI/CD depois)
- SerenityOS (começou solo, CI/CD depois)

---

- **Data:** 2026-09-27
- **Fim do documento.**