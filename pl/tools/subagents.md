---
title: "Podagenci"
---

# Podagenci

Podagenci to uruchomienia agenta w tle, wywoływane z istniejącej sesji agenta. Działają we własnej sesji (`agent:<agentId>:subagent:<uuid>`) i po zakończeniu **ogłaszają** swój wynik z powrotem do kanału czatu żądającego.

## Komenda ukośnika

Użyj `/subagents`, aby sprawdzić lub kontrolować uruchomienia podagentów w **bieżącej sesji**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` pokazuje metadane uruchomienia (status, znaczniki czasu, identyfikator sesji, ścieżkę transkryptu, cleanup).

Główne cele:

- Równoległe wykonywanie zadań typu „research / długie zadanie / wolne narzędzie” bez blokowania głównej sesji.
- Izolacja podagentów domyślnie (separacja sesji + opcjonalny sandbox).
- Ograniczenie powierzchni narzędzi trudnych do nadużycia: podagenci **nie** otrzymują narzędzi sesyjnych domyślnie.
- Wsparcie konfigurowalnej głębokości zagnieżdżenia dla wzorca orkiestratora.

Uwaga dotycząca kosztów: każdy podagent ma **własny** kontekst i zużycie tokenów. W przypadku ciężkich lub powtarzalnych zadań ustaw tańszy model dla podagentów, a głównego agenta pozostaw na modelu wyższej jakości. Możesz to skonfigurować przez `agents.defaults.subagents.model` lub nadpisania per-agent.

## Narzędzie

Użyj `sessions_spawn`:

- Uruchamia podagenta (`deliver: false`, globalna kolejka: `subagent`)
- Następnie wykonuje krok announce i publikuje odpowiedź ogłoszenia do kanału czatu żądającego
- Domyślny model: dziedziczy po wywołującym, chyba że ustawisz `agents.defaults.subagents.model` (lub `agents.list[].subagents.model`); jawne `sessions_spawn.model` ma pierwszeństwo.
- Domyślne thinking: dziedziczy po wywołującym, chyba że ustawisz `agents.defaults.subagents.thinking` (lub `agents.list[].subagents.thinking`); jawne `sessions_spawn.thinking` ma pierwszeństwo.

Parametry narzędzia:

- `task` (wymagane)
- `label?` (opcjonalne)
- `agentId?` (opcjonalne; uruchom pod innym identyfikatorem agenta, jeśli dozwolone)
- `model?` (opcjonalne; nadpisuje model podagenta; nieprawidłowe wartości są pomijane, a podagent uruchamia się na domyślnym modelu z ostrzeżeniem w wyniku narzędzia)
- `thinking?` (opcjonalne; nadpisuje poziom thinking dla uruchomienia podagenta)
- `runTimeoutSeconds?` (domyślnie `0`; gdy ustawione, uruchomienie podagenta jest przerywane po N sekundach)
- `cleanup?` (`delete|keep`, domyślnie `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: lista identyfikatorów agentów, które można wskazać przez `agentId` (`["*"]`, aby zezwolić na dowolny). Domyślnie: tylko agent żądający.

Discovery:

- Użyj `agents_list`, aby zobaczyć, które identyfikatory agentów są obecnie dozwolone dla `sessions_spawn`.

Auto-archive:

- Sesje podagentów są automatycznie archiwizowane po `agents.defaults.subagents.archiveAfterMinutes` (domyślnie: 60).
- Archiwizacja używa `sessions.delete` i zmienia nazwę transkryptu na `*.deleted.<timestamp>` (ten sam folder).
- `cleanup: "delete"` archiwizuje natychmiast po announce (transkrypt nadal jest zachowany przez zmianę nazwy).
- Auto-archive działa w trybie best-effort; oczekujące timery są tracone, jeśli gateway zostanie zrestartowany.
- `runTimeoutSeconds` **nie** powoduje auto-archiwizacji; jedynie zatrzymuje uruchomienie. Sesja pozostaje do momentu auto-archiwizacji.
- Auto-archive dotyczy zarówno sesji głębokości 1, jak i 2.

## Zagnieżdżeni podagenci

Domyślnie podagenci nie mogą uruchamiać własnych podagentów (`maxSpawnDepth: 1`). Możesz włączyć jeden poziom zagnieżdżenia, ustawiając `maxSpawnDepth: 2`, co umożliwia **wzorzec orkiestratora**: main → orchestrator sub-agent → worker sub-sub-agents.

### Jak włączyć

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Poziomy głębokości

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Główny agent                                 | Zawsze                      |
| 1     | `agent:<id>:subagent:<uuid>`                 | Podagent (orkiestrator, gdy dozwolona głębokość 2) | Tylko jeśli `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Pod-podagent (worker)                        | Nigdy                        |

### Łańcuch announce

Wyniki wracają w górę łańcucha:

1. Worker głębokości 2 kończy → ogłasza do swojego rodzica (orkiestratora głębokości 1)
2. Orkiestrator głębokości 1 otrzymuje announce, syntetyzuje wyniki, kończy → ogłasza do main
3. Główny agent otrzymuje announce i dostarcza wynik użytkownikowi

Każdy poziom widzi tylko announce od swoich bezpośrednich dzieci.

### Polityka narzędzi według głębokości

- **Głębokość 1 (orkiestrator, gdy `maxSpawnDepth >= 2`)**: Otrzymuje `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, aby zarządzać swoimi dziećmi. Inne narzędzia sesyjne/systemowe pozostają zabronione.
- **Głębokość 1 (worker, gdy `maxSpawnDepth == 1`)**: Brak narzędzi sesyjnych (domyślne zachowanie).
- **Głębokość 2 (worker)**: Brak narzędzi sesyjnych — `sessions_spawn` jest zawsze zabronione na głębokości 2. Nie może uruchamiać dalszych dzieci.

