---
summary: "npm install -g openclaw@latest"
read_when:
  - Vous avez besoin d'une méthode d'installation autre que le démarrage rapide
  - Vous voulez déployer sur une plateforme cloud
  - Vous devez mettre à jour, migrer ou désinstaller
title: "Aperçu de l’installation"
---

# Aperçu de l’installation

Déjà suivi [Pour commencer] (/start/getting-started ) ? Vous êtes prêt — cette page est destinée aux méthodes alternatives d'installation, aux instructions spécifiques à la plate-forme et à la maintenance.

## Configuration requise

- **[Node 22+](/install/node)** (le [script d'installation de l'installateur](#install-methods) l'installera s'il est manquant)
- macOS, Linux ou Windows via WSL2
- `pnpm` uniquement si vous compilez depuis les sources

<Note>
<Note>Sous Windows, nous recommandons fortement d’exécuter OpenClaw sous [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).
</Note></Note>

## Choisir votre methode d’installation

<Tip>
Le **script d'installation** est le moyen recommandé pour installer OpenClaw. Il gère la détection, l'installation et l'intégration des nœuds en une seule étape.
</Tip>

<AccordionGroup>
  <Accordion title="Installer script" icon="rocket" defaultOpen>Installe `openclaw` globalement via npm et lance la prise en main.

    ````
    ```
    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw.ai/install. h | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw. i/install. s1 | iex
        ```
      
</Tab>
    
</Tabs>
    
    C'est tout — le script gère la détection de Node, installation et embarquement.
    
    Pour ignorer l'intégration et juste installer le binaire :
    
    <Tabs>
      <Tab title="macOS / Linux / WSL2">
        ```bash
        curl -fsSL https://openclaw. i/install. h | bash -s -- --no-onboard
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        & ([scriptblock]::Create((iwr -useb https://openclaw. i/install. s1))) -NoOnboard
        ```
      
</Tab>
    
</Tabs>
    
    Pour tous les drapeaux, env vars, and CI/automation options, see [Installer internals](/install/installer).
    ```
    ````

  
</Accordion>

  <Accordion title="npm / pnpm" icon="package"><Accordion title="npm / pnpm" icon="package">Si vous disposez déjà de Node2+ et préférez gérer l’installation vous-même:

    ````
    ```
    <Tabs>
      <Tab title="npm">
        ```bash
        npm install -g openclaw@latest
        openclaw onboard --install-daemon
        ```
    
        <Accordion title="erreurs de build sharp�?">
          Si libvips est installé globalement (courant sur macOS via Homebrew) et que `sharp` échoue, forcez les binaires précompilés�:
    
          ```bash
          SHARP_IGNORE_GLOBAL_LIBVIPS=1 npm install -g openclaw@latest
          ```
    
          Si vous voyez `sharp: Please add node-gyp to your dependencies`, installez soit les outils de build (macOS�: Xcode CLT + `npm install -g node-gyp`), soit utilisez la variable d’environnement ci-dessus.
        
</Accordion>
      
</Tab>
      <Tab title="pnpm">
        ```bash
        pnpm add -g openclaw@latest
        pnpm approve-builds -g        # approve openclaw, node-llama-cpp, sharp, etc.
        openclaw onboard --install-daemon
        ```
    
        <Note>
        pnpm nécessite une approbation explicite pour les paquets contenant des scripts de build. Après que la première installation affiche l’avertissement «�Ignored build scripts�», exécutez `pnpm approve-builds -g` et sélectionnez les paquets listés.
        
</Note>
      
</Tab>
    
</Tabs>
    ```
    ````

  
</Accordion>

  <Accordion title="From source" icon="github">Pour les contributeurs ou toute personne souhaitant exécuter depuis un dépôt local.

    ````
    ```
    git clone https://github.com/openclaw/openclaw.git
    cd openclaw
    pnpm install
    pnpm ui:build # auto-installs UI deps on first run
    pnpm build
    openclaw onboard --install-daemon
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Autres options d’installation

<CardGroup cols={2}>
  <Card title="Docker" href="/install/docker" icon="container">
    déploiements confinés ou sans tête.
  
</Card>
  <Card title="Podman" href="/install/podman" icon="container">
    Conteneur rootless : exécutez `setup-podman.sh` une fois, puis le script de lancement.
  
</Card>
  <Card title="Nix" href="/install/nix" icon="snowflake">
    Installation déclarative via Nix.
  
</Card>
  <Card title="Ansible" href="/install/ansible" icon="server">
    provisionnement automatique de la flotte.
  
</Card>
  <Card title="Bun" href="/install/bun" icon="zap">
    Utilisation CLI-seulement via le runtime Bun.
  
</Card>
</CardGroup>

## Apres l’installation

Vérifiez que tout fonctionne :

```bash
openclaw doctor # vérifier les problèmes de configuration
openclaw status # gateway status
openclaw dashboard # ouvrir l'interface utilisateur du navigateur
```

Si vous avez besoin de chemins d’exécution personnalisés, utilisez :

- `OPENCLAW_HOME` pour les chemins internes basés sur le répertoire personnel
- `OPENCLAW_STATE_DIR` pour l’emplacement de l’état mutable
- `OPENCLAW_CONFIG_PATH` pour l’emplacement du fichier de configuration

Voir [Variables d’environnement](/help/environment) pour la priorité et les détails complets.

## Depannage : `openclaw` introuvable (PATH)

<Accordion title="PATH diagnosis and fix"><Accordion title="PATH diagnosis and fix">Diagnostic rapide :

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

Si `$(npm prefix -g)/bin` (macOS/Linux) ou `$(npm prefix -g)` (Windows) **n’est pas** present dans `echo "$PATH"`, votre shell ne peut pas trouver les binaires npm globaux (y compris `openclaw`).

Correctif : ajoutez-le a votre fichier de demarrage du shell (zsh : `~/.zshrc`, bash : `~/.bashrc`) :

```bash
# macOS / Linux
export PATH="$(npm prefix -g)/bin:$PATH"
```

Sous Windows, ajoutez la sortie de `npm prefix -g` a votre PATH.

Puis ouvrez un nouveau terminal (ou `rehash` dans zsh / `hash -r` dans bash). 
</Accordion>

## Mise a jour / desinstallation

<CardGroup cols={3}>
  <Card title="Updating" href="/install/updating" icon="refresh-cw">Maintenez OpenClaw à jour.
</Card>
  <Card title="Migrating" href="/install/migrating" icon="arrow-right">
    Déplacer vers une nouvelle machine.
  
</Card>
  <Card title="Uninstall" href="/install/uninstall" icon="trash-2">Supprimez complètement OpenClaw.
</Card>
</CardGroup>

