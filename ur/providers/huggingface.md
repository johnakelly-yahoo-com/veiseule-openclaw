---
summary: "Hugging Face Inference سیٹ اپ (auth + ماڈل کا انتخاب)"
read_when:
  - آپ OpenClaw کے ساتھ Hugging Face Inference استعمال کرنا چاہتے ہیں
  - آپ کو HF ٹوکن env var یا CLI auth انتخاب کی ضرورت ہے
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) ایک واحد router API کے ذریعے OpenAI-compatible چیٹ کمپلیشنز فراہم کرتے ہیں۔ آپ کو ایک ہی ٹوکن کے ساتھ متعدد ماڈلز (DeepSeek، Llama، اور دیگر) تک رسائی حاصل ہوتی ہے۔ OpenClaw **OpenAI-compatible endpoint** (صرف chat completions) استعمال کرتا ہے؛ text-to-image، embeddings، یا speech کے لیے براہِ راست [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) استعمال کریں۔

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` یا `HF_TOKEN` (fine-grained ٹوکن جس میں **Make calls to Inference Providers** کی اجازت ہو)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: واحد HF ٹوکن؛ [pricing](https://huggingface.co/docs/inference-providers/pricing) پرووائیڈر کی ریٹس کے مطابق ہے جس میں ایک مفت درجہ بھی شامل ہے۔

## Quick start

1. [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) پر جا کر **Make calls to Inference Providers** اجازت کے ساتھ ایک fine-grained ٹوکن بنائیں۔
2. آن بورڈنگ چلائیں اور پرووائیڈر ڈراپ ڈاؤن میں **Hugging Face** منتخب کریں، پھر اشارہ ملنے پر اپنی API key درج کریں:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** ڈراپ ڈاؤن میں اپنی مطلوبہ ماڈل منتخب کریں (اگر آپ کے پاس درست ٹوکن ہو تو فہرست Inference API سے لوڈ ہوتی ہے؛ بصورت دیگر ایک بلٹ اِن فہرست دکھائی جاتی ہے)۔ آپ کا انتخاب بطور ڈیفالٹ ماڈل محفوظ کر لیا جاتا ہے۔
4. آپ بعد میں کنفیگ میں ڈیفالٹ ماڈل سیٹ یا تبدیل بھی کر سکتے ہیں:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Non-interactive مثال

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

یہ `huggingface/deepseek-ai/DeepSeek-R1` کو بطور ڈیفالٹ ماڈل سیٹ کر دے گا۔

## Environment نوٹ

اگر Gateway بطور daemon (launchd/systemd) چل رہا ہو، تو یقینی بنائیں کہ `HUGGINGFACE_HUB_TOKEN` یا `HF_TOKEN`
اس پراسیس کے لیے دستیاب ہو (مثال کے طور پر `~/.openclaw/.env` میں یا
`env.shellEnv` کے ذریعے)۔

## Model دریافت اور آن بورڈنگ ڈراپ ڈاؤن

OpenClaw ماڈلز کو براہِ راست **Inference endpoint** کو کال کر کے دریافت کرتا ہے:

```bash
GET https://router.huggingface.co/v1/models
```

(اختیاری: مکمل فہرست کے لیے `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` یا `$HF_TOKEN` بھیجیں؛ کچھ endpoints بغیر auth کے جزوی فہرست واپس کرتے ہیں۔) جواب OpenAI طرز کا `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`۔

جب آپ Hugging Face API key کنفیگر کرتے ہیں (onboarding، `HUGGINGFACE_HUB_TOKEN`، یا `HF_TOKEN` کے ذریعے)، OpenClaw دستیاب chat-completion models دریافت کرنے کے لیے یہ GET استعمال کرتا ہے۔ **interactive onboarding** کے دوران، جب آپ اپنا token درج کرتے ہیں تو آپ کو **Default Hugging Face model** کا dropdown نظر آتا ہے جو اسی فہرست سے بھرا جاتا ہے (یا اگر درخواست ناکام ہو جائے تو built-in کیٹلاگ سے)۔ رن ٹائم پر (مثلاً Gateway startup کے وقت)، جب key موجود ہو تو OpenClaw کیٹلاگ ریفریش کرنے کے لیے دوبارہ **GET** `https://router.huggingface.co/v1/models` کال کرتا ہے۔ فہرست کو ایک built-in کیٹلاگ کے ساتھ ضم کیا جاتا ہے (میٹا ڈیٹا جیسے context window اور لاگت کے لیے)۔ اگر درخواست ناکام ہو جائے یا کوئی key سیٹ نہ ہو تو صرف built-in کیٹلاگ استعمال کیا جاتا ہے۔

