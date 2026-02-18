---
title: "WhatsApp"
---

# WhatsApp (قناة الويب)

الحالة: جاهز للإنتاج عبر WhatsApp Web (`Baileys`). يمتلك الـ Gateway الجلسة (الجلسات) المرتبطة.

<CardGroup cols={3}>
  <Card title="الإقران" icon="link" href="/channels/pairing">
    السياسة الافتراضية للرسائل الخاصة من مرسلين غير معروفين هي الإقران.
  
</Card>
  <Card title="استكشاف أخطاء القناة" icon="wrench" href="/channels/troubleshooting">
    تشخيصات متعددة القنوات وأدلة الإصلاح.
  
</Card>
  <Card title="تهيئة Gateway" icon="settings" href="/gateway/configuration">
    أنماط وأمثلة تهيئة القناة الكاملة.
  
</Card>
</CardGroup>

## الإعداد السريع

<Steps>
  <Step title="تهيئة سياسة الوصول إلى WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="ربط WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    لحساب محدد:

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="تشغيل Gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="الموافقة على أول طلب إقران (عند استخدام وضع الإقران)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    تنتهي طلبات الإقران بعد ساعة واحدة. الحد الأقصى للطلبات المعلّقة هو 3 لكل قناة.

  
</Step>
</Steps>

<Note>
توصي OpenClaw بتشغيل WhatsApp على رقم منفصل كلما أمكن. (بيانات القناة وتدفق الإعداد مُحسّنان لهذا النمط، لكن إعدادات الرقم الشخصي مدعومة أيضًا.)
</Note>

## أنماط النشر

<AccordionGroup>
  <Accordion title="رقم مخصص (موصى به)">
    هذا هو النمط التشغيلي الأنظف:

    - هوية WhatsApp منفصلة لـ OpenClaw
    - حدود أوضح لقوائم سماح DM والتوجيه
    - احتمال أقل لالتباس الدردشة مع الذات

    نمط سياسة أدنى:

    ```json5
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="الرقم الشخصي (حل احتياطي)">
    يدعم الإعداد وضع الرقم الشخصي ويكتب إعدادًا أساسيًا مناسبًا للدردشة مع الذات:

    - `dmPolicy: "allowlist"`
    - تتضمن `allowFrom` رقمك الشخصي
    - `selfChatMode: true`

    في وقت التشغيل، تعتمد حمايات الدردشة مع الذات على الرقم الذاتي المرتبط و `allowFrom`.

  
</Accordion>

  <Accordion title="نطاق قناة WhatsApp Web فقط">
    قناة المراسلة تعتمد على WhatsApp Web (`Baileys`) في بنية قنوات OpenClaw الحالية.

    لا توجد قناة Twilio WhatsApp منفصلة ضمن سجل القنوات المدمج.

  
</Accordion>
</AccordionGroup>

## نموذج وقت التشغيل

- يمتلك الـ Gateway مقبس WhatsApp وحلقة إعادة الاتصال.
- يتطلب الإرسال الصادر مستمع WhatsApp نشطًا للحساب المستهدف.
- يتم تجاهل محادثات الحالة والبث (`@status`, `@broadcast`).
- تستخدم الدردشات المباشرة قواعد جلسة DM (`session.dmScope`؛ الافتراضي `main` يدمج DMs في الجلسة الرئيسية للوكيل).
- جلسات المجموعات معزولة (`agent:<agentId>:whatsapp:group:<jid>`).

## التحكم في الوصول والتفعيل

<Tabs>
  <Tab title="سياسة DM">
    يتحكم `channels.whatsapp.dmPolicy` في الوصول إلى الدردشة المباشرة:

    - `pairing` (افتراضي)
    - `allowlist`
    - `open` (يتطلب أن تتضمن `allowFrom` القيمة `"*"`)
    - `disabled`

    تقبل `allowFrom` أرقام بنمط E.164 (يتم تطبيعها داخليًا).

    تجاوز متعدد الحسابات: `channels.whatsapp.accounts.<id>.dmPolicy` (و `allowFrom`) يتقدمان على الإعدادات الافتراضية على مستوى القناة لذلك الحساب.

    تفاصيل السلوك في وقت التشغيل:

    - يتم حفظ عمليات الإقران في مخزن السماح للقناة ودمجها مع `allowFrom` المُهيّأ
    - إذا لم يتم تكوين قائمة سماح، يُسمح بالرقم الذاتي المرتبط افتراضيًا
    - لا يتم أبدًا إقران رسائل DM الصادرة `fromMe` تلقائيًا

  
</Tab>

  <Tab title="سياسة المجموعات + قوائم السماح">
    وصول المجموعات له طبقتان:

    1. **قائمة سماح عضوية المجموعة** (`channels.whatsapp.groups`)
       - إذا تم حذف `groups` فجميع المجموعات مؤهلة
       - إذا وُجدت `groups` فهي تعمل كقائمة سماح للمجموعات (مسموح باستخدام `"*"`)

    2. **سياسة مرسل المجموعة** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: يتم تجاوز قائمة سماح المرسلين
       - `allowlist`: يجب أن يطابق المرسل `groupAllowFrom` (أو `*`)
       - `disabled`: حظر جميع الرسائل الواردة من المجموعات

    احتياطي قائمة سماح المرسل:

    - إذا لم يتم تعيين `groupAllowFrom`، يعود وقت التشغيل إلى `allowFrom` عند توفره

    ملاحظة: إذا لم يوجد أي قسم `channels.whatsapp` إطلاقًا، فإن احتياطي سياسة المجموعات يكون فعليًا `open`.

  
</Tab>

  <Tab title="الإشارات + /activation">
    تتطلب ردود المجموعات إشارة (mention) افتراضيًا.

    يشمل اكتشاف الإشارة:

    - إشارات WhatsApp الصريحة لهوية البوت
    - أنماط regex المُهيّأة للإشارة (`agents.list[].groupChat.mentionPatterns`، والاحتياطي `messages.groupChat.mentionPatterns`)
    - اكتشاف الرد على البوت ضمنيًا (مرسل الرد يطابق هوية البوت)

    أمر تفعيل على مستوى الجلسة:

    - `/activation mention`
    - `/activation always`

    يقوم `activation` بتحديث حالة الجلسة (وليس التهيئة العامة). وهو مقيّد بالمالك.

  
</Tab>
</Tabs>

## سلوك الرقم الشخصي والدردشة مع الذات

عندما يكون الرقم الذاتي المرتبط موجودًا أيضًا ضمن `allowFrom`، يتم تفعيل حمايات الدردشة مع الذات:

- تخطي إيصالات القراءة لدورات الدردشة مع الذات
- تجاهل سلوك التشغيل التلقائي لمعرّفات الإشارة الذي قد يؤدي إلى تنبيه نفسك
- إذا لم يتم تعيين `messages.responsePrefix`، فإن ردود الدردشة مع الذات تستخدم افتراضيًا `[{identity.name}]` أو `[openclaw]`

## توحيد الرسائل والسياق

<AccordionGroup>
  <Accordion title="الغلاف الوارد + سياق الرد">
    يتم تغليف رسائل WhatsApp الواردة ضمن الغلاف الوارد المشترك.

    عند وجود رد مقتبس، يُلحَق السياق بهذا الشكل:

    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```

    كما يتم ملء حقول بيانات الرد الوصفية عند توفرها (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, مرسل JID/E.164).

  
