---
title: "کنفیگریشن ریفرنس"
description: "~/.openclaw/openclaw.json کے لیے مکمل فیلڈ بہ فیلڈ ریفرنس"
---

# کنفیگریشن ریفرنس

`~/.openclaw/openclaw.json` میں دستیاب ہر فیلڈ۔ ٹاسک پر مبنی جائزے کے لیے، دیکھیں [Configuration](/gateway/configuration)।

کنفیگ فارمیٹ **JSON5** ہے (تبصرے + آخر میں کوما کی اجازت ہے)۔ تمام فیلڈز اختیاری ہیں — OpenClaw شامل نہ ہونے پر محفوظ ڈیفالٹس استعمال کرتا ہے۔

---

## چینلز

ہر چینل خودکار طور پر شروع ہوتا ہے جب اس کا کنفیگ سیکشن موجود ہو (جب تک `enabled: false` نہ ہو)۔

### DM اور گروپ رسائی

تمام چینلز DM پالیسیز اور گروپ پالیسیز کی معاونت کرتے ہیں:

| DM پالیسی                             | رویہ                                                                                              |
| ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `pairing` (ڈیفالٹ) | نامعلوم بھیجنے والوں کو ایک بار استعمال ہونے والا پیئرنگ کوڈ ملتا ہے؛ مالک کو منظوری دینی ہوتی ہے |
| `allowlist`                           | صرف وہ بھیجنے والے جو `allowFrom` میں ہوں (یا پیئر شدہ allow اسٹور میں)        |
| `open`                                | تمام آنے والے DMs کی اجازت دیں (اس کے لیے `allowFrom: ["*"]` درکار ہے)         |
| `disabled`                            | تمام آنے والے DMs کو نظر انداز کریں                                                               |

| گروپ پالیسی                             | رویہ                                                                                  |
| --------------------------------------- | ------------------------------------------------------------------------------------- |
| `allowlist` (ڈیفالٹ) | صرف وہ گروپس جو کنفیگر کی گئی allowlist سے مطابقت رکھتے ہوں                           |
| `open`                                  | گروپ allowlists کو بائی پاس کریں (mention-gating اب بھی لاگو ہوگا) |
| `disabled`                              | تمام گروپ/روم پیغامات بلاک کریں                                                       |

<Note>
`channels.defaults.groupPolicy` اس وقت ڈیفالٹ متعین کرتا ہے جب کسی پرووائیڈر کا `groupPolicy` سیٹ نہ ہو۔
Pairing کوڈز 1 گھنٹے کے بعد میعاد ختم ہو جاتے ہیں۔ Pending DM pairing درخواستیں **فی چینل 3** تک محدود ہوتی ہیں۔
Slack/Discord کے لیے ایک خاص fallback ہے: اگر ان کا provider سیکشن مکمل طور پر موجود نہ ہو تو runtime group policy `open` پر حل ہو سکتی ہے (اسٹارٹ اپ وارننگ کے ساتھ)۔
</Note>

### WhatsApp

WhatsApp گیٹ وے کے ویب چینل (Baileys Web) کے ذریعے چلتا ہے۔ جب لنک شدہ سیشن موجود ہو تو یہ خود بخود شروع ہو جاتا ہے۔

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

- آؤٹ باؤنڈ کمانڈز بطور ڈیفالٹ `default` اکاؤنٹ استعمال کرتی ہیں اگر موجود ہو؛ ورنہ ترتیب شدہ (sorted) کنفیگر شدہ اکاؤنٹ IDs میں سے پہلا استعمال کیا جاتا ہے۔
- Legacy سنگل اکاؤنٹ Baileys auth ڈائریکٹری کو `openclaw doctor` کے ذریعے `whatsapp/default` میں منتقل کیا جاتا ہے۔
- فی اکاؤنٹ اوور رائیڈز: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

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

- Bot token: `channels.telegram.botToken` یا `channels.telegram.tokenFile`، جبکہ ڈیفالٹ اکاؤنٹ کے لیے `TELEGRAM_BOT_TOKEN` بطور fallback استعمال ہوتا ہے۔
- `configWrites: false` Telegram کی جانب سے شروع کی گئی config تبدیلیوں کو بلاک کرتا ہے (supergroup ID مائیگریشنز، `/config set|unset`)۔
- Telegram اسٹریمنگ پریویوز `sendMessage` + `editMessageText` استعمال کرتے ہیں (ڈائریکٹ اور گروپ چیٹس دونوں میں کام کرتا ہے)۔
- Retry پالیسی: دیکھیں [Retry policy](/concepts/retry).

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

- Token: `channels.discord.token`، جبکہ ڈیفالٹ اکاؤنٹ کے لیے `DISCORD_BOT_TOKEN` بطور fallback استعمال ہوتا ہے۔
- ڈیلیوری ٹارگٹس کے لیے `user:<id>` (DM) یا `channel:<id>` (guild channel) استعمال کریں؛ صرف عددی IDs قبول نہیں کیے جاتے۔
- Guild slugs چھوٹے حروف میں ہوتے ہیں اور اسپیس کی جگہ `-` استعمال کیا جاتا ہے؛ چینل کیز slugged نام استعمال کرتی ہیں (`#` کے بغیر)۔ Guild IDs کو ترجیح دیں۔
- Bot کی جانب سے بھیجے گئے پیغامات کو بطور ڈیفالٹ نظر انداز کیا جاتا ہے۔ `allowBots: true` انہیں فعال کرتا ہے (اپنے پیغامات پھر بھی فلٹر رہتے ہیں)۔
- `maxLinesPerMessage` (ڈیفالٹ 17) لمبے پیغامات کو تقسیم کر دیتا ہے چاہے وہ 2000 حروف سے کم ہوں۔
- `channels.discord.ui.components.accentColor` Discord components v2 کنٹینرز کے لیے accent رنگ سیٹ کرتا ہے۔

**Reaction notification modes:** `off` (کوئی نہیں)، `own` (صرف bot کے پیغامات، ڈیفالٹ)، `all` (تمام پیغامات)، `allowlist` ( `guilds.<id>`.users\` میں موجود صارفین سے تمام پیغامات پر)۔

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

- Service account JSON: inline (`serviceAccount`) یا فائل پر مبنی (`serviceAccountFile`)۔
- Env fallback: `GOOGLE_CHAT_SERVICE_ACCOUNT` یا `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`۔
- ڈلیوری ٹارگٹس کے لیے `spaces/<spaceId>` یا `users/<userId|email>` استعمال کریں۔

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
          systemPrompt: "صرف مختصر جوابات دیں۔",
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

- **Socket mode** کے لیے `botToken` اور `appToken` دونوں درکار ہیں (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` بطور ڈیفالٹ اکاؤنٹ env fallback)۔
- **HTTP mode** کے لیے `botToken` کے ساتھ `signingSecret` درکار ہے (روٹ پر یا فی اکاؤنٹ)۔
- `configWrites: false` Slack کی جانب سے شروع کی گئی کنفیگ رائٹس کو بلاک کرتا ہے۔
- ڈلیوری ٹارگٹس کے لیے `user:<id>` (DM) یا `channel:<id>` استعمال کریں۔

