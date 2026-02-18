---
title: "OpenRouter"
---

# OpenRouter

OpenRouter ایک **unified API** فراہم کرتا ہے جو ایک ہی endpoint اور API key کے پیچھے متعدد ماڈلز تک ریکویسٹ روٹ کرتا ہے۔ یہ OpenAI-compatible ہے، اس لیے زیادہ تر OpenAI SDKs بیس URL تبدیل کر کے کام کر لیتے ہیں۔

## CLI سیٹ اپ

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## کنفیگ ٹکڑا

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

## نوٹس

- ماڈل حوالہ جات `openrouter/<provider>/<model>` ہیں۔
- مزید ماڈل/فراہم کنندہ کے اختیارات کے لیے [/concepts/model-providers](/concepts/model-providers) دیکھیں۔
- OpenRouter اندرونی طور پر آپ کی API کلید کے ساتھ Bearer ٹوکن استعمال کرتا ہے۔

