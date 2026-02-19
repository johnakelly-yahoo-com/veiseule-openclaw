---
summary: "استخدام API المتوافق مع OpenAI من NVIDIA في OpenClaw"
read_when:
  - تريد استخدام نماذج NVIDIA في OpenClaw
  - تحتاج إلى إعداد NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

توفر NVIDIA واجهة API متوافقة مع OpenAI على `https://integrate.api.nvidia.com/v1` لنماذج Nemotron و NeMo. قم بالمصادقة باستخدام مفتاح API من [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## إعداد CLI

قم بتصدير المفتاح مرة واحدة، ثم شغّل onboarding وحدد نموذج NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

إذا كنت لا تزال تمرر `--token`، فتذكر أنه سيظهر في سجل shell ومخرجات `ps`؛ يُفضل استخدام متغير البيئة كلما أمكن.

## مقتطف من ملف الإعداد

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## معرّفات النماذج

- `nvidia/llama-3.1-nemotron-70b-instruct` (افتراضي)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## ملاحظات

- نقطة نهاية `/v1` متوافقة مع OpenAI؛ استخدم مفتاح API من NVIDIA NGC.
- يتم تفعيل المزود تلقائيًا عند تعيين `NVIDIA_API_KEY`؛ ويستخدم إعدادات افتراضية ثابتة (نافذة سياق 131,072 توكن، وحد أقصى 4,096 توكن).
