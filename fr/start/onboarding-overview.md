---
summary: "Aperçu des options et des flux d’onboarding OpenClaw"
read_when:
  - Choisir un parcours d’onboarding
  - Configuration d’un nouvel environnement
title: "Aperçu de l’onboarding"
sidebarTitle: "Aperçu de l’onboarding"
---

# Aperçu de l’onboarding

OpenClaw prend en charge plusieurs parcours d’onboarding selon l’emplacement d’exécution du Gateway
et la manière dont vous préférez configurer les fournisseurs.

## Choisissez votre parcours d’onboarding

- **Assistant CLI** pour macOS, Linux et Windows (via WSL2).
- **Application macOS** pour un premier lancement guidé sur Mac Apple silicon ou Intel.

## Assistant d’onboarding CLI

Exécutez l’assistant dans un terminal :

```bash
openclaw onboard
```

Utilisez l’assistant CLI lorsque vous souhaitez un contrôle total du Gateway, de l’espace de travail,
des canaux et des Skills. Docs :

- [Assistant de configuration (CLI)](/start/wizard)
- [commande `openclaw onboard`](/cli/onboard)

## Configuration de l’app macOS

Utilisez l’app OpenClaw lorsque vous souhaitez une configuration entièrement guidée sur macOS. Docs :

- [Configuration (app macOS)](/start/onboarding)

## Fournisseur personnalisé

Si vous avez besoin d’un endpoint qui n’est pas listé, y compris des fournisseurs hébergés qui exposent des API OpenAI ou Anthropic standard, choisissez **Custom Provider** dans l’assistant CLI. Il vous sera demandé de :

- Choisir OpenAI-compatible, Anthropic-compatible ou **Unknown** (détection automatique).
- Saisir une URL de base et une clé API (si requise par le fournisseur).
- Fournir un ID de modèle et un alias facultatif.
- Choisir un Endpoint ID afin que plusieurs endpoints personnalisés puissent coexister.

Pour les étapes détaillées, suivez la documentation CLI ci-dessus.

