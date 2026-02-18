---
title: "Fonctionnement interne de l’installateur"
---

# Fonctionnement interne de l’installateur

OpenClaw fournit trois scripts d’installation, servis depuis `openclaw.ai`.

| Script                             | Plateforme             | Ce qu’il fait                                                                                 |
| ---------------------------------- | ---------------------- | --------------------------------------------------------------------------------------------- |
| [`install.sh`](#installsh)         | macOS / Linux / WSL    | Installe Node si nécessaire, installe OpenClaw via npm (par défaut) ou git, et peut lancer l’onboarding. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL    | Installe Node + OpenClaw dans un préfixe local (`~/.openclaw`). Aucun accès root requis.     |
| [`install.ps1`](#installps1)       | Windows (PowerShell)   | Installe Node si nécessaire, installe OpenClaw via npm (par défaut) ou git, et peut lancer l’onboarding. |

## Commandes rapides

<Tabs>
  <Tab title="install.sh">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```

    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --help
    ```
  </Tab>

  <Tab title="install-cli.sh">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
    ```

    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --help
    ```
  </Tab>

  <Tab title="install.ps1">
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```

    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag beta -NoOnboard -DryRun
    ```
  </Tab>
</Tabs>

<Note>
Si l’installation réussit mais que `openclaw` n’est pas trouvé dans un nouveau terminal, consultez le [dépannage Node.js](/install/node#troubleshooting).
</Note>

---

## install.sh

<Tip>
Recommandé pour la plupart des installations interactives sur macOS/Linux/WSL.
</Tip>

### Flux (install.sh)

<Steps>
  <Step title="Detect OS">
    Prend en charge macOS et Linux (y compris WSL). Si macOS est détecté, installe Homebrew s’il est manquant.
  </Step>
  <Step title="Ensure Node.js 22+">
    Vérifie la version de Node et installe Node 22 si nécessaire (Homebrew sur macOS, scripts NodeSource sur Linux apt/dnf/yum).
  </Step>
  <Step title="Ensure Git">
    Installe Git s’il est manquant.
  </Step>
  <Step title="Install OpenClaw">
    - Méthode `npm` (par défaut) : installation npm globale  
    - Méthode `git` : clone/met à jour le dépôt, installe les dépendances avec pnpm, build, puis installe un wrapper dans `~/.local/bin/openclaw`
  </Step>
  <Step title="Post-install tasks">
    - Exécute `openclaw doctor --non-interactive` lors des mises à niveau et des installations via git (au mieux)  
    - Tente l’onboarding lorsque approprié (TTY disponible, onboarding non désactivé, vérifications bootstrap/config OK)  
    - Définit par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1`
  </Step>
</Steps>

### Détection d’un checkout source

Si exécuté à l’intérieur d’un checkout OpenClaw (`package.json` + `pnpm-workspace.yaml`), le script propose :

- utiliser le checkout (`git`), ou  
- utiliser l’installation globale (`npm`)

Si aucun TTY n’est disponible et qu’aucune méthode d’installation n’est définie, la valeur par défaut est `npm` avec un avertissement.

Le script se termine avec le code `2` en cas de sélection de méthode invalide ou de valeur `--install-method` invalide.

### Exemples (install.sh)

<Tabs>
  <Tab title="Default">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  </Tab>
  <Tab title="Skip onboarding">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-onboard
    ```
  </Tab>
  <Tab title="Git install">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
    ```
  </Tab>
  <Tab title="Dry run">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --dry-run
    ```
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag                            | Description                                                |
| ------------------------------- | ---------------------------------------------------------- |
| `--install-method npm\|git`     | Choisir la méthode d’installation (par défaut : `npm`). Alias : `--method` |
| `--npm`                         | Raccourci pour la méthode npm                              |
| `--git`                         | Raccourci pour la méthode git. Alias : `--github`          |
| `--version <version\|dist-tag>` | Version npm ou dist-tag (par défaut : `latest`)            |
| `--beta`                        | Utiliser le dist-tag beta si disponible, sinon `latest`    |
| `--git-dir <path>`              | Répertoire du checkout (par défaut : `~/openclaw`). Alias : `--dir` |
| `--no-git-update`               | Ignorer `git pull` pour un checkout existant               |
| `--no-prompt`                   | Désactiver les invites                                     |
| `--no-onboard`                  | Ignorer l’onboarding                                       |
| `--onboard`                     | Activer l’onboarding                                       |
| `--dry-run`                     | Afficher les actions sans appliquer les changements        |
| `--verbose`                     | Activer la sortie de debug (`set -x`, logs npm niveau notice) |
| `--help`                        | Afficher l’aide (`-h`)                                     |

  </Accordion>

  <Accordion title="Environment variables reference">

| Variable                                    | Description                                   |
| ------------------------------------------- | --------------------------------------------- |
| `OPENCLAW_INSTALL_METHOD=git\|npm`          | Méthode d’installation                        |
| `OPENCLAW_VERSION=latest\|next\|<semver>`   | Version npm ou dist-tag                       |
| `OPENCLAW_BETA=0\|1`                        | Utiliser la version beta si disponible        |
| `OPENCLAW_GIT_DIR=<path>`                   | Répertoire du checkout                        |
| `OPENCLAW_GIT_UPDATE=0\|1`                  | Activer/désactiver les mises à jour git       |
| `OPENCLAW_NO_PROMPT=1`                      | Désactiver les invites                        |
| `OPENCLAW_NO_ONBOARD=1`                     | Ignorer l’onboarding                          |
| `OPENCLAW_DRY_RUN=1`                        | Mode dry run                                  |
| `OPENCLAW_VERBOSE=1`                        | Mode debug                                    |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Niveau de log npm                             |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`          | Contrôle du comportement sharp/libvips (par défaut : `1`) |

  </Accordion>
</AccordionGroup>

---

## install-cli.sh

<Info>
Conçu pour les environnements où vous souhaitez tout installer sous un préfixe local (par défaut `~/.openclaw`) sans dépendre du Node système.
</Info>

### Flux (install-cli.sh)

<Steps>
  <Step title="Install local Node runtime">
    Télécharge l’archive Node (par défaut `22.22.0`) dans `<prefix>/tools/node-v<version>` et vérifie le SHA-256.
  </Step>
  <Step title="Ensure Git">
    Si Git est manquant, tente l’installation via apt/dnf/yum sur Linux ou Homebrew sur macOS.
  </Step>
  <Step title="Install OpenClaw under prefix">
    Installe avec npm en utilisant `--prefix <prefix>`, puis écrit un wrapper dans `<prefix>/bin/openclaw`.
  </Step>
</Steps>

### Exemples (install-cli.sh)

<Tabs>
  <Tab title="Default">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash
    ```
  </Tab>
  <Tab title="Custom prefix + version">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --prefix /opt/openclaw --version latest
    ```
  </Tab>
  <Tab title="Automation JSON output">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
    ```
  </Tab>
  <Tab title="Run onboarding">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --onboard
    ```
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag                   | Description                                                                     |
| ---------------------- | ------------------------------------------------------------------------------- |
| `--prefix <path>`      | Préfixe d’installation (par défaut : `~/.openclaw`)                            |
| `--version <ver>`      | Version OpenClaw ou dist-tag (par défaut : `latest`)                           |
| `--node-version <ver>` | Version de Node (par défaut : `22.22.0`)                                        |
| `--json`               | Émettre des événements NDJSON                                                    |
| `--onboard`            | Exécuter `openclaw onboard` après l’installation                                 |
| `--no-onboard`         | Ignorer l’onboarding (par défaut)                                                |
| `--set-npm-prefix`     | Sous Linux, forcer le préfixe npm vers `~/.npm-global` si le préfixe actuel n’est pas accessible en écriture |
| `--help`               | Afficher l’aide (`-h`)                                                            |

  </Accordion>

  <Accordion title="Environment variables reference">

| Variable                                    | Description                                                                       |
| ------------------------------------------- | --------------------------------------------------------------------------------- |
| `OPENCLAW_PREFIX=<path>`                    | Préfixe d’installation                                                            |
| `OPENCLAW_VERSION=<ver>`                    | Version OpenClaw ou dist-tag                                                      |
| `OPENCLAW_NODE_VERSION=<ver>`               | Version de Node                                                                   |
| `OPENCLAW_NO_ONBOARD=1`                     | Ignorer l’onboarding                                                              |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | Niveau de log npm                                                                 |
| `OPENCLAW_GIT_DIR=<path>`                   | Chemin de recherche pour nettoyage legacy (utilisé lors de la suppression de l’ancien checkout du sous-module `Peekaboo`) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1`          | Contrôle du comportement sharp/libvips (par défaut : `1`)                        |

  </Accordion>
</AccordionGroup>

---

## install.ps1

### Flux (install.ps1)

<Steps>
  <Step title="Ensure PowerShell + Windows environment">
    Nécessite PowerShell 5+.
  </Step>
  <Step title="Ensure Node.js 22+">
    S’il est manquant, tente l’installation via winget, puis Chocolatey, puis Scoop.
  </Step>
  <Step title="Install OpenClaw">
    - Méthode `npm` (par défaut) : installation npm globale avec le `-Tag` sélectionné  
    - Méthode `git` : clone/met à jour le dépôt, installe/build avec pnpm, puis installe un wrapper dans `%USERPROFILE%\.local\bin\openclaw.cmd`
  </Step>
  <Step title="Post-install tasks">
    Ajoute le répertoire bin nécessaire au PATH utilisateur lorsque possible, puis exécute `openclaw doctor --non-interactive` lors des mises à niveau et des installations via git (au mieux).
  </Step>
</Steps>

### Exemples (install.ps1)

<Tabs>
  <Tab title="Default">
    ```powershell
    iwr -useb https://openclaw.ai/install.ps1 | iex
    ```
  </Tab>
  <Tab title="Git install">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git
    ```
  </Tab>
  <Tab title="Custom git directory">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -InstallMethod git -GitDir "C:\openclaw"
    ```
  </Tab>
  <Tab title="Dry run">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -DryRun
    ```
  </Tab>
  <Tab title="Debug trace">
    ```powershell
    # install.ps1 has no dedicated -Verbose flag yet.
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```
  </Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Flag                      | Description                                            |
| ------------------------- | ------------------------------------------------------ |
| `-InstallMethod npm\|git` | Méthode d’installation (par défaut : `npm`)            |
| `-Tag <tag>`              | npm dist-tag (par défaut : `latest`)                   |
| `-GitDir <path>`          | Répertoire du checkout (par défaut : `%USERPROFILE%\openclaw`) |
| `-NoOnboard`              | Ignorer l’onboarding                                   |
| `-NoGitUpdate`            | Ignorer `git pull`                                     |
| `-DryRun`                 | Afficher uniquement les actions                        |

  </Accordion>

  <Accordion title="Environment variables reference">

| Variable                           | Description        |
| ---------------------------------- | ------------------ |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Méthode d’installation |
| `OPENCLAW_GIT_DIR=<path>`          | Répertoire du checkout |
| `OPENCLAW_NO_ONBOARD=1`            | Ignorer l’onboarding |
| `OPENCLAW_GIT_UPDATE=0`            | Désactiver `git pull` |
| `OPENCLAW_DRY_RUN=1`               | Mode dry run        |

  </Accordion>
</AccordionGroup>

<Note>
Si `-InstallMethod git` est utilisé et que Git est absent, le script s’arrête et affiche le lien vers Git for Windows.
</Note>

---

## CI et automatisation

Utilisez des flags/variables d’environnement non interactifs pour des exécutions prévisibles.

<Tabs>
  <Tab title="install.sh (non-interactive npm)">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --no-prompt --no-onboard
    ```
  </Tab>
  <Tab title="install.sh (non-interactive git)">
    ```bash
    OPENCLAW_INSTALL_METHOD=git OPENCLAW_NO_PROMPT=1 \
      curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  </Tab>
  <Tab title="install-cli.sh (JSON)">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
    ```
  </Tab>
  <Tab title="install.ps1 (skip onboarding)">
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    ```
  </Tab>
</Tabs>

---

## Problèmes courants

<AccordionGroup>
  <Accordion title="Why is Git required?">
    Git est requis pour la méthode d’installation `git`. Pour les installations `npm`, Git est tout de même vérifié/installé afin d’éviter les erreurs `spawn git ENOENT` lorsque des dépendances utilisent des URLs git.
  </Accordion>

  <Accordion title="Why does npm hit EACCES on Linux?">
    Certaines configurations Linux pointent le préfixe global npm vers des chemins appartenant à root. `install.sh` peut basculer le préfixe vers `~/.npm-global` et ajouter l’export PATH aux fichiers rc du shell (lorsqu’ils existent).
  </Accordion>

  <Accordion title="sharp/libvips issues">
    Les scripts définissent par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1` pour éviter que sharp ne se compile contre la libvips système. Pour remplacer :

    ```bash
    SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  </Accordion>

  <Accordion title='Windows: "npm error spawn git / ENOENT"'>
    Installez Git for Windows, rouvrez PowerShell, puis relancez l’installateur.
  </Accordion>

  <Accordion title='Windows: "openclaw is not recognized"'>
    Exécutez `npm config get prefix`, ajoutez `\bin`, ajoutez ce répertoire au PATH utilisateur, puis rouvrez PowerShell.
  </Accordion>

  <Accordion title="Windows: how to get verbose installer output">
    `install.ps1` n’expose pas actuellement de switch `-Verbose`.  
    Utilisez le traçage PowerShell pour un diagnostic au niveau du script :

    ```powershell
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```
  </Accordion>

  <Accordion title="openclaw not found after install">
    Généralement un problème de PATH. Consultez le [dépannage Node.js](/install/node#troubleshooting).
  </Accordion>
</AccordionGroup>
