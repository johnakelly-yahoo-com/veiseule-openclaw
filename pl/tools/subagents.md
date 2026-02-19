---
summary: "Subagenci: uruchamianie odizolowanych przebiegów agentów, które ogłaszają wyniki z powrotem w czacie żądającym"
read_when:
  - Chcesz wykonywać pracę w tle/równolegle za pomocą agenta
  - Zmieniasz politykę sessions_spawn lub narzędzi subagenta
title: "Stopping Sub-Agents"
---

# Nadpisania per agent

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

## Polecenie z ukośnikiem

Użyj komendy ukośnika `/subagents`, aby sprawdzić i kontrolować uruchomienia subagentów w bieżącej sesji:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` pokazuje metadane uruchomienia (status, znaczniki czasu, id sesji, ścieżka transkryptu, czyszczenie).

Główne cele:

- Równoleglenie pracy typu „research / długie zadanie / wolne narzędzie” bez blokowania głównego przebiegu.
- Domyślna izolacja sub‑agentów (separacja sesji + opcjonalny sandboxing).
- Ograniczenie powierzchni narzędzi podatnej na nadużycia: sub‑agenci **nie** otrzymują domyślnie narzędzi sesji.
- Obsługa konfigurowalnej głębokości zagnieżdżenia dla wzorców orkiestratora.

Each sub-agent has its **own** context and token usage. W przypadku ciężkich lub powtarzalnych
zadań ustaw tańszy model dla sub‑agentów, a głównego agenta pozostaw na modelu wyższej jakości.
Konfiguracja per agent: `agents.list[].subagents.model`

## #> [limit] [tools]\`

Jawny parametr `model` w wywołaniu `sessions_spawn`

- Uruchamia przebieg sub‑agenta (`deliver: false`, globalny kanał: `subagent`)
- Następnie wykonuje krok ogłoszenia i publikuje odpowiedź ogłoszenia na kanale czatu żądającego
- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Konfiguracja per agent: `agents.list[].subagents.thinking`

Parametry narzędzia:

- `task`
- `etykieta`
- Utwórz pod innym identyfikatorem agenta (musi być dozwolone)
- <Note>
Nieprawidłowe wartości modelu są po cichu pomijane — sub-agent uruchamia się na następnym prawidłowym domyślnym modelu z ostrzeżeniem w wyniku narzędzia.
</Note>
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Automatically aborts the sub-agent run after the specified time
- `"delete"` \\

Lista dozwolonych:

- `agents.list[].subagents.allowAgents`: lista identyfikatorów agentów, które mogą być wskazane przez `agentId` (`["*"]` aby zezwolić na dowolne). Domyślnie: tylko agent żądający.

Wykrywanie:

- <Tip>
Użyj narzędzia `agents_list`, aby sprawdzić, które identyfikatory agentów są obecnie dozwolone dla `sessions_spawn`.
</Tip>

Automatyczna archiwizacja

- Sesje sub-agentów są automatycznie archiwizowane po konfigurowalnym czasie:
- <Note>
Archiwizacja zmienia nazwę transkryptu na \`\*.deleted.<timestamp> (ten sam folder).
- `"delete"` archiwizuje natychmiast po ogłoszeniu
- - **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- Automatyczne archiwizowanie dotyczy w równym stopniu sesji o głębokości 1 i 2.

## Tworzenie sub-agentów między agentami

- **No nested spawning:** Sub-agents cannot spawn their own sub-agents. Możesz włączyć jeden poziom zagnieżdżenia, ustawiając `maxSpawnDepth: 2`, co umożliwia zastosowanie **wzorca orkiestratora**: main → pod-agent orkiestrator → pod-pod-agenci roboczy.

### Jak to działa

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Poziomy głębokości

| Głębokość | Struktura klucza sesji                                                                                                                                                                                                                                                                                                                                                                                                       | Rola                                                                        | Może tworzyć potomne?            |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- | -------------------------------- |
| 0         | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | _(agent wywołujący)_                                     | Zawsze                           |
| 1         | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | Pod-agent (orkiestrator, gdy dozwolona jest głębokość 2) | Tylko jeśli `maxSpawnDepth >= 2` |
| 2         | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Zarządzanie subagentami (`/subagents`)                   | Nigdy                            |

### Łańcuch ogłoszeń

Wyniki przepływają w górę łańcucha:

1. Worker poziomu 2 kończy → ogłasza do swojego rodzica (orkiestratora poziomu 1)
2. Orkiestrator poziomu 1 otrzymuje ogłoszenie, syntetyzuje wyniki, kończy → ogłasza do main
3. Agent główny otrzymuje ogłoszenie i przekazuje je użytkownikowi

Każdy poziom widzi ogłoszenia wyłącznie od swoich bezpośrednich potomków.

### Polityka narzędzi

- **Poziom 1 (orkiestrator, gdy `maxSpawnDepth >= 2`)**: Otrzymuje `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, aby móc zarządzać swoimi potomkami. Inne narzędzia sesji/systemowe pozostają niedozwolone.
- **Poziom 1 (liść, gdy `maxSpawnDepth == 1`)**: Brak narzędzi sesji (obecne domyślne zachowanie).
- **Poziom 2 (worker liść)**: Brak narzędzi sesji — `sessions_spawn` jest zawsze niedozwolone na poziomie 2. Nie może tworzyć dalszych potomków.

