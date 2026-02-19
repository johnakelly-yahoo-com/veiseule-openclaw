---
summary: "إعداد Slack لوضع Socket أو وضع Webhook عبر HTTP"
read_when:
  - عند إعداد Slack أو استكشاف أخطاء وضع Socket/HTTP في Slack
title: "Slack"
---

# Slack

الحالة: جاهز للاستخدام في بيئة الإنتاج للرسائل الخاصة (DMs) + القنوات عبر تكاملات تطبيق Slack. الوضع الافتراضي هو Socket Mode؛ كما أن وضع HTTP Events API مدعوم أيضًا.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Slack DMs تكون في وضع الاقتران افتراضيًا.
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">سلوك الأوامر الأصلي وفهرس الأوامر.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">أدلة تشخيص وإصلاح عبر القنوات.
</Card>
</CardGroup>

## إعداد سريع (للمبتدئين)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        في إعدادات تطبيق Slack:

        ```
        **Socket Mode** → فعّل الخيار. ثم انتقل إلى **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** مع النطاق `connections:write`. انسخ **App Token** (`xapp-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            الرجوع إلى متغيرات البيئة (للحساب الافتراضي فقط):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="الاشتراك في أحداث التطبيق">
          اشترك في أحداث البوت التالية:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          فعّل أيضًا **Messages Tab** في App Home للرسائل الخاصة (DMs).
        
</Step>
      
        <Step title="تشغيل Gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        استخدم وضع Webhook عبر HTTP عندما يكون الـ Gateway قابلاً للوصول من Slack عبر HTTPS (وهو شائع في نشر الخوادم). يستخدم وضع HTTP واجهة الأحداث + التفاعلية + الأوامر المائلة مع عنوان طلب مشترك.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Step>
      
        <Step title="استخدام مسارات webhook فريدة لوضع HTTP متعدد الحسابات">
          يتم دعم وضع HTTP لكل حساب على حدة.
      
          امنح كل حساب `webhookPath` مميزًا حتى لا تتعارض عمليات التسجيل.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## نموذج الرموز (Tokens)

- `botToken` + `appToken` مطلوبان لوضع Socket Mode.
- وضع HTTP يتطلب `botToken` + `signingSecret`.
- الرموز في الإعدادات تتجاوز الرجوع إلى متغيرات البيئة.
- الرجوع إلى متغيرات البيئة `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` ينطبق فقط على الحساب الافتراضي.
- `userToken` (`xoxp-...`) يقتصر على الإعدادات فقط (لا يوجد رجوع إلى متغيرات البيئة) ويكون افتراضيًا بسلوك للقراءة فقط (`userTokenReadOnly: true`).
- اختياري: أضف `chat:write.customize` إذا كنت تريد أن تستخدم الرسائل الصادرة هوية الوكيل النشطة (اسم مستخدم `username` وأيقونة مخصصان). `icon_emoji` يستخدم صيغة `:emoji_name:`.

<Tip>
بالنسبة للإجراءات/قراءة الدليل، يمكن تفضيل user token عند تهيئته. حتى مع `userTokenReadOnly: false`، يبقى رمز البوت مفضّلًا للكتابة عند توفره.
</Tip>

## التحكم في الوصول والتوجيه

