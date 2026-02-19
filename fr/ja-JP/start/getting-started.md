---
read_when:
  - Configuration initiale à partir de zéro
  - Je veux connaître le chemin le plus court vers un chat fonctionnel
summary: Installez OpenClaw et lancez votre premier chat en quelques minutes.
title: Introduction
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Introduction

Objectif : obtenir un premier chat fonctionnel avec une configuration minimale, en partant de zéro.

<Info>
Méthode la plus rapide pour chatter : ouvrez le Control UI (aucune configuration de canal requise). Exécutez `openclaw dashboard` pour chatter dans le navigateur, ou<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Hôte Gateway</Tooltip>ouvrez `http://127.0.0.1:18789/`.
Documentation : [Dashboard](/web/dashboard) et [Control UI](/web/control-ui).
</Info>

## Prérequis

- Node 22 ou version ultérieure

<Tip>
Si vous n’êtes pas sûr, vérifiez la version de Node avec `node --version`.
</Tip>

## Configuration rapide (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
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

    ```
    <Note>
    Autres méthodes d’installation et prérequis : [Installation](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    L’assistant configure l’authentification, les paramètres Gateway et les canaux optionnels.
    Pour plus de détails, consultez [Assistant d’onboarding](/start/wizard).
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Si vous avez installé le service, il devrait déjà être en cours d’exécution :

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Si le Control UI se charge, le Gateway est prêt à être utilisé.
</Check>

## Vérifications optionnelles et fonctionnalités supplémentaires

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Utile pour des tests rapides ou le dépannage.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Un canal configuré est requis.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## En savoir plus

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">    Référence complète de l’assistant CLI et options avancées.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">    Flux de première exécution de l’application macOS.
  
</Card>
</Columns>

## État après achèvement

- Gateway en cours d’exécution
- Authentification configurée
- Accès au Control UI ou canal connecté

## Prochaines étapes

- Sécurité et approbation des DM : [Appairage](/channels/pairing)
- Connecter davantage de canaux : [Canaux](/channels)
- Workflows avancés et build depuis les sources : [Configuration](/start/setup)
