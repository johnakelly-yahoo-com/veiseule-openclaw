---
summary: "إعداد Together AI (المصادقة + اختيار النموذج)"
read_when:
  - تريد استخدام Together AI مع OpenClaw
  - تحتاج إلى متغير بيئة مفتاح API أو خيار مصادقة عبر CLI
---

# Together AI

توفر [Together AI](https://together.ai) إمكانية الوصول إلى نماذج مفتوحة المصدر رائدة بما في ذلك Llama وDeepSeek وKimi والمزيد من خلال API موحدة.

- المزود: `together`
- المصادقة: `TOGETHER_API_KEY`
- API: متوافق مع OpenAI

## البدء السريع

1. قم بتعيين مفتاح API (يوصى بتخزينه للـ Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. تعيين نموذج افتراضي:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## مثال غير تفاعلي

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

سيؤدي ذلك إلى تعيين `together/moonshotai/Kimi-K2.5` كنموذج افتراضي.

## ملاحظة حول البيئة

إذا كان Gateway يعمل كخدمة daemon (launchd/systemd)، فتأكد من أن `TOGETHER_API_KEY`
متاح لتلك العملية (على سبيل المثال، في `~/.clawdbot/.env` أو عبر
`env.shellEnv`).

## النماذج المتاحة

توفر Together AI إمكانية الوصول إلى العديد من النماذج مفتوحة المصدر الشائعة:

- **GLM 4.7 Fp8** - النموذج الافتراضي مع نافذة سياق 200K
- **Llama 3.3 70B Instruct Turbo** - تنفيذ سريع وفعّال للتعليمات
- **Llama 4 Scout** - نموذج رؤية مع فهم للصور
- **Llama 4 Maverick** - رؤية واستدلال متقدم
- **DeepSeek V3.1** - نموذج قوي للبرمجة والاستدلال
- **DeepSeek R1** - نموذج استدلال متقدم
- **Kimi K2 Instruct** - نموذج عالي الأداء مع نافذة سياق 262K

تدعم جميع النماذج إكمال الدردشة القياسي وهي متوافقة مع OpenAI API.
