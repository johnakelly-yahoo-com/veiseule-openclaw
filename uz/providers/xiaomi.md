---
title: "Xiaomi MiMo"
---

# Xiaomi MiMo

Xiaomi MiMo — **MiMo** modellari uchun API platformasi. U bilan mos keladigan REST APIlarni taqdim etadi
OpenAI and Anthropic formats and uses API keys for authentication. Create your API key in
the [Xiaomi MiMo console](https://platform.xiaomimimo.com/#/console/api-keys). OpenClaw uses
the `xiaomi` provider with a Xiaomi MiMo API key.

## Model haqida umumiy ma’lumot

- **mimo-v2-flash**: 262144 tokenli kontekst oynasi, Anthropic Messages API bilan mos.
- Asosiy URL: `https://api.xiaomimimo.com/anthropic`
- Avtorizatsiya: `Bearer $XIAOMI_API_KEY`

## CLI sozlamalari

```bash
openclaw onboard --auth-choice xiaomi-api-key
# or non-interactive
openclaw onboard --auth-choice xiaomi-api-key --xiaomi-api-key "$XIAOMI_API_KEY"
```

## Konfiguratsiya namunasi

```json5
{
  env: { XIAOMI_API_KEY: "your-key" },
  agents: { defaults: { model: { primary: "xiaomi/mimo-v2-flash" } } },
  models: {
    mode: "merge",
    providers: {
      xiaomi: {
        baseUrl: "https://api.xiaomimimo.com/anthropic",
        api: "anthropic-messages",
        apiKey: "XIAOMI_API_KEY",
        models: [
          {
            id: "mimo-v2-flash",
            name: "Xiaomi MiMo V2 Flash",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 262144,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Notes

- Model havolasi: `xiaomi/mimo-v2-flash`.
- The provider is injected automatically when `XIAOMI_API_KEY` is set (or an auth profile exists).
- See [/concepts/model-providers](/concepts/model-providers) for provider rules.


