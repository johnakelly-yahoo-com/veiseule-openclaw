---
title: "إنشاء Skills"
---

# إنشاء Skills مخصّصة 🛠

تم تصميم OpenClaw ليكون سهل التوسعة. تُعد «Skills» الطريقة الأساسية لإضافة قدرات جديدة إلى مساعدك.

## ما هي Skill؟

الـ Skill هي دليل يحتوي على ملف `SKILL.md` (يوفّر تعليمات وتعريفات الأدوات إلى نموذج اللغة الكبير) ويمكن أن يتضمن اختياريًا بعض السكربتات أو الموارد.

## خطوة بخطوة: Skillك الأولى

### 1. إنشاء الدليل

توجد Skills ضمن مساحة عملك، وعادةً في `~/.openclaw/workspace/skills/`. أنشئ مجلدًا جديدًا لـ Skillك:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

### 2. تعريف `SKILL.md`

أنشئ ملف `SKILL.md` داخل ذلك الدليل. يستخدم هذا الملف واجهة YAML أمامية للبيانات الوصفية وMarkdown للتعليمات.

```markdown
---
name: hello_world
description: A simple skill that says hello.
---

# Hello World Skill

When the user asks for a greeting, use the `echo` tool to say "Hello from your custom skill!".
```

### 3. إضافة أدوات (اختياري)

يمكنك تعريف أدوات مخصّصة في الواجهة الأمامية أو توجيه الوكيل لاستخدام أدوات النظام الموجودة (مثل `bash` أو `browser`).

### 4. تحديث OpenClaw

اطلب من الوكيل «تحديث Skills» أو أعد تشغيل Gateway (البوابة). سيكتشف OpenClaw الدليل الجديد ويفهرس `SKILL.md`.

## أفضل الممارسات

- **كن موجزًا**: وجّه النموذج إلى _ما_ الذي يجب فعله، لا كيف يكون ذكاءً اصطناعيًا.
- **السلامة أولًا**: إذا كانت Skillك تستخدم `bash`، فتأكّد من أن المطالبات لا تسمح بحقن أوامر عشوائية من إدخال مستخدم غير موثوق.
- **الاختبار محليًا**: استخدم `openclaw agent --message "use my new skill"` للاختبار.

## Skills المشتركة

يمكنك أيضًا استعراض Skills والمساهمة بها عبر [ClawHub](https://clawhub.com).

