# Lab 3 (no repo) — GitHub Copilot CLI

**Durata**: ~20'
**Prerequisiti**: Node.js 22+
**Setup**: una cartella vuota

> 📚 Doc:
> - About: <https://docs.github.com/en/copilot/concepts/agents/about-copilot-cli>
> - Install: <https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli>
> - Use: <https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli>

## Setup (3')

```bash
npm install -g @github/copilot
copilot --version

mkdir copilot-lab3 && cd copilot-lab3
copilot
```

Al primo lancio:
1. Conferma trust della cartella (`Yes, and remember`)
2. `/login` e completa il device code flow

Flag utili:
```bash
copilot --resume      # riprende l'ultima sessione nel cwd
copilot --continue    # continua quella corrente
copilot --allow-all   # ⚠️ alias --yolo: NESSUNA conferma sui tool
```

## Step 0 — Bootstrap (3')

Senza uscire dalla CLI, prompt:
```text
Crea in questa cartella un piccolo progetto Python "log-analyzer":
- parser.py con funzione parse_line(line) -> LogEntry
- cli.py che legge un file e stampa il conteggio per livello
- tests/test_parser.py con 3 test pytest
- samples/sim.log con 6 righe di esempio (mix INFO/WARNING/ERROR)
Mostra i diff prima di scrivere i file.
```

Approva i tool quando richiesto. Esegui i test (l'agent stesso può farlo).

## Step 1 — Plan mode (5')

Premi **Shift+Tab** per entrare in **Plan mode** (read-only).

```text
Voglio aggiungere a cli.py un sotto-comando `summarize @samples/sim.log` che
produca un riepilogo testuale degli errori e dei warning, raggruppato per
modulo (deducendolo dalla parte iniziale del messaggio). Genera un piano
dettagliato senza scrivere codice.
```

Review del piano. Premi **Shift+Tab** di nuovo per uscire da Plan mode.

## Step 2 — Agent + approvazione tool (6')

```text
Esegui il piano discusso. Per modifiche multi-file mostrami il diff prima di
applicarle. Lancia pytest alla fine.
```

Strategie di approvazione:
- Prima esecuzione di un comando → `Yes`
- Comandi sicuri ricorrenti (`pytest`, `python -m`) → **"Yes and approve TOOL for the rest of the session"**
- Comandi distruttivi (`rm`, `git push`) → `No` + correzione inline

In sessione:
- `/usage` → premium request consumati
- `/context` → riempimento del context window
- `/compact` → riassume la cronologia

## Step 3 — Riferimenti file e use case shell (3')

Prova a chiedere all'agent (sempre dalla CLI):

```text
# Lavoro su file specifici
Esamina @parser.py e @tests/test_parser.py. Suggerisci 3 test mancanti
con pytest.parametrize. NON modificare ancora i file: solo proposta.

# Use case "shell wizard" — utili anche fuori da progetti applicativi
Crea un Makefile che lancia pytest, calcola coverage e produce un report HTML.
Genera un .gitignore Python completo.
Spiega cosa fa: find . -name "*.pyc" -delete -o -name "__pycache__" -exec rm -rf {} +
Pulisci tutti i file di build (.pytest_cache, __pycache__, *.pyc) ricorsivamente.

# Use case HDL (anche senza file VHDL: sono script puri)
Genera un Makefile per simulare con GHDL tutti i .vhd in src/, elaborate tb_top, dump VCD.
Genera un Tcl script per ModelSim: vlib, vcom di tutti i .vhd, vsim del top, run -all, quit.
```

## Cosa imparate
- Quando usare CLI vs chat di VS Code (CLI = workflow shell, automazione, scripting)
- Plan mode con `Shift+Tab`
- Importanza di leggere e approvare i comandi proposti
- Riferimenti `@file` per dare contesto puntuale
- Copilot CLI come "shell wizard" anche su cartelle non-progetto

---

## Appendice — alternativa legacy `gh copilot`

```bash
gh extension install github/gh-copilot
gh copilot suggest "find vhdl files modified in last 24 hours"
gh copilot explain "git rebase -i HEAD~5 --autosquash --keep-base"
```
Ottima per **un singolo comando** senza entrare in agent loop.
