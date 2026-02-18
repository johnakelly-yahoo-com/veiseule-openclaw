---
title: "Zarządzanie sesjami Deep Dive"
---

# Zarządzanie sesjami i kompaktowanie (dogłębna analiza)

Ten dokument wyjaśnia, jak OpenClaw zarządza sesjami od początku do końca:

- **Routing sesji** (jak wiadomości przychodzące mapują się na `sessionKey`)
- **Magazyn sesji** (`sessions.json`) i co śledzi
- **Utrwalanie transkryptów** (`*.jsonl`) oraz ich struktura
- **Higiena transkryptów** (poprawki specyficzne dla dostawcy przed uruchomieniami)
- **Limity kontekstu** (okno kontekstu vs śledzone tokeny)
- **Kompaktowanie** (ręczne + automatyczne) oraz miejsca podpięcia prac przed kompaktowaniem
- **Ciche porządkowanie** (np. zapisy pamięci, które nie powinny generować widocznego dla użytkownika wyjścia)

Jeśli najpierw chcesz zapoznać się z widokiem wysokopoziomowym, zacznij od:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Źródło prawdy: Gateway

OpenClaw jest zaprojektowany wokół pojedynczego **procesu Gateway**, który jest właścicielem stanu sesji.

- Interfejsy użytkownika (aplikacja na macOS, webowy Control UI, TUI) powinny odpytywać Gateway o listy sesji i liczniki tokenów.
- W trybie zdalnym pliki sesji znajdują się na hoście zdalnym; „sprawdzanie lokalnych plików na Macu” nie odzwierciedla tego, czego używa Gateway.

---

## Dwie warstwy trwałości

OpenClaw utrwala sesje w dwóch warstwach:

1. **Magazyn sesji (`sessions.json`)**
   - Mapa klucz/wartość: `sessionKey -> SessionEntry`
   - Mały, mutowalny, bezpieczny do edycji (lub usuwania wpisów)
   - Śledzi metadane sesji (bieżący identyfikator sesji, ostatnią aktywność, przełączniki, liczniki tokenów itp.)

2. **Transkrypt (`<sessionId>.jsonl`)**
   - Transkrypt typu append-only ze strukturą drzewa (wpisy mają `id` + `parentId`)
   - Przechowuje faktyczną rozmowę + wywołania narzędzi + podsumowania kompaktowania
   - Używany do odbudowy kontekstu modelu dla kolejnych tur

---

## Lokalizacje na dysku

Na agenta, na hoście Gateway:

- Magazyn: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Transkrypty: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Sesje tematów Telegrama: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw rozwiązuje je poprzez `src/config/sessions.ts`.

---

## Klucze sesji (`sessionKey`)

`sessionKey` identyfikuje _do którego koszyka rozmów_ należysz (routing + izolacja).

Typowe wzorce:

