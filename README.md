# Trabalho Prático 2 — Gerenciador de Memória Virtual

Este é o projeto-base para implementação de um simulador de gerência de memória virtual.

## Estrutura

```text
vm_manager/
├── Makefile
├── README.md
├── include/
│   ├── config.h
│   ├── tlb.h
│   ├── page_table.h
│   ├── memory.h
│   └── statistics.h
├── src/
│   ├── main.c
│   ├── tlb.c
│   ├── page_table.c
│   ├── memory.c
│   └── statistics.c
├── data/
│   └── generate_data.py
└── report/
```

## Gerar arquivos comuns de entrada

Entre no diretório `data` e execute:

```bash
python3 generate_data.py
```

Isso criará:

- `BACKING_STORE.bin`
- `addresses_random.txt`
- `addresses_location.txt`

## Compilação

Na raiz do projeto:

```bash
make
```

## Execução

```bash
./vm < data/addresses_random.txt
```

ou

```bash
./vm < data/addresses_location.txt
```

## Observação

As funções principais estão propositalmente incompletas e marcadas com `TODO`.
Vocês devem implementar a lógica de:

- Tradução de endereços;
- Tabela de páginas;
- TLB com FIFO;
- Tratamento de page fault;
- Substituição de páginas com LRU aproximado;
- Estatísticas.


## Resultados Experimentais

Foram realizados testes com dois conjuntos de endereços lógicos: um padrão com localidade de referência (`addresses_location.txt`) e outro com endereços aleatórios (`addresses_random.txt`).

### Resultado com localidade de referência

```text
Number of Translated Addresses = 10000
Page Faults = 1164
Page Fault Rate = 0.116
TLB Hits = 7925
TLB Hit Rate = 0.792
```

### Resultado com endereços aleatórios

```text
Number of Translated Addresses = 10000
Page Faults = 4958
Page Fault Rate = 0.496
TLB Hits = 654
TLB Hit Rate = 0.065
```

### Análise

Os resultados demonstram o impacto da localidade de referência no desempenho do sistema de memória virtual.

No conjunto com localidade, os acessos permanecem concentrados em regiões próximas da memória, permitindo maior reutilização das entradas da TLB e das páginas já carregadas na memória física. Como consequência, foram observadas menores taxas de Page Fault e maior taxa de TLB Hit.

Já no conjunto aleatório, os acessos são distribuídos por diferentes regiões da memória, reduzindo a reutilização de páginas e entradas da TLB. Isso aumentou a taxa de Page Fault e reduziu significativamente a eficiência da TLB.

Os resultados obtidos estão de acordo com o comportamento esperado para sistemas de memória virtual.

