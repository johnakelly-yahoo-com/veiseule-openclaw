---
summary: "متحدہ ماڈل رسائی اور لاگت کی ٹریکنگ کے لیے OpenClaw کو LiteLLM Proxy کے ذریعے چلائیں"
read_when:
  - آپ OpenClaw کو LiteLLM proxy کے ذریعے روٹ کرنا چاہتے ہیں
  - آپ کو LiteLLM کے ذریعے لاگت کی ٹریکنگ، لاگنگ، یا ماڈل روٹنگ کی ضرورت ہے
---

# LiteLLM

[LiteLLM](https://litellm.ai) ایک اوپن سورس LLM گیٹ وے ہے جو 100+ ماڈل فراہم کنندگان کے لیے ایک متحدہ API فراہم کرتا ہے۔ مرکزی لاگت کی ٹریکنگ، لاگنگ، اور اپنے OpenClaw کنفیگ میں تبدیلی کیے بغیر بیک اینڈ تبدیل کرنے کی سہولت حاصل کرنے کے لیے OpenClaw کو LiteLLM کے ذریعے روٹ کریں۔

## OpenClaw کے ساتھ LiteLLM کیوں استعمال کریں؟

- **لاگت کی ٹریکنگ** — دیکھیں کہ OpenClaw تمام ماڈلز پر بالکل کتنا خرچ کرتا ہے
- **ماڈل روٹنگ** — Claude، GPT-4، Gemini، Bedrock کے درمیان بغیر کنفیگ تبدیلی کے سوئچ کریں
- **ورچوئل کیز** — OpenClaw کے لیے خرچ کی حد کے ساتھ کیز بنائیں
- **لاگنگ** — ڈیبگنگ کے لیے مکمل ریکویسٹ/ریسپانس لاگز
- **فال بیکس** — اگر آپ کا بنیادی فراہم کنندہ دستیاب نہ ہو تو خودکار فیل اوور

## فوری آغاز

### آن بورڈنگ کے ذریعے

```bash
openclaw onboard --auth-choice litellm-api-key
```

### دستی سیٹ اپ

1. LiteLLM Proxy شروع کریں:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. OpenClaw کو LiteLLM کی طرف پوائنٹ کریں:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

بس اتنا ہی۔ OpenClaw اب LiteLLM کے ذریعے روٹنگ کرتا ہے۔

## کنفیگریشن

### ماحولیاتی متغیرات

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### کنفیگ فائل

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## ورچوئل کیز

OpenClaw کے لیے خرچ کی حد کے ساتھ ایک مخصوص کی بنائیں:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

تیار کردہ کی کو `LITELLM_API_KEY` کے طور پر استعمال کریں۔

## ماڈل روٹنگ

LiteLLM ماڈل ریکوئسٹس کو مختلف بیک اینڈز کی طرف روٹ کر سکتا ہے۔ اپنے LiteLLM کے `config.yaml` میں کنفیگر کریں:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw مسلسل `claude-opus-4-6` کی درخواست کرتا رہتا ہے — روٹنگ LiteLLM سنبھالتا ہے۔

## استعمال دیکھنا

LiteLLM کا ڈیش بورڈ یا API چیک کریں:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## نوٹس

- LiteLLM بطور ڈیفالٹ `http://localhost:4000` پر چلتا ہے
- OpenClaw، OpenAI سے مطابقت رکھنے والے `/v1/chat/completions` اینڈپوائنٹ کے ذریعے کنیکٹ ہوتا ہے
- OpenClaw کی تمام خصوصیات LiteLLM کے ذریعے کام کرتی ہیں — کوئی پابندی نہیں

## یہ بھی دیکھیں

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

