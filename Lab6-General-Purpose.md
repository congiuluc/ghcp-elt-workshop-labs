# Lab 6 — Mini progetto end-to-end

**Durata**: ~30'
**Setup**: una cartella vuota
**Linguaggi**: Python · Java · .NET (scegli uno)
**Modalità Copilot**: tutta la "cassetta degli attrezzi"

> Variante general-purpose del [Lab 6 HDL](Lab6-Finale-HDL.md). Stesso workflow end-to-end, su un mini-IP **software** invece di un IP HDL.

## Obiettivo
Realizzare end-to-end **da zero** un mini-progetto a scelta usando l'intero workflow:
- `copilot-instructions.md` con il coding style del linguaggio (Lab 2 GP)
- Prompt file riutilizzabile (es. `/add-tests`)
- Custom agent "Code Reviewer" (Lab 4 GP)
- Plan mode + Agent mode (Lab 1)
- Copilot CLI per script di build/lint/CI (Lab 3)

## Setup (2')
```bash
mkdir copilot-lab6-gp && cd copilot-lab6-gp && code .
mkdir -p .github/prompts .github/agents
```
Copia (o rigenera) `copilot-instructions.md`, un prompt file `add-tests.prompt.md`, e l'agent `code-reviewer.agent.md` dai lab precedenti.

## Scegli un progetto (in team)

### Opzione A — Rate limiter (token bucket)
- Classe `RateLimiter` con configurazione `capacity`, `refill_per_second`
- Metodo `try_acquire(n=1) -> bool`
- Test parametrici + test di concorrenza
- CLI di stress-test che spara N richieste/sec e stampa la frazione accettata

### Opzione B — Mini key-value cache LRU
- `LRUCache(capacity)` con `get/put` O(1) (dict + linked list / `LinkedHashMap` / `OrderedDict`)
- TTL opzionale per chiave
- Statistiche (hit ratio, evictions)
- Suite di test + benchmark veloce

### Opzione C — Parser di un mini formato di log
- Grammatica: `[timestamp] LEVEL [module] message [k=v ...]`
- API: `parse_line(s) -> LogRecord`, `parse_file(path) -> Iterator[LogRecord]`
- CLI: `app summarize file.log --by module`
- Test su edge case (linee vuote, kv malformate, severità sconosciute)

## Workflow consigliato

1. **Plan mode**: chiedi a Copilot un piano dettagliato per l'opzione scelta, partendo da cartella vuota e specificando il linguaggio
2. Review del piano in team — modifica/raffina (struttura cartelle, dipendenze, test framework, target di linting)
3. **Agent mode**: scaffold completo (sorgenti + test + build file + README)
4. **Custom agent "Code Reviewer"**: review finale del codice generato
5. **Copilot CLI**: rifinisci uno script di build/lint/CI:
   - Python: `Makefile` con `make test lint`, `pyproject.toml`, GitHub Action
   - Java: `pom.xml` Maven o `build.gradle.kts`, GitHub Action `mvn -q verify`
   - .NET: `dotnet test`, `dotnet format`, GitHub Action con `actions/setup-dotnet@v4`
6. Esegui i test e il linter

## Esempio di prompt iniziale (Plan mode)

```
Sto partendo da una cartella vuota. Voglio realizzare <Opzione A/B/C> in
<Python 3.11+ | Java 21 | .NET 10>.

Il workspace ha già:
- copilot-instructions.md con lo standard aziendale per <linguaggio>
- prompt file /add-tests
- custom agent "Code Reviewer"

Genera un piano dettagliato che includa:
- struttura cartelle (src/, tests/, docs/)
- file da creare con responsabilità
- dipendenze esterne minime
- strategia di test (parametrici + concorrenza dove rilevante)
- script di build/lint/CI
- ordine consigliato di esecuzione e checkpoint di verifica
```

## Discussione finale (5')

In plenaria:
- Quanto è stato utile Copilot partendo da zero (vs HDL)?
- Quali prompt/instructions sono finiti nel vostro "kit personale"?
- Quali differenze avete percepito fra HDL e general purpose nella **qualità** del codice generato?
- Cosa portate a casa di immediatamente applicabile lunedì mattina?

## Cosa imparate
- Mettere insieme tutti i pezzi della giornata in un workflow reale
- Iniziare un progetto **da zero** in modo guidato e strutturato (anche fuori HDL)
- Iniziare a costruire un kit di personalizzazioni riutilizzabile in azienda — trasversale ai linguaggi del team
