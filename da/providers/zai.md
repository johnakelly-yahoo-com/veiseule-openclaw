---
title: "Z.AI"
---

# Z.AI

Z.AI er API-platformen for **GLM**-modeller. Det giver REST API'er til GLM og bruger API-nøgler
til godkendelse. Opret din API-nøgle i Z.AI-konsollen. OpenClaw bruger `zai` udbyder
med en Z.AI API-nøgle.

## CLI-opsætning

```bash
openclaw onboard --auth-choice zai-api-key
# or non-interactive
openclaw onboard --zai-api-key "$ZAI_API_KEY"
```

## Konfigurationsudsnit

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } },
}
```

## Noter

- GLM-modeller er tilgængelige som `zai/<model>` (eksempel: `zai/glm-4.7`).
- Se [/providers/glm](/providers/glm) for overblik over modelfamilien.
- Z.AI bruger Bearer-autentificering med din API-nøgle.

