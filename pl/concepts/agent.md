---
summary: "Środowisko uruchomieniowe agenta (osadzony pi-mono), kontrakt obszaru roboczego i bootstrap sesji"
read_when:
  - Zmiana środowiska uruchomieniowego agenta, bootstrapu obszaru roboczego lub zachowania sesji
title: "Środowisko uruchomieniowe agenta"
---

# Środowisko uruchomieniowe agenta 🤖

OpenClaw uruchamia pojedyncze, osadzone środowisko uruchomieniowe agenta wywiedzione z **pi-mono**.

## Obszar roboczy (wymagany)

OpenClaw używa pojedynczego katalogu obszaru roboczego agenta (`agents.defaults.workspace`) jako **jedyny** katalog roboczy agenta (`cwd`) dla narzędzi i kontekstu.

Zalecane: użyj `openclaw setup`, aby utworzyć `~/.openclaw/openclaw.json`, jeśli nie istnieje, oraz zainicjalizować pliki obszaru roboczego.

Pełny układ obszaru roboczego + przewodnik tworzenia kopii zapasowych: [Agent workspace](/concepts/agent-workspace)

Jeśli włączone jest `agents.defaults.sandbox`, sesje inne niż główna mogą to nadpisać,
korzystając z obszarów roboczych per sesja w `agents.defaults.sandbox.workspaceRoot` (zob.
[Konfiguracja Gateway](/gateway/configuration)).

## Pliki bootstrap (wstrzykiwane)

Wewnątrz `agents.defaults.workspace` OpenClaw oczekuje następujących plików edytowalnych przez użytkownika:

- `AGENTS.md` — instrukcje operacyjne + „pamięć”
- `SOUL.md` — persona, granice, ton
- `TOOLS.md` — notatki narzędzi utrzymywane przez użytkownika (np. `imsg`, `sag`, konwencje)
- `BOOTSTRAP.md` — jednorazowy rytuał pierwszego uruchomienia (usuwany po zakończeniu)
- `IDENTITY.md` — nazwa/wibe/emoji agenta
- `USER.md` — profil użytkownika + preferowana forma zwracania się

W pierwszej turze nowej sesji OpenClaw wstrzykuje zawartość tych plików bezpośrednio do kontekstu agenta.

Puste pliki są pomijane. Duże pliki są przycinane i skracane z markerem, aby prompty pozostały zwięzłe (pełną treść zobacz w pliku).

Jeśli plik nie istnieje, OpenClaw wstrzykuje pojedynczą linię markera „missing file” (a `openclaw setup` utworzy bezpieczny szablon domyślny).

`BOOTSTRAP.md` jest tworzony wyłącznie dla **zupełnie nowego obszaru roboczego** (brak innych plików bootstrap). Jeśli usuniesz go po ukończeniu rytuału, nie powinien być odtwarzany przy kolejnych restartach.

Aby całkowicie wyłączyć tworzenie plików bootstrap (dla wstępnie przygotowanych obszarów roboczych), ustaw:

```json5
{ agent: { skipBootstrap: true } }
```

## Wbudowane narzędzia

Narzędzia podstawowe (read/exec/edit/write oraz powiązane narzędzia systemowe) są zawsze dostępne,
z zastrzeżeniem polityki narzędzi. `apply_patch` jest opcjonalne i bramkowane przez
`tools.exec.applyPatch`. `TOOLS.md` **nie** kontroluje, które narzędzia istnieją; to
wskazówki dotyczące tego, jak _Ty_ chcesz, aby były używane.

## Umiejętności

OpenClaw ładuje skills z trzech lokalizacji (obszar roboczy wygrywa przy konflikcie nazw):

- Dołączone (dostarczane z instalacją)
- Zarządzane/lokalne: `~/.openclaw/skills`
- Obszar roboczy: `<workspace>/skills`

Skills mogą być bramkowane przez konfigurację/zmienne środowiskowe (zob. `skills` w [Konfiguracji Gateway](/gateway/configuration)).

## Integracja pi-mono

OpenClaw ponownie wykorzystuje fragmenty bazy kodu pi-mono (modele/narzędzia), ale **zarządzanie sesjami, wykrywanie i okablowanie narzędzi należą do OpenClaw**.

- Brak środowiska uruchomieniowego agenta pi-coding.
- Nie są konsultowane ustawienia `~/.pi/agent` ani `<workspace>/.pi`.

## Sesje

Transkrypty sesji są przechowywane jako JSONL w:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

Identyfikator sesji jest stabilny i wybierany przez OpenClaw.
Starsze foldery sesji Pi/Tau **nie** są odczytywane.

## Układ kierowniczy podczas przesyłania strumieniowego

Gdy tryb kolejki to `steer`, przychodzące wiadomości są wstrzykiwane do bieżącego uruchomienia.
Kolejka jest sprawdzana **po każdym wywołaniu narzędzia**; jeśli obecna jest wiadomość w kolejce,
pozostałe wywołania narzędzi z bieżącej wiadomości asystenta są pomijane (wyniki narzędzi z błędem
„Skipped due to queued user message.”), a następnie przed kolejną odpowiedzią asystenta wstrzykiwana
jest zakolejkowana wiadomość użytkownika.

Gdy tryb kolejki to `followup` lub `collect`, przychodzące wiadomości są wstrzymywane do
zakończenia bieżącej tury, po czym rozpoczyna się nowa tura agenta z zakolejkowanymi ładunkami. Zob. [Kolejka](/concepts/queue) — tryby oraz zachowanie debounce/limitów.

Strumieniowanie blokowe wysyła ukończone bloki asystenta natychmiast po ich zakończeniu; jest ono
**domyślnie wyłączone** (`agents.defaults.blockStreamingDefault: "off"`).
Dostosuj granicę za pomocą `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; domyślnie text_end).
Steruj miękkim dzieleniem bloków za pomocą `agents.defaults.blockStreamingChunk` (domyślnie
800–1200 znaków; preferuje podziały akapitów, następnie nowe linie; zdania na końcu).
Scalaj strumieniowane fragmenty za pomocą `agents.defaults.blockStreamingCoalesce`, aby ograniczyć
spam jednoliniowy (łączenie oparte na bezczynności przed wysłaniem). Kanały inne niż Telegram
wymagają jawnego `*.blockStreaming: true`, aby włączyć odpowiedzi blokowe.
Rozszerzone podsumowania narzędzi są emitowane przy starcie narzędzia (bez debounce); interfejs
Control UI strumieniuje wyjście narzędzi przez zdarzenia agenta, gdy są dostępne.
Więcej szczegółów: [Strumieniowanie + fragmentacja](/concepts/streaming).

## Odwołania do modeli

Odwołania do modeli w konfiguracji (na przykład `agents.defaults.model` i `agents.defaults.models`) są parsowane przez
podział według **pierwszego** `/`.

- Używaj `provider/model` podczas konfigurowania modeli.
- Jeśli identyfikator modelu sam w sobie zawiera `/` (styl OpenRouter), dołącz prefiks dostawcy (przykład: `openrouter/moonshotai/kimi-k2`).
- Jeśli pominiesz dostawcę, OpenClaw traktuje wejście jako alias lub model dla **domyślnego dostawcy** (działa tylko wtedy, gdy w identyfikatorze modelu nie ma `/`).

## Konfiguracja (minimalna)

Co najmniej ustaw:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (zdecydowanie zalecane)

---

_Następnie: [Czaty grupowe](/channels/group-messages)_ 🦞
