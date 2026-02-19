---
summary: "Overzicht van OpenClaw-onboardingopties en -flows"
read_when:
  - Een onboardingpad kiezen
  - Een nieuwe omgeving instellen
title: "Onboardingoverzicht"
sidebarTitle: "Onboardingoverzicht"
---

# Onboardingoverzicht

OpenClaw ondersteunt meerdere onboardingpaden, afhankelijk van waar de Gateway draait
en hoe je providers wilt configureren.

## Kies je onboardingpad

- **CLI-wizard** voor macOS, Linux en Windows (via WSL2).
- **macOS-app** voor een begeleide eerste uitvoering op Apple silicon- of Intel-Macs.

## CLI-onboardingwizard

Voer de wizard uit in een terminal:

```bash
openclaw onboard
```

Gebruik de CLI-wizard wanneer je volledige controle wilt over de Gateway, workspace,
kanalen en skills. Documentatie:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## Onboarding via de macOS-app

Gebruik de OpenClaw-app wanneer je een volledig begeleide installatie op macOS wilt. Documentatie:

- [Onboarding (macOS App)](/start/onboarding)

## Aangepaste provider

Als je een endpoint nodig hebt dat niet in de lijst staat, inclusief gehoste providers die standaard OpenAI- of Anthropic-API’s aanbieden, kies dan **Custom Provider** in de CLI-wizard. Je wordt gevraagd om:

- Kies OpenAI-compatibel, Anthropic-compatibel of **Unknown** (automatisch detecteren).
- Voer een base URL en API-sleutel in (indien vereist door de provider).
- Geef een model-ID en optionele alias op.
- Kies een Endpoint ID zodat meerdere aangepaste endpoints naast elkaar kunnen bestaan.

Volg voor gedetailleerde stappen de bovenstaande CLI-onboardingdocumentatie.

