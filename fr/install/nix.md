---
summary: "Installer OpenClaw de manière déclarative avec Nix"
read_when:
  - Vous voulez des installations reproductibles et avec retour en arriere
  - Vous utilisez deja Nix/NixOS/Home Manager
  - Vous voulez que tout soit fige et gere de maniere declarative
title: "Nix"
---

# Installation Nix

La methode recommandee pour executer OpenClaw avec Nix est via **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — un module Home Manager « tout inclus ».

## Demarrage rapide

Collez ceci dans votre agent IA (Claude, Cursor, etc.) :

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

> **📦 Guide complet : [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> Le depot nix-openclaw est la source de verite pour l’installation Nix. Cette page n’est qu’un apercu rapide.

## Ce que vous obtenez

- Gateway (passerelle) + application macOS + outils (whisper, spotify, cameras) — tous figes
- Service Launchd qui survit aux redemarrages
- Systeme de plugins avec configuration declarative
- Retour en arriere instantane : `home-manager switch --rollback`

---

## Comportement a l’execution en mode Nix

Lorsque `OPENCLAW_NIX_MODE=1` est defini (automatique avec nix-openclaw) :

OpenClaw prend en charge un **mode Nix** qui rend la configuration deterministe et desactive les flux d’auto-installation.
Activez-le en exportant :

```bash
OPENCLAW_NIX_MODE=1
```

Sur macOS, l’application GUI n’herite pas automatiquement des variables d’environnement du shell. Vous pouvez
egalement activer le mode Nix via defaults :

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### Chemins de configuration et d’etat

OpenClaw lit la configuration JSON5 depuis `OPENCLAW_CONFIG_PATH` et stocke les donnees mutables dans `OPENCLAW_STATE_DIR`.
Si nécessaire, vous pouvez également définir `OPENCLAW_HOME` pour contrôler le répertoire personnel de base utilisé pour la résolution interne des chemins.

- `OPENCLAW_HOME` (priorité par défaut : `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (par defaut : `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (par defaut : `$OPENCLAW_STATE_DIR/openclaw.json`)

Lors de l’execution sous Nix, definissez-les explicitement vers des emplacements geres par Nix afin que l’etat d’execution et la configuration
restent hors du store immuable.

### Comportement a l’execution en mode Nix

- Les flux d’auto-installation et d’auto-mutation sont desactives
- Les dependances manquantes affichent des messages de remediation specifiques a Nix
- L’interface affiche une bannière de mode Nix en lecture seule lorsqu’elle est presente

## Note de packaging (macOS)

Le flux de packaging macOS s’attend a un modele Info.plist stable a l’emplacement :

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) copie ce modele dans le bundle de l’application et corrige les champs dynamiques
(ID de bundle, version/build, Git SHA, cles Sparkle). Cela permet de conserver un plist deterministe pour le packaging SwiftPM
et les builds Nix (qui ne s’appuient pas sur une chaine d’outils Xcode complete).

## Associe

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — guide de configuration complet
- [Assistant](/start/wizard) — configuration CLI hors Nix
- [Docker](/install/docker) — configuration en conteneur

