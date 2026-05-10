# Lab 1 (no repo) — Agent mode + Plan mode

**Durata**: ~25'
**Setup**: una cartella vuota aperta in VS Code
**Modalità Copilot**: Plan mode → Agent mode

> 📚 Doc: <https://code.visualstudio.com/docs/copilot/agents/overview>

## Obiettivo
Costruire **da zero** un piccolo CLI `log-analyzer` che:
1. Legge un file di log di simulazione (formato `[timestamp] LEVEL message`)
2. Conta `ERROR` / `WARNING` / `NOTE`
3. Supporta `--format text|json|junit` per l'output

Il vero focus è il **workflow Plan → Agent**, non il tool finale.

## Setup (2')
```bash
mkdir copilot-lab1 && cd copilot-lab1 && code .
```
Scegli un linguaggio (uno qualsiasi: Python, Java, C++, .NET, Node) — dichiaralo nel primo prompt e Copilot scaffolderà di conseguenza.

Apri Copilot Chat (`Ctrl+Alt+I`).

## Step 0 — Bootstrap del progetto (3')

Modalità **Agent**. Prompt:
```
Crea da zero un progetto <linguaggio scelto> chiamato log-analyzer:
- Struttura minimale (src/, tests/)
- Un modulo che parsifica linee del tipo "[12:34:56] ERROR something"
  in oggetti LogEntry (timestamp, level, message)
- Un CLI che stampa un report testuale del conteggio per livello
- Almeno 2 test unitari per il parser
- File di esempio samples/sim.log con 6-8 righe miste
- README con istruzioni di build/test
NON aggiungere ancora il formato JSON/JUnit: lo facciamo allo step successivo.
```

Verifica che build/test passino.

## Step 1 — Plan mode (8')

Dal **dropdown agente** seleziona **Plan** (built-in, read-only).

Prompt:
```
Devo aggiungere al CLI il supporto per --format json e --format junit
(JUnit XML), selezionabile da CLI. Output su stdout o su file con --output.
Il file XML deve includere ogni LogEntry come <testcase> con
<failure> per gli errori e <skipped> per i warning. Aggiungi anche test
per entrambi i formati. Genera un piano dettagliato.
```

Review del piano:
- Chiedi di aggiungere step per `--help`
- Chiedi di estrarre la formattazione in moduli/classi separate
- Chiedi di aggiornare anche eventuale build file (`pom.xml`, `CMakeLists.txt`, `.csproj`, `package.json`)

Salva o screenshotta il piano per la discussione finale.

## Step 2 — Agent mode (10')

Premi **Start Implementation** (handoff dal piano) o cambia agente in **Agent**.

Prompt:
```
Esegui il piano discusso. Per ogni file modificato mostra il diff.
Esegui la suite di test alla fine.
```

Approva/rifiuta i passi. Verifica:
```bash
# Esempi (adatta al linguaggio scelto)
<run-tests>
<run-cli> samples/sim.log --format junit
<run-cli> samples/sim.log --format json --output report.json
```

## Step 3 — Reflection (2')
- Quanto del piano iniziale era già "implementabile" senza correzioni?
- Cosa ha sbagliato l'agent? Cosa avete corretto a mano?
- Lo step 0 (bootstrap) è stato più o meno preciso degli step incrementali?

## Bonus
- Aggiungi un formato `markdown`
- Aggiungi un flag `--severity-min warning` che filtra l'output
- Genera una GitHub Action che esegue i test e pubblica il JUnit XML

## Cosa imparate
- Differenza tra Plan e Agent mode
- Quanto Copilot è efficace anche partendo da zero
- L'importanza di un buon piano prima dell'esecuzione
- Quando intervenire come "umano in the loop"
