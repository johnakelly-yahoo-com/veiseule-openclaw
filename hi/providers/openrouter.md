---
title: "OpenRouter"
---

# OpenRouter

OpenRouter एक **एकीकृत API** प्रदान करता है जो अनुरोधों को कई मॉडलों तक एक ही के पीछे रूट करता है
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## CLI सेटअप

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## विन्यास स्निपेट

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

## टिप्पणियाँ

- मॉडल संदर्भ `openrouter/<provider>/<model>` हैं।
- अधिक मॉडल/प्रदाता विकल्पों के लिए, देखें [/concepts/model-providers](/concepts/model-providers)।
- OpenRouter आंतरिक रूप से आपकी API कुंजी के साथ एक बेयरर टोकन का उपयोग करता है।
