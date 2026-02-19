---
summary: "مرجع CLI لأمر `openclaw plugins` (السرد، التثبيت، التمكين/التعطيل، الفحص)"
read_when:
  - تريد تثبيت أو إدارة إضافات Gateway التي تُحمَّل داخل العملية
  - تريد تصحيح أخطاء تحميل البرنامج المساعد
title: "الإضافات"
---

# `openclaw plugins`

إدارة إضافات/امتدادات Gateway (تُحمَّل داخل العملية).

ذو صلة:

- نظام الإضافات: [Plugins](/tools/plugin)
- بيان الإضافة + المخطط: [Plugin manifest](/plugins/manifest)
- تعزيز الأمان: [Security](/gateway/security)

## الأوامر

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

تأتي الإضافات المضمّنة مع OpenClaw لكنها تبدأ معطّلة. استخدم `plugins enable` لـ
تفعيلها.

يجب على جميع الإضافات تضمين ملف `openclaw.plugin.json` مع مخطط JSON مضمن
(`configSchema`، حتى لو كان فارغًا). تؤدي البيانات الوصفية أو المخططات المفقودة/غير الصالحة إلى
منع تحميل الإضافة وفشل التحقق من التهيئة.

### التثبيت

```bash
openclaw plugins install <path-or-spec>
```

ملاحظة أمنية: تعامل مع تثبيت الإضافات كما لو كنت تُشغّل شيفرة. يُفضَّل استخدام إصدارات مُثبّتة.

مواصفات Npm هي **registry-only** (اسم الحزمة + إصدار/وسم اختياري). يتم رفض
مواصفات Git/URL/file. يتم تثبيت الاعتماديات باستخدام `--ignore-scripts` لأسباب تتعلق بالأمان.

الأرشيفات المدعومة: `.zip`، `.tgz`، `.tar.gz`، `.tar`.

استخدم `--link` لتجنّب نسخ دليل محلي (يُضاف إلى `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### إلغاء التثبيت

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

يقوم `uninstall` بإزالة سجلات الإضافة من `plugins.entries` و`plugins.installs`،
وقائمة السماح الخاصة بالإضافة، وإدخالات `plugins.load.paths` المرتبطة عند الاقتضاء.
بالنسبة لإضافات الذاكرة النشطة، تتم إعادة تعيين فتحة الذاكرة إلى `memory-core`.

افتراضيًا، يؤدي إلغاء التثبيت أيضًا إلى إزالة دليل تثبيت الإضافة ضمن
جذر امتدادات دليل الحالة النشط (`$OPENCLAW_STATE_DIR/extensions/<id>`). استخدم
`--keep-files` للاحتفاظ بالملفات على القرص.

`--keep-config` مدعوم كاسم بديل مهمل لـ `--keep-files`.

### التحديث

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

تنطبق التحديثات فقط على الإضافات المُثبّتة من npm (المتتبَّعة في `plugins.installs`).

