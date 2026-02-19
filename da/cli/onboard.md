---
summary: "CLI-reference for `openclaw onboard` (interaktiv introduktionsguide)"
read_when:
  - Du vil have guidet opsætning af gateway, workspace, autentificering, kanaler og Skills
title: "onboard"
---

# `openclaw onboard`

Interaktiv introduktionsguide (lokal eller fjern Gateway-opsætning).

## Relaterede vejledninger

- CLI-introduktionshub: [Onboarding Wizard (CLI)](/start/wizard)
- CLI-introduktionsreference: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI-automatisering: [CLI Automation](/start/wizard-cli-automation)
- macOS-introduktion: [Onboarding (macOS App)](/start/onboarding)
- macOS-introduktion: [Onboarding (macOS App)](/start/onboarding)

## Eksempler

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Flow-noter:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` er valgfri i ikke-interaktiv tilstand. Hvis udeladt, kontrollerer onboarding `CUSTOM_API_KEY`.

Ikke-interaktive Z.AI endpoint-valg:

Bemærk: `--auth-choice zai-api-key` registrerer nu automatisk det bedste Z.AI endpoint til din nøgle (foretrækker den generelle API med `zai/glm-5`).
Hvis du specifikt ønsker GLM Coding Plan endpoints, skal du vælge `zai-coding-global` eller `zai-coding-cn`.

```bash
# Endpoint-valg uden prompt
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Andre Z.AI endpoint-valg:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow-noter:

- `quickstart`: minimale prompts, genererer automatisk et gateway-token.
- `manual`: fulde prompts for port/bind/autentificering (alias for `advanced`).
- Hurtigste første chat: `openclaw dashboard` (Control UI, ingen kanalopsætning).
- Custom Provider: forbind til et hvilket som helst OpenAI- eller Anthropic-kompatibelt endpoint,
  inklusive hostede udbydere, der ikke er angivet på listen. Brug Unknown for automatisk registrering.

## Almindelige opfølgende kommandoer

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` betyder ikke ikke-interaktiv tilstand. Brug `--non-interactive` til scripts.
</Note>
