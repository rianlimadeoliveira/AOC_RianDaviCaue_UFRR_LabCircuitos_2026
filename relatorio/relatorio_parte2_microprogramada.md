<!-- Seções 3.2, 3.3 e 3.5 (parte microprogramada). Numeração continua a da Parte I e da seção 3.1 (última tabela: 15; última figura: 19). Imagens em ../parte2/evidencias/. -->

### 3.2 Tabela de tempo e CPI

A Tabela 16 mostra o valor de cada um dos nove sinais de controle (onze bits) em cada estado. Como cada sinal depende só do estado, ela também é a tabela verdade dos sinais, com cinco linhas úteis. O arquivo `tabela_tempo.xlsx` traz a mesma tabela, o cálculo do CPI e o microprograma.

**Tabela 16 — Tabela de tempo.**

| Estado | PCWr | PCWrC | PCSrc | MemRd | IRWr | RegWr | SrcA | SrcB | ALUOp |
|---|---|---|---|---|---|---|---|---|---|
| S0 (Busca) | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 01 | 00 |
| S1 (Decod.) | 0 | 0 | X | 0 | 0 | 0 | 0 | 10 | 00 |
| S2 (Exec. tipo-R) | 0 | 0 | X | 0 | 0 | 0 | 1 | 00 | 10 |
| S3 (Concl. tipo-R) | 0 | 0 | X | 0 | 0 | 1 | X | XX | XX |
| S4 (Concl. desvio) | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 00 | 01 |

Cada X tem uma justificativa (Tabela 17). Nos circuitos, os X foram fixados em valores concretos: PCSource = 0 nos estados sem escrita no PC e, em S3, SrcA = 1, SrcB = 00 e ALUOp = 00 na versão microprogramada.

**Tabela 17 — Justificativa das condições indiferentes.**

| Estado | Sinal | Por que é indiferente |
|---|---|---|
| S1, S2, S3 | PCSource | O PC não é escrito (PCWrite = 0 e PCWriteCond = 0), então a origem do valor é irrelevante. |
| S3 | SrcA, SrcB, ALUOp | Reg[rd] recebe o ULAOut já gravado em S2, e a saída da ULA não é usada nesse estado. |

**Ciclos e CPI.** Uma instrução tipo-R percorre S0, S1, S2 e S3, e consome 4 ciclos. Uma instrução beq percorre S0, S1 e S4, e consome 3 ciclos (Tabela 18).

**Tabela 18 — Ciclos e CPI para a mistura de instruções.**

| Classe | Frequência | Estados percorridos | Ciclos | Contribuição |
|---|---|---|---|---|
| Tipo-R (add, sub, and, or) | 70% | S0, S1, S2, S3 | 4 | 0,70 × 4 = 2,8 |
| beq | 30% | S0, S1, S4 | 3 | 0,30 × 3 = 0,9 |
| **CPI médio** | | | | **3,7** |

### 3.3 Microprograma

A unidade microprogramada guarda os sinais em uma ROM de 8 posições por 8 bits (Componente 05). O sequenciamento é feito por um microcontador de programa (µPC) de 3 bits (Componentes 01 e 13), que é um registrador de três flip-flops D com um somador +1. A entrada OP_BEQ desempenha o papel do BEQ da versão cabeada.

**Formato da microinstrução.** A microinstrução tem 8 bits, divididos em quatro campos de 2 bits (Tabela 19). O campo Fonte = 00 implica PCWrite = 1.

**Tabela 19 — Campos da microinstrução.**

| Campo | Bits | Codificação |
|---|---|---|
| ULAOp | 7–6 | 00 = soma; 01 = subtração; 10 = definida pelo funct; 11 = reservado |
| Fonte | 5–4 | 00 = PC e constante 4; 01 = A e B; 10 = PC e extensão de sinal deslocada de 2; 11 = reservado |
| Ação | 3–2 | 00 = nenhuma; 01 = IR ← Mem[PC]; 10 = Reg[rd] ← ULAOut; 11 = PC ← ULAOut se Zero |
| Seq | 1–0 | 00 = próxima; 01 = volta à busca; 10 = despacho pelo opcode; 11 = reservado |

**Tabela 20 — Microprograma completo.**

| Rótulo | Endereço | ULAOp | Fonte | Ação | Seq | Binário | Hexadecimal |
|---|---|---|---|---|---|---|---|
| Busca | 0 | 00 | 00 | 01 | 00 | 00000100 | 0x04 |
| Decod | 1 | 00 | 10 | 00 | 10 | 00100010 | 0x22 |
| ExecR | 2 | 10 | 01 | 00 | 00 | 10010000 | 0x90 |
| ConclR | 3 | 00 | 01 | 10 | 01 | 00011001 | 0x19 |
| ConclBEQ | 4 | 01 | 01 | 11 | 01 | 01011101 | 0x5D |