**Reaction notification modes:** `off`, `own` (ڈیفالٹ), `all`, `allowlist` (`reactionAllowlist` سے)۔

**Thread session isolation:** `thread.historyScope` ہر تھریڈ کے لیے علیحدہ (ڈیفالٹ) یا چینل میں مشترکہ ہو سکتا ہے۔ `thread.inheritParent` نئے تھریڈز میں پیرنٹ چینل کا ٹرانسکرپٹ کاپی کرتا ہے۔

| ایکشن گروپ | ڈیفالٹ | نوٹس                                     |
| ---------- | ------ | ---------------------------------------- |
| reactions  | فعال   | ری ایکٹ کریں + ری ایکشنز کی فہرست دیکھیں |
| messages   | فعال   | پڑھیں/بھیجیں/ترمیم کریں/حذف کریں         |
| pins       | فعال   | پن کریں/ان پن کریں/فہرست دیکھیں          |
| memberInfo | فعال   | رکن کی معلومات                           |
| emojiList  | فعال   | کسٹم ایموجی فہرست                        |

### Mattermost

Mattermost بطور پلگ اِن فراہم کیا جاتا ہے: `openclaw plugins install @openclaw/mattermost`۔

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

چیٹ موڈز: `oncall` (@-mention پر جواب، ڈیفالٹ)، `onmessage` (ہر پیغام پر)، `onchar` (ٹرگر پری فکس سے شروع ہونے والے پیغامات)۔

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

**Reaction notification modes:** `off`, `own` (ڈیفالٹ), `all`, `allowlist` (`reactionAllowlist` سے)۔

### iMessage

OpenClaw `imsg rpc` (JSON-RPC بذریعہ stdio) چلاتا ہے۔ کسی daemon یا پورٹ کی ضرورت نہیں۔

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

- Messages DB کے لیے Full Disk Access درکار ہے۔
- `chat_id:<id>` اہداف کو ترجیح دیں۔ چیٹس کی فہرست کے لیے `imsg chats --limit 20` استعمال کریں۔
- `cliPath` کو SSH ریپر کی طرف پوائنٹ کیا جا سکتا ہے؛ SCP اٹیچمنٹ حاصل کرنے کے لیے `remoteHost` سیٹ کریں۔

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### متعدد اکاؤنٹس (تمام چینلز)

ہر چینل پر متعدد اکاؤنٹس چلائیں (ہر ایک کا اپنا `accountId` ہو):

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

- جب `accountId` شامل نہ کیا جائے تو `default` استعمال ہوتا ہے (CLI + routing)۔
- Env ٹوکنز صرف **default** اکاؤنٹ پر لاگو ہوتے ہیں۔
- بنیادی چینل سیٹنگز تمام اکاؤنٹس پر لاگو ہوتی ہیں جب تک کہ فی اکاؤنٹ اووررائیڈ نہ کی جائیں۔
- ہر اکاؤنٹ کو مختلف ایجنٹ کی طرف روٹ کرنے کے لیے `bindings[].match.accountId` استعمال کریں۔

### گروپ چیٹ مینشن گیٹنگ

گروپ پیغامات میں بطور ڈیفالٹ **mention لازمی** ہے (میٹاڈیٹا مینشن یا regex پیٹرنز)۔ یہ WhatsApp، Telegram، Discord، Google Chat، اور iMessage گروپ چیٹس پر لاگو ہوتا ہے۔

**مینشن کی اقسام:**

- **میٹاڈیٹا مینشنز**: پلیٹ فارم کے مقامی @-مینشنز۔ WhatsApp سیلف چیٹ موڈ میں نظر انداز کیے جاتے ہیں۔
- **ٹیکسٹ پیٹرنز**: `agents.list[].groupChat.mentionPatterns` میں موجود Regex پیٹرنز۔ ہمیشہ چیک کیے جاتے ہیں۔
- مینشن گیٹنگ صرف اسی وقت نافذ ہوتی ہے جب شناخت ممکن ہو (مقامی مینشنز یا کم از کم ایک پیٹرن)۔

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

`messages.groupChat.historyLimit` عالمی ڈیفالٹ سیٹ کرتا ہے۔ چینلز `channels.<channel>` کے ذریعے اووررائیڈ کر سکتے ہیں`.historyLimit` (یا فی اکاؤنٹ)۔ غیر فعال کرنے کے لیے `0` سیٹ کریں۔

#### DM ہسٹری کی حدود

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

حل: فی-DM اووررائیڈ → پرووائیڈر ڈیفالٹ → کوئی حد نہیں (سب محفوظ رہیں گے)۔