- Główna/bezpośrednia rozmowa (na agenta): `agent:<agentId>:<mainKey>` (domyślnie `main`)
- Grupa: `agent:<agentId>:<channel>:group:<id>`
- Pokój/kanał (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` lub `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (o ile nie nadpisano)

Kanoniczne zasady są udokumentowane w [/concepts/session](/concepts/session).

---

## Identyfikatory sesji (`sessionId`)

Każdy `sessionKey` wskazuje na bieżący `sessionId` (plik transkryptu, który kontynuuje rozmowę).

Zasady praktyczne:

- **Reset** (`/new`, `/reset`) tworzy nowy `sessionId` dla tego `sessionKey`.
- **Reset dzienny** (domyślnie 4:00 czasu lokalnego na hoście gateway) tworzy nowy `sessionId` przy następnej wiadomości po granicy resetu.
- **Wygaśnięcie bezczynności** (`session.reset.idleMinutes` lub legacy `session.idleMinutes`) tworzy nowy `sessionId`, gdy wiadomość nadejdzie po oknie bezczynności. Gdy skonfigurowane są oba (dzienny + bezczynność), wygrywa to, które wygaśnie pierwsze.

Szczegół implementacyjny: decyzja zapada w `initSessionState()` w `src/auto-reply/reply/session.ts`.

---

## Schemat magazynu sesji (`sessions.json`)

Typem wartości magazynu jest `SessionEntry` w `src/config/sessions.ts`.

Kluczowe pola (lista niepełna):

- `sessionId`: bieżący identyfikator transkryptu (nazwa pliku jest od niego wyprowadzana, chyba że ustawiono `sessionFile`)
- `updatedAt`: znacznik czasu ostatniej aktywności
- `sessionFile`: opcjonalne jawne nadpisanie ścieżki transkryptu
- `chatType`: `direct | group | room` (pomaga interfejsom UI i polityce wysyłki)
- `provider`, `subject`, `room`, `space`, `displayName`: metadane do etykietowania grup/kanałów
- Przełączniki:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (nadpisanie per sesję)
- Wybór modelu:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Liczniki tokenów (best-effort / zależne od dostawcy):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: jak często auto-kompaktowanie zakończyło się dla tego klucza sesji
- `memoryFlushAt`: znacznik czasu ostatniego opróżnienia pamięci przed kompaktowaniem
- `memoryFlushCompactionCount`: liczba kompaktowań w momencie ostatniego uruchomienia opróżnienia

Magazyn jest bezpieczny do edycji, ale Gateway jest autorytetem: może przepisywać lub rehydratować wpisy w trakcie działania sesji.

---

## Struktura transkryptu (`*.jsonl`)

Transkrypty są zarządzane przez `@mariozechner/pi-coding-agent` w `SessionManager`.

Plik jest w formacie JSONL:

- Pierwsza linia: nagłówek sesji (`type: "session"`, zawiera `id`, `cwd`, `timestamp`, opcjonalnie `parentSession`)
- Następnie: wpisy sesji z `id` + `parentId` (drzewo)

Istotne typy wpisów:

- `message`: wiadomości użytkownik/asystent/toolResult
- `custom_message`: wiadomości wstrzykiwane przez rozszerzenia, które _wchodzą_ do kontekstu modelu (mogą być ukryte w UI)
- `custom`: stan rozszerzenia, który _nie wchodzi_ do kontekstu modelu
- `compaction`: utrwalone podsumowanie kompaktowania z `firstKeptEntryId` i `tokensBefore`
- `branch_summary`: utrwalone podsumowanie przy nawigacji po gałęzi drzewa

OpenClaw celowo **nie** „naprawia” transkryptów; Gateway używa `SessionManager` do ich odczytu/zapisu.

---

## Okna kontekstu vs śledzone tokeny

Istotne są dwie różne koncepcje:

1. **Okno kontekstu modelu**: twardy limit per model (tokeny widoczne dla modelu)
2. **Liczniki magazynu sesji**: statystyki kroczące zapisywane w `sessions.json` (używane przez /status i dashboardy)

Jeśli dostrajasz limity:

- Okno kontekstu pochodzi z katalogu modeli (i może być nadpisane przez konfigurację).
- `contextTokens` w magazynie to wartość szacunkowa/raportowa w czasie działania; nie traktuj jej jako ścisłej gwarancji.

Więcej informacji: [/token-use](/reference/token-use).

---

## Kompaktowanie: czym jest

Kompaktowanie streszcza starszą część rozmowy do utrwalonego wpisu `compaction` w transkrypcie i zachowuje nienaruszone nowsze wiadomości.

Po kompaktowaniu kolejne tury widzą:

- Podsumowanie kompaktowania
- Wiadomości po `firstKeptEntryId`

Kompaktowanie jest **trwałe** (w przeciwieństwie do przycinania sesji). Zobacz [/concepts/session-pruning](/concepts/session-pruning).

---

## Kiedy zachodzi auto-kompaktowanie (runtime Pi)

W osadzonym agencie Pi auto-kompaktowanie uruchamia się w dwóch przypadkach:

1. **Odzyskiwanie po przepełnieniu**: model zwraca błąd przepełnienia kontekstu → kompaktowanie → ponowienie.
2. **Utrzymanie progu**: po udanej turze, gdy:

`contextTokens > contextWindow - reserveTokens`

Gdzie:

- `contextWindow` to okno kontekstu modelu
- `reserveTokens` to zapas zarezerwowany na prompt + następne wyjście modelu

Są to semantyki runtime Pi (OpenClaw konsumuje zdarzenia, ale Pi decyduje, kiedy kompaktować).

---

## Ustawienia kompaktowania (`reserveTokens`, `keepRecentTokens`)

Ustawienia kompaktowania Pi znajdują się w ustawieniach Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw dodatkowo egzekwuje próg bezpieczeństwa dla uruchomień osadzonych:

- Jeśli `compaction.reserveTokens < reserveTokensFloor`, OpenClaw go podnosi.
- Domyślny próg minimalny to `20000` tokenów.
- Ustaw `agents.defaults.compaction.reserveTokensFloor: 0`, aby wyłączyć próg minimalny.
- Jeśli jest już wyższy, OpenClaw pozostawia go bez zmian.

Dlaczego: pozostawić wystarczający zapas na wieloturowe „porządkowanie” (np. zapisy pamięci), zanim kompaktowanie stanie się nieuniknione.

Implementacja: `ensurePiCompactionReserveTokens()` w `src/agents/pi-settings.ts`
(wywoływane z `src/agents/pi-embedded-runner.ts`).

---

## Powierzchnie widoczne dla użytkownika

Możesz obserwować kompaktowanie i stan sesji poprzez:

- `/status` (w dowolnej sesji czatu)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Tryb szczegółowy: `🧹 Auto-compaction complete` + licznik kompaktowań

---

## Ciche gospodarstwo domowe (`NO_REPLY`)

OpenClaw obsługuje „ciche” tury dla zadań w tle, w których użytkownik nie powinien widzieć wyjścia pośredniego.

Konwencja:

- Asystent rozpoczyna swoje wyjście od `NO_REPLY`, aby wskazać „nie dostarczać odpowiedzi użytkownikowi”.
- OpenClaw usuwa/tłumi to w warstwie dostarczania.

Od `2026.1.10` OpenClaw tłumi także **strumieniowanie szkiców/wskaźników pisania**, gdy częściowy fragment zaczyna się od `NO_REPLY`, dzięki czemu ciche operacje nie ujawniają częściowego wyjścia w trakcie tury.

---

## „Opróżnianie pamięci” przed kompaktowaniem (zaimplementowane)

Cel: zanim dojdzie do auto-kompaktowania, uruchomić cichą, agentową turę, która zapisze trwały
stan na dysk (np. `memory/YYYY-MM-DD.md` w obszarze roboczym agenta), tak aby kompaktowanie nie mogło
wymazać krytycznego kontekstu.

OpenClaw stosuje podejście **opróżnienia przed progiem**:

1. Monitoruj użycie kontekstu sesji.
2. Gdy przekroczy „miękki próg” (poniżej progu kompaktowania Pi), uruchom ciche
   polecenie „zapisz pamięć teraz” do agenta.
3. Użyj `NO_REPLY`, aby użytkownik nic nie zobaczył.

Konfiguracja (`agents.defaults.compaction.memoryFlush`):

- `enabled` (domyślnie: `true`)
- `softThresholdTokens` (domyślnie: `4000`)
- `prompt` (wiadomość użytkownika dla tury opróżniania)
- `systemPrompt` (dodatkowy prompt systemowy dołączany do tury opróżniania)

Uwagi:

- Domyślne prompty (użytkownika/systemowy) zawierają wskazówkę `NO_REPLY` do tłumienia dostarczania.
- Opróżnienie uruchamia się raz na cykl kompaktowania (śledzone w `sessions.json`).
- Opróżnienie działa tylko dla osadzonych sesji Pi (backendy CLI je pomijają).
- Opróżnienie jest pomijane, gdy obszar roboczy sesji jest tylko do odczytu (`workspaceAccess: "ro"` lub `"none"`).
- Zobacz [Memory](/concepts/memory), aby poznać układ plików obszaru roboczego i wzorce zapisu.

Pi udostępnia także hak `session_before_compact` w API rozszerzeń, jednak logika opróżniania OpenClaw znajduje się dziś po stronie Gateway.

---

## Lista kontrolna rozwiązywania problemów

- Zły klucz sesji? Zacznij od [/concepts/session](/concepts/session) i potwierdź `sessionKey` w `/status`.
- Niezgodność magazyn vs transkrypt? Potwierdź host Gateway i ścieżkę magazynu z `openclaw status`.
- Nadmierne kompaktowanie? Sprawdź:
  - okno kontekstu modelu (zbyt małe)
  - ustawienia kompaktowania (`reserveTokens` zbyt wysokie względem okna modelu może powodować wcześniejsze kompaktowanie)
  - nadmiar tool-result: włącz/dostrój przycinanie sesji
- Ciche skręty przeciekają? Potwierdź, że odpowiedź zaczyna się od `NO_REPLY` (dokładny token) i że używasz wersji zawierającej poprawkę tłumienia strumieniowania.

