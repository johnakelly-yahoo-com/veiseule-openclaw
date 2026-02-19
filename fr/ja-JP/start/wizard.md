---
read_when:
  - Lors de l’exécution de l’assistant d’onboarding ou de la configuration
  - Lors de la configuration d’une nouvelle machine
sidebarTitle: Wizard (CLI)
summary: "Assistant d’onboarding CLI : configuration interactive du Gateway, de l’espace de travail, des canaux et des Skills"
title: Assistant d’onboarding (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Assistant d’onboarding (CLI)

L’assistant d’onboarding CLI est le parcours recommandé pour configurer OpenClaw sur macOS, Linux et Windows (via WSL2). Il configure les paramètres par défaut de l’espace de travail, les canaux et les Skills, en plus d’un Gateway local ou d’une connexion à un Gateway distant.

```bash
openclaw onboard
```

<Info>
Méthode la plus rapide pour démarrer votre premier chat : ouvrez le Control UI (aucune configuration de canal requise). Exécutez `openclaw dashboard` pour discuter dans votre navigateur. Documentation : [Dashboard](/web/dashboard).
</Info>

## Démarrage rapide vs configuration détaillée

L’assistant commence par vous proposer un choix entre **Démarrage rapide** (paramètres par défaut) et **Configuration détaillée** (contrôle complet).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Gateway local sur loopback
    - Espace de travail existant ou espace de travail par défaut
    - Port Gateway `18789`
    - Jeton d’authentification Gateway généré automatiquement (également généré sur loopback)
    - Publication Tailscale désactivée
    - DM Telegram et WhatsApp autorisés par liste blanche par défaut (la saisie du numéro de téléphone peut être demandée)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Affiche le flux complet de prompts pour le mode, l’espace de travail, le Gateway, les canaux, les démons et les Skills
  
</Tab>
</Tabs>

## Détails de l’onboarding CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Description complète des flux locaux et distants, matrice d’authentification et de modèles, sortie de configuration, RPC de l’assistant, fonctionnement de signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Recettes d’onboarding non interactif et exemples automatisés de `agents add`.
  
</Card>
</Columns>

## Commandes de suivi courantes

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` n’implique pas le mode non interactif. Pour les scripts, utilisez `--non-interactive`.
</Note>

<Tip>
Recommandé : configurez une clé API Brave Search afin que les agents puissent utiliser `web_search` (`web_fetch` fonctionne sans clé). Méthode la plus simple : exécutez `openclaw configure --section web` pour enregistrer `tools.web.search.apiKey`. Documentation : [Outils Web](/tools/web).
</Tip>

## Documentation associée

- Référence des commandes CLI : [`openclaw onboard`](/cli/onboard)
- Onboarding de l’application macOS : [Onboarding](/start/onboarding)
- Procédure de premier démarrage de l’agent : [Bootstrap de l’agent](/start/bootstrapping)
