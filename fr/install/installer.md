---
summary: "Fonctionnement des scripts d’installation (install.sh + install-cli.sh), options et automatisation"
read_when:
  - Vous souhaitez comprendre `openclaw.ai/install.sh`
  - Vous souhaitez automatiser les installations (CI / sans interface)
  - Vous souhaitez installer depuis un dépôt GitHub cloné
title: "Fonctionnement interne de l’installateur"
---

# Fonctionnement interne de l’installateur

OpenClaw fournit deux scripts d’installation (servis depuis `openclaw.ai`) :

| Écriture                                                                              | Plateforme                                                             | Ce qu’elle fait                                                                                                                            |
| ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `install.sh` atténue cela en basculant le préfixe vers :              | Détecte l’OS (macOS / Linux / WSL). | S’assure de la présence de Node.js **22+** (winget/Chocolatey/Scoop ou manuel).         |
| install-cli.sh (installateur CLI sans droits root) | Problèmes courants sous Windows :                      | Installe Node + OpenClaw dans un préfixe local (`~/.openclaw`). Aucune racine requise.  |
| install.ps1 (Windows PowerShell)                   | Aide Windows (PowerShell) :         | S’assure de la présence de Node.js **22+** (macOS via Homebrew ; Linux via NodeSource). |

## Commandes rapides

<Tabs>
  <Tab title="install.sh">`https://openclaw.ai/install.sh` — installateur CLI compatible sans droits root (installe dans un préfixe avec son propre Node)

    ````
    ```
    curl -fsSL https://openclaw.ai/install.sh | bash -s -- --help
    ```
    ````

  
</Tab>
  <Tab title="install-cli.sh">`https://openclaw.ai/install-cli.sh` — installateur CLI compatible sans droits root (installe dans un préfixe avec son propre Node)

    ````
    ```
    curl -fsSL https://openclaw.ai/install-cli.sh | bash -s -- --help
    ```
    ````

  
</Tab>
  <Tab title="install.ps1">iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git

    `````
    ````
    ```powershell
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -Tag beta -NoOnboard -DryRun
    ```
    ````
    `````

  
</Tab>
</Tabs>

