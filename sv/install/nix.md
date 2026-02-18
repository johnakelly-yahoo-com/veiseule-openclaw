---
title: "Nix"
---

# Nix-installation

Det rekommenderade sättet att köra OpenClaw med Nix är via **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — en Home Manager-modul med allt inkluderat.

## Snabbstart

Klistra in detta till din AI-agent (Claude, Cursor, etc.):

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

> **📦 Fullständig guide: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Den nix-openclaw repo är källan till sanningen för Nix installation. Denna sida är bara en snabb översikt.

## Vad du får

- Gateway + macOS-app + verktyg (whisper, spotify, kameror) — allt pinnat
- Launchd-tjänst som överlever omstarter
- Plugin-system med deklarativ konfig
- Omedelbar återställning: `home-manager switch --rollback`

---

## Körbeteende i Nix-läge

När `OPENCLAW_NIX_MODE=1` är satt (automatiskt med nix-openclaw):

OpenClaw stöder ett **Nix-läge** som gör konfigurationen deterministisk och inaktiverar automatiska installationsflöden.
Aktivera det genom att exportera:

```bash
OPENCLAW_NIX_MODE=1
```

På macOS ärver GUI-appen inte automatiskt shell env vars. Du kan
också aktivera Nix-läge via standard:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Konfig- och tillståndssökvägar

OpenClaw läser JSON5-konfig från `OPENCLAW_CONFIG_PATH` och lagrar föränderlig data i `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (default precedence: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (standard: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (standard: `$OPENCLAW_STATE_DIR/openclaw.json`)

När du kör under Nix ska dessa sättas explicit till Nix-hanterade platser så att körningstillstånd och konfig
hålls borta från den oföränderliga store:n.

### Körbeteende i Nix-läge

- Flöden för automatisk installation och självmutation är inaktiverade
- Saknade beroenden visar Nix-specifika åtgärdsmeddelanden
- UI visar en skrivskyddad Nix-lägesbanner när den finns

## Paketeringsnotering (macOS)

Paketeringsflödet för macOS förväntar sig en stabil Info.plist-mall på:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) kopierar denna mall till appbuntet och patchar dynamiska fält
(bunt ID, version/build, Git SHA, Sparkle nycklar). Detta håller plist deterministisk för SwiftPM
förpackningar och Nix bygger (som inte förlitar sig på en fullständig Xcode verktygskedja).

## Relaterat

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — fullständig konfigureringsguide
- [Guide](/start/wizard) — icke-Nix CLI-konfigurering
- [Docker](/install/docker) — containeriserad konfigurering
