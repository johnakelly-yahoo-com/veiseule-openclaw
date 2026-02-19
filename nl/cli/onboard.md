---
summary: "CLI-referentie voor `openclaw onboard` (interactieve onboardingwizard)"
read_when:
  - Je wilt begeleide installatie voor Gateway, werkruimte, authenticatie, kanalen en Skills
title: "onboard"
---

# `openclaw onboard`

Interactieve onboardingwizard (lokale of externe Gateway-installatie).

## Gerelateerde gidsen

- CLI-onboardinghub: [Onboarding Wizard (CLI)](/start/wizard)
- Onboarding-overzicht: [Onboarding Overview](/start/onboarding-overview)
- CLI-automatisering: [CLI Automation](/start/wizard-cli-automation)
- CLI-onboardingreferentie: [CLI Onboarding Reference](/start/wizard-cli-reference)
- macOS-onboarding: [Onboarding (macOS App)](/start/onboarding)

## Voorbeelden

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Niet-interactieve aangepaste provider:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` is optioneel in niet-interactieve modus. Indien weggelaten controleert onboarding `CUSTOM_API_KEY`.

Niet-interactieve Z.AI endpoint-keuzes:

Opmerking: `--auth-choice zai-api-key` detecteert nu automatisch het beste Z.AI-endpoint voor je sleutel (geeft de voorkeur aan de algemene API met `zai/glm-5`).
Als je specifiek de GLM Coding Plan-endpoints wilt gebruiken, kies dan `zai-coding-global` of `zai-coding-cn`.

```bash
# Endpointselectie zonder prompt
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Andere Z.AI endpoint-keuzes:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow-notities:

- `quickstart`: minimale prompts, genereert automatisch een gateway-token.
- `manual`: volledige prompts voor poort/binding/auth (alias van `advanced`).
- Snelste eerste chat: `openclaw dashboard` (Control UI, geen kanaalconfiguratie).
- Custom Provider: verbind met elk OpenAI- of Anthropic-compatibel endpoint,
  inclusief gehoste providers die niet in de lijst staan. Gebruik Unknown om automatisch te detecteren.

## Gemeenschappelijke follow-upopdrachten

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` impliceert geen niet-interactieve modus. Gebruik `--non-interactive` voor scripts.
</Note>
