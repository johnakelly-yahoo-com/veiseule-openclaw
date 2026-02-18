---
title: "Z.AI"
---

# Z.AI

Z.AI — **GLM** modellari uchun API platformasi. U GLM uchun REST APIlar taqdim etadi va API kalitlaridan foydalanadi.
for authentication. Create your API key in the Z.AI console. OpenClaw uses the `zai` provider
with a Z.AI API key.

## CLI sozlash

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Config namunasi

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## Eslatmalar

- GLM models are available as `zai/<model>` (example: `zai/glm-4.7`).
- Model oilasi haqida umumiy ma’lumot uchun [/providers/glm](/providers/glm) ga qarang.
- Z.AI API kalitingiz bilan Bearer auth dan foydalanadi.
