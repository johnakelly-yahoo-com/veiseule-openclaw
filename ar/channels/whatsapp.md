---
summary: "دعم قناة WhatsApp، عناصر التحكم في الوصول، سلوك التسليم، والعمليات"
read_when:
  - عند العمل على سلوك قناة WhatsApp/الويب أو توجيه صندوق الوارد
title: "WhatsApp"
---

# WhatsApp (قناة الويب)

الحالة: WhatsApp Web عبر Baileys فقط. يمتلك Gateway الجلسة (الجلسات).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    سياسة الرسائل الخاصة (DM) الافتراضية هي الاقتران للمرسلين غير المعروفين.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    تشخيصات متعددة القنوات وأدلة الإصلاح.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">    أنماط وأمثلة إعداد القنوات الكاملة.
  
</Card>
</CardGroup>

## خريطة سريعة للتهيئة

<Steps>
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    لحساب محدد:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
وافق عبر: `openclaw pairing approve whatsapp <code>` (اعرض القائمة بـ `openclaw pairing list whatsapp`).
```

    ```
    تنتهي الرموز بعد ساعة؛ وتُحدّد الطلبات المعلّقة بثلاثة لكل قناة.
    ```

  
</Step>
</Steps>

<Note>
ممتاز للإبقاء على WhatsApp الشخصي منفصلًا — ثبّت WhatsApp Business وسجّل رقم OpenClaw هناك. (يتم تحسين بيانات تعريف القناة وتدفق الإعداد الأولي لهذا النمط، لكن يتم أيضًا دعم إعدادات الأرقام الشخصية.)
</Note>

## أنماط النشر

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    هذا هو نمط التشغيل الأكثر نظافة:

    ```
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

  <Accordion title="Personal-number fallback">
    يدعم الإعداد الأولي وضع الرقم الشخصي ويكتب إعدادًا أساسيًا مناسبًا للمحادثة الذاتية:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    تعتمد قناة منصة المراسلة على WhatsApp Web (``Baileys``) في البنية الحالية لقنوات OpenClaw.

    ```
    لا توجد قناة مراسلة WhatsApp منفصلة عبر Twilio ضمن سجل قنوات الدردشة المدمج.
    ```

  
</Accordion>
</AccordionGroup>

## نموذج التشغيل

- يتولى Gateway إدارة مقبس WhatsApp وحلقة إعادة الاتصال.
- تتطلب عمليات الإرسال الصادرة وجود مستمع WhatsApp نشط للحساب المستهدف.
- تُتجاهل محادثات الحالة/البث.
- تستخدم المحادثات المباشرة قواعد جلسات DM (`session.dmScope`؛ القيمة الافتراضية `main` تدمج الرسائل الخاصة في الجلسة الرئيسية للوكيل).
- جلسات المجموعات معزولة (`agent:<agentId>:whatsapp:group:<jid>`).

## التحكم في الوصول والتفعيل

<Tabs>
  <Tab title="DM policy">**سياسة الرسائل الخاصة**: يتحكم `channels.whatsapp.dmPolicy` في الوصول إلى الدردشة المباشرة (الافتراضي: `pairing`).

    ```
    
    **لماذا تطلبون رقم هاتفي في المعالج؟**  
    يستخدمه المعالج لضبط **قائمة السماح/المالك** بحيث تُسمح رسائل DM الخاصة بك. لا يُستخدم للإرسال التلقائي. إذا شغّلت على رقم WhatsApp الشخصي، استخدم الرقم نفسه وفعّل `channels.whatsapp.selfChatMode`.
    
    ## توحيد الرسائل (ما يراه النموذج)
    
    - `Body` هو نص الرسالة الحالية مع الغلاف.
    
    - يُلحَق سياق الرد المقتبس **دائمًا**:
    
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    يتكون الوصول إلى المجموعات من طبقتين:

    ```
    `channels.whatsapp.groups` (قائمة سماح المجموعات + افتراضات تقييد الإشارة؛ استخدم `"*"` للسماح للجميع)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    تتطلب الردود في المجموعات الإشارة (mention) بشكل افتراضي.

    ```
    يشمل اكتشاف الإشارة:
    
    - إشارات WhatsApp الصريحة إلى هوية البوت
    - أنماط regex للإشارة المكوّنة (`agents.list[].groupChat.mentionPatterns`، والبديل `messages.groupChat.mentionPatterns`)
    - اكتشاف الرد الضمني على البوت (مطابقة مُرسل الرد مع هوية البوت)
    
    أمر التفعيل على مستوى الجلسة:
    
    - `/activation mention`
    - `/activation always`
    
    يقوم `activation` بتحديث حالة الجلسة (وليس الإعدادات العامة). وهو مقيّد بالمالك.
    ```

  
