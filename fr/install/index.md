---
title: "Aperçu de l’installation"
---

# Aperçu de l’installation

Déjà suivi [Pour commencer](/start/getting-started) ? Vous êtes prêt — cette page est destinée aux méthodes alternatives d'installation, aux instructions spécifiques à la plate-forme et à la maintenance.

## Configuration requise

- **[Node 22+](/install/node)** (le [script d'installation](#install-methods) l'installera s'il est manquant)
- macOS, Linux ou Windows
- `pnpm` uniquement si vous compilez depuis les sources

<Note>
Sous Windows, nous recommandons fortement d’exécuter OpenClaw sous [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
</Note>

## Choisir votre méthode d’installation

<Tip>
Le **script d'installation** est le moyen recommandé pour installer OpenClaw. Il gère la détection de Node, l'installation et la prise en main en une seule étape.
</Tip>

<AccordionGroup>
  <Accordion title="Installer script" icon="rocket" defaultOpen>
    Télécharge la CLI, l’installe globalement via npm et lance l’assistant de prise en main.

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      </Tab>
    </Tabs>

    C'est tout — le script gère la détection de Node, l'installation et la prise en main.

    Pour ignorer la prise en main et simplement installer le binaire :

    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash -s -- --no-onboard
        ```
      </Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        & ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
        ```
      </Tab>
    </Tabs>

    Pour tous les flags, variables d’environnement et options CI/automatisation, consultez [Installer internals](/install/installer).

  </Accordion>

  <Accordion title="npm / pnpm" icon="package">
    Si vous disposez déjà de Node 22+ et préférez gérer l’installation vous-même :

    <Tabs>
      <Tab title="npm">
        ```bash
        npm install -g openclaw@latest
        openclaw onboard --install-daemon
        ```

        <Accordion title="erreurs de build sharp ?">
          Si libvips est installé globalement (courant sur macOS via Homebrew) et que `sharp` échoue, forcez les binaires précompilés :

          ```bash
          SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
          ```

          Si vous voyez `sharp: Please add node-gyp to your dependencies`, installez les outils de build (macOS : Xcode CLT + `npm install -g node-gyp`) ou utilisez la variable d’environnement ci-dessus.
        </Accordion>
      </Tab>
      <Tab title="pnpm">
        ```bash
        pnpm add -g openclaw@latest
        pnpm approve-builds -g        # approve openclaw, node-llama-cpp, sharp, etc.
        openclaw onboard --install-daemon
        ```

        <Note>
        pnpm nécessite une approbation explicite pour les paquets contenant des scripts de build. Après que la première installation affiche l’avertissement « Ignored build scripts », exécutez `pnpm approve-builds -g` et sélectionnez les paquets listés.
        </Note>
      </Tab>
    </Tabs>

  </Accordion>

  <Accordion title="From source" icon="github">
    Pour les contributeurs ou toute personne souhaitant exécuter depuis un dépôt local.

    <Steps>
      <Step title="Clone and build">
        Clonez le dépôt [OpenClaw repo](https://github.com/openclaw/openclaw) et compilez :

        ```bash
        git clone https://github.com/openclaw/openclaw.git
        cd openclaw
        pnpm install
        pnpm ui:build
        pnpm build
        ```
      </Step>
      <Step title="Link the CLI">
        Rendez la commande `openclaw` disponible globalement :

        ```bash
        pnpm link --global
        ```

        Sinon, ignorez le lien et exécutez les commandes via `pnpm openclaw ...` depuis le dépôt.
      </Step>
      <Step title="Run onboarding">
        ```bash
        openclaw onboard --install-daemon
        ```
      </Step>
    </Steps>

    Pour des workflows de développement plus avancés, consultez [Setup](/start/setup).

  </Accordion>
</AccordionGroup>

## Autres options d’installation

<CardGroup cols={2}>
  <Card title="Docker" href="/install/docker" icon="container">
    Déploiements confinés ou sans interface graphique.
  </Card>
  <Card title="Podman" href="/install/podman" icon="container">
    Conteneur rootless : exécutez `setup-podman.sh` une fois, puis le script de lancement.
  </Card>
  <Card title="Nix" href="/install/nix" icon="snowflake">
    Installation déclarative via Nix.
  </Card>
  <Card title="Ansible" href="/install/ansible" icon="server">
    Provisionnement automatisé de flotte.
  </Card>
  <Card title="Bun" href="/install/bun" icon="zap">
    Utilisation CLI uniquement via le runtime Bun.
  </Card>
</CardGroup>

## Après l’installation

Vérifiez que tout fonctionne :

```bash
openclaw doctor         # vérifier les problèmes de configuration
openclaw status         # gateway status
openclaw dashboard      # ouvrir l'interface navigateur
```

Si vous avez besoin de chemins d’exécution personnalisés, utilisez :

- `OPENCLAW_HOME` pour les chemins internes basés sur le répertoire personnel
- `OPENCLAW_STATE_DIR` pour l’emplacement de l’état mutable
- `OPENCLAW_CONFIG_PATH` pour l’emplacement du fichier de configuration

Voir [Variables d’environnement](/help/environment) pour la priorité et les détails complets.

## Dépannage : `openclaw` introuvable

<Accordion title="PATH diagnosis and fix">
  Diagnostic rapide :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) **n’est pas** présent dans votre `$PATH`, votre shell ne peut pas trouver les binaires npm globaux (y compris `openclaw`).

Correctif — ajoutez-le à votre fichier de démarrage du shell (`~/.zshrc` ou `~/.bashrc`) :

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Sous Windows, ajoutez la sortie de `npm prefix -g` à votre PATH.

Puis ouvrez un nouveau terminal (ou `rehash` dans zsh / `hash -r` dans bash).
</Accordion>

## Mise à jour / désinstallation

<CardGroup cols={3}>
  <Card title="Updating" href="/install/updating" icon="refresh-cw">
    Maintenez OpenClaw à jour.
  </Card>
  <Card title="Migrating" href="/install/migrating" icon="arrow-right">
    Déplacer vers une nouvelle machine.
  </Card>
  <Card title="Uninstall" href="/install/uninstall" icon="trash-2">
    Supprimez complètement OpenClaw.
  </Card>
</CardGroup>
