# Lab 6 (no repo) — Mini progetto HDL end-to-end

**Durata**: ~30'
**Setup**: una cartella vuota
**Modalità Copilot**: tutta la "cassetta degli attrezzi"

## Obiettivo
Realizzare end-to-end **da zero** un mini-IP a scelta usando l'intero workflow:
- `copilot-instructions.md` con il coding style aziendale (Lab 2 no-repo)
- Prompt file riutilizzabile (Lab 2 no-repo)
- Custom agent "RTL Reviewer" (Lab 2 no-repo)
- Plan mode + Agent mode (Lab 1 no-repo)
- Copilot CLI per script di build/sim (Lab 3 no-repo)

## Setup (2')
```bash
mkdir copilot-lab6 && cd copilot-lab6 && code .
mkdir -p src tb sim .github/prompts .github/agents
```
Copia (o rigenera) `.github/copilot-instructions.md`, il prompt file `generate-testbench.prompt.md` e l'agent `rtl-reviewer.agent.md` dal Lab 2 no-repo.

## Scegli un progetto (in team)

### Opzione A — UART transmitter (VHDL)
- Entity `uart_tx` parametrico (`BAUD`, `CLK_FREQ_HZ`, `DATA_BITS`, `PARITY`)
- FSM: IDLE → START → DATA → PARITY → STOP
- Testbench self-checking con loopback verso un `uart_rx` (anch'esso da generare)

### Opzione B — Counter parametrico modulo programmabile (Verilog/SV)
- `counter_modulo` up/down, modulo programmabile da porta
- Output: `count`, `tc` (terminal count)
- Refactoring guidato + testbench + SDC

### Opzione C — SPI master (VHDL o SV)
- 4 modi (CPOL/CPHA), `DATA_BITS` parametrico
- Testbench con slave loopback

## Workflow consigliato

1. **Plan mode**: chiedi a Copilot un piano dettagliato per realizzare l'IP scelto, partendo da cartella vuota
2. Review del piano in team — modifica/raffina
3. **Agent mode**: scaffold completo (entity/module + testbench + Makefile / `compile.do`)
4. **Custom agent "RTL Reviewer"**: review finale del codice generato
5. **Copilot CLI**: rifinisci lo script di simulazione (`Makefile` per GHDL o `compile.do` per ModelSim)
6. (Se tempo) Esegui la simulazione

## Esempio di prompt iniziale (Plan mode)

```
Sto partendo da una cartella vuota. Voglio realizzare un <Opzione A/B/C>.
Il workspace ha già copilot-instructions.md con lo standard aziendale,
un prompt file /generate-testbench, e un custom agent "RTL Reviewer".

Genera un piano dettagliato che includa:
- struttura cartelle
- file RTL da creare (con responsabilità)
- testbench e copertura funzionale
- script di build/simulazione (Makefile per GHDL)
- ordine consigliato di esecuzione e checkpoint di verifica
```

## Discussione finale (5')

In plenaria:
- Quanto è stato utile Copilot partendo davvero da zero?
- Quali prompt/instructions sono finiti nel vostro "kit personale"?
- Quali use case quotidiani vorreste automatizzare con prompt files o custom agent?
- Cosa portate a casa di immediatamente applicabile lunedì mattina?

## Cosa imparate
- Mettere insieme tutti i pezzi della giornata in un workflow reale
- Iniziare un progetto **da zero** in modo guidato e strutturato
- Iniziare a costruire un kit di personalizzazioni riutilizzabile in azienda
