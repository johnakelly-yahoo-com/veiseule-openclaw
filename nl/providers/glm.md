---
title: "GLM-modellen"
---

# GLM-modellen

GLM is een **modelfamilie** (geen bedrijf) die beschikbaar is via het Z.AI-platform. In OpenClaw worden GLM
-modellen benaderd via de `zai`-provider en model-ID's zoals `zai/glm-4.7`.

## CLI-installatie

```bash
openclaw onboard --auth-choice zai-api-key
```

## Config-fragment

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## Notities

- GLM-versies en beschikbaarheid kunnen veranderen; raadpleeg de documentatie van Z.AI voor de nieuwste informatie.
- Voorbeeld-model-ID's zijn onder andere `glm-4.7` en `glm-4.6`.
- Voor provider-details, zie [/providers/zai](/providers/zai).

