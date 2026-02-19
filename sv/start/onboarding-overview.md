---
summary: "Översikt över OpenClaw onboarding-alternativ och flöden"
read_when:
  - Välja en onboarding-väg
  - Konfigurera en ny miljö
title: "Onboarding-översikt"
sidebarTitle: "Onboarding-översikt"
---

# Onboarding-översikt

OpenClaw stöder flera onboarding-vägar beroende på var Gateway körs
och hur du föredrar att konfigurera leverantörer.

## Välj din onboarding-väg

- **CLI-guide** för macOS, Linux och Windows (via WSL2).
- **macOS-app** för en guidad första start på Apple silicon- eller Intel-Macar.

## CLI-onboardingguide

Kör guiden i en terminal:

```bash
openclaw onboard
```

Använd CLI-guiden när du vill ha full kontroll över Gateway, workspace,
kanaler och skills. Dokumentation:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard`-kommandot](/cli/onboard)

## Onboarding via macOS-app

Använd OpenClaw-appen när du vill ha en helt guidad installation på macOS. Dokumentation:

- [Onboarding (macOS App)](/start/onboarding)

## Anpassad leverantör

Om du behöver en endpoint som inte finns listad, inklusive hostade leverantörer som exponerar standard OpenAI- eller Anthropic-API:er, välj **Custom Provider** i CLI-guiden. Du kommer att bli ombedd att:

- Välj OpenAI-kompatibel, Anthropic-kompatibel eller **Unknown** (automatisk identifiering).
- Ange en bas-URL och API-nyckel (om det krävs av leverantören).
- Ange ett modell-ID och ett valfritt alias.
- Välj ett Endpoint ID så att flera anpassade endpoints kan samexistera.

För detaljerade steg, följ CLI-onboardingdokumentationen ovan.