</Accordion>

  <Accordion title="عناصر الوسائط النائبة واستخراج الموقع/جهة الاتصال">
    يتم توحيد الرسائل الواردة التي تحتوي على وسائط فقط باستخدام عناصر نائبة مثل:

    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`

    يتم تحويل بيانات الموقع وجهات الاتصال إلى سياق نصي قبل التوجيه.

  
</Accordion>

  <Accordion title="حقن سجل المجموعات المعلّق">
    في المجموعات، يمكن تخزين الرسائل غير المعالجة مؤقتًا وحقنها كسياق عند تشغيل البوت أخيرًا.

    - الحد الافتراضي: `50`
    - التهيئة: `channels.whatsapp.historyLimit`
    - الاحتياطي: `messages.groupChat.historyLimit`
    - `0` يعطّل الميزة

    علامات الحقن:

    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`

  
</Accordion>

  <Accordion title="إيصالات القراءة">
    إيصالات القراءة مفعّلة افتراضيًا لرسائل WhatsApp الواردة المقبولة.

    تعطيل عام:

    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```

    تجاوز لكل حساب:

    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```

    تتجاوز دورات الدردشة مع الذات إيصالات القراءة حتى عند تفعيلها عالميًا.

  
</Accordion>
</AccordionGroup>

## التسليم، التجزئة، والوسائط

<AccordionGroup>
  <Accordion title="تجزئة النص">
    - الحد الافتراضي للتجزئة: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - وضع `newline` يفضّل حدود الفقرات (الأسطر الفارغة)، ثم يعود إلى تجزئة آمنة حسب الطول
  
</Accordion>

  <Accordion title="سلوك الوسائط الصادرة">
    - يدعم الصور والفيديو والصوت (ملاحظة صوتية PTT) والمستندات
    - يتم إعادة كتابة `audio/ogg` إلى `audio/ogg; codecs=opus` للتوافق مع الملاحظات الصوتية
    - يتم دعم تشغيل GIF المتحرك عبر `gifPlayback: true` عند إرسال الفيديو
    - يتم تطبيق العنوان (Caption) على أول عنصر وسائط عند إرسال رد متعدد الوسائط
    - يمكن أن يكون مصدر الوسائط HTTP(S) أو `file://` أو مسارات محلية
  
