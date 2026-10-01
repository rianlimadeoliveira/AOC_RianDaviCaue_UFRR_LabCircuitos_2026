# AOC — Laboratório de Circuitos (Versão Introdutória)

## 1. Identificação

- **Disciplina:** Arquitetura e Organização de Computadores
- **Curso / Instituição:** Ciência da Computação — UFRR
- **Semestre:** 2026.2
- **Professor:** Prof. Dr. Herbert Oliveira Rocha

| Integrante | Matrícula | Responsabilidades |
|------------|-----------|-------------------|
| Rian       | 2025015769 | Parte II — unidade de controle cabeada; README |
| Davi       | 2025015061 | Parte I — módulo de memória, cache, planilha; parte do relatório |
| Cauê       | 2025015670 | Parte II — unidade de controle microprogramada |

## 2. Ferramentas e como abrir os arquivos

- **Logisim-Evolution:** versão **3.8.0 ou superior**.
- Como abrir cada `.circ`: no Logisim-Evolution, **Arquivo → Abrir** e selecione o arquivo. O circuito a simular é escolhido no painel da esquerda:
  - `parte1/parte1_memoria.circ`: circuito `main` (módulo de memória). Subcircuitos `Decodificador` e `BancoDeRegistradores`.
  - `parte1/parte1_cache.circ`: circuito `main` (cache com o somador de endereço, usado no traço da Tabela 7 e nos testes T-03 a T-05) e circuito `ConflitoPrimos` (teste da Seção 5.6). Subcircuitos `cache`, `Arranjo_dados`, `Decodificador_7segmentos` e `DetectorPrimos`.
  - `parte2/cabeada/parte2_cabeada.circ`: circuito `main` (unidade de controle cabeada).
  - `parte2/microprogamada/Micropramada_vf.circ`: circuito `parte2_microprogramada` (unidade de controle microprogramada). O conteúdo da ROM de microcódigo está em `parte2/microprogamada/microcodigo.txt` e pode ser carregado na ROM com **Carregar imagem**.
- Para simular, use a ferramenta de interação (mão) para alterar os pinos de entrada e o pino de clock.

## 3. Arquivos entregues

| Arquivo | Descrição |
|---------|-----------|
| `relatorio/relatorio_tecnico.pdf` | Relatório técnico (versão de entrega) |
| `parte1/parte1_memoria.circ` | Módulo de memória de 64 bytes: decodificador de endereços, ROM, RAM, banco de registradores, E/S, MUX de saída e bit de paridade |
| `parte1/parte1_cache.circ` | Cache de mapeamento direto (4 linhas, blocos de 2 bytes), somador de endereço, arranjo de dados e teste de conflito com o detector de primos |
| `parte1/parte1_planilha.xlsx` | Planilha de acertos e faltas do traço, indicadores (taxa de acertos, AMAT) e teste da Seção 5.6 |
| `parte1/evidencias/` | Capturas de tela dos testes T-01 a T-05 e dos circuitos da Parte I |
| `parte2/cabeada/parte2_cabeada.circ` | Unidade de controle cabeada (5 estados, 3 flip-flops D, lógica minimizada) |
| `parte2/cabeada/tabela_tempo.xlsx` | Tabela de tempo da versão cabeada |
| `parte2/microprogamada/Micropramada_vf.circ` | Unidade de controle microprogramada (µPC de 3 bits, ROM de microcódigo, despacho pelo opcode) |
| `parte2/microprogamada/microcodigo.txt` | Conteúdo da ROM de microcódigo (`04 22 90 19 5d 00 00 00`) |
| `parte2/microprogamada/tabela_tempo (1).xlsx` | Tabela de tempo da versão microprogramada |
| `parte2/*.png`, `parte2/evidencias-microprogamada/` | Capturas de tela dos testes T-06 a T-09 e diagrama de estados |
| `componentes/` | Vazia: os componentes estão instanciados dentro dos arquivos `.circ` (ver Seção 4) |

## 4. Onde cada componente foi instanciado

