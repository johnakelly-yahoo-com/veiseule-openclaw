---
summary: "بثّ رسالة WhatsApp إلى عدة وكلاء"
read_when:
  - تهيئة مجموعات البثّ
  - تصحيح أخطاء ردود متعددة الوكلاء في WhatsApp
status: experimental
title: "مجموعات البثّ"
---

# مجموعات البثّ

**الحالة:** تجريبي  
**الإصدار:** أُضيف في 2026.1.9

## نظرة عامة

تُمكّن مجموعات البثّ عدة وكلاء من معالجة الرسالة نفسها والردّ عليها في الوقت نفسه. يتيح لك ذلك إنشاء فرق وكلاء متخصصة تعمل معًا داخل مجموعة WhatsApp واحدة أو محادثة مباشرة — وكل ذلك باستخدام رقم هاتف واحد.

النطاق الحالي: **WhatsApp فقط** (قناة الويب).

يتم تقييم مجموعات البثّ بعد قوائم السماح الخاصة بالقناة وقواعد تفعيل المجموعات. في مجموعات WhatsApp، يعني هذا أن البثّ يحدث عندما يكون OpenClaw سيقوم بالردّ عادةً (على سبيل المثال: عند الذكر، بحسب إعدادات مجموعتك).

## حالات الاستخدام

### 1. فرق وكلاء متخصصة

نشر عدة وكلاء بمهام ذرّية ومركّزة:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

يعالج كل وكيل الرسالة نفسها ويقدّم منظوره المتخصص.

### 2. دعم متعدد اللغات

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. سير عمل ضمان الجودة

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. أتمتة المهام

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## التهيئة

### الإعداد الأساسي

أضِف قسمًا علويًا باسم `broadcast` (بجوار `bindings`). المفاتيح هي مُعرّفات أقران WhatsApp:

