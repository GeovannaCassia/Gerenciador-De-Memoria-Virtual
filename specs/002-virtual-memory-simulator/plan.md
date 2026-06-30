# Implementation Plan: Virtual Memory Simulator

**Branch**: `002-virtual-memory-simulator` | **Date**: 2026-06-29 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/002-virtual-memory-simulator/spec.md`

## Summary

Implementar o simulador de gerenciamento de memória virtual em C99, preservando a estrutura existente do projeto. O simulador deve traduzir endereços lógicos para endereços físicos usando uma tabela de páginas de 256 entradas, um TLB de 16 entradas com substituição FIFO, memória física de 128 quadros e o algoritmo Aging para substituição de páginas por demanda.

## Technical Context

**Language/Version**: C99

**Primary Dependencies**: Nenhuma dependência externa além da biblioteca padrão C

**Storage**: `BACKING_STORE.bin` com 256 páginas de 256 bytes cada

**Testing**: Compilação com Make, execução de traces de endereços e comparação com saída esperada

**Target Platform**: Windows (MSYS2/MinGW + Make) e Linux

**Project Type**: Simulador CLI de memória virtual

**Performance Goals**: Comportamento correto, determinístico e compatível com a estrutura existente

**Constraints**: Não alterar interfaces públicas existentes; não modificar a estrutura de pastas; implementar apenas os TODOs quando possível

**Scale/Scope**: Traduzir 32-bit logical addresses usando apenas os 16 bits menos significativos; suportar 256 páginas lógicas e 128 quadros físicos

## Constitution Check

- A implementação está alinhada com a preservação da arquitetura do projeto-base.
- Não serão introduzidas dependências externas.
- As mudanças ficarão limitadas às áreas de TODO existentes sempre que possível.
- A modificação é compatível com Windows e Linux.

## Project Structure

### Documentation (this feature)

```text
specs/002-virtual-memory-simulator/
├── plan.md
├── spec.md
└── checklists/
    └── requirements.md
```

### Source Code (repository root)

```text
src/
├── main.c
├── memory.c
├── page_table.c
├── statistics.c
├── tlb.c

include/
├── config.h
├── memory.h
├── page_table.h
├── statistics.h
├── tlb.h
```

**Structure Decision**: Implementar nas unidades de código existentes, sem criar novas pastas. A lógica de tradução, página e TLB deve permanecer dentro dos módulos já previstos.

## Implementation Approach

A implementação será feita em fases alinhadas com a ordem desejada da especificação, garantindo que cada etapa dependa apenas das anteriores.

### Fase 1 — Tradução de endereços

- Usar apenas os 16 bits menos significativos do endereço lógico de 32 bits.
- Extrair o número da página como os bits 8–15 e o offset como os bits 0–7.
- Implementar essa lógica com operações bitwise em C.
- Validar a extração sem executar lógica de página ou TLB.

### Fase 2 — Leitura do BACKING_STORE

- Implementar leitura aleatória de `BACKING_STORE.bin` usando `fopen`, `fread`, `fseek` e `fclose`.
- Garantir acesso a páginas de 256 bytes por meio de offsets de arquivo calculados como `page_number * 256`.
- Criar função de carregamento de uma página para memória física.

### Fase 3 — Tabela de páginas

- Implementar a estrutura de tabela de páginas com 256 entradas.
- Cada entrada deve conter um número de quadro e um bit de validade.
- Incluir operações de consulta, atualização e invalidação.

### Fase 4 — Memória física

- Implementar o gerenciador de memória física com 128 quadros de 256 bytes.
- Controlar quadros livres e ocupados.
- Implementar carregamento de páginas em quadros livres.

### Fase 5 — Tratamento de page fault

- Detectar falta de página quando a entrada de tabela de páginas não for válida.
- Ao encontrar um quadro livre, carregar a página do backing store para memória física.
- Atualizar a tabela de páginas e o TLB.

### Fase 6 — Política Aging (LRU aproximado)

- Atribuir um bit de referência e um contador de envelhecimento de 8 bits para cada página residente.
- Em cada acesso, definir `reference_bit = 1`.
- Em cada atualização de envelhecimento, fazer `counter >>= 1`, inserir `reference_bit` no bit mais significativo, e zerar `reference_bit`.
- Ao substituir, selecionar a página com menor contador e invalidar sua entrada de tabela e TLB.

### Fase 7 — TLB FIFO

- Implementar TLB com 16 entradas contendo número da página, número do quadro e bit válido.
- Implementar busca, inserção e invalidação.
- Usar política FIFO para substituir entradas quando o TLB estiver cheio.

### Fase 8 — Estatísticas

- Contar total de referências, page faults e TLB hits durante execução.
- Calcular taxa de page fault e taxa de TLB hit ao final.
- Preservar o formato de saída existente.

### Fase 9 — Integração geral

- Integrar a extração de endereço, tabela de páginas, TLB, memória física e page faults em um fluxo único.
- Assegurar tradução final de `physical_address = frame * 256 + offset`.
- Retornar endereço físico e byte acessado.

### Fase 10 — Testes finais

- Executar com os arquivos de teste fornecidos.
- Verificar a saída e as estatísticas esperadas.
- Confirmar compatibilidade Windows/Linux através do Makefile existente.

## Files to Modify

- `src/main.c`
- `src/memory.c`
- `src/page_table.c`
- `src/tlb.c`
- `src/statistics.c`
- `include/memory.h`
- `include/page_table.h`
- `include/tlb.h`
- `include/statistics.h`

## Task Breakdown and Dependencies

1. Tradução de endereços
2. Leitura do BACKING_STORE
3. Implementação da tabela de páginas
4. Implementação da memória física
5. Tratamento de page fault
6. Implementação do Aging
7. Implementação do TLB FIFO
8. Cálculo de estatísticas
9. Integração do fluxo completo
10. Testes finais

**Dependências entre tarefas**:

- Tarefa 2 depende da conclusão da tarefa 1 para saber a página solicitada.
- Tarefa 3 depende da tarefa 1 e deve ser concluída antes da tarefa 5.
- Tarefa 4 depende da tarefa 2 para saber como carregar páginas.
- Tarefa 5 depende das tarefas 3 e 4.
- Tarefa 6 depende da tarefa 5 e de dados de residentes de quadro.
- Tarefa 7 depende da tarefa 1 e deve ser integrada antes da tarefa 9.
- Tarefa 8 depende das tarefas 5, 6 e 7 para contabilizar corretamente os eventos.
- Tarefa 9 depende da conclusão de todas fases anteriores.

## Validation Plan

- Executar `make` e garantir compilação limpa.
- Testar a extração de página/offset com endereços de borda (`0x0000`, `0xFFFF`, `0x1234`).
- Validar a leitura do backing store carregando páginas específicas.
- Confirmar o comportamento da tabela de páginas e do TLB com trace tests.
- Verificar a substituição Aging usando um trace que preenche todos os quadros.
- Comparar a saída final de endereço físico, byte retornado e estatísticas com os resultados esperados.

## Risks and Mitigations

- Risco: mudanças na saída existente podem quebrar comparações com testes.
  - Mitigação: não alterar formato de saída e validar com arquivos de teste existentes.
- Risco: uso incorreto do algoritmo Aging pode comprometer substituição.
  - Mitigação: isolar a lógica de envelhecimento em um módulo dedicado e testar com casos de acesso controlados.
- Risco: incompatibilidade Windows/Linux.
  - Mitigação: usar apenas APIs padrão C99 e validar via compilação nos dois ambientes, se possível.
