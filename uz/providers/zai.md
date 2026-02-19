---
summary: "OpenClaw bilan Z.AI (GLM modellari) dan foydalaning"
read_when:
  - You want Z.AI / GLM models in OpenClaw
  - You need a simple ZAI_API_KEY setup
title: "Z.AI"
---

# Z.AI

Z.AI is the API platform for **GLM** models. It provides REST APIs for GLM and uses API keys
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