</Accordion>

  <Accordion title="حدود حجم الوسائط وسلوك الاحتياطي">
    - حد حفظ الوسائط الواردة: `channels.whatsapp.mediaMaxMb` (الافتراضي `50`)
    - حد الوسائط الصادرة للردود التلقائية: `agents.defaults.mediaMaxMb` (الافتراضي `5MB`)
    - يتم تحسين الصور تلقائيًا (تغيير الحجم/الجودة) لتناسب الحدود
    - عند فشل إرسال الوسائط، يتم إرسال تحذير نصي بدلاً من إسقاط الرد بصمت
  
</Accordion>
</AccordionGroup>

## تفاعلات الإقرار

يدعم WhatsApp تفاعلات إقرار فورية عند استلام الرسائل الواردة عبر `channels.whatsapp.ackReaction`.

```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

ملاحظات السلوك:

- يتم الإرسال فور قبول الرسالة الواردة (قبل الرد)
- يتم تسجيل الإخفاقات دون منع تسليم الرد الطبيعي
- وضع المجموعة `mentions` يتفاعل عند الدورات التي يتم تشغيلها عبر الإشارة؛ تفعيل المجموعة `always` يعمل كتجاوز لهذا الفحص
- يستخدم WhatsApp الإعداد `channels.whatsapp.ackReaction` (ولا يتم استخدام `messages.ackReaction` القديم هنا)

## تعدد الحسابات وبيانات الاعتماد

<AccordionGroup>
  <Accordion title="اختيار الحساب والافتراضيات">
    - تأتي معرّفات الحسابات من `channels.whatsapp.accounts`
    - اختيار الحساب الافتراضي: `default` إن وُجد، وإلا أول معرّف حساب مُهيّأ (مرتّب)
    - يتم تطبيع معرّفات الحسابات داخليًا لأغراض البحث
  
</Accordion>

  <Accordion title="مسارات بيانات الاعتماد والتوافق القديم">
    - مسار المصادقة الحالي: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - ملف احتياطي: `creds.json.bak`
    - يتم التعرف على المصادقة الافتراضية القديمة في `~/.openclaw/credentials/` وترحيلها عند الحاجة
  
</Accordion>

  <Accordion title="سلوك تسجيل الخروج">
    `openclaw channels logout --channel whatsapp [--account <id>]` يمسح حالة مصادقة WhatsApp لذلك الحساب.

    في أدلة المصادقة القديمة، يتم الاحتفاظ بـ `oauth.json` بينما تُحذف ملفات Baileys.

  
</Accordion>
</AccordionGroup>

## الأدوات، الإجراءات، وكتابة التهيئة

- يدعم الوكيل أداة تفاعل WhatsApp (`react`).
- بوابات الإجراءات:
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- كتابة التهيئة التي تبدأها القناة مفعّلة افتراضيًا (يمكن التعطيل عبر `channels.whatsapp.configWrites=false`).

## استكشاف الأخطاء وإصلاحها

<AccordionGroup>
  <Accordion title="غير مرتبط (مطلوب QR)">
    العَرَض: تُظهر حالة القناة أنها غير مرتبطة.

    الحل:

    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```

  
</Accordion>

  <Accordion title="مرتبط لكن غير متصل / حلقة إعادة اتصال">
    العَرَض: حساب مرتبط مع انقطاعات متكررة أو محاولات إعادة اتصال.

    الحل:

    ```bash
    openclaw doctor
    openclaw logs --follow
    ```

    عند الحاجة، أعد الربط باستخدام `channels login`.

  
</Accordion>

  <Accordion title="لا يوجد مستمع نشط عند الإرسال">
    تفشل عمليات الإرسال الصادر فورًا عند عدم وجود مستمع Gateway نشط للحساب المستهدف.

    تأكد من أن الـ Gateway يعمل وأن الحساب مرتبط.

  
</Accordion>

  <Accordion title="يتم تجاهل رسائل المجموعات بشكل غير متوقع">
    تحقق بالترتيب التالي:

    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - إدخالات قائمة سماح `groups`
    - تقييد الإشارة (`requireMention` + أنماط الإشارة)

  
</Accordion>

  <Accordion title="تحذير بيئة Bun">
    يجب تشغيل بيئة WhatsApp في الـ Gateway باستخدام Node. يتم وضع علامة على Bun كغير متوافق للتشغيل المستقر لقنوات WhatsApp/Telegram.
  
</Accordion>
</AccordionGroup>

## مراجع التهيئة

المرجع الأساسي:

- [مرجع التهيئة - WhatsApp](/gateway/configuration-reference#whatsapp)

حقول WhatsApp المهمة:

- الوصول: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- التسليم: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- تعدد الحسابات: `accounts.<id>.enabled`, `accounts.<id>.authDir`, تجاوزات على مستوى الحساب
- العمليات: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- سلوك الجلسة: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## مرتبط

- [الإقران](/channels/pairing)
- [توجيه القنوات](/channels/channel-routing)
- [استكشاف الأخطاء وإصلاحها](/channels/troubleshooting)