## Model names اور قابلِ ترمیم اختیارات

- **Name from API:** ماڈل کا display name **GET /v1/models سے حاصل کیا جاتا ہے** جب API `name`، `title`، یا `display_name` واپس کرے؛ بصورتِ دیگر اسے model id سے اخذ کیا جاتا ہے (مثلاً `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”)۔
- **Override display name:** آپ config میں ہر ماڈل کے لیے ایک کسٹم label سیٹ کر سکتے ہیں تاکہ وہ CLI اور UI میں آپ کی مرضی کے مطابق ظاہر ہو:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / policy selection:** یہ منتخب کرنے کے لیے کہ router backend کیسے چنتا ہے، **model id** کے آخر میں ایک suffix شامل کریں:

  - **`:fastest`** — سب سے زیادہ throughput (router منتخب کرتا ہے؛ provider کا انتخاب **locked** ہوتا ہے — کوئی interactive backend picker نہیں ہوتا)۔
  - **`:cheapest`** — فی output token سب سے کم لاگت (router منتخب کرتا ہے؛ provider کا انتخاب **locked** ہوتا ہے)۔
  - **`:provider`** — کسی مخصوص backend کو لازمی منتخب کریں (مثلاً `:sambanova`، `:together`)۔

  جب آپ **:cheapest** یا **:fastest** منتخب کرتے ہیں (مثلاً onboarding model dropdown میں)، تو provider لاک ہو جاتا ہے: router لاگت یا رفتار کی بنیاد پر فیصلہ کرتا ہے اور کوئی اختیاری “prefer specific backend” مرحلہ دکھایا نہیں جاتا۔ آپ انہیں `models.providers.huggingface.models` میں علیحدہ entries کے طور پر شامل کر سکتے ہیں یا `model.primary` کو suffix کے ساتھ سیٹ کر سکتے ہیں۔ آپ [Inference Provider settings](https://hf.co/settings/inference-providers) میں اپنی default ترتیب بھی سیٹ کر سکتے ہیں (کوئی suffix نہیں = وہی ترتیب استعمال ہوگی)۔

- **Config merge:** جب config کو merge کیا جاتا ہے تو `models.providers.huggingface.models` میں موجود entries (مثلاً `models.json` میں) برقرار رکھی جاتی ہیں۔ لہٰذا وہاں سیٹ کیے گئے کوئی بھی کسٹم `name`، `alias`، یا model options محفوظ رہتے ہیں۔

## Model IDs اور کنفیگریشن کی مثالیں

Model refs اس فارمیٹ میں ہوتے ہیں `huggingface/<org>/<model>` (Hub طرز کے IDs)۔ نیچے دی گئی فہرست **GET** `https://router.huggingface.co/v1/models` سے ہے؛ آپ کے کیٹلاگ میں مزید ماڈلز شامل ہو سکتے ہیں۔

**مثالی IDs (inference endpoint سے):**

| Model                                  | Ref (آگے `huggingface/` شامل کریں) |
| -------------------------------------- | ----------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                             |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                           |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                       |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                            |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                      |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                   |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                    |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                 |
| GLM 4.7                | `zai-org/GLM-4.7`                                     |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                |

آپ ماڈل آئی ڈی کے ساتھ `:fastest`، `:cheapest`، یا `:provider` (مثلاً `:together`، `:sambanova`) شامل کر سکتے ہیں۔ [Inference Provider settings](https://hf.co/settings/inference-providers) میں اپنی ڈیفالٹ ترتیب مقرر کریں؛ مکمل فہرست کے لیے [Inference Providers](https://huggingface.co/docs/inference-providers) اور **GET** `https://router.huggingface.co/v1/models` دیکھیں۔

### مکمل کنفیگریشن کی مثالیں

**Primary DeepSeek R1 with Qwen fallback:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen بطور ڈیفالٹ، :cheapest اور :fastest ویریئنٹس کے ساتھ:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS عرفی ناموں کے ساتھ:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**:provider کے ساتھ مخصوص بیک اینڈ کو لازمی منتخب کریں:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**پالیسی سفکس کے ساتھ متعدد Qwen اور DeepSeek ماڈلز:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```
