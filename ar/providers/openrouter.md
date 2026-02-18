---
title: "OpenRouter"
---

# OpenRouter

يوفّر OpenRouter **واجهة برمجة تطبيقات موحّدة** تقوم بتوجيه الطلبات إلى العديد من النماذج خلف نقطة نهاية واحدة ومفتاح API واحد. وهو متوافق مع OpenAI، لذا تعمل معظم حِزم SDK الخاصة بـ OpenAI عبر تبديل عنوان URL الأساسي.

## إعداد CLI

```bash
openclaw onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## مقتطف إعدادات

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

## ملاحظات

- مراجع النماذج هي `openrouter/<provider>/<model>`.
- لمزيد من خيارات النماذج/الموفّرين، راجع [/concepts/model-providers](/concepts/model-providers).
- يستخدم OpenRouter رمز Bearer مع مفتاح API الخاص بك خلف الكواليس.

