---
summary: "مرجع CLI للأمر `openclaw onboard` (معالج تهيئة أولية تفاعلي)"
read_when:
  - عندما تريد إعدادًا موجّهًا لـ Gateway ومساحة العمل والمصادقة والقنوات وSkills
title: "onboard"
---

# `openclaw onboard`

معالج تهيئة أولية تفاعلي (إعداد Gateway محلي أو عن بُعد).

## الأدلة ذات الصلة

- مركز تهيئة CLI: [معالج التهيئة الأولية (CLI)](/start/wizard)
- نظرة عامة على الإعداد الأولي: [Onboarding Overview](/start/onboarding-overview)
- أتمتة CLI: [أتمتة CLI](/start/wizard-cli-automation)
- مرجع تهيئة CLI: [مرجع تهيئة CLI](/start/wizard-cli-reference)
- التهيئة على macOS: [التهيئة الأولية (تطبيق macOS)](/start/onboarding)

## أمثلة

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

موفّر مخصّص غير تفاعلي:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

يُعد `--custom-api-key` اختياريًا في الوضع غير التفاعلي. إذا تم حذفه، يتحقق الإعداد الأولي من `CUSTOM_API_KEY`.

خيارات نقطة نهاية Z.AI غير التفاعلية:

ملاحظة: يقوم `--auth-choice zai-api-key` الآن باكتشاف أفضل نقطة نهاية Z.AI لمفتاحك تلقائيًا (ويُفضّل واجهة API العامة مع `zai/glm-5`).
إذا كنت تريد تحديدًا نقاط نهاية GLM Coding Plan، فاختر `zai-coding-global` أو `zai-coding-cn`.

```bash
# اختيار نقطة النهاية بدون مطالبة
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# خيارات أخرى لنقطة نهاية Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

ملاحظات التدفق:

- `quickstart`: مطالبات حدّية، مع إنشاء رمز Gateway تلقائيًا.
- `manual`: مطالبات كاملة للمنفذ/الربط/المصادقة (اسم مستعار لـ `advanced`).
- أسرع بدء لأول محادثة: `openclaw dashboard` (واجهة التحكم، دون إعداد قناة).
- موفّر مخصّص: الاتصال بأي نقطة نهاية متوافقة مع OpenAI أو Anthropic،
  بما في ذلك الموفّرين المستضافين غير المدرجين في القائمة. استخدم Unknown للاكتشاف التلقائي.

## أوامر المتابعة الشائعة

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` لا يعني وضعًا غير تفاعلي. استخدم `--non-interactive` للبرامج النصية.
</Note>

