---
summary: "Plan: odizolować browser act:evaluate od kolejki Playwright przy użyciu CDP, z pełnymi terminami end-to-end i bezpieczniejszym rozwiązywaniem referencji"
owner: "openclaw"
status: "szkic"
last_updated: "2026-02-10"
title: "Refaktoryzacja Browser Evaluate CDP"
---

# Plan refaktoryzacji Browser Evaluate CDP

## Kontekst

`act:evaluate` wykonuje dostarczony przez użytkownika JavaScript na stronie. Obecnie działa za pośrednictwem Playwright
(`page.evaluate` lub `locator.evaluate`). Playwright serializuje polecenia CDP na stronę, więc
zawieszone lub długotrwałe evaluate może zablokować kolejkę poleceń strony i sprawić, że każda późniejsza akcja
na tej karcie będzie wyglądać na „zawieszoną”.

PR #13498 dodaje praktyczną siatkę bezpieczeństwa (ograniczone czasowo evaluate, propagację przerwania oraz odzyskiwanie w miarę możliwości). Ten dokument opisuje większą refaktoryzację, która sprawia, że `act:evaluate` jest z natury
odizolowane od Playwright, tak aby zawieszone evaluate nie mogło zablokować zwykłych operacji Playwright.

## Cele

- `act:evaluate` nie może trwale blokować późniejszych akcji przeglądarki na tej samej karcie.
- Limity czasu stanowią jedno źródło prawdy end-to-end, aby wywołujący mógł polegać na określonym budżecie.
- Przerwanie i limit czasu są traktowane w ten sam sposób w HTTP oraz w dyspozycji w procesie.
- Obsługiwane jest wskazywanie elementów dla evaluate bez wyłączania wszystkiego z Playwright.
- Zachowanie wstecznej kompatybilności dla istniejących wywołań i ładunków danych.

## Poza zakresem

- Zastąpienie wszystkich akcji przeglądarki (click, type, wait itp.) z implementacjami CDP.
- Usuń istniejącą siatkę bezpieczeństwa wprowadzoną w PR #13498 (pozostaje ona użytecznym mechanizmem zapasowym).
- Wprowadź nowe niebezpieczne możliwości wykraczające poza istniejącą bramkę `browser.evaluateEnabled`.
- Dodaj izolację procesu (proces/wątek roboczy) dla evaluate. Jeśli po tej refaktoryzacji nadal będziemy obserwować trudne do odzyskania
  stany zawieszenia, jest to pomysł na kolejne działania.

## Obecna architektura (dlaczego dochodzi do zawieszeń)

Na wysokim poziomie:

- Wywołujący wysyłają `act:evaluate` do usługi kontroli przeglądarki.
- Handler trasy wywołuje Playwright w celu wykonania kodu JavaScript.
- Playwright serializuje polecenia strony, więc evaluate, które nigdy się nie kończy, blokuje kolejkę.
- Zablokowana kolejka powoduje, że późniejsze operacje kliknięcia/wpisywania/oczekiwania na karcie mogą sprawiać wrażenie zawieszenia.

## Proponowana architektura

### 1. Propagacja limitu czasu

Wprowadź jedno pojęcie budżetu i wyprowadzaj z niego wszystkie pozostałe elementy:

- Wywołujący ustawia `timeoutMs` (lub termin w przyszłości).
- Zewnętrzny limit czasu żądania, logika handlera trasy oraz budżet wykonania wewnątrz strony
  korzystają z tego samego budżetu, z niewielkim zapasem tam, gdzie jest to potrzebne na narzut serializacji.
- Przerwanie jest propagowane jako `AbortSignal` wszędzie, aby anulowanie było spójne.

Kierunek implementacji:

- Dodaj niewielką funkcję pomocniczą (na przykład `createBudget({ timeoutMs, signal })`), która zwraca:
  - `signal`: powiązany AbortSignal
  - `deadlineAtMs`: bezwzględny termin
  - `remainingMs()`: pozostały budżet dla operacji podrzędnych
- Użyj tej funkcji pomocniczej w:
  - `src/browser/client-fetch.ts` (HTTP i dyspozycja w procesie)
  - `src/node-host/runner.ts` (ścieżka proxy)
  - implementacjach akcji przeglądarki (Playwright i CDP)

### 2. Oddzielny silnik Evaluate (ścieżka CDP)

Dodaj implementację evaluate opartą na CDP, która nie współdzieli kolejki poleceń na stronę używanej przez Playwright. Kluczową właściwością jest to, że transport evaluate to oddzielne połączenie WebSocket
oraz oddzielna sesja CDP dołączona do celu.

Kierunek implementacji:

