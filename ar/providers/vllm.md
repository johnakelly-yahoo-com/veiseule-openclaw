---
summary: "تشغيل OpenClaw مع vLLM (خادم محلي متوافق مع OpenAI)"
read_when:
  - تريد تشغيل OpenClaw مقابل خادم vLLM محلي
  - تريد نقاط نهاية /v1 متوافقة مع OpenAI باستخدام نماذجك الخاصة
title: "vLLM"
---

# vLLM

يمكن لـ vLLM خدمة النماذج مفتوحة المصدر (وبعض النماذج المخصصة) عبر واجهة HTTP API **متوافقة مع OpenAI**. يمكن لـ OpenClaw الاتصال بـ vLLM باستخدام واجهة API ‏`openai-completions`.

يمكن لـ OpenClaw أيضًا **الاكتشاف التلقائي** للنماذج المتاحة من vLLM عند تفعيل ذلك باستخدام `VLLM_API_KEY` (أي قيمة تعمل إذا كان خادمك لا يفرض المصادقة) وعدم تعريف إدخال صريح `models.providers.vllm`.

## البدء السريع

1. ابدأ تشغيل vLLM باستخدام خادم متوافق مع OpenAI.

يجب أن يكشف عنوان URL الأساسي الخاص بك عن نقاط نهاية `/v1` (مثل `/v1/models` و `/v1/chat/completions`). يعمل vLLM عادةً على:

- `http://127.0.0.1:8000/v1`

2. فعّل (أي قيمة تعمل إذا لم يتم إعداد مصادقة):

```bash
export VLLM_API_KEY="vllm-local"
```

3. اختر نموذجًا (استبدل بأحد معرّفات نماذج vLLM لديك):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## اكتشاف النماذج (موفّر ضمني)

عند تعيين `VLLM_API_KEY` (أو وجود ملف تعريف مصادقة) وعدم تعريف `models.providers.vllm`، سيقوم OpenClaw بالاستعلام عن:

- `GET http://127.0.0.1:8000/v1/models`

…ثم تحويل المعرّفات المُعادة إلى إدخالات نماذج.

إذا قمت بتعيين `models.providers.vllm` صراحةً، فسيتم تخطي الاكتشاف التلقائي ويجب عليك تعريف النماذج يدويًا.

## تكوين صريح (نماذج يدوية)

استخدم التكوين الصريح عندما:

- يعمل vLLM على مضيف/منفذ مختلف.
- تريد تثبيت قيم `contextWindow`/`maxTokens`.
- يتطلب خادمك مفتاح API حقيقيًا (أو ترغب في التحكم في الرؤوس).

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

## استكشاف الأخطاء وإصلاحها

- تحقق من أن الخادم يمكن الوصول إليه:

```bash
curl http://127.0.0.1:8000/v1/models
```

- إذا فشلت الطلبات بسبب أخطاء مصادقة، فقم بتعيين `VLLM_API_KEY` حقيقي يتطابق مع إعداد خادمك، أو قم بتكوين الموفّر صراحةً ضمن `models.providers.vllm`.

