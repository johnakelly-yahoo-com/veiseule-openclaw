---
title: "hooks"
---

# `openclaw hooks`

إدارة خطافات الوكيل (أتمتة قائمة على الأحداث لأوامر مثل `/new` و`/reset` وبدء تشغيل الـ Gateway).

ذو صلة:

- الخطافات: [Hooks](/automation/hooks)
- خطافات الإضافات: [Plugins](/tools/plugin#plugin-hooks)

## قائمة جميع الروابط

```bash
openclaw hooks list
```

يعرض جميع الخطافات المكتشفة من مجلدات مساحة العمل والمُدارة والمُضمَّنة.

**الخيارات:**

- `--eligible`: إظهار الخطافات المؤهلة فقط (المتطلبات مستوفاة)
- `--json`: الإخراج بصيغة JSON
- `-v, --verbose`: إظهار معلومات تفصيلية بما في ذلك المتطلبات المفقودة

**مثال على الإخراج:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**مثال (تفصيلي):**

```bash
openclaw hooks list --verbose
```

يعرض المتطلبات المفقودة للخطافات غير المؤهلة.

**مثال (JSON):**

```bash
openclaw hooks list --json
```

يعيد JSON مُنظَّمًا للاستخدام البرمجي.

## الحصول على معلومات الخطاف

```bash
openclaw hooks info <name>
```

يعرض معلومات تفصيلية حول خطاف معيّن.

**المعاملات:**

- `<name>`: اسم الخطاف (مثل `session-memory`)

**الخيارات:**

- `--json`: الإخراج بصيغة JSON

**مثال:**

```bash
openclaw hooks info session-memory
```

**الإخراج:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## التحقق من أهلية الخطافات

```bash
openclaw hooks check
```

يعرض ملخص حالة أهلية الخطافات (عدد الجاهز مقابل غير الجاهز).

**الخيارات:**

- `--json`: الإخراج بصيغة JSON

**مثال على الإخراج:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## تمكين خطاف

```bash
openclaw hooks enable <name>
```

تمكين خطاف معيّن عبر إضافته إلى التهيئة الخاصة بك (`~/.openclaw/config.json`).

**ملاحظة:** الخطافات المُدارة بواسطة الإضافات تُظهر `plugin:<id>` في `openclaw hooks list` ولا يمكن تمكينها أو تعطيلها من هنا. بدلاً من ذلك، قم بتمكين أو تعطيل الإضافة.

**المعاملات:**

- `<name>`: اسم الخطاف (مثل `session-memory`)

**مثال:**

```bash
openclaw hooks enable session-memory
```

**الإخراج:**

```
✓ Enabled hook: 💾 session-memory
```

**ما الذي يفعله:**

- يتحقق من وجود الخطاف وأنه مؤهل
- يُحدِّث `hooks.internal.entries.<name>.enabled = true` في التهيئة الخاصة بك
- يحفظ التهيئة على القرص

**بعد التمكين:**

- أعد تشغيل الـ Gateway لإعادة تحميل الخطافات (إعادة تشغيل تطبيق شريط القائمة على macOS، أو إعادة تشغيل عملية الـ Gateway في وضع التطوير).

## تعطيل خطاف

```bash
openclaw hooks disable <name>
```

تعطيل خطاف معيّن عبر تحديث التهيئة الخاصة بك.

**المعاملات:**

- `<name>`: اسم الخطاف (مثل `command-logger`)

**مثال:**

```bash
openclaw hooks disable command-logger
```

**الإخراج:**

```
⏸ Disabled hook: 📝 command-logger
```

**بعد التعطيل:**

- أعد تشغيل الـ Gateway لإعادة تحميل الخطافات

## تثبيت الخطافات

```bash
openclaw hooks install <path-or-spec>
```

تثبيت حزمة خطافات من مجلد/أرشيف محلي أو من npm.

**ما الذي يفعله:**

- ينسخ حزمة الخطافات إلى `~/.openclaw/hooks/<id>`
- يفعّل الخطافات المثبّتة في `hooks.internal.entries.*`
- يسجّل عملية التثبيت ضمن `hooks.internal.installs`

**الخيارات:**

- `-l, --link`: ربط دليل محلي بدلاً من النسخ (يضيفه إلى `hooks.internal.load.extraDirs`)

**الأرشيفات المدعومة:** `.zip` و`.tgz` و`.tar.gz` و`.tar`

**أمثلة:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## تحديث الخطافات

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

تحديث حِزم الخطافات المثبّتة (تثبيتات npm فقط).

**الخيارات:**

- `--all`: تحديث جميع حِزم الخطافات المتعقَّبة
- `--dry-run`: إظهار ما الذي سيتغيّر دون الكتابة

## الخطافات المُضمَّنة

### session-memory

يحفظ سياق الجلسة في الذاكرة عند إصدارك `/new`.

**التمكين:**

```bash
openclaw hooks enable session-memory
```

**الإخراج:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**انظر:** [توثيق session-memory](/automation/hooks#session-memory)

### command-logger

يسجّل جميع أحداث الأوامر في ملف تدقيق مركزي.

**التمكين:**

```bash
openclaw hooks enable command-logger
```

**الإخراج:** `~/.openclaw/logs/commands.log`

**عرض السجلات:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**انظر:** [توثيق command-logger](/automation/hooks#command-logger)

### soul-evil

يستبدل محتوى `SOUL.md` المُحقَن بمحتوى `SOUL_EVIL.md` أثناء نافذة تطهير أو باحتمال عشوائي.

**التمكين:**

```bash
openclaw hooks enable soul-evil
```

**انظر:** [SOUL Evil Hook](/hooks/soul-evil)

### boot-md

يشغّل `BOOT.md` عند بدء تشغيل الـ Gateway (بعد بدء القنوات).

**الأحداث**: `gateway:startup`

**التمكين**:

```bash
openclaw hooks enable boot-md
```

**انظر:** [توثيق boot-md](/automation/hooks#boot-md)