- Nowy moduł, na przykład `src/browser/cdp-evaluate.ts`, który:
  - Łączy się ze skonfigurowanym punktem końcowym CDP (gniazdo na poziomie przeglądarki).
  - Używa `Target.attachToTarget({ targetId, flatten: true })`, aby uzyskać `sessionId`.
  - Uruchamia jedno z poniższych:
    - `Runtime.evaluate` dla evaluate na poziomie strony lub
    - `DOM.resolveNode` plus `Runtime.callFunctionOn` dla evaluate na poziomie elementu.
  - W przypadku przekroczenia limitu czasu lub przerwania:
    - Wysyła `Runtime.terminateExecution` w miarę możliwości dla danej sesji.
    - Zamyka WebSocket i zwraca czytelny błąd.

Uwagi:

- To nadal wykonuje JavaScript na stronie, więc przerwanie może powodować skutki uboczne. Zaletą
  jest to, że nie blokuje kolejki Playwright i można to anulować na
  poziomie warstwy transportowej poprzez zakończenie sesji CDP.

### 3. Historia ref (Targetowanie elementów bez pełnego przepisywania)

Najtrudniejszą częścią jest targetowanie elementów. CDP wymaga uchwytu DOM lub `backendDOMNodeId`, podczas gdy
dziś większość działań w przeglądarce korzysta z lokalizatorów Playwright opartych na refach ze snapshotów.

Rekomendowane podejście: zachować istniejące refy, ale dodać opcjonalny identyfikator rozwiązywalny przez CDP.

#### 3.1 Rozszerzenie przechowywanych informacji o ref

Rozszerz przechowywane metadane ref roli, aby opcjonalnie zawierały identyfikator CDP:

- Obecnie: `{ role, name, nth }`
- Propozycja: `{ role, name, nth, backendDOMNodeId?: number }`

Pozwala to zachować działanie wszystkich istniejących akcji opartych na Playwright oraz umożliwia CDP evaluate przyjęcie
tej samej wartości `ref`, gdy `backendDOMNodeId` jest dostępny.

#### 3.2 Wypełnianie backendDOMNodeId w momencie tworzenia snapshotu

Podczas tworzenia snapshotu roli:

1. Wygeneruj istniejącą mapę ref roli jak dotychczas (role, name, nth).
2. Pobierz drzewo AX przez CDP (`Accessibility.getFullAXTree`) i oblicz równoległą mapę
   `(role, name, nth) -> backendDOMNodeId`, używając tych samych zasad obsługi duplikatów.
3. Scal identyfikator z powrotem z przechowywanymi informacjami o ref dla bieżącej karty.

Jeśli mapowanie dla danego ref się nie powiedzie, pozostaw `backendDOMNodeId` jako undefined. Dzięki temu funkcja ma charakter
best-effort i jest bezpieczna do wdrożenia.

#### 3.3 Zachowanie evaluate z ref

W `act:evaluate`:

- Jeśli `ref` jest obecny i ma `backendDOMNodeId`, uruchom evaluate elementu przez CDP.
- Jeśli `ref` jest obecny, ale nie ma `backendDOMNodeId`, przejdź do ścieżki Playwright (z
  mechanizmem zabezpieczającym).

Opcjonalna furtka awaryjna:

- Rozszerz strukturę żądania tak, aby bezpośrednio akceptowała `backendDOMNodeId` dla zaawansowanych użytkowników (oraz
  do debugowania), pozostawiając `ref` jako główny interfejs.

### 4. Zachowaj ścieżkę odzyskiwania jako ostateczność

Nawet przy CDP evaluate istnieją inne sposoby na zablokowanie karty lub połączenia. Zachowaj
istniejące mechanizmy odzyskiwania (terminate execution + disconnect Playwright) jako ostateczność
dla:

- starszych wywołań (legacy callers)
- środowisk, w których podłączenie CDP jest zablokowane
- nieoczekiwanych przypadków brzegowych Playwright

## Plan wdrożenia (jedna iteracja)

### Elementy do dostarczenia

- Silnik evaluate oparty na CDP, który działa poza kolejką poleceń Playwright dla pojedynczej strony.
- Jeden, spójny budżet timeout/abort end-to-end używany konsekwentnie przez wywołujących i handlery.
- Metadane ref, które mogą opcjonalnie przenosić `backendDOMNodeId` dla evaluate elementu.
- `act:evaluate` preferuje silnik CDP, gdy to możliwe, a w przeciwnym razie przechodzi na Playwright.
- Testy potwierdzające, że zablokowane evaluate nie blokuje kolejnych akcji.
- Logi/metryki, które uwidaczniają błędy i mechanizmy fallback.

### Lista kontrolna implementacji

1. Dodaj współdzielony pomocnik „budget”, który połączy `timeoutMs` + nadrzędny `AbortSignal` w:
   - pojedynczy `AbortSignal`
   - bezwzględny deadline
   - pomocnika `remainingMs()` dla operacji podrzędnych
2. Zaktualizuj wszystkie ścieżki wywołań, aby korzystały z tego pomocnika, tak by `timeoutMs` wszędzie oznaczał to samo:
   - `src/browser/client-fetch.ts` (HTTP oraz wywołania w procesie)
   - `src/node-host/runner.ts` (ścieżka proxy node)
   - Wrappery CLI wywołujące `/act` (dodaj `--timeout-ms` do `browser evaluate`)
