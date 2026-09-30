# Relatório Técnico — AOC Laboratório de Circuitos

<!-- Rascunho-fonte. Ao finalizar, exportar para relatorio_tecnico.pdf (nome exigido pelo enunciado). -->

## 1. Introdução

## 2. Parte I — Subsistema de memória

### 2.1 Módulo de memória (mapa de memória e tabela verdade dos CS)

### 2.2 Bit de paridade

### 2.3 Cache (divisão do endereço e comparador de rótulos)

### 2.4 Planilha e indicadores

### 2.5 Teste de conflito (detector de primos)

### 2.6 Testes (T-01 a T-05)

## 3. Parte II — Unidade de controle

### 3.1 Unidade de controle cabeada

A unidade de controle foi implementada como uma máquina de estados finitos de cinco estados (S0 a S4), codificados em três flip-flops D (Q2, Q1, Q0). O reset é assíncrono e leva a máquina ao estado S0. Como o caminho de dados não foi montado, o decodificador de opcode foi abstraído em uma única entrada, chamada BEQ: vale 1 quando a instrução é beq e 0 quando é do tipo R.

**Tabela 1 — Estados da unidade de controle.**

| Estado | Código (Q2Q1Q0) | Nome | O que acontece |
|---|---|---|---|
| S0 | 000 | Busca | IR ← Mem[PC]; PC ← PC + 4 |
| S1 | 001 | Decodificação | A ← Reg[rs]; B ← Reg[rt]; ULAOut ← PC + (ext(imm) << 2) |
| S2 | 010 | Execução tipo-R | ULAOut ← A op B |
| S3 | 011 | Conclusão tipo-R | Reg[rd] ← ULAOut |
| S4 | 100 | Conclusão de desvio | A − B; se Zero, PC ← ULAOut |

As transições entre os estados são as seguintes: S0 sempre vai para S1; em S1, o opcode decide o caminho, e uma instrução tipo-R (BEQ = 0) vai para S2, enquanto uma instrução beq (BEQ = 1) vai para S4; S2 vai para S3; S3 e S4 voltam para S0. O diagrama completo está na Figura 1 e a tabela de transição, com os valores que entram nos flip-flops D, está na Tabela 2.

**Tabela 2 — Tabela de transição.**

| Estado atual (Q2Q1Q0) | BEQ | Estado seguinte | D2 D1 D0 |
|---|---|---|---|
| 000 (S0) | X | 001 (S1) | 0 0 1 |
| 001 (S1) | 0 | 010 (S2) | 0 1 0 |
| 001 (S1) | 1 | 100 (S4) | 1 0 0 |
| 010 (S2) | X | 011 (S3) | 0 1 1 |
| 011 (S3) | X | 000 (S0) | 0 0 0 |
| 100 (S4) | X | 000 (S0) | 0 0 0 |
| 101, 110, 111 | X | indiferente | X X X |

**Figura 1 — Diagrama de estados da unidade de controle.**

![Diagrama de estados da unidade de controle](../parte2/evidencias/diagrama_estados.png)


#### Estados não utilizados

Com cinco estados e três flip-flops, sobram três combinações que a máquina não usa: 101, 110 e 111. Optou-se por tratá-las como condições indiferentes (X) nos mapas de Karnaugh, o que permite agrupamentos maiores e, portanto, equações com menos portas. A alternativa seria forçar o retorno ao estado de busca, o que exigiria portas extras para detectar esses três códigos.

Essa escolha só é segura se a máquina, ao cair acidentalmente em um desses códigos (por exemplo, por ruído elétrico), voltar sozinha a um estado válido. Isso é verificado ao final desta seção, com as equações finais em mãos. Na partida, o reset assíncrono já garante que a máquina comece em S0.

### 3.2 Tabela de tempo e CPI

### 3.3 Microprograma

### 3.4 Comparação entre as duas versões

### 3.5 Testes (T-06 a T-09)

## 4. Declaração de uso de IA generativa

## 5. Conclusão

## Referências
