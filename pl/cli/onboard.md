---
summary: "Referencja CLI dla `openclaw onboard` (interaktywny kreator wdrażania)"
read_when:
  - Chcesz przejść przez konfigurację z przewodnikiem dla Gateway, obszaru roboczego, uwierzytelniania, kanałów i Skills
title: "onboard"
---

# `openclaw onboard`

Interaktywny kreator wdrażania (lokalna lub zdalna konfiguracja Gateway).

## Powiązane przewodniki

- Centrum wdrażania CLI: [Onboarding Wizard (CLI)](/start/wizard)
- Przegląd onboardingu: [Onboarding Overview](/start/onboarding-overview)
- Automatyzacja CLI: [CLI Automation](/start/wizard-cli-automation)
- Referencja wdrażania CLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Wdrażanie na macOS: [Onboarding (macOS App)](/start/onboarding)

## Examples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Niestandardowy dostawca w trybie nieinteraktywnym:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` jest opcjonalne w trybie nieinteraktywnym. Jeśli pominięte, onboarding sprawdza `CUSTOM_API_KEY`.

Wybory endpointów Z.AI w trybie nieinteraktywnym:

Uwaga: `--auth-choice zai-api-key` automatycznie wykrywa teraz najlepszy endpoint Z.AI dla Twojego klucza (preferuje ogólne API z `zai/glm-5`).
Jeśli konkretnie chcesz endpointy GLM Coding Plan, wybierz `zai-coding-global` lub `zai-coding-cn`.

```bash
# Wybór endpointu bez promptu
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Inne opcje endpointu Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow notes:

- `quickstart`: minimalne monity, automatycznie generuje token gateway.
- `manual`: pełne monity dla portu/bindowania/uwierzytelniania (alias `advanced`).
- Najszybszy pierwszy czat: `openclaw dashboard` (UI sterujące, bez konfiguracji kanałów).
- Niestandardowy dostawca: połącz z dowolnym endpointem zgodnym z OpenAI lub Anthropic,
  w tym z hostowanymi dostawcami niewymienionymi na liście. Użyj Unknown, aby wykryć automatycznie.

## Common follow-up commands

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` nie oznacza trybu nieinteraktywnego. Do skryptów użyj `--non-interactive`.
</Note>