### Możesz dodatkowo ograniczyć narzędzia subagentów:

Każda sesja agenta (na dowolnym poziomie głębokości) może mieć jednocześnie maksymalnie `maxChildrenPerAgent` (domyślnie: 5) aktywnych potomków. Zapobiega to niekontrolowanemu rozrostowi liczby potomków z jednego orkiestratora.

### Kaskadowe zatrzymanie

Zatrzymanie orkiestratora poziomu 1 automatycznie zatrzymuje wszystkie jego potomne procesy poziomu 2:

- `/stop` na głównym czacie zatrzymuje wszystkich agentów poziomu 1 i kaskadowo ich potomków poziomu 2.
- `) on the dedicated `subagent\` queue lane.
- `/subagents kill all` zatrzymuje wszystkich pod-agentów dla żądającego i działa kaskadowo.

## Uwierzytelnianie

Uwierzytelnianie subagenta jest rozstrzygane według **identyfikatora agenta**, a nie typu sesji:

- Klucz sesji pod-agenta to `agent:<agentId>:subagent:<uuid>`.
- Magazyn uwierzytelniania jest ładowany z `agentDir` docelowego agenta
- Profile uwierzytelniania głównego agenta są scalane jako **zapasowe** (profile agenta mają pierwszeństwo w przypadku konfliktów)

Uwaga: scalanie jest addytywne, więc profile main są zawsze dostępne jako zapasowe.
Fully isolated auth per sub-agent is not currently supported.

## Ogłoszenie

Pod-agenci raportują wyniki poprzez krok ogłoszenia:

- Krok ogłoszenia jest wykonywany w sesji pod-agenta (nie w sesji żądającego).
- Jeśli pod-agent odpowie dokładnie `ANNOUNCE_SKIP`, nic nie zostanie opublikowane.
- When the sub-agent finishes, it announces its findings back to the requester chat.
- Odpowiedzi ogłoszeń zachowują trasowanie wątków/tematów, gdy jest dostępne (wątki Slack, tematy Telegram, wątki Matrix).
- Wiadomości ogłoszeń są normalizowane do stabilnego szablonu:
  - `Status:` wyprowadzony z wyniku uruchomienia (`success`, `error`, `timeout` lub `unknown`).
  - `Result:` treść podsumowania z kroku ogłoszenia (lub `(not available)` jeśli brak).
  - `Notes:` szczegóły błędu i inne przydatne informacje kontekstowe.
- `Status` nie jest wywnioskowany z wyniku modelu; pochodzi z sygnałów rezultatu wykonania.

Ładunki ogłoszeniowe zawierają na końcu wiersz ze statystykami (nawet gdy są opakowane):

- Czas działania (np. `runtime 5m12s`)
- Zużycie tokenów (wejście/wyjście/razem)
- Szacowanym kosztem (gdy ceny modeli są skonfigurowane przez `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` oraz ścieżka do transkrypcji (aby główny agent mógł pobrać historię przez `sessions_history` lub sprawdzić plik na dysku)

## Dostosowywanie narzędzi subagentów

Aby ograniczyć subagentów **wyłącznie** do określonych narzędzi:

- Jawny parametr `thinking` w wywołaniu `sessions_spawn`
- `sessions_history`
- `sessions_send`
- Narzędzie `sessions_spawn`

Gdy `maxSpawnDepth >= 2`, pod-agenci orkiestratora na poziomie 1 dodatkowo otrzymują `sessions_spawn`, `subagents`, `sessions_list` oraz `sessions_history`, aby mogli zarządzać swoimi dziećmi.

Nadpisz przez konfigurację:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

## Współbieżność

Pod-agenci korzystają z dedykowanej wewnętrznej kolejki w procesie:

- :subagent:
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## Zatrzymaj działającego subagenta

- The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.
- `/subagents stop <id>`

## Ograniczenia

- Ogłoszenie pod-agenta jest **best-effort**. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
- Kontekst pod-agenta wstrzykuje tylko `AGENTS.md` + `TOOLS.md` (bez `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ani `BOOTSTRAP.md`).
- Maksymalna głębokość zagnieżdżenia to 5 (zakres `maxSpawnDepth`: 1–5). Poziom 2 jest zalecany w większości zastosowań.
- `maxChildrenPerAgent` ogranicza liczbę aktywnych dzieci na sesję (domyślnie: 5, zakres: 1–20).

