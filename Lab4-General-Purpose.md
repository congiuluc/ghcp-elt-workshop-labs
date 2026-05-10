# Lab 4 — Test generation + bug hunting

**Durata**: ~30'
**Setup**: una cartella vuota
**Linguaggi**: Python · Java · .NET (scegli uno; gli step sono identici)
**Modalità Copilot**: Agent → Custom agent "Code Reviewer" → Edit/Inline chat

> Variante general-purpose del [Lab 4 HDL](Lab4-VHDL-Testbench.md). Stesso schema didattico (genera codice con bug intenzionale → testbench → review → fix), applicato a una **FIFO/RingBuffer** invece di un modulo VHDL.

## Obiettivo
1. Generare con Copilot una `RingBuffer<T>` thread-safe (con un **bug intenzionale**: race condition o off-by-one)
2. Generare la suite di test
3. Trovare il bug col custom agent "Code Reviewer"
4. Correggerlo e validare

## Setup (2')
```bash
mkdir copilot-lab4-gp && cd copilot-lab4-gp && code .
```
Crea (o riutilizza dal Lab 2) `.github/copilot-instructions.md` con il coding style del linguaggio scelto. Riferimento: [varianti general purpose Lab 2](Lab2-Custom-Instructions-Prompts-Skills.md#varianti-general-purpose-python--java--c--net).

Crea anche un custom agent **Code Reviewer** in `.github/agents/code-reviewer.agent.md`:

```markdown
---
description: Senior reviewer per Python/Java/.NET — concorrenza, edge case, naming
tools: ['search/codebase', 'search', 'editFiles']
---

Sei un senior backend engineer. Quando ti chiedono review devi cercare SEMPRE:
- Race condition / sezioni critiche non protette
- Off-by-one su indici, capacità, puntatori wrap-around
- Eccezioni inghiottite o generiche (catch Exception/Throwable)
- Naming non conforme allo standard del progetto
- Test mancanti su edge case (vuoto, pieno, wrap)

Output: tabella | File | Riga | Severità | Problema | Fix proposto |
Severità: BLOCKER | MAJOR | MINOR | NIT
Termina con riepilogo numerico.
```

## Step 1 — Genera la RingBuffer (8')

Modalità **Agent**. Scegli il prompt per il tuo linguaggio.

### Variant Python
```
Genera src/ring_buffer.py con classe RingBuffer thread-safe:
- __init__(capacity: int)
- put(item) -> None  (blocca o solleva Full se pieno?  → solleva queue.Full)
- get() -> item      (solleva queue.Empty se vuoto)
- __len__, is_full, is_empty
- Usa threading.Lock per thread safety
- Type hints obbligatori, docstring Google-style

⚠️ INTRODUCI INTENZIONALMENTE QUESTO BUG (NON commentarlo):
- in __len__ calcola la lunghezza con (write_idx - read_idx) % capacity
  SENZA tenere un contatore separato → restituisce 0 sia quando è vuoto
  sia quando è PIENO (classico bug del ring buffer).

Aggiungi un breve commento iniziale TODO generico, NIENTE menzione del bug.
```

### Variant Java
```
Genera src/main/java/com/example/RingBuffer.java come classe generica
RingBuffer<T> thread-safe:
- costruttore RingBuffer(int capacity)
- void put(T item) throws IllegalStateException se pieno
- T get() throws NoSuchElementException se vuoto
- int size(), boolean isFull(), boolean isEmpty()
- Sincronizza con synchronized o ReentrantLock
- Javadoc su API pubblica

⚠️ INTRODUCI INTENZIONALMENTE QUESTO BUG (NON commentarlo):
- size() calcola (writeIdx - readIdx) % capacity SENZA contatore separato
  → restituisce 0 sia se vuoto sia se PIENO.
```

### Variant .NET
```
Genera src/RingBuffer/RingBuffer.cs come classe generica RingBuffer<T>
thread-safe (target net10.0):
- costruttore RingBuffer(int capacity)
- void Put(T item)  → InvalidOperationException se pieno
- T Get()           → InvalidOperationException se vuoto
- int Count, bool IsFull, bool IsEmpty
- lock(_sync) per sincronizzazione
- XML doc comments su API pubblica

⚠️ INTRODUCI INTENZIONALMENTE QUESTO BUG (NON commentarlo):
- Count calcola (_writeIdx - _readIdx) % _capacity senza un contatore
  dedicato → restituisce 0 sia se vuoto sia se PIENO.
```

## Step 2 — Genera i test (8')

### Variant Python
```
Genera tests/test_ring_buffer.py con pytest:
- test caso nominale (put/get singolo)
- test FIFO order su 5 elementi
- test pieno → put solleva Full
- test vuoto → get solleva Empty
- test wrap-around (riempi, svuota a metà, riempi di nuovo)
- test concorrenza: 4 thread producer + 4 consumer per 1000 op,
  asserisci che il numero di get == numero di put
Usa pytest.fixture per la fixture buffer di capacity 8.
```
Esegui: `pytest -q`

### Variant Java
```
Genera src/test/java/com/example/RingBufferTest.java con JUnit 5 + AssertJ:
- @ParameterizedTest per put/get base, FIFO, full, empty, wrap
- @Test concorrenza con ExecutorService (4 producer + 4 consumer, 1000 op)
- AssertJ per asserzioni
Crea pom.xml con junit-jupiter 5.x e assertj-core.
```
Esegui: `mvn -q test`

### Variant .NET
```
Crea solution con `dotnet new sln -n RingBufferLab` e:
- progetto src/RingBuffer/RingBuffer.csproj  (classlib net10.0)
- progetto tests/RingBuffer.Tests/RingBuffer.Tests.csproj (xunit, FluentAssertions, Moq)
- aggiungili alla solution
- in RingBuffer.Tests genera xUnit Theory parametrizzati per put/get/full/empty/wrap
- un Fact con Parallel.For per testare concorrenza (4 producer + 4 consumer × 1000 op)
- usa FluentAssertions
```
Esegui: `dotnet test`

**Atteso**: il test sulla `size`/`Count`/`__len__` quando il buffer è pieno **fallisce** (rivela il bug).

## Step 3 — Code review col custom agent (8')

Seleziona dal dropdown il custom agent **"Code Reviewer"**:
```
Fai code review di #file:<path-al-RingBuffer>. Cerca specialmente: 
race condition, off-by-one, edge case su buffer pieno/vuoto/wrap.
```

**Atteso**: l'agent identifica il bug su `size`/`Count`/`__len__` e l'ambiguità vuoto-vs-pieno con due indici puri.

## Step 4 — Fix (4')

**Inline Chat** (`Ctrl+I`) sulla classe:
```
Risolvi l'ambiguità vuoto-vs-pieno aggiungendo un campo count separato
incrementato in put e decrementato in get (sotto lock). Aggiorna size()
di conseguenza. Mantieni la firma pubblica invariata.
```

Rilancia i test: tutti devono passare.

## Discussione
- Quale variante alternativa avreste usato (es. lasciare 1 slot inutilizzato per distinguere vuoto/pieno)?
- Il custom agent ha trovato anche problemi minori non richiesti (naming, docstring)?
- Il test di concorrenza era flaky? Cosa lo renderebbe più robusto?

## Cosa imparate
- Generazione di codice + test end-to-end **da zero**
- Iniezione controllata di bug "didattici" (anche su linguaggi general purpose)
- Custom agent come "linter intelligente" su concorrenza ed edge case
- Limiti del modello e importanza della review umana — anche fuori da HDL