<Tabs>
  <Tab title="DM policy">للسماح للجميع: عيّن `channels.slack.dm.policy="open"` و`channels.slack.dm.allowFrom=["*"]`.

    ```
    - `pairing` (افتراضي)
    - `allowlist`
    - `open` (يتطلب أن يتضمن `channels.slack.allowFrom` القيمة `"*"`؛ قديم: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    إعدادات DM:
    
    - `dm.enabled` (افتراضيًا true)
    - `channels.slack.allowFrom` (مفضل)
    - `dm.allowFrom` (قديم)
    - `dm.groupEnabled` (الرسائل الجماعية الخاصة افتراضيًا false)
    - `dm.groupChannels` (قائمة سماح اختيارية لـ MPIM)
    
    يتم الاقتران في الرسائل الخاصة باستخدام `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">يتحكم `channels.slack.groupPolicy` في التعامل مع القنوات (`open|disabled|allowlist`).

    ```
    متصل لكن لا توجد ردود في القنوات: القناة محظورة بواسطة `groupPolicy` أو غير مدرجة في قائمة السماح `channels.slack.channels`.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">رسائل القنوات تتطلب الإشارة (mention) افتراضيًا.

    ```
    يتحكم ضبط الإشارة عبر `channels.slack.channels` (عيّن `requireMention` إلى `true`)؛ كما تُحتسب `agents.list[].groupChat.mentionPatterns` (أو `messages.groupChat.mentionPatterns`) كإشارات أيضًا.
    ```

  
</Tab>
</Tabs>

## الأوامر وسلوك أوامر الشرطة المائلة (slash)

- الوضع الافتراضي للأوامر الأصلية في Slack هو الإيقاف ما لم تضبط `channels.slack.commands.native: true` (القيمة العامة `commands.native` هي `"auto"` والتي تُبقي Slack مُعطّلًا).
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  },
  },
  }
- عند تفعيل الأوامر الأصلية، قم بتسجيل أوامر slash المطابقة في Slack (أسماء `/<command>`).
- إذا فعّلت الأوامر الأصلية، فأضف إدخال `slash_commands` واحدًا لكل أمر تريد كشفه (مطابقًا لقائمة `/help`). تجاوز ذلك باستخدام `channels.slack.commands.native`.

إعدادات أمر slash الافتراضية:

- `enabled`: عيّن `false` لتعطيل القناة.
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

تستخدم جلسات slash مفاتيح معزولة:

- تستخدم الأوامر المائلة جلسات `agent:<agentId>:slack:slash:<userId>` (البادئة قابلة للتهيئة عبر `channels.slack.slashCommand.sessionPrefix`).

ومع ذلك يتم توجيه تنفيذ الأمر إلى جلسة المحادثة المستهدفة (`CommandTargetSessionKey`).

## الخيوط (Threads)، الجلسات، وعلامات الرد

- يتم توجيه الرسائل الخاصة كـ `direct`؛ والقنوات كـ `channel`؛ وMPIMs كـ `group`.
- مع الإعداد الافتراضي `session.dmScope=main`، يتم دمج رسائل Slack الخاصة ضمن الجلسة الرئيسية للوكيل.
- تُطابِق القنوات جلسات `agent:<agentId>:slack:channel:<channelId>`.
- يمكن أن تنشئ الردود في الخيوط لاحقات جلسة للخيط (`:thread:<threadTs>`) عند الاقتضاء.
- القيمة الافتراضية لـ `channels.slack.thread.historyScope` هي `thread`؛ و`thread.inheritParent` افتراضيًا `false`.
- `channels.slack.thread.initialHistoryLimit` يتحكم في عدد رسائل الخيط الحالية التي يتم جلبها عند بدء جلسة خيط جديدة (الافتراضي `20`؛ عيّنها إلى `0` للتعطيل).

عناصر التحكم في الرد ضمن الخيوط:

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- خيار قديم احتياطي للمحادثات المباشرة: `channels.slack.dm.replyToMode`

يتم دعم علامات الرد اليدوية:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

ملاحظة: يؤدي `replyToMode="off"` إلى تعطيل ترابط الردود الضمني. لا تزال وسوم `[[reply_to_*]]` الصريحة مُعتمدة.

## الوسائط، التقسيم، والتسليم

<AccordionGroup>
  <Accordion title="Inbound attachments">
    يتم تنزيل مرفقات ملفات Slack من عناوين URL خاصة مستضافة على Slack (تدفق طلبات بمصادقة الرمز) وكتابتها في مخزن الوسائط عند نجاح الجلب والسماح بحدود الحجم.

    ```
    يبلغ الحد الأقصى الافتراضي لحجم البيانات الواردة وقت التشغيل `20MB` ما لم يتم تجاوزه عبر `channels.slack.mediaMaxMb`.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - تستخدم مقاطع النص `channels.slack.textChunkLimit` (الافتراضي 4000)
    - يفعّل `channels.slack.chunkMode="newline"` التقسيم حسب الفقرات أولاً
    - تستخدم عمليات إرسال الملفات واجهات Slack upload APIs ويمكن أن تتضمن ردودًا ضمن سلاسل المحادثات (`thread_ts`)
    - يتبع الحد الأقصى للوسائط الصادرة قيمة `channels.slack.mediaMaxMb` عند تهيئته؛ وإلا تستخدم عمليات الإرسال الافتراضية حسب نوع MIME من مسار معالجة الوسائط
  
</Accordion>

  <Accordion title="Delivery targets">
    الأهداف الصريحة المفضلة:

    ```
    - `user:<id>` للرسائل المباشرة (DMs)
    - `channel:<id>` للقنوات
    
    يتم فتح رسائل Slack المباشرة عبر Slack conversation APIs عند الإرسال إلى أهداف المستخدمين.
    ```

  
