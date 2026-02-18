---
title: "OpenAI"
---

# OpenAI

OpenAI, GPT मॉडल्स के लिए डेवलपर API प्रदान करता है। Codex, **ChatGPT साइन-इन** को सब्सक्रिप्शन के लिए समर्थन करता है।
access or **API key** sign-in for usage-based access. Codex cloud requires ChatGPT sign-in.

## विकल्प A: OpenAI API कुंजी (OpenAI प्लेटफ़ॉर्म)

**किसके लिए सर्वश्रेष्ठ:** सीधे API एक्सेस और उपयोग-आधारित बिलिंग।
Get your API key from the OpenAI dashboard.

### CLI सेटअप

```bash
openclaw onboard --auth-choice openai-api-key
# or non-interactive
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### विन्यास स्निपेट

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

## विकल्प B: OpenAI Code (Codex) सब्सक्रिप्शन

**किसके लिए सर्वश्रेष्ठ:** API key के बजाय ChatGPT/Codex सब्सक्रिप्शन एक्सेस का उपयोग करना।
Codex cloud requires ChatGPT sign-in, while the Codex CLI supports ChatGPT or API key sign-in.

### CLI सेटअप (Codex OAuth)

```bash
# Run Codex OAuth in the wizard
openclaw onboard --auth-choice openai-codex

# Or run OAuth directly
openclaw models auth login --provider openai-codex
```

### विन्यास स्निपेट (Codex सब्सक्रिप्शन)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

## नोट्स

- मॉडल संदर्भ हमेशा `provider/model` का उपयोग करते हैं (देखें [/concepts/models](/concepts/models))।
- प्रमाणीकरण विवरण और पुन: उपयोग नियम [/concepts/oauth](/concepts/oauth) में हैं।


