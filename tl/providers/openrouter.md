---
title: "OpenRouter"
---

# OpenRouter

Ang OpenRouter ay nagbibigay ng **pinag-isang API** na nagpapadala ng mga kahilingan sa maraming modelo sa likod ng iisang
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## Pag-set up ng CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## Snippet ng config

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },
    },
  },
}
```

## Mga tala

- Ang mga model ref ay `openrouter/<provider>/<model>`.
- Para sa higit pang opsyon sa model/provider, tingnan ang [/concepts/model-providers](/concepts/model-providers).
- Gumagamit ang OpenRouter ng Bearer token kasama ang iyong API key sa ilalim ng hood.
