---
summary: "Sanggunian ng CLI para sa `openclaw onboard` (interactive na onboarding wizard)"
read_when:
  - Gusto mo ng guided na setup para sa Gateway, workspace, auth, mga channel, at Skills
title: "onboard"
---

# `openclaw onboard`

Interactive na onboarding wizard (lokal o remote na setup ng Gateway).

## Mga Kaugnay na Gabay

- Sentro ng onboarding ng CLI: [Gabayan sa Pagsisimula (CLI)](/start/wizard)
- Sanggunian ng CLI onboarding: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Automation ng CLI: [CLI Automation](/start/wizard-cli-automation)
- Automation ng CLI: [CLI Automation](/start/wizard-cli-automation)
- macOS onboarding: [Onboarding (macOS App)](/start/onboarding)

## Mga halimbawa

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Mga tala sa flow:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

Ang `--custom-api-key` ay opsyonal sa non-interactive mode. Kung hindi ito ibinigay, sinusuri ng onboarding ang `CUSTOM_API_KEY`.

Mga pagpipilian sa non-interactive Z.AI endpoint:

Tandaan: ang `--auth-choice zai-api-key` ay awtomatikong tinutukoy ang pinakamahusay na Z.AI endpoint para sa iyong key (mas pinipili ang general API na may `zai/glm-5`).
Kung partikular mong nais ang GLM Coding Plan endpoints, piliin ang `zai-coding-global` o `zai-coding-cn`.

```bash
# Promptless endpoint selection
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Other Z.AI endpoint choices:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Mga tala sa flow:

- `quickstart`: minimal na mga prompt, awtomatikong bumubuo ng gateway token.
- `manual`: kumpletong mga prompt para sa port/bind/auth (alias ng `advanced`).
- Pinakamabilis na unang chat: `openclaw dashboard` (Control UI, walang setup ng channel).
- Custom Provider: ikonekta ang anumang OpenAI o Anthropic compatible na endpoint,
  kabilang ang mga hosted provider na hindi nakalista. Gamitin ang Unknown para sa auto-detect.

## Mga karaniwang follow-up na command

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
Ang `--json` ay hindi nangangahulugang non-interactive mode. Gamitin ang `--non-interactive` para sa mga script.
</Note>