</Accordion>
</AccordionGroup>

## الإجراءات والضوابط

يمكن تقييد إجراءات أدوات Slack باستخدام `channels.slack.actions.*`:

مجموعات الإجراءات المتاحة في أدوات Slack الحالية:

| سياسة المجموعات | الافتراضي |
| --------------- | --------- |
| messages        | مفعّل     |
| reactions       | مفعّل     |
| pins            | مفعّل     |
| memberInfo      | مفعّل     |
| emojiList       | مفعّل     |

## الأحداث والسلوك التشغيلي

- يتم تحويل تعديلات/حذف الرسائل وبث سلاسل المحادثات إلى أحداث نظام.
- يتم تحويل أحداث إضافة/إزالة التفاعلات إلى أحداث نظام.
- يتم تحويل أحداث انضمام/مغادرة الأعضاء، وإنشاء/إعادة تسمية القنوات، وإضافة/إزالة التثبيت إلى أحداث نظام.
- يمكن لـ `channel_id_changed` ترحيل مفاتيح تهيئة القناة عند تمكين `configWrites`.
- تُعامل بيانات موضوع/غرض القناة الوصفية كسياق غير موثوق ويمكن حقنها في سياق التوجيه.

## تشعيب الردود

يرسل `ackReaction` رمزًا تعبيريًا للتأكيد بينما يقوم OpenClaw بمعالجة رسالة واردة.

ترتيب المعالجة:

- ولتعدد الحسابات، اضبط `channels.slack.accounts.<id>`.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- الرجوع إلى الرمز التعبيري لهوية الوكيل (`agents.list[].identity.emoji`، وإلا "👀")

ملاحظات

- يتوقع Slack استخدام الأسماء المختصرة (على سبيل المثال `"eyes"`).
- استخدم `""` لتعطيل التفاعل لقناة أو حساب.

## قائمة التحقق من البيان والنطاقات

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    إذا قمت بتهيئة `channels.slack.userToken`، فعادةً ما تكون نطاقات القراءة:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## استكشاف الأخطاء وإصلاحها

<AccordionGroup>
  <Accordion title="No replies in channels">
    تحقق، بالترتيب:

    ```
    `allow`: السماح/المنع للقناة عندما `groupPolicy="allowlist"`.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    تحقق:

    ```
    يتطلب `allowlist` إدراج القنوات في `channels.slack.channels`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    تحقق من رموز bot وapp وتفعيل Socket Mode في إعدادات تطبيق Slack.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    تحقق من:

    ```
    - signing secret
    - مسار webhook
    - عناوين Slack Request URLs (الأحداث + التفاعلية + Slash Commands)
    - قيمة `webhookPath` فريدة لكل حساب HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    تأكد مما إذا كنت تقصد:

    ```
    {
      channels: {
        slack: {
          enabled: true,
          appToken: "xapp-...",
          botToken: "xoxb-...",
          userToken: "xoxp-...",
          userTokenReadOnly: false,
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## مراجع التهيئة

الأولوية:

- أنشئ تطبيق Slack (From scratch) في [https://api.slack.com/apps](https://api.slack.com/apps).

  حقول Slack عالية الأهمية:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - الوصول إلى الرسائل المباشرة (DM): `dm.enabled`, `dmPolicy`, `allowFrom` (القديم: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - للسماح **بعدم وجود أي قنوات**، عيّن `channels.slack.groupPolicy: "disabled"` (أو أبقِ قائمة السماح فارغة).
  - الخيوط/السجل: `replyToMode`، `replyToModeByChatType`، `thread.*`، `historyLimit`، `dmHistoryLimit`، `dms.*.historyLimit`
  - التسليم: `textChunkLimit`، `chunkMode`، `mediaMaxMb`
  - العمليات/الميزات: `configWrites`، `commands.native`، `slashCommand.*`، `actions.*`، `userToken`، `userTokenReadOnly`

## ذو صلة

- [الاقتران](/channels/pairing)
- `channel`: قنوات قياسية (عامة/خاصة)
- لتدفق الفرز: [/channels/troubleshooting](/channels/troubleshooting).
- [الإعدادات](/gateway/configuration)
- قائمة الأوامر الكاملة + التهيئة: [الأوامر المائلة](/tools/slash-commands)
