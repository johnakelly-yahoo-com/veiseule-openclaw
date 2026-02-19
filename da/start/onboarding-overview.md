---
summary: "Oversigt over OpenClaw onboarding-muligheder og -forløb"
read_when:
  - Valg af onboarding-sti
  - Opsætning af et nyt miljø
title: "Onboarding-oversigt"
sidebarTitle: "Onboarding-oversigt"
---

# Onboarding-oversigt

OpenClaw understøtter flere onboarding-stier afhængigt af, hvor Gateway kører,
og hvordan du foretrækker at konfigurere udbydere.

## Vælg din onboarding-sti

- **CLI-guide** til macOS, Linux og Windows (via WSL2).
- **macOS-app** til en guidet første opstart på Apple silicon eller Intel Macs.

## CLI onboarding-guide

Kør guiden i en terminal:

```bash
openclaw onboard
```

Brug CLI-guiden, når du ønsker fuld kontrol over Gateway, workspace,
kanaler og skills. Dokumentation:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard`-kommando](/cli/onboard)

## macOS app onboarding

Brug OpenClaw-appen, når du ønsker en fuldt guidet opsætning på macOS. Dokumentation:

- [Onboarding (macOS App)](/start/onboarding)

## Brugerdefineret udbyder

Hvis du har brug for et endpoint, der ikke er angivet, herunder hostede udbydere der eksponerer standard OpenAI- eller Anthropic-API’er, skal du vælge **Custom Provider** i CLI-guiden. Du vil blive bedt om at:

- Vælge OpenAI-kompatibel, Anthropic-kompatibel eller **Unknown** (automatisk registrering).
- Indtaste en base-URL og API-nøgle (hvis krævet af udbyderen).
- Angive et model-ID og et valgfrit alias.
- Vælge et Endpoint ID, så flere brugerdefinerede endpoints kan sameksistere.

Følg CLI-onboarding-dokumentationen ovenfor for detaljerede trin.
