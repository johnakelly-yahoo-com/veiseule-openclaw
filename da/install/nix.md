---
title: "Nix"
---

# Nix-installation

Den anbefalede måde at køre OpenClaw med Nix er via **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — et Home Manager-modul med alt inkluderet.

## Hurtig start

Indsæt dette til din AI-agent (Claude, Cursor osv.):

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

> **📦 Fuld guide: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Den nix-openclaw repo er kilden til sandheden for Nix installation. Denne side er blot et hurtigt overblik.

## Hvad du får

- Gateway + macOS-app + værktøjer (whisper, spotify, kameraer) — alt fastlåst
- Launchd-tjeneste, der overlever genstarter
- Pluginsystem med deklarativ konfiguration
- Øjeblikkelig rollback: `home-manager switch --rollback`

---

## Runtime-adfærd i Nix-tilstand

Når `OPENCLAW_NIX_MODE=1` er sat (automatisk med nix-openclaw):

OpenClaw understøtter en \*\* Nix-tilstand \*\*, der gør konfigurationen deterministisk og deaktiverer auto-installationsstrømme.
Aktiver det ved at eksportere:

```bash
OPENCLAW_NIX_MODE=1
```

På macOS er GUI app ikke automatisk arve shell env vars. Du kan
også aktivere Nix tilstand via standardværdier:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Konfigurations- og tilstandsstier

OpenClaw læser JSON5-konfiguration fra `OPENCLAW_CONFIG_PATH` og gemmer mutable data i `OPENCLAW_STATE_DIR`.
When needed, you can also set `OPENCLAW_HOME` to control the base home directory used for internal path resolution.

- `OPENCLAW_HOME` (standard prioritet: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (standard: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (standard: `$OPENCLAW_STATE_DIR/openclaw.json`)

Når du kører under Nix, skal disse sættes eksplicit til Nix-administrerede placeringer, så runtime-tilstand og konfiguration
holdes ude af den uforanderlige store.

### Runtime-adfærd i Nix-tilstand

- Auto-installation og selv-modificerende flows er deaktiveret
- Manglende afhængigheder viser Nix-specifikke løsningsbeskeder
- UI’et viser et skrivebeskyttet Nix-tilstandsbanner, når det er til stede

## Pakkeringsnote (macOS)

Pakkeringsflowet for macOS forventer en stabil Info.plist-skabelon på:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) kopierer denne skabelon til apppakken og patches dynamiske felter
(bundle ID, version/build, Git SHA, Sparkle nøgler). Dette holder plist deterministisk for SwiftPM
emballage og Nix bygger (som ikke er afhængig af en fuld Xcode værktøjskæde).

## Relateret

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — fuld opsætningsguide
- [Wizard](/start/wizard) — CLI-opsætning uden Nix
- [Docker](/install/docker) — containeriseret opsætning
