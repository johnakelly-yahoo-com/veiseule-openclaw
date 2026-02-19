---
summary: "شغّل OpenClaw عبر LiteLLM Proxy للوصول الموحّد إلى النماذج وتتبع التكاليف"
read_when:
  - تريد توجيه OpenClaw عبر وكيل LiteLLM
  - تحتاج إلى تتبع التكاليف أو التسجيل (logging) أو توجيه النماذج عبر LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) هو بوابة LLM مفتوحة المصدر توفّر API موحّدًا لأكثر من 100 مزوّد نماذج. وجّه OpenClaw عبر LiteLLM للحصول على تتبع مركزي للتكاليف والتسجيل، مع مرونة تبديل المزوّدين الخلفيين دون تغيير إعدادات OpenClaw.

## لماذا استخدام LiteLLM مع OpenClaw؟

- **تتبع التكاليف** — اعرف بدقة ما ينفقه OpenClaw عبر جميع النماذج
- **توجيه النماذج** — بدّل بين Claude و GPT-4 و Gemini و Bedrock دون تغيير الإعدادات
- **مفاتيح افتراضية** — أنشئ مفاتيح بحدود إنفاق لـ OpenClaw
- **التسجيل** — سجلات كاملة للطلبات/الاستجابات لأغراض تصحيح الأخطاء
- **البدائل الاحتياطية (Fallbacks)** — تحويل تلقائي عند تعطل المزوّد الأساسي

## بدء سريع

### عبر الإعداد الأولي

```bash
openclaw onboard --auth-choice litellm-api-key
```

### إعداد يدوي

1. ابدأ LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. وجّه OpenClaw إلى LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

هذا كل شيء. يقوم OpenClaw الآن بالتوجيه عبر LiteLLM.

## الإعداد

### متغيرات البيئة

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### ملف الإعداد

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

## المفاتيح الافتراضية

أنشئ مفتاحًا مخصصًا لـ OpenClaw مع حدود للإنفاق:

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

استخدم المفتاح الذي تم إنشاؤه كـ `LITELLM_API_KEY`.

## توجيه النماذج

يمكن لـ LiteLLM توجيه طلبات النماذج إلى واجهات خلفية مختلفة. قم بالإعداد في ملف `config.yaml` الخاص بـ LiteLLM:

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

يستمر OpenClaw في طلب `claude-opus-4-6` — بينما يتولى LiteLLM عملية التوجيه.

## عرض الاستخدام

تحقق من لوحة تحكم LiteLLM أو من خلال API:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## ملاحظات

- يعمل LiteLLM على `http://localhost:4000` بشكل افتراضي
- يتصل OpenClaw عبر نقطة النهاية المتوافقة مع OpenAI وهي `/v1/chat/completions`
- تعمل جميع ميزات OpenClaw من خلال LiteLLM — بدون أي قيود

## انظر أيضًا

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)