- محادثات المجموعات: JID للمجموعة (مثل `120363403215116621@g.us`)
- DM: E.164 رقم الهاتف (على سبيل المثال `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**النتيجة:** عندما يقوم OpenClaw بالردّ في هذه الدردشة، سيشغّل الوكلاء الثلاثة جميعًا.

### استراتيجية المعالجة

التحكم في كيفية معالجة الوكلاء للرسائل:

#### المتوازي (افتراضي)

يعالج جميع الوكلاء الرسالة في الوقت نفسه:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### التسلسلي

يعالج الوكلاء الرسائل بالترتيب (ينتظر كل واحد اكتمال السابق):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### مثال كامل

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## كيف يعمل

### تدفّق الرسائل

1. **رسالة واردة** تصل إلى مجموعة WhatsApp
2. **فحص البثّ**: يتحقق النظام مما إذا كان مُعرّف النظير ضمن `broadcast`
3. **إذا كان ضمن قائمة البثّ**:
   - يعالج جميع الوكلاء المدرجين الرسالة
   - لكل وكيل مفتاح جلسة خاص وسياق معزول
   - تتم المعالجة بالتوازي (افتراضيًا) أو بالتسلسل
4. **إذا لم يكن ضمن قائمة البثّ**:
   - تُطبّق آلية التوجيه العادية (أول ربط مطابق)

ملاحظة: لا تتجاوز مجموعات البثّ قوائم السماح الخاصة بالقناة أو قواعد تفعيل المجموعات (الذكر/الأوامر/إلخ). إنها تغيّر فقط _أيّ الوكلاء يعملون_ عندما تكون الرسالة مؤهلة للمعالجة.

### عزل الجلسات

يحافظ كل وكيل في مجموعة بثّ على فصلٍ كامل لما يلي:

- **مفاتيح الجلسات** (`agent:alfred:whatsapp:group:120363...` مقابل `agent:baerbel:whatsapp:group:120363...`)
- **سجلّ المحادثة** (لا يرى الوكيل رسائل الوكلاء الآخرين)
- **مساحة العمل** (sandboxes منفصلة إذا كانت مهيّأة)
- **الوصول إلى الأدوات** (قوائم سماح/منع مختلفة)
- **الذاكرة/السياق** (ملفات IDENTITY.md و SOUL.md وغيرها منفصلة)
- **مخزن سياق المجموعة** (الرسائل الحديثة للمجموعة المستخدمة كسياق) يكون مشتركًا لكل نظير، لذا يرى جميع وكلاء البثّ السياق نفسه عند التفعيل

يتيح ذلك لكل وكيل أن يمتلك:

- شخصيات مختلفة
- وصولًا مختلفًا إلى الأدوات (مثل: للقراءة فقط مقابل قراءة/كتابة)
- نماذج مختلفة (مثل: opus مقابل sonnet)
- Skills مختلفة مُثبّتة

### مثال: جلسات معزولة

في المجموعة `120363403215116621@g.us` مع الوكلاء `["alfred", "baerbel"]`:

**سياق Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**سياق Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## أفضل الممارسات

### 1. إبقاء الوكلاء مُركّزين

صمّم كل وكيل بمسؤولية واحدة واضحة:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **جيّد:** لكل وكيل مهمة واحدة  
❌ **سيّئ:** وكيل عام واحد باسم "dev-helper"

### 2. استخدام أسماء وصفية

اجعل ما يفعله كل وكيل واضحًا:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. تهيئة وصول مختلف إلى الأدوات

امنح الوكلاء الأدوات التي يحتاجونها فقط:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. مراقبة الأداء

عند وجود عدد كبير من الوكلاء، ضع في الاعتبار:

- استخدام `"strategy": "parallel"` (افتراضيًا) للسرعة
- حصر مجموعات البثّ في 5–10 وكلاء
- استخدام نماذج أسرع للوكلاء الأبسط

### 5. التعامل المرن مع الإخفاقات

يفشل الوكلاء بشكل مستقل. لا يمنع خطأ أحد الوكلاء الآخرين من العمل:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## التوافق

### الموفّرون

تعمل مجموعات البثّ حاليًا مع:

- ✅ WhatsApp (مُنفّذ)
- 🚧 Telegram (مخطّط)
- 🚧 Discord (مخطّط)
- 🚧 Slack (مخطّط)

### التوجيه

تعمل مجموعات البثّ جنبًا إلى جنب مع التوجيه الحالي:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: يردّ alfred فقط (توجيه عادي)
- `GROUP_B`: يردّ agent1 **و** agent2 (بثّ)

**الأولوية:** يأخذ `broadcast` الأسبقية على `bindings`.

## استكشاف الأخطاء وإصلاحها

### عدم استجابة الوكلاء

**تحقق من:**

1. وجود مُعرّفات الوكلاء ضمن `agents.list`
2. تنسيق معرف الند صحيح (مثال: `120363403215116621@g.us`)
3. عدم وجود الوكلاء في قوائم المنع

**تصحيح الأخطاء:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### استجابة وكيل واحد فقط

**السبب:** قد يكون مُعرّف النظير ضمن `bindings` وليس ضمن `broadcast`.

**الحل:** أضِف إلى تهيئة البثّ أو أزِل من الارتباطات.

### مشكلات الأداء

**إذا كان الأداء بطيئًا مع عدد كبير من الوكلاء:**

- تقليل عدد الوكلاء لكل مجموعة
- استخدام نماذج أخفّ (sonnet بدل opus)
- التحقق من زمن بدء تشغيل sandbox

## أمثلة

### المثال 1: فريق مراجعة الشيفرة

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**يرسل المستخدم:** مقتطف شيفرة  
**الردود:**

- code-formatter: «تم إصلاح المسافات البادئة وإضافة تلميحات الأنواع»
- security-scanner: «⚠️ ثغرة حقن SQL في السطر 12»
- test-coverage: «نسبة التغطية 45%، اختبارات حالات الخطأ مفقودة»
- docs-checker: «docstring مفقود للدالة `process_data`»

### المثال 2: دعم متعدد اللغات

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## مرجع API

### مخطط التهيئة

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### الحقول

- `strategy` (اختياري): كيفية معالجة الوكلاء
  - `"parallel"` (افتراضي): يعالج جميع الوكلاء في الوقت نفسه
  - `"sequential"`: يعالج الوكلاء وفق ترتيب المصفوفة
- `[peerId]`: JID لمجموعة WhatsApp، أو رقم E.164، أو مُعرّف نظير آخر
  - القيمة: مصفوفة من مُعرّفات الوكلاء الذين ينبغي أن يعالجوا الرسائل

## القيود

1. **الحد الأقصى للوكلاء:** لا يوجد حدّ صارم، لكن قد يكون 10+ وكلاء بطيئًا
2. **السياق المشترك:** لا يرى الوكلاء ردود بعضهم البعض (حسب التصميم)
3. **ترتيب الرسائل:** قد تصل الردود المتوازية بأي ترتيب
4. **حدود المعدّل:** تُحتسب جميع الوكلاء ضمن حدود معدّل WhatsApp

## تحسينات مستقبلية

ميزات مخطّط لها:

- [ ] وضع السياق المشترك (يرى الوكلاء ردود بعضهم البعض)
- [ ] تنسيق الوكلاء (يمكن للوكلاء إرسال إشارات لبعضهم)
- [ ] اختيار ديناميكي للوكلاء (اختيار الوكلاء بناءً على محتوى الرسالة)
- [ ] أولويات الوكلاء (يردّ بعض الوكلاء قبل الآخرين)

## انظر أيضًا

- [تهيئة متعددة الوكلاء](/tools/multi-agent-sandbox-tools)
- [تهيئة التوجيه](/channels/channel-routing)
- [إدارة الجلسات](/concepts/sessions)

