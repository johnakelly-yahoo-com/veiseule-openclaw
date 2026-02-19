---
summary: "حالة دعم بوت Discord، والقدرات، والتهيئة"
read_when:
  - العمل على ميزات قناة Discord
title: "Discord"
---

# Discord (Bot API)

الحالة: جاهز للرسائل المباشرة (DM) وقنوات نصّ الخوادم (guild) عبر بوابة بوت Discord الرسمية.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    الرسائل الخاصة في Discord تستخدم وضع الاقتران افتراضيًا.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">    سلوك الأوامر الأصلية وكتالوج الأوامر.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    تشخيص عبر القنوات وتدفق الإصلاح.
  
</Card>
</CardGroup>

## إعداد سريع (للمبتدئين)

<Steps>
  <Step title="Create a Discord bot and enable intents">إنشاء تطبيق Discord + مستخدم البوت

    ```
    **Server Members Intent** (موصى به؛ مطلوب لبعض عمليات البحث عن الأعضاء/المستخدمين ومطابقة قوائم السماح في الخوادم)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    الرجوع إلى متغيرات البيئة للحساب الافتراضي:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">ادعُ البوت إلى خادمك مع الأذونات اللازمة لقراءة/إرسال الرسائل حيث تريد استخدامه.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    تنتهي صلاحية رموز الاقتران بعد ساعة واحدة.
    ```

  
</Step>
</Steps>

<Note>
يتم حل التوكن بناءً على الحساب. قيم التوكن في الإعدادات تتقدم على الرجوع إلى متغيرات البيئة. `DISCORD_BOT_TOKEN` يُستخدم فقط للحساب الافتراضي.
</Note>

## نموذج وقت التشغيل

- Gateway يمتلك اتصال Discord.
- `[[reply_to_current]]` — الرد على رسالة Discord المُحفِّزة.
- يمكن للوكيل استدعاء `discord` بإجراءات مثل:
- المحادثات المباشرة تُدمج في الجلسة الرئيسية للوكيل (الافتراضي `agent:main:main`)؛ بينما تبقى قنوات الخوادم معزولة كـ `agent:<agentId>:discord:channel:<channelId>` (تستخدم أسماء العرض `discord:<guildSlug>#<channelSlug>`).
- `dm.groupEnabled`: تمكين الرسائل المباشرة الجماعية (الافتراضي `false`).
- تستخدم الأوامر الأصلية مفاتيح جلسات معزولة (`agent:<agentId>:discord:slash:<userId>`) بدل الجلسة المشتركة `main`.

## التحكم في الوصول والتوجيه

<Tabs>
  <Tab title="DM policy">للحفاظ على السلوك القديم «مفتوح للجميع»: عيّن `channels.discord.dm.policy="open"` و `channels.discord.dm.allowFrom=["*"]`.

    ```
    لقائمة سماح صارمة: عيّن `channels.discord.dm.policy="allowlist"` وأدرج المرسلين في `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">    يتم التحكم في معالجة Guild عبر `channels.discord.groupPolicy`:

    ```
    {
      channels: {
        discord: {
          enabled: true,
          dm: { enabled: false },
          guilds: {
            YOUR_GUILD_ID: {
              users: ["YOUR_USER_ID"],
              requireMention: true,
              channels: {
                help: { allow: true, requireMention: true },
              },
            },
          },
          retry: {
            attempts: 3,
            minDelayMs: 500,
            maxDelayMs: 30000,
            jitter: 0.1,
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    إذا عيّنت فقط `DISCORD_BOT_TOKEN` ولم تُنشئ قسم `channels.discord`، فإن وقت التشغيل
        يعيّن `groupPolicy` افتراضيًا إلى `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    يتم تقييد رسائل Guild بالإشارة (mention) بشكل افتراضي.

    ```
    يشمل اكتشاف الإشارة ما يلي:
    
    - الإشارة الصريحة إلى البوت
    - أنماط الإشارة المُعدّة (`agents.list[].groupChat.mentionPatterns`، والاحتياطي `messages.groupChat.mentionPatterns`)
    - سلوك الرد الضمني على البوت في الحالات المدعومة
    
    يتم إعداد `requireMention` لكل Guild/قناة على حدة (`channels.discord.guilds...`).
    
    الرسائل الخاصة الجماعية (Group DMs):
    
    - الافتراضي: يتم تجاهلها (`dm.groupEnabled=false`)
    - يمكن السماح بها اختياريًا عبر قائمة سماح في `dm.groupChannels` (معرّفات القنوات أو الأسماء المختصرة)
    ```

  
</Tab>
</Tabs>

### توجيه الوكلاء بناءً على الأدوار