</Tab>
</Tabs>

## سلوك الرقم الشخصي والمحادثة الذاتية

عندما يكون رقمك المرتبط موجودًا أيضًا في `allowFrom`، يتم تفعيل وسائل الحماية الخاصة بالمحادثة الذاتية في WhatsApp:

- يتجاوز وضع الدردشة مع الذات إيصالات القراءة دائمًا.
- تجاهل سلوك التفعيل التلقائي عبر mention-JID الذي قد يؤدي إلى إرسال تنبيه لنفسك
- تكون بادئة ردود الدردشة مع الذات افتراضيًا `[{identity.name}]` عند الضبط (وإلا `[openclaw]`)
  إذا لم يكن `messages.responsePrefix` مضبوطًا.

## تطبيع الرسائل والسياق

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    يتم تغليف رسائل WhatsApp الواردة ضمن الغلاف المشترك للوارد.

    ````
    إذا وُجد رد مقتبس، يُضاف السياق بالشكل التالي:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    كما يتم تعبئة حقول بيانات الرد الوصفية عند توفرها (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    يتم تطبيع الرسائل الواردة التي تحتوي على وسائط فقط باستخدام عناصر نائبة مثل:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    يتم تحويل بيانات الموقع وجهات الاتصال إلى سياق نصي قبل التوجيه.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    في المجموعات، يمكن تخزين الرسائل غير المعالجة مؤقتًا وحقنها كسياق عند تفعيل البوت أخيرًا.

    ```
    - الحد الافتراضي: `50`
    - الإعداد: `channels.whatsapp.historyLimit`
    - البديل: `messages.groupChat.historyLimit`
    - `0` للتعطيل
    
    علامات الحقن:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">افتراضيًا، يعلّم الـ Gateway رسائل WhatsApp الواردة كمقروءة (علامتا الصح الزرقاء) بمجرد قبولها.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## التسليم، التقسيم، والوسائط

<AccordionGroup>
  <Accordion title="Text chunking">تجزئة أسطر جديدة اختيارية: اضبط `channels.whatsapp.chunkMode="newline"` للتقسيم عند الأسطر الفارغة (حدود الفقرات) قبل تجزئة الطول.
</Accordion>

  <Accordion title="Outbound media behavior">
    - يدعم حمولات الصور والفيديو والصوت (ملاحظة صوتية PTT) والمستندات
    - يتم إعادة كتابة `audio/ogg` إلى `audio/ogg; codecs=opus` لضمان توافق الملاحظات الصوتية
    - يتم دعم تشغيل صور GIF المتحركة عبر `gifPlayback: true` عند إرسال الفيديو
    - يتم تطبيق التسميات التوضيحية على أول عنصر وسائط عند إرسال رد متعدد الوسائط
    - يمكن أن يكون مصدر الوسائط HTTP(S) أو `file://` أو مسارات محلية
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - الحد الأقصى لحفظ الوسائط الواردة: `channels.whatsapp.mediaMaxMb` (الافتراضي `50`)
    - الحد الأقصى للوسائط الصادرة في الردود التلقائية: `agents.defaults.mediaMaxMb` (الافتراضي `5MB`)
    - يتم تحسين الصور تلقائيًا (تغيير الحجم/الجودة) لتناسب الحدود
    - عند فشل إرسال الوسائط، يتم إرسال تحذير نصي بديل للعنصر الأول بدلًا من إسقاط الرد بصمت
  
</Accordion>
</AccordionGroup>

## تفاعلات الإقرار بالاستلام

`channels.whatsapp.ackReaction` (تفاعل تلقائي عند استلام الرسالة: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

السلوك:

- تُرسل فور قبول الرسالة الواردة (قبل الرد)
- يتم تسجيل حالات الفشل لكنها لا تمنع تسليم الرد بشكل طبيعي
- في وضع المجموعات `mentions` يتم التفاعل عند التفعيل عبر الإشارة؛ أما تفعيل المجموعة `always` فيعمل كتجاوز لهذا التحقق
- يتجاهل WhatsApp `messages.ackReaction`؛ استخدم `channels.whatsapp.ackReaction` بدلًا منه.

## تعدد الحسابات وبيانات الاعتماد

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - معرّفات الحسابات تأتي من `channels.whatsapp.accounts`
    - اختيار الحساب الافتراضي: `default` إذا كان موجودًا، وإلا أول معرّف حساب مُكوَّن (بعد الفرز)
    - تتم مُطابقة معرّفات الحسابات داخليًا بعد توحيد تنسيقها
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">هيّئ WhatsApp في `~/.openclaw/openclaw.json`.<accountId>تُخزَّن بيانات الاعتماد في `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
</Accordion>

  <Accordion title="Logout behavior">تسجيل دخول متعدد الحسابات: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>]` يمسح حالة مصادقة WhatsApp لذلك الحساب.

    ```
    في أدلة المصادقة القديمة، يتم الاحتفاظ بملف `oauth.json` بينما تتم إزالة ملفات مصادقة Baileys.
    ```

  
</Accordion>
</AccordionGroup>

## الأدوات، والإجراءات، وعمليات كتابة الإعدادات

- يدعم وكيل النظام أداة إجراء التفاعل في WhatsApp (`react`).
- بوابات الإجراءات:
  - `channels.whatsapp.actions.reactions` (تقييد تفاعلات أداة WhatsApp).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## استكشاف الأخطاء وإصلاحها (سريع)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">العَرَض: يُظهر `channels status` القيمة `linked: false` أو تحذير «Not linked».

    ```
    تسجيل الخروج: `openclaw channels logout` (أو `--account <id>`) يحذف حالة مصادقة WhatsApp (مع الاحتفاظ بالمشترك `oauth.json`).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    العرض: حساب مرتبط مع تكرار حالات قطع الاتصال أو محاولات إعادة الاتصال.
  

    ```
    الحل: `openclaw doctor` (أو أعد تشغيل الـ Gateway). إذا استمر، أعد الربط عبر `channels login` وتحقق من `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">تفشل عمليات الإرسال الصادرة فورًا عند عدم وجود مستمع بوابة نشط للحساب المستهدف.

    ```
    تأكد من أن البوابة قيد التشغيل وأن الحساب مرتبط.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    تحقق بالترتيب التالي:
  

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - إدخالات قائمة السماح `groups`
    - تقييد الإشارات (`requireMention` + أنماط الإشارة)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    يجب أن يستخدم وقت تشغيل بوابة WhatsApp بيئة Node. WhatsApp (Baileys) وTelegram غير موثوقين على Bun. شغّل الـ Gateway باستخدام **Node**.
  
</Accordion>
</AccordionGroup>

## مراجع إعدادات التكوين

المرجع الأساسي:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

حقول WhatsApp عالية الأهمية:

- الوصول: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- التسليم: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- تعدد الحسابات: `accounts.<id>.enabled`, `accounts.<id>.authDir`، تجاوزات على مستوى الحساب
- العمليات: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- سلوك الجلسة: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>`messages.groupChat.historyLimit\`

## ذو صلة

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

