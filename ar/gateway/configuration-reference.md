---
title: "مرجع الإعدادات"
description: "مرجع كامل لكل حقل في ~/.openclaw/openclaw.json"
---

# مرجع الإعدادات

جميع الحقول المتاحة في `~/.openclaw/openclaw.json`. للاطلاع على نظرة عامة موجهة للمهام، راجع [Configuration](/gateway/configuration).

تنسيق الإعداد هو **JSON5** (يُسمح بالتعليقات والفواصل اللاحقة). جميع الحقول اختيارية — يستخدم OpenClaw إعدادات افتراضية آمنة عند عدم تحديدها.

---

## القنوات

تبدأ كل قناة تلقائيًا عند وجود قسم الإعداد الخاص بها (ما لم يكن `enabled: false`).

### الوصول عبر الرسائل الخاصة والمجموعات

تدعم جميع القنوات سياسات الرسائل الخاصة وسياسات المجموعات:

| سياسة الرسائل الخاصة                     | السلوك                                                                              |
| ---------------------------------------- | ----------------------------------------------------------------------------------- |
| `pairing` (الافتراضي) | يتلقى المرسلون غير المعروفين رمز اقتران لمرة واحدة؛ يجب على المالك الموافقة         |
| `allowlist`                              | فقط المرسلون المدرجون في `allowFrom` (أو في مخزن السماح المقترن) |
| `open`                                   | السماح بجميع الرسائل الخاصة الواردة (يتطلب `allowFrom: ["*"]`)   |
| `disabled`                               | تجاهل جميع الرسائل الخاصة الواردة                                                   |

| سياسة المجموعات                            | السلوك                                                                         |
| ------------------------------------------ | ------------------------------------------------------------------------------ |
| `allowlist` (الافتراضي) | فقط المجموعات المطابقة لقائمة السماح المُكوَّنة                                |
| `open`                                     | تجاوز قوائم السماح للمجموعات (لا يزال تقييد الإشارة مطبقًا) |
| `disabled`                                 | حظر جميع رسائل المجموعات/الغرف                                                 |

<Note>
يحدد `channels.defaults.groupPolicy` السياسة الافتراضية عندما لا يتم تعيين `groupPolicy` لموفر معيّن.
تنتهي صلاحية رموز الاقتران بعد ساعة واحدة. يتم تحديد طلبات اقتران الرسائل الخاصة المعلقة بحد أقصى **3 لكل قناة**.
لدى Slack/Discord وضع احتياطي خاص: إذا كان قسم الموفر الخاص بهما مفقودًا بالكامل، يمكن أن تُحل سياسة المجموعات أثناء التشغيل إلى `open` (مع تحذير عند بدء التشغيل).
</Note>

### WhatsApp

يعمل WhatsApp من خلال قناة الويب الخاصة بالبوابة (Baileys Web). يبدأ تلقائيًا عند وجود جلسة مرتبطة.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (false in self-chat mode)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
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

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- تستخدم الأوامر الصادرة الحساب `default` افتراضيًا إذا كان موجودًا؛ وإلا فسيتم استخدام أول معرف حساب مُهيأ (بعد الفرز).
- يتم ترحيل مجلد مصادقة Baileys القديم ذي الحساب الواحد بواسطة `openclaw doctor` إلى `whatsapp/default`.
- تجاوزات لكل حساب: `channels.whatsapp.accounts.<id>`.sendReadReceipts`، `channels.whatsapp.accounts.<id>`.dmPolicy`، `channels.whatsapp.accounts.<id>`.allowFrom\`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
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
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- رمز البوت: `channels.telegram.botToken` أو `channels.telegram.tokenFile`، مع استخدام `TELEGRAM_BOT_TOKEN` كخيار احتياطي للحساب الافتراضي.
- `configWrites: false` يمنع عمليات كتابة الإعدادات التي تبدأ من Telegram (ترحيل معرفات supergroup، وأوامر `/config set|unset`).
- تستخدم معاينات البث في Telegram الأمرين `sendMessage` و`editMessageText` (وتعمل في المحادثات الخاصة والجماعية).
- سياسة إعادة المحاولة: راجع [Retry policy](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
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
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
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
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
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

- الرمز: `channels.discord.token`، مع استخدام `DISCORD_BOT_TOKEN` كخيار احتياطي للحساب الافتراضي.
- استخدم `user:<id>` (رسالة خاصة DM) أو `channel:<id>` (قناة ضمن guild) كوجهات للإرسال؛ يتم رفض المعرفات الرقمية المجردة.
- تكون معرفات guild بحروف صغيرة مع استبدال المسافات بـ `-`؛ وتستخدم مفاتيح القنوات الاسم المُحوّل إلى slug (من دون `#`). يُفضّل استخدام معرفات guild.
- يتم تجاهل الرسائل التي ينشئها البوت افتراضيًا. يؤدي `allowBots: true` إلى تمكينها (مع الاستمرار في تصفية رسائل البوت نفسه).
- يقوم `maxLinesPerMessage` (القيمة الافتراضية 17) بتقسيم الرسائل الطويلة حتى إذا كانت أقل من 2000 حرف.
- يحدد `channels.discord.ui.components.accentColor` لون التمييز لحاويات Discord components v2.