As posições 5, 6 e 7 ficam livres (00). O conteúdo carregado na ROM, em `microcodigo.txt`, é `04 22 90 19 5d 00 00 00`. Em ConclR, ULAOp = 00 é indiferente, porque a ULA não é usada nesse estado. Os estados 101, 110 e 111 nunca são alcançados: o µPC só recebe µPC + 1 (dentro de 0 a 4), 0, 2 ou 4.

**Sequenciamento.** O multiplexador de próximo endereço (Componente 02) é comandado pelo campo Seq: 00 → µPC + 1; 01 → 0 (volta à busca); 10 → despacho, que escolhe 2 (OP_BEQ = 0) ou 4 (OP_BEQ = 1); 11 (reservado) → 0. A saída do multiplexador é o PROX, que entra no registrador do µPC. A ROM ocupa 8 × 8 = 64 bits.

**Circuito.** O circuito está em `parte2_microprogramada.circ` (Figura 20). Os pinos estão na Tabela 21.

**Figura 20 — Circuito principal da unidade microprogramada.**

![Circuito principal da unidade microprogramada](../parte2/evidencias/circuito_principal_microprogramada.png)

**Tabela 21 — Elementos e pinos da unidade microprogramada.**

| Elemento | Componente | Pinos e função |
|---|---|---|
| upc | Registrador de 3 flip-flops D (Comp. 01), atuando como µPC (Comp. 13) | D (3 bits) recebe PROX; Q (3 bits) é o µPC; CLK; WE fixo em 1; R ligado a RESET |
| Somador +1 | Somador de 3 bits | Entradas: µPC e constante 1; saída µPC + 1 |
| ROM 8×8 | ROM (Comp. 05) | Endereço A (3 bits) = µPC; saída de 8 bits dividida em UO1UO0 (ULAOp), FT1FT0 (Fonte), AC1AC0 (Ação) e SQ1SQ0 (Seq) |
| MUX de próximo endereço | Multiplexador de 4 entradas (Comp. 02) | Seleção = SQ1SQ0; saída = PROX |
| MUX de despacho | Multiplexador de 2 entradas | Seleção = OP_BEQ; 0 → constante 2; 1 → constante 4 |
| Display | Decodificador de 7 segmentos (Comp. 16) | Mostra o µPC; o 4º bit vai a uma constante 0 (GND) |
| CLK, RESET, OP_BEQ, ZERO | Pinos de entrada, 1 bit | Relógio; zera o µPC; 0 = tipo-R e 1 = beq; simula o Zero da ULA |
| Saídas de controle | Pinos de saída | PCWrite, PCWriteCond, PCSource, MemRead, IRWrite, RegWrite, ALUSrcA, ALUSrcB1, ALUSrcB0, ALUOp1, ALUOp0, uPC_2..uPC_0 |

Os campos da ROM são decodificados nos sinais de controle por portas AND, OR e NOT (Figura 21 e Tabela 22).

**Figura 21 — Esquema de portas: decodificação dos campos da microinstrução.**

![Decodificação dos campos](../parte2/evidencias/circuito_logica_decodificacao.png)

**Tabela 22 — Equações de decodificação.**

| Sinal | Equação | Campo |
|---|---|---|
| PCWrite e ALUSrcB0 | FT1' · FT0' | Fonte = 00 |
| ALUSrcA | FT1' · FT0 | Fonte = 01 |
| ALUSrcB1 | FT1 | Fonte = 10 |
| MemRead e IRWrite | AC1' · AC0 | Ação = 01 |
| RegWrite | AC1 · AC0' | Ação = 10 |
| PCWriteCond e PCSource | AC1 · AC0 | Ação = 11 |
| ALUOp1, ALUOp0 | UO1, UO0 | ULAOp (direto) |

### 3.5 Testes (T-06 a T-09)

Método: simulação habilitada, pulso automático desligado, um ciclo completo de relógio por estado e pinos RESET, OP_BEQ e ZERO acionados com a ferramenta de interação. O display de 7 segmentos mostra o estado atual. As Tabelas 23 a 26 registram as entradas, as saídas esperadas e as observadas em cada teste, e as capturas estão nas Figuras 22 a 32.

**Tabela 23 — T-06, ciclo de busca (`parte2_microprogramada.circ`).**

| Momento | Entradas | Esperado | Observado | Figura |
|---|---|---|---|---|
| Após RESET | RESET = 1, OP_BEQ = 0 | S0; PCWrite, MemRead, IRWrite, ALUSrcB0 = 1 | display 0; esses quatro sinais em 1 | 22 |
| 1 ciclo depois | RESET = 0 | S1; IRWrite volta a 0 | display 1; só ALUSrcB1 em 1 | 23 |

**Figura 22 — T-06, estado S0: sequenciador e saídas.**

![T-06 S0 sequenciador](../parte2/evidencias/T06_S0_sequenciador.png)

![T-06 S0 saídas](../parte2/evidencias/T06_S0_saidas.png)

**Figura 23 — T-06, estado S1: sequenciador e saídas.**

![T-06 S1 sequenciador](../parte2/evidencias/T06_S1_sequenciador.png)

