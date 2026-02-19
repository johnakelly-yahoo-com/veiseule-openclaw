---
summary: "„Zaawansowana konfiguracja i przepływy pracy deweloperskiej dla OpenClaw”"
read_when:
  - Konfiguracja nowej maszyny
  - Chcesz „najświeższe i najlepsze” bez psucia własnej konfiguracji
title: "„Konfiguracja”"
---

# Konfiguracja

<Note>
Jeśli konfigurujesz po raz pierwszy, zacznij od [Pierwsze kroki](/start/getting-started).
Szczegóły kreatora znajdziesz w [Kreatorze wdrażania](/start/wizard).
</Note>

Ostatnia aktualizacja: 2026-01-01

## TL;DR

- **Dostosowanie poza repozytorium:** `~/.openclaw/workspace` (obszar roboczy) + `~/.openclaw/openclaw.json` (konfiguracja).
- **Stabilny przepływ pracy:** zainstaluj aplikację na macOS; pozwól jej uruchamiać dołączony Gateway.
- **Bleeding edge:** uruchom Gateway samodzielnie przez `pnpm gateway:watch`, a następnie pozwól aplikacji na macOS podłączyć się w trybie Lokalnym.

## Wymagania wstępne (ze źródła)

- Node `>=22`
- `pnpm`
- Docker (opcjonalnie; tylko dla konfiguracji kontenerowej/e2e — zob. [Docker](/install/docker))

## Strategia dostosowywania (aby aktualizacje nie bolały)

Jeśli chcesz „100% pod siebie” _i_ łatwe aktualizacje, trzymaj personalizację w:

- **Konfiguracja:** `~/.openclaw/openclaw.json` (JSON/JSON5‑ish)
- **Obszar roboczy:** `~/.openclaw/workspace` (skills, prompty, pamięci; najlepiej prywatne repozytorium git)

Bootstrap raz:

```bash
openclaw setup
```

Z poziomu tego repozytorium użyj lokalnego wejścia CLI:

```bash
openclaw setup
```

Jeśli nie masz jeszcze instalacji globalnej, uruchom przez `pnpm openclaw setup`.

## Uruchamianie Gateway z tego repozytorium

Po `pnpm build` możesz uruchomić spakowane CLI bezpośrednio:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## Stabilny przepływ pracy (najpierw aplikacja na macOS)

1. Zainstaluj i uruchom **OpenClaw.app** (pasek menu).
2. Przejdź checklistę wdrażania/uprawnień (monity TCC).
3. Upewnij się, że Gateway jest **Lokalny** i działa (aplikacja nim zarządza).
4. Połącz powierzchnie (przykład: WhatsApp):

```bash
openclaw channels login
```

5. Szybka weryfikacja:

```bash
openclaw health
```

Jeśli wdrażanie nie jest dostępne w Twojej wersji:

- Uruchom `openclaw setup`, potem `openclaw channels login`, a następnie uruchom Gateway ręcznie (`openclaw gateway`).

## Bleeding edge (Gateway w terminalu)

Cel: pracować nad Gateway w TypeScript, mieć hot reload i zachować podłączony interfejs aplikacji na macOS.

### 0. (Opcjonalnie) Uruchom także aplikację na macOS ze źródeł

Jeśli chcesz, aby aplikacja na macOS również była bleeding edge:

```bash
./scripts/restart-mac.sh
```

### 1. Uruchom deweloperski Gateway

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` uruchamia gateway w trybie watch i przeładowuje przy zmianach w TypeScript.

### 2. Wskaż aplikacji na macOS działający Gateway

W **OpenClaw.app**:

- Tryb połączenia: **Lokalny**
  Aplikacja podłączy się do działającego gateway na skonfigurowanym porcie.

### 3. Weryfikacja

- Status Gateway w aplikacji powinien brzmieć **„Using existing gateway …”**
- Albo przez CLI:

```bash
openclaw health
```

### Pistolety piekarnicze

- **Zły port:** Gateway WS domyślnie używa `ws://127.0.0.1:18789`; utrzymuj ten sam port w aplikacji i CLI.
- **Gdzie przechowywany jest stan:**
  - Poświadczenia: `~/.openclaw/credentials/`
  - Sesje: `~/.openclaw/agents/<agentId>/sessions/`
  - Logi: `/tmp/openclaw/`

## Mapa przechowywania poświadczeń

Użyj tego przy debugowaniu uwierzytelniania lub decydowaniu, co archiwizować:

- **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Token bota Telegram**: konfiguracja/zmienne środowiskowe lub `channels.telegram.tokenFile`
- **Token bota Discord**: konfiguracja/zmienne środowiskowe (plik z tokenem nie jest jeszcze obsługiwany)
- **Tokeny Slack**: konfiguracja/zmienne środowiskowe (`channels.slack.*`)
- **Listy dozwolonych parowań**: `~/.openclaw/credentials/<channel>-allowFrom.json`
- **Profile uwierzytelniania modeli**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **Import starszego OAuth**: `~/.openclaw/credentials/oauth.json`
  Więcej szczegółów: [Bezpieczeństwo](/gateway/security#credential-storage-map).

## Aktualizacje (bez demolowania konfiguracji)

- Trzymaj `~/.openclaw/workspace` i `~/.openclaw/` jako „Twoje rzeczy”; nie umieszczaj osobistych promptów/konfiguracji w repozytorium `openclaw`.
- Aktualizacja źródeł: `git pull` + `pnpm install` (gdy zmieni się lockfile) + dalej używaj `pnpm gateway:watch`.

## Linux (usługa użytkownika systemd)

Instalacje linuksowe używają usługi **user** systemd. Domyślnie systemd zatrzymuje usługi użytkownika po wylogowaniu/bezczynności, co zabija Gateway. Wdrażanie próbuje włączyć lingering za Ciebie (może poprosić o sudo). Jeśli nadal jest wyłączone, uruchom:

```bash
sudo loginctl enable-linger $USER
```

Dla serwerów „always‑on” lub wieloużytkownikowych rozważ usługę **systemową** zamiast użytkownika (bez potrzeby lingering). Zob. Zobacz notatki systemowe [Gateway runbook](/gateway).

## Powiązana dokumentacja

- [Gateway runbook](/gateway) (flagi, nadzór, porty)
- [Konfiguracja Gateway](/gateway/configuration) (schemat konfiguracji + przykłady)
- [Discord](/channels/discord) i [Telegram](/channels/telegram) (tagi odpowiedzi + ustawienia replyToMode)
- [Konfiguracja asystenta OpenClaw](/start/openclaw)
- [Aplikacja na macOS](/platforms/macos) (cykl życia gateway)

