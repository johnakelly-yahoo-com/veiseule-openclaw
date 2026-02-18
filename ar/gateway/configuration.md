---
title: "التهيئة"
---

# التهيئة 🔧

يقرأ OpenClaw ملف تهيئة **JSON5** اختياريًا من `~/.openclaw/openclaw.json` (تُسمح التعليقات والفواصل اللاحقة).

إذا كان الملف مفقودًا، يستخدم OpenClaw إعدادات افتراضية «آمنة إلى حدٍّ ما» (وكيل Pi مُضمَّن + جلسات لكل مُرسِل + مساحة عمل `~/.openclaw/workspace`). عادةً لا تحتاج إلى ملف تهيئة إلا من أجل:

- تقييد من يمكنه تشغيل البوت (`channels.whatsapp.allowFrom`، `channels.telegram.allowFrom`، إلخ)
- التحكّم في قوائم السماح للمجموعات وسلوك الإشارات (`channels.whatsapp.groups`، `channels.telegram.groups`، `channels.discord.guilds`، `agents.list[].groupChat`)
- تخصيص بادئات الرسائل (`messages`)
- تعيين مساحة عمل الوكيل (`agents.defaults.workspace` أو `agents.list[].workspace`)
- ضبط الإعدادات الافتراضية للوكيل المُضمَّن (`agents.defaults`) وسلوك الجلسات (`session`)
- تعيين هوية لكل وكيل (`agents.list[].identity`)

> **جديد على التهيئة؟** اطّلع على دليل [أمثلة التهيئة](/gateway/configuration-examples) للحصول على أمثلة كاملة مع شروح مفصّلة!

## التحقق الصارم من التكوين

لا يقبل OpenClaw إلا التهيئات التي تطابق المخطط بالكامل.
المفاتيح غير المعروفة، أو الأنواع غير الصحيحة، أو القيم غير الصالحة تجعل Gateway **يرفض البدء** حفاظًا على الأمان.

عند فشل التحقّق:

- لا يبدأ Gateway.
- يُسمح فقط بأوامر التشخيص (على سبيل المثال: `openclaw doctor`، `openclaw logs`، `openclaw health`، `openclaw status`، `openclaw service`، `openclaw help`).
- شغّل `openclaw doctor` لعرض المشكلات بدقّة.
- شغّل `openclaw doctor --fix` (أو `--yes`) لتطبيق عمليات الترحيل/الإصلاح.

لا يكتب Doctor أي تغييرات إلا إذا اخترت صراحةً `--fix`/`--yes`.

## المخطط + تلميحات واجهة المستخدم

يعرض Gateway تمثيل JSON Schema للتهيئة عبر `config.schema` لمحرّرات واجهة المستخدم.
تعرض واجهة التحكم نموذجًا مبنيًا على هذا المخطط، مع محرّر **Raw JSON** كخيار طوارئ.

يمكن لإضافات القنوات والامتدادات تسجيل مخطط + تلميحات واجهة المستخدم لتهيئتها، بحيث تبقى إعدادات القنوات قائمة على المخطط عبر التطبيقات دون نماذج مُشفّرة يدويًا.

تُرفَق التلميحات (التسميات، التجميع، الحقول الحسّاسة) مع المخطط كي تتمكّن العملاء من عرض نماذج أفضل دون معرفة مُسبقة بالتهيئة.

## التطبيق + إعادة التشغيل (RPC)

استخدم `config.apply` للتحقّق وكتابة التهيئة كاملةً وإعادة تشغيل Gateway في خطوة واحدة.
يكتب إشارة إعادة تشغيل ويُجري تنبيهًا لآخر جلسة نشطة بعد عودة Gateway.

تحذير: يستبدل `config.apply` **التهيئة بالكامل**. إذا أردت تغيير مفاتيح قليلة فقط،
فاستخدم `config.patch` أو `openclaw config set`. احتفِظ بنسخة احتياطية من `~/.openclaw/openclaw.json`.

Params:

- `raw` (string) — حمولة JSON5 لكامل التهيئة
- `baseHash` (اختياري) — تجزئة التهيئة من `config.get` (مطلوبة عند وجود تهيئة مسبقًا)
- `sessionKey` (اختياري) — مفتاح آخر جلسة نشطة لتنبيه الاستيقاظ
- `note` (اختياري) — ملاحظة لإدراجها في إشارة إعادة التشغيل
- `restartDelayMs` (اختياري) — مهلة قبل إعادة التشغيل (الافتراضي 2000)