سپورٹڈ: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`۔

#### سیلف چیٹ موڈ

سیلف چیٹ موڈ فعال کرنے کے لیے اپنا نمبر `allowFrom` میں شامل کریں (مقامی @-مینشنز کو نظر انداز کرتا ہے، صرف ٹیکسٹ پیٹرنز پر جواب دیتا ہے):

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

### کمانڈز (چیٹ کمانڈ ہینڈلنگ)

```json5
{
  commands: {
    native: "auto", // جب سپورٹ ہو تو مقامی کمانڈز رجسٹر کریں
    text: true, // چیٹ پیغامات میں /commands کو پارس کریں
    bash: false, // ! کی اجازت دیں (عرف: /bash)
    bashForegroundMs: 2000,
    config: false, // /config کی اجازت دیں
    debug: false, // /debug کی اجازت دیں
    restart: false, // /restart + gateway restart ٹول کی اجازت دیں
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- ٹیکسٹ کمانڈز لازماً **الگ** پیغامات ہوں جن کے آغاز میں `/` ہو۔
- `native: "auto"` Discord/Telegram کے لیے مقامی کمانڈز آن کرتا ہے، Slack کو بند رکھتا ہے۔
- فی چینل اووررائیڈ کریں: `channels.discord.commands.native` (bool یا `"auto"`)۔ `false` پہلے سے رجسٹر شدہ کمانڈز کو صاف کر دیتا ہے۔
- `channels.telegram.customCommands` اضافی Telegram بوٹ مینو اندراجات شامل کرتا ہے۔
- `bash: true` `!` کو فعال کرتا ہے۔ <cmd>`میزبان شیل کے لیے۔`tools.elevated.enabled`کی ضرورت ہے اور بھیجنے والا`tools.elevated.allowFrom.<channel>` میں ہونا چاہیے`.
- `config: true` `/config` کو فعال کرتا ہے (`openclaw.json` کو پڑھتا/لکھتا ہے)۔
- `channels.<provider>``.configWrites` ہر چینل کے مطابق config میں تبدیلیوں کو کنٹرول کرتا ہے (ڈیفالٹ: true)۔
- `allowFrom` ہر provider کے لیے الگ ہوتا ہے۔ جب سیٹ کیا جائے تو یہی **واحد** اجازت کا ذریعہ ہوتا ہے (چینل allowlists/pairing اور `useAccessGroups` کو نظرانداز کر دیا جاتا ہے)۔
- `useAccessGroups: false` کمانڈز کو اجازت دیتا ہے کہ وہ رسائی گروپ پالیسیوں کو بائی پاس کریں جب `allowFrom` سیٹ نہ ہو۔

</Accordion>

---

## Agent کی ڈیفالٹ ترتیبات

### `agents.defaults.workspace`

ڈیفالٹ: `~/.openclaw/workspace`۔

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

اختیاری repository روٹ جو system prompt کی Runtime لائن میں دکھایا جاتا ہے۔ اگر سیٹ نہ ہو تو OpenClaw ورک اسپیس سے اوپر کی طرف تلاش کر کے خودکار طور پر شناخت کرتا ہے۔

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

ورک اسپیس bootstrap فائلوں (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`) کی خودکار تخلیق کو غیر فعال کرتا ہے۔

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

کٹنے (truncate) سے پہلے ہر ورک اسپیس bootstrap فائل کے لیے زیادہ سے زیادہ حروف۔ ڈیفالٹ: `20000`۔

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

تمام ورک اسپیس bootstrap فائلوں میں شامل کیے جانے والے کل زیادہ سے زیادہ حروف۔ ڈیفالٹ: `24000`۔

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

system prompt کے سیاق و سباق کے لیے ٹائم زون (پیغام کے ٹائم اسٹیمپس نہیں)۔ میزبان کے ٹائم زون پر واپس آتا ہے۔

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

system prompt میں وقت کا فارمیٹ۔ ڈیفالٹ: `auto` (OS کی ترجیح)۔

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

- `model.primary`: فارمیٹ `provider/model` (مثلاً `anthropic/claude-opus-4-6`)۔ اگر آپ provider شامل نہ کریں تو OpenClaw `anthropic` فرض کرتا ہے (متروک)۔
- `models`: `/model` کے لیے ترتیب دیا گیا ماڈل کیٹلاگ اور allowlist۔ ہر اندراج میں `alias` (شارٹ کٹ) اور `params` (فراہم کنندہ کے مطابق: `temperature`, `maxTokens`) شامل ہو سکتے ہیں۔
- `imageModel`: صرف اُس وقت استعمال ہوتا ہے جب بنیادی ماڈل میں امیج اِن پٹ کی سہولت موجود نہ ہو۔
- `maxConcurrent`: تمام سیشنز میں ایجنٹ کی بیک وقت چلنے والی زیادہ سے زیادہ رنز (ہر سیشن پھر بھی سلسلہ وار رہے گا)۔ ڈیفالٹ: 1۔

**بلٹ اِن alias شارٹ ہینڈز** (صرف اُس وقت لاگو ہوتے ہیں جب ماڈل `agents.defaults.models` میں موجود ہو):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

آپ کے ترتیب دیے گئے aliases ہمیشہ ڈیفالٹس پر فوقیت رکھتے ہیں۔

Z.AI GLM-4.x ماڈلز میں thinking موڈ خودکار طور پر فعال ہو جاتا ہے جب تک کہ آپ `--thinking off` سیٹ نہ کریں یا خود `agents.defaults.models["zai/<model>"].params.thinking` متعین نہ کریں۔

### `agents.defaults.cliBackends`

صرف متن پر مبنی فال بیک رنز (بغیر tool calls) کے لیے اختیاری CLI بیک اینڈز۔ جب API فراہم کنندگان ناکام ہو جائیں تو بطور بیک اپ مفید۔

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

- CLI بیک اینڈز متن کو بنیادی حیثیت دیتے ہیں؛ tools ہمیشہ غیر فعال رہتے ہیں۔
- جب `sessionArg` سیٹ ہو تو سیشنز سپورٹ کیے جاتے ہیں۔
- جب `imageArg` فائل پاتھ قبول کرے تو امیج پاس تھرو سپورٹ کیا جاتا ہے۔

### `agents.defaults.heartbeat`

وقفے وقفے سے چلنے والے heartbeat رنز۔

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
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

- `every`: دورانیے کی اسٹرنگ (ms/s/m/h)۔ ڈیفالٹ: `30m`۔
- فی ایجنٹ: `agents.list[].heartbeat` سیٹ کریں۔ جب کوئی بھی ایجنٹ `heartbeat` متعین کرتا ہے تو **صرف وہی ایجنٹس** heartbeat چلائیں گے۔
- Heartbeats مکمل ایجنٹ ٹرنز چلاتے ہیں — کم وقفہ زیادہ ٹوکنز استعمال کرے گا۔

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

- `mode`: `default` یا `safeguard` (طویل ہسٹری کے لیے حصوں میں خلاصہ سازی)۔ دیکھیں [Compaction](/concepts/compaction).
- `memoryFlush`: پائیدار یادداشتیں محفوظ کرنے کے لیے auto-compaction سے پہلے خاموش agentic ٹرن۔ جب workspace صرف پڑھنے کے قابل ہو تو اسے چھوڑ دیا جاتا ہے۔

### `agents.defaults.contextPruning`

LLM کو بھیجنے سے پہلے in-memory context سے **پرانے tool results** کو ہٹا دیتا ہے۔ ڈسک پر موجود سیشن ہسٹری میں **کوئی** تبدیلی نہیں کرتا۔

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
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

- `mode: "cache-ttl"` pruning passes کو فعال کرتا ہے۔
- `ttl` یہ کنٹرول کرتا ہے کہ pruning دوبارہ کب چل سکتی ہے (آخری cache touch کے بعد)۔
- Pruning پہلے بڑے tool results کو soft-trim کرتا ہے، پھر ضرورت پڑنے پر پرانے tool results کو hard-clear کرتا ہے۔

**Soft-trim** ابتدا اور اختتام برقرار رکھتا ہے اور درمیان میں `...` شامل کرتا ہے۔

**Hard-clear** پورے tool result کو placeholder سے تبدیل کر دیتا ہے۔

نوٹس:

- Image blocks کو کبھی trim یا clear نہیں کیا جاتا۔
- Ratios حروف کی تعداد پر مبنی ہوتے ہیں (تقریباً)، درست token counts نہیں۔
- اگر `keepLastAssistants` سے کم assistant پیغامات موجود ہوں تو pruning چھوڑ دی جاتی ہے۔

</Accordion>

رویے کی تفصیلات کے لیے [Session Pruning](/concepts/session-pruning) دیکھیں۔

### Block streaming

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

- Non-Telegram چینلز میں block replies فعال کرنے کے لیے واضح طور پر `*.blockStreaming: true` درکار ہے۔
- Channel overrides: `channels.<channel>`.blockStreamingCoalesce`(اور فی اکاؤنٹ ویریئنٹس)۔ Signal/Slack/Discord/Google Chat میں ڈیفالٹ`minChars: 1500\` ہے۔
- `humanDelay`: block replies کے درمیان بے ترتیب وقفہ۔ `natural` = 800–2500ms۔ فی-agent override: `agents.list[].humanDelay`۔

رویہ اور chunking کی تفصیلات کے لیے [Streaming](/concepts/streaming) دیکھیں۔

### Typing indicators

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

- ڈیفالٹس: براہِ راست چیٹس/mentions کے لیے `instant`، اور بغیر mention والے گروپ چیٹس کے لیے `message`۔
- فی سیشن overrides: `session.typingMode`, `session.typingIntervalSeconds`۔

دیکھیں [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

embedded agent کے لیے اختیاری **Docker sandboxing**۔ مکمل رہنمائی کے لیے [Sandboxing](/gateway/sandboxing) دیکھیں۔

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

**Workspace رسائی:**

- `none`: فی-scope sandbox workspace `~/.openclaw/sandboxes` کے تحت
- `ro`: sandbox workspace `/workspace` پر، agent workspace `/agent` پر صرف پڑھنے کے قابل mount کیا گیا
- `rw`: agent workspace کو `/workspace` پر read/write کے ساتھ mount کیا گیا

**دائرہ کار:**

- `session`: ہر سیشن کے لیے علیحدہ کنٹینر + ورک اسپیس
- `agent`: ہر ایجنٹ کے لیے ایک کنٹینر + ورک اسپیس (ڈیفالٹ)
- `shared`: مشترکہ کنٹینر اور ورک اسپیس (سیشنز کے درمیان کوئی علیحدگی نہیں)

**`setupCommand`** کنٹینر بننے کے بعد ایک بار چلایا جاتا ہے (بذریعہ `sh -lc`)۔ نیٹ ورک ایگریس، قابلِ تحریر روٹ، اور روٹ یوزر درکار ہے۔

**کنٹینرز کا ڈیفالٹ `network: "none"` ہوتا ہے** — اگر ایجنٹ کو آؤٹ باؤنڈ رسائی چاہیے تو اسے `"bridge"` پر سیٹ کریں۔

**ان باؤنڈ اٹیچمنٹس** فعال ورک اسپیس میں `media/inbound/*` کے اندر اسٹیج کی جاتی ہیں۔

**`docker.binds`** اضافی ہوسٹ ڈائریکٹریز کو ماؤنٹ کرتا ہے؛ گلوبل اور فی-ایجنٹ بائنڈز کو ضم کیا جاتا ہے۔

**Sandboxed browser** (`sandbox.browser.enabled`): کنٹینر میں Chromium + CDP۔ noVNC URL سسٹم پرامپٹ میں شامل کیا جاتا ہے۔ مین کنفیگ میں `browser.enabled` کی ضرورت نہیں ہے۔

- `allowHostControl: false` (ڈیفالٹ) سینڈ باکس سیشنز کو ہوسٹ براؤزر کو ہدف بنانے سے روکتا ہے۔
- `sandbox.browser.binds` اضافی ہوسٹ ڈائریکٹریز کو صرف سینڈ باکس براؤزر کنٹینر میں ماؤنٹ کرتا ہے۔ جب سیٹ کیا جائے (بشمول `[]`)، تو یہ براؤزر کنٹینر کے لیے `docker.binds` کو تبدیل کر دیتا ہے۔

</Accordion>

امیجز بنائیں:

```bash
scripts/sandbox-setup.sh           # مین سینڈ باکس امیج
scripts/sandbox-browser-setup.sh   # اختیاری براؤزر امیج
```

### `agents.list` (فی-ایجنٹ اووررائیڈز)

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
        model: "anthropic/claude-opus-4-6", // or { primary, fallbacks }
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

- `id`: مستحکم ایجنٹ آئی ڈی (لازمی)۔
- `default`: جب متعدد سیٹ ہوں تو پہلا منتخب ہوگا (وارننگ لاگ کی جاتی ہے)۔ اگر کوئی سیٹ نہ ہو تو فہرست کا پہلا اندراج ڈیفالٹ ہوگا۔
- `model`: اسٹرنگ فارم صرف `primary` کو اووررائیڈ کرتا ہے؛ آبجیکٹ فارم `{ primary, fallbacks }` دونوں کو اووررائیڈ کرتا ہے (`[]` گلوبل فال بیکس کو غیر فعال کر دیتا ہے)۔
- `identity.avatar`: ورک اسپیس سے متعلقہ راستہ، `http(s)` URL، یا `data:` URI۔
- `identity` ڈیفالٹس اخذ کرتا ہے: `ackReaction`، `emoji` سے اور `mentionPatterns`، `name`/`emoji` سے۔
- `subagents.allowAgents`: `sessions_spawn` کے لیے ایجنٹ آئی ڈیز کی اجازت فہرست (`["*"]` = کوئی بھی؛ ڈیفالٹ: صرف وہی ایجنٹ)۔

---

## ملٹی-ایجنٹ روٹنگ

ایک Gateway کے اندر متعدد علیحدہ ایجنٹس چلائیں۔ دیکھیں [Multi-Agent](/concepts/multi-agent)۔

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

### بائنڈنگ میچ فیلڈز

- `match.channel` (لازمی)
- `match.accountId` (اختیاری؛ `*` = کوئی بھی اکاؤنٹ؛ خالی ہو تو = ڈیفالٹ اکاؤنٹ)
- `match.peer` (اختیاری؛ `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (اختیاری؛ چینل کے مطابق)

**متعین میچ آرڈر:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (عین مطابق، بغیر peer/guild/team)
5. `match.accountId: "*"` (چینل بھر میں)
6. ڈیفالٹ ایجنٹ

ہر درجے کے اندر، سب سے پہلے میچ ہونے والی `bindings` انٹری کو ترجیح حاصل ہوتی ہے۔

### ہر ایجنٹ کے لیے علیحدہ رسائی پروفائلز

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

ترجیحی ترتیب کی تفصیلات کے لیے [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) دیکھیں۔

---

## سیشن

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
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: DMs کو کس طرح گروپ کیا جاتا ہے۔
  - `main`: تمام DMs مرکزی سیشن شیئر کرتے ہیں۔
  - `per-peer`: تمام چینلز میں بھیجنے والے کی id کے لحاظ سے علیحدہ۔
  - `per-channel-peer`: ہر چینل + بھیجنے والے کے حساب سے علیحدہ (متعدد صارفین کے ان باکس کے لیے تجویز کردہ)۔
  - `per-account-channel-peer`: ہر اکاؤنٹ + چینل + بھیجنے والے کے لحاظ سے علیحدہ (متعدد اکاؤنٹس کے لیے تجویز کردہ)۔
- **`identityLinks`**: کراس چینل سیشن شیئرنگ کے لیے معیاری ids کو provider-prefix والے peers سے میپ کریں۔
- **`reset`**: بنیادی ری سیٹ پالیسی۔ `daily` مقامی وقت کے مطابق `atHour` پر ری سیٹ ہوتا ہے؛ `idle`، `idleMinutes` کے بعد ری سیٹ ہوتا ہے۔ اگر دونوں کنفیگر ہوں تو جو پہلے ایکسپائر ہو وہ لاگو ہوگا۔
- **`resetByType`**: ہر قسم کے لیے علیحدہ اووررائیڈز (`direct`, `group`, `thread`)۔ پرانی `dm` کو `direct` کے متبادل کے طور پر قبول کیا جاتا ہے۔
- **`mainKey`**: پرانا فیلڈ۔ رن ٹائم اب مرکزی ڈائریکٹ چیٹ بکٹ کے لیے ہمیشہ `"main"` استعمال کرتا ہے۔
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`، پرانا `dm` بطور متبادل)، `keyPrefix`، یا `rawKeyPrefix` کے ذریعے میچ کریں۔ سب سے پہلا deny لاگو ہوگا۔
- **`maintenance`**: `warn` اخراج پر فعال سیشن کو خبردار کرتا ہے؛ `enforce` صفائی اور روٹیشن نافذ کرتا ہے۔

</Accordion>

---

## پیغامات

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
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
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### جواب کا سابقہ

ہر چینل/اکاؤنٹ کے لیے اووررائیڈز: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`۔

ترتیبِ ترجیح (سب سے مخصوص کو فوقیت): اکاؤنٹ → چینل → گلوبل۔ `""` غیر فعال کرتا ہے اور سلسلہ روک دیتا ہے۔ `"auto"` سے `[{identity.name}]` اخذ کیا جاتا ہے۔

**ٹیمپلیٹ ویری ایبلز:**

| ویری ایبل         | وضاحت                 | مثال                                     |
| ----------------- | --------------------- | ---------------------------------------- |
| `{model}`         | مختصر ماڈل نام        | `claude-opus-4-6`                        |
| `{modelFull}`     | مکمل ماڈل شناخت کنندہ | `anthropic/claude-opus-4-6`              |
| `{provider}`      | فراہم کنندہ کا نام    | `anthropic`                              |
| `{thinkingLevel}` | موجودہ تھنکنگ لیول    | `high`, `low`, `off`                     |
| `{identity.name}` | ایجنٹ کی شناخت کا نام | (بالکل "auto" کی طرح) |

ویری ایبلز کیس سے غیر حساس ہیں۔ `{think}`، `{thinkingLevel}` کا متبادل نام ہے۔

### تصدیقی ری ایکشن

- بطور ڈیفالٹ فعال ایجنٹ کی `identity.emoji`، بصورت دیگر "👀"۔ غیر فعال کرنے کے لیے `""` سیٹ کریں۔
- فی چینل اووررائیڈز: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`۔
- ریزولوشن کی ترتیب: اکاؤنٹ → چینل → `messages.ackReaction` → شناخت فال بیک۔
- دائرہ کار: `group-mentions` (ڈیفالٹ)، `group-all`، `direct`، `all`۔
- `removeAckAfterReply`: جواب کے بعد ack ہٹا دیتا ہے (صرف Slack/Discord/Telegram/Google Chat)۔

### ان باؤنڈ ڈی باؤنس

اسی بھیجنے والے کی تیزی سے آنے والی صرف ٹیکسٹ میسجز کو ایک ہی ایجنٹ ٹرن میں یکجا کرتا ہے۔ میڈیا/اٹیچمنٹس فوراً فلیش ہو جاتے ہیں۔ کنٹرول کمانڈز ڈی باؤنسنگ کو بائی پاس کرتی ہیں۔

### TTS (ٹیکسٹ ٹو اسپیچ)

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

- `auto` آٹو-TTS کو کنٹرول کرتا ہے۔ `/tts off|always|inbound|tagged` فی سیشن اووررائیڈ کرتا ہے۔
- `summaryModel` آٹو سمری کے لیے `agents.defaults.model.primary` کو اووررائیڈ کرتا ہے۔
- API کیز بطور فال بیک `ELEVENLABS_API_KEY`/`XI_API_KEY` اور `OPENAI_API_KEY` استعمال کرتی ہیں۔

---

## ٹاک

ٹاک موڈ کے لیے ڈیفالٹس (macOS/iOS/Android)۔

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

- Voice IDs بطور فال بیک `ELEVENLABS_VOICE_ID` یا `SAG_VOICE_ID` استعمال کرتے ہیں۔
- `apiKey` بطور فال بیک `ELEVENLABS_API_KEY` استعمال کرتا ہے۔
- `voiceAliases` Talk ہدایات کو دوستانہ نام استعمال کرنے کی اجازت دیتا ہے۔

---

## ٹولز

### ٹول پروفائلز

`tools.profile`، `tools.allow`/`tools.deny` سے پہلے ایک بنیادی allowlist سیٹ کرتا ہے:

| پروفائل     | شامل ہے                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | صرف `session_status`                                                                      |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | کوئی پابندی نہیں (unset جیسا ہی)                                       |

### ٹول گروپس

| گروپ               | ٹولز                                                                                             |
| ------------------ | ------------------------------------------------------------------------------------------------ |
| `group:runtime`    | `exec`, `process` (`bash` کو `exec` کے متبادل نام کے طور پر قبول کیا جاتا ہے) |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                           |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`         |
| `group:memory`     | `memory_search`, `memory_get`                                                                    |
| `group:web`        | `web_search`, `web_fetch`                                                                        |
| `group:ui`         | `browser`, `canvas`                                                                              |
| `group:automation` | `cron`, `gateway`                                                                                |
| `group:messaging`  | `message`                                                                                        |
| `group:nodes`      | `nodes`                                                                                          |
| `group:openclaw`   | تمام built-in ٹولز (provider plugins شامل نہیں)                               |

### `tools.allow` / `tools.deny`

عالمی ٹول allow/deny پالیسی (deny کو فوقیت حاصل ہے). کیس سے غیر حساس، `*` وائلڈ کارڈز کی سپورٹ کے ساتھ۔ Docker سینڈ باکس بند ہونے پر بھی لاگو ہوتا ہے۔

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

مخصوص providers یا models کے لیے ٹولز کو مزید محدود کریں۔ ترتیب: بنیادی پروفائل → provider پروفائل → allow/deny۔

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

ایلیویٹڈ (host) exec رسائی کو کنٹرول کرتا ہے:

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

- فی ایجنٹ اووررائیڈ (`agents.list[].tools.elevated`) صرف مزید پابندی لگا سکتا ہے۔
- `/elevated on|off|ask|full` ہر سیشن کے لیے اسٹیٹ محفوظ کرتا ہے؛ اِن لائن ہدایات صرف ایک پیغام پر لاگو ہوتی ہیں۔
- ایلیویٹڈ `exec` ہوسٹ پر چلتا ہے اور سینڈ باکسنگ کو بائی پاس کرتا ہے۔

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

ان باؤنڈ میڈیا کی سمجھ بوجھ (image/audio/video) کی ترتیب:

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

**Provider اندراج** (`type: "provider"` یا چھوڑ دیا جائے):

- `provider`: API provider کی شناخت (`openai`, `anthropic`, `google`/`gemini`, `groq`, وغیرہ)
- `model`: model id اووررائیڈ
- `profile` / `preferredProfile`: auth پروفائل کا انتخاب

**CLI اندراج** (`type: "cli"`):

- `command`: چلانے کے لیے executable
- `args`: ٹیمپلیٹڈ آرگومنٹس (جیسے `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` وغیرہ کی سپورٹ)

**مشترکہ فیلڈز:**

- `capabilities`: اختیاری فہرست (`image`, `audio`, `video`). ڈیفالٹس: `openai`/`anthropic`/`minimax` → image، `google` → image+audio+video، `groq` → audio۔
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: فی اندراج اووررائیڈز۔
- ناکامی کی صورت میں اگلا اندراج استعمال کیا جاتا ہے۔

Provider auth معیاری ترتیب کی پیروی کرتا ہے: auth پروفائلز → env vars → `models.providers.*.apiKey`۔

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

- `model`: بنائے گئے ذیلی ایجنٹس کے لیے ڈیفالٹ ماڈل۔ اگر چھوڑ دیا جائے تو ذیلی ایجنٹس کالر کا ماڈل وراثت میں لیتے ہیں۔
- فی سب ایجنٹ ٹول پالیسی: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## کسٹم پرووائیڈرز اور بیس URLs

OpenClaw، pi-coding-agent ماڈل کیٹلاگ استعمال کرتا ہے۔ config میں `models.providers` کے ذریعے یا `~/.openclaw/agents/<agentId>/agent/models.json` میں کسٹم پرووائیڈرز شامل کریں۔

```json5
{
  models: {
    mode: "merge", // merge (ڈیفالٹ) | replace
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

- کسٹم تصدیق کی ضروریات کے لیے `authHeader: true` + `headers` استعمال کریں۔
- `OPENCLAW_AGENT_DIR` (یا `PI_CODING_AGENT_DIR`) کے ذریعے ایجنٹ config روٹ کو اووررائیڈ کریں۔

### پرووائیڈر کی مثالیں

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

Cerebras کے لیے `cerebras/zai-glm-4.7` استعمال کریں؛ براہِ راست Z.AI کے لیے `zai/glm-4.7`۔

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

`OPENCODE_API_KEY` (یا `OPENCODE_ZEN_API_KEY`) سیٹ کریں۔ شارٹ کٹ: `openclaw onboard --auth-choice opencode-zen`۔

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

`ZAI_API_KEY` سیٹ کریں۔ `z.ai/*` اور `z-ai/*` قابلِ قبول عرفیات ہیں۔ شارٹ کٹ: `openclaw onboard --auth-choice zai-api-key`۔

- عمومی اینڈ پوائنٹ: `https://api.z.ai/api/paas/v4`
- کوڈنگ اینڈ پوائنٹ (ڈیفالٹ): `https://api.z.ai/api/coding/paas/v4`
- عمومی اینڈ پوائنٹ کے لیے، بیس URL اووررائیڈ کے ساتھ ایک کسٹم پرووائیڈر متعین کریں۔

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

چین کے اینڈ پوائنٹ کے لیے: `baseUrl: "https://api.moonshot.cn/v1"` یا `openclaw onboard --auth-choice moonshot-api-key-cn`۔

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

Anthropic کے ساتھ مطابقت رکھنے والا، بلٹ اِن پرووائیڈر۔ شارٹ کٹ: `openclaw onboard --auth-choice kimi-code-api-key`۔

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

Base URL میں `/v1` شامل نہیں ہونا چاہیے (Anthropic کلائنٹ اسے خود شامل کرتا ہے)۔ شارٹ کٹ: `openclaw onboard --auth-choice synthetic-api-key`۔

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

`MINIMAX_API_KEY` سیٹ کریں۔ شارٹ کٹ: `openclaw onboard --auth-choice minimax-api`۔

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) دیکھیں۔ خلاصہ: مضبوط ہارڈویئر پر LM Studio Responses API کے ذریعے MiniMax M2.1 چلائیں؛ فال بیک کے لیے ہوسٹڈ ماڈلز کو merge حالت میں رکھیں۔

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

- `allowBundled`: صرف بنڈلڈ skills کے لیے اختیاری allowlist (managed/workspace skills متاثر نہیں ہوتے)۔
- `entries.<skillKey>.enabled: false` کسی skill کو غیر فعال کر دیتا ہے، چاہے وہ بنڈلڈ/انسٹالڈ ہو۔
- `entries.<skillKey>.apiKey`: اُن skills کے لیے سہولت جو ایک بنیادی env var متعین کرتے ہیں۔

---

## پلگ اِنز

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

- `~/.openclaw/extensions`، `<workspace>/.openclaw/extensions`، اور `plugins.load.paths` سے لوڈ کیا گیا۔
- **Config میں تبدیلیوں کے لیے gateway کو دوبارہ شروع کرنا ضروری ہے۔**
- `allow`: اختیاری allowlist (صرف درج شدہ plugins لوڈ ہوں گے)۔ `deny` کو ترجیح حاصل ہے۔

[Plugins](/tools/plugin) دیکھیں۔

---

## براؤزر

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

- `evaluateEnabled: false`، `act:evaluate` اور `wait --fn` کو غیر فعال کر دیتا ہے۔
- Remote پروفائلز صرف attach کے لیے ہوتے ہیں (start/stop/reset غیر فعال)۔
- خودکار شناخت کی ترتیب: اگر ڈیفالٹ براؤزر Chromium پر مبنی ہو → Chrome → Brave → Edge → Chromium → Chrome Canary۔
- کنٹرول سروس: صرف loopback (پورٹ `gateway.port` سے اخذ کی جاتی ہے، ڈیفالٹ `18791`)۔

---

## UI

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

- `seamColor`: مقامی ایپ UI کروم کے لیے نمایاں رنگ (Talk Mode ببل ٹنٹ وغیرہ)۔
- `assistant`: Control UI شناخت کو اوور رائیڈ کرتا ہے۔ فعال ایجنٹ کی شناخت کو بطور متبادل استعمال کرتا ہے۔

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

- `mode`: `local` (gateway چلائیں) یا `remote` (ریموٹ gateway سے منسلک ہوں)۔ جب تک `local` نہ ہو، Gateway شروع ہونے سے انکار کر دیتا ہے۔
- `port`: WS + HTTP کے لیے ایک واحد multiplexed پورٹ۔ ترجیحی ترتیب: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`۔
- `bind`: `auto`، `loopback` (ڈیفالٹ)، `lan` (`0.0.0.0`)، `tailnet` (صرف Tailscale IP)، یا `custom`۔
- **Auth**: ڈیفالٹ طور پر لازمی ہے۔ Non-loopback bind کے لیے مشترکہ token/password درکار ہے۔ Onboarding وزرڈ ڈیفالٹ طور پر ایک token تیار کرتا ہے۔
- `auth.mode: "trusted-proxy"`: auth کو شناخت سے آگاہ reverse proxy کے سپرد کرتا ہے اور `gateway.trustedProxies` سے شناختی ہیڈرز پر اعتماد کرتا ہے (دیکھیں [Trusted Proxy Auth](/gateway/trusted-proxy-auth))۔
- `auth.allowTailscale`: جب `true` ہو تو Tailscale Serve کے شناختی ہیڈرز auth کو پورا کرتے ہیں (`tailscale whois` کے ذریعے تصدیق شدہ)۔ جب `tailscale.mode = "serve"` ہو تو ڈیفالٹ `true` ہوتا ہے۔
- `auth.rateLimit`: ناکام auth کو محدود کرنے کا اختیاری نظام۔ ہر کلائنٹ IP اور ہر auth دائرہ کار پر لاگو ہوتا ہے (shared-secret اور device-token کو الگ الگ ٹریک کیا جاتا ہے)۔ بلاک شدہ کوششوں پر `429` + `Retry-After` واپس کیا جاتا ہے۔
  - `auth.rateLimit.exemptLoopback` کا ڈیفالٹ `true` ہے؛ اگر آپ جان بوجھ کر localhost ٹریفک کو بھی rate-limit کرنا چاہتے ہیں (ٹیسٹ سیٹ اپ یا سخت proxy تعیناتی کے لیے) تو اسے `false` پر سیٹ کریں۔
- `tailscale.mode`: `serve` (صرف tailnet، loopback bind) یا `funnel` (عوامی، auth درکار)۔
- `remote.transport`: `ssh` (ڈیفالٹ) یا `direct` (ws/wss)۔ `direct` کے لیے، `remote.url` کا `ws://` یا `wss://` ہونا لازمی ہے۔
- `gateway.remote.token` صرف remote CLI کالز کے لیے ہے؛ یہ مقامی gateway auth کو فعال نہیں کرتا۔
- `trustedProxies`: reverse proxy IPs جو TLS کو ختم کرتے ہیں۔ صرف وہی پراکسیز درج کریں جن پر آپ کا کنٹرول ہو۔
- `gateway.tools.deny`: HTTP `POST /tools/invoke` کے لیے اضافی بلاک کیے گئے ٹول نام (ڈیفالٹ deny فہرست میں اضافہ کرتا ہے)۔
- `gateway.tools.allow`: ڈیفالٹ HTTP deny فہرست سے ٹول نام ہٹائیں۔

</Accordion>

### OpenAI کے ساتھ مطابقت رکھنے والے endpoints

- Chat Completions: ڈیفالٹ طور پر غیر فعال۔ اسے `gateway.http.endpoints.chatCompletions.enabled: true` کے ساتھ فعال کریں۔
- Responses API: `gateway.http.endpoints.responses.enabled`۔
- Responses URL-input کی سختی:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### متعدد انسٹینس آئسولیشن

ایک ہی ہوسٹ پر منفرد پورٹس اور state ڈائریکٹریز کے ساتھ متعدد gateways چلائیں:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

سہولت کے فلیگز: `--dev` (استعمال کرتا ہے `~/.openclaw-dev` + پورٹ `19001`)، `--profile <name>` (استعمال کرتا ہے `~/.openclaw-<name>`)۔

دیکھیں [Multiple Gateways](/gateway/multiple-gateways)۔

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

Auth: `Authorization: Bearer <token>` یا `x-openclaw-token: <token>`۔

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
  - درخواست کے payload سے `sessionKey` صرف اسی صورت قبول کیا جائے گا جب `hooks.allowRequestSessionKey=true` ہو (ڈیفالٹ: `false`)۔
- `POST /hooks/<name>` → `hooks.mappings` کے ذریعے حل کیا جاتا ہے

<Accordion title="Mapping details">

- `match.path`، `/hooks` کے بعد آنے والے سب-پاتھ سے میچ کرتا ہے (مثال کے طور پر `/hooks/gmail` → `gmail`)۔
- `match.source` عمومی پاتھز کے لیے payload کے کسی فیلڈ سے میچ کرتا ہے۔
- `{{messages[0].subject}}` جیسے ٹیمپلیٹس payload سے پڑھتے ہیں۔
- `transform` کسی JS/TS ماڈیول کی طرف اشارہ کر سکتا ہے جو ایک hook action واپس کرے۔
  - `transform.module` ایک relative path ہونا چاہیے اور `hooks.transformsDir` کے اندر ہی رہے گا (absolute paths اور traversal مسترد کر دیے جاتے ہیں)۔
- `agentId` مخصوص agent کی طرف روٹ کرتا ہے؛ نامعلوم IDs ڈیفالٹ پر واپس چلے جاتے ہیں۔
- `allowedAgentIds`: واضح روٹنگ کو محدود کرتا ہے (`*` یا خالی = سب کی اجازت، `[]` = سب کی ممانعت)۔
- `defaultSessionKey`: وہ اختیاری مقررہ سیشن کی جو explicit `sessionKey` کے بغیر hook agent رنز کے لیے استعمال ہوتی ہے۔
- `allowRequestSessionKey`: `/hooks/agent` کالرز کو `sessionKey` سیٹ کرنے کی اجازت دیتا ہے (ڈیفالٹ: `false`)۔
- `allowedSessionKeyPrefixes`: واضح `sessionKey` ویلیوز (request + mapping) کے لیے اختیاری prefix allowlist، مثال کے طور پر `["hook:"]`۔
- `deliver: true` حتمی جواب کو کسی چینل پر بھیجتا ہے؛ `channel` کا ڈیفالٹ `last` ہے۔
- `model` اس hook رن کے لیے LLM کو override کرتا ہے (اگر model catalog سیٹ ہو تو اس کی اجازت ہونا ضروری ہے)۔

</Accordion>

### Gmail انضمام

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

- Gateway کنفیگر ہونے پر بوٹ کے وقت خودکار طور پر `gog gmail watch serve` شروع کر دیتا ہے۔ `OPENCLAW_SKIP_GMAIL_WATCHER=1` سیٹ کریں تاکہ اسے غیر فعال کیا جا سکے۔
- Gateway کے ساتھ علیحدہ `gog gmail watch serve` مت چلائیں۔

---

## Canvas ہوسٹ

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway پورٹ کے تحت HTTP پر ایجنٹ کے ذریعے قابلِ ترمیم HTML/CSS/JS اور A2UI فراہم کرتا ہے:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- صرف لوکل کے لیے: `gateway.bind: "loopback"` برقرار رکھیں (ڈیفالٹ)۔
- نان-لوپ بیک بائنڈز: canvas روٹس کے لیے Gateway auth درکار ہے (token/password/trusted-proxy)، بالکل دیگر Gateway HTTP سرفیسز کی طرح۔
- Node WebViews عام طور پر auth ہیڈرز نہیں بھیجتے؛ نوڈ کے پیئر اور کنیکٹ ہونے کے بعد، Gateway ایک پرائیویٹ-IP فال بیک کی اجازت دیتا ہے تاکہ نوڈ canvas/A2UI لوڈ کر سکے بغیر URLs میں راز ظاہر کیے۔
- فراہم کردہ HTML میں live-reload کلائنٹ شامل کرتا ہے۔
- خالی ہونے پر ابتدائی `index.html` خودکار طور پر بناتا ہے۔
- اسی طرح `/__openclaw__/a2ui/` پر A2UI بھی فراہم کرتا ہے۔
- تبدیلیوں کے لیے gateway کو دوبارہ شروع کرنا ضروری ہے۔
- بڑی ڈائریکٹریز یا `EMFILE` خرابیوں کے لیے live reload غیر فعال کریں۔

---

## ڈسکوری

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

- `minimal` (ڈیفالٹ): TXT ریکارڈز سے `cliPath` + `sshPort` خارج کرتا ہے۔
- `full`: `cliPath` + `sshPort` شامل کرتا ہے۔
- ہوسٹ نیم کا ڈیفالٹ `openclaw` ہے۔ `OPENCLAW_MDNS_HOSTNAME` کے ذریعے اووررائیڈ کریں۔

### وائڈ-ایریا (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` کے تحت ایک unicast DNS-SD زون لکھتا ہے۔ کراس نیٹ ورک ڈسکوری کے لیے، اسے DNS سرور (CoreDNS تجویز کردہ) + Tailscale split DNS کے ساتھ جوڑیں۔

سیٹ اپ: `openclaw dns setup --apply`۔

---

## ماحول

### `env` (inline env ویری ایبلز)

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

- Inline env ویری ایبلز صرف اسی صورت لاگو ہوتے ہیں جب process env میں وہ key موجود نہ ہو۔
- `.env` فائلز: CWD `.env` + `~/.openclaw/.env` (کوئی بھی موجودہ ویری ایبلز کو اووررائیڈ نہیں کرتی)۔
- `shellEnv`: آپ کے login shell پروفائل سے غائب متوقع keys درآمد کرتا ہے۔
- مکمل ترجیحی ترتیب کے لیے [Environment](/help/environment) دیکھیں۔

### Env ویری ایبل متبادل

کسی بھی config اسٹرنگ میں env ویری ایبلز کو `${VAR_NAME}` کے ساتھ ریفرنس کریں:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- صرف بڑے حروف والے نام میچ ہوتے ہیں: `[A-Z_][A-Z0-9_]*`۔
- غائب یا خالی ویری ایبلز پر config لوڈ کے وقت خرابی آتی ہے۔
- لفظی `${VAR}` کے لیے `$${VAR}` استعمال کریں۔
- `$include` کے ساتھ کام کرتا ہے۔

---

## Auth اسٹوریج

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

- ہر ایجنٹ کے لیے Auth پروفائلز `<agentDir>/auth-profiles.json` میں محفوظ کیے جاتے ہیں۔
- `~/.openclaw/credentials/oauth.json` سے پرانے OAuth امپورٹس۔
- دیکھیں [OAuth](/concepts/oauth)۔

---

## لاگنگ

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

- ڈیفالٹ لاگ فائل: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`۔
- مستقل راستے کے لیے `logging.file` سیٹ کریں۔
- `--verbose` ہونے پر `consoleLevel` بڑھ کر `debug` ہو جاتا ہے۔

---

## Wizard

CLI وزارڈز (`onboard`, `configure`, `doctor`) کے ذریعے لکھی گئی میٹا ڈیٹا:

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

## شناخت

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

macOS آن بورڈنگ اسسٹنٹ کے ذریعے لکھا گیا۔ ڈیفالٹس اخذ کرتا ہے:

- `messages.ackReaction` کو `identity.emoji` سے اخذ کیا جاتا ہے (بصورتِ دیگر 👀 استعمال ہوتا ہے)
- `mentionPatterns` کو `identity.name`/`identity.emoji` سے اخذ کیا جاتا ہے
- `avatar` قبول کرتا ہے: ورک اسپیس سے متعلقہ راستہ، `http(s)` URL، یا `data:` URI

---

## Bridge (پرانا، ہٹا دیا گیا)

موجودہ بلڈز میں اب TCP برج شامل نہیں ہے۔ Nodes، Gateway WebSocket کے ذریعے منسلک ہوتے ہیں۔ `bridge.*` کیز اب config اسکیما کا حصہ نہیں ہیں (ہٹانے تک ویلیڈیشن ناکام رہے گی؛ `openclaw doctor --fix` نامعلوم کیز کو ہٹا سکتا ہے)۔

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
    sessionRetention: "24h", // دورانیے کی اسٹرنگ یا false
  },
}
```

- `sessionRetention`: مکمل شدہ cron سیشنز کو حذف کرنے سے پہلے کتنی دیر تک محفوظ رکھا جائے۔ ڈیفالٹ: `24h`۔

دیکھیں [Cron Jobs](/automation/cron-jobs)۔

---

## میڈیا ماڈل ٹیمپلیٹ ویری ایبلز

`tools.media.*.models[].args` میں پھیلائے جانے والے ٹیمپلیٹ پلیس ہولڈرز:

| ویری ایبل          | وضاحت                                                                      |
| ------------------ | -------------------------------------------------------------------------- |
| `{{Body}}`         | مکمل موصول ہونے والے پیغام کا باڈی                                         |
| `{{RawBody}}`      | خام باڈی (ہسٹری/بھیجنے والے کے ریپرز کے بغیر)           |
| `{{BodyStripped}}` | گروپ مینشنز ہٹا کر باڈی                                                    |
| `{{From}}`         | بھیجنے والے کی شناخت                                                       |
| `{{To}}`           | منزل کی شناخت                                                              |
| `{{MessageSid}}`   | چینل پیغام کی شناخت                                                        |
| `{{SessionId}}`    | موجودہ سیشن UUID                                                           |
| `{{IsNewSession}}` | نیا سیشن بننے پر "true"                                                    |
| `{{MediaUrl}}`     | آنے والے میڈیا کا فرضی URL                                                 |
| `{{MediaPath}}`    | لوکل میڈیا کا راستہ                                                        |
| `{{MediaType}}`    | میڈیا کی قسم (image/audio/document/…)                   |
| `{{Transcript}}`   | آڈیو ٹرانسکرپٹ                                                             |
| `{{Prompt}}`       | CLI اندراجات کے لیے طے شدہ میڈیا پرامپٹ                                    |
| `{{MaxChars}}`     | CLI اندراجات کے لیے طے شدہ زیادہ سے زیادہ آؤٹ پٹ حروف                      |
| `{{ChatType}}`     | `"direct"` یا `"group"`                                                    |
| `{{GroupSubject}}` | گروپ کا موضوع (حتی الامکان)                             |
| `{{GroupMembers}}` | گروپ اراکین کا پیش منظر (حتی الامکان)                   |
| `{{SenderName}}`   | بھیجنے والے کا نمائشی نام (حتی الامکان)                 |
| `{{SenderE164}}`   | بھیجنے والے کا فون نمبر (حتی الامکان)                   |
| `{{Provider}}`     | پرووائیڈر کا اشارہ (whatsapp, telegram, discord, وغیرہ) |

---

## کنفیگ میں شامل (`$include`)

کنفیگ کو متعدد فائلوں میں تقسیم کریں:

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

**ضم کرنے کا طریقہ کار:**

- واحد فائل: موجودہ آبجیکٹ کو مکمل طور پر تبدیل کر دیتی ہے۔
- فائلوں کی ارے: ترتیب کے مطابق ڈیپ-مرج کی جاتی ہے (بعد والی پہلے والی کو اووررائیڈ کرتی ہے)۔
- ہم سطح کی کیز: includes کے بعد مرج ہوتی ہیں (شامل کردہ ویلیوز کو اووررائیڈ کرتی ہیں)۔
- نیسٹڈ includes: زیادہ سے زیادہ 10 لیول گہرائی تک۔
- پاتھس: relative (شامل کرنے والی فائل کے لحاظ سے)، absolute، یا `../` پیرنٹ ریفرنسز۔
- خرابیاں: گمشدہ فائلوں، پارس ایررز، اور سرکلر includes کے لیے واضح پیغامات۔

---

_متعلقہ: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
