---
summary: "حالة دعم بوت Telegram، والإمكانات، والتهيئة"
read_when:
  - العمل على ميزات Telegram أو webhooks
title: "Telegram"
---

# Telegram (Bot API)

الحالة: جاهز للإنتاج للمحادثات الخاصة مع البوت + المجموعات عبر grammY. يعتمد الاستطلاع الطويل افتراضيًا؛ webhook اختياري.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    سياسة الرسائل الخاصة (DM) الافتراضية في Telegram هي الاقتران.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    تشخيصات وإجراءات إصلاح عبر القنوات.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    أنماط وأمثلة إعدادات القناة الكاملة.
  
</Card>
</CardGroup>

## الإعداد (المسار السريع)

<Steps>
  <Step title="Create the bot token in BotFather">افتح Telegram وابدأ محادثة مع **@BotFather** ([رابط مباشر](https://t.me/BotFather)). تأكّد من أن المعرّف هو بالضبط `@BotFather`.

    ```
    نفّذ `/newbot`، ثم اتبع التعليمات (الاسم + اسم مستخدم ينتهي بـ `bot`).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    خيار متغيرات البيئة: `TELEGRAM_BOT_TOKEN=...` (يعمل للحساب الافتراضي).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    تنتهي صلاحية رموز الاقتران بعد ساعة واحدة.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    أضف البوت إلى مجموعتك، ثم عيّن `channels.telegram.groups` و`groupPolicy` بما يتوافق مع نموذج الوصول لديك.
  
</Step>
</Steps>

<Note>
ترتيب حلّ الرموز المميزة يعتمد على الحساب. عمليًا، تتقدّم قيم الإعدادات على متغيرات البيئة الاحتياطية، و`TELEGRAM_BOT_TOKEN` ينطبق فقط على الحساب الافتراضي.
</Note>

## إعدادات جانب Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">تعمل بوتات Telegram افتراضيًا في **وضع الخصوصية**، ما يقيّد الرسائل الجماعية التي تتلقّاها.

    ```
    `/setprivacy` — التحكّم فيما إذا كان البوت يرى جميع رسائل المجموعة.
    ```

  
</Accordion>

  <Accordion title="Group permissions">تُحدَّد حالة المشرف داخل المجموعة (واجهة Telegram).

    ```
    تتلقّى البوتات المشرفة دائمًا جميع
    رسائل المجموعة، لذا استخدم حالة المشرف إذا احتجت رؤية كاملة.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — السماح/المنع لإضافة البوت إلى المجموعات.
    ```

  
</Accordion>
</AccordionGroup>

## التحكم في الوصول والتفعيل

<Tabs>
  <Tab title="DM policy">
    يتحكم `channels.telegram.dmPolicy` في الوصول إلى الرسائل المباشرة:


    ```
    - `pairing` (الافتراضي)
    - `allowlist`
    - `open` (يتطلب أن تتضمن `allowFrom` القيمة `"*"`)
    - `disabled`
    
    يقبل `channels.telegram.allowFrom` معرفات مستخدمي Telegram الرقمية. يتم قبول البادئات `telegram:` / `tg:` وتوحيدها.
    يقبل معالج الإعداد الإدخال بصيغة `@username` ويحوّله إلى معرفات رقمية.
    إذا قمت بالترقية وكان إعدادك يحتوي على إدخالات قائمة سماح بصيغة `@username`، شغّل `openclaw doctor --fix` لتحويلها (بأفضل جهد؛ يتطلب رمز Telegram bot). 
    
    ### العثور على معرف مستخدم Telegram الخاص بك
    
    الطريقة الأكثر أمانًا (بدون بوت تابع لجهة خارجية):
    
    1. أرسل رسالة خاصة إلى البوت.
    2. شغّل `openclaw logs --follow`.
    3. اقرأ `from.id`.
    
    طريقة Bot API الرسمية:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    أرسل رسالة خاصة إلى `@userinfobot` أو `@getidsbot` واستخدم معرّف المستخدم المُعاد.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    يوجد عنصران تحكّم مستقلان:


    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">تتطلب ردود المجموعات ذكرًا افتراضيًا (ذكر @ الأصلي أو `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    يمكن أن تأتي الإشارة (Mention) من:
    
    - إشارة أصلية `@botusername`، أو
    - أنماط الإشارة في:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    مفاتيح تبديل الأوامر على مستوى الجلسة:
    
    - `/activation always`
    - `/activation mention`
    
    تحدّث هذه الأوامر حالة الجلسة فقط. استخدم الإعدادات للحفظ الدائم.
    
    مثال إعداد دائم:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    أعد توجيه أي رسالة من المجموعة إلى `@userinfobot` أو `@getidsbot` على Telegram لرؤية معرّف الدردشة (رقم سالب مثل `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## سلوك وقت التشغيل

- قناة Telegram Bot API مملوكة لـ Gateway.
- توجيه حتمي: الردود تعود إلى Telegram؛ ولا يختار النموذج القنوات.
- تُطبَّع الرسائل الواردة إلى غلاف القناة المشترك مع سياق الردّ ومواضع وسائط.
- يتم عزل جلسات المجموعات حسب معرف المجموعة. يضيف `:topic:<threadId>` إلى مفتاح جلسة مجموعة Telegram بحيث يكون كل موضوع معزولًا.
- يرسل مؤشرات الكتابة والردود مع `message_thread_id` لتبقى الردود ضمن الموضوع.
- يستخدم الاستطلاع الطويل (Long polling) grammY runner مع تسلسل لكل دردشة/لكل خيط. يستخدم الاستطلاع الطويل مُشغّل grammY مع تسلسل لكل دردشة؛ ويُقيَّد التوازي العام بواسطة `agents.defaults.maxConcurrent`.
- لا يدعم Telegram Bot API إيصالات القراءة؛ ولا يوجد خيار `sendReadReceipts`.

## مرجع الميزات

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">يمكن لـ OpenClaw بث ردود جزئية في الرسائل الخاصة على Telegram باستخدام `sendMessageDraft`.

    ```
    المتطلب:
    
    - `channels.telegram.streamMode` ليس `"off"` (الافتراضي: `"partial"`)
    
    الأوضاع:
    
    - `off`: بدون معاينة مباشرة
    - `partial`: تحديثات معاينة متكررة من نص جزئي
    - `block`: تحديثات معاينة مجزأة باستخدام `channels.telegram.draftChunk`
    
    القيم الافتراضية لـ `draftChunk` عند `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    يتم تقييد `maxChars` بواسطة `channels.telegram.textChunkLimit`.
    
    يعمل هذا في الدردشات المباشرة والمجموعات/المواضيع.
    
    بالنسبة للردود النصية فقط، يحتفظ OpenClaw بنفس رسالة المعاينة ويجري تعديلًا نهائيًا في مكانها (بدون رسالة ثانية).
    
    بالنسبة للردود المعقدة (مثل حمولات الوسائط)، يعود OpenClaw إلى التسليم النهائي العادي ثم يحذف رسالة المعاينة.
    
    `streamMode` منفصل عن البث الكتلي (block streaming). عند تفعيل البث الكتلي صراحةً لـ Telegram، يتخطى OpenClaw بث المعاينة لتجنب البث المزدوج.
    
    بث الاستدلال الخاص بـ Telegram فقط:
    
    - `/reasoning stream` يرسل الاستدلال إلى المعاينة المباشرة أثناء التوليد
    - يتم إرسال الإجابة النهائية بدون نص الاستدلال
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">يستخدم نص Telegram الصادر `parse_mode: "HTML"` (مجموعة وسوم Telegram المدعومة).

    ```
    - يتم تحويل النص الشبيه بـ Markdown إلى HTML آمن لـ Telegram.
    - يتم تهريب (escape) HTML الخام من النموذج لتقليل أخطاء التحليل في Telegram.
    - إذا رفض Telegram الـ HTML المُحلَّل، يعيد OpenClaw المحاولة كنص عادي.
    
    تكون معاينات الروابط مفعّلة افتراضيًا ويمكن تعطيلها باستخدام `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    يتم تسجيل قائمة أوامر Telegram عند بدء التشغيل باستخدام `setMyCommands`.


    ```
    يسجّل OpenClaw أوامر أصلية (مثل `/status`، `/reset`، `/model`) مع قائمة بوت Telegram عند بدء التشغيل. يمكنك إضافة أوامر مخصّصة إلى القائمة عبر التهيئة:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    القواعد:
    
    - يتم توحيد الأسماء (إزالة `/` في البداية، وتحويلها إلى أحرف صغيرة)
    - النمط الصالح: `a-z`، `0-9`، `_`، الطول `1..32`
    - لا يمكن للأوامر المخصصة تجاوز الأوامر الأصلية
    - يتم تخطي التعارضات/التكرارات وتسجيلها
    
    ملاحظات:
    
    - الأوامر المخصصة هي إدخالات قائمة فقط؛ لا تنفّذ سلوكًا تلقائيًا
    - يمكن لأوامر الإضافات/المهارات أن تعمل عند كتابتها حتى لو لم تظهر في قائمة Telegram
    
    إذا تم تعطيل الأوامر الأصلية، تتم إزالة الأوامر المدمجة. قد تستمر أوامر الإضافات/المخصصة في التسجيل إذا تم إعدادها.
    
    فشل شائع في الإعداد:
    
    - يشير `setMyCommands failed` عادةً إلى أن اتصال DNS/HTTPS الصادر إلى `api.telegram.org` محظور.
    
    ### أوامر اقتران الجهاز (`device-pair` plugin)
    
    عند تثبيت إضافة `device-pair`:
    
    1. `/pair` ينشئ رمز إعداد
    2. الصق الرمز في تطبيق iOS
    3. `/pair approve` يوافق على أحدث طلب معلّق
    
    مزيد من التفاصيل: [الاقتران](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    إعداد نطاق لوحة المفاتيح المضمنة:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    لتهيئة لكل حساب:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = لا تُقبل أي رسائل مجموعات إطلاقًا
      الافتراضي هو `groupPolicy: "allowlist"` (محجوب ما لم تُضف `groupAllowFrom`).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    عند نقر المستخدم زرًا، تُرسل بيانات الاستدعاء إلى الوكيل كرسالة بالتنسيق:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">    تتضمن إجراءات أدوات Telegram ما يلي:


    ```
    الأداة: `telegram` مع إجراء `react` (`chatId`، `messageId`، `emoji`).
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">يدعم Telegram ردودًا مترابطة اختيارية عبر الوسوم:

    ```
    - `[[reply_to_current]]` يرد على الرسالة المُشغِّلة
    - `[[reply_to:<id>]]` يرد على معرف رسالة Telegram محدد
    
    يتحكم `channels.telegram.replyToMode` في آلية المعالجة:
    
    - `off` (الافتراضي)
    - `first`
    - `all`
    
    ملاحظة: يؤدي `off` إلى تعطيل ربط الردود الضمني. لا تزال وسوم `[[reply_to_*]]` الصريحة محترمة.
    
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    مجموعات المنتدى الفائقة:


    ```
    **مجموعات المنتدى:** تتضمن التفاعلات في مجموعات المنتدى `message_thread_id` وتستخدم مفاتيح جلسة مثل `agent:main:telegram:group:{chatId}:topic:{threadId}`.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">    ### الرسائل الصوتية


    ```
    يميز Telegram بين الملاحظات الصوتية وملفات الصوت.
    
    - الافتراضي: سلوك ملف صوتي
    - الوسم `[[audio_as_voice]]` في رد الوكيل لفرض الإرسال كملاحظة صوتية
    
    مثال على إجراء رسالة:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### رسائل الفيديو
    
    يميز Telegram بين ملفات الفيديو وملاحظات الفيديو.
    
    مثال على إجراء رسالة:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    لا تدعم ملاحظات الفيديو التسميات التوضيحية؛ يتم إرسال نص الرسالة المرفق بشكل منفصل.
    
    ### الملصقات
    
    معالجة الملصقات الواردة:
    
    - ‏WEBP ثابت: يتم تنزيله ومعالجته (عنصر نائب `<media:sticker>`)
    - ‏TGS متحرك: يتم تجاهله
    - ‏WEBM فيديو: يتم تجاهله
    
    حقول سياق الملصق:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    ملف ذاكرة التخزين المؤقت للملصقات:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    يتم وصف الملصقات مرة واحدة (عند الإمكان) وتخزينها مؤقتًا لتقليل استدعاءات الرؤية المتكررة.
    
    تفعيل إجراءات الملصقات:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    إرسال الملصقات
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    **البحث عن ملصقات:**
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">يتلقى تحديث `message_reaction` من Telegram API

    ```
    عند التفعيل، يضع OpenClaw أحداث نظام في قائمة الانتظار مثل:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    الإعداد:
    
    - `channels.telegram.reactionNotifications`: ‏`off | own | all` (الافتراضي: `own`)
    - `channels.telegram.reactionLevel`: ‏`off | ack | minimal | extensive` (الافتراضي: `minimal`)
    
    ملاحظات:
    
    - ‏`own` تعني تفاعلات المستخدمين مع الرسائل التي أرسلها البوت فقط (بأفضل جهد عبر ذاكرة تخزين الرسائل المرسلة).
    - لا يوفر Telegram معرفات الخيوط في تحديثات التفاعل.
      - المجموعات غير المنتدى تُوجَّه إلى جلسة دردشة المجموعة
      - مجموعات المنتدى تُوجَّه إلى جلسة الموضوع العام للمجموعة (`:topic:1`)، وليس إلى الموضوع الأصلي بالضبط
    
    يتضمن `allowed_updates` للاستطلاع/الويب هوك `message_reaction` تلقائيًا.
    
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    يرسل `ackReaction` رمزًا تعبيريًا للتأكيد أثناء معالجة OpenClaw لرسالة واردة.


    ```
    ترتيب الحل:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - الرجوع إلى رمز هوية الوكيل (`agents.list[].identity.emoji`، وإلا "👀")
    
    ملاحظات:
    
    - يتوقع Telegram رمزًا تعبيريًا بصيغة يونيكود (مثل "👀").
    - استخدم `""` لتعطيل التفاعل لقناة أو حساب معين.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">    يتم تمكين عمليات كتابة إعدادات القناة افتراضيًا (`configWrites !== false`).

    ```
    تتم ترقية مجموعة إلى supergroup ويُصدر Telegram `migrate_to_chat_id` (يتغيّر معرّف الدردشة).
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">    الافتراضي: الاستقصاء الطويل (long polling).

    ```
    إذا كان عنوان URL العام مختلفًا، استخدم وكيلًا عكسيًا ووجّه `channels.telegram.webhookUrl` إلى نقطة النهاية العامة.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    يُجزَّأ النص الصادر إلى `channels.telegram.textChunkLimit` (الافتراضي 4000).
    تجزئة اختيارية على الأسطر الجديدة: عيّن `channels.telegram.chunkMode="newline"` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل تجزئة الطول.
    تُقيَّد تنزيلات/تحميلات الوسائط بـ `channels.telegram.mediaMaxMb` (الافتراضي 5).
    - يقوم `channels.telegram.timeoutSeconds` بتجاوز مهلة عميل Telegram API (إذا لم يتم تعيينه، يتم تطبيق الإعداد الافتراضي لـ grammY).
    يستخدم سياق سجلّ المجموعة `channels.telegram.historyLimit` (أو `channels.telegram.accounts.*.historyLimit`)، مع الرجوع إلى `messages.groupChat.historyLimit`. عيّن `0` لتعطيله (الافتراضي 50).
    - عناصر التحكم في سجل الرسائل الخاصة (DM):
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>تعيد استدعاءات Telegram API الصادرة المحاولة عند أخطاء الشبكة العابرة/429 مع تراجع أُسّي وإزاحة عشوائية. يمكن التهيئة عبر `channels.telegram.retry`.

    ```
    يمكن أن يكون هدف الإرسال عبر CLI رقم معرّف الدردشة (chat ID) أو اسم المستخدم:
    ```

```bash
مثال: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## استكشاف الأخطاء وإصلاحها

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    إذا عيّنت `channels.telegram.groups.*.requireMention=false`، فيجب تعطيل **وضع الخصوصية** في Bot API الخاص بـ Telegram.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - عند وجود `channels.telegram.groups`، يجب إدراج المجموعة (أو تضمين `"*"`)
    - تحقق من عضوية البوت في المجموعة
    - راجع السجلات: `openclaw logs --follow` لمعرفة أسباب التجاهل
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - قم بتفويض هوية المُرسِل (الاقتران و/أو `allowFrom` الرقمي)
    - يظل تفويض الأوامر ساريًا حتى عندما تكون سياسة المجموعة `open`
    - يشير `setMyCommands failed` عادةً إلى مشكلات في الوصول عبر DNS/HTTPS إلى `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - قد يؤدي استخدام Node 22+ مع fetch/proxy مخصص إلى سلوك إجهاض فوري إذا لم تتطابق أنواع AbortSignal.
    - تقوم بعض المستضيفات بحل `api.telegram.org` إلى IPv6 أولًا؛ ويمكن أن يتسبب خروج IPv6 المعطّل في فشل متقطع لواجهة Telegram API.
    - تحقق من استجابات DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

مزيد من المساعدة: [استكشاف أخطاء القنوات](/channels/troubleshooting).

## مرجع التهيئة (Telegram)

المرجع الأساسي:

- `channels.telegram.enabled`: تمكين/تعطيل بدء القناة.

- `channels.telegram.botToken`: رمز البوت (BotFather).

- `channels.telegram.tokenFile`: قراءة الرمز من مسار ملف.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (الافتراضي: الإقران).

- `channels.telegram.allowFrom`: قائمة السماح للرسائل الخاصة (DM) (معرّفات مستخدمي Telegram الرقمية). يتطلب `open` وجود `"*"`. يمكن للأمر `openclaw doctor --fix` حل إدخالات `@username` القديمة إلى معرّفات رقمية.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (الافتراضي: قائمة سماح).

- `channels.telegram.groupAllowFrom`: قائمة سماح مرسلي المجموعات (معرّفات/أسماء مستخدمين). يمكن للأمر `openclaw doctor --fix` حل إدخالات `@username` القديمة إلى معرّفات رقمية.

- `channels.telegram.groups`: افتراضيات لكل مجموعة + قائمة سماح (استخدم `"*"` للافتراضيات العامة).
  - `channels.telegram.groups.<id>.groupPolicy`: تجاوز لكل مجموعة لـ groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: افتراضي بوابة الذِكر.
  - `channels.telegram.groups.<id>.skills`: مُرشِّح skills (الحذف = كل skills، الفارغ = لا شيء).
  - `channels.telegram.groups.<id>.allowFrom`: تجاوز قائمة سماح المرسلين لكل مجموعة.
  - `channels.telegram.groups.<id>.systemPrompt`: مطالبة نظام إضافية للمجموعة.
  - `channels.telegram.groups.<id>.enabled`: تعطيل المجموعة عندما `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: تجاوزات لكل موضوع (نفس حقول المجموعة).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: تجاوز لكل موضوع لـ groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: تجاوز بوابة الذِكر لكل موضوع.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (الافتراضي: قائمة سماح).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: تجاوز لكل حساب.

- `channels.telegram.replyToMode`: `off | first | all` (الافتراضي: `first`).

- `channels.telegram.textChunkLimit`: حجم تجزئة الصادر (أحرف).

- `channels.telegram.chunkMode`: `length` (افتراضي) أو `newline` للتقسيم على الأسطر الفارغة (حدود الفقرات) قبل تجزئة الطول.

- `channels.telegram.linkPreview`: تبديل معاينات الروابط للرسائل الصادرة (الافتراضي: true).

- `channels.telegram.streamMode`: `off | partial | block` (بثّ المسودات).

- `channels.telegram.mediaMaxMb`: حد الوسائط الواردة/الصادرة (MB).

- `channels.telegram.retry`: سياسة إعادة المحاولة لاستدعاءات Telegram API الصادرة (عدد المحاولات، minDelayMs، maxDelayMs، jitter).

- `channels.telegram.network.autoSelectFamily`: تجاوز Node autoSelectFamily (true=تمكين، false=تعطيل). الافتراضي معطّل على Node 22 لتجنّب مهلات Happy Eyeballs.

- `channels.telegram.proxy`: عنوان URL للوكيل لاستدعاءات Bot API (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: تمكين وضع webhook (يتطلب `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: سرّ webhook (مطلوب عند تعيين webhookUrl).

- `channels.telegram.webhookPath`: مسار webhook المحلي (الافتراضي `/telegram-webhook`).

- يرتبط المستمع المحلي بـ `0.0.0.0:8787` ويخدم `POST /telegram-webhook` افتراضيًا.

- `channels.telegram.actions.reactions`: بوابة تفاعلات أداة Telegram.

- `channels.telegram.actions.sendMessage`: بوابة إرسال رسائل أداة Telegram.

- `channels.telegram.actions.deleteMessage`: بوابة حذف رسائل أداة Telegram.

- `channels.telegram.actions.sticker`: بوابة إجراءات ملصقات Telegram — الإرسال والبحث (الافتراضي: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — التحكّم في أي التفاعلات تُطلق أحداث النظام (الافتراضي: `own` عند عدم التعيين).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — التحكّم في قدرة الوكيل على التفاعل (الافتراضي: `minimal` عند عدم التعيين).

- [مرجع الإعدادات - Telegram](/gateway/configuration-reference#telegram)

حقول Telegram ذات الإشارة العالية:

- بدء التشغيل/المصادقة: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- التحكم في الوصول: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- الأوامر/القائمة: `commands.native`, `customCommands`
- المح threads/الردود: `replyToMode`
- اختياري (فقط لـ `streamMode: "block"`):
- التنسيق/التسليم: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- الوسائط/الشبكة: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- وضع webhook: عيّن `channels.telegram.webhookUrl` و `channels.telegram.webhookSecret` (اختياريًا `channels.telegram.webhookPath`).
- الإجراءات/القدرات: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- التفاعلات: `reactionNotifications`, `reactionLevel`
- الكتابة/السجل: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## ذو صلة

- التفاصيل: [الإقران](/channels/pairing)
- [توجيه القنوات](/channels/channel-routing)
- التهيئة الكاملة: [التهيئة](/gateway/configuration)
