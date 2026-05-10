# Esercitazioni — variante "no repo"

Versione alternativa dei lab pensata per chi **non vuole / non può clonare i repo demo** (`05-Repo-Demo/`). Ogni lab parte da una **cartella vuota** e usa Copilot stesso per generare il codice di partenza.

| # | File | Equivalente "con repo" | Durata |
|---|------|------------------------|--------|
| 1 | [Lab1-Agent-Plan-Mode.md](Lab1-Agent-Plan-Mode.md) | [Lab 1](../Lab1-Agent-Plan-Mode.md) | 25' |
| 2 | [Lab2-Custom-Instructions-Prompts-Skills.md](Lab2-Custom-Instructions-Prompts-Skills.md) | [Lab 2](../Lab2-Custom-Instructions-Prompts-Skills.md) | 20' |
| 3 | [Lab3-Copilot-CLI.md](Lab3-Copilot-CLI.md) | [Lab 3](../Lab3-Copilot-CLI.md) | 20' |
| 4 | [Lab4-VHDL-Testbench.md](Lab4-VHDL-Testbench.md) | [Lab 4](../Lab4-VHDL-Testbench.md) | 30' |
| 5 | [Lab5-Verilog-FSM.md](Lab5-Verilog-FSM.md) | [Lab 5](../Lab5-Verilog-FSM.md) | 30' |
| 6 | [Lab6-Finale-HDL.md](Lab6-Finale-HDL.md) | [Lab 6](../Lab6-Finale-HDL.md) | 30' |

### Variante general purpose dei lab pomeridiani (Python / Java / .NET)

Per chi non lavora su HDL o vuole fare i lab 4-6 su un linguaggio applicativo:

| # | File | Equivalente HDL | Tema |
|---|------|-----------------|------|
| 4 | [Lab4-General-Purpose.md](Lab4-General-Purpose.md) | Lab 4 (FIFO VHDL) | RingBuffer thread-safe + bug intenzionale (off-by-one vuoto/pieno) |
| 5 | [Lab5-General-Purpose.md](Lab5-General-Purpose.md) | Lab 5 (FSM Verilog)  | TrafficLightController + 2 bug (mutable shared state, switch non esaustivo) → refactoring |
| 6 | [Lab6-General-Purpose.md](Lab6-General-Purpose.md) | Lab 6 (mini IP HDL) | Mini progetto a scelta: rate limiter / LRU cache / log parser |

## Setup minimo

```bash
# 1. VS Code + estensione GitHub Copilot autenticata
# 2. Copilot CLI:  npm install -g @github/copilot
# 3. Una cartella vuota a tua scelta, es:
mkdir copilot-lab && cd copilot-lab && code .
```

Tool toolchain (solo se vuoi *eseguire* davvero il codice generato — la maggior parte degli obiettivi didattici è raggiungibile anche senza):

- **Lab 1/3 general purpose**: Python 3.11+, oppure Java 21, oppure CMake/g++, oppure .NET 10
- **Lab 4/5/6 HDL**: GHDL (VHDL) o Icarus Verilog (opzionali — la review e il refactoring si fanno comunque)

## Differenze chiave rispetto alla variante con repo

| Aspetto | Con repo | No repo |
|---------|----------|---------|
| Codice di partenza | già presente | generato da Copilot allo step 0 |
| Tempo "operativo" | identico | + 3-5' per la generazione iniziale |
| Test infrastructure | preconfigurata | generata insieme al codice |
| Bug "didattici" (latch, blocking ecc.) | preinseriti nei file | introdotti via prompt esplicito |

## Quando preferire questa variante

- Workshop su laptop nuovi / VM dove non vuoi gestire dipendenze
- Sessioni più brevi dove vuoi mostrare anche la **fase di bootstrap**
- Per dimostrare quanto Copilot sia efficace anche da workspace **completamente vuoto**
