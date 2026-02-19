---
summary: "إعداد Hugging Face Inference (المصادقة + اختيار النموذج)"
read_when:
  - تريد استخدام Hugging Face Inference مع OpenClaw
  - تحتاج إلى متغير البيئة لرمز HF أو خيار المصادقة عبر CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) توفر إكمالات الدردشة المتوافقة مع OpenAI من خلال واجهة API موجه واحدة. تحصل على إمكانية الوصول إلى العديد من النماذج (DeepSeek وLlama وغيرها) باستخدام رمز واحد. يستخدم OpenClaw **نقطة النهاية المتوافقة مع OpenAI** (إكمالات الدردشة فقط)؛ ولإنشاء الصور من النص أو embeddings أو الكلام، استخدم [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) مباشرةً.

- المزوّد: `huggingface`
- المصادقة: `HUGGINGFACE_HUB_TOKEN` أو `HF_TOKEN` (رمز دقيق الصلاحيات مع إذن **Make calls to Inference Providers**)
- واجهة API: متوافقة مع OpenAI (`https://router.huggingface.co/v1`)
- الفوترة: رمز HF واحد؛ تعتمد [الأسعار](https://huggingface.co/docs/inference-providers/pricing) على أسعار المزوّدين مع طبقة مجانية.

## البدء السريع

1. أنشئ رمزًا دقيق الصلاحيات من [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) مع إذن **Make calls to Inference Providers**.
2. شغّل الإعداد الأولي واختر **Hugging Face** من قائمة المزوّد المنسدلة، ثم أدخل مفتاح API الخاص بك عند الطلب:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. من قائمة **Default Hugging Face model** المنسدلة، اختر النموذج الذي تريده (يتم تحميل القائمة من Inference API عند توفر رمز صالح؛ وإلا سيتم عرض قائمة مدمجة). يتم حفظ اختيارك كنموذج افتراضي.
4. يمكنك أيضًا تعيين النموذج الافتراضي أو تغييره لاحقًا في الإعدادات:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

سيؤدي ذلك إلى تعيين `huggingface/deepseek-ai/DeepSeek-R1` كنموذج افتراضي.

## ملاحظة حول البيئة

إذا كان Gateway يعمل كخدمة daemon (‏launchd/systemd)، فتأكد من أن `HUGGINGFACE_HUB_TOKEN` أو `HF_TOKEN`
متاحان لتلك العملية (على سبيل المثال، في `~/.openclaw/.env` أو عبر
`env.shellEnv`).

## اكتشاف النماذج وقائمة الاختيار أثناء الإعداد

يكتشف OpenClaw النماذج من خلال استدعاء **نقطة Inference مباشرةً**:

```bash
GET https://router.huggingface.co/v1/models
```

(اختياري: أرسل `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` أو `$HF_TOKEN` للحصول على القائمة الكاملة؛ تعيد بعض نقاط النهاية مجموعة فرعية بدون مصادقة.) الاستجابة تكون بصيغة OpenAI على النحو `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

عند تهيئة مفتاح Hugging Face API (عبر الإعداد الأولي، أو `HUGGINGFACE_HUB_TOKEN`، أو `HF_TOKEN`)، يستخدم OpenClaw طلب GET هذا لاكتشاف نماذج إكمال الدردشة المتاحة. أثناء **الإعداد التفاعلي**، بعد إدخال الرمز الخاص بك سترى قائمة منسدلة **Default Hugging Face model** مُعبأة من تلك القائمة (أو من الكتالوج المدمج إذا فشل الطلب). أثناء التشغيل (مثل بدء تشغيل Gateway)، عند وجود مفتاح، يستدعي OpenClaw مرة أخرى طلب **GET** إلى `https://router.huggingface.co/v1/models` لتحديث الكتالوج. يتم دمج القائمة مع كتالوج مدمج (لبيانات وصفية مثل نافذة السياق والتكلفة). إذا فشل الطلب أو لم يتم تعيين مفتاح، يتم استخدام الكتالوج المدمج فقط.

## أسماء النماذج والخيارات القابلة للتعديل

- **الاسم من API:** يتم **تعبئة اسم عرض النموذج من GET /v1/models** عندما تعيد API الحقول `name` أو `title` أو `display_name`؛ وإلا يتم اشتقاقه من معرف النموذج (على سبيل المثال `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **تجاوز اسم العرض:** يمكنك تعيين تسمية مخصصة لكل نموذج في الإعدادات ليظهر بالشكل الذي تريده في CLI وواجهة المستخدم:

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

- **اختيار المزوّد / السياسة:** أضف لاحقة إلى **معرف النموذج** لتحديد كيفية اختيار الموجّه للواجهة الخلفية:

  - **`:fastest`** — أعلى معدل إنتاجية (يختاره الموجّه؛ اختيار المزوّد **مقفل** — لا يوجد مُحدد تفاعلي للواجهة الخلفية).
  - **`:cheapest`** — أقل تكلفة لكل رمز إخراج (يختاره الموجّه؛ اختيار المزوّد **مقفل**).
  - **`:provider`** — فرض واجهة خلفية محددة (على سبيل المثال `:sambanova`، `:together`).

  عند اختيار **:cheapest** أو **:fastest** (على سبيل المثال في قائمة اختيار النموذج أثناء الإعداد الأولي)، يتم قفل المزوّد: يقرر الموجّه بناءً على التكلفة أو السرعة ولا يتم عرض خطوة اختيار اختيارية لـ "تفضيل واجهة خلفية محددة". يمكنك إضافة هذه كعناصر منفصلة في `models.providers.huggingface.models` أو تعيين `model.primary` مع اللاحقة. يمكنك أيضًا تعيين الترتيب الافتراضي الخاص بك في [إعدادات موفّر الاستدلال](https://hf.co/settings/inference-providers) (بدون لاحقة = استخدام هذا الترتيب).

- **دمج الإعدادات:** يتم الاحتفاظ بالإدخالات الموجودة في `models.providers.huggingface.models` (على سبيل المثال في `models.json`) عند دمج الإعدادات. وبالتالي يتم الاحتفاظ بأي `name` أو `alias` أو خيارات نموذج مخصصة قمت بتعيينها هناك.

## معرفات النماذج وأمثلة الإعداد

تستخدم مراجع النماذج الصيغة `huggingface/<org>/<model>` (معرفات بأسلوب Hub). القائمة أدناه مأخوذة من **GET** `https://router.huggingface.co/v1/models`؛ قد يتضمن الكتالوج لديك المزيد.

**أمثلة على المعرفات (من نقطة نهاية الاستدلال):**

| النموذج                                | المرجع (أضف البادئة `huggingface/`) |
| -------------------------------------- | ------------------------------------------------------ |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                              |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                            |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                        |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                             |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                       |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                    |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                     |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                  |
| GLM 4.7                | `zai-org/GLM-4.7`                                      |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                 |

يمكنك إضافة `:fastest` أو `:cheapest` أو `:provider` (على سبيل المثال `:together`، `:sambanova`) إلى معرّف النموذج. عيّن الترتيب الافتراضي في [إعدادات موفّر الاستدلال](https://hf.co/settings/inference-providers)؛ راجع [موفّري الاستدلال](https://huggingface.co/docs/inference-providers) و **GET** `https://router.huggingface.co/v1/models` للحصول على القائمة الكاملة.

### أمثلة إعداد كاملة

**DeepSeek R1 أساسي مع Qwen كبديل احتياطي:**

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

**Qwen كنموذج افتراضي، مع نسختي :cheapest و :fastest:**

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

**DeepSeek + Llama + GPT-OSS مع أسماء مستعارة:**

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

**فرض مزوّد خلفي محدد باستخدام :provider:**

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

**عدة نماذج Qwen و DeepSeek مع لاحقات السياسات:**

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

