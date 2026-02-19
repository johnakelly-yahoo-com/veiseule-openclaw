---
summary: "Zainstaluj OpenClaw deklaratywnie za pomocą Nix"
read_when:
  - Chcesz instalacji odtwarzalnych i z możliwością cofania
  - Już korzystasz z Nix/NixOS/Home Manager
  - Chcesz, aby wszystko było przypięte i zarządzane deklaratywnie
title: "Nix"
---

# Instalacja Nix

Zalecanym sposobem uruchamiania OpenClaw z Nix jest **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — moduł Home Manager „z pełnym zestawem”.

## Szybki start

Wklej to do swojego agenta AI (Claude, Cursor itp.):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 Pełny przewodnik: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Repozytorium nix-openclaw jest źródłem prawdy dla instalacji Nix. Ta strona to jedynie szybki przegląd.

## Co otrzymujesz

- Gateway + aplikację na macOS + narzędzia (whisper, spotify, kamery) — wszystko przypięte
- Usługę launchd, która przetrwa restarty
- System wtyczek z konfiguracją deklaratywną
- Natychmiastowe cofanie zmian: `home-manager switch --rollback`

---

## Zachowanie środowiska uruchomieniowego w trybie Nix

Gdy ustawione jest `OPENCLAW_NIX_MODE=1` (automatycznie z nix-openclaw):

OpenClaw obsługuje **tryb Nix**, który czyni konfigurację deterministyczną i wyłącza przepływy auto‑instalacji.
Włącz go, eksportując:

```bash
OPENCLAW_NIX_MODE=1
```

W macOS aplikacja GUI nie dziedziczy automatycznie zmiennych środowiskowych powłoki. Możesz
również włączyć tryb Nix za pomocą defaults:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Ścieżki konfiguracji i stanu

OpenClaw czyta konfigurację JSON5 z `OPENCLAW_CONFIG_PATH` i przechowuje dane mutowalne w `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (domyślna kolejność pierwszeństwa: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (domyślnie: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (domyślnie: `$OPENCLAW_STATE_DIR/openclaw.json`)

Podczas uruchamiania w Nix ustaw je jawnie na lokalizacje zarządzane przez Nix, aby stan środowiska uruchomieniowego i konfiguracja
pozostały poza niemutowalnym store.

### Zachowanie środowiska uruchomieniowego w trybie Nix

- Przepływy auto‑instalacji i samomutacji są wyłączone
- Brakujące zależności zgłaszają komunikaty naprawcze specyficzne dla Nix
- Interfejs wyświetla baner trybu Nix tylko do odczytu, gdy jest obecny

## Uwaga dotycząca pakowania (macOS)

Proces pakowania macOS oczekuje stabilnego szablonu Info.plist w:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) kopiuje ten szablon do pakietu aplikacji i łata pola dynamiczne
(identyfikator pakietu, wersja/build, Git SHA, klucze Sparkle). Dzięki temu plist pozostaje deterministyczny dla
pakowania SwiftPM oraz buildów Nix (które nie polegają na pełnym łańcuchu narzędzi Xcode).

## Powiązane

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — pełny przewodnik konfiguracji
- [Wizard](/start/wizard) — konfiguracja CLI bez Nix
- [Docker](/install/docker) — konfiguracja konteneryzowana
