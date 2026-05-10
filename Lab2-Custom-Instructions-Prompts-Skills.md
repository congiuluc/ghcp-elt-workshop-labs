# Lab 2 — Custom instructions, Prompt files, Custom agent

**Durata**: ~20'
**Setup**: una cartella vuota (può essere la stessa del Lab 1, o una nuova)
**Modalità Copilot**: Ask + Agent

> 📚 Doc:
> - Custom instructions: <https://code.visualstudio.com/docs/copilot/customization/custom-instructions>
> - Prompt files: <https://code.visualstudio.com/docs/copilot/customization/prompt-files>
> - Custom agents: <https://code.visualstudio.com/docs/copilot/customization/custom-agents>

## Obiettivo
Personalizzare Copilot creando da zero:
1. `copilot-instructions.md` — coding style HDL aziendale
2. Prompt file per generare testbench
3. Custom agent "RTL Reviewer"

Tutto questo **senza alcun codice preesistente**: useremo Copilot stesso per generare un piccolo modulo VHDL su cui testare le personalizzazioni.

## Setup (1')
```bash
mkdir copilot-lab2 && cd copilot-lab2 && code .
mkdir -p .github/prompts .github/agents src
```

## Step 1 — Workspace instructions (5')

Crea `.github/copilot-instructions.md` (o chiedi a Copilot in Agent mode di generarlo):

```markdown
# Coding style HDL aziendale

Quando generi o modifichi codice VHDL/Verilog/SystemVerilog applica sempre:

## Header file (obbligatorio)
-- =============================================================
-- Company:    <company>
-- Project:    <project>
-- Module:     <module>
-- Description:<one-line>
-- Author:     <author>
-- Date:       <YYYY-MM-DD>
-- License:    Proprietary
-- =============================================================

## Naming
- signal/variable: snake_case
- generic/parameter/constant: UPPER_SNAKE
- entity/module: snake_case
- testbench: prefisso `tb_`

## Stile RTL
- Reset sincrono attivo alto, nome `rst`
- Clock primario: `clk`
- Mai latch inferiti: tutti i rami if/case devono assegnare i segnali
- `case` deve avere sempre `when others`/`default`
- Process clocked separati dai combinatori
- In Verilog: `<=` in always_ff, `=` in always_comb

## Testbench
- Self-checking con assertions
- Termina con `std.env.finish` (VHDL) o `$finish` (Verilog)
- Includi clock generation, stimoli, check
```

**Verifica immediata**. In chat (Ask):
```
Genera un'entity VHDL `edge_detector` con clk, rst, sig_in, rising_edge_out, falling_edge_out.
Salva in src/edge_detector.vhd.
```
Controlla che header, naming e reset sync siano applicati senza che tu li abbia chiesti.

> 💡 In alternativa puoi usare `AGENTS.md` nella root: cross-compatibile con altri tool AI.

## Step 2 — Prompt file riutilizzabile (5')

Crea `.github/prompts/generate-testbench.prompt.md`:

```markdown
---
description: Genera un testbench VHDL self-checking standard aziendale
agent: agent
---

Genera un testbench VHDL-2008 self-checking per l'entity in #file:${input:filename}.

Requisiti:
- Header aziendale come da copilot-instructions.md
- Clock 100 MHz, reset sincrono attivo alto
- Stimoli: caso nominale + 2 edge case + reset durante operazione
- Asserzioni con assert ... severity error
- Termina con std.env.finish
- Salva in tb_${input:filename}
```

Invocalo dalla chat:
```
/generate-testbench filename=edge_detector.vhd
```

## Step 3 — Custom agent "RTL Reviewer" (8')

Crea `.github/agents/rtl-reviewer.agent.md`:

```markdown
---
description: Esperto RTL VHDL/Verilog per code review
tools: ['search/codebase', 'search', 'editFiles']
model: Claude Sonnet 4.6 (copilot)
---

Sei un senior RTL engineer.
Quando l'utente ti chiede di fare review di codice VHDL/Verilog/SystemVerilog devi:

1. Cercare SEMPRE:
   - Latch inferiti involontari
   - Sensitivity list incomplete
   - Mix blocking/non-blocking errato (Verilog)
   - Reset inconsistenti (sync vs async)
   - Output combinatori a rischio glitch
   - Naming non conforme allo standard aziendale

2. Output: tabella markdown con colonne:
   | File | Riga | Severità | Problema | Fix proposto |

3. Severità: BLOCKER | MAJOR | MINOR | NIT

4. Non modificare codice senza chiedere conferma esplicita.

5. Termina sempre con un riepilogo numerico:
   "Riepilogo: 2 BLOCKER, 1 MAJOR, 3 MINOR".
```

**Test del custom agent**. Selezionalo dal dropdown e prova:
```
Fai code review di #file:edge_detector.vhd
```

Per provare un caso "interessante" chiedi prima a Copilot in Agent mode:
```
Modifica src/edge_detector.vhd inserendo intenzionalmente un latch inferito
nel process di rilevamento del fronte (ometti il default in un ramo combinatorio).
NON aggiungere commenti che segnalino il bug.
```
Poi rilancia il review: l'agent deve trovare il latch.

## Cosa imparate
- Differenza tra istruzioni globali (workspace) e istruzioni di task (prompt file)
- Come standardizzare l'output di Copilot in azienda
- Come creare un agent specializzato con tool ristretti
- Quanto è efficace Copilot anche su un workspace appena nato

---

## Varianti general purpose (Python / Java / C++ / .NET)

Stesso schema, su una cartella vuota in cui chiedi a Copilot di generare un piccolo modulo (es. `parser di stringhe` o `calcolatrice di espressioni`). Le `copilot-instructions.md` per i 4 linguaggi sono identiche alla [variante con repo](../Lab2-Custom-Instructions-Prompts-Skills.md#varianti-general-purpose-python--java--c).
