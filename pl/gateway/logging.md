---
title: "Logowanie"
---

# Logowanie

Aby zapoznać się z przeglądem dla użytkownika (CLI + UI sterowania + konfiguracja), zobacz [/logging](/logging).

OpenClaw ma dwie „powierzchnie” logowania:

- **Wyjście konsoli** (to, co widać w terminalu / UI debugowania).
- **Logi plikowe** (linie JSON) zapisywane przez logger gateway.

## Logger oparty na plikach

- Domyślny rotujący plik logów znajduje się w `/tmp/openclaw/` (jeden plik dziennie): `openclaw-YYYY-MM-DD.log`
  - Data używa lokalnej strefy czasowej hosta gateway.
- Ścieżkę pliku logów i poziom można skonfigurować przez `~/.openclaw/openclaw.json`:
  - `logging.file`
  - `logging.level`

Format pliku to jeden obiekt JSON na linię.

Karta Logi w UI sterowania śledzi ten plik przez gateway (`logs.tail`).
CLI może zrobić to samo:

```bash
openclaw logs --follow
```

**Szczegółowość a poziomy logów**

- **Logi plikowe** są kontrolowane wyłącznie przez `logging.level`.
- `--verbose` wpływa tylko na **szczegółowość konsoli** (oraz styl logów WS); **nie**
  podnosi poziomu logów plikowych.
- Aby zapisać w logach plikowych szczegóły dostępne tylko w trybie verbose, ustaw `logging.level` na `debug` lub
  `trace`.

## Przechwytywanie konsoli

CLI przechwytuje `console.log/info/warn/error/debug/trace` i zapisuje je do logów plikowych,
jednocześnie nadal wypisując je na stdout/stderr.

Szczegółowość konsoli można niezależnie dostroić za pomocą:

- `logging.consoleLevel` (domyślnie `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Redakcja podsumowań narzędzi

Szczegółowe podsumowania narzędzi (np. `🛠️ Exec: ...`) mogą maskować wrażliwe tokeny, zanim trafią do
strumienia konsoli. Dotyczy to **wyłącznie narzędzi** i nie modyfikuje logów plikowych.

- `logging.redactSensitive`: `off` | `tools` (domyślnie: `tools`)
- `logging.redactPatterns`: tablica ciągów regex (nadpisuje domyślne)
  - Użyj surowych ciągów regex (automatycznie `gi`), lub `/pattern/flags`, jeśli potrzebujesz niestandardowych flag.
  - Dopasowania są maskowane przez zachowanie pierwszych 6 + ostatnich 4 znaków (długość >= 18), w przeciwnym razie `***`.
  - Domyślne reguły obejmują typowe przypisania kluczy, flagi CLI, pola JSON, nagłówki bearer, bloki PEM oraz popularne prefiksy tokenów.

## Logi WebSocket gateway

Gateway wypisuje logi protokołu WebSocket w dwóch trybach:

- **Tryb normalny (bez `--verbose`)**: drukowane są tylko „interesujące” wyniki RPC:
  - błędy (`ok=false`)
  - wolne wywołania (domyślny próg: `>= 50ms`)
  - błędy parsowania
- **Tryb verbose (`--verbose`)**: wypisuje cały ruch żądań/odpowiedzi WS.

### Styl logów WS

`openclaw gateway` obsługuje przełącznik stylu per gateway:

- `--ws-log auto` (domyślnie): tryb normalny jest zoptymalizowany; tryb verbose używa wyjścia kompaktowego
- `--ws-log compact`: wyjście kompaktowe (sparowane żądanie/odpowiedź) w trybie verbose
- `--ws-log full`: pełne wyjście per ramka w trybie verbose
- `--compact`: alias dla `--ws-log compact`

Przykłady:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## Formatowanie konsoli (logowanie podsystemów)

Formater konsoli jest **świadomy TTY** i drukuje spójne, prefiksowane linie.
Loggery podsystemów utrzymują wyjście pogrupowane i czytelne.

Zachowanie:

- **Prefiksy podsystemów** na każdej linii (np. `[gateway]`, `[canvas]`, `[tailscale]`)
- **Kolory podsystemów** (stałe dla danego podsystemu) plus kolorowanie poziomów
- **Kolorowanie, gdy wyjście jest TTY lub środowisko wygląda jak bogaty terminal** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), z poszanowaniem `NO_COLOR`
- **Skrócone prefiksy podsystemów**: usuwa wiodące `gateway/` + `channels/`, zachowuje ostatnie 2 segmenty (np. `whatsapp/outbound`)
- **Pod-loggery według podsystemu** (automatyczny prefiks + pole strukturalne `{ subsystem }`)
- **`logRaw()`** dla wyjścia QR/UX (bez prefiksu, bez formatowania)
- **Style konsoli** (np. `pretty | compact | json`)
- **Poziom logów konsoli** oddzielny od poziomu logów plikowych (plik zachowuje pełną szczegółowość, gdy `logging.level` jest ustawione na `debug`/`trace`)
- **Treści wiadomości WhatsApp** są logowane na poziomie `debug` (użyj `--verbose`, aby je zobaczyć)

Pozwala to zachować stabilność istniejących logów plikowych, jednocześnie czyniąc interaktywne wyjście czytelnym.