### Limit uruchomień per agent

Każda sesja agenta (na dowolnej głębokości) może mieć maksymalnie `maxChildrenPerAgent` (domyślnie: 5) aktywnych dzieci jednocześnie. Zapobiega to niekontrolowanemu rozrostowi z jednego orkiestratora.

### Cascade stop

Zatrzymanie orkiestratora głębokości 1 automatycznie zatrzymuje wszystkie jego dzieci głębokości 2:

- `/stop` w głównym czacie zatrzymuje wszystkich agentów głębokości 1 i kaskadowo ich dzieci głębokości 2.
- `/subagents kill <id>` zatrzymuje konkretnego podagenta i kaskadowo jego dzieci.
- `/subagents kill all` zatrzymuje wszystkich podagentów dla żądającego i wykonuje kaskadę.

## Uwierzytelnianie

Uwierzytelnianie podagenta jest rozstrzygane według **identyfikatora agenta**, a nie typu sesji:

- Klucz sesji podagenta to `agent:<agentId>:subagent:<uuid>`.
- Magazyn uwierzytelniania jest ładowany z `agentDir` tego agenta.
- Profile uwierzytelniania głównego agenta są scalane jako **zapasowe**; profile agenta mają pierwszeństwo w przypadku konfliktów.

Uwaga: scalanie jest addytywne, więc profile głównego agenta są zawsze dostępne jako zapasowe. W pełni izolowane uwierzytelnianie per agent nie jest jeszcze wspierane.

## Announce

Podagenci raportują wyniki przez krok announce:

- Krok announce działa wewnątrz sesji podagenta (nie w sesji żądającej).
- Jeśli podagent odpowie dokładnie `ANNOUNCE_SKIP`, nic nie zostanie opublikowane.
- W przeciwnym razie odpowiedź announce jest publikowana do kanału czatu żądającego przez kolejne wywołanie `agent` (`deliver=true`).
- Odpowiedzi announce zachowują trasowanie wątków/tematów, gdy jest dostępne (wątki Slack, tematy Telegram, wątki Matrix).
- Wiadomości announce są normalizowane do stabilnego szablonu:
  - `Status:` wyprowadzony z wyniku wykonania (`success`, `error`, `timeout` lub `unknown`).
  - `Result:` treść podsumowania z kroku announce (lub `(not available)` jeśli brak).
  - `Notes:` szczegóły błędu i inne przydatne informacje.
- `Status` nie jest wnioskowany z wyjścia modelu; pochodzi z sygnałów runtime.

Payload announce zawiera linię statystyk na końcu (nawet gdy jest opakowany):

- Czas wykonania (np. `runtime 5m12s`)
- Zużycie tokenów (input/output/total)
- Szacowany koszt, gdy ceny modeli są skonfigurowane (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` oraz ścieżkę do transkryptu (aby główny agent mógł pobrać historię przez `sessions_history` lub sprawdzić plik na dysku)

## Tool Policy (narzędzia podagenta)

Domyślnie podagenci otrzymują **wszystkie narzędzia z wyjątkiem narzędzi sesyjnych** i narzędzi systemowych:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Gdy `maxSpawnDepth >= 2`, podagenci głębokości 1 (orkiestratorzy) dodatkowo otrzymują `sessions_spawn`, `subagents`, `sessions_list` i `sessions_history`, aby mogli zarządzać swoimi dziećmi.

Nadpisanie przez konfigurację:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Współbieżność

Podagenci używają dedykowanej kolejki w tym samym procesie:

- Nazwa kolejki: `subagent`
- Współbieżność: `agents.defaults.subagents.maxConcurrent` (domyślnie `8`)

## Zatrzymywanie

- Wysłanie `/stop` w czacie żądającym przerywa sesję żądającą i zatrzymuje wszystkie aktywne uruchomienia podagentów z niej wywołane, kaskadowo do zagnieżdżonych dzieci.
- `/subagents kill <id>` zatrzymuje konkretnego podagenta i kaskadowo jego dzieci.

## Ograniczenia

- Announce podagenta działa w trybie **best-effort**. Jeśli gateway zostanie zrestartowany, oczekujące „announce back” zostaną utracone.
- Podagenci współdzielą zasoby procesu gateway; traktuj `maxConcurrent` jako zawór bezpieczeństwa.
- `sessions_spawn` jest zawsze nieblokujące: zwraca `{ status: "accepted", runId, childSessionKey }` natychmiast.
- Kontekst podagenta wstrzykuje tylko `AGENTS.md` + `TOOLS.md` (bez `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ani `BOOTSTRAP.md`).
- Maksymalna głębokość zagnieżdżenia to 5 (`maxSpawnDepth` zakres: 1–5). Głębokość 2 jest rekomendowana w większości przypadków.
- `maxChildrenPerAgent` ogranicza liczbę aktywnych dzieci na sesję (domyślnie: 5, zakres: 1–20).
