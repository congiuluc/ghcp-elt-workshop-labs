# Lab 4 — VHDL: FIFO da zero, testbench, bug hunting

**Durata**: ~30'
**Setup**: una cartella vuota
**Modalità Copilot**: Agent → Custom agent "RTL Reviewer" (Lab 2) → Edit/Inline chat

> 📚 GHDL: <https://ghdl.github.io/ghdl/>

## Obiettivo
1. Generare con Copilot una FIFO sincrona VHDL (con un **bug intenzionale**: latch inferito)
2. Generare un testbench self-checking
3. Trovare il bug col custom agent "RTL Reviewer"
4. Correggerlo e validare

## Setup (2')
```bash
mkdir copilot-lab4 && cd copilot-lab4 && code .
```
Assicurati di avere:
- `.github/copilot-instructions.md` con il coding style HDL del [Lab 2](Lab2-Custom-Instructions-Prompts-Skills.md) (copialo o rigeneralo)
- (Opzionale) `.github/prompts/generate-testbench.prompt.md`
- (Opzionale) `.github/agents/rtl-reviewer.agent.md`

## Step 1 — Genera la FIFO (8')

Modalità **Agent**. Prompt:
```
Genera un modulo VHDL-2008 fifo_sync.vhd con:
- entity fifo_sync
- generic: WIDTH (default 8), DEPTH (default 16)
- porte: clk, rst, wr_en, rd_en, din[WIDTH-1:0], dout[WIDTH-1:0],
  full, empty, almost_full, count[ceil(log2(DEPTH+1))-1:0]
- Implementazione con array interno + puntatori wr_ptr/rd_ptr/count
- Reset sincrono attivo alto

⚠️ INTRODUCI INTENZIONALMENTE QUESTO BUG (non commentarlo nel codice):
- nel process combinatorio che assegna almost_full, ometti il ramo else
  in modo da generare un latch inferito (almost_full deve essere '1' quando
  count >= DEPTH-2, ma NON assegnare nulla nell'altro ramo).

Salva in src/fifo_sync.vhd.
```

> 💡 Se preferisci un'entity senza bug, salta l'ultima istruzione e procedi col Lab — il custom agent dovrebbe comunque trovare miglioramenti minori.

## Step 2 — Genera il testbench (8')

Se hai creato il prompt file nel Lab 2:
```
/generate-testbench filename=src/fifo_sync.vhd
```

Altrimenti, prompt diretto:
```
Genera un testbench VHDL-2008 self-checking per #file:src/fifo_sync.vhd.

Requisiti:
- Clock 100 MHz, reset sincrono attivo alto
- Header aziendale standard
- Stimoli: scrittura fino a riempire (verifica full=1), lettura fino a
  svuotare (verifica empty=1), almost_full a count >= DEPTH-2,
  reset durante operazione, scrittura mentre full
- Self-checking con assert ... severity error
- Termina con std.env.finish
- Salva in src/tb_fifo_sync.vhd
```

Se hai GHDL:
```bash
ghdl -a --std=08 src/fifo_sync.vhd
ghdl -a --std=08 src/tb_fifo_sync.vhd
ghdl -e --std=08 tb_fifo_sync
ghdl -r --std=08 tb_fifo_sync --vcd=out.vcd --stop-time=10us
```

## Step 3 — Code review col custom agent (8')

Seleziona dal dropdown agente il custom agent **"RTL Reviewer"** (creato nel Lab 2). Oppure usa Ask:
```
Fai code review RTL di #file:src/fifo_sync.vhd. Cerca specialmente:
latch inferiti, sensitivity list incomplete, naming non conforme.
```

**Atteso**: l'agent identifica il latch inferito su `almost_full`.

## Step 4 — Fix (4')

**Inline Chat** (`Ctrl+I`) sul process incriminato:
```
Correggi il latch inferito su almost_full aggiungendo un default e
mantenendo il comportamento previsto (1 quando count >= DEPTH-2, 0 altrimenti).
```

Se hai GHDL, ri-simula e controlla che `almost_full` sia ora deterministico nel VCD.

## Discussione (opzionale)
- Sarebbe stato più sicuro come **assegnazione concorrente** (`almost_full <= '1' when count >= DEPTH-2 else '0';`)?
- Cosa cambierebbe con reset asincrono?
- Come introdurresti una nuova generic `ALMOST_FULL_THRESHOLD` con Copilot in Edit mode?

## Cosa imparate
- Generazione end-to-end di un modulo HDL **da zero**
- Iniezione controllata di bug "didattici" via prompt
- Custom agent come "linter intelligente" su codice RTL
- Limiti del modello su HDL — perché serve sempre la review umana
