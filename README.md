# Esercitazioni

Raccolta dei lab del corso.

| # | File | Durata |
|---|------|--------|
| 1 | [Lab1-Agent-Plan-Mode.md](Lab1-Agent-Plan-Mode.md) | 25' |
| 2 | [Lab2-Custom-Instructions-Prompts-Skills.md](Lab2-Custom-Instructions-Prompts-Skills.md) | 20' |
| 3 | [Lab3-Copilot-CLI.md](Lab3-Copilot-CLI.md) | 20' |
| 4 | [Lab4-VHDL-Testbench.md](Lab4-VHDL-Testbench.md) | 30' |
| 5 | [Lab5-Verilog-FSM.md](Lab5-Verilog-FSM.md) | 30' |
| 6 | [Lab6-Finale-HDL.md](Lab6-Finale-HDL.md) | 30' |

### Percorso general purpose (Python / Java / .NET)

Alternativa ai lab HDL 4-6 su linguaggi applicativi:

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

Toolchain (solo se vuoi *eseguire* davvero il codice generato):

- **Lab 1/3 general purpose**: Python 3.11+, oppure Java 21, oppure CMake/g++, oppure .NET 10
- **Lab 4/5/6 HDL**: GHDL (VHDL) o Icarus Verilog (opzionali — la review e il refactoring si fanno comunque)