3. Zaimplementuj `src/browser/cdp-evaluate.ts`:
   - połącz się z gniazdem CDP na poziomie przeglądarki
   - `Target.attachToTarget`, aby uzyskać `sessionId`
   - uruchom `Runtime.evaluate` dla evaluate strony
   - uruchom `DOM.resolveNode` + `Runtime.callFunctionOn` dla evaluate elementu
   - przy przekroczeniu czasu/anulowaniu: best-effort `Runtime.terminateExecution`, a następnie zamknij gniazdo
4. Rozszerz przechowywane metadane referencji roli, aby opcjonalnie zawierały `backendDOMNodeId`:
   - zachowaj istniejące zachowanie `{ role, name, nth }` dla akcji Playwright
   - dodaj `backendDOMNodeId?: number` do targetowania elementów przez CDP
5. Wypełnij `backendDOMNodeId` podczas tworzenia snapshotu (best-effort):
   - pobierz drzewo AX przez CDP (`Accessibility.getFullAXTree`)
   - oblicz mapowanie `(role, name, nth) -> backendDOMNodeId` i scal je z przechowywaną mapą referencji
   - jeśli mapowanie jest niejednoznaczne lub brakujące, pozostaw id jako undefined
6. Zaktualizuj routing `act:evaluate`:
   - jeśli brak `ref`: zawsze użyj evaluate przez CDP
   - jeśli `ref` rozwiązuje się do `backendDOMNodeId`: użyj evaluate elementu przez CDP
   - w przeciwnym razie: zastosuj fallback do evaluate w Playwright (nadal ograniczone czasowo i możliwe do anulowania)
7. Zachowaj istniejącą ścieżkę odzyskiwania „last resort” jako fallback, a nie domyślną ścieżkę.
8. Dodaj testy:
   - zablokowane evaluate przekracza limit czasu w ramach budżetu, a kolejne kliknięcie/pisanie kończy się sukcesem
   - anulowanie przerywa evaluate (rozłączenie klienta lub timeout) i odblokowuje kolejne akcje
   - błędy mapowania powodują czysty fallback do Playwright
9. Dodaj obserwowalność:
   - czas trwania evaluate i liczniki timeoutów
   - użycie terminateExecution
   - wskaźnik fallbacków (CDP -> Playwright) oraz ich powody

### Kryteria akceptacji

- Celowo zawieszone `act:evaluate` zwraca wynik w ramach budżetu wywołującego i nie blokuje
  tabu dla kolejnych akcji.
- `timeoutMs` zachowuje się spójnie w CLI, narzędziu agenta, proxy node oraz wywołaniach w procesie.
- Jeśli `ref` można odwzorować na `backendDOMNodeId`, evaluate elementu używa CDP; w przeciwnym razie ścieżka zapasowa nadal jest ograniczona i możliwa do odzyskania.

## Plan testów

- Testy jednostkowe:
  - Logika dopasowania `(role, name, nth)` między referencjami ról a węzłami drzewa AX.
  - Zachowanie pomocnika budżetu (headroom, obliczanie pozostałego czasu).
- Testy integracyjne:
  - Timeout CDP evaluate zwraca wynik w ramach budżetu i nie blokuje następnej akcji.
  - Abort anuluje evaluate i wywołuje próbę zakończenia (best-effort).
- Testy kontraktowe:
  - Upewnij się, że `BrowserActRequest` i `BrowserActResponse` pozostają kompatybilne.

## Ryzyka i działania zapobiegawcze

- Mapowanie jest niedoskonałe:
  - Działanie zapobiegawcze: mapowanie best-effort, fallback do Playwright evaluate oraz dodanie narzędzi debugujących.
- `Runtime.terminateExecution` ma skutki uboczne:
  - Działanie zapobiegawcze: używać tylko przy timeout/abort i dokumentować to zachowanie w błędach.
- Dodatkowy narzut:
  - Działanie zapobiegawcze: pobierać drzewo AX tylko gdy wymagane są snapshoty, buforować per target i utrzymywać krótkotrwałą sesję CDP.
- Ograniczenia przekaźnika rozszerzenia:
  - Działanie zapobiegawcze: używać API dołączania na poziomie przeglądarki, gdy gniazda per strona nie są dostępne, oraz zachować obecną ścieżkę Playwright jako fallback.

## Otwarte pytania

- Czy nowy silnik powinien być konfigurowalny jako `playwright`, `cdp` czy `auto`?
- Czy chcemy udostępnić nowy format „nodeRef” dla zaawansowanych użytkowników, czy pozostawić tylko `ref`?
- W jaki sposób snapshoty ramek i snapshoty ograniczone selektorem powinny uczestniczyć w mapowaniu AX?
