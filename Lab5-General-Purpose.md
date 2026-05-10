# Lab 5 (no repo, general purpose) — State machine: refactoring + bug hunting

**Durata**: ~30'
**Setup**: una cartella vuota
**Linguaggi**: Python · Java · .NET (scegli uno; gli step sono identici)
**Modalità Copilot**: Agent (bootstrap) → Custom agent "Code Reviewer" → Edit/Agent

> Variante general-purpose del [Lab 5 HDL](Lab5-Verilog-FSM.md). Stesso schema didattico (FSM con 2 bug intenzionali → review → refactoring), applicato a un controllore semaforico software invece di Verilog.

## Obiettivo
1. Generare un `TrafficLightController` con **2 bug intenzionali** (mutable shared state, switch senza default)
2. Identificarli col custom agent
3. Refattorare verso un design più pulito (state pattern / enum + transition map)
4. Generare test parametrizzati che coprano la sequenza completa

## Setup (1')
```bash
mkdir copilot-lab5-gp && cd copilot-lab5-gp && code .
```
Riutilizza `.github/copilot-instructions.md` e il custom agent **Code Reviewer** del [Lab 4 GP](Lab4-General-Purpose.md).

## Step 1 — Genera la FSM (6')

Modalità **Agent**.

### Variant Python
```
Genera src/traffic_light.py con classe TrafficLightController:
- stati: GREEN_MAIN (20 tick) → YELLOW (5) → RED_PED (30) → RED_TO_GREEN (5) → GREEN_MAIN
- metodo tick() avanza di 1 tick
- metodo press_pedestrian_button() accelera GREEN_MAIN → YELLOW
- proprietà main_light, ped_light → "red"|"yellow"|"green"

⚠️ INTRODUCI INTENZIONALMENTE QUESTI BUG (NON commentarli):
1. Usa una LISTA mutable a livello di CLASSE come default per i timer
   (state_timers = [20, 5, 30, 5]) e modificala in tick() → mutable default
   condiviso tra istanze.
2. In tick() usa un if/elif sugli stati SENZA un branch else finale
   → stato non gestito = silenzioso no-op, nessuna eccezione.

Aggiungi solo un TODO generico in cima, niente menzione dei bug.
```

### Variant Java
```
Genera src/main/java/com/example/TrafficLightController.java:
- enum State { GREEN_MAIN, YELLOW, RED_PED, RED_TO_GREEN }
- timer cycles: 20, 5, 30, 5
- void tick(), void pressPedestrianButton()
- State currentState(), String mainLight(), String pedLight()

⚠️ INTRODUCI INTENZIONALMENTE QUESTI BUG (NON commentarli):
1. Memorizza i timer in una static Map<State,Integer> condivisa fra istanze
   e mutala in tick() → race + stato condiviso.
2. tick() usa un switch SENZA default (o senza throw): uno stato extra
   in futuro passerebbe inosservato.
```

### Variant .NET
```
Genera src/TrafficLight/TrafficLightController.cs (target net10.0):
- enum TrafficState { GreenMain, Yellow, RedPed, RedToGreen }
- timer cicli 20/5/30/5
- public void Tick(), public void PressPedestrianButton()
- public TrafficState CurrentState, public string MainLight, public string PedLight
- XML doc comments

⚠️ INTRODUCI INTENZIONALMENTE QUESTI BUG (NON commentarli):
1. I timer per stato vivono in un Dictionary<TrafficState,int> STATIC mutato
   in Tick() → stato condiviso fra istanze.
2. Tick() usa uno switch SENZA un ramo default che lanci → silenziosamente
   no-op se aggiungi un nuovo stato.
```

## Step 2 — Code review (8')

Custom agent **"Code Reviewer"**:
```
Fai code review di #file:<path-al-TrafficLightController>. Cerca specialmente:
mutable shared state (default mutabili, static collections), gestione esaustiva
di switch/match (default/else mancante), thread safety.
```

**Atteso** — almeno:
1. Mutable shared state (lista/Map/Dictionary di livello classe o static)
2. Switch/if-elif senza branch finale di "stato non gestito"
3. (Bonus) Mancanza di immutabilità sull'API esposta

## Step 3 — Refactoring (10')

Modalità **Edit** o **Agent**. Prompt:
```
Refattora TrafficLightController seguendo questi vincoli:

1. Stato di transizione e durate immutabili a livello di tipo
   (Python: dataclass(frozen=True) o tuple namedtuple;
    Java: record o enum con metodo nextState();
    .NET: readonly record struct o IReadOnlyDictionary<TrafficState, StateConfig>).

2. Nessuno stato condiviso fra istanze: i timer sono campi di istanza.

3. Nello switch/if-elif aggiungi un ramo finale che lancia
   (Python: raise AssertionError; Java: throw new IllegalStateException;
    .NET: throw new UnreachableException o ArgumentOutOfRangeException).

4. API pubblica invariata.
5. Aggiungi una guard nel costruttore se la capacità tick è non valida.
```

## Step 4 — Test parametrizzati (5')

Genera test (`pytest.parametrize` / JUnit `@ParameterizedTest` / xUnit `Theory`) per:
- sequenza completa green → yellow → red(ped) → red_to_green → green
- press_pedestrian_button durante GREEN_MAIN → transizione anticipata a YELLOW
- 2 istanze in parallelo NON devono interferire (verifica del fix #1)
- aggiungere un nuovo valore all'enum/stato e confermare che ora il default branch lancia (per Python: monkey-patch dello stato)

Esegui:
- Python: `pytest -q`
- Java:   `mvn -q test`
- .NET:   `dotnet test`

## Cosa imparate
- Refactoring guidato da Copilot mantenendo il contratto pubblico
- Riconoscere code smell tipici (mutable shared state, switch non esaustivi) — analoghi software dei "latch inferiti" HDL
- Iniezione di bug "didattici" anche fuori da HDL
- Differenza pratica fra approccio "datafied" (transition table) e if/elif annidati