**أوضاع إشعارات التفاعل:** `off` (بدون)، `own` (رسائل البوت، الافتراضي)، `all` (جميع الرسائل)، `allowlist` (من `guilds.<id>`.users\` على جميع الرسائل).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
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

- ملف حساب الخدمة بصيغة JSON: مضمّن (`serviceAccount`) أو من خلال ملف (`serviceAccountFile`).
- خيارات البيئة الاحتياطية: `GOOGLE_CHAT_SERVICE_ACCOUNT` أو `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- استخدم `spaces/<spaceId>` أو `users/<userId|email>` كوجهات للإرسال.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
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
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
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

- **وضع Socket** يتطلب كلاً من `botToken` و`appToken` (استخدم `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` كخيار احتياطي عبر متغيرات البيئة للحساب الافتراضي).
- **وضع HTTP** يتطلب `botToken` بالإضافة إلى `signingSecret` (على المستوى الجذري أو لكل حساب).
- `configWrites: false` يمنع عمليات كتابة الإعدادات التي تبدأ من Slack.
- استخدم `user:<id>` (رسالة خاصة DM) أو `channel:<id>` كوجهات للإرسال.

**أوضاع إشعارات التفاعل:** `off`، `own` (الافتراضي)، `all`، `allowlist` (من `reactionAllowlist`).

**عزل جلسات الخيوط:** تكون `thread.historyScope` لكل خيط على حدة (الافتراضي) أو مشتركة عبر القناة. `thread.inheritParent` ينسخ سجل محادثة القناة الأصلية إلى سلاسل الرسائل الجديدة.

| مجموعة الإجراءات       | افتراضي | ملاحظات                           |
| ---------------------- | ------- | --------------------------------- |
| التفاعلات              | مفعّل   | إضافة تفاعل + عرض قائمة التفاعلات |
| الرسائل                | مفعّل   | قراءة/إرسال/تعديل/حذف             |
| التثبيتات              | مفعّل   | تثبيت/إلغاء تثبيت/عرض القائمة     |
| معلومات العضو          | مفعّل   | معلومات العضو                     |
| قائمة الرموز التعبيرية | مفعّل   | قائمة الرموز التعبيرية المخصصة    |

### Mattermost

يأتي Mattermost كمكوّن إضافي: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

أوضاع الدردشة: `oncall` (الرد عند الإشارة @، افتراضي)، `onmessage` (كل رسالة)، `onchar` (الرسائل التي تبدأ ببادئة تشغيل).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**أوضاع إشعارات التفاعل:** `off`، `own` (افتراضي)، `all`، `allowlist` (من `reactionAllowlist`).

### iMessage

يقوم OpenClaw بتشغيل `imsg rpc` (‏JSON-RPC عبر stdio). لا يتطلب خادمًا دائمًا أو منفذًا.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- يتطلب صلاحية الوصول الكامل إلى القرص لقاعدة بيانات الرسائل.
- يُفضَّل استخدام الأهداف بصيغة `chat_id:<id>`. استخدم `imsg chats --limit 20` لعرض قائمة المحادثات.
- يمكن أن يشير `cliPath` إلى مُغلِّف SSH؛ اضبط `remoteHost` لاستخدام SCP في جلب المرفقات.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### حسابات متعددة (جميع القنوات)

شغّل عدة حسابات لكل قناة (لكلٍ منها `accountId` خاص به):

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

- يُستخدم `default` عند عدم تحديد `accountId` (في CLI + التوجيه).
- تنطبق رموز البيئة فقط على حساب **default**.
- تنطبق إعدادات القناة الأساسية على جميع الحسابات ما لم يتم تجاوزها لكل حساب على حدة.
- استخدم `bindings[].match.accountId` لتوجيه كل حساب إلى وكيل مختلف.

### تقييد الإشارة في الدردشة الجماعية

تتطلب الرسائل الجماعية افتراضيًا **إشارة مطلوبة** (إشارة عبر البيانات الوصفية أو أنماط regex). ينطبق على محادثات المجموعات في WhatsApp وTelegram وDiscord وGoogle Chat وiMessage.

**أنواع الإشارات:**

- **إشارات البيانات الوصفية**: إشارات @ الأصلية في المنصة. يتم تجاهلها في وضع الدردشة الذاتية في WhatsApp.
- **أنماط النص**: أنماط Regex في `agents.list[].groupChat.mentionPatterns`. يتم التحقق منها دائمًا.
- يتم فرض تقييد الإشارة فقط عندما يكون الاكتشاف ممكنًا (إشارات أصلية أو وجود نمط واحد على الأقل).

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

يحدد `messages.groupChat.historyLimit` القيمة الافتراضية العامة. `channels.<channel>``.historyLimit` (أو لكل حساب). اضبط القيمة على `0` للتعطيل.

#### حدود سجل الرسائل الخاصة (DM)

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

آلية التحديد: تجاوز لكل DM → الافتراضي لمزود الخدمة → بدون حد (يتم الاحتفاظ بالجميع).

المدعوم: `telegram`، `whatsapp`، `discord`، `slack`، `signal`، `imessage`، `msteams`.

#### وضع الدردشة الذاتية

قم بتضمين رقمك الخاص في `allowFrom` لتفعيل وضع الدردشة الذاتية (يتجاهل إشارات @ الأصلية ويستجيب فقط لأنماط النص):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### الأوامر (معالجة أوامر الدردشة)

```json5
{
  commands: {
    native: "auto", // تسجيل الأوامر الأصلية عند الدعم
    text: true, // تحليل أوامر /commands في رسائل الدردشة
    bash: false, // السماح بـ ! (الاسم البديل: /bash)
    bashForegroundMs: 2000,
    config: false, // السماح بـ /config
    debug: false, // السماح بـ /debug
    restart: false, // السماح بـ /restart + أداة إعادة تشغيل gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- يجب أن تكون الأوامر النصية رسائل **مستقلة** تبدأ بـ `/`.
- `native: "auto"` يفعّل الأوامر الأصلية لـ Discord/Telegram ويُبقي Slack معطلاً.
- يمكن التجاوز لكل قناة: `channels.discord.commands.native` (قيمة منطقية أو `"auto"`). `false` يمسح الأوامر التي تم تسجيلها سابقًا.
- `channels.telegram.customCommands` يضيف عناصر إضافية إلى قائمة بوت Telegram.
- `bash: true` يفعّل `! <cmd>` for host shell. يتطلب `tools.elevated.enabled` وأن يكون المرسل ضمن `tools.elevated.allowFrom.<channel>`.\`.
- `config: true` يفعّل `/config` (قراءة/كتابة `openclaw.json`).
- `channels.<provider>``.configWrites` يتحكم في تعديلات الإعدادات لكل قناة (الافتراضي: true).
- `allowFrom` خاص بكل مزود. عند تعيينه، يكون **المصدر الوحيد** للتخويل (يتم تجاهل قوائم السماح/الاقتران الخاصة بالقناة و`useAccessGroups`).
- `useAccessGroups: false` يسمح للأوامر بتجاوز سياسات مجموعات الوصول عندما لا يتم تعيين `allowFrom`.

</Accordion>

---

## الإعدادات الافتراضية للوكيل

### `agents.defaults.workspace`

القيمة الافتراضية: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

جذر المستودع الاختياري الذي يظهر في سطر Runtime ضمن موجه النظام. إذا لم يتم تعيينه، يقوم OpenClaw باكتشافه تلقائيًا من خلال الانتقال تصاعديًا بدءًا من مساحة العمل.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

يعطّل الإنشاء التلقائي لملفات تهيئة مساحة العمل (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

الحد الأقصى لعدد الأحرف لكل ملف تهيئة في مساحة العمل قبل الاقتطاع. القيمة الافتراضية: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

الحد الأقصى لإجمالي عدد الأحرف المُدرجة عبر جميع ملفات تهيئة مساحة العمل. القيمة الافتراضية: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

المنطقة الزمنية لسياق موجه النظام (وليست للطوابع الزمنية للرسائل). يتم الرجوع إلى المنطقة الزمنية للمضيف عند عدم التعيين.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

تنسيق الوقت في موجه النظام. القيمة الافتراضية: `auto` (تفضيل نظام التشغيل).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
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
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: بالتنسيق `provider/model` (مثال: `anthropic/claude-opus-4-6`). إذا حذفت اسم المزوّد، سيفترض OpenClaw القيمة `anthropic` (مهمل).
- `models`: كتالوج النماذج المُكوَّن وقائمة السماح لأمر `/model`. يمكن أن يتضمن كل إدخال `alias` (اختصار) و`params` (خاصة بالمزوّد: `temperature`, `maxTokens`).
- `imageModel`: يُستخدم فقط إذا كان النموذج الأساسي لا يدعم إدخال الصور.
- `maxConcurrent`: الحد الأقصى لتشغيل الوكلاء بالتوازي عبر الجلسات (تظل كل جلسة متسلسلة). القيمة الافتراضية: 1.

**اختصارات الأسماء المستعارة المدمجة** (تنطبق فقط عندما يكون النموذج ضمن `agents.defaults.models`):

| الاسم المستعار | النموذج                         |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

الأسماء المستعارة التي قمت بتكوينها تتجاوز دائمًا الإعدادات الافتراضية.

تقوم نماذج Z.AI GLM-4.x بتفعيل وضع التفكير تلقائيًا ما لم تقم بتعيين `--thinking off` أو تعريف `agents.defaults.models["zai/<model>"].params.thinking` بنفسك.

### `agents.defaults.cliBackends`

واجهات CLI اختيارية للتشغيل الاحتياطي النصي فقط (بدون استدعاءات أدوات). مفيدة كخيار احتياطي عند فشل مزودي API.

```json5
{
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

- واجهات CLI تعتمد على النص أولًا؛ ويتم دائمًا تعطيل الأدوات.
- يتم دعم الجلسات عند تعيين `sessionArg`.
- يتم دعم تمرير الصور عندما يقبل `imageArg` مسارات الملفات.

### `agents.defaults.heartbeat`

تشغيل heartbeat بشكل دوري.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m يعطّل الميزة
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: سلسلة مدة زمنية (ms/s/m/h). القيمة الافتراضية: `30m`.
- لكل وكيل: قم بتعيين `agents.list[].heartbeat`. عند قيام أي وكيل بتعريف `heartbeat`، **فقط هؤلاء الوكلاء** سيقومون بتشغيل heartbeat.
- تشغيل heartbeat ينفّذ دورة وكيل كاملة — الفواصل الأقصر تستهلك عددًا أكبر من الرموز (tokens).

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
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

- `mode`: `default` أو `safeguard` (تلخيص مُجزأ للسجلات الطويلة). انظر [Compaction](/concepts/compaction).
- `memoryFlush`: دورة وكيل صامتة قبل الضغط التلقائي لتخزين الذكريات الدائمة. يتم تخطيها عندما تكون مساحة العمل للقراءة فقط.

### `agents.defaults.contextPruning`

يقوم بإزالة **نتائج الأدوات القديمة** من السياق المخزَّن في الذاكرة قبل إرسالها إلى LLM. لا يقوم **بتعديل** سجل الجلسة المخزَّن على القرص.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // مدة زمنية (ms/s/m/h)، الوحدة الافتراضية: الدقائق
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` يفعّل عمليات التقليم.
- `ttl` يتحكم في عدد المرات التي يمكن فيها تشغيل التقليم مرة أخرى (بعد آخر استخدام لذاكرة التخزين المؤقت).
- يقوم Pruning بقصّ نتائج الأدوات الكبيرة أولاً بشكلٍ مرن، ثم يزيل النتائج الأقدم بشكلٍ صارم عند الحاجة.

**Soft-trim** يحتفظ بالبداية + النهاية ويُدرج `...` في المنتصف.

**Hard-clear** يستبدل نتيجة الأداة بالكامل بعنصر نائب.

ملاحظات:

- لا يتم قصّ/إزالة كتل الصور مطلقًا.
- تعتمد النِّسب على عدد الأحرف (تقريبية)، وليست على عدد الرموز (tokens) بدقة.
- إذا كان عدد رسائل المساعد أقل من `keepLastAssistants`، يتم تخطي عملية التقليم.

</Accordion>

راجع [Session Pruning](/concepts/session-pruning) لمزيد من التفاصيل حول السلوك.

### البثّ على شكل كتل

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- تتطلب القنوات غير Telegram تعيين `*.blockStreaming: true` صراحةً لتمكين الردود على شكل كتل.
- تجاوزات القنوات: `channels.<channel>`.blockStreamingCoalesce`(وكذلك المتغيرات الخاصة بكل حساب). القيم الافتراضية في Signal/Slack/Discord/Google Chat هي`minChars: 1500\`.
- `humanDelay`: توقّف عشوائي بين الردود على شكل كتل. `natural` = من 800 إلى 2500 مللي ثانية. تجاوز خاص بكل وكيل: `agents.list[].humanDelay`.

راجع [Streaming](/concepts/streaming) لمزيد من التفاصيل حول السلوك وتقسيم الكتل.

### مؤشرات الكتابة

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- القيم الافتراضية: `instant` للمحادثات المباشرة/عند الإشارة، و`message` لمحادثات المجموعات بدون إشارة.
- تجاوزات لكل جلسة: `session.typingMode`، `session.typingIntervalSeconds`.

راجع [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

إعداد **Docker sandboxing** اختياري للوكيل المضمَّن. راجع [Sandboxing](/gateway/sandboxing) للحصول على الدليل الكامل.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
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
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
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

<Accordion title="Sandbox details">

**الوصول إلى مساحة العمل:**

- `none`: مساحة عمل sandbox لكل نطاق ضمن `~/.openclaw/sandboxes`
- `ro`: مساحة عمل sandbox في `/workspace`، ويتم ربط مساحة عمل الوكيل بوضع القراءة فقط في `/agent`
- `rw`: يتم ربط مساحة عمل الوكيل للقراءة/الكتابة في `/workspace`

**النطاق:**

- `session`: حاوية + مساحة عمل لكل جلسة
- `agent`: حاوية واحدة + مساحة عمل لكل وكيل (افتراضي)
- `shared`: حاوية ومساحة عمل مشتركة (بدون عزل بين الجلسات)

يتم تشغيل **`setupCommand`** مرة واحدة بعد إنشاء الحاوية (عبر `sh -lc`). يتطلب اتصال شبكة خارجي (egress)، وجذرًا قابلاً للكتابة، ومستخدم root.

**تكون الحاويات افتراضيًا على `network: "none"`** — اضبطها على `"bridge"` إذا كان الوكيل يحتاج إلى وصول خارجي.

يتم وضع **المرفقات الواردة** في `media/inbound/*` داخل مساحة العمل النشطة.

يقوم **`docker.binds`** بربط أدلة إضافية من المضيف؛ ويتم دمج عمليات الربط العامة وتلك الخاصة بكل وكيل.

**متصفح معزول** (`sandbox.browser.enabled`): Chromium + CDP داخل حاوية. يتم حقن رابط noVNC في موجّه النظام. لا يتطلب تفعيل `browser.enabled` في الإعدادات الرئيسية.

- `allowHostControl: false` (الافتراضي) يمنع الجلسات المعزولة من استهداف متصفح الجهاز المضيف.
- `sandbox.browser.binds` يقوم بتركيب أدلة إضافية من الجهاز المضيف داخل حاوية متصفح العزل فقط. عند تعيينه (بما في ذلك `[]`)، فإنه يستبدل `docker.binds` لحاوية المتصفح.

</Accordion>

بناء الصور:

```bash
scripts/sandbox-setup.sh           # صورة العزل الرئيسية
scripts/sandbox-browser-setup.sh   # صورة المتصفح الاختيارية
```

### `agents.list` (تجاوز الإعدادات لكل وكيل)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // أو { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: معرّف وكيل ثابت (مطلوب).
- `default`: عند تعيين عدة وكلاء كافتراضيين، يتم اختيار الأول (مع تسجيل تحذير). إذا لم يتم تعيين أي وكيل كافتراضي، فسيكون أول عنصر في القائمة هو الافتراضي.
- `model`: الصيغة النصية تتجاوز `primary` فقط؛ وصيغة الكائن `{ primary, fallbacks }` تتجاوز كليهما (`[]` يعطّل البدائل العامة).
- `identity.avatar`: مسار نسبي إلى مساحة العمل، أو رابط `http(s)`، أو `data:` URI.
- `identity` يشتق القيم الافتراضية: `ackReaction` من `emoji`، و`mentionPatterns` من `name`/`emoji`.
- `subagents.allowAgents`: قائمة سماح لمعرّفات الوكلاء لـ `sessions_spawn` (`["*"]` = أي وكيل؛ الافتراضي: نفس الوكيل فقط).

---

## توجيه متعدد الوكلاء

تشغيل عدة وكلاء معزولين داخل Gateway واحد. انظر [Multi-Agent](/concepts/multi-agent).

```json5
{
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
}
```

### حقول مطابقة الربط

- `match.channel` (مطلوب)
- `match.accountId` (اختياري؛ `*` = أي حساب؛ عند الحذف = الحساب الافتراضي)
- `match.peer` (اختياري؛ `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (اختياري؛ خاص بالقناة)

**ترتيب مطابقة حتمي:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (مطابقة تامة، بدون peer/guild/team)
5. `match.accountId: "*"` (على مستوى القناة)
6. الوكيل الافتراضي

ضمن كل مستوى، يتم اختيار أول عنصر مطابق في `bindings`.

### ملفات تعريف الوصول لكل وكيل

<Accordion title="Full access (no sandbox)">

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

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
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

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
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

</Accordion>

انظر [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) لمعرفة تفاصيل الأولوية.

---

## الجلسة

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // حقل قديم (يستخدم وقت التشغيل دائمًا "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: كيفية تجميع الرسائل الخاصة (DMs).
  - `main`: جميع الرسائل الخاصة تشترك في الجلسة الرئيسية.
  - `per-peer`: عزل حسب معرّف المرسل عبر القنوات.
  - `per-channel-peer`: عزل لكل قناة + مرسل (موصى به لصناديق الوارد متعددة المستخدمين).
  - `per-account-channel-peer`: عزل لكل حساب + قناة + مرسل (موصى به للحسابات المتعددة).
- **`identityLinks`**: ربط المعرّفات الأساسية بالأقران ذوي بادئة المزوّد لمشاركة الجلسة عبر القنوات.
- **`reset`**: سياسة إعادة التعيين الأساسية. `daily` يعيد التعيين عند `atHour` حسب التوقيت المحلي؛ و`idle` يعيد التعيين بعد `idleMinutes`. عند تكوين الاثنين معًا، يُعتمد أيهما ينتهي أولًا.
- **`resetByType`**: تجاوزات حسب النوع (`direct`، `group`، `thread`). يُقبل `dm` القديم كاسم بديل لـ `direct`.
- **`mainKey`**: حقل قديم. يستخدم وقت التشغيل الآن دائمًا `"main"` لحاوية الدردشة المباشرة الرئيسية.
- **`sendPolicy`**: المطابقة حسب `channel`، أو `chatType` (`direct|group|channel`، مع الاسم البديل القديم `dm`)، أو `keyPrefix`، أو `rawKeyPrefix`. أول قاعدة `deny` يتم تطبيقها.
- **`maintenance`**: `warn` يحذّر الجلسة النشطة عند الإزالة؛ و`enforce` يطبّق التقليم والتدوير.

</Accordion>

---

## الرسائل

```json5
{
  messages: {
    responsePrefix: "🦞", // أو "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 يعطّل الميزة
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### بادئة الاستجابة

تجاوزات لكل قناة/حساب: `channels.<channel>.responsePrefix`، `channels.<channel>.accounts.<id>.responsePrefix`.

آلية التحديد (الأكثر تحديدًا يفوز): الحساب → القناة → العام. `""` يعطّل ويوقف التسلسل. `"auto"` يستمد `[{identity.name}]`.

**متغيرات القالب:**

| المتغير           | الوصف                  | مثال                                         |
| ----------------- | ---------------------- | -------------------------------------------- |
| `{model}`         | الاسم المختصر للنموذج  | `claude-opus-4-6`                            |
| `{modelFull}`     | المعرّف الكامل للنموذج | `anthropic/claude-opus-4-6`                  |
| `{provider}`      | اسم المزوّد            | `anthropic`                                  |
| `{thinkingLevel}` | مستوى التفكير الحالي   | `high`، `low`، `off`                         |
| `{identity.name}` | اسم هوية الوكيل        | (نفس القيمة مثل `"auto"`) |

المتغيرات غير حساسة لحالة الأحرف. `{think}` هو اسم بديل لـ `{thinkingLevel}`.

### تفاعل التأكيد (Ack)

- القيمة الافتراضية هي `identity.emoji` للوكيل النشط، وإلا `"👀"`. عيّن القيمة `""` للتعطيل.
- تجاوزات لكل قناة: `channels.<channel>
  .ackReaction`، `channels.<channel>
  .accounts.<id>
  .ackReaction`.
- ترتيب تحديد القيمة: الحساب → القناة → `messages.ackReaction` → هوية بديلة.
- النطاق: `group-mentions` (افتراضي)، `group-all`، `direct`، `all`.
- `removeAckAfterReply`: يزيل تأكيد الاستلام بعد الرد (Slack/Discord/Telegram/Google Chat فقط).

### إزالة الارتداد للرسائل الواردة (Inbound debounce)

يجمع الرسائل النصية السريعة فقط من نفس المرسل في دور واحد للوكيل. يتم تمرير الوسائط/المرفقات فورًا. أوامر التحكم تتجاوز آلية إزالة الارتداد.

### TTS (تحويل النص إلى كلام)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
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

- `auto` يتحكم في التفعيل التلقائي لـ TTS. `/tts off|always|inbound|tagged` يتجاوز الإعداد لكل جلسة.
- `summaryModel` يتجاوز `agents.defaults.model.primary` للملخص التلقائي.
- تعود مفاتيح API افتراضيًا إلى `ELEVENLABS_API_KEY`/`XI_API_KEY` و `OPENAI_API_KEY`.

---

## Talk

الإعدادات الافتراضية لوضع Talk (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- تعود معرّفات الصوت افتراضيًا إلى `ELEVENLABS_VOICE_ID` أو `SAG_VOICE_ID`.
- `apiKey` يعود افتراضيًا إلى `ELEVENLABS_API_KEY`.
- `voiceAliases` يتيح لتوجيهات Talk استخدام أسماء ودّية.

---

## الأدوات

### ملفات تعريف الأدوات

`tools.profile` يحدد قائمة سماح أساسية قبل `tools.allow`/`tools.deny`:

| الملف الشخصي | يتضمن                                                                                     |
| ------------ | ----------------------------------------------------------------------------------------- |
| `minimal`    | `session_status` only                                                                     |
| `coding`     | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging`  | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`       | بدون قيود (كما لو لم يتم التعيين)                                      |

### مجموعات الأدوات

| المجموعة           | الأدوات                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` مقبول كاسم مستعار لـ `exec`)                |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | جميع الأدوات المدمجة (باستثناء إضافات المزوّدين)                      |

### `tools.allow` / `tools.deny`

سياسة السماح/المنع العامة للأدوات (المنع له الأولوية). غير حساسة لحالة الأحرف، وتدعم أحرف البدل `*`. يتم تطبيقها حتى عند تعطيل بيئة Docker المعزولة.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

تقييد إضافي للأدوات لمزوّدين أو نماذج محددة. الترتيب: الملف التعريفي الأساسي → ملف المزوّد → السماح/المنع.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

يتحكم في صلاحية تنفيذ الأوامر المرتفعة (على المضيف):

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- يمكن للتخصيص لكل وكيل (`agents.list[].tools.elevated`) أن يفرض قيودًا إضافية فقط.
- `/elevated on|off|ask|full` يحفظ الحالة لكل جلسة؛ أما التوجيهات المضمنة فتُطبَّق على رسالة واحدة فقط.
- تنفيذ `exec` المرتفع يعمل على المضيف ويتجاوز العزل (sandboxing).

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

يُهيّئ فهم الوسائط الواردة (صور/صوت/فيديو):

```json5
{
  tools: {
    media: {
      concurrency: 2,
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

<Accordion title="Media model entry fields">

**إدخال Provider** (`type: "provider"` أو عند حذفه):

- `provider`: معرّف مزود API (`openai`، `anthropic`، `google`/`gemini`، `groq`، إلخ.)
- `model`: تجاوز معرّف النموذج
- `profile` / `preferredProfile`: اختيار ملف تعريف المصادقة

**إدخال CLI** (`type: "cli"`):

- `command`: الملف التنفيذي المطلوب تشغيله
- `args`: معاملات مُقوالَبة (تدعم `{{MediaPath}}`، `{{Prompt}}`، `{{MaxChars}}`، إلخ.)

**الحقول المشتركة:**

- `capabilities`: قائمة اختيارية (`image`، `audio`، `video`). القيم الافتراضية: `openai`/`anthropic`/`minimax` → صورة، `google` → صورة+صوت+فيديو، `groq` → صوت.
- `prompt`، `maxChars`، `maxBytes`، `timeoutSeconds`، `language`: تجاوزات لكل إدخال.
- عند الفشل يتم الرجوع إلى الإدخال التالي.

تتبع مصادقة Provider الترتيب القياسي: ملفات تعريف المصادقة → متغيرات البيئة → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: النموذج الافتراضي للوكلاء الفرعيين الذين يتم إنشاؤهم. إذا تم حذفه، يرث الوكلاء الفرعيون نموذج المُستدعي.
- سياسة الأدوات لكل وكيل فرعي: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## موفرو الخدمة المخصصون وعناوين URL الأساسية

يستخدم OpenClaw كتالوج نماذج pi-coding-agent. أضف موفري خدمة مخصصين عبر `models.providers` في الإعدادات أو `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
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

- استخدم `authHeader: true` مع `headers` لاحتياجات المصادقة المخصصة.
- يمكن تجاوز مسار جذر إعدادات الوكيل باستخدام `OPENCLAW_AGENT_DIR` (أو `PI_CODING_AGENT_DIR`).

### أمثلة على Provider

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

استخدم `cerebras/zai-glm-4.7` لـ Cerebras؛ و `zai/glm-4.7` لـ Z.AI المباشر.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

قم بتعيين `OPENCODE_API_KEY` (أو `OPENCODE_ZEN_API_KEY`). اختصار: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

قم بتعيين `ZAI_API_KEY`. يتم قبول `z.ai/*` و `z-ai/*` كأسماء مستعارة. اختصار: `openclaw onboard --auth-choice zai-api-key`.

- نقطة النهاية العامة: `https://api.z.ai/api/paas/v4`
- نقطة نهاية البرمجة (الافتراضية): `https://api.z.ai/api/coding/paas/v4`
- بالنسبة لنقطة النهاية العامة، عرّف مزودًا مخصصًا مع تجاوز base URL.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
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

بالنسبة لنقطة نهاية الصين: `baseUrl: "https://api.moonshot.cn/v1"` أو `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

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

متوافق مع Anthropic، مزود مدمج. اختصار: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

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

يجب أن لا يتضمن Base URL المسار `/v1` (يقوم عميل Anthropic بإضافته). اختصار: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
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

قم بتعيين `MINIMAX_API_KEY`. اختصار: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

راجع [Local Models](/gateway/local-models). باختصار: شغّل MiniMax M2.1 عبر LM Studio Responses API على عتاد قوي؛ واحتفظ بالنماذج المستضافة مدمجة كخيار احتياطي.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: قائمة سماح اختيارية للمهارات المضمنة فقط (لا تتأثر المهارات المُدارة/الخاصة بمساحة العمل).
- `entries.<skillKey>`.enabled: false\` يعطّل المهارة حتى لو كانت مضمنة/مثبتة.
- `entries.<skillKey>`.apiKey\`: خيار تسهيلي للمهارات التي تعرّف متغير بيئة أساسيًا.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- يتم التحميل من `~/.openclaw/extensions` و `<workspace>/.openclaw/extensions` بالإضافة إلى `plugins.load.paths`.
- **تتطلب تغييرات الإعدادات إعادة تشغيل Gateway.**
- `allow`: قائمة سماح اختيارية (يتم تحميل الـ plugins المدرجة فقط). `deny` له الأولوية.

راجع [Plugins](/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` يعطّل `act:evaluate` و `wait --fn`.
- ملفات التعريف البعيدة للاتصال فقط (بدء/إيقاف/إعادة تعيين معطّلة).
- ترتيب الاكتشاف التلقائي: المتصفح الافتراضي إذا كان قائمًا على Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- خدمة التحكم: على loopback فقط (المنفذ مشتق من `gateway.port`، الافتراضي `18791`).

---

## واجهة المستخدم

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: لون التمييز لعناصر واجهة التطبيق الأصلية (لون فقاعة وضع التحدث، إلخ).
- `assistant`: تجاوز هوية واجهة التحكم. يعود إلى هوية الوكيل النشط.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: ‏`local` (تشغيل gateway) أو ‏`remote` (الاتصال بـ gateway بعيد). يرفض Gateway البدء ما لم يكن الوضع `local`.
- `port`: منفذ واحد متعدد الاستخدامات لكل من WS و HTTP. الأولوية: ‏`--port` > ‏`OPENCLAW_GATEWAY_PORT` > ‏`gateway.port` > ‏`18789`.
- `bind`: ‏`auto`، ‏`loopback` (افتراضي)، ‏`lan` (`0.0.0.0`)، ‏`tailnet` (عنوان Tailscale IP فقط)، أو ‏`custom`.
- **المصادقة**: مطلوبة افتراضيًا. يتطلب الربط بغير loopback رمزًا أو كلمة مرور مشتركة. يقوم معالج الإعداد الأولي بإنشاء رمز افتراضيًا.
- `auth.mode: "trusted-proxy"`: تفويض المصادقة إلى وكيل عكسي مدرك للهوية والثقة في ترويسات الهوية من `gateway.trustedProxies` (راجع [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: عند تعيينها إلى `true`، تُعتبر ترويسات هوية Tailscale Serve كافية للمصادقة (يتم التحقق عبر `tailscale whois`). القيمة الافتراضية `true` عندما يكون `tailscale.mode = "serve"`.
- `auth.rateLimit`: محدد اختياري لمحاولات المصادقة الفاشلة. يُطبق لكل عنوان IP عميل ولكل نطاق مصادقة (يتم تتبع shared-secret و device-token بشكل مستقل). المحاولات المحظورة تُرجع `429` مع `Retry-After`.
  - القيمة الافتراضية لـ `auth.rateLimit.exemptLoopback` هي `true`؛ عيّنها إلى `false` إذا كنت تريد عمدًا تطبيق تحديد المعدل على حركة localhost أيضًا (لبيئات الاختبار أو إعدادات الوكيل الصارمة).
- `tailscale.mode`: ‏`serve` (ضمن tailnet فقط، ربط loopback) أو ‏`funnel` (عام، يتطلب مصادقة).
- `remote.transport`: ‏`ssh` (افتراضي) أو ‏`direct` (ws/wss). بالنسبة إلى `direct`، يجب أن يكون `remote.url` بصيغة `ws://` أو `wss://`.
- `gateway.remote.token` مخصص لاستدعاءات CLI البعيدة فقط؛ ولا يفعّل مصادقة gateway المحلية.
- `trustedProxies`: عناوين IP للوكلاء العكسيين الذين ينهون TLS. قم بإدراج الوكلاء الذين تتحكم بهم فقط.
- `gateway.tools.deny`: أسماء أدوات إضافية محظورة لطلب HTTP ‏`POST /tools/invoke` (توسّع قائمة الحظر الافتراضية).
- `gateway.tools.allow`: إزالة أسماء أدوات من قائمة الحظر الافتراضية لـ HTTP.

</Accordion>

### نقاط نهاية متوافقة مع OpenAI

- Chat Completions: معطّلة افتراضيًا. قم بتمكينها باستخدام `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: ‏`gateway.http.endpoints.responses.enabled`.
- تعزيز أمان إدخال عناوين URL في Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### عزل متعدد المثيلات

تشغيل عدة Gateways على نفس المضيف باستخدام منافذ وأدلة حالة فريدة:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

خيارات ملائمة: `--dev` (يستخدم `~/.openclaw-dev` + المنفذ `19001`)، `--profile <name>` (يستخدم `~/.openclaw-<name>`).

راجع [Multiple Gateways](/gateway/multiple-gateways).

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
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

المصادقة: `Authorization: Bearer <token>` أو `x-openclaw-token: <token>`.

**نقاط النهاية:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - يتم قبول `sessionKey` من حمولة الطلب فقط عندما تكون `hooks.allowRequestSessionKey=true` (الافتراضي: `false`).
- `POST /hooks/<name>` → يتم حله عبر `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` يطابق المسار الفرعي بعد `/hooks` (مثال: `/hooks/gmail` → `gmail`).
- `match.source` يطابق حقلاً في الحمولة للمسارات العامة.
- القوالب مثل `{{messages[0].subject}}` تقرأ من الحمولة.
- `transform` يمكن أن يشير إلى وحدة JS/TS تُرجع إجراء hook.
  - `transform.module` يجب أن يكون مسارًا نسبيًا ويبقى ضمن `hooks.transformsDir` (يتم رفض المسارات المطلقة ومحاولات الانتقال بين الأدلة).
- `agentId` يوجّه إلى وكيل محدد؛ المعرّفات غير المعروفة تعود إلى الافتراضي.
- `allowedAgentIds`: يقيّد التوجيه الصريح (`*` أو عند الحذف = السماح للجميع، `[]` = منع الجميع).
- `defaultSessionKey`: مفتاح جلسة ثابت اختياري لتشغيل وكيل hook دون `sessionKey` صريح.
- `allowRequestSessionKey`: السماح لمستدعِي `/hooks/agent` بتعيين `sessionKey` (الافتراضي: `false`).
- `allowedSessionKeyPrefixes`: قائمة سماح اختيارية للبادئات لقيم `sessionKey` الصريحة (الطلب + التعيين)، مثال: `["hook:"]`.
- `deliver: true` يرسل الرد النهائي إلى قناة؛ القيمة الافتراضية لـ `channel` هي `last`.
- `model` يتجاوز LLM لهذا التشغيل الخاص بـ hook (يجب أن يكون مسموحًا إذا تم تعيين كتالوج للنماذج).

</Accordion>

### تكامل Gmail

```json5
{
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
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- يقوم Gateway بتشغيل `gog gmail watch serve` تلقائيًا عند الإقلاع عند تكوينه. قم بتعيين `OPENCLAW_SKIP_GMAIL_WATCHER=1` لتعطيله.
- لا تقم بتشغيل `gog gmail watch serve` منفصل بالتزامن مع Gateway.

---

## مضيف Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // أو OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- يقدّم ملفات HTML/CSS/JS القابلة للتعديل من قبل الوكيل وواجهة A2UI عبر HTTP ضمن منفذ Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- محلي فقط: أبقِ `gateway.bind: "loopback"` (الافتراضي).
- عند الربط بغير loopback: تتطلب مسارات canvas مصادقة Gateway (token/password/trusted-proxy)، كما هو الحال مع بقية واجهات HTTP الخاصة بـ Gateway.
- عادةً لا ترسل Node WebViews ترويسات المصادقة؛ بعد إقران العقدة واتصالها، يسمح Gateway بالرجوع إلى عنوان IP خاص كخيار احتياطي حتى تتمكن العقدة من تحميل canvas/A2UI دون تسريب الأسرار في عناوين URL.
- يحقن عميل التحديث الفوري (live-reload) في ملفات HTML المقدَّمة.
- ينشئ تلقائيًا ملف `index.html` ابتدائيًا عند عدم وجود محتوى.
- يقدّم أيضًا A2UI على المسار `/__openclaw__/a2ui/`.
- تتطلب التغييرات إعادة تشغيل Gateway.
- عطّل التحديث الفوري للمجلدات الكبيرة أو عند ظهور أخطاء `EMFILE`.

---

## الاكتشاف

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (الافتراضي): يحذف `cliPath` و `sshPort` من سجلات TXT.
- `full`: يتضمن `cliPath` و `sshPort`.
- اسم المضيف الافتراضي هو `openclaw`. يمكن التجاوز باستخدام `OPENCLAW_MDNS_HOSTNAME`.

### النطاق الواسع (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

يكتب منطقة DNS-SD أحادي الإرسال ضمن `~/.openclaw/dns/`. للاكتشاف عبر الشبكات، استخدمه مع خادم DNS (يُوصى بـ CoreDNS) بالإضافة إلى Tailscale split DNS.

الإعداد: `openclaw dns setup --apply`.

---

## البيئة

### `env` (متغيرات البيئة المضمّنة)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- يتم تطبيق متغيرات البيئة المضمّنة فقط إذا كانت غير موجودة في بيئة العملية.
- ملفات `.env`: ملف `.env` في CWD و `~/.openclaw/.env` (ولا يقوم أيٌّ منهما بتجاوز المتغيرات الموجودة).
- `shellEnv`: يستورد المفاتيح المتوقعة غير الموجودة من ملف تعريف shell الخاص بتسجيل الدخول.
- راجع [Environment](/help/environment) للاطلاع على أولوية التحميل الكاملة.

### استبدال متغيرات البيئة

يمكنك الإشارة إلى متغيرات البيئة في أي سلسلة إعداد باستخدام `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- تتم مطابقة الأسماء المكتوبة بأحرف كبيرة فقط: `[A-Z_][A-Z0-9_]*`.
- تؤدي المتغيرات المفقودة أو الفارغة إلى حدوث خطأ عند تحميل الإعداد.
- استخدم `$${VAR}` للهروب والحصول على `${VAR}` حرفيًا.
- يعمل مع `$include`.

---

## تخزين المصادقة

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

- يتم تخزين ملفات تعريف المصادقة لكل وكيل في `<agentDir>/auth-profiles.json`.
- يتم استيراد OAuth القديم من `~/.openclaw/credentials/oauth.json`.
- راجع [OAuth](/concepts/oauth).

---

## التسجيل

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- ملف السجل الافتراضي: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- قم بتعيين `logging.file` لمسار ثابت.
- يرتفع `consoleLevel` إلى `debug` عند استخدام `--verbose`.

---

## المعالج

البيانات الوصفية التي تكتبها معالجات CLI (`onboard`، `configure`، `doctor`):

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

---

## الهوية

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

تتم كتابته بواسطة مساعد الإعداد على macOS. يستمد القيم الافتراضية:

- `messages.ackReaction` من `identity.emoji` (يعود إلى 👀 في حال عدم توفره)
- `mentionPatterns` من `identity.name`/`identity.emoji`
- يقبل `avatar`: مسارًا نسبيًا لمساحة العمل، أو رابط `http(s)`، أو معرّف URI من نوع `data:`

---

## Bridge (قديم، تمت إزالته)

لم تعد الإصدارات الحالية تتضمن TCP bridge. تتصل العقد عبر Gateway WebSocket. لم تعد مفاتيح `bridge.*` جزءًا من مخطط الإعدادات (يفشل التحقق حتى تتم إزالتها؛ يمكن للأمر `openclaw doctor --fix` إزالة المفاتيح غير المعروفة).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // سلسلة مدة زمنية أو false
  },
}
```

- `sessionRetention`: المدة التي يتم خلالها الاحتفاظ بجلسات Cron المكتملة قبل حذفها. القيمة الافتراضية: `24h`.

راجع [مهام Cron](/automation/cron-jobs).

---

## متغيرات قالب نموذج الوسائط

عناصر القالب النائبة التي يتم توسيعها في `tools.media.*.models[].args`:

| المتغير            | الوصف                                                              |
| ------------------ | ------------------------------------------------------------------ |
| `{{Body}}`         | نص الرسالة الواردة بالكامل                                         |
| `{{RawBody}}`      | النص الخام (من دون سجل المحادثة/أغلفة المرسل)   |
| `{{BodyStripped}}` | النص بعد إزالة الإشارات في المجموعات                               |
| `{{From}}`         | معرّف المرسل                                                       |
| `{{To}}`           | معرّف الوجهة                                                       |
| `{{MessageSid}}`   | معرّف رسالة القناة                                                 |
| `{{SessionId}}`    | معرّف UUID للجلسة الحالية                                          |
| `{{IsNewSession}}` | القيمة "true" عند إنشاء جلسة جديدة                                 |
| `{{MediaUrl}}`     | رابط وهمي (pseudo-URL) للوسائط الواردة          |
| `{{MediaPath}}`    | المسار المحلي للوسائط                                              |
| `{{MediaType}}`    | نوع الوسائط (image/audio/document/…)            |
| `{{Transcript}}`   | النص المُفرَّغ من الصوت                                            |
| `{{Prompt}}`       | موجّه الوسائط المُعالج لإدخالات CLI                                |
| `{{MaxChars}}`     | الحد الأقصى لعدد أحرف المخرجات المُعالج لإدخالات CLI               |
| `{{ChatType}}`     | "direct" أو "group"                                                |
| `{{GroupSubject}}` | عنوان المجموعة (بأفضل جهد ممكن)                 |
| `{{GroupMembers}}` | معاينة أعضاء المجموعة (بأفضل جهد ممكن)          |
| `{{SenderName}}`   | اسم عرض المرسل (بأفضل جهد ممكن)                 |
| `{{SenderE164}}`   | رقم هاتف المرسل (بأفضل جهد ممكن)                |
| `{{Provider}}`     | مؤشر المزوّد (whatsapp، telegram، discord، إلخ) |

---

## يتضمن الإعداد (`$include`)

تقسيم الإعداد إلى عدة ملفات:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**سلوك الدمج:**

- ملف واحد: يستبدل الكائن الذي يحتويه.
- مصفوفة ملفات: يتم دمجها بعمق حسب الترتيب (اللاحق يتجاوز السابق).
- المفاتيح الشقيقة: يتم دمجها بعد الـ includes (وتتجاوز القيم المضمّنة).
- تضمينات متداخلة: حتى 10 مستويات عمق.
- المسارات: نسبية (بالنسبة للملف الذي يتضمنها)، أو مطلقة، أو مراجع إلى المجلد الأب `../`.
- الأخطاء: رسائل واضحة للملفات المفقودة، وأخطاء التحليل، وحالات التضمين الدائري.

---

_ذو صلة: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