استخدم `bindings[].match.roles` لتوجيه أعضاء Discord guild إلى وكلاء مختلفين حسب معرّف الدور. تقبل عمليات الربط المعتمدة على الأدوار معرّفات الأدوار فقط ويتم تقييمها بعد عمليات الربط peer أو parent-peer وقبل عمليات الربط الخاصة بـ guild فقط. إذا كان الربط يحدد أيضًا حقول مطابقة أخرى (على سبيل المثال `peer` + `guildId` + `roles`)، فيجب أن تتطابق جميع الحقول المُعدّة.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## إعداد Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    أنشئ بوت Discord وانسخ رمز البوت (bot token).
    ```

  
</Accordion>

  <Accordion title="Privileged intents">في **Bot** → **Privileged Gateway Intents**، فعّل:

    ```
    عادةً **لا** تحتاج إلى **Presence Intent**. تعيين حالة حضور البوت نفسه (إجراء `setPresence`) يستخدم OP3 للبوابة ولا يتطلب هذا المقصد؛ يلزم فقط إذا أردت تلقي تحديثات الحضور لأعضاء خادم آخرين.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    مُولّد رابط OAuth:

    ```
    - scopes: `bot`, `applications.commands`
    
    الأذونات الأساسية المعتادة:
    
    - عرض القنوات
    - إرسال الرسائل
    - قراءة سجل الرسائل
    - تضمين الروابط
    - إرفاق الملفات
    - إضافة التفاعلات (اختياري)
    
    تجنب منح `Administrator` ما لم تكن هناك حاجة صريحة لذلك.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    فعّل وضع المطوّر في Discord، ثم انسخ:

    ```
    - معرّف الخادم
    - معرّف القناة
    - معرّف المستخدم
    
    يفضّل استخدام المعرّفات الرقمية في إعدادات OpenClaw لضمان تدقيق وفحص أكثر موثوقية.
    ```

  
</Accordion>
</AccordionGroup>

## الأوامر الأصلية وتوثيق الأوامر

- الأوامر الأصلية الاختيارية: القيمة الافتراضية لـ `commands.native` هي `"auto"` (مفعّل لـ Discord/Telegram، ومعطّل لـ Slack).
- يُتحكَّم بالسلوك عبر `channels.discord.replyToMode`:
- يمكن التجاوز عبر `channels.discord.commands.native: true|false|"auto"`؛ ويقوم `false` بمسح الأوامر المسجّلة سابقًا.
- يستخدم توثيق الأوامر الأصلية نفس قوائم السماح/السياسات في Discord كما في معالجة الرسائل العادية.
- قد تبقى أوامر Slash مرئية في واجهة Discord لمستخدمين غير مُدرجين؛ يفرض OpenClaw قوائم السماح عند التنفيذ ويرد «غير مخوّل».

راجع [Exec approvals](/tools/exec-approvals) و [Slash commands](/tools/slash-commands) لتدفق الموافقات والأوامر الأوسع.

## تفاصيل الميزة

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    يدعم Discord وسوم الرد في مخرجات الوكيل:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    يتم التحكم بها عبر `channels.discord.replyToMode`:
    
    - `off` (الافتراضي)
    - `first`
    - `all`
    
    ملاحظة: يؤدي `off` إلى تعطيل تسلسل الردود الضمني. ومع ذلك، يتم احترام وسوم `[[reply_to_*]]` الصريحة.
    
    يتم إظهار معرّفات الرسائل ضمن السياق/السجل بحيث يمكن للوكلاء استهداف رسائل محددة.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    سياق سجل Guild:

    ```
    - `channels.discord.historyLimit` الافتراضي `20`
    - الاحتياطي: `messages.groupChat.historyLimit`
    - `0` لتعطيل الميزة
    
    عناصر التحكم في سجل الرسائل الخاصة (DM):
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    سلوك Threads:
    
    - يتم توجيه Discord threads كجلسات قنوات
    - يمكن استخدام بيانات الـ thread الأصلية للربط مع جلسة الأصل
    - ترث إعدادات الـ thread إعدادات القناة الأصلية ما لم يوجد إعداد خاص بالـ thread
    
    يتم حقن مواضيع القنوات كسياق **غير موثوق** (وليس كـ system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: وضع أحداث نظام التفاعلات (`off`، `own`، `all`، `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    يرسل `ackReaction` رمزًا تعبيريًا للتأكيد أثناء قيام OpenClaw بمعالجة رسالة واردة.

    ```
    ترتيب المعالجة:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - الرمز التعبيري لهوية الوكيل كخيار احتياطي (`agents.list[].identity.emoji`، وإلا "👀")
    
    ملاحظات:
    
    - يقبل Discord الرموز التعبيرية الموحدة (unicode) أو أسماء الرموز التعبيرية المخصصة.
    - استخدم `""` لتعطيل التفاعل لقناة أو حساب.
    ```

  
</Accordion>

  <Accordion title="Config writes">.channels`، تُرفَض القنوات غير المدرجة افتراضيًا.

    ```
    يؤثر هذا على تدفقات `/config set|unset` (عند تمكين ميزات الأوامر).
    
    تعطيل:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    قم بتوجيه حركة WebSocket الخاصة بـ Discord gateway عبر وكيل HTTP(S) باستخدام `channels.discord.proxy`.

```json5
مثال «خادم واحد، السماح لي فقط، السماح لقناة #help فقط»:
```

    ```
    تجاوز الإعداد لكل حساب:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    فعّل تحليل PluralKit لربط الرسائل المُمرّرة بهوية عضو النظام:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    ملاحظات:
    
    - يمكن لقوائم السماح استخدام `pk:<memberId>`
    - تتم مطابقة أسماء عرض الأعضاء حسب الاسم/الاسم المختصر
    - تعتمد عمليات البحث على معرّف الرسالة الأصلي وتكون مقيّدة بنافذة زمنية
    - إذا فشل البحث، تُعامل الرسائل المُمرّرة كرسائل بوت ويتم تجاهلها ما لم يكن `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    يتم تطبيق تحديثات الحالة (Presence) فقط عند تعيين حقل الحالة أو النشاط.

    ```
    مثال على الحالة فقط:
    ```

```json5
أو في التهيئة: `channels.discord.token: "..."`.
```

    ```
    مثال على النشاط (الحالة المخصصة هي نوع النشاط الافتراضي):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    مثال على البث:
    ```

```json5
هيّئ OpenClaw باستخدام `channels.discord.token` (أو `DISCORD_BOT_TOKEN` كخيار احتياطي).
```

    ```
    خريطة أنواع النشاط:
    
    - 0: Playing
    - 1: Streaming (يتطلب `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (يستخدم نص النشاط كحالة؛ الرمز التعبيري اختياري)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    يدعم Discord الموافقات على التنفيذ (exec) عبر الأزرار في الرسائل الخاصة (DM)، ويمكنه اختياريًا نشر طلبات الموافقة في القناة الأصلية.

    ```
    مسار الإعداد:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`، الافتراضي: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    عندما يكون `target` هو `channel` أو `both`، يظهر طلب الموافقة في القناة. يمكن فقط للموافقين المُعدّين استخدام الأزرار؛ أما المستخدمون الآخرون فيتلقون رفضًا مؤقتًا (ephemeral). تتضمن طلبات الموافقة نص الأمر، لذا فعّل الإرسال إلى القناة فقط في القنوات الموثوقة. إذا تعذر اشتقاق معرّف القناة من مفتاح الجلسة، يعود OpenClaw إلى الإرسال عبر DM.
    
    إذا فشلت الموافقات بسبب معرّفات موافقة غير معروفة، فتحقق من قائمة الموافقين ومن تفعيل الميزة.
    
    مستندات ذات صلة: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## إجراءات الأدوات

تتضمن إجراءات رسائل Discord المراسلة، وإدارة القنوات، والإشراف، والحالة (presence)، وإجراءات البيانات الوصفية.

أمثلة أساسية:

- `readMessages`، `sendMessage`، `editMessage`، `deleteMessage`
- `react` / `reactions` (إضافة أو سرد التفاعلات)
- `timeout`، `kick`، `ban`
- الحالة: `setPresence`

توجد بوابات الإجراءات ضمن `channels.discord.actions.*`.

السلوك الافتراضي للبوابة:

| مجموعة الإجراءات                                                                                              | الافتراضي |
| ------------------------------------------------------------------------------------------------------------- | --------- |
| `stickers`، `emojiUploads`، `stickerUploads`، `polls`، `permissions`، `messages`، `threads`، `pins`، `search` | مفعّل     |
| roleInfo                                                                                                      | معطل      |
| moderation                                                                                                    | معطل      |
| presence                                                                                                      | معطل      |

## واجهة المستخدم Components v2

يستخدم OpenClaw مكونات Discord v2 لموافقات التنفيذ وعلامات السياق المتقاطع. يمكن لإجراءات رسائل Discord أيضًا قبول `components` لواجهة مستخدم مخصصة (متقدم؛ يتطلب مثيلات مكونات Carbon)، بينما تظل `embeds` القديمة متاحة ولكن لا يُنصح بها.

- يحدد `channels.discord.ui.components.accentColor` لون التمييز المستخدم في حاويات مكونات Discord (hex).
- يتم التعيين لكل حساب باستخدام `channels.discord.accounts.<id>`.ui.components.accentColor\`.
- يتم تجاهل `embeds` عند وجود components v2.

مثال:

```json5
القناة (مثل `#help`) → **Copy Channel ID**
```

## messages

تعرض الرسائل الصوتية في Discord معاينة لشكل الموجة وتتطلب صوت OGG/Opus بالإضافة إلى بيانات وصفية. يقوم OpenClaw بإنشاء شكل الموجة تلقائيًا، لكنه يحتاج إلى توفر `ffmpeg` و `ffprobe` على مضيف البوابة لفحص ملفات الصوت وتحويلها.

المتطلبات والقيود:

- قدّم **مسار ملف محلي** (يتم رفض عناوين URL).
- احذف المحتوى النصي (لا يسمح Discord بإرسال نص + رسالة صوتية في نفس الحمولة).
- يتم قبول أي تنسيق صوتي؛ يقوم OpenClaw بالتحويل إلى OGG/Opus عند الحاجة.

مثال:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## استكشاف الأخطاء وإصلاحها

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - فعّل Message Content Intent
    - فعّل Server Members Intent عند الاعتماد على تحليل المستخدم/العضو
    - أعد تشغيل البوابة بعد تغيير intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - تحقّق من `groupPolicy`
    - تحقّق من قائمة السماح الخاصة بالخوادم ضمن `channels.discord.guilds`
    - إذا كان مخطط `channels` موجودًا في الخادم، فستُسمح فقط القنوات المدرجة
    - تحقّق من سلوك `requireMention` وأنماط الإشارة
    
    فحوصات مفيدة:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    الأسباب الشائعة:

    ```
    `groupPolicy`: يتحكم في التعامل مع قنوات الخوادم (`open|disabled|allowlist`)؛ يتطلب `allowlist` قوائم سماح للقنوات.
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**تدقيق الأذونات** (`channels status --probe`) يتحقق فقط من معرّفات القنوات الرقمية.

    ```
    إذا كنت تستخدم مفاتيح slug، فقد يستمر التطابق أثناء التشغيل في العمل، لكن لا يمكن لأداة الفحص التحقق بالكامل من الأذونات.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **الرسائل المباشرة لا تعمل**: `channels.discord.dm.enabled=false`، أو `channels.discord.dm.policy="disabled"`، أو لم تتم الموافقة عليك بعد (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">تُتجاهل رسائل البوت افتراضيًا؛ عيّن `channels.discord.allowBots=true` للسماح بها (تبقى رسائل البوت نفسه مُصفّاة).

    ```
    تحذير: إذا سمحت بالرد على بوتات أخرى (`channels.discord.allowBots=true`)، فامنع حلقات الرد بين البوتات عبر قوائم السماح `requireMention` و `channels.discord.guilds.*.channels.<id> .users`، و/أو إزالة الحواجز في `AGENTS.md` و `SOUL.md`.
    ```

  
</Accordion>
</AccordionGroup>

## مؤشرات مرجعية للإعدادات

المرجع الأساسي:

- [مرجع الإعدادات - Discord](/gateway/configuration-reference#discord)

حقول Discord عالية الأهمية:

- `presence` (حالة/نشاط البوت، الافتراضي `false`)
- `guilds.<id> .channels.<channel> .allow`: السماح/المنع للقناة عندما `groupPolicy="allowlist"`.
- استخدم `commands.useAccessGroups: false` لتجاوز فحوصات مجموعات الوصول للأوامر.
- الرد/السجل: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- التسليم: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- الوسائط/إعادة المحاولة: `mediaMaxMb`, `retry`
- الإجراءات: `actions.*`
- الحالة: `activity`, `status`, `activityType`, `activityUrl`
- واجهة المستخدم: `ui.components.accentColor`
- الميزات: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## السلامة والتشغيل

- تعامل مع رموز البوت كأسرار (`DISCORD_BOT_TOKEN` مفضل في البيئات الخاضعة للإشراف).
- امنح أقل قدر ممكن من أذونات Discord.
- إذا كانت حالة/نشر الأوامر قديمة، فأعد تشغيل البوابة وأعد التحقق باستخدام `openclaw channels status --probe`.

## ذو صلة

- [الاقتران](/channels/pairing)
- [توجيه القنوات](/channels/channel-routing)
- [استكشاف الأخطاء وإصلاحها](/channels/troubleshooting)
- قائمة الأوامر الكاملة + التهيئة: [Slash commands](/tools/slash-commands)
