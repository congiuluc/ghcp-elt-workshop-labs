# Lab 5 (no repo) — Verilog: FSM da zero, refactoring Mealy → Moore, SDC

**Durata**: ~30'
**Setup**: una cartella vuota
**Modalità Copilot**: Agent (bootstrap) → Custom agent "RTL Reviewer" → Edit/Agent

> 📚 Icarus Verilog: <https://steveicarus.github.io/iverilog/> · Verilator: <https://www.veripool.org/verilator>

## Obiettivo
1. Generare con Copilot un controllore semaforico (Verilog) **con 2 bug intenzionali**
2. Identificarli col custom agent
3. Refattorare da Mealy a Moore
4. Generare un file SDC e un testbench SV

## Setup (1')
```bash
mkdir copilot-lab5 && cd copilot-lab5 && code .
```
Assicurati di avere `.github/copilot-instructions.md` (Lab 2 no-repo) e il custom agent **RTL Reviewer**.

## Step 1 — Genera la FSM (6')

Modalità **Agent**. Prompt:
```
Genera un modulo Verilog traffic_light_fsm.v con:
- input clk, rst_n, ped_button
- output [1:0] light_main, light_ped (00=red, 01=yellow, 10=green)
- FSM Mealy a 4 stati: GREEN_MAIN (20 cicli) → YELLOW (5) → RED_PED (30) → RED_TO_GREEN (5) → GREEN_MAIN
- ped_button accelera la transizione GREEN_MAIN → YELLOW

⚠️ INTRODUCI INTENZIONALMENTE QUESTI BUG (NON commentarli nel codice):
1. Usa '=' (blocking) invece di '<=' nei process always @(posedge clk)
   che assegnano lo stato e il timer.
2. Il timer NON viene mai resettato quando si transita di stato
   (overflow possibile su esecuzioni lunghe).

Aggiungi un breve commento in cima al file con TODO generici (refactor,
add testbench), senza menzionare i bug.

Salva in src/traffic_light_fsm.v.
```

## Step 2 — Code review (8')

Custom agent **"RTL Reviewer"** (oppure Ask):
```
Fai code review SystemVerilog/Verilog di #file:src/traffic_light_fsm.v.
Cerca specialmente: blocking vs non-blocking, sensitivity list,
reset polarity/sync, timer privo di bound.
```

**Atteso** — almeno:
1. Uso di `=` in `always @(posedge clk)` invece di `<=`
2. Timer non resettato tra stati
3. (Eventualmente) reset polarity / mix sync-async

## Step 3 — Refactoring Mealy → Moore (10')

Modalità **Edit** o **Agent**. Prompt:
```
Refattora #file:src/traffic_light_fsm.v da FSM Mealy a FSM Moore.

Vincoli:
- Mantieni l'interfaccia di porte invariata
- Output light_main e light_ped DEVONO essere registrati
- Correggi i blocking '=' in always_ff (usa '<=')
- Aggiungi reset del timer alla transizione di stato
- Usa always_ff e always_comb (SystemVerilog)
- Mantieni i tempi di stato (20/5/30/5) come parameter
- Aggiungi assertions SVA per: timer_max, output one-hot per stato,
  no green simultaneo main+ped

Salva come src/traffic_light_fsm_moore.sv (mantieni l'originale per confronto).
```

## Step 4 — SDC constraints (3')

```
Genera src/constraints.sdc per il modulo top traffic_light_fsm assumendo:
- Clock primario clk = 100 MHz
- Input/output delay = 2 ns
- False path su rst_n
- ped_button asincrono — false path o multicycle?
Aggiungi commenti che spiegano ogni constraint.
```

## Step 5 — Testbench (2')

```
Genera src/tb_traffic.sv per la versione Moore.

Scenari:
- Sequenza completa green → yellow → red(ped) → red_to_green → green
- ped_button: transizione dopo timer
- Reset durante red(ped): torna in green dopo reset
Usa $error per i fallimenti, $finish alla fine.
```

Se hai Icarus:
```bash
iverilog -g2012 src/traffic_light_fsm_moore.sv src/tb_traffic.sv -o sim
vvp sim
```

## Cosa imparate
- Refactoring guidato da Copilot mantenendo il contratto di interfaccia
- Generazione di constraint files (Copilot conosce SDC/TCL meglio di quanto si pensi)
- Differenza Mealy vs Moore vista all'opera
- Iniezione controllata di bug "didattici" anche su Verilog
