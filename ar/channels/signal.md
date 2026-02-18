---
title: "Signal"
---

# Signal (signal-cli)

الحالة: تكامل خارجي عبر CLI. يتواصل Gateway مع `signal-cli` عبر HTTP JSON-RPC + SSE.

## الإعداد السريع (للمبتدئين)

1. استخدم **رقم Signal منفصل** للبوت (موصى به).
2. ثبّت `signal-cli` (يتطلب Java).
3. اربط جهاز البوت وابدأ الخدمة:
   - `signal-cli link -n "OpenClaw"`
4. هيّئ OpenClaw وابدأ Gateway.

التهيئة الدنيا:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## ما هو

- قناة Signal عبر `signal-cli` (وليست libsignal مدمجة).
- توجيه حتمي: تعود الردود دائمًا إلى Signal.
- تشارك الرسائل الخاصة (DMs) جلسة الوكيل الرئيسية؛ بينما تكون المجموعات معزولة (`agent:<agentId>:signal:group:<groupId>`).

## كتابات التهيئة

افتراضيًا، يُسمح لـ Signal بكتابة تحديثات التهيئة المُحفَّزة بواسطة `/config set|unset` (يتطلب `commands.config: true`).

للتعطيل:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## نموذج الأرقام (مهم)

- يتصل Gateway **بجهاز Signal** (حساب `signal-cli`).
- إذا شغّلت البوت على **حساب Signal الشخصي**، فسيتجاهل رسائلك أنت (حماية من الحلقة).
- من أجل "أنا أقوم بنص البوت و يجيب عليه"، استخدم **رقم بوت منفصل**.

## الإعداد (المسار السريع)

1. ثبّت `signal-cli` (يتطلب Java).
2. اربط حساب البوت:
   - `signal-cli link -n "OpenClaw"` ثم امسح رمز QR في Signal.
3. هيّئ Signal وابدأ Gateway.

مثال:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

دعم تعدد الحسابات: استخدم `channels.signal.accounts` مع تهيئة لكل حساب و`name` اختياري. راجع [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) للنمط المشترك.

## وضع الخدمة الخارجية (httpUrl)

إذا رغبت في إدارة `signal-cli` بنفسك (بدء JVM البارد البطيء، تهيئة الحاويات، أو وحدات CPU مشتركة)، شغّل الخدمة بشكل منفصل ووجّه OpenClaw إليها:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

يتجاوز هذا التشغيل التلقائي والانتظار عند البدء داخل OpenClaw. لبدء بطيء عند التشغيل التلقائي، اضبط `channels.signal.startupTimeoutMs`.

## التحكم في الوصول (الرسائل الخاصة + المجموعات)

DMs:

- الافتراضي: `channels.signal.dmPolicy = "pairing"`.
- يتلقى المُرسِلون غير المعروفين رمز إقران؛ وتُتجاهل الرسائل حتى الموافقة (تنتهي الرموز بعد ساعة).
- الموافقة عبر:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- الإقران هو تبادل الرموز الافتراضي لرسائل Signal الخاصة. التفاصيل: [Pairing](/channels/pairing)
- تُخزَّن المُرسِلات ذات UUID فقط (من `sourceUuid`) كـ `uuid:<id>` في `channels.signal.allowFrom`.

المجموعات:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- يتحكم `channels.signal.groupAllowFrom` بمن يمكنه التحفيز في المجموعات عند تعيين `allowlist`.

## كيف يعمل (السلوك)

- يعمل `signal-cli` كخدمة؛ ويقرأ Gateway الأحداث عبر SSE.
- تُطبَّع الرسائل الواردة إلى غلاف القناة المشترك.
- تُوجَّه الردود دائمًا إلى الرقم أو المجموعة نفسها.

## الوسائط + الحدود

- يُجزَّأ النص الصادر إلى `channels.signal.textChunkLimit` (الافتراضي 4000).
- تجزئة اختيارية حسب الأسطر الجديدة: اضبط `channels.signal.chunkMode="newline"` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل التجزئة حسب الطول.
- دعم المرفقات (base64 تُجلب من `signal-cli`).
- حد الوسائط الافتراضي: `channels.signal.mediaMaxMb` (الافتراضي 8).
- استخدم `channels.signal.ignoreAttachments` لتجاوز تنزيل الوسائط.
- يستخدم سياق سجلّ المجموعات `channels.signal.historyLimit` (أو `channels.signal.accounts.*.historyLimit`) مع الرجوع إلى `messages.groupChat.historyLimit`. عطّل ذلك بضبط `0` (الافتراضي 50).

## مؤشرات الكتابة + إيصالات القراءة