مثال (عبر `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## التحديثات الجزئية (RPC)

استخدم `config.patch` لدمج تحديث جزئي في التهيئة الحالية دون الكتابة فوق
المفاتيح غير ذات الصلة. يطبّق دلالات JSON merge patch:

- تندمج الكائنات تكراريًا
- `null` يحذف مفتاحًا
- تُستبدل المصفوفات
  وكما في `config.apply`، يتم التحقّق وكتابة التهيئة وتخزين إشارة إعادة التشغيل وجدولة
  إعادة تشغيل Gateway (مع تنبيه اختياري عند توفير `sessionKey`).

Params:

- `raw` (string) — حمولة JSON5 تحتوي فقط المفاتيح المراد تغييرها
- `baseHash` (مطلوب) — تجزئة التهيئة من `config.get`
- `sessionKey` (اختياري) — مفتاح آخر جلسة نشطة لتنبيه الاستيقاظ
- `note` (اختياري) — ملاحظة لإدراجها في إشارة إعادة التشغيل
- `restartDelayMs` (اختياري) — مهلة قبل إعادة التشغيل (الافتراضي 2000)

مثال:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## أقل تهيئة ممكنة (نقطة بدء مُوصى بها)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

ابنِ الصورة الافتراضية مرة واحدة باستخدام:

```bash
scripts/sandbox-setup.sh
```

## وضع الدردشة الذاتية (مُوصى به للتحكّم بالمجموعات)

لمنع البوت من الرد على إشارات @ في مجموعات WhatsApp (والرد فقط على محفّزات نصية محددة):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## تضمينات التهيئة (`$include`)

قسّم تهيئتك إلى ملفات متعددة باستخدام التوجيه `$include`. يفيد ذلك في:

- تنظيم التهيئات الكبيرة (مثل تعريفات وكلاء لكل عميل)
- مشاركة الإعدادات المشتركة عبر البيئات
- إبقاء التهيئات الحسّاسة منفصلة

### الاستخدام الأساسي

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### سلوك الدمج

- **ملف واحد**: يستبدل الكائن الذي يحتوي `$include`
- **مصفوفة ملفات**: دمج عميق للملفات بالترتيب (اللاحق يغلّب السابق)
- **مع مفاتيح شقيقة**: تُدمج المفاتيح الشقيقة بعد التضمينات (تغلّب القيم المُضمَّنة)
- **مفاتيح شقيقة + مصفوفات/بدائيات**: غير مدعوم (يجب أن يكون المحتوى المُضمَّن كائنًا)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### تضمينات متداخلة

يمكن للملفات المُضمَّنة أن تحتوي بدورها على توجيهات `$include` (حتى 10 مستويات عمق):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### حلّ المسارات

- **مسارات نسبية**: تُحلّ نسبةً إلى الملف المُضمِّن
- **مسارات مطلقة**: تُستخدم كما هي
- **دلائل عليا**: تعمل مراجع `../` كما هو متوقّع

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### معالجة الأخطاء

- **ملف مفقود**: خطأ واضح مع المسار المُحلّ
- **خطأ تحليل**: يبيّن أي ملف مُضمَّن فشل
- **تضمينات دائرية**: تُكتشف ويُبلّغ عنها مع سلسلة التضمين

### مثال: إعداد قانوني متعدد العملاء

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## الخيارات الشائعة

### Env vars + `.env`

يقرأ OpenClaw متغيرات البيئة من العملية الأب (الصدفة، launchd/systemd، CI، إلخ).

بالإضافة إلى ذلك، يحمّل:

- `.env` من دليل العمل الحالي (إن وُجد)
- ملف احتياطي عام `.env` من `~/.openclaw/.env` (المعروف أيضًا بـ `$OPENCLAW_STATE_DIR/.env`)

لا يتجاوز ملف `env` ملف env الموجود حالياً.

يمكنك أيضًا توفير متغيرات بيئة ضمنية داخل التهيئة. تُطبّق فقط إذا كان مفتاح العملية مفقودًا
(نفس قاعدة عدم التجاوز):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

انظر [/environment](/help/environment) للاطّلاع على الأسبقية الكاملة والمصادر.

### `env.shellEnv` (اختياري)

خيار ملائم بالاشتراك: إذا كان مفعّلًا ولم تُضبط أي من المفاتيح المتوقعة بعد، يشغّل OpenClaw صدفة تسجيل الدخول لديك ويستورد فقط المفاتيح المتوقعة المفقودة (ولا يتجاوز القيم).
هذا يعادل فعليًا استيراد ملف تعريف الصدفة لديك.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

مكافئ Env :

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### استبدال Env var في التهيئة

يمكنك الإشارة مباشرةً إلى متغيرات البيئة في أي قيمة نصية بالتهيئة باستخدام
صيغة `${VAR_NAME}`. يتم الاستبدال عند تحميل التهيئة وقبل التحقّق.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**القواعد:**

- لا تُطابِق إلا أسماء متغيرات البيئة بالحروف الكبيرة: `[A-Z_][A-Z0-9_]*`
- المتغيرات المفقودة أو الفارغة تُسبّب خطأ عند تحميل التهيئة
- الهروب باستخدام `$${VAR}` لإخراج `${VAR}` حرفيًا
- تعمل مع `$include` (تُطبّق الاستبدالات أيضًا على الملفات المُضمَّنة)

**الاستبدال الضمني:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### تخزين المصادقة (OAuth + مفاتيح API)

يخزّن OpenClaw ملفات المصادقة **لكل وكيل** (OAuth + مفاتيح API) في:

- `<agentDir>/auth-profiles.json` (الافتراضي: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

انظر أيضًا: [/concepts/oauth](/concepts/oauth)

استيرادات OAuth القديمة:

- `~/.openclaw/credentials/oauth.json` (أو `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

يحافظ وكيل Pi المُضمَّن على ذاكرة تخزين مؤقتة وقت التشغيل في:

- `<agentDir>/auth.json` (تُدار تلقائيًا؛ لا تُعدّل يدويًا)

دليل الوكيل القديم (قبل تعدد الوكلاء):

- `~/.openclaw/agent/*` (يُرحّل بواسطة `openclaw doctor` إلى `~/.openclaw/agents/<defaultAgentId>/agent/*`)

التجاوزات:

- دليل OAuth (استيراد قديم فقط): `OPENCLAW_OAUTH_DIR`
- دليل الوكيل (تجاوز جذر الوكيل الافتراضي): `OPENCLAW_AGENT_DIR` (مُفضّل)، `PI_CODING_AGENT_DIR` (قديم)

عند أول استخدام، يستورد OpenClaw إدخالات `oauth.json` إلى `auth-profiles.json`.

### `auth`

بيانات وصفية اختيارية لملفات المصادقة. **لا** تخزّن أسرارًا؛ بل تربط
معرّفات الملفات بمزوّد + وضع (وبريد إلكتروني اختياري) وتحدّد ترتيب
تدوير المزوّد المستخدم للتجاوز عند الفشل.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

هوية اختيارية لكل وكيل تُستخدم للإعدادات الافتراضية وتجربة الاستخدام. يكتبها معالج الإعداد الأولي على macOS.

إذا كانت مُعيّنة، يستنتج OpenClaw القيم الافتراضية (فقط عندما لا تكون قد عيّنتها صراحةً):

- `messages.ackReaction` من `identity.emoji` **للوكيل النشط** (الافتراضي 👀)
- `agents.list[].groupChat.mentionPatterns` من `identity.name`/`identity.emoji` للوكيل (ليعمل “@Samantha” عبر مجموعات Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
- `identity.avatar` يقبل مسار صورة نسبي لمساحة العمل أو URL/‏data URL بعيد. يجب أن تكون الملفات المحلية داخل مساحة عمل الوكيل.

يقبل `identity.avatar`:

- مسارًا نسبيًا لمساحة العمل (يجب أن يبقى داخل مساحة عمل الوكيل)
- عنوان URL من `http(s)`
- URI من `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

بيانات وصفية يكتبها معالجات CLI (`onboard`، `configure`، `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- ملف السجل الافتراضي: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- إذا أردت مسارًا ثابتًا، عيّن `logging.file` إلى `/tmp/openclaw/openclaw.log`.
- يمكن ضبط إخراج وحدة التحكم بشكل منفصل عبر:
  - `logging.consoleLevel` (الافتراضي `info`، ويرتفع إلى `debug` عند `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- يمكن تنقيح ملخّصات الأدوات لتجنّب تسرّب الأسرار:
  - `logging.redactSensitive` (`off` | `tools`، الافتراضي: `tools`)
  - `logging.redactPatterns` (مصفوفة تعابير نمطية؛ تتجاوز الافتراضيات)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

يتحكّم في كيفية التعامل مع الدردشات المباشرة في WhatsApp (DMs):

- `"pairing"` (الافتراضي): يحصل المرسلون المجهولون على رمز إقران؛ يجب على المالك الموافقة
- `"allowlist"`: السماح فقط للمرسلين في `channels.whatsapp.allowFrom` (أو مخزن السماح المقترن)
- `"open"`: السماح بجميع DMs الواردة (**يتطلّب** تضمين `channels.whatsapp.allowFrom` لـ `"*"`)
- `"disabled"`: تجاهل جميع DMs الواردة

تنتهي صلاحية رموز الإقران بعد ساعة؛ ولا يرسل البوت رمز إقران إلا عند إنشاء طلب جديد. تُحدَّد طلبات إقران DMs المعلّقة بحدّ أقصى **3 لكل قناة** افتراضيًا.

موافقات الإقران:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

قائمة سماح بأرقام هواتف E.164 التي يمكنها تشغيل ردود WhatsApp التلقائية (**DMs فقط**).
إذا كانت فارغة و `channels.whatsapp.dmPolicy="pairing"`، سيتلقى المرسلون المجهولون رمز إقران.
للمجموعات، استخدم `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

يتحكّم فيما إذا كانت رسائل WhatsApp الواردة تُعلَّم كمقروءة (علامتا زرقاء). الافتراضي: `true`.

يتجاوز وضع الدردشة الذاتية إيصالات القراءة دائمًا، حتى عند التمكين.

تجاوز لكل حساب: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (تعدّد الحسابات)

تشغيل عدة حسابات WhatsApp في Gateway واحد:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

ملاحظات:

- تُوجَّه الأوامر الصادرة افتراضيًا إلى الحساب `default` إن وُجد؛ وإلا فأول معرّف حساب مُهيّأ (مرتب).
- يُرحَّل دليل مصادقة Baileys أحادي الحساب القديم بواسطة `openclaw doctor` إلى `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

تشغيل عدة حسابات لكل قناة (لكل حساب `accountId` خاص به و `name` اختياري):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

ملاحظات:

- يُستخدم `default` عندما يُهمل `accountId` (CLI + التوجيه).
- تنطبق رموز البيئة فقط على الحساب **الافتراضي**.
- تُطبّق إعدادات القناة الأساسية (سياسة المجموعات، بوابة الإشارات، إلخ) على جميع الحسابات ما لم تُتجاوز لكل حساب. تطبيق على جميع الحسابات ما لم يتم تجاوزها لكل حساب.
- استخدم `bindings[].match.accountId` لتوجيه كل حساب إلى agents.defaults مختلف.

### بوابة الإشارات في الدردشة الجماعية (`agents.list[].groupChat` + `messages.groupChat`)

تفترض رسائل المجموعات افتراضيًا **اشتراط الإشارة** (إشارة وصفية أو أنماط regex). ينطبق ذلك على مجموعات WhatsApp وTelegram وDiscord وGoogle Chat وiMessage.

**أنواع الإشارات:**

- **الإشارات الوصفية**: إشارات @ الأصلية للمنصة (مثل النقر للإشارة في WhatsApp). تُتجاهل في وضع الدردشة الذاتية لـ WhatsApp (انظر `channels.whatsapp.allowFrom`).
- **أنماط النص**: أنماط Regex مُعرَّفة في `agents.list[].groupChat.mentionPatterns`. تُفحَص دائمًا بغضّ النظر عن وضع الدردشة الذاتية.
- لا تُفرَض بوابة الإشارات إلا عندما يكون اكتشاف الإشارة ممكنًا (إشارات أصلية أو على الأقل `mentionPattern` واحد).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

يُعيّن `messages.groupChat.historyLimit` الافتراضي العام لسياق سجلّ المجموعات. يمكن للقنوات التجاوز عبر `channels.<channel>.historyLimit` (أو `channels.<channel>.accounts.*.historyLimit` لتعدّد الحسابات). عيّن `0` لتعطيل لفّ السجل.

#### حدود تاريخ DM

تستخدم محادثات DMs سجلًا قائمًا على الجلسات يديره الوكيل. يمكنك تقييد عدد أدوار المستخدم المحتفَظ بها لكل جلسة DM:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

ترتيب الحسم:

1. تجاوز لكل DM: `channels.<provider>.dms[userId].historyLimit`
2. افتراضي المزوّد: `channels.<provider>.dmHistoryLimit`
3. بلا حد (الاحتفاظ بكل السجل)

المزوّدون المدعومون: `telegram`، `whatsapp`، `discord`، `slack`، `signal`، `imessage`، `msteams`.

تجاوز لكل وكيل (له الأسبقية عند الضبط، حتى `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

ذكر الافتراضي عند البوابة لكل قناة (`channels.whatsapp.groups`، `channels.telegram.groups`، `channels.imessage.groups`، `channels.discord.guilds`). عندما يتم تعيين `*.groups'، فإنها تعمل أيضا كقائمة مسموح بها للمجموعة؛ تشمل `\*' للسماح لجميع المجموعات.

للرد **فقط** على مشغلات نص محددة (تجاهل الأصل @-mentions):

```json5
1. {
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### سياسة المجموعة (لكل قناة)

استخدم 'channels.\*.groupPolicy' للتحكم فيما إذا كانت رسائل المجموعة/الغرفة مقبولة على الإطلاق:

```json5
2. {
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

ملاحظات:

- `فتح"`: المجموعات تتخطى قوائم السماح بذلك؛ لا تزال البوابة المذكورة منطبقة.
- `معطل`: حظر جميع رسائل المجموعة/الغرفة.
- `"allowlist"`: فقط السماح للمجموعات/الغرف التي تتطابق مع قائمة المسموح بتكوينها.
- 'channels.defaults.groupPolicy' يعين الافتراضي عندما يتم إلغاء تعيين 'groupPolicy' لمقدم الخدمة.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams يستخدم \`groupAllowFrom' (الخلفية: 'allowFrom' صراحة).
- Discord/Slack استخدام قائمة السماح للقناة (`channels.discord.guilds.*.channels`، `channels.slack.channels`).
- لا تزال مجموعة DMs (Discord/Slack) خاضعة لـ `dm.groupEnabled` + `dm.groupChannels`.
- الافتراضي هو `سياسة المجموعات: "allowlist"` (ما لم يتم تجاوزها بواسطة `channels.defaults.groupPolicy`)؛ إذا لم يتم تكوين قائمة السماح ، يتم حظر الرسائل الجماعية.

### توجيه متعدد العوامل (`الوكلاءs.list` + `الارتباطات`)

تشغيل عدة عناصر معزولة (مساحة عمل منفصلة، 'وكيل دير\`، دورات) داخل بوابة واحدة.
يتم توجيه الرسائل الواردة إلى وكيل عن طريق الربط.

- المعاملات:
  - `id`: معرف العامل المستقر (مطلوب).
  - 'الافتراضي\`: اختياري؛ عندما يتم تعيين المتعددين، يفوز الأول ويتم تسجيل التحذير.
    إذا لم يتم تعيين أي منها، فإن **أول إدخال** في القائمة هو الوكيل الافتراضي.
  - `name`: اسم العرض للوكيل.
  - `مساحة العمل`: الافتراضي `~/.openclaw/workspace-<agentId>(لـ `main`، العودة إلى `agents.defaults.workspace\`).
  - `agentDir`: الافتراضي `~/.openclaw/agents/<agentId>/agent`.
  - 'model`: النموذج الافتراضي لكل وكيل ، يلغي `agents.defaults.model' لذلك الوكيل.
    - شكل السلسلة: `مزود/model"`، يلغي فقط `agents.defaults.model.primary`
    - شكل الكائن: `{ primary, fallbacks }` (تجاوز الاحتياطات `agents.defaults.model.fallbacks`; `[]` يعطل الارتداد العالمي لذلك الوكيل)
  - `هوية`: اسم كل عامل/موضوع/emoji (يستخدم لذكر الأنماط + ردود الفعل).
  - `محادثة المجموعة`: لكل عامل ذكر البوابة (`ذكر:Patterns`).
  - `sandbox`: لكل عامل sandbox config (تجاوز `agents.defaults.sandbox`).
    - `mode`: `"off"` <unk> `"non-main"` <unk> `"all"`
    - `مساحة العمل`: `"لا شيء"` <unk> `ro"` <unk> `"rw"`
    - `النطاق`: `الدورة` <unk> `الوكيل"` <unk> `"المشتركة"`
    - `workspaceRoot`: جذر ساتل العمل المخصص
    - `docker`: تجاوزات مرفأ لكل عامل (مثل `image` و`network` و`env` و`setupcommand` والحدود؛ وتجاهلت عندما `النطاق: "المشترك"`)
    - `المتصفح`: تجاوز المتصفح الرملي لكل عامل (تم تجاهله عندما `النطاق: "مشاركة"`)
    - 'prune': تجاوز مربع رمل لكل عامل (تم تجاهله عندما 'النطاق: 'المشتركة\`)
  - 'العوامل الفرعية\`: افتراضات العوامل الفرعية لكل عميل.
    - `الوكلاء`: السماح بقائمة معارف الوكيل لـ `sessions_spawn' من هذا الوكيل (`["\*"]\` = السماح بأي وسيل؛ الافتراضي: فقط نفس الوكيل)
  - `أدوات`: قيود على أداة لكل عامل (تطبق قبل سياسة أداة صندوق الرمل).
    - 'الملف الشخصي\`: ملف تعريف الأداة الأساسية (يطبق قبل السماح/الرفض)
    - `حائز`: مجموعة من أسماء الأدوات المسموح بها
    - `نكر`: مجموعة من أسماء الأدوات المحرومة (فوز)
- 'agents.defaults': افتراضي الوكيل المشترك (النموذج، مساحة العمل، صندوق الرملة، إلخ.).
- `ملزمات]`: توجّه الرسائل الواردة إلى `وكيل هوية`.
  - \`match.channel' (مطلوب)
  - `match.accountId` (اختياري؛ `*` = أي حساب؛ حذف = الحساب الافتراضي)
  - `match.peer` (اختياري؛ `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (اختياري؛ قناة محددة)

ترتيب المباراة الوزيرية:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. \`match.accountId' (بالضبط، لا أقران/guild/team)
5. \`match.accountId: "\*" (channel-wide, no peer/guild/team)
6. الوكيل الافتراضي (`agents.list[].default`، وإلا أدخل أول قائمة أخرى، `main"`)

داخل كل مستوى من مستويات المطابقة، يفوز أول إدخال مطابق في "الارتباطات".

#### ملفات وصول لكل وكيل (متعدد الوكلاء)

يمكن لكل وكيل أن يحمل سياسة العلبة الرملية + الخاصة به. استخدم هذا لمزج الوصول إلى المستويات
في بوابة واحدة:

- **الوصول الكامل** (الوكيل الشخصي)
- أدوات **للقراءة فقط** + مساحة العمل
- **لا يوجد وصول إلى نظام الملفات** (الرسائل/أدوات الجلسة فقط)

راجع [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) للحصول على أمثلة إضافية عن الأسبقية و
.

الوصول الكامل (بدون علبة رملية):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

أدوات القراءة فقط + مساحة العمل للقراءة فقط:

```json5
3. {
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

لا يوجد وصول إلى نظام الملفات (تمكين الرسائل/أدوات الجلسة):

```json5
4. {
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

على سبيل المثال: حسابان في WhatsApp → وكيلان:

```json5
5. {
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (اختياري)

رسالة الوكيل إلى الوكيل تختار:

```json5
{
  أدوات: {
    وكيل الوكيل: {
      enableled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `messages.liste`

يتحكم في كيفية تصرف الرسائل الواردة عندما يكون وكيل يعمل بالفعل نشطا.

```json5
6. {
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

7. قم بعمل إزالة اهتزاز (Debounce) للرسائل الواردة السريعة من **نفس المرسل** بحيث تتحول عدة رسائل متتالية إلى دور واحد للوكيل. يُنطاق الإخماد لكل قناة + محادثة، ويستخدم أحدث رسالة لربط الردود/المعرّفات.

```json5
{
  رسائل: {
    غير محدد: {
      debounceMs: 2000, // 0 يعطل
      بأي قناة: {
        أي تطبيق: 5000،
        سلسلة: 1500,
        Discord: 1500,
      },
    },
  },
}
```

ملاحظات:

- حذف دفعات **رسائل نصية فقط**؛ وسائط الإعلام/المرفقات تمضي على الفور.
- أوامر التحكم (مثلاً: `/liste`, `/new`) تجاوز التنصيص حتى تظل ثابتة.

### `الأوامر` (مناولة أمر الدردشة)

التحكم في كيفية تمكين أوامر الدردشة عبر الموصلات.

```json5
{
  الأوامر: {
    الأصلي: "تلقائي"، // تسجيل الأوامر الأصلية عند دعم النص (تلقائي)
    : صحيح, // تحليل الأوامر اللاصقة في رسائل الدردشة
    باش: خطأ، // اسمح ب! (الاسم المستعار /bash) (المضيف فقط؛ يتطلب أدوات. القوائم المسموح بها)
    bashForegroundMs: 2000, // bash في مقدمة النافذة (0 خلفيات فورا)
    config: false, // السماح /config (يكتب إلى قرص)
    التصحيح: خطأ، // السماح /debug (تشغيل/تجاوز الوقت فقط)
    إعادة التشغيل: خطأ، // السماح / إعادة التشغيل + إعادة تشغيل أداة إعادة تشغيل البوابة
    useAccessGroups: true, // إنفاذ السماح لمجموعة الوصول/السياسات للأوامر
  },
}
```

ملاحظات:

- يجب إرسال أوامر النص كرسالة **وقائمة بذاتها** واستخدام القيادات `/` (لا توجد أسماء مستعارة للنص السريع).
- \`commands.text: false' يعطل تحليل رسائل الدردشة للأوامر.
- `commands.native: "auto"` (default) يعمل على الأوامر الأصلية للديسكورد/تيليجرام ويترك Slack متوقفة ؛ القنوات غير المدعومة تبقى النص فقط.
- تعيين `commands.native: true<unk> false` لإجبار الجميع، أو تجاوز كل قناة مع `channels.discord.commands.native`، `channels.telegram.commands.native`، `channels.slack.commands.native` (bool أو `auto"`). "false" مسح الأوامر المسجلة سابقاً على ديسكورد/تيليجرام عند بدء التشغيل؛ تتم إدارة الأوامر Slack في تطبيق Slack .
- يضيف `channels.telegram.customcommands` إدخالات إضافية في قائمة البوت تيليجرام. يتم تطبيع الأسماء؛ يتم تجاهل التعارض مع الأوامر الأصلية.
- `commands.bash: true' تمكين `! <cmd>` لتشغيل أوامر قذيفة المضيف (`/bash <cmd>'يعمل أيضًا كاسم مستعار). 8. يتطلب `tools.elevated.enabled` وإدراج المرسل في قائمة السماح ضمن `tools.elevated.allowFrom.<channel>`.\`.
- `commands.bashForegroundMs` يتحكم في طول فترة الانتظار قبل الخلفية. بينما يتم تشغيل وظيفة باش ، \`! الطلبات <cmd>تم رفضها (واحدة في كل مرة).
- `commands.config: true' تمكين `/config' (reads/writes `openclaw.json`).
- `القنوات.<provider>تهيئة بوابات .configWrites` التي بدأتها تلك القناة (الافتراضي: صحيح). ينطبق هذا على `/config set<unk> unset` زائداً الترحيل التلقائي الخاص بالمزود (تغييرات معرف المجموعة الخارقة تيليجرام، تغييرات معرف القناة Slack Tack).
- 'commands.debug: true' قم بتمكين '/debug' (runtime فقط overrides).
- 'commands.restart: true' تمكين '/restart' والبوابة أداة إعادة تشغيل الإجراء.
- 'commands.useAccessGroups: false' يسمح للأوامر بتجاوز السماح للمجموعة الوصول إلى المجموعة/السياسات.
- أوامر الشرطة المائلة والتوجيهات تُحترم فقط للمرسلين **المخوّلين**. الاعتماد مستمد من
  السماح للقناة / الاقتران بالإضافة إلى `commands.useAccessGroups`.

### `web` (تشغيل قناة ويب)

WhatsApp يعمل من خلال قناة ويب البوابة (Baileys Web). يبدأ تلقائياً عند وجود جلسة مترابطة.
تعيين `web.enabled: خطأ` لإبقائها مغلقة بشكل افتراضي.

```json5
9. {
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram` (نقل البوت)

OpenClaw يبدأ Telegram فقط عندما يكون قسم التكوين `channels.telegram` موجوداً. تم حل الرمز المميز للبوت من `channels.telegram.botToken` (أو `channels.telegram.tokenFile`)، مع `TELEGRAM_BOT_TOKEN` كرد احتياطي للحساب الافتراضي.
تعيين `channels.telegram.enabled: false` لتعطيل بدء التشغيل التلقائي.
حياة الدعم المتعدد الحسابات تحت 'channels.telegram.accounts' (انظر القسم متعدد الحسابات أعلاه). تطبيق رمز Env فقط على الحساب الافتراضي.
تعيين `channels.telegram.configWrites: false` لمنع كتابات التكوين التي بدأتها تيليجرام (بما في ذلك هجرات معرف المجموعة الخارقة و `/config set<unk> unset`).

```json5
10. {
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

مشروع ملاحظات التدفق:

- يستخدم Telegram `sendMessageDraft` (مسودة الفقاعة، وليس رسالة حقيقية).
- يتطلب **مواضيع الدردشة الخاصة** (message_thread_id في DM; البوت لديه موضوعات مفعلة).
- '/reasoning stream\` هي مسارات منطقية في المشروع، ثم ترسل الإجابة النهائية.
  إعادة محاولة الافتراضات والسلوك موثقة في [إعادة المحاولة](/concepts/retry).

### `channels.discord` (نقل البوت)

11. قم بتهيئة بوت Discord عن طريق تعيين رمز البوت (bot token) والقيود الاختيارية:
    دعم تعدد الحسابات موجود ضمن `channels.discord.accounts` (انظر قسم تعدد الحسابات أعلاه). تطبيق رمز Env فقط على الحساب الافتراضي.

```json5
12. {
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw يبدأ ديسكورد فقط عند وجود قسم التكوين `channels.discord'. تم حل الرمز المميز من `channels.discord.token'، مع `DISCORD_BOT_TOKEN' كرد احتياطي للحساب الافتراضي (ما لم يكن `channels.discord.enabled' 'false'). استخدم `user:<id>` (DM) أو `channel:<id>` (Guild channel) عند تحديد أهداف التسليم لأوامر الأقران/LI؛ المعرفات الرقمية غير المباشرة غامضة ومرفوضة.
منحدرات الرابطة صغيرة مع استبدال المسافات بـ `-`؛ مفاتيح القناة تستخدم اسم القناة الموهجة (لا يوجد رائد `#`). تفضيل معارف الرابطة كمفاتيح لتجنب إعادة تسمية الغموض.
يتم تجاهل الرسائل المؤلفة من بوت بشكل افتراضي. تمكين مع \`channels.discord.allow', (لا تزال الرسائل الخاصة يتم تصفيتها لمنع حلقات الرد الذاتي).
وضع اشعار رد الفعل:

- `off`: بلا أحداث تفاعل.
- `own`: التفاعلات على رسائل البوت نفسه (الافتراضي).
- `all`: كل التفاعلات على جميع الرسائل.
- `allowlist`: التفاعلات من `guilds.<id>.users` على جميع الرسائل (قائمة فارغة تعطل).
  النص الصادر مقسم إلى `channels.discord.textChunkLimit` (الافتراضي 2000). تعيين `channels.discord.chunkMode="newline"` لتقسيم على الخطوط الفارغة (حدود الفقرة) قبل طول القطع. يمكن لعملاء ديسكورد أن يقطعوا رسائل طويلة جداً، لذلك يقوم `channels.discord.maxLinesPerMessage` (الافتراضي 17) بتقسيم الردود الطويلة متعددة الأسطر حتى عندما تقل عن 2000 حرف.
  إعادة محاولة الافتراضات والسلوك موثقة في [إعادة المحاولة](/concepts/retry).

### `channels.googlechat` (Chat API webhook)

محادثة جوجل تعمل عبر HTTP webhooks مع مصادقة على مستوى التطبيق (حساب الخدمة).
حياة الدعم المتعدد الحسابات تحت \`channels.googlechat.accounts' (انظر القسم متعدد الحسابات أعلاه). لا ينطبق Env إلا على الحساب الافتراضي.

```json5
13. {
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; improves mention detection
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // optional; "open" requires ["*"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

ملاحظات:

- حساب الخدمة JSON يمكن أن يكون مضمن ('serviceAccount`) أو مستندا إلى ملف ('serviceAccountFile`).
- انس الارتداد للحساب الافتراضي: `GOOGLE_CHAT_SERVICE_ACCOUNT` أو `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- يجب أن يطابق 'الجمهور' + 'الجمهور' إعدادات مصادقة الويب هوك لتطبيق الدردشة.
- استخدم `المسافات/<spaceId>` أو \`المستخدمين/<userId|email>عند تحديد أهداف التسليم.

### \`channels.slack' (وضع المقطع)

Slack يعمل في وضع المقبس ويتطلب كل من رمز بوت و رمز التطبيق:

```json5
14. {
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50, // include last N channel/group messages as context (0 disables)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

حساب دعم متعدد تحت 'channels.slack.accounts' (انظر القسم متعدد الحسابات أعلاه). تطبيق رمز Env فقط على الحساب الافتراضي.

يبدأ OpenClaw Slack عندما يتم تمكين المزود ويتم تعيين كلا الرموز (عن طريق التكوين أو `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). استخدم `user:<id>` (DM) أو `channel:<id>عند تحديد أهداف التسليم لأوامر الأقران/LI.
تعيين `channels.slack.configWrites: false`لمنع كتابة الإعدادات التي بدأها Slack-بوتقة (بما في ذلك هجرات معرف القناة و`/config set<unk> unset\`).

يتم تجاهل الرسائل المؤلفة من بوت بشكل افتراضي. تمكين باستخدام `channels.slack.allowالبلاغ` أو `channels.slack.channels.<id>.allowBots`.

وضع اشعار رد الفعل:

- `off`: بلا أحداث تفاعل.
- `own`: التفاعلات على رسائل البوت نفسه (الافتراضي).
- `all`: كل التفاعلات على جميع الرسائل.
- `allowlist`: ردود الفعل من \`channels.slack.reactionAllowlist' على جميع الرسائل (تعطيل القائمة الفارغة).

جلسة مناقشة منعزلة:

- 'channels.slack.thread.historyScope' يتحكم فيما إذا كان تاريخ الموضوع لكل موضوع ('thread', default) أو مشاركته عبر القناة ('channel\`).
- 'channels.slack.thread.inheritParent' يتحكم فيما إذا كانت جلسات المواضيع الجديدة ترث نص القناة الأصلية (الافتراضي: false).

مجموعات عمل "بلاك" (إجراءات أداة "التلاعب"):

| مجموعة الإجراءات | الافتراضي | الملاحظات                   |
| ---------------- | --------- | --------------------------- |
| reactions        | مفعّل     | تفاعل + سرد التفاعلات       |
| messages         | مفعّل     | قراءة/إرسال/تعديل/حذف       |
| pins             | مفعّل     | تثبيت/إلغاء التثبيت/القائمة |
| memberInfo       | مفعّل     | معلومات الأعضاء             |
| emojiList        | مفعّل     | قائمة الرموز المخصصة        |

### \`channels.mattermost' (bot token)

يُقدَّم Mattermost كمكوّن إضافي ولا يكون مضمّنًا مع التثبيت الأساسي.
ثبّته أولاً: `openclaw plugins install @openclaw/mattermost` (أو `./extensions/mattermost` من git Checout).

Mattermost يتطلب رمز بوت بالإضافة إلى عنوان URL الأساسي للخادم الخاص بك:

```json5
{
  القناة: {
    مهمة: {
      مفعل: صحيح,
      botToken: "mm-token",
      baseUrl: "https://chat. إكساميل. om",
      dmPolicy: "الاقتران"،
      المحادثة: "oncall", // على المكالمة <unk> onmessage <unk> onchar
      oncharfixes: [">", "! ]،
      قطعة نص: 4000،
      جزء موعد: "طول"،
    },
  },
}
```

يبدأ OpenClaw Mattermost عند تكوين الحساب (بوت توكين + قاعدة URL) ومفعل. يتم حل عنوان URL المميز + قاعدة من 'channels.mattermost.botToken' + 'channels.mattermost.baseUrl' أو 'MATTERMOST_BOT_TOKEN' + 'MATTERMOST_URL\` للحساب الافتراضي (ما لم يكن 'channels.mattermost.enabled' 'false').

وضع الدردشة:

- 'oncall' (الافتراضي): الرد على رسائل القناة فقط عندما @mentioned.
- `onmessage`: الرد على كل رسالة في القناة.
- `onchar`: الرد عندما تبدأ الرسالة ببادئة مشغلة (`channels.mattermost.oncharPrefixes`، الافتراضي `[">"، "!"]`).

التحكم بالدخول:

- DM: `channels.mattermost.dmPolicy="الاقتران"` (المرسلون غير المعروفين يحصلون على رمز اقتران).
- الرسائل الخاصة العامة: `channels.mattermost.dmPolicy="open"` بالإضافة إلى `channels.mattermost.allowFrom=["*"]`.
- المجموعات: `channels.mattermost.groupPolicy="allowlist"" افتراضياً (ذكر). استخدم `channels.mattermost.groupAllowFrom\` لتقييد المرسلين.

حياة الدعم المتعدد الحسابات تحت `channels.mattermost.accounts' (انظر القسم متعدد الحسابات أعلاه). لا ينطبق Env إلا على الحساب الافتراضي.
استخدم `channel:<id>`أو`user:<id>(أو `@username`) عند تحديد أهداف التسليم؛ والمعلمات bis يتم التعامل معها كمعرف للقناة.

### `channels.signal` (signal-cli)

ردود فعل الإشارات يمكن أن تنبعث من أحداث النظام (أداة التفاعل المشتركة):

```json5
{
  القنوات : {
    إشارة: {
      ReactionNotifices: "own", /off <unk> own <unk> all <unk> allowlist
      ReactionAlallowlist ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"]،
      HistyLimit: 50، // تشمل آخر رسائل المجموعة N كسياق (0 تعطل)
    },
  },
}
```

وضع اشعار رد الفعل:

- `off`: بلا أحداث تفاعل.
- `own`: التفاعلات على رسائل البوت نفسه (الافتراضي).
- `all`: كل التفاعلات على جميع الرسائل.
- `allowlist`: ردود الفعل من \`channels.signal.reactionAllowlist' على جميع الرسائل (تعطيل قائمة فارغة).

### \`channels.imessage' (imsg CLI)

OpenClaw spans \`imsg rpc' (JSON-RPC فوق stdio). لا توجد حاجة إلى رديمون أو ميناء.

```json5
15. {
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SCP for remote attachments when using SSH wrapper
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // include last N group messages as context (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

حياة دعم الحسابات المتعددة تحت \`channels.imessage.accounts' (انظر القسم متعدد الحسابات أعلاه).

ملاحظات:

- يتطلب الوصول الكامل للقرص إلى الرسائل DB.
- أول إرسال سيطلب إذن تلقائي للرسائل.
- تفضيل الأهداف `chat_id:<id>`. استخدم محادثة `imsg --الحد 20` لقائمة المحادثات.
- `channels.imessage.cliPath' يمكن أن يشير إلى نص مغلف (على سبيل المثال 'ssh' لماك آخر يعمل بـ `imsg rpc')؛ استخدم مفاتيح SSH لتجنب طلبات كلمة المرور.
- بالنسبة لأغلفة SSH البعيدة، قم بتعيين `channels.imessage.remoteHost` لجلب المرفقات عن طريق SCP عندما يتم تمكين \`includeFacilments'.

غلاف مثال:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

يعين **دليل فضاء العمل العالمي الوحيد** الذي يستخدمه الوكيل لعمليات الملفات.

الافتراضي: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

إذا تم تمكين `agents.defaults.sandbox'، يمكن للجلسات غير الرئيسية تجاوز هذا مع مساحة العمل الخاصة بها
لكل نطاق تحت `agents.defaults.sandbox.workspaceRoot\`.

### `agents.defaults.repoRoot`

جذر مستودع اختياري لإظهاره في خط وقت التشغيل الخاص بموجه النظام. إذا لم يتم تعيينه، يحاول OpenClaw
الكشف عن دليل `.git` عن طريق المشي صعودا من مساحة العمل (ودليل العمل الحالي
. ويجب أن يكون المسار موجودا لاستخدامه.

```json5
{
  الوكلاء: { الافتراضي: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

تعطيل الإنشاء التلقائي لملفات bootstrap لفضاء العمل (`AGENTS.md`، `SOUL.md`، `TOOLS.md`، `IDENTITY.md`، `USER.md`، `HEARTBEAT.md`، و`BOTSTRAP.md`).

استخدم هذا للنشر قبل البذور حيث تأتي ملفات مساحة العمل الخاصة بك من المستودع.

```json5
{
  الوكلاء: { الإفتراضي: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

الحد الأقصى لحروف كل ملف التمهيد في فضاء العمل محققة في موجه النظام
قبل التخزين. الافتراضي: `20000`.

عندما يتجاوز ملف هذا الحد، يسجل OpenClaw تحذيرا ويحقن
head/tail مقتطفة مع علامة.

```json5
{
  الوكلاء: { الإفتراضي: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

تعيين المنطقة الزمنية للمستخدم لـ **سياق طلب النظام** (ليس للطوابع الزمنية في
مظاريف الرسالة). إذا لم يتم تعيينه، يستخدم OpenClaw المنطقة الزمنية المضيفة عند التشغيل.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

يتحكم في **تنسيق الوقت** المبين في قسم التاريخ والوقت الحالي لمبادرة النظام.
الافتراضي: 'تلقائي\` تفضيلات (OS).

```json5
{
  الوكلاء: { الإفتراضي: { timeFormat: "تلقائي" } }, // auto <unk> 12 <unk> 24
}
```

### `الرسائل`

يتحكم في البادئات غير المحددة/الصادرة وردود الفعل العلوية الاختيارية.
انظر [Messages](/concepts/messages على الانتظار والجلسات وسياق البث.

```json5
{
  رسالة: {
    responsePrefix: "🦞", // أو "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    remove AckAfterReply: false,
  },
}
```

يتم تطبيق 'responsePrefix' على **جميع الردود الصادرة** (ملخصات الأدوات، كتلة
البث والردود النهائية) عبر القنوات ما لم تكن موجودة بالفعل.

التجاوزات يمكن تكوينها لكل قناة وكل حساب:

- يتجاوز `channels.<channel>.responsePrefix`
- يتجاوز `channels.<channel>.accounts.<id>.responsePrefix`

ترتيب الحسم (الأكثر تحديدًا يفوز):

1. يتجاوز `channels.<channel>.accounts.<id>.responsePrefix`
2. يتجاوز `channels.<channel>.responsePrefix`
3. `messages.responsePrefix`

الألفاظ

- 'غير محدد\` يصل إلى المستوى التالي.
- `""` يعطل البادئة صراحة ويوقف التعاقب.
- `"auto"` يستنبط `[{identity.name}]` بالنسبة للوكيل الموجه.

وتنطبق التجاوزات على جميع القنوات، بما في ذلك الإضافات، وعلى كل نوع من الردود الصادرة.

إذا لم يتم تعيين `messages.responsePrefix'، لا يتم تطبيق البادئة بشكل افتراضي. ردود WhatsApp الذاتية
هي الاستثناء: فهي افتراضية إلى '[{identity.name}]` عند الضبط، وإلا
`[openclaw]`، لذلك تظل المحادثات نفسها مقروءة.
عيّنه إلى `"تلقائي"` لاشتقاق `[{identity.name}]` للوكيل الموجه (عند تعيينه).

#### متغيرات القالب

يمكن أن يتضمن سلسلة "responsePrefix" متغيرات قالب تحل بشكل ديناميكي:

| المتغير           | الوصف                | مثال                                       |
| ----------------- | -------------------- | ------------------------------------------ |
| `{model}`         | اسم النموذج القصير   | `claude-opus-4-6`, `gpt-4o`                |
| `{modelFull}`     | معرف النموذج الكامل  | `البشري/claude-opus-4-6`                   |
| `{provider}`      | اسم المزود           | `بشرية`, `openai`                          |
| `{thinkingLevel}` | مستوى التفكير الحالي | `عالي`، `منخفض`، `إيقاف`                   |
| `{identity.name}` | اسم هوية الوكيل      | (نفس الوضع `التلقائية`) |

المتغيرات حساسة لحالة الأحرف (`{MODEL}` = `{model}`). `{think}` هو اسم مستعار لـ `{thinkingLevel}`.
تبقى المتغيرات التي لم يتم حلها كنص حرفي.

```json5
{
  رسائل: {
    الرد السابق: "[{model} <unk> think:{thinkingLevel}]",
  },
}
```

مثال على المخرج: `[claude-opus-4-6 <unk> think:high] إليك استجابتي...`

يتم تكوين بادئة WhatsApp الواردة عبر `channels.whatsapp.messagePrefix` (مهملة:
`messages.messagePrefix`). يبقى الافتراضي **بدون تغيير**: `"[openclaw]"` عندما يكون
`channels.whatsapp.allowFrom` فارغاً، وإلا `""` (بدون بادئة). عند استخدام
`"[openclaw]"`، سيستعمل OpenClaw بدلاً من ذلك `[{identity.name}]` عندما يكون الوكيل
لديه مجموعة \`identity.name'.

يرسل "ackReaction" رد فعل الرموز التعبيرية لأفضل جهد للإقرار بالرسائل الواردة
على القنوات التي تدعم ردود الفعل (Slack/Discord/Telegram/Google Chat). الافتراضي لـ
الوكيل النشط `identity.emoji' عند تعيينه، وإلا `👀"`. عيّنه إلى `""\` لتعطيل. (Automatic Translation)

التحكم "ackReactionSscope" عند إطلاق التفاعل:

- 'ذكر المجموعة' (الافتراضي): فقط عندما تتطلب المجموعة / الغرفة ذكر **و** البوت ذكر
- 'المجموعة\`: جميع رسائل المجموعة/الغرفة
- 'مباشرة\`: الرسائل المباشرة فقط
- `كل`: جميع الرسائل

'removeAckAfterReply' يزيل رد البوت الزائد بعد إرسال رد
(Slack/Discord/Telegram/Google Chat فقط). الافتراضي: `خطأ`.

#### `messages.tts`

تمكين النص إلى الكلام للردود الصادرة. عند التفعيل، ينشئ OpenClaw الصوت
باستخدام ElevenLabs أو OpenAI ويرفقه بالردود. Telegram يستخدم ملاحظات صوتية Opus
؛ قنوات أخرى ترسل الصوت MP3.

```json5
16. {
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

ملاحظات:

- 'messages.tts.auto' تحكم auto<unk> TTS ('off`، 'دائما`، 'inbound`، 'tagged`).
- '/tts off<unk> دائما <unk> inbound<unk> tagged' يقوم بتعيين الوضع التلقائي للدورة (تجاوز التكوين).
- 'messages.tts.enabled' هو إرث؛ ينقله الطبيب إلى 'messages.tts.auto\`.
- مخزن "PrefsPath" التجاوزات المحلية (المزود/الحد/الموجز).
- \`maxTextLength' هو حد ثابت لمدخلات TTS؛ الملخصات مقتطعة لتلائم ذلك.
- 'summaryModel' يلغي 'agents.defaults.model.primary' للملخص التلقائي.
  - يقبل `provider/model` أو اسم مستعار من `agents.defaults.models`.
- 'modelOverrides' تمكن التجاوزات المدفوعة بنموذج مثل '[[tts:...]]' (بشكل افتراضي).
- '/tts limit' و '/tts summary' التحكم في إعدادات ملخص المستخدم لكل مستخدم.
- القيم "apiKey" تعود إلى `ELEVENLABS_API_KEY`/`XI_API_KEY` و `OPENAI_API_KEY`.
- يتخطى موقع الـ ElevenLabs API الأساسي URL لـ '11labs.baseUrl'.
- يدعم `11.labs.voiceSettings` `stability`/`similarityBoost`/`style` (0..1)، و
  `usespeakerBoost`، و `السرعة` (0.5..2.0).

### `تحدث`

الافتراضي لوضع الحديث (macOS/iOS/Android). ترجع معرفات الصوت إلى `ELEVENLABS_VOICE_ID` أو `SAG_VOICE_ID` عند إلغاء تعيينها.
"apiKey" يعود إلى `ELEVENLABS_API_KEY` (أو ملف تعريف قذيفة البوابة) عند إلغاء تعيينه.
'voiceAliases' تتيح توجيهات الحديث استخدام أسماء ودية (على سبيل المثال 'صوت`: 'Clawd`).

```json5
{
  تحدث: {
    VoeId: "{ labs_voice_id",
    VoeAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    }،
    نموذج: "11._v3",
    OuttFormat: "mp3_44100_128",
    apiKey: "number@@1labs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

يتحكم في وقت تشغيل العامل المدمج (نموذج/تفكير/لفظ/مهل).
'agents.defaults.models' يحدد كتالوج النموذج المشكل (ويعمل كقائمة مسموح لـ '/model\`).
'agents.defaults.model.primary' يعين النموذج الافتراضي؛ 'agents.defaults.model.fallbacks' هي إخفاقات عالمية.
'agents.defaults.imageModel' اختياري ويتم **استخدامه فقط إذا كان النموذج الأساسي يفتقر إلى إدخال الصورة**.
يمكن لكل مدخل "agents.defaults.models" أن يشمل ما يلي:

- 'alias` (اختصار النموذج الاختياري، مثل '/opus`).
- `params` (شروط API الاختيارية الخاصة بموظفي الخدمة تمرير إلى طلب النموذج).

'params' تطبق أيضا على عمليات البث (وكيل مدمج + compaction). المفاتيح المدعومة اليوم: `درجة الحرارة`، `maxTokens`. تدمج هذه الخيارات مع خيارات وقت المكالمة؛ وفوز القيم التي يقدمها المتصلون. 'درجة الحرارة\` هي عقدة متقدمة - تترك بدون تعيين ما لم تكن تعرف الإعدادات الافتراضية للنموذج وتحتاج إلى تغيير.

مثال:

```json5
{
  الوكلاء: {
    الافتراضي: {
      models: {
        "الإنسان / claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5. ": {
          Params: { maxTokens: 8192 },
        }،
      },
    },
  },
}
```

نماذج Z.AI GLM-4.x تمكين وضع التفكير تلقائيًا ما لم تقم بما يلي:

- تعيين `--التفكير إيقاف`، أو
- تعريف `agents.defaults.models["zai/<model>"].params.thinking` بنفسك.

كما أن OpenClaw يشحن بعض الاختصارات المستعارة المدمجة. الافتراضي ينطبق فقط عندما يكون النموذج
موجود بالفعل في `agents.defaults.models`:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `الإنسان / claude-sonnett-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini' -> `google/gemini-3-pro-preview\`
- `gemini-flash` -> `google/gemini-3-flash-preview`

إذا قمت بتكوين نفس الاسم المستعار (حساس لحالة الأحرف) بنفسك، فإن القيمة الخاصة بك تفوز (الافتراضي لا يتجاوز مطلقاً).

على سبيل المثال: Opus 4.6 الاساسي مع MiniMax M2.1

```json5
{
  الوكلاء: {
    الافتراضي: {
      models: {
        "الإنسان / claude-opus-4-6": { alias: "opus" }،
        "minimax/MiniMax-M2. ": { alias: "minimax" },
      }،
      نموذج: {
        في المقام الأول: "الانسان/claude-opus-4-6"،
        سقاطات الظهور: ["minimax/MiniMax-M2. "]،
      }،
    }،
  }،
}
```

MiniMax wri: تعيين `MINIMAX_API_KEY` (env) أو تكوين `models.providers.minimax`.

#### `agents.defaults.cliBackends` (CLI fallback)

دعم CLI اختياري لتشغيل النسخ الاحتياطية فقط (لا مكالمات أدوات). هذه مفيدة كمسار نسخ إحتياطي
عندما يفشل موفري API. يتم دعم مرور الصورة عند تكوين
`imageArg` الذي يقبل مسارات الملف.

ملاحظات:

- نسخ CLI هي **النص أولاً**؛ الأدوات معطلة دائمًا.
- يتم دعم الجلسات عندما يتم تعيين \`sessionArg'؛ يتم الإبقاء على معارف الجلسة في كل الخلفية.
- بالنسبة لـ 'clauD-cli'، الافتراضي يتم سلكيا. تجاوز مسار الأوامر إذا كان PATH الحد الأدنى
  (تشغيل/نظام).

مثال:

```json5
17. {
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

```json5
18. {
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### 'agents.defaults.contextPruning' (تقسيم الأدوات - نتيجة)

وتقاطع `agents.defaults.contextPruning' **نتائج الأداة القديمة** من سياق الذاكرة مباشرة قبل إرسال طلب إلى إدارة تراخيص المشروبات الكحولية.
إنه يفعل **لا** تعديل سجل الجلسات على القرص ('*.jsonl` لا يزال كاملاً).

والغرض من ذلك هو الحد من استخدام العملات الرمزية لعوامل الدردشة التي تجمع نواتج كبيرة من الأدوات على مر الزمن.

المستوى العالي:

- لا تلمس رسائل المستخدم/المساعد.
- يحمي آخر رسائل المساعد "keeplastAssistants" (لا توجد نتائج للأداة بعد تلك النقطة مطبعة).
- يحمي بادئة التمهيد (لا شيء قبل طباعة رسالة المستخدم الأولى).
- الأساليب:
  - `تكييفي`: نواتج الأدوات الزائدة الحجم (حافظ على الرأس / الذيل) عندما تتقاطع نسبة السياق المقدرة مع `softTrimRatio`.
    19. ثم يقوم بمسح نتائج الأدوات الأقدم المؤهلة مسحًا صارمًا عندما تتجاوز نسبة السياق المقدّرة `hardClearRatio` **و**
    يكون هناك حجم كافٍ من نتائج الأدوات القابلة للتقليم (`minPrunableToolChars`).
  - `عدوانية`: يستبدل دائما نتائج الأداة المؤهلة قبل القطع بـ \`hardclear.placeholder' (بدون فحص النسبة).

(ما هي التغيرات في السياق التي أُرسلت إلى جيش تحرير السودان):

- **Soft-trim**: فقط لـ _oversized_ نتائج الأداة. الحفاظ على البداية + النهاية وإدراج `...` في الوسط.
  - سابقاً: `نتائج الأداة("…مخرجات طويلة جداً…")`
  - بعد: `نتائج الأداة("HEAD…\n...\n…TAIL\n\n[نتائج الأداة ممزوجة: …]")`
- **Hard-clear**: استبدل نتيجة الأداة بأكملها بالعنصر النائب.
  - سابقاً: `نتائج الأداة("…مخرجات طويلة جداً…")`
  - بعد ذلك: `نتائج الأداة("[تم مسح محتوى نتيجة الأداة القديمة]")`

الملاحظات / القيود الحالية:

- نتائج الأداة التي تحتوي على **تم تخطي كتل الصور** (لم يتم قط/مسحها) الآن.
- وتستند "نسبة السياق" المقدرة إلى **الحروف** (تقريبية)، وليس العملات الرمزية بالضبط.
- إذا كانت الجلسة لا تحتوي على الأقل على رسائل مساعد 'keeplastAssistants' بعد، يتم تخطي التقويم.
- وفي الوضع `العدواني'، يتم تجاهل `hardclear.enabled' (يستعاض دائما عن نتائج الأداة المؤهلة بـ `hardclear.placeholder`).

الافتراضي (قابل للتكيف):

```json5
{
  الوكلاء: { defaults: { contextPruning: { mode: "adaptive" } },
}
```

لتعطيل:

```json5
{
  الوكلاء: { defaults: { contextPruning: { mode: "off" } },
}
```

الافتراضي (عندما يكون `وضع` `متكيف"` أو `عدواني`):

- `keepLastAssistants`: `3`
- `softTrimRatio`: \`0.<unk> (قابل للتكيف فقط)
- `hardclearRatio`: \`0.5' (قابل للتكيف فقط)
- `minPrunableToolChars`: `50000` (التكيُّف فقط)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (قابل للتكيف فقط)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

مثال (عدواني، حد أدنى):

```json5
{
  الوكلاء: { defaults: { contextPruning: { mode: "aggressive" } },
}
```

مثال (مترقب تكييفي):

```json5
{
  الوكلاء: {
    الافتراضي: {
      contextPruning: {
        وضع : "تكيفي"،
        keeplastAssistans: 3,
        softTrimRatio: 0.
        hardClearRatio: 0. ,
        minPrunableolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardclear: {مفعل: صحيح, العنصر النائب: "[المحتوي الناتج عن الأداة القديمة محذوف]" },
        // اختياري: تقييد الطباعة إلى أدوات محددة (ينتصر فيه؛ يدعم "*" البطاقات البرية)
        أدوات: { deny: ["المتصفح"، "كانفاس"] }،
      },
    },
  },
}
```

انظر [/concepts/session-pruning](/concepts/session-pruning) للحصول على تفاصيل السلوك.

#### \`agents.defaults.compaction' (احتياطي الصفحة + فلتر الذاكرة)

'agents.defaults.compaction.mode' يختار استراتيجية تلخيص المدمجة. الافتراضي إلى 'الافتراضي\`؛ قم بتعيين 'حماية' لتمكين ملخص مقسم لتاريخ طويل جدا. انظر [/concepts/compaction](/concepts/compaction).

'agents.defaults.compaction.reserveTokensFloor' يفرض قيمة الحد الأدنى لـ 'reserveTokens'
لـ Pi compaction (الافتراضي: '20000`). عيّنه إلى `0\` لتعطيل الأرض.

'agents.defaults.compaction.memoryFlush' يقوم بتشغيل دورة **صامتة** قبل
تلقائيًا ، حيث يوعز إلى النموذج بتخزين الذاكرة الدائمة على القرص (على سبيل المثال:
`memory/YYY-MM-DD.md`). إنه يشغل عندما يتجاوز تقدير رمز الجلسة عتبة
ناعمة تحت الحد الفاصل.

الافتراضي القديم:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: الإفتراضية المدمجة مع `NO_REPLY`
- ملاحظة: يتم تخطي مسح الذاكرة عندما تكون مساحة العمل للقراءة فقط
  (`agents.defaults.sandbox.workspaceAccess: "ro"` أو `"none"`).

مثال (مضغوط):

```json5
20. {
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

حظر التدفق:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (معطّل افتراضيًا).

- تجاوز القناة: '_.blockStreaming' (ومتغيرات كل حساب) لإجبار حظر البث على تشغيل/إيقاف.
  تتطلب القنوات غير تيليجرام '_.blockStreaming: true' صراحة لتمكين ردود الكتل.

- `agents.defaults.blockStreamingBreak`: `"text_end"` أو `message_end"` (default: text_end).

- `agents.defaults.blockStreamingChunk`: القطع الناعم لكتل البث. الافتراضي إلى
  800–1200 حرف، يفضل فترات استراحة الفقرات (`\n\n`)، ثم الأسطر الإخبارية، ثم الجمل.
  مثال:

  ```json5
  {
    الوكلاء: { الإفتراضي: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: دمج كتل البث قبل الإرسال.
  الافتراضي إلى `{ idleMs: 1000 }` وترث `minChars` من `blockStreamingChunk`
  مع `maxChars` كحد أقصى لنص القناة. إشارة / كلاك/ديسكورد/جوجل الدردشة الافتراضية
  إلى `minChars: 1500` ما لم يتم تجاوزها.
  تجاوز القناة: `channels.whatsapp.blockStreamingCoalesce`، `channels.telegram.blockStreamingCoalesce`،
  `channels.discord.blockStreamingCoalesce`، `channels.slack.blockStreamingCoalesce`، `channels.mattermost.blockStreamingCoalesce`،
  `channels.signal.blockStreamingCoalesce`، `channels.blockStreamingCoalesce`، `channels.imessage.blockStreamingCoalesce`، `، `channels.msteams.blockStreamingCoalesce`،
  `channels.golechat.blockaming.Streamesce\`
  (وتنوعا مختلفا).

- `agents.defaults.humanDelay`: إيقاف مؤقت عشوائي بين **ردود الكتلة** بعد الأولى.
  أوضاع: `إيقاف` (الافتراضي)، `طبيعي` (800–2500ms)، `custom` (use `minMs`/`maxMs`).
  التجاوز: `الوكلاء.list[].humanDelay`.
  مثال:

  ```json5
  {
    الوكلاء: { defaults: { humanDelay: { mode: "طبيعي" } },
  }
  ```

  راجع [/concepts/streaming](/concepts/streaming) للسلوك + تفاصيل القطع.

مؤشرات الكتابة:

- `agents.defaults.typingMode`: `"unver" <unk> "instant" <unk> "thinking" <unk> "message"`. الافتراضي إلى
  'instant' للمحادثة المباشرة / ذكر و 'رسالة' لمحادثات المجموعة غير المذكورة.
- `session.typingMode`: تجاوز لكل جلسة للوضع.
- 'agents.defaults.typingIntervalSeconds': كم من المرات يتم تحديث إشارة الكتابة (الافتراضي: 6).
- 'session.typingIntervalSeconds': تجاوز الفاصل الزمني للتحديث.
  انظر [/concepts/typing-indicators](/concepts/typing-indicators) للحصول على تفاصيل السلوك.

'agents.defaults.model.primary' يجب أن يكون 'provider/model' (على سبيل المثال: 'anthropic/claude-opus-4-6`).
الأسماء المستعارة تأتي من `agents.defaults.models.\*.alias`(على سبيل المثال`Opus`).
إذا حذفت مقدم الخدمة، يفترض OpenClaw حاليًا أن 'أنثروبيك` كرد فعل مؤقت
خروج.
نماذج Z.AI متاحة كـ `zai/<model>(على سبيل المثال `zai/glm-4.7`) وتتطلب
`ZAI_API_KEY`(أو الإرث`Z_AI_API_KEY\`) في البيئة.

إعدادات \`agents.defaults.heartbeat' تعمل بشكل دوري heartbeat:

- `كل`: سلسلة المدة (`ms`, `s`, `m`, `h`)؛ دقائق الوحدة الافتراضية. الافتراضي:
  `30 م`. تعيين `0m` لتعطيل.
- `نموذج`: نموذج تجاوز اختياري لتشغيل القلب (`مزود/نموذج`).
- 'includeReasoning`: عندما 'true'، فإن دبات القلب ستنقل أيضا رسالة 'السبب` المنفصلة عندما تكون متاحة (نفس شكل '/preing on`). الافتراضي: `خطأ\`.
- `الدورة`: مفتاح الدورة الاختيارية للتحكم في أي جلسة تدخل فيها نبضات القلب. الافتراضي: `main`.
- `إلى`: تجاوز المستلم الاختياري (معرف قناة محددة، على سبيل المثال E.164 لـ WhatsApp، معرف الدردشة لتيليغرام).
- `target`: قناة تسليم اختيارية (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). الافتراضي: `الأخير`.
- 'سرعة`: تجاوز اختياري لجسم القلب (الافتراضي: 'قراءة HEARTBEAT.md إذا كان موجودا (سياق فضاء العمل). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). يتم إرسال التجاوزات حرفياً؛ قم بتضمين سطر "قراءة HEARTBEAT.md" إذا كنت لا تزال تريد قراءة الملف.
- `ackMaxChars`: الحد الأقصى للحروف المسموح بها بعد `HEARTBEAT_OK` قبل التسليم (الافتراضي: 300).

نبضات القلب لكل وكيل:

- تعيين `agents.list[].heartbeat` لتمكين أو تجاوز إعدادات القلب لوكيل معين.
- إذا كان أي وكيل يعرّف 'ضربة القلب\`، **فقط هؤلاء الوكلاء** يشتغلون بضبات القلب؛ الإعدادات الافتراضية
  تصبح خط الأساس المشترك لهؤلاء الوكلاء.

تشغّل نبضات القلب دورات وكيل كاملة. 21. الفواصل الزمنية الأقصر تستهلك رموزًا (tokens) أكثر؛ انتبه إلى `every`، واجعل `HEARTBEAT.md` صغيرًا جدًا، و/أو اختر `model` أرخص.

إعدادات \`tools.exec' الافتراضية للخلفية الخارجية:

- `الخلفية`: الوقت قبل الخلفية التلقائية (مللي ، الافتراضي 10000)
- `timeoutSec`: القتل التلقائي بعد فترة التشغيل هذه (ثواني، افتراضي 1800)
- `CleupMs`: كم من الوقت للحفاظ على الجلسات المنتهية في الذاكرة (مم، الافتراضي 1800000)
- `notifyOnExit`: enqueue a system event + request heartbeat عند الخروج من خارج الخلفية (الافتراضي صحيح)
- `applicyPatch.enabled`: تمكين تجربة `applicy_patch` (OpenAI/OpenAI Codex فقط؛ خطأ افتراضي)
- 22. `applyPatch.allowModels`: قائمة سماح اختيارية لمعرّفات النماذج (مثل `gpt-5.2` أو `openai/gpt-5.2`)
      ملاحظة: يتوفر `applyPatch` فقط ضمن `tools.exec`.

إعدادات 'tools.web' البحث على الويب + جلب أدوات :

- \`tools.web.search.enabled' (الافتراضي: صحيح عند وجود المفتاح)
- `tools.web.search.apiKey` (موصى به: تعيين عن طريق `openclaw configure --section web`، أو استخدام `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults` (1–10، الافتراضي 5)
- `tools.web.search.timeoutSeconds` (الافتراضي 30)
- `tools.web.search.cacheTtlMinutes` (الافتراضي 15)
- `tools.web.fetch.enabled` (الافتراضي)
- `tools.web.fetch.maxChars` (الافتراضي 50000)
- \`tools.web.fetch.maxCharsCap' (الافتراضي 50000; shps maxChars من config/tool calls)
- `tools.web.fetch.timeoutSeconds` (الافتراضي 30)
- `tools.web.fetch.cacheTtlMinutes` (الافتراضي 15)
- `tools.web.fetch.userAgent` (تجاوز اختياري)
- \`tools.web.fetch.readability' (الافتراضي صحيح؛ تعطيل استخدام تنظيف HTML الأساسي فقط)
- `tools.web.fetch.firecrawl.enabled` (الافتراضي صحيح عندما يتم تعيين مفتاح API)
- `tools.web.fetch.firecrawl.apiKey' (اختياري؛ الافتراضي إلى `FIRECRAWL_API_KEY\`)
- `tools.web.fetch.firecrawl.baseUrl` (الافتراضي [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (الافتراضي صحيح)
- `tools.web.fetch.firecrawl.maxAgeMs` (اختياري)
- `tools.web.fetch.firecrawl.timeoutSeconds` (اختياري)

إعدادات `tools.media` لفهم الوسائط الواردة (image/audio/video):

- `tools.media.models`: قائمة نموذجية مشتركة (بطاقة القدرة؛ تستخدم بعد قوائم الحد الأقصى).
- `tools.media.concurrency`: أقصى قدرة متزامنة (الافتراضي 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - 'تمكين\`: اختيار عدم التبديل (الافتراضي صحيح عندما يتم تكوين النماذج).
  - `سرعة`: تجاوز فوري اختياري (صورة/فيديو إلحاق تلميح `maxChars` تلقائياً).
  - `maxChars`: الحد الأقصى لإخراج الأحرف (الافتراضي 500 للصورة/الفيديو؛ إلغاء تعيين الصوت).
  - `maxBytes`: الحد الأقصى لحجم الوسائط المرسلة (الافتراضي: صورة 10MB، الصوت 20MB، الفيديو 50MB).
  - 'timeoutSeconds': مهلة الطلب (الافتراضي: الصورة 60، الصوت 60، الفيديو 120).
  - `اللغة`: تلميح صوتي اختياري.
  - `المرفقات`: سياسة المرفقات (`نموذج`، `maxattachments`، `المفضل`).
  - `scope`: بوابة اختيارية (أول فوز) مع `match.channel` أو `match.chatType` أو `match.keyPrefix`.
  - `النماذج`: قائمة مرتبة بمدخلات النموذج؛ وتعود الأخطاء أو الوسائط الزائدة الحجم إلى الإدخال التالي.
- إدخال كل 'نماذج[]:
  - إدخال مقدم الخدمة (\`نوع: "مزود" أو محذوف):
    - `مقدم الخدمة`: معرف مزود API (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc).
    - `model`: model id overide (مطلوب للصورة؛ الافتراضي إلى `gpt-4o-mini-transcribe`/`whisper-big v3-turbo` لمزودي الصوت، و `gemini-3-flash-preview` للفيديو).
    - 'الملف الشخصي` / 'تفضيل البروفايل`: اختيار الملف الشخصي للمصادقة.
  - مدخل CLI (`نوع: "العميل"`):
    - `أمر`: قابل للتنفيذ للتشغيل.
    - `args`: قوالب الأرجف (يدعم`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, الخ).
  - `القدرات`: القائمة الاختيارية (`image` و`audio` و`video`) لبوابة إدخال مشترك. الافتراضي عند حذفه: `openai`/`anthropic`/`minimax` → صورة، `google` → image+audio+video, `groq` → صوت.
  - `prompt`، `maxChars`، `maxBytes`، `timeoutSeconds`، `اللغة` يمكن تجاوزها لكل إدخال.

إذا لم يتم تكوين أي نماذج (أو 'تمكين: خطأ\`)، يتم تخطي الفهم؛ ولا يزال النموذج يتلقى المرفقات الأصلية.

مصادقة موفر يتبع ترتيب المصادقة النموذجي القياسي (Auth profiles, env vars مثل `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`، أو `models.providers.*.apiKey`).

مثال:

```json5
23. {
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

إعدادات `agents.defaults.subagents` الافتراضية:

- `نموذج`: النموذج الافتراضي للوكلاء الفرعيين الذين تم تفريغهم (سلسلة أو `{ primary, fallbacks }`). وإذا تم إغفاله، يرث الوكلاء الفرعيون نموذج المتصل، ما لم يتم تجاوزه لكل وكيل أو لكل مكالمة.
- `maxConcurrent`: يعمل الحد الأقصى للوكيل الفرعي المتزامنة (الافتراضي 1)
- `archiveAfterMinutes`: جلسات وكيل فرعي الأرشيف التلقائي بعد N دقائق (الافتراضي 60؛ تعيين '0\` لتعطيل)
- سياسة أداة كل عامل فرعي: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny winins)

'tools.profile' يحدد **قائمة للأداة الأساسية** قبل `tools.allow`/`tools.deny`:

- `minimal`: `session_status` فقط
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: بلا قيود (مماثل لعدم الضبط)

تجاوز لكل وكيل: `agents.list[].tools.profile`.

مثال (الرسائل فقط افتراضيًا، والسماح بأدوات Slack وDiscord أيضًا):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

مثال (ملف تعريف البرمجة، لكن منع exec/process في كل مكان):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

تتيح لك `tools.byProvider` **المزيد من الأدوات المقيدة** لمزودين معينين (أو 'مزود/نموذج`واحد).
تجاوز لكل وكيل:`agents.list[].tools.byProvider\`.

الطلب: الملف الشخصي الأساسي → ملف تعريف المزود → السماح / رفض السياسات.
تقبل مفاتيح المزود إما `المزود` (مثل `google-antivity `) أو `المزود/model`
(على سبيل المثال `openai/gpt-5.2`).

مثال (الإبقاء على ملف تعريف البرمجة العام، لكن أدوات حدّية لـ Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

مثال (قائمة السماح لموفر/نموذج محدد):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

'tools.allow' / 'tools.deny' تكوين أداة عالمية تسمح / ترفض السياسة (فوز).
المطابقة غير حساسة لحالة الأحرف وتدعم '_' البطاقات البرية ('_' تعني جميع الأدوات).
يتم تطبيق هذا حتى عندما يكون صندوق رمل Docker **إيقاف**.

مثال (تعطيل المتصفح/قماش في كل مكان):

```json5
{
  أداة: { deny: ["المتصفح"، "canvas"] },
}
```

مجموعات الأدوات (الاختصار) تعمل في سياسات الأدوات **العالمية** و **لكل وكيل**:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: جميع أدوات OpenClaw المدمجة (باستثناء إضافات الموفّرين)

أدوات التحكم 'tools.elevated' ارتفعت (المضيف) إلى الخارج:

- `تمكين`: السماح بالوضع المرتفع (صحيح افتراضي)
- `allowFrom`: قوائم السماح لكل قناة (فارغة = معطلة)
  - `Whatsapp`: E.164 أرقام
  - `تلغرام`: معارف الدردشة أو أسماء المستخدمين
  - `discord`: معلمات المستخدم أو أسماء المستخدمين (يعود إلى \`channels.discord.dm.allowFrom' إذا كان محذوف)
  - `إشارة`: أرقام E.164
  - `التسلسل`: معارف الأنابيب/الدردشة
  - `webchat`: معارف الجلسة أو أسماء المستخدمين

مثال:

```json5
{
  أدوات: {
    مرتفعة: {
      مفعل: صحيح,
      السماح من: {
        أي تطبيق: ["+155550123"]،
        Discord: ["steipete"، "1234567890123"]،
      },
    },
  },
}
```

التجاوز لكل وكيل (قيد إضافي):

```json5
24. {
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

ملاحظات:

- 'tools.elevated' هو خط الأساس العالمي. 'agents.list[].tools.elevated' لا يمكن إلا أن يزيد من القيود (كلاهما يجب أن يسمح بذلك).
- حالة مخزن '/رفع على <unk> off<unk> ask<unk> full' لكل مفتاح جلسة؛ تنطبق التوجيهات المضمنة على رسالة واحدة.
- يرفع "exec" على المضيف ويتجاوز صندوق الرمال.
- ولا تزال سياسة الأدوات تنطبق؛ وإذا رُفض "exec"، لا يمكن استخدام الزيادة في الأداء.

'agents.defaults.maxConcurrent' يعين الحد الأقصى لعدد عمليات تشغيل الوكيل المدمجة التي يمكن
تنفيذها بالتوازي عبر الجلسات. كل جلسة لا تزال متسلسلة (تشغيل واحد
لكل مفتاح جلسة في كل مرة). الافتراضي: 1.

### `agents.defaults.sandbox`

اختياري **صندوق رمل دوكر** للعميل المدمج. مخصص للجلسات غير الرئيسية
حتى لا يستطيعون الوصول إلى نظام المضيف الخاص بك.

التفاصيل: [Sandboxing](/gateway/sandboxing)

الافتراضي (إذا تم تفعيل):

- نطاق: `الوكيل"` (حاوية واحدة + مساحة عمل لكل وكيل)
- صورة مستندة إلى ديدان ديبا
- وكيل الوصول إلى فضاء العمل: `مساحات العمل: "لا شيء"` (الافتراضي)
  - `لا شيء"`: استخدم مساحة عمل لكل نطاق تحت `~/.openclaw/sandboxes`
- `ro"`: حافظ على مساحة العمل في `/workspace`، وقم بتركيب مكان العمل العامل للقراءة فقط في `/agent` (تعطل `الكتابة`/`edit`/`applicy_patch`)
  - `"rw"`: تركيب وكيل مساحة العمل يقرأ/يكتب في `/workspace`
- التنظيف التلقائي: خمول > 24 ساعة أو العمر > 7 أيام
- سياسة الأدوات: السماح فقط `exec`, `process`, `read`, `write`, `edit`, `applicy_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (eny wins)
  - تكوين عبر `tools.sandbox.tools`، تجاوز لكل وكيل عبر `agents.list[].tools.sandbox.tools`
  - اختصارات مجموعة الأدوات المدعومة في سياسة الصندوق الرملي: `group:runtime`، `group:fs`، `group:sessions`، `group:memory` (انظر [Sandbox vs Tool Policy vs Upvated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- المتصفح الرملي الاختياري (Chromium + CDP, noVNC Obser)
- التعقيدات المتشددة: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

تحذير: `النطاق: "المشتركة"` يعني حاوية مشتركة ومساحة عمل مشتركة. لا
عزلة ما بين الدورات. استخدم `النطاق: "دورة"` للعزل في كل دورة.

الإرث: `perSession` لا يزال مدعوماً (`true` → `scope: "session"`,
`false` → `scope: "شارك"`).

'إعداد الأوامر' يعمل **مرة واحدة** بعد إنشاء الحاوية (داخل الحاوية عن طريق 'sh -lc\`).
لتثبيت الحزمة، تأكد من تقدم الشبكة، وجذر FS قابل للكتابة، ومستخدم الجذر.

```json5
25. {
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

بناء صورة صندوق الرمل الافتراضية مرة واحدة:

```bash
scripts/sandbox-setup.sh
```

ملاحظة: الحاويات الافتراضية لـ `Net: "none"`؛ قم بتعيين `agents.defaults.sandbox.docker.network`
إلى `"bridge"` (أو الشبكة المخصصة) إذا كان الوكيل يحتاج إلى الوصول إلى الخارج.

ملاحظة: يتم تجهيز المرفقات الواردة في حيز العمل النشط في `media/inint/*`. مع `مساحات العمل: "rw"`، وهذا يعني أن الملفات مكتوبة في مساحة عمل الوكيل.

ملاحظة: 'docker.binds' يتكون من أدلة مضيفة إضافية؛ يتم دمج الروابط العالمية والملزمة.

قم بإنشاء صورة المتصفح الاختيارية مع:

```bash
scripts/sandbox-browser-setup.sh
```

عندما 'agents.defaults.sandbox.browser.enabled=true\`، تستخدم أداة المتصفح مثيل Sandboxed
Chromium (CDP). إذا تم تمكين noVNC (الافتراضي عند headless=false)،
يتم حقن رابط noVNC في موجه النظام حتى يتمكن الوكيل من الرجوع إليه.
هذا لا يتطلب 'browser.enabled' في التكوين الرئيسي؛ يتم حقن رابط sandbox
في كل دورة.

`agents.defaults.sandbox.browser.allowhostControl` (الافتراضي: false) يسمح لجلسات
sandboxed أن تستهدف صراحة خادم **المضيف** للتحكم في المتصفح
عن طريق أداة المتصفح (`target: "host"`). 26. اترك هذا الخيار معطّلًا إذا كنت تريد عزلًا صارمًا لصندوق الحماية.

السماح بقوائم وحدة التحكم عن بعد:

- `allowedControlUrls`: عناوين URL المحددة للتحكم المسموح بها لـ `الهدف: "custom"`.
- 'allowedControlHosts\`: أسماء المضيفين المسموح بها (المضيف فقط، لا ميناء).
- `allowedControlPorts`: الموانئ المسموح بها (الافتراضي: http=80، https=443).
  الافتراضي: جميع قوائم السماح غير محددة (لا يوجد قيد). الافتراضي 'allowhostControl' إلى الكاذب.

### `النماذج` (المزودين المخصصين + عناوين URL الأساسية)

يستخدم OpenClaw الكتالوج النموذجي **P-coding-agent**. يمكنك إضافة موفري مخصصين
(LiteLM، خوادم متوافقة مع OpenAI، وكلاء أنثروبيك، إلخ.) بكتابة
`~/.openclaw/agents/<agentId>/agent/models.json` أو بتحديد نفس المخطط داخل اعدادات OpenClaw الخاصة بك
تحت `models.providers`.
نظرة عامة لكل موفر + أمثلة: [/concepts/model-providers](/concepts/model-providers).

عندما يكون `models.providers` حاضراً، يكتب OpenClaw /يدمج `models.json` في
`~/.openclaw/agents/<agentId>/agent/` عند البدء:

- السلوك الافتراضي: **الدمج** (الحفاظ على مقدمي الخدمات الحاليين، التجاوزات على الاسم)
- تعيين `models.mode: "replace"` لاستبدال محتويات الملف

حدد النموذج عن طريق `agents.defaults.model.primary` (provider/model).

```json5
27. {
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {},
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### رمز OpenCode Zen (وكيل متعدد الطراز)

رمز OpenCode Zen هو بوابة متعددة النماذج مع نقاط نهاية لكل نموذج. يستخدم OpenClaw
موفر `opencode` المدمج من pi-ai; تعيين `OPENCODE_API_KEY` (أو
`OPENCODE_ZEN_API_KEY`) من [https://opencode.ai/auth](https://opencode.ai/auth).

ملاحظات:

- النموذج يرفض استخدام `opencode/<modelId>` (مثال: `opencode/claude-opus-4-6`).
- إذا قمت بتمكين قائمة السماح عن طريق `agents.defaults.models`، قم بإضافة كل نموذج تخطط لاستخدامه.
- اختصار: `openclaw onboard--Opth-Opcode-zen`.

```json5
{
  الوكلاء: {
    الإفتراضي: {
      model: { preary: "opencode/claude-opus-4-6" }،
      النماذج: { "opencode/claude-opus-4-6": { alias: "Opus" } }،
    },
  },
}
```

### Z.AI (GLM-4.7) - دعم مستعار مقدم الخدمة

وتتوفر نماذج Z.AI عن طريق مزود 'zai`المدمج. تعيين`ZAI_API_KEY\`
في بيئتك والإشارة إلى النموذج بواسطة المزود/النموذج.

اختصار: `openclaw onboard--قم باختيار zai-api-key`.

```json5
{
  الوكلاء: {
    الإفتراضي: {
      model: { preary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} ,
    },
  },
}
```

ملاحظات:

- 'z.ai/_' و 'z-ai/_' يقبلان الأسماء المستعارة ويعملان على تطبيع 'zai/\*\`.
- إذا كان 'ZAI_API_KEY' مفقودا، فإن طلبات 'zai/\*' ستفشل مع خطأ المصادقة في وقت التشغيل.
- خطأ مثال: `لم يتم العثور على مفتاح API للمزود "زاي".`
- نقطة نهاية Z.AI العامة هي `https://api.z.ai/api/paas/v4`. طلبات برمجة GLM
  تستخدم نقطة نهاية الترميز المخصصة `https://api.z.ai/api/coding/paas/v4`.
  يستخدم موفر 'zai' المدمج نقطة نهاية الترميز. إذا كنت بحاجة إلى
  endpoint العامة، قم بتحديد موفر مخصص في `models.providers` مع تجاوز URL الأساسي
  (انظر قسم مقدمي الخدمات المخصص أعلاه).
- استخدام عنصر نائب مزيف في المستندات/التكوينات؛ لا يخصص أبدا مفاتيح API الحقيقية.

### Moonshot AI (Kimi)

استخدم نقطة طرفية متوافقة لقطة القمر:

```json5
28. {
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

ملاحظات:

- تعيين `MOONSHOT_API_KEY` في البيئة أو استخدام `openclaw onboard--قم باختيار القمر api-key`.
- المرجع النموذجي: `القمر/kimi-k2.5`.
- بالنسبة لنقطة نهاية الصين، إما:
  - تشغيل `openclaw onboard--auth-choice moonshot-api-key-cn' (المعالج سيقوم بتعيين `https://api.moonshot.cn/v1\`)، أو
  - قم بتعيين `baseUrl: "https://api.moonshot.cn/v1"` في `models.providers.moonshot`.

### Kimi Coding

استخدم نقطة نهاية رمز كيمي (AI) لقطة القمر (متوافقة مع الأنثروبيك، مزود مدمج):

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

ملاحظات:

- تعيين `KIMI_API_KEY` في البيئة أو استخدام `openclaw onboard--قم باختيار kimi-code-api-key`.
- المرجع النموذجي: `ترميز kimi-coding/k2p5`.

### اصطناعي (متوافق مع الأنثروبيك)

استخدم نقطة النهاية المتوافقة مع الأنثروبيك:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

ملاحظات:

- تعيين `SYNTHETIC_API_KEY` أو استخدام `openclaw على متن - اختيار اصطناعي-api-key`.
- رقم النماذج: `التركيب/hf:MiniMaxAI/MiniMax-M2.1`.
- عنوان URL الأساسي يجب أن يحذف `/v1` لأن عميل Anthropic يرفقه.

### النماذج المحلية (LM Studio) - الإعداد الموصى به

انظر [/gateway/local-models](/gateway/local-models) للحصول على التوجيه المحلي الحالي. TL;DR: قم بتشغيل MiniMax M2.1 عبر LM Studio Responses API على المعدات الجادة؛ حافظ على النماذج المستضافة مدمجة للاسترداد.

### MiniMax M2.1

استخدم MiniMax M2.1 مباشرة بدون استوديو LM:

```json5
29. {
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Pricing: update in models.json if you need exact cost tracking.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

ملاحظات:

- تعيين متغير البيئة `MINIMAX_API_KEY` أو استخدام `openclaw على متن - auth-choice minimax-api`.
- النموذج المتاح: \`MiniMax-M2.1' (الافتراضي).
- تحديث التسعير في `models.json` إذا كنت بحاجة إلى تتبع دقيق للتكلفة.

### سيريبراس (GLM 4.6 / 4.7)

استخدام Cerebras عبر نقطة طرفية متوافقة مع OpenAI:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-... },
  الوكلاء: {
    الإفتراضي: {
      نموذج: {
        في المقام الأول: "cerebras/zai-glm-4. ""
        يعطي: ["cerebras/zai-glm-4. "]،
      }،
      النماذج : {
        "cerebras/zai-glm-4. ": { alias: "GLM 4.7 (Cerebras") },
        "cerebras/zai-glm-4.6": { alias: "GLM 4. (Cerebras)" },
      },
    },
  },
  نموذج: {
    : "الاندماج"
    مقدمي الخدمات: {
      cerebras: {
        baseUrl: "https://api. إيريبراس. i/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        نماذج [
          { id: "zai-glm-4. "، الاسم: "GLM 4. (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4. (Cerebras)" },
        ]،
      }،
    },
  },
}
```

ملاحظات:

- استخدام 'cerebras/zai-glm-4.7' للسيريبراس؛ استخدام 'zai/glm-4.7' للزي آي المباشر.
- تعيين `CEREBRAS_API_KEY` في البيئة أو التكوين.

ملاحظات:

- التطبيقات المدعومة: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- استخدم "Auteheader: true" + "headers" لاحتياجات المصادقة المخصصة.
- تجاوز جذر إعداد الوكيل مع `OPENCLAW_AGENT_DIR` (أو `PI_CODING_AGENT_DIR`)
  إذا كنت تريد تخزين `models.json` في مكان آخر (الافتراضي: `~/.openclaw/agents/main/agent`).

### `الجلسة`

التحكم في نطاق الجلسة، إعادة تعيين السياسة، إعادة تعيين المشغلات، وحيثما يتم كتابة متجر الجلسة.

```json5
`resetByType`: تجاوزات لكل جلسة لأنواع `direct` و`group` و`thread`.
```

الحقول:

- `mainKey`: مفتاح الدلو المباشر (الافتراضي: `main"). مفيد عندما تريد "إعادة تسمية" موضوع DM الأساسي دون تغيير `وكيل\`.
  - ملاحظة صندوق الرملة: `agents.defaults.sandbox.mode: "unmain"` يستخدم هذا المفتاح لكشف الجلسة الرئيسية. أي مفتاح جلسة لا يتطابق مع 'mainKey' (المجموعات/القنوات) هو مربع.
- `dmSscope`: كيفية تجميع جلسات DM (الافتراضي: `main"`).
  - `أساساً`: تشترك جميع مشاريع الطرائق في الدورة الرئيسية من أجل الاستمرارية.
  - `للأنداد`: عزل DMs بواسطة معرف المرسل عبر القنوات.
  - `لكل قناة-أقران`: عزل DMs لكل قناة + المرسل (موصى به للصندوقات الواردة متعددة المستخدمين).
  - 'per-account-channel-peer': عزل DMs لكل حساب + قناة + المرسل (موصى به لصناديق الوارد متعددة الحسابات).
  - وضع DM آمن (موصى به): تعيين `session.dmSscope: "per-channel-peer"` عندما يستطيع أشخاص متعددون تحويل البوت (صناديق الوارد المشتركة، قوائم السماح بتعدد الأشخاص، أو `dmPolicy: "open"`).
- `identityLinks`: خرائط معارف إلى أقران مزودين مسبقين بحيث يشارك نفس الشخص جلسة DM عبر القنوات عند استخدام `per-peer` أو `per-channel-peer` أو `per-account-channel-peer`.
  - مثال: `alice: ["telegram:123456789", "Discord:987654321012345678"]`.
- 'إعادة تعيين\`: سياسة إعادة تعيين الأولية. الافتراضي إلى إعادة الضبط اليومية في الساعة 4:00 بالتوقيت المحلي على البوابة.
  - `mode`: `daily` أو `idle` (الافتراضي: `daily` عندما يكون `reset` موجوداً).
  - `ساعة`: الساعة المحلية (0-23) لإعادة تعيين الحدود اليومية.
  - `idleMinutes`: نافذة خاملة منزلة في دقائق. عند تفعيل اليومي + الخمول معًا، يفوز أيّهما ينتهي أولًا.
- يتم قبول المفتاح القديم `dm` كاسم مستعار لـ `direct`. %%{init: {
  'theme': 'base',
  'themeVariables': {
  'primaryColor': '#ffffff',
  'primaryTextColor': '#000000',
  'primaryBorderColor': '#000000',
  'lineColor': '#000000',
  'secondaryColor': '#f9f9fb',
  'tertiaryColor': '#ffffff',
  'clusterBkg': '#f9f9fb',
  'clusterBorder': '#000000',
  'nodeBorder': '#000000',
  'mainBkg': '#ffffff',
  'edgeLabelBackground': '#ffffff'
  }
  }}%%
  flowchart TB
  subgraph Client["Client Machine"]
  direction TB
  A["OpenClaw.app"]
  B["ws://127.0.0.1:18789\n(local port)"]
  T["SSH Tunnel"]```
      A --> B
      B --> T
  end
  subgraph Remote["Remote Machine"]
      direction TB
      C["Gateway WebSocket"]
      D["ws://127.0.0.1:18789"]

      C --> D
  end
  T --> C
  ```
  - إذا قمت فقط بتعيين الإرث 'session.idleMinutes' دون أي 'reset\`/'resetByType'، يبقى OpenClaw في وضع الخمول فقط لتحقيق التوافق الخلفي.
- `قلب إدليمينوتس`: تجاوز خامل اختياري لفحوص نبض القلب (إعادة الضبط اليومية لا تزال تنطبق عند التمكين).
- 'agentToAgent.maxPingPongTurns': الحد الأقصى لدوران الرد - الظهر بين الطالب/الهدف (0-5، الافتراضي 5).
- `sendPolicy.default`: `allow' أو `deny' عند عدم وجود قاعدة متطابقة.
- `sendPolicy.rules[]`: match by `channel`, `chatType` (`direct<unk> group<unk> room`), or `keyPrefix` (مثل `cron:`). أولا رفض الفوز؛ وإلا اسمح

### `المهارات` (تكوين المهارات)

التحكم في قائمة السماح المجمعة، تثبيت التفضيلات، مجلدات المهارات الإضافية، وتجاوز كل مهارة
ينطبق على **حزم** المهارات و `~/.openclaw/skills` (مهارات مساحة العمل
لا يزال الفوز على تنازع الأسماء).

الحقول:

- `allowBundled`: قائمة سماح اختيارية لـ Skills **المضمّنة** فقط. 31. إذا تم تعيينه، فستكون فقط
  المهارات المجمّعة (bundled) مؤهلة (المهارات المُدارة/مهارات مساحة العمل غير متأثرة).
- `load.extraDirs`: أدلة Skills إضافية للمسح (أدنى أولوية).
- `install.preferBrew`: تفضيل مثبّتات brew عند توفرها (الافتراضي: true).
- 'install.nodeManager': تفضيل تثبيت العقدة ('npm' <unk> 'pnpm' <unk> 'yarn'، الافتراضي: npm).
- `entries.<skillKey>`: تجاوز التكوين لكل مهارة.

الحقول الخاصة بكل Skill:

- `enabled`: عيّن `false` لتعطيل Skill حتى لو كانت مضمّنة/مثبّتة.
- `env`: متغيرات البيئة التي تُحقن أثناء تشغيل الوكيل (فقط إذا لم تكن مضبوطة بالفعل).
- `apiKey`: الملاءمة الاختيارية للمهارات التي تعلن عن مخرج أساسي (مثل 'nano-Banana-pro` → 'GEMINI_API_KEY`).

مثال:

```json5
32. {
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### "الإضافات" (الملحقات)

التحكم في اكتشاف الاضافات، السماح/رفض التكوين، و لكل مكون إضافي. يتم تحميل الإضافات
من '~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, بالإضافة إلى أي إدخالات
`plugins.load.paths\`. **تغييرات التكوين تتطلب إعادة تشغيل البوابة.**
راجع [/plugin](/tools/plugin) للاستخدام الكامل.

الحقول:

- 'تمكين\`: التبديل الرئيسي لتحميل الإضافات (الافتراضي: صحيح).
- `سمح`: قائمة السماح الاختيارية بمعارف الإضافة؛ عند تعيينها، فقط تحميل الإضافات المدرجة في القائمة.
- `نكر`: القائمة الاختيارية لمعارف الإضافات (يني يفوز).
- 'load.paths': ملفات إضافية أو أدلة إضافية للتحميل (مطلقة أو '~\`).
- `إدخالات'.<pluginId>`: تجاوز المكونات الإضافية.
  - 'تمكين\`: تعيين 'خطأ' لتعطيل.
  - `config`: كائن تكوين خاص بالبرنامج المساعد (تم التحقق من صحتها بواسطة البرنامج المساعد إذا تم توفيره).

مثال:

```json5
33. {
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### 'المتصفح' (متصفح مفتوح المدار)

OpenClaw يمكن أن يبدأ مثيل **كروم/Brave/Edge/Chromium لـ openclaw ويكشف عن خدمة صغيرة للتحكم في الظهور.
يمكن للملفات الشخصية أن تشير إلى متصفح **عن بعد** يقوم على Chromium** عبر `ملفات تعريفية.<name>.cdpUrl`. الملفات الشخصية
مرفقة فقط (تشغيل/إيقاف/إعادة تعيين معطلة).

يبقى `browser.cdpUrl` لتكوينات الملف الشخصي الواحد القديمة وكقاعدة
مخطط / مضيف للملفات الشخصية التي تقوم فقط بتعيين `cdpPort`.

القيم الافتراضية:

- مفعل: `true`
- تقييم تمكين: 'true' (تعيين 'false' لتعطيل 'act:evaluate' و 'انتظر --fn')
- خدمة التحكم: حلقة استرجاع فقط (منفذ مشتقة من `gateway.port`، الافتراضي `18791`)
- رابط CDP: `http://127.0.0.1:18791` (خدمة التحكم + 1، ملف واحد الإرث)
- لون الملف الشخصي: `#FF4500` (جراد البرتقالي)
- ملاحظة: يتم تشغيل خادم التحكم عن طريق بوابة تشغيل (OpenClaw.app menubar, أو `openclaw بوابة`).
- طلب الكشف التلقائي: المتصفح الافتراضي إذا كان مستندا إلى الكروميوم؛ خلاف ذلك كروم → شجاعة → الحافة → Chromium III Chrome Canary.

```json5
{
  المتصفح: {
    مفعل: صحيح،
    تقييم تمكين: صحيح،
    /cdpUrl: "http://127. .0. :18792", // تركة تجاوز الملفّ الجانبي
    الافتراضي Profile: "كروم"، الملامح
    : {
      openclaw: { cdpPort: 18800, اللون: "#FF4500" },
      work: { cdpPort: 18801, اللون: "#0066CC" },
      بعيد: { cdpUrl: "http://10. .0.42:9222", col: "#00A00" },
    },
    col: "#FF4500",
    // متقدم :
    // بدون رأس: خطأ،
    // noSandbox: false,
    // executablePath: "/Applications/Browser. pp/contents/MacOS/Brave Browser",
    // ملحق Only: false, // تعيين صحيح عند نفق CDP بعيد إلى localhost
  },
}
```

### `ui` (المظهر)

لون اللكنة الاختيارية المستخدمة من قبل التطبيقات الأصلية لكروم واجهة المستخدم (على سبيل المثال صنف فقاعة وضع الحديث).

إذا كان غير محدد، فإن العملاء يسقطون مرة أخرى باللون الأزرق المكتمل.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB أو #RRGGBB)
    // اختياري: تجاوز هوية مساعد التحكم في واجهة المستخدم.
    // إذا لم يتم تعيين واجهة التحكم تستخدم هوية الوكيل النشط (التكوين أو IDENTITY). د).
    المساعد: {
      الاسم: "OpenClaw"،
      صورة: "CB"، // emoji ، نص قصير أو صورة URL/data URI
    },
  },
}
```

### `بوابة` (وضع خادم Gateway + ربط)

استخدم \`gateway.mode' للإعلان صراحة عما إذا كان ينبغي لهذه الآلة أن تشغل البوابة.

القيم الافتراضية:

- وضع: **إلغاء التشكيل** (تعامل كـ "لا تبدأ تلقائيا")
- bind: `loopback`
- المنفذ: '18789' (منفذ واحد لـ WS + HTTP)

```json5
{
  بوابة : {
    وضع "local", // أو "بعيد"
    منفذ 18789, // WS + HTTP multiplex
    ملزم: "حلقة"،
    // controlUi: { enabled: صحيح, مسار basePath: "/openclaw" }
    // auth: mode: "token", الرمز المميز: "to-token" } // / token بوابات WS + التحكم في الوصول إلى واجهة المستخدم
    // tailscale: modul: "off" <unk> "serve" <unk> "funnel" }
  }،
}
```

التحكم في مسار قاعدة واجهة المستخدم:

- 'gateway.controlUi.basePath' يعين بادئة عنوان URL حيث يتم خدمة واجهة التحكم.
- أمثلة: `"/ui"`، `"/openclaw"`، `"/apps/openclaw"`.
- الافتراضي: الجذر (`/`) (بدون تغيير).
- 'gateway.controlUi.root' يعين جذر filesystem لموجودات واجهة المستخدم (الافتراضي: `dist/control-ui`).
- 'gateway.controlUi.allowInsecureAuth' يسمح بالمصادقة الرمزية فقط في واجهة التحكم عندما يتم حذف هوية الجهاز
  (عادة عبر HTTP). الافتراضي: `خطأ`. تفضيل HTTPS
  (خدمة النطاق السري) أو `127.0.0.1`.
- 'gateway.controlUi.dangerouslyDisableDeviceAuth' يعطل التحقق من هوية الجهاز لـ
  واجهة التحكم (token/password فقط). الافتراضي: `خطأ`. زجاج قاطع فقط.

مستندات ذات صلة:

- [واجهة التحكم](/web/control-ui)
- [نظرة عامة](/web)
- [Tailscale](/gateway/tailscale)
- [الوصول عن بُعد](/gateway/remote)

الوكلاء الموثوقون:

- `gateway.trustedProxies`: قائمة عناوين IP الوكيل العكسي التي تنهي TLS أمام البوابة.
- عندما يأتي اتصال من أحد عناوين IP هذه، يستخدم OpenClaw 'x-forwarded-for ' (أو 'x-realal-ip') لتحديد IP العميل لفحوصات الاقتران المحلية و HTTP Aut/محليا.
- قائمة فقط بالوكلاء الذين تسيطرون عليهم بشكل كامل، وتأكد من **الكتابة** الواردة `x-forwarded-for`.

ملاحظات:

- 'openclaw بوابة' ترفض البدء ما لم يتم تعيين 'gateway.mode' إلى 'local' (أو اجتياز علم التجاوز).
- 'gateway.port' يتحكم في المنفذ المتعدد الوحيد المستخدم في WebSocket + HTTP (التحكم في واجهة واجهة المستخدم، روابط ، A2UI).
- نقطة نهاية إكمال الدردشة OpenAI: **معطل بشكل افتراضي**؛ تمكين مع `gateway.http.endpoints.chatCompletions.enabled: true`.
- السوابق: `--port` > `OPENCLAW_GATEWAY_PORT` > `Gateway.port` > الافتراضي `18789`.
- مصادقة البوابة مطلوبة بشكل افتراضي (token/password أو هوية خدمة النطاق السريع). يتطلب ربط عدم الرجوع رمز/كلمة مرور مشتركة.
- يقوم معالج onboarding بإنشاء رمز البوابة بشكل افتراضي (حتى على الحلقة التكرارية).
- 'gateway.remote.token' هو **فقط** لمكالمات CLI البعيدة؛ وهو لا يمكن مصادقة البوابة المحلية. تم تجاهل 'gateway.token'.

المصادقة والجدول السريع:

- 'gateway.auth.mode' يحدد متطلبات مصافحة ('token' أو 'كلمة المرور\`). عند عدم التعيين، يتم افتراض مصادقة الرمز المميز.
- 'gateway.auth.token' يخزن الرمز المميز المشترك للمصادقة الرمزية (المستخدم من قبل CLI على نفس الآلة).
- عندما يتم تعيين \`gateway.auth.mode'، فقط يتم قبول هذه الطريقة (بالإضافة إلى عناوين تاكيس اختيارية).
- يمكن تعيين `gateway.auth.password' هنا، أو عبر `OPENCLAW_GATEWAY_PASSWORD' (موصى به).
- 'gateway.auth.allowTailscale' يسمح لرأس هوية خدمة النطاق
  (tailscale-user-login') بإرضاء المصادقة عند وصول الطلب على loopback
  مع 'x-forwarded-for`، و 'x-forwarded-proto`، و 'x-forwarded-host`. يقوم OpenClaw
    بالتحقق من الهوية عن طريق حل عنوان 'x-forwarded-for' عن طريق
  'حسب المقياس التوكيفي' قبل قبوله. عندما 'true'، لا تحتاج طلبات الخدمة
    الرمز المميز/كلمة المرور؛ قم بتعيين 'false' لطلب بيانات الاعتماد الصريحة. الافتراضي إلى
  `true` عندما 'tailscale.mode = "serve"` و وضع المصادقة ليس `كلمة المرور`.
- `gateway.tailscale.mode: "serve"` يستخدم خدمة النطاق الضوئي (الذيفة فقط، حلقة الظهور).
- `gateway.tailscale.mode: "funnel"` يعرض لوحة المعلومات علناً؛ يتطلب المصادقة.
- إعادة ضبط إعدادات Serve/Funnel عند الإغلاق.

الافتراضي للعميل البعيد (CLI):

- 'gateway.remote.url' يعين عنوان URL الافتراضي للبوابة WebSocket للمكالمات CLI عندما 'gateway.mode = "remote"\`.
- 'gateway.remote.transport' يحدد النقل البعيد macOS ('ssh' الافتراضي، 'direct' للws/wss). عندما 'direct`، 'gateway.remote.url` يجب أن يكون 'ws://` أو 'wss://`. الافتراضي `ws://host` إلى المنفذ `18789`.
- 'gateway.remote.token' يوفر الرمز المميز للمكالمات البعيدة (اتركه غير معين بدون مؤذية).
- 'gateway.remote.password' يوفر كلمة المرور للمكالمات البعيدة (اتركه غير معين بدون خطأ).

سلوك تطبيق macOS:

- OpenClaw.app ساعة '~/.openclaw/openclaw.json' و مفاتيح التبديل حية عندما يتغير 'gateway.mode' أو 'gateway.remote.url'.
- إذا تم إلغاء تعيين \`gateway.mode' ولكن 'gateway.remote.url' ، فإن تطبيق macOS يعالجه على أنه وضع بعيد.
- عند تغيير وضع الاتصال في تطبيق macOS، فإنه يكتب 'gateway.mode' (و 'gateway.remote.url' + 'gateway.remote.transport' في الوضع البعيد) إلى ملف config

```json5
{
  بوابة: {
    وضع "بعيد"،
    بعيد: {
      url: "ws://gateway.tailnet:18789",
      token: "You token",
      كلمة المرور: "كلمة المرور الخاصة بك"،
    },
  },
}
```

مثال النقل المباشر (تطبيق ماك أوس):

```json5
{
  بوابة: {
    وضع "بعيد"،
    بعيد: {
      نقل: "مباشرة"،
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### 'gateway.reload' (إعادة تحميل ساخنة)

وتشاهد البوابة '~/.openclaw.json' (أو 'OPENCLAW_CONFIG_PATH\`) وتطبق التغييرات تلقائيا.

الأساليب:

- 'هجين' (الافتراضي): تطبيق التغييرات الآمنة بشكل ساخن؛ إعادة تشغيل البوابة للتغييرات الحرجة.
- "ساخن": فقط تطبيق التغييرات الآمنة الساخنة؛ سجل عندما يتطلب الأمر إعادة التشغيل.
- `إعادة التشغيل`: إعادة تشغيل البوابة على أي تغيير في الإعداد.
- `إيقاف`: تعطيل إعادة التحميل الساخن.

```json5
{
  بوابة: {
    إعادة التحميل: {
      mode: "هجين",
      debounceMs: 300,
    },
  },
}
```

#### إعادة تحميل المصفوفة الساخنة (الملفات + التأثير)

الملفات المشاهدة:

- `~/.openclaw/openclaw.json` (أو `OPENCLAW_CONFIG_PATH`)

تطبيق ساخن (لا إعادة تشغيل البوابة كاملة):

- `hooks` (webhook auth/path/mappings) + `hooks.gmail` (إعادة تشغيل مشغل البريد الإلكتروني)
- `المتصفح` (إعادة تشغيل خادم التحكم في المتصفح)
- \`cron' (إعادة تشغيل خدمة cron + تحديث concurren)
- \`agents.defaults.heartbeat' (إعادة تشغيل معالج heartbeat)
- `web` (إعادة تشغيل قناة WatsApp)
- `telegram` و`discord` و`signal` و`imessage` (إعادة تشغيل القناة)
- `وكيل`، `نماذج`، `توجيه`، `رسائل`، `دورة`، `Whatsapp`، `تسجيل الدخول`، `المهارات`، `ui`، `تحدث`، `هوية`، `معالج` (القراءات الديناميكية)

يتطلب إعادة تشغيل البوابة بالكامل:

- `gateway` (port/bind/auth/control UI/tailscale)
- `جسر` (الإرث)
- `الاكتشاف`
- `canvasHost`
- `الإضافات`
- أي مسار تكوين غير معروف/غير مدعوم (الافتراضي لإعادة التشغيل من أجل السلامة)

### العزل المتعدد الأشكال

لتشغيل العديد من البوابات على مضيف واحد (لفائض الحاجة أو بوت الإنقاذ)، عزل التكوين لكل مثيل + ضبط واستخدام موانئ فريدة:

- `OPENCLAW_CONFIG_PATH` (تكوين لكل مثال)
- `OPENCLAW_STATE_DIR` (دورات/اعتمادات)
- `agents.defaults.workspace` (الذاكرات)
- 'gateway.port' (فريدة من نوعها في كل حالة)

أعلام الراحة (CLI):

- 'openclaw --dev …' يستخدم '~/.openclaw-dev' + ينقل المنافذ من القاعدة '19001\`
- `openclaw --الملف الشخصي <name> …` يستخدم `~/.openclaw-<name>` (منفذ عبر config/env/flags)

انظر [Gateway runbook](/gateway) لرسم خرائط الموانئ المشتقة (Gateway/browser/canvas).
انظر [البوابات المتعددة](/gateway/multiple-gateways) للحصول على تفاصيل عزل المتصفح/منفذ DP.

مثال:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw Porateway 19001
```

### `hooks` (Gateway webhooks)

تمكين نقطة نهاية HTTP بسيطة على خادم HTTP.

القيم الافتراضية:

- مفعل: 'خطأ\`
- المسار: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
34. {
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

يجب أن تتضمن الطلبات الرمز المميز للربط:

- `إذن: حامله <token>` **أو**
- `x-openclaw-token: <token>`

نقاط النهاية:

- `POST /hooks/wake` → `{ text ووضع؟: "الآن"<unk> "next-heartbeat" }`
- `POST /hooks/agent' → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSecond? }\`
- `POST /hooks/<name>` حل بواسطة `hooks.mappings`

'/hooks/agent' يقوم دائما بنشر ملخص في الجلسة الرئيسية (ويمكن أن يؤدي اختياريا إلى نبض قلبي مباشر عن طريق 'نمط الإيقاف: "الآن").

ملاحظات رسم الخرائط:

- 'match.path' يتطابق مع المسار الفرعي بعد '/hooks' (على سبيل المثال '/hooks/gmail' → 'gmail\`).
- 'match.source' يتطابق مع حقل الحمولة (على سبيل المثال '{ المصدر: ”gmail“ }\`) حتى تتمكن من استخدام مسار عام '/hooks/ingest'.
- قوالب مثل `{{messages[0].subject}}` مقروءة من الحمولة.
- 'تحويل\` يمكن أن يشير إلى وحدة JS/TS التي ترجع إجراء الإرتباط.
- 'تسليم: صحيح' يرسل الرد النهائي إلى قناة؛ افتراضات 'القناة' إلى 'الأخير' (يعود إلى WhatsApp).
- إذا لم يكن هناك مسار تسليم سابق، قم بتعيين 'channel\` + 'to' بشكل صريح (مطلوب لأفرقة Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS).
- 'model' يلغي نظام LLM لتشغيل هذا الرابط ('provider/model' أو الاسم المستعار يجب السماح به إذا تم تعيين 'agents.defaults.models\`).

إعدادات مساعد Gmail (مستخدمة من قبل `openclaw webhooks gmail sep` / `تشغيل`):

```json5
35. {
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Optional: use a cheaper model for Gmail hook processing
      // Falls back to agents.defaults.model.fallbacks, then primary, on auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    },
  },
}
```

تجاوز النموذج لروابط Gmail :

- يحدد \`hooks.gmail.model' نموذجا لاستخدامه في معالجة رابط Gmail (الافتراضي إلى الجلسة الأساسية).
- يقبل طلبات 'provider/model' أو أسماء مستعارة من 'agents.defaults.models\`.
- يعود إلى `agents.defaults.model.fallbacks`، ثم إلى `agents.defaults.model.primary`، على auth/rate-limit/timeouts.
- إذا تم تعيين \`agents.defaults.models'، قم بتضمين نموذج الروابط في قائمة الأماكن.
- عند بدء التشغيل، يحذر إذا لم يكن النموذج المكون في كتالوج النموذج أو قائمة المسموح بها.
- 'hooks.gmail.thinking' يحدد مستوى التفكير الافتراضي لـ Gmail hooks ويتم تجاوزه بـ per-hook 'thinking'.

البوابة التشغيل التلقائي:

- إذا تم تعيين `hooks.enabled=true` و `hooks.gmail.account`، تبدأ البوابة
  `gog gmail watch serve` عند التشغيل وتجديد الساعة.
- تعيين `OPENCLAW_SKIP_GMAIL_WATCHER=1` لتعطيل البداية التلقائية (للتشغيل اليدوي).
- 36. تجنّب تشغيل أمر `gog gmail watch serve` بشكل منفصل إلى جانب الـ Gateway؛ لأنه سيفشل مع الخطأ:
      `listen tcp 127.0.0.1:8788: bind: address already in use`.

ملاحظة: عند تشغيل 'tailscale.mode'، الافتراضي لـ 'serve.path' إلى '/' حتى
Tailscaly يمكن للوكيل '/gmail-pubsub' بشكل صحيح (يجرد بادئة المسار المحدد).
إذا كنت بحاجة إلى الخلفية لاستلام المسار المحدد مسبقا، قم بتعيين
`hooks.gmail.tailscale.target` إلى عنوان URL كامل (واجعل 'serve.path\`).

### 'canvasHost' (LAN/tailnet Canvas ملف خادم + إعادة التحميل المباشر)

تغطى البوابة دليل HTML/CSS/JS عبر HTTP بحيث يمكن ببساطة عقد iOS/Android "canvas.navigate" لها.

الجذر الافتراضي: `~/. pclaw/workspace/canvas`  
منفذ افتراضي: `1879="(تم اختياره لتجنب منفذ CDP الخاص بمتصفح openclaw '18792`)  
يستمع الخادم على **مضيف ربط البوابة** (LAN أو Tailnet) حتى تصل العقد.

الخادم:

- يخدم الملفات تحت `canvashost.root`
- حقن عميل صغير لإعادة التحميل المباشر إلى HTML الخدمة
- يشاهد الدليل ويبث إعادة التحميل عبر نقطة نهاية WebSocket في `/__openclaw__/ws`
- إنشاء تلقائي لبدء "index.html" عندما يكون الدليل فارغاً (لذلك يمكنك رؤية شيء على الفور)
- أيضا يخدم A2UI في `/__openclaw__/a2ui/` ويعلن عن العقد باسم `canvasHostUrl`
  (دائما ما تستخدمه العقد لـ Canvas/A2UI)

تعطيل إعادة التحميل المباشر (ومشاهدة الملفات) إذا كان المجلد كبيرًا أو ضغطت على `EMFILE`:

- التكوين: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    الجذر: "~/.openclaw/workspace/canvas",
    port: 18793,
    LiveReload: true,
  },
}
```

تتطلب التغييرات إلى 'canvasHost.\*' إعادة تشغيل البوابة (إعادة التحميل سيتم إعادة التشغيل).

للتعطيل:

- التكوين: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `جسر` (جسر TCP القديم، تمت إزالته)

لم تعد الإصدارات الحالية تتضمن مستمع جسر TCP ؛ يتم تجاهل مفاتيح الضبط 'bridge.\*'.
تتصل العقد عبر بوابة WebSocket. ويحتفظ بهذا الجزء لأغراض الرجوع إليه تاريخيا.

السلوك القديم:

- ويمكن أن تكشف البوابة عن جسر بسيط للعقد (iOS/Android)، عادة على المنفذ `18790`.

القيم الافتراضية:

- مفعل: `true`
- المنفذ: `18790`
- ربط: `lan` (إرتباطات إلى `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (يمكن الوصول إليها على أي واجهة، بما في ذلك LAN/Wi<unk> Fi و Tailscale)
- 'ذيل الشبكة\`: يرتبط فقط ببروتوكول IP الخاص بالآلة على خط المقياس (موصى به بالنسبة لفيينا لندن)
- `الخلفية`: `127.0.0.1` (محلي فقط)
- `auto`: يفضل الذيل IP إذا كان موجودا، أو 'lan\`

TLS:

- `bridge.tls.enabled`: تمكين TLS للاتصال بالجسر (TLS-فقط عند التمكين).
- `bridge.tls.autoGenerate`: أنشئ قصاصة موقعة ذاتيا عندما لا يكون هناك قصاص/مفتاح (الافتراضي: صحيح).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: مسارات PEM لشهادة الجسر + المفتاح الخاص.
- `bridge.tls.caPath`: مجموعة PEM CA اختيارية (جذور مخصصة أو mTLS في المستقبل).

عندما يتم تمكين TLS ، تعلن البوابة 'bridgeTls=1\` و 'bridgeTlsSha256' في سجلات اكتشاف TXT
حتى يمكن للعقد تثبيت على الشهادة. 37. تستخدم الاتصالات اليدوية أسلوب الثقة عند أول استخدام (trust-on-first-use) إذا لم يتم تخزين بصمة بعد.
القصاصات التي يتم إنشاؤها تلقائياً تتطلب "openssl" على PATH؛ إذا فشل الجيل فلن يبدأ.

```json5
38. {
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Uses ~/.openclaw/bridge/tls/bridge-{cert,key}.pem when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS) وضع البث

التحكم في البث لاكتشاف شبكة mDNS (`_openclaw-gw._tcp`).

- 'أدنى' (الافتراضي): حذف 'cliPath' + 'sshPort' من سجلات TXT
- 'كامل\`: تشمل 'cliPath' + 'sshPort' في سجلات TXT
- "إيقاف": تعطيل بث mDNS بالكامل
- اسم المضيف: الافتراضي إلى 'openclaw' (الإعلانات 'openclaw.local`). تجاوز مع `OPENCLAW_MDNS_HOSTNAME\`.

```json5
{
  اكتشاف: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS<unk> SD)

عند التمكين، تكتب البوابة منطقة DNS-SD unicast لـ `_openclaw-gw._tcp` تحت `~/.openclaw/dns/' باستخدام نطاق الاكتشاف المكون (على سبيل المثال: `openclaw.internal.\`).

لجعل iOS/Android يكتشف عبر الشبكات (فيينا <unk> London)، زوج هذا مع:

- خادم DNS على البوابة المضيفة لخدمة المجال الذي اخترته (CoreDNS موصى به)
- المقياس الخيالي **تقسيم DNS** حتى يقوم العملاء بحل هذا النطاق عبر خادم DNS

مساعد إعداد لمرة واحدة (مضيف):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## متغيرات قالب نموذج الوسائط

ويتسع نطاق العناصر النائبة للنماذج في `tools.media.*.models[].args` و`tools.media.models[].args` (وأي حقول حجة نموذجية مقبلة).

متغير <unk> وصف <unk> <unk> ---------------------- <unk> ------------------------------------------------------------ <unk> ------------- <unk> -------- <unk> ---------- <unk> ------ <unk> ------ <unk> ---- <unk> ------ <unk> ------ <unk> -------- <unk> ------ <unk> ------- <unk> ----- <unk> --- <unk> --- <unk> --- <unk> <unk>{{Body}}<unk> ` جسم الرسالة المرسلة بالكامل <unk> <unk> A`{{RawBody}}`جسم الرسالة المرسلة (لا التاريخ/sender أفضل لتحليل الأوامر <unk> <unk>`{{BodyStripped}}`الجسم مع ذكر مجموعة تم تجريدها (أفضل الافتراضي للوكلاء) <unk> <unk>`{{From}}`<unk> المرسل (. 64 لـ WhatsApp؛ قد تختلف في كل قناة) <unk> <unk>`{{To}}`<unk> Ddestination ID <unk> <unk>`{{MessageSid}}`<unk> ChChanmessage id (عند توفر) <unk> <unk>`{{SessionId}}`<unk> هذه الدورة UUID <unk> <unk>`{{IsNewSession}}<unk> `true"` عند إنشاء جلسة جديدة <unk> <unk> `{{MediaUrl}}<unk> الوسائط المدرجة pseudo-URL (إذا كانت موجودة) <unk> <unk>{{MediaPath}}<unk> ممر الوسائط المحلية (إذا تم تنزيلها) <unk> <unk> `{{MediaType}}<unk> Media نوع (image/audio/document/…)                                             39. |
\| `{{Transcript}}`   | تفريغ الصوت (عند التمكين)                                                 |
\| `{{Prompt}}`       | موجه الوسائط المحلول لإدخالات CLI                                           |
\| `{{MaxChars}}`     | الحد الأقصى المحلول لأحرف الإخراج لإدخالات CLI                               |
\| `{{ChatType}}`     | "direct" أو "group"                                                         |
\| `{{GroupSubject}}` | موضوع المجموعة (بأفضل جهد)                                                     |
\| `{{GroupMembers}}` | معاينة أعضاء المجموعة (بأفضل جهد)                                             |
\| `{{SenderName}}`   | اسم عرض المرسل (بأفضل جهد)                                               |
\| `{{SenderE164}}`   | رقم هاتف المرسل (بأفضل جهد)                                               |
\| `{{Provider}}`     | تلميح المزوّد (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## جدولة كرون (جدول)

كرون هو جدولة تملكها مجموعة غاتواي للإيقاعات والوظائف المقررة. انظر [Cron jobs](/automation/cron-jobs) للحصول على لمحة عامة عن الميزة وأمثلة CLI.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_التالي: [تشغيل الوكيل](/concepts/agent)_ 🦞
