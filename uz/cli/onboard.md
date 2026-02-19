---
summary: "CLI uchun `openclaw onboard` bo‘yicha ma’lumotnoma (interaktiv boshlang‘ich sozlash ustasi)"
read_when:
  - You want guided setup for gateway, workspace, auth, channels, and skills
title: "onboard"
---

# `openclaw onboard`

Interaktiv boshlang‘ich sozlash ustasi (mahalliy yoki masofaviy Gateway sozlamasi).

## Tegishli qo‘llanmalar

- CLI boshlang‘ich sozlash markazi: [Boshlang‘ich sozlash ustasi (CLI)](/start/wizard)
- Onboarding haqida umumiy ma’lumot: [Onboarding Overview](/start/onboarding-overview)
- CLI onboarding reference: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI automation: [CLI Automation](/start/wizard-cli-automation)
- macOS onboarding: [Onboarding (macOS App)](/start/onboarding)

## Examples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Interaktiv bo‘lmagan custom provider:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` non-interactive rejimda ixtiyoriy. Agar ko‘rsatilmasa, onboarding `CUSTOM_API_KEY` ni tekshiradi.

Non-interactive Z.AI endpoint variantlari:

Eslatma: `--auth-choice zai-api-key` endi kalitingiz uchun eng yaxshi Z.AI endpointni avtomatik aniqlaydi (`zai/glm-5` bilan umumiy API’ni afzal ko‘radi).
Agar aynan GLM Coding Plan endpointlarini xohlasangiz, `zai-coding-global` yoki `zai-coding-cn` ni tanlang.

```bash
# Promptsiz endpoint tanlash
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Boshqa Z.AI endpoint variantlari:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow notes:

- `quickstart`: minimal prompts, auto-generates a gateway token.
- `manual`: full prompts for port/bind/auth (alias of `advanced`).
- Fastest first chat: `openclaw dashboard` (Control UI, no channel setup).
- Custom Provider: ro‘yxatda ko‘rsatilmagan host qilingan provayderlar hamda OpenAI yoki Anthropic bilan mos keladigan istalgan endpointga ulaning. Avtomatik aniqlash uchun Unknown dan foydalaning.

## Common follow-up commands

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` does not imply non-interactive mode. Skriptlar uchun `--non-interactive` dan foydalaning.
</Note>
