---
summary: "Reference CLI pour `openclaw onboard` (assistant de prise en main interactif)"
read_when:
  - Vous souhaitez une configuration guidee pour la passerelle, l'espace de travail, l'authentification, les canaux et les Skills
title: "onboard"
---

# `openclaw onboard`

Assistant de prise en main interactif (configuration de la Gateway (passerelle) locale ou distante).

## Guides associes

- Hub de prise en main CLI : [Onboarding Wizard (CLI)](/start/wizard)
- Présentation de l’onboarding : [Onboarding Overview](/start/onboarding-overview)
- Automatisation CLI : [CLI Automation](/start/wizard-cli-automation)
- Reference de prise en main CLI : [CLI Onboarding Reference](/start/wizard-cli-reference)
- Prise en main macOS : [Onboarding (macOS App)](/start/onboarding)

## Exemples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Fournisseur personnalisé non interactif :

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` est optionnel en mode non interactif. S’il est omis, l’onboarding vérifie `CUSTOM_API_KEY`.

Choix d’endpoint Z.AI en mode non interactif :

Remarque : `--auth-choice zai-api-key` détecte désormais automatiquement le meilleur endpoint Z.AI pour votre clé (privilégie l’API générale avec `zai/glm-5`).
Si vous souhaitez spécifiquement les endpoints GLM Coding Plan, choisissez `zai-coding-global` ou `zai-coding-cn`.

```bash
# Sélection d’endpoint sans invite
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Autres choix d’endpoint Z.AI :
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Notes de flux :

- `quickstart` : invites minimales, genere automatiquement un jeton de passerelle.
- `manual` : invites completes pour port/liaison/authentification (alias de `advanced`).
- Premier chat le plus rapide : `openclaw dashboard` (UI de controle, aucune configuration de canal).
- Fournisseur personnalisé : connectez n’importe quel endpoint compatible OpenAI ou Anthropic,
  y compris des fournisseurs hébergés non listés. Utilisez Unknown pour la détection automatique.

## Commandes de suivi courantes

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` n'implique pas un mode non interactif. Utilisez `--non-interactive` pour les scripts.
</Note>