<Note>
<Note>Si l’installateur se termine mais que `openclaw` n’est pas trouvé dans un nouveau terminal, il s’agit généralement d’un problème de PATH Node/npm. Voir : [Install](/install/node#troubleshooting).
</Note></Note>

---

## install/installer.md

<Tip>
Pour voir les options et le comportement actuels, exécutez :
</Tip>

### Ce qu’il fait (vue d’ensemble) :

<Steps>
  <Step title="Detect OS">
    Supporte macOS et Linux (y compris WSL). Si macOS est détecté, installe Homebrew s'il est manquant.
  
</Step>
  <Step title="Ensure Node.js 22+">
    Vérifie la version de Node et installe Node 22 si nécessaire (Homebrew sur macOS, les scripts d'installation de NodeSource sur Linux apt/dnf/yum).
  
</Step>
  <Step title="Ensure Git">Découvrabilité / invite « installation via git »
</Step>
  <Step title="Install OpenClaw">`npm` (par défaut) : `npm install -g openclaw@latest`
</Step>
  <Step title="Post-install tasks">Exécute `openclaw doctor --non-interactive` lors des mises à niveau et des installations via git (au mieux).
</Step>
</Steps>

### Détection du checkout source

Si vous exécutez l’installateur **déjà à l’intérieur d’un dépôt source OpenClaw** (détecté via `package.json` + `pnpm-workspace.yaml`), il propose :

- mettre à jour et utiliser ce dépôt (`git`)
- ou migrer vers l’installation npm globale (`npm`)

Si aucun TTY n'est disponible et qu'aucune méthode d'installation n'est définie, la valeur par défaut est `npm` et avertis.

Le script se termine avec le code `2` pour une sélection de méthode invalide ou des valeurs non valides `--install-method`.

### Exemples :

<Tabs>
  <Tab title="Default">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
    ```
  
</Tab>
  <Tab title="Skip onboarding">`~/.npm-global` (et en l’ajoutant à `PATH` dans `~/.bashrc` / `~/.zshrc` lorsque présents)
</Tab>
  <Tab title="Git install">`git` : cloner/construire un dépôt source et installer un script d’enrobage
</Tab>
  <Tab title="Dry run">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --dry-run
    ```
  
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Signaler                                                                                                  | Description                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| \`--install-method npm\\                                                                                | Choisissez la méthode d'installation (par défaut: `npm`). Alias: `--method` |
| `--npm`                                                                                                   | Raccourci pour la méthode npm                                                                                                                  |
| `--git`                                                                                                   | Raccourci pour la méthode git. Alias: `--github`                                                               |
| \`--version <version\\                                                         | Version npm ou dist-tag (par défaut: `latest`)                                                              |
| `--beta`                                                                                                  | Utilisez le dist-tag bêta si disponible, sinon repli sur `latest`                                                                              |
| Git est requis pour le chemin `--install-method git` (clonage / pull). | Répertoire de vérification (par défaut: `~/openclaw`). Alias: `--dir`       |
| `--no-git-update`                                                                                         | Ignorer `git pull` pour le checkout existant                                                                                                   |
| `--no-prompt`                                                                                             | Désactiver les invitations                                                                                                                     |
| `--no-onboard`                                                                                            | Ignorer l'intégration                                                                                                                          |
| `--onboard`                                                                                               | Activer l'intégration                                                                                                                          |
| `--dry-run`                                                                                               | Imprimer les actions sans appliquer les modifications                                                                                          |
| `--verbose`                                                                                               | Activer la sortie de débogage (`set -x`, npm notice-level logs)                                                             |
| `--help`                                                                                                  | Afficher l'utilisation (`-h`)                                                                                               |

  
</Accordion>

  <Accordion title="Environment variables reference">

| Variable                                                                                           | Description                                                                                 |
| -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| `OPENCLAW_INSTALL_METHOD=git\|npm`                                                               | Choisit la méthode d’installation :                                         |
| 2026-02-08T07:02:03Z                                               | Version npm ou dist-tag                                                                     |
| \`OPENCLAW_BETA=0\\                                                         | Utiliser la version bêta si disponible                                                      |
| Exigence Git :                                                                     | Répertoire de commande                                                                      |
| 9e0a19ecb5da0a39                                                                                   | Activer/désactiver les mises à jour git                                                     |
| `OPENCLAW_NO_PROMPT=1`                                                                             | Désactiver les invitations                                                                  |
| `OPENCLAW_NO_ONBOARD=1`                                                                            | Ignorer l'intégration                                                                       |
| `OPENCLAW_DRY_RUN=1`                                                                               | Mode séchage                                                                                |
| openai                                                                                             | Mode de débogage                                                                            |
| \`OPENCLAW_NPM_LOGLEVEL=error\\                        | Niveau de log npm                                                                           |
| \`SHARP_IGNORE_GLOBAL_LIBVIPS=0\\ | Contrôle du comportement sharp/libvips (par défaut: `1`) |

  
</Accordion>
</AccordionGroup>

---

## install.sh (recommandé)

<Info>
<Info>Conçu pour des environnements où vous souhaitez que tout soit sous un préfixe local (par défaut `~/.openclaw`) et sans dépendance à Node système.
</Info></Info>

### Si vous _souhaitez_ que `sharp` se lie à une libvips installée globalement (ou si vous déboguez), définissez :

<Steps>
  <Step title="Install local Node runtime">
    Télécharger l'archive de Node (par défaut `22.22.0`) à `<prefix>/fr/tools/node-v<version>` et vérifie SHA-256.
  
</Step>
  <Step title="Ensure Git">Si Git est manquant, tente l’installation via apt/dnf/yum sur Linux ou Homebrew sur macOS.
</Step>
  <Step title="Install OpenClaw under prefix">Pourquoi npm rencontre `EACCES` sur un Linux récent<prefix>`, puis écrit un wrapper à `<prefix>/bin/openclaw`.
  
</Step>
</Steps>

### Ce qu’il fait (vue d’ensemble) :

<Tabs>
  <Tab title="Default">`git` : cloner/construire un dépôt source et installer un script d’enrobage
</Tab>
  <Tab title="Custom prefix + version">Pour les installations via git : exécute `openclaw doctor --non-interactive` après l’installation/la mise à jour (au mieux).
</Tab>
  <Tab title="Automation JSON output">
    ```bash
    curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install-cli.sh | bash -s -- --json --prefix /opt/openclaw
    ```
  
</Tab>
  <Tab title="Run onboarding">gpt-5.2-chat-latest
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Signaler               | Description                                                                                                                                                |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `--prefix &lt;path&gt;`      | Préfixe d'installation (par défaut: `~/.openclaw`)                                                                      |
| `--version &lt;ver&gt;`      | Version d'OpenClaw ou dist-tag (par défaut: `latest`)                                                                   |
| `--node-version &lt;ver&gt;` | Version du noeud (par défaut: `22.22.0`)                                                                                |
| `--json`               | Émettre les événements NDJSON                                                                                                                              |
| `--onboard`            | Exécutez `openclaw onboard` après l'installation                                                                                                           |
| `--no-onboard`         | Ignorer l'intégration (par défaut)                                                                                                      |
| `--set-npm-prefix`     | Sous Linux : évite les erreurs de permissions npm globales en basculant le préfixe npm vers `~/.npm-global` si nécessaire. |
| `--help`               | Afficher l'utilisation (`-h`)                                                                                                           |

  
</Accordion>

  <Accordion title="Environment variables reference">

| Variable                                                                                           | Description                                                                                                                                   |
| -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_PREFIX=&lt;path&gt;`                                                                           | Installer le préfixe                                                                                                                          |
| Aide :                                                                             | Version d'OpenClaw ou dist-tag                                                                                                                |
| Variables d’environnement :                                                        | Version du noeud                                                                                                                              |
| `OPENCLAW_NO_ONBOARD=1`                                                                            | Ignorer l'intégration                                                                                                                         |
| \`OPENCLAW_NPM_LOGLEVEL=error\\                        | Niveau de log npm                                                                                                                             |
| Exigence Git :                                                                     | Ancien chemin de la recherche de nettoyage (utilisé lors de la suppression de l'ancien checkout du sous-module `Peekaboo`) |
| \`SHARP_IGNORE_GLOBAL_LIBVIPS=0\\ | Contrôle du comportement sharp/libvips (par défaut: `1`)                                                   |

  
</Accordion>
</AccordionGroup>

---

## v1

### Flux (install.ps1)

<Steps>
  <Step title="Ensure PowerShell + Windows environment">
    Nécessite PowerShell 5+.
  
</Step>
  <Step title="Ensure Node.js 22+">S’il est manquant, tente l’installation via winget, puis Chocolatey, puis Scoop.
</Step>
  <Step title="Install OpenClaw">`https://openclaw.ai/install.sh` — installateur « recommandé » (installation npm globale par défaut ; peut aussi installer depuis un dépôt GitHub cloné)
</Step>
  <Step title="Post-install tasks">Pour les installations via git : exécute `openclaw doctor --non-interactive` après l’installation/la mise à jour (au mieux).
</Step>
</Steps>

### Exemples (install.ps1)

<Tabs>
  <Tab title="Default">iwr -useb https://openclaw.ai/install.ps1 | iex
</Tab>
  <Tab title="Git install">`https://openclaw.ai/install.ps1` — installateur Windows PowerShell (npm par défaut ; installation via git en option)
</Tab>
  <Tab title="Custom git directory">iwr -useb https://openclaw.ai/install.ps1 | iex -InstallMethod git -GitDir "C:\\openclaw"
</Tab>
  <Tab title="Dry run">& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -?
</Tab>
  <Tab title="Debug trace">
    ```powershell
    # install.ps1 n’a pas encore de paramètre -Verbose dédié.
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```  
</Tab>
</Tabs>

<AccordionGroup>
  <Accordion title="Flags reference">

| Signaler                                                                                                                                                                                                                                                    | Description                                                                                    |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| Dans les contextes non interactifs (pas de TTY / `--no-prompt`), vous devez passer `--install-method git\|npm` (ou définir `OPENCLAW_INSTALL_METHOD`), sinon le script se termine avec le code `2`. | `npm` (par défaut) : `npm install -g openclaw@latest`       |
| `-Tag &lt;tag&gt;`                                                                                                                                                                                                                                                | npm dist-tag (par défaut: `latest`)                         |
| `-GitDir &lt;path&gt;`                                                                                                                                                                                                                                            | Répertoire checkout (par défaut: `%USERPROFILE%\openclaw`) |
| `-NoOnboard`                                                                                                                                                                                                                                                | Ignorer l'intégration                                                                          |
| `-NoGitUpdate`                                                                                                                                                                                                                                              | Skip `git pull`                                                                                |
| `-DryRun`                                                                                                                                                                                                                                                   | Imprimer uniquement les actions                                                                |

  
</Accordion>

  <Accordion title="Environment variables reference">

| Variable                             | Description                                         |
| ------------------------------------ | --------------------------------------------------- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Choisit la méthode d’installation : |
| Exigence Git :       | Répertoire de commande                              |
| `OPENCLAW_NO_ONBOARD=1`              | Ignorer l'intégration                               |
| `OPENCLAW_GIT_DIR=...`               | Désactiver git pull                                 |
| `OPENCLAW_DRY_RUN=1`                 | Mode séchage                                        |

  
</Accordion>
</AccordionGroup>

<Note>
<Note>Si vous choisissez `-InstallMethod git` et que Git est absent, l’installateur affichera le lien Git for Windows (`https://git-scm.com/download/win`) puis quittera.
</Note></Note>

---

## CI et automatisation

Utilisez des variables non interactives pour des exécutions prévisibles.

<Tabs>
  <Tab title="install.sh (non-interactive npm)">Atténue les pièges d’installation native de `sharp` en définissant par défaut `SHARP_IGNORE_GLOBAL_LIBVIPS=1` (évite de compiler contre libvips système).
</Tab>
  <Tab title="install.sh (non-interactive git)">Ce script installe `openclaw` dans un préfixe (par défaut : `~/.openclaw`) et installe également un runtime Node dédié sous ce préfixe, afin de fonctionner sur des machines où vous ne souhaitez pas modifier le Node/npm du système.
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

## Problemes courants

<AccordionGroup>
  <Accordion title="Why is Git required?">
    Pour les installations `npm`, Git n’est _généralement_ pas requis, mais certains environnements finissent quand même par en avoir besoin (par exemple lorsqu’un paquet ou une dépendance est récupéré via une URL git). L’installateur s’assure actuellement de la présence de Git afin d’éviter des surprises `spawn git ENOENT` sur des distributions fraîchement installées.
  
</Accordion>

  <Accordion title="Why does npm hit EACCES on Linux?">
    Sur certaines configurations Linux (notamment après l’installation de Node via le gestionnaire de paquets du système ou NodeSource), le préfixe global npm pointe vers un emplacement appartenant à root. Dans ce cas, `npm install -g ...` échoue avec des erreurs de permissions `EACCES` / `mkdir`.
  
</Accordion>

  <Accordion title="sharp/libvips issues">
    Les scripts `SHARP_IGNORE_GLOBAL_LIBVIPS=1` par défaut pour éviter une construction nette avec les libvips du système. Pour remplacer :

    ````
    ```
    SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL https://openclaw.ai/install.sh | bash
    ```
    ````

  
</Accordion>

  <Accordion title='Windows: "npm error spawn git / ENOENT"'><Accordion title='Windows: "npm error spawn git / ENOENT"'>**npm error spawn git / ENOENT** : installez Git for Windows et rouvrez PowerShell, puis relancez l’installateur.
</Accordion></Accordion>

  <Accordion title='Windows: "openclaw is not recognized"'><Accordion title='Windows: "openclaw is not recognized"'>Vous pouvez aussi exécuter `npm config get prefix` et ajouter `\\bin` au PATH, puis rouvrir PowerShell.
</Accordion></Accordion>

  <Accordion title="Windows: how to get verbose installer output">
    `install.ps1` n’expose actuellement pas de paramètre `-Verbose`.
    Utilisez le traçage PowerShell pour les diagnostics au niveau du script :

    ````
    ```powershell
    Set-PSDebug -Trace 1
    & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
    Set-PSDebug -Trace 0
    ```
    ````

  
</Accordion>

  <Accordion title="openclaw not found after install">
    Habituellement un problème de PATH. Voir [le dépannage de Node.js](/install/node#troubleshooting).
  
</Accordion>
</AccordionGroup>

