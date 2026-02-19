---
summary: "OpenClaw کو vLLM (OpenAI-compatible لوکل سرور) کے ساتھ چلائیں"
read_when:
  - آپ OpenClaw کو لوکل vLLM سرور کے ساتھ چلانا چاہتے ہیں
  - آپ اپنے ماڈلز کے ساتھ OpenAI-compatible `/v1` اینڈ پوائنٹس چاہتے ہیں
title: "vLLM"
---

# vLLM

vLLM ایک **OpenAI-compatible** HTTP API کے ذریعے اوپن سورس (اور کچھ کسٹم) ماڈلز فراہم کر سکتا ہے۔ OpenClaw `openai-completions` API کا استعمال کرتے ہوئے vLLM سے کنیکٹ ہو سکتا ہے۔

OpenClaw `VLLM_API_KEY` کے ساتھ آپٹ اِن کرنے پر (اگر آپ کا سرور auth نافذ نہیں کرتا تو کوئی بھی ویلیو کام کرے گی) اور جب آپ `models.providers.vllm` کی واضح انٹری متعین نہیں کرتے، تو vLLM سے دستیاب ماڈلز کو **auto-discover** بھی کر سکتا ہے۔

## Quick start

1. OpenAI-compatible سرور کے ساتھ vLLM شروع کریں۔

آپ کے base URL کو `/v1` اینڈ پوائنٹس (مثلاً `/v1/models`, `/v1/chat/completions`) ایکسپوز کرنے چاہئیں۔ vLLM عام طور پر اس پر چلتا ہے:

- `http://127.0.0.1:8000/v1`

2. آپٹ اِن کریں (اگر auth کنفیگر نہیں ہے تو کوئی بھی ویلیو کام کرے گی):

```bash
export VLLM_API_KEY="vllm-local"
```

3. ایک ماڈل منتخب کریں (اپنے vLLM ماڈل IDs میں سے کسی ایک سے تبدیل کریں):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## ماڈل دریافت کرنا (implicit provider)

جب `VLLM_API_KEY` سیٹ ہو (یا کوئی auth پروفائل موجود ہو) اور آپ `models.providers.vllm` متعین **نہیں** کرتے، تو OpenClaw یہ کوئری کرے گا:

- `GET http://127.0.0.1:8000/v1/models`

…اور واپس آنے والے IDs کو ماڈل انٹریز میں تبدیل کر دے گا۔

اگر آپ `models.providers.vllm` کو واضح طور پر سیٹ کرتے ہیں تو auto-discovery چھوڑ دی جائے گی اور آپ کو ماڈلز دستی طور پر متعین کرنے ہوں گے۔

## واضح کنفیگریشن (دستی ماڈلز)

واضح کنفیگ استعمال کریں جب:

- vLLM کسی مختلف ہوسٹ/پورٹ پر چل رہا ہو۔
- آپ `contextWindow`/`maxTokens` ویلیوز کو مقرر کرنا چاہتے ہوں۔
- آپ کے سرور کو حقیقی API key درکار ہو (یا آپ ہیڈرز کو کنٹرول کرنا چاہتے ہوں)۔

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## خرابیوں کا ازالہ

- چیک کریں کہ سرور تک رسائی ممکن ہے:

```bash
curl http://127.0.0.1:8000/v1/models
```

- اگر ریکوئسٹس auth کی خرابی کے ساتھ ناکام ہوں، تو ایک حقیقی `VLLM_API_KEY` سیٹ کریں جو آپ کے سرور کی کنفیگریشن سے مطابقت رکھتی ہو، یا provider کو `models.providers.vllm` کے تحت واضح طور پر کنفیگر کریں۔