![T-06 S1 saídas](../parte2/evidencias/T06_S1_saidas.png)

**Tabela 24 — T-07, instrução tipo-R (OP_BEQ = 0).**

| Estado | Display | Sinais em 1 (esperado e observado) | Figura |
|---|---|---|---|
| S0 | 0 | PCWrite, MemRead, IRWrite, ALUSrcB0 | 24 |
| S1 | 1 | ALUSrcB1 | 24 |
| S2 | 2 | ALUSrcA, ALUOp1 | 24 |
| S3 | 3 | RegWrite, ALUSrcA | 25 |
| S0 (volta) | 0 | PCWrite, MemRead, IRWrite, ALUSrcB0 | 25 |

A instrução consome quatro ciclos e a máquina volta a S0. Como o caminho de dados não foi montado, o "rd recebe o resultado" é demonstrado por RegWrite = 1 em S3.

**Figura 24 — T-07, estados S0, S1 e S2.**

![T-07 S0](../parte2/evidencias/T07_S0.png)

![T-07 S1](../parte2/evidencias/T07_S1.png)

![T-07 S2](../parte2/evidencias/T07_S2.png)

**Figura 25 — T-07, estado S3 e retorno a S0.**

![T-07 S3](../parte2/evidencias/T07_S3.png)

![T-07 volta](../parte2/evidencias/T07_volta.png)

**Tabela 25 — T-08, desvio tomado e não tomado.**

| Caso | Entradas | Esperado | Observado | Figura |
|---|---|---|---|---|
| Tomado | OP_BEQ = 1, ZERO = 1 | S4; PCWriteCond, PCSource, ALUSrcA, ALUOp0 = 1; PC ← ULAOut | display 4; esses sinais em 1 | 27 |
| Não tomado | OP_BEQ = 1, ZERO = 0 | S4; mesmos sinais em 1; PC não muda | display 4; mesmos sinais em 1 | 27 |

Os sinais de controle dependem só do estado, e por isso são iguais nos dois casos. O PC só é gravado quando PCWriteCond · ZERO = 1. O ZERO foi simulado por um pino de entrada, pois o caminho de dados não foi montado. A Figura 26 mostra o caminho S0 → S1 → S4 → S0.

**Figura 26 — T-08, estado S4 e retorno a S0.**

![T-08 S4](../parte2/evidencias/T08_S4.png)

![T-08 volta](../parte2/evidencias/T08_volta.png)

**Figura 27 — T-08, estado S4 com ZERO = 1 (tomado) e ZERO = 0 (não tomado).**

![T-08 tomado](../parte2/evidencias/T08_tomado.png)

![T-08 não tomado](../parte2/evidencias/T08_nao_tomado.png)

**Tabela 26 — T-09, sinais de controle por estado nas duas versões (OP_BEQ = BEQ = 0 de S0 a S3 e 1 em S4).**

| Estado | Versão | PCW | PCWC | PCSrc | MemRd | IRW | RegW | SrcA | SrcB1 | SrcB0 | ALUOp1 | ALUOp0 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| S0 | Micro | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 |
|  | Cabeada | 1 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 |
| S1 | Micro | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
|  | Cabeada | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 |
| S2 | Micro | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 0 |
|  | Cabeada | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 0 |
| S3 | Micro | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 0 |
|  | Cabeada | 0 | 0 | 0 | 0 | 0 | 1 | 1 | 1 | 0 | 1 | 0 |
| S4 | Micro | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 1 |
|  | Cabeada | 0 | 1 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 1 |

Nos estados S0, S1, S2 e S4, as duas versões geram exatamente os mesmos onze sinais. Em S3 coincidem todos os sinais definidos (RegWrite = 1 e ALUSrcA = 1). A cabeada mostra também ALUSrcB1 = 1 e ALUOp1 = 1, e a microprogramada mostra 0 nesses dois sinais, porque SrcB e ALUOp são indiferentes em S3 (Tabela 17). A sequência de estados é a mesma nas duas versões: S0 → S1 → S2 → S3 → S0 (tipo-R) e S0 → S1 → S4 → S0 (beq). As capturas da microprogramada são as Figuras 24 a 26, e as da cabeada são as Figuras 28 a 32.

**Figura 28 — T-09, cabeada, estado S0.**

![T-09 cabeada S0](../parte2/evidencias/T09_cab_S0.png)

**Figura 29 — T-09, cabeada, estado S1.**

![T-09 cabeada S1](../parte2/evidencias/T09_cab_S1.png)

**Figura 30 — T-09, cabeada, estado S2.**

![T-09 cabeada S2](../parte2/evidencias/T09_cab_S2.png)

**Figura 31 — T-09, cabeada, estado S3 (ALUSrcB1 e ALUOp1 em 1, sinais indiferentes).**

![T-09 cabeada S3](../parte2/evidencias/T09_cab_S3.png)

**Figura 32 — T-09, cabeada, estado S4 (BEQ = 1).**

![T-09 cabeada S4](../parte2/evidencias/T09_cab_S4.png)

