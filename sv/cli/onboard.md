---
summary: "CLI-referens för `openclaw onboard` (interaktiv introduktionsguide)"
read_when:
  - Du vill ha guidad konfigurering för gateway, arbetsyta, autentisering, kanaler och Skills
title: "onboard"
---

# `openclaw onboard`

Interaktiv introduktionsguide (lokal eller fjärr-Gateway-konfigurering).

## Relaterad dokumentation

- CLI-introduktionsnav: [Introduktionsguide (CLI)](/start/wizard)
- CLI-introduktionsreferens: [CLI Introduktionsreferens](/start/wizard-cli-reference)
- CLI-automatisering: [CLI Automation](/start/wizard-cli-automation)
- macOS-introduktion: [Introduktion (macOS-app)](/start/onboarding)
- macOS-introduktion: [Introduktion (macOS-app)](/start/onboarding)

## Exempel

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Flödesanteckningar:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` är valfri i icke-interaktivt läge. Om den utelämnas kontrollerar onboarding `CUSTOM_API_KEY`.

Icke-interaktiva val av Z.AI-endpoint:

Obs: `--auth-choice zai-api-key` identifierar nu automatiskt den bästa Z.AI-endpointen för din nyckel (föredrar det generella API:et med `zai/glm-5`).
Om du specifikt vill använda GLM Coding Plan-endpoints, välj `zai-coding-global` eller `zai-coding-cn`.

```bash
# Endpointval utan prompt
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Andra Z.AI-endpointval:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flödesanteckningar:

- `quickstart`: minimala uppmaningar, genererar automatiskt en gateway-token.
- `manual`: fullständiga uppmaningar för port/bind/autentisering (alias till `advanced`).
- Snabbaste första chatten: `openclaw dashboard` (Control UI, ingen kanalinställning).
- Custom Provider: anslut till valfri OpenAI- eller Anthropic-kompatibel endpoint,
  inklusive hostade leverantörer som inte är listade. Använd Unknown för automatisk identifiering.

## Vanliga uppföljande kommandon

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` innebär inte icke-interaktivt läge. Använd `--non-interactive` för skript.
</Note>