| Nº | Componente | Arquivo | Subcircuito |
|----|------------|---------|-------------|
| 01 | Flip-flop D e flip-flop JK | D: `parte2/cabeada/parte2_cabeada.circ`; JK: `parte1/parte1_cache.circ` | D: `main` (FF2, FF1, FF0, registrador de estado); JK: `cache` (bits de validade `Valido_L0` a `Valido_L3`) |
| 02 | Multiplexador de 4 entradas | `parte1/parte1_memoria.circ`; `parte1/parte1_cache.circ`; `parte2/microprogamada/Micropramada_vf.circ` | `main` (seleção da saída de dados por A5A4); `cache` (rótulo e validade da linha indexada); `parte2_microprogramada` (próximo endereço do µPC) |
| 03 | XOR a partir de AND, NOT e OR | `parte1/parte1_cache.circ` | `cache` (comparador de rótulos de 3 bits) |
| 04 | Somador de 8 bits com constante 4 | Não implementado | O caminho de dados da Parte II não foi montado |
| 05 | Memória ROM de 8 bits | `parte1/parte1_memoria.circ`; `parte2/microprogamada/Micropramada_vf.circ` | `main` (ROM de programa, região 0x00–0x0F); `parte2_microprogramada` (ROM de microcódigo 8 × 8) |
| 06 | Memória RAM de 8 bits | `parte1/parte1_memoria.circ`; `parte1/parte1_cache.circ` | `main` (RAM de dados 0x10–0x1F e região de E/S); `Arranjo_dados` (`Dados_byte0` e `Dados_byte1`, arranjo de dados da cache) |
| 07 | Banco de registradores de 8 bits | `parte1/parte1_memoria.circ` | `BancoDeRegistradores` (região 0x20–0x2F) |
| 08 | Somador de 8 bits | `parte1/parte1_cache.circ` | `main` (cálculo do endereço apresentado à cache: Base + Deslocamento) |
| 09 | Detector da sequência "101" | Não incluído no repositório | — |
| 10 | ULA de 8 bits | Não implementado | O caminho de dados da Parte II não foi montado |
| 11 | Extensor de sinal de 4 para 8 bits | Não implementado | O caminho de dados da Parte II não foi montado |
| 12 | Máquina de estados com portas lógicas | `parte2/cabeada/parte2_cabeada.circ` | `main` (unidade de controle cabeada) |
| 13 | Contador síncrono | `parte1/parte1_cache.circ`; `parte2/microprogamada/Micropramada_vf.circ` | `cache` (contador de acertos) e `ConflitoPrimos` (contador de 3 bits do teste); `parte2_microprogramada` (µPC: registrador de 3 bits com somador +1) |
| 14 | Detector de paridade ímpar | `parte1/parte1_memoria.circ` | `main` (bit de paridade gerado na escrita e verificado na leitura da RAM) |
| 15 | Otimização por mapas de Karnaugh | `parte2/cabeada/parte2_cabeada.circ`; `parte1/parte1_cache.circ` | `main` (equações de próximo estado e dos sinais de controle); `DetectorPrimos` (equação do detector) |
| 16 | Decodificador de 7 segmentos | `parte1/parte1_cache.circ` | `Decodificador_7segmentos` (display do contador de acertos, dentro de `cache`) |
| 17 | Detector de número primo (4 bits) | `parte1/parte1_cache.circ` | `DetectorPrimos` (usado em `ConflitoPrimos`) |

## 5. Declaração de uso de IA generativa

- **Davi:** Uso do Claude Code para orientação, revisão dos circuitos e criação do modelo base para planilha e relatório.
- **Rian:** Uso do Claude Code para orientação, revisão dos circuitos e desenho do diagrama de estados.
- **Cauê:** Foi utilizada a ferramenta de IA generativa Claude (Anthropic) como apoio na conferência do microcódigo e na orientação sobre o uso do Logisim-Evolution durante os testes; os circuitos, as simulações e as capturas de tela foram feitos pela equipe.

## 6. Divisão do trabalho

| Integrante | O que fez |
|------------|-----------|
| **Davi** | Parte I: módulo de memória (`parte1_memoria.circ`), cache (`parte1_cache.circ`, incluindo arranjo de dados e teste de conflito com o detector de primos), planilha (`parte1_planilha.xlsx`), evidências da Parte I e parte do relatório |
| **Rian** | Parte II: unidade de controle cabeada (`parte2_cabeada.circ`), diagrama de estados e tabela de tempo da versão cabeada; README |
| **Cauê** | Parte II: unidade de controle microprogramada (`Micropramada_vf.circ`), microcódigo, tabela de tempo e evidências da versão microprogramada |