- **مؤشرات الكتابة**: يرسل OpenClaw إشارات الكتابة عبر `signal-cli sendTyping` ويُحدِّثها أثناء تنفيذ الرد.
- **إيصالات القراءة**: عند كون `channels.signal.sendReadReceipts` true، يمرّر OpenClaw إيصالات القراءة للرسائل الخاصة المسموح بها.
- لا يوفّر signal-cli إيصالات قراءة للمجموعات.

## التفاعلات (أداة الرسائل)

- استخدم `message action=react` مع `channel=signal`.
- الأهداف: مُرسِل E.164 أو UUID (استخدم `uuid:<id>` من مخرجات الإقران؛ يعمل UUID المجرد أيضًا).
- `messageId` هو طابع Signal الزمني للرسالة التي تتفاعل معها.
- تتطلب تفاعلات المجموعات `targetAuthor` أو `targetAuthorUuid`.

أمثلة:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

التهيئة:

- `channels.signal.actions.reactions`: تمكين/تعطيل إجراءات التفاعل (الافتراضي true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` يعطّل تفاعلات الوكيل (ستفشل أداة الرسائل `react`).
  - `minimal`/`extensive` يفعّل تفاعلات الوكيل ويضبط مستوى الإرشاد.
- تجاوزات لكل حساب: `channels.signal.accounts.<id>.actions.reactions`، `channels.signal.accounts.<id>.reactionLevel`.

## أهداف التسليم (CLI/cron)

- الرسائل الخاصة: `signal:+15551234567` (أو E.164 عادي).
- رسائل UUID الخاصة: `uuid:<id>` (أو UUID مجرد).
- المجموعات: `signal:group:<groupId>`.
- أسماء المستخدمين: `username:<name>` (إن كانت مدعومة بحساب Signal لديك).

## استكشاف الأخطاء وإصلاحها

شغّل هذا التسلسل أولًا:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

ثم قم بتأكيد حالة إقران DM إذا لزم الأمر:

```bash
openclaw pairing list signal
```

إخفاقات شائعة:

- الخدمة قابلة للوصول لكن لا توجد ردود: تحقّق من إعدادات الحساب/الخدمة (`httpUrl`، `account`) ووضع الاستقبال.
- تجاهل الرسائل الخاصة: المُرسِل بانتظار الموافقة على الإقران.
- تجاهل رسائل المجموعات: قيود مُرسِل/ذكر المجموعة تمنع التسليم.

لمسار الفرز: [/channels/troubleshooting](/channels/troubleshooting).

## مرجع التهيئة (Signal)

التهيئة الكاملة: [Configuration](/gateway/configuration)

خيارات الموفّر:

- `channels.signal.enabled`: تمكين/تعطيل بدء القناة.
- `channels.signal.account`: رقم E.164 لحساب البوت.
- `channels.signal.cliPath`: المسار إلى `signal-cli`.
- `channels.signal.httpUrl`: عنوان URL كامل للخدمة (يتجاوز المضيف/المنفذ).
- `channels.signal.httpHost`، `channels.signal.httpPort`: ربط الخدمة (الافتراضي 127.0.0.1:8080).
- `channels.signal.autoStart`: تشغيل الخدمة تلقائيًا (الافتراضي true إذا لم يُضبط `httpUrl`).
- `channels.signal.startupTimeoutMs`: مهلة انتظار بدء التشغيل بالمللي ثانية (حد أقصى 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: تجاوز تنزيل المرفقات.
- `channels.signal.ignoreStories`: تجاهل القصص من الخدمة.
- `channels.signal.sendReadReceipts`: تمرير إيصالات القراءة.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: الإقران).
- `channels.signal.allowFrom`: قائمة سماح للرسائل الخاصة (E.164 أو `uuid:<id>`). يتطلب `open` وجود `"*"`. لا يدعم Signal أسماء المستخدمين؛ استخدم معرفات الهاتف/UUID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (الافتراضي: قائمة السماح).
- `channels.signal.groupAllowFrom`: قائمة سماح مُرسِلي المجموعات.
- `channels.signal.historyLimit`: الحد الأقصى لرسائل المجموعات المضمَّنة كسياق (0 يعطّل).
- `channels.signal.dmHistoryLimit`: حد سجلّ الرسائل الخاصة بعدد أدوار المستخدم. تجاوزات لكل مستخدم: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: حجم تجزئة الإخراج (أحرف).
- `channels.signal.chunkMode`: `length` (الافتراضي) أو `newline` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل التجزئة حسب الطول.
- `channels.signal.mediaMaxMb`: حد الوسائط الواردة/الصادرة (ميغابايت).

خيارات عامة ذات صلة:

- `agents.list[].groupChat.mentionPatterns` (لا يدعم Signal الإشارات الأصلية).
- `messages.groupChat.mentionPatterns` (الاحتياطي العام).
- `messages.responsePrefix`.


