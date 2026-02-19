---
title: "Konfiguratsiya bo‘yicha ma’lumotnoma"
description: "~/.openclaw/openclaw.json uchun to‘liq maydonma-maydon ma’lumotnoma"
---

# Konfiguratsiya bo‘yicha ma’lumotnoma

`~/.openclaw/openclaw.json` dagi barcha mavjud maydonlar. Vazifaga yo‘naltirilgan umumiy ko‘rinish uchun [Configuration](/gateway/configuration) ga qarang.

Config formati **JSON5** (izohlar + oxirida vergul qo‘yishga ruxsat beriladi). Barcha maydonlar ixtiyoriy — ko‘rsatilmaganda OpenClaw xavfsiz standart sozlamalardan foydalanadi.

---

## Kanallar

Har bir kanal konfiguratsiya bo‘limi mavjud bo‘lsa avtomatik ishga tushadi (`enabled: false` bo‘lmasa).

### DM va guruh kirishi

Barcha kanallar DM siyosatlari va guruh siyosatlarini qo‘llab-quvvatlaydi:

| DM siyosati                             | Xatti-harakat                                                                                     |
| --------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `pairing` (standart) | Noma’lum jo‘natuvchilar bir martalik pairing kodini oladi; egasi tasdiqlashi kerak                |
| `allowlist`                             | Faqat `allowFrom` dagi jo‘natuvchilar (yoki pairing orqali ruxsat berilganlar) |
| `open`                                  | Barcha kiruvchi DMlarga ruxsat berish (`allowFrom: ["*"]` talab qilinadi)      |
| `disabled`                              | Barcha kiruvchi DMlarni e’tiborsiz qoldirish                                                      |

| Guruh siyosati                            | Xatti-harakat                                                                                |
| ----------------------------------------- | -------------------------------------------------------------------------------------------- |
| `allowlist` (standart) | Faqat sozlangan allowlist ga mos keladigan guruhlar                                          |
| `open`                                    | Guruh allowlistlarini chetlab o‘tish (mention-gating baribir amal qiladi) |
| `disabled`                                | Barcha guruh/xona xabarlarini bloklash                                                       |

<Note>
`channels.defaults.groupPolicy` provayderning `groupPolicy` qiymati o‘rnatilmagan bo‘lsa, standart qiymatni belgilaydi.
Ulanish (pairing) kodlari 1 soatdan keyin muddati tugaydi. Kutilayotgan DM ulanish so‘rovlari **har bir kanal uchun 3 ta** bilan cheklangan.
Slack/Discord uchun maxsus zaxira rejimi mavjud: agar ularning provayder bo‘limi umuman mavjud bo‘lmasa, ish vaqtida guruh siyosati `open` ga o‘rnatilishi mumkin (ishga tushirishda ogohlantirish bilan).
</Note>

### WhatsApp

WhatsApp gateway’ning veb kanali (Baileys Web) orqali ishlaydi. Ulangan sessiya mavjud bo‘lsa, avtomatik ravishda ishga tushadi.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // ko‘k belgilari (self-chat rejimida false)
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

- Chiquvchi buyruqlar sukut bo‘yicha `default` akkauntidan foydalanadi; agar u mavjud bo‘lmasa, sozlangan birinchi akkaunt ID’si (saralangan) ishlatiladi.
- Eski yagona akkauntli Baileys auth papkasi `openclaw doctor` tomonidan `whatsapp/default` ga ko‘chiriladi.
- Har bir akkaunt uchun alohida sozlamalar: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

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
          systemPrompt: "Javoblar qisqa bo‘lsin.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Mavzudan chetga chiqma.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git zaxira nusxasi" },
        { command: "generate", description: "Rasm yaratish" },
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

- Bot tokeni: `channels.telegram.botToken` yoki `channels.telegram.tokenFile`, sukut bo‘yicha akkaunt uchun zaxira sifatida `TELEGRAM_BOT_TOKEN`.
- `configWrites: false` Telegram tomonidan boshlangan konfiguratsiya yozuvlarini bloklaydi (superguruh ID migratsiyasi, `/config set|unset`).
- Telegram oqim (stream) ko‘rinishlari `sendMessage` + `editMessageText` dan foydalanadi (to‘g‘ridan-to‘g‘ri va guruh chatlarida ishlaydi).
- Qayta urinish siyosati: qarang [Retry policy](/concepts/retry).

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
              systemPrompt: "Faqat qisqa javoblar.",
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

- Token: `channels.discord.token`, sukut bo‘yicha akkaunt uchun zaxira sifatida `DISCORD_BOT_TOKEN`.
- Yetkazib berish manzillari uchun `user:<id>` (DM) yoki `channel:<id>` (guild kanali) dan foydalaning; oddiy raqamli ID’lar rad etiladi.
- Guild slug’lari kichik harflarda bo‘ladi va bo‘sh joylar `-` bilan almashtiriladi; kanal kalitlari slug qilingan nomdan foydalanadi (`#`siz). Guild ID’larini afzal ko‘ring.
- Bot tomonidan yozilgan xabarlar sukut bo‘yicha e’tiborga olinmaydi. `allowBots: true` ularni yoqadi (botning o‘z xabarlari baribir filtrlanadi).
- `maxLinesPerMessage` (sukut bo‘yicha 17) xabar 2000 belgidan kam bo‘lsa ham, uzun xabarlarni bo‘lib yuboradi.
- `channels.discord.ui.components.accentColor` Discord components v2 konteynerlari uchun aksent rangini belgilaydi.

**Reaksiya bildirishnoma rejimlari:** `off` (yo‘q), `own` (bot xabarlari, sukut bo‘yicha), `all` (barcha xabarlar), `allowlist` (`guilds.<id>`.users\` dagilardan barcha xabarlarda).

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

- Service account JSON: inline (`serviceAccount`) yoki fayl orqali (`serviceAccountFile`).
- Muhit (env) zaxira o‘zgaruvchilari: `GOOGLE_CHAT_SERVICE_ACCOUNT` yoki `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Yetkazib berish manzillari uchun `spaces/<spaceId>` yoki `users/<userId|email>` dan foydalaning.

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
          systemPrompt: "Faqat qisqa javoblar.",
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

- **Socket mode** `botToken` va `appToken` ni talab qiladi (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` standart akkaunt muhit o‘zgaruvchilari orqali zaxira sifatida).
- **HTTP mode** `botToken` hamda `signingSecret` ni talab qiladi (ildiz darajasida yoki har bir akkaunt uchun alohida).
- `configWrites: false` Slack tomonidan boshlangan konfiguratsiya yozuvlarini bloklaydi.
- Yetkazib berish manzillari uchun `user:<id>` (DM) yoki `channel:<id>` dan foydalaning.

**Reaksiya bildirishnoma rejimlari:** `off`, `own` (standart), `all`, `allowlist` (`reactionAllowlist` dan).

**Thread sessiyasini izolyatsiya qilish:** `thread.historyScope` har bir thread uchun alohida (standart) yoki butun kanal bo‘ylab umumiy bo‘lishi mumkin. `thread.inheritParent` yangi threadlarga ota kanal transkriptini nusxalaydi.

| Amallar guruhi | Standart | Izohlar                                  |
| -------------- | -------- | ---------------------------------------- |
| reaksiyalar    | yoqilgan | Reaksiya qo‘shish + reaksiyalar ro‘yxati |
| xabarlar       | yoqilgan | O‘qish/jo‘natish/tahrirlash/o‘chirish    |
| pinlar         | yoqilgan | Pin qilish/pinni olib tashlash/ro‘yxat   |
| memberInfo     | yoqilgan | A’zo ma’lumotlari                        |
| emojiList      | yoqilgan | Maxsus emoji ro‘yxati                    |

### Mattermost

Mattermost plagin sifatida yetkaziladi: `openclaw plugins install @openclaw/mattermost`.

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

Chat rejimlari: `oncall` (@-mention bo‘lganda javob beradi, standart), `onmessage` (har bir xabarga), `onchar` (trigger prefiksi bilan boshlanuvchi xabarlarga).

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

**Reaksiya bildirishnoma rejimlari:** `off`, `own` (standart), `all`, `allowlist` (`reactionAllowlist` dan).

### iMessage

OpenClaw `imsg rpc` ni ishga tushiradi (stdio orqali JSON-RPC). Daemon yoki port talab qilinmaydi.

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

- Messages DB uchun Full Disk Access talab qilinadi.
- `chat_id:<id>` manzillarini afzal ko‘ring. Chatlarni roʻyxatlash uchun `imsg chats --limit 20` dan foydalaning.
- `cliPath` SSH wrapper’ga ishora qilishi mumkin; SCP orqali biriktirmalarni yuklab olish uchun `remoteHost` ni sozlang.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Ko‘p akkaunt (barcha kanallar)

Har bir kanal uchun bir nechta akkauntni ishga tushiring (har biri o‘zining `accountId` iga ega):

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

- `accountId` ko‘rsatilmagan bo‘lsa, `default` ishlatiladi (CLI + marshrutlash).
- Env tokenlar faqat **default** akkauntga qo‘llaniladi.
- Asosiy kanal sozlamalari, agar akkaunt darajasida o‘zgartirilmagan bo‘lsa, barcha akkauntlarga qo‘llaniladi.
- Har bir akkauntni turli agentga yo‘naltirish uchun `bindings[].match.accountId` dan foydalaning.

### Guruh chatida mention bo‘yicha cheklash

Guruh xabarlari sukut bo‘yicha **mention talab qiladi** (metadata mention yoki regex andozalari). WhatsApp, Telegram, Discord, Google Chat va iMessage guruh chatlariga qo‘llaniladi.

**Mention turlari:**

- **Metadata mentionlar**: Platformaning o‘ziga xos @-mentionlari. WhatsApp self-chat rejimida e’tiborga olinmaydi.
- **Matn andozalari**: `agents.list[].groupChat.mentionPatterns` ichidagi regex andozalar. Har doim tekshiriladi.
- Mention bo‘yicha cheklash faqat aniqlash imkoni bo‘lganda (native mentionlar yoki kamida bitta andoza mavjud bo‘lsa) qo‘llaniladi.

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

`messages.groupChat.historyLimit` global sukut qiymatini belgilaydi. Kanallar `channels.<channel>` orqali o‘zgartirishi mumkin.historyLimit`(yoki akkaunt darajasida). O‘chirish uchun`0\` qiymatini o‘rnating.

#### DM tarix chegaralari

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

Yechim tartibi: DM darajasidagi override → provayderning sukut qiymati → cheklov yo‘q (hammasi saqlanadi).

Qo‘llab-quvvatlanadi: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat rejimi

Self-chat rejimini yoqish uchun o‘z raqamingizni `allowFrom` ga qo‘shing (native @-mentionlar e’tiborga olinmaydi, faqat matn andozalariga javob beradi):

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

### Buyruqlar (chat buyruqlarini boshqarish)

```json5
{
  commands: {
    native: "auto", // register native commands when supported
    text: true, // parse /commands in chat messages
    bash: false, // allow ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // allow /config
    debug: false, // allow /debug
    restart: false, // allow /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Matn buyruqlari boshida `/` bilan keladigan **alohida** xabar bo‘lishi kerak.
- `native: "auto"` Discord/Telegram uchun native buyruqlarni yoqadi, Slack’da o‘chiq qoldiradi.
- Har bir kanal uchun alohida sozlash: `channels.discord.commands.native` (bool yoki `"auto"`). `false` avval ro‘yxatdan o‘tgan buyruqlarni tozalaydi.
- `channels.telegram.customCommands` Telegram bot menyusiga qo‘shimcha bandlar qo‘shadi.
- `bash: true` `!` ni yoqadi <cmd>`host shell uchun.`tools.elevated.enabled`yoqilgan bo‘lishi va jo‘natuvchi`tools.elevated.allowFrom.<channel>` da ko‘rsatilgan bo‘lishi kerak`.
- `config: true` `/config` ni yoqadi (`openclaw.json` ni o‘qiydi/yozadi).
- `channels.<provider>.configWrites` har bir kanal bo‘yicha konfiguratsiya o‘zgarishlarini boshqaradi (standart: true).
- `allowFrom` har bir provider uchun alohida sozlanadi. Agar o‘rnatilgan bo‘lsa, bu **yagona** avtorizatsiya manbai hisoblanadi (kanal allowlist/pairing va `useAccessGroups` e’tiborga olinmaydi).
- `useAccessGroups: false` `allowFrom` o‘rnatilmagan bo‘lsa, buyruqlarga access-group siyosatlarini chetlab o‘tishga ruxsat beradi.

</Accordion>

---

## Agent standart sozlamalari

### `agents.defaults.workspace`

Standart: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Tizim promptidagi Runtime qatorida ko‘rsatiladigan ixtiyoriy repository ildiz papkasi. Agar o‘rnatilmagan bo‘lsa, OpenClaw workspace’dan yuqoriga qarab avtomatik aniqlaydi.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Workspace bootstrap fayllarini (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`) avtomatik yaratishni o‘chiradi.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Har bir workspace bootstrap fayli uchun qisqartirishdan oldingi maksimal belgilar soni. Standart: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Barcha workspace bootstrap fayllari bo‘yicha kiritiladigan umumiy maksimal belgilar soni. Standart: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Tizim prompt konteksti uchun vaqt mintaqasi (xabar vaqt belgilari uchun emas). Agar ko‘rsatilmagan bo‘lsa, host vaqt mintaqasiga qaytadi.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Tizim promptidagi vaqt formati. Standart: `auto` (OS sozlamasiga ko‘ra).

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

- `model.primary`: format `provider/model` (masalan, `anthropic/claude-opus-4-6`). Agar provider ko‘rsatilmasa, OpenClaw `anthropic` deb qabul qiladi (eskirgan).
- `models`: sozlangan model katalogi va `/model` uchun allowlist. Har bir yozuv `alias` (yorliq) va `params` (provayderga xos: `temperature`, `maxTokens`) ni o‘z ichiga olishi mumkin.
- `imageModel`: faqat asosiy model tasvir kiritishni qo‘llab-quvvatlamasa ishlatiladi.
- `maxConcurrent`: sessiyalar bo‘yicha maksimal parallel agent ishga tushirishlar soni (har bir sessiya baribir ketma-ket bajariladi). Standart: 1.

**O‘rnatilgan alias qisqartmalari** (faqat model `agents.defaults.models` ichida bo‘lsa qo‘llaniladi):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Siz sozlagan aliaslar har doim standartlardan ustun turadi.

Z.AI GLM-4.x modellari avtomatik ravishda thinking rejimini yoqadi, agar siz `--thinking off` ni o‘rnatmagan bo‘lsangiz yoki `agents.defaults.models["zai/<model>"].params.thinking` ni o‘zingiz belgilamagan bo‘lsangiz.

### `agents.defaults.cliBackends`

Faqat matnli zaxira ishga tushirishlar (tool chaqiruvlarisiz) uchun ixtiyoriy CLI backendlar. API provayderlari ishlamay qolganda zaxira sifatida foydali.

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

- CLI backendlar avvalo matnga yo‘naltirilgan; toollar har doim o‘chirilgan bo‘ladi.
- `sessionArg` o‘rnatilganda sessiyalar qo‘llab-quvvatlanadi.
- `imageArg` fayl yo‘llarini qabul qilsa, tasvirni to‘g‘ridan-to‘g‘ri uzatish qo‘llab-quvvatlanadi.

### `agents.defaults.heartbeat`

Davriy heartbeat ishga tushirishlari.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m o‘chiradi
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Agar mavjud bo‘lsa HEARTBEAT.md ni o‘qing...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: davomiylik satri (ms/s/m/h). Standart: `30m`.
- Har bir agent uchun: `agents.list[].heartbeat` ni o‘rnating. Agar biror agent `heartbeat` ni belgilasa, **faqat o‘sha agentlar** heartbeat bajaradi.
- Heartbeatlar to‘liq agent siklini bajaradi — qisqaroq interval ko‘proq token sarflaydi.

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
          systemPrompt: "Sessiya kompaksiyaga yaqinlashmoqda. Doimiy xotiralarni hozir saqlang.",
          prompt: "Har qanday uzoq muddatli eslatmalarni memory/YYYY-MM-DD.md ga yozing; saqlash uchun hech narsa bo‘lmasa NO_REPLY bilan javob bering.",
        },
      },
    },
  },
}
```

- `mode`: `default` yoki `safeguard` (uzoq tarixlar uchun bo‘laklab qisqartirish). [Compaction](/concepts/compaction) ga qarang.
- `memoryFlush`: doimiy xotiralarni saqlash uchun avtomatik kompaksiyadan oldingi jim agentik aylanish. Workspace faqat o‘qish uchun bo‘lsa, o‘tkazib yuboriladi.

### `agents.defaults.contextPruning`

LLMga yuborishdan oldin xotiradagi **eski vosita natijalarini** qisqartiradi. Diskdagi sessiya tarixini **o‘zgartirmaydi**.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // davomiylik (ms/s/m/h), standart birlik: daqiqa
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

- `mode: "cache-ttl"` qisqartirish jarayonlarini yoqadi.
- `ttl` qisqartirish qanchalik tez-tez qayta ishga tushishi mumkinligini boshqaradi (oxirgi keshga murojaatdan so‘ng).
- Qisqartirish avval haddan tashqari katta vosita natijalarini yumshoq qisqartiradi, zarur bo‘lsa keyinroq eski natijalarni to‘liq tozalaydi.

**Soft-trim** bosh va oxir qismini saqlab, o‘rtasiga `...` qo‘yadi.

**Hard-clear** butun vosita natijasini placeholder bilan almashtiradi.

Eslatmalar:

- Rasm bloklari hech qachon qisqartirilmaydi yoki tozalanmaydi.
- Nisbatlar belgilar soniga asoslangan (taxminiy), aniq tokenlar soniga emas.
- Agar `keepLastAssistants` dan kam assistant xabarlari mavjud bo‘lsa, qisqartirish o‘tkazib yuboriladi.

</Accordion>

Xulq-atvor tafsilotlari uchun [Session Pruning](/concepts/session-pruning) ga qarang.

### Blokli oqim

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (minMs/maxMs dan foydalaning)
    },
  },
}
```

- Telegram bo‘lmagan kanallarda blokli javoblarni yoqish uchun `*.blockStreaming: true` aniq ko‘rsatilishi kerak.
- Kanal override’lari: `channels.<channel>``.blockStreamingCoalesce` (va har bir akkaunt uchun variantlari). Signal/Slack/Discord/Google Chat uchun standart `minChars: 1500`.
- `humanDelay`: blokli javoblar orasidagi tasodifiy pauza. `natural` = 800–2500ms. Har bir agent uchun override: `agents.list[].humanDelay`.

Xulq-atvor va bo‘laklash tafsilotlari uchun [Streaming](/concepts/streaming) ga qarang.

### Yozish indikatori

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

- Standart sozlamalar: to‘g‘ridan-to‘g‘ri chatlar/mentionlar uchun `instant`, mention qilinmagan guruh chatlari uchun `message`.
- Har bir sessiya uchun override’lar: `session.typingMode`, `session.typingIntervalSeconds`.

[Typing Indicators](/concepts/typing-indicators) ga qarang.

### `agents.defaults.sandbox`

Ichki agent uchun ixtiyoriy **Docker sandboxing**. To‘liq qo‘llanma uchun [Sandboxing](/gateway/sandboxing) ga qarang.

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

**Workspace ruxsati:**

- `none`: har bir scope uchun `~/.openclaw/sandboxes` ostida alohida sandbox workspace
- `ro`: sandbox workspace `/workspace` da, agent workspace `/agent` ga faqat o‘qish uchun ulangan
- `rw`: agent workspace `/workspace` ga o‘qish/yozish huquqi bilan ulangan

**Scope:**

- `session`: har bir sessiya uchun konteyner + ish maydoni
- `agent`: har bir agent uchun bitta konteyner + ish maydoni (standart)
- `shared`: umumiy konteyner va ish maydoni (sessiyalararo izolyatsiyasiz)

**`setupCommand`** konteyner yaratilgandan so‘ng bir marta ishga tushadi (`sh -lc` orqali). Tarmoq chiqishiga (egress), yozish mumkin bo‘lgan root va root foydalanuvchi talab etiladi.

**Konteynerlar standart holatda `network: "none"` bilan ishga tushadi** — agar agentga tashqi kirish kerak bo‘lsa, uni `"bridge"` ga o‘rnating.

**Kirish (inbound) qo‘shimchalari** faol ish maydonidagi `media/inbound/*` papkasiga joylanadi.

**`docker.binds`** qo‘shimcha host kataloglarini mount qiladi; global va har bir agent uchun bind’lar birlashtiriladi.

**Sandboxed brauzer** (`sandbox.browser.enabled`): konteyner ichida Chromium + CDP. noVNC URL tizim promptiga qo‘shiladi. Asosiy konfiguratsiyada `browser.enabled` talab qilinmaydi.

- `allowHostControl: false` (standart) sandboxed sessiyalarning host brauzerga murojaat qilishini bloklaydi.
- `sandbox.browser.binds` qo‘shimcha host kataloglarini faqat sandbox brauzer konteyneriga mount qiladi. O‘rnatilganda (shu jumladan `[]`), brauzer konteyneri uchun `docker.binds` o‘rnini bosadi.

</Accordion>

Image’larni yig‘ish:

```bash
scripts/sandbox-setup.sh           # asosiy sandbox image
scripts/sandbox-browser-setup.sh   # ixtiyoriy brauzer image
```

### `agents.list` (har bir agent uchun override sozlamalar)

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
        model: "anthropic/claude-opus-4-6", // yoki { primary, fallbacks }
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

- `id`: barqaror agent identifikatori (majburiy).
- `default`: bir nechta belgilansa, birinchisi ustunlik qiladi (ogohlantirish log qilinadi). Agar hech biri belgilanmasa, ro‘yxatdagi birinchi element standart bo‘ladi.
- `model`: satr ko‘rinishi faqat `primary` ni override qiladi; obyekt ko‘rinishi `{ primary, fallbacks }` ikkalasini ham override qiladi (`[]` global fallback’larni o‘chiradi).
- `identity.avatar`: ish maydoniga nisbiy yo‘l, `http(s)` URL yoki `data:` URI.
- `identity` standart qiymatlarni hosil qiladi: `ackReaction` — `emoji` dan, `mentionPatterns` — `name`/`emoji` dan.
- `subagents.allowAgents`: `sessions_spawn` uchun ruxsat etilgan agent id’lari ro‘yxati (`["*"]` = istalgan; standart: faqat o‘sha agentning o‘zi).

---

## Ko‘p agentli marshrutlash

Bitta Gateway ichida bir nechta izolyatsiyalangan agentlarni ishga tushiring. Qarang: [Multi-Agent](/concepts/multi-agent).

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

### Binding moslash maydonlari

- `match.channel` (majburiy)
- `match.accountId` (ixtiyoriy; `*` = istalgan akkaunt; ko‘rsatilmasa = standart akkaunt)
- `match.peer` (ixtiyoriy; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (ixtiyoriy; kanalga xos)

**Deterministik moslash tartibi:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (aniq moslik, peer/guild/team ko‘rsatilmagan)
5. `match.accountId: "*"` (butun kanal bo‘yicha)
6. Standart agent

Har bir daraja ichida birinchi mos keladigan `bindings` yozuvi ustunlik qiladi.

### Har bir agent uchun kirish profillari

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

Ustuvorlik tafsilotlari uchun [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) sahifasiga qarang.

---

## Sessiya

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

- **`dmScope`**: DMlar qanday guruhlanishini belgilaydi.
  - `main`: barcha DMlar bitta asosiy sessiyani bo‘lishadi.
  - `per-peer`: kanallar bo‘yicha jo‘natuvchi id’si asosida ajratadi.
  - `per-channel-peer`: har bir kanal + jo‘natuvchi bo‘yicha ajratadi (ko‘p foydalanuvchili inboxlar uchun tavsiya etiladi).
  - `per-account-channel-peer`: har bir account + kanal + jo‘natuvchi bo‘yicha ajratadi (ko‘p accountlar uchun tavsiya etiladi).
- **`identityLinks`**: kanallararo sessiyani bo‘lishish uchun kanonik id’larni provider-prefiksli foydalanuvchilar bilan moslaydi.
- **`reset`**: asosiy reset siyosati. `daily` — mahalliy vaqt bo‘yicha `atHour` da reset qiladi; `idle` — `idleMinutes` dan so‘ng reset qiladi. Ikkalasi ham sozlangan bo‘lsa, qaysi biri oldin tugasa, o‘sha ustunlik qiladi.
- **`resetByType`**: turiga qarab (`direct`, `group`, `thread`) alohida sozlamalar. Legacy `dm` `direct` uchun alias sifatida qabul qilinadi.
- **`mainKey`**: legacy maydon. Runtime endi asosiy direct-chat bucket uchun har doim `"main"` dan foydalanadi.
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`, legacy `dm` alias bilan), `keyPrefix` yoki `rawKeyPrefix` bo‘yicha moslashtiradi. Birinchi `deny` ustunlik qiladi.
- **`maintenance`**: `warn` — sessiya o‘chirilishidan oldin faol sessiyani ogohlantiradi; `enforce` — tozalash va rotatsiyani majburiy qo‘llaydi.

</Accordion>

---

## Xabarlar

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

### Javob prefiksi

Har bir kanal/account uchun alohida sozlamalar: `channels.<channel> .responsePrefix`, `channels.<channel> .accounts.<id> .responsePrefix`.

Aniqlash tartibi (eng aniq ustunlik qiladi): account → channel → global. `""` o‘chiradi va kaskadni to‘xtatadi. `"auto"` `[{identity.name}]` dan hosil qilinadi.

**Shablon o‘zgaruvchilari:**

| O‘zgaruvchi                     | Tavsif                      | Misol                                     |
| ------------------------------- | --------------------------- | ----------------------------------------- |
| `{model}`                       | Qisqa model nomi            | `claude-opus-4-6`                         |
| {modelFull}                     | To‘liq model identifikatori | `anthropic/claude-opus-4-6`               |
| {provider}                      | Provayder nomi              | `anthropic`                               |
| {thinkingLevel}                 | Joriy fikrlash darajasi     | `high`, `low`, `off`                      |
| {identity.name} | Agent identifikatsiya nomi  | ("auto" bilan bir xil) |

O‘zgaruvchilar katta-kichik harflarga sezgir emas. `{think}` — `{thinkingLevel}` uchun alias.

### Tasdiqlash reaksiyasi

- Standart bo‘yicha faol agentning `identity.emoji`, aks holda `"👀"`. O‘chirish uchun `""` qilib belgilang.
- Har bir kanal uchun alohida sozlamalar: `channels.<channel>`
  `.ackReaction`, `channels.<channel>`
  `.accounts.<id>`
  `.ackReaction`.
- Aniqlash tartibi: account → channel → `messages.ackReaction` → identity fallback.
- Qo‘llanish doirasi: `group-mentions` (standart), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: javobdan keyin tasdiq reaksiyasini olib tashlaydi (faqat Slack/Discord/Telegram/Google Chat).

### Kiruvchi xabarlarni kechiktirib birlashtirish (debounce)

Bir yuboruvchidan tez-tez kelgan faqat matnli xabarlarni bitta agent javobiga birlashtiradi. Media/fayl biriktirmalari darhol yuboriladi. Boshqaruv buyruqlari debounce’ni chetlab o‘tadi.

### TTS (matnni nutqqa aylantirish)

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

- `auto` avtomatik TTS’ni boshqaradi. `/tts off|always|inbound|tagged` har bir sessiya uchun ustuvor sozlamani belgilaydi.
- `summaryModel` avtomatik xulosa uchun `agents.defaults.model.primary` o‘rnini bosadi.
- API kalitlari `ELEVENLABS_API_KEY`/`XI_API_KEY` va `OPENAI_API_KEY` qiymatlariga murojaat qiladi.

---

## Talk

Talk rejimi uchun standart sozlamalar (macOS/iOS/Android).

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

- Voice ID’lar `ELEVENLABS_VOICE_ID` yoki `SAG_VOICE_ID` qiymatlariga murojaat qiladi.
- `apiKey` `ELEVENLABS_API_KEY` ga avtomatik tarzda o‘tadi (fallback qiladi).
- `voiceAliases` Talk direktivalarida qulay nomlardan foydalanish imkonini beradi.

---

## Vositalar

### Vosita profillari

`tools.profile` `tools.allow`/`tools.deny` dan oldin asosiy ruxsatlar ro‘yxatini (allowlist) belgilaydi:

| Profil      | O‘z ichiga oladi                                                                          |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | faqat `session_status`                                                                    |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Hech qanday cheklov yo‘q (belgilanmagandek bir xil)                    |

### Vosita guruhlari

| Guruh              | Vositalar                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` `exec` uchun alias sifatida qabul qilinadi) |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Barcha ichki vositalar (provider plaginlari bundan mustasno)          |

### `tools.allow` / `tools.deny`

Global vosita ruxsat/taqiq siyosati (taqiq ustun keladi). Katta-kichik harflarga sezgir emas, `*` andozalarini qo‘llab-quvvatlaydi. Docker sandbox o‘chiq bo‘lsa ham qo‘llaniladi.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Muayyan provayderlar yoki modellar uchun vositalarni qo‘shimcha ravishda cheklaydi. Tartib: asosiy profil → provayder profili → allow/deny.

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

Kengaytirilgan (host) exec kirishini boshqaradi:

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

- Har bir agent uchun alohida sozlama (`agents.list[].tools.elevated`) faqat yanada cheklashi mumkin.
- `/elevated on|off|ask|full` holatni har bir sessiya uchun saqlaydi; inline direktivalar faqat bitta xabarga qo‘llaniladi.
- Kengaytirilgan `exec` hostda ishlaydi va sandbox cheklovlarini chetlab o‘tadi.

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
        apiKey: "brave_api_key", // yoki BRAVE_API_KEY env
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

Kiruvchi media kontentini (image/audio/video) tushunishni sozlaydi:

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

**Provider yozuvi** (`type: "provider"` yoki ko‘rsatilmagan):

- `provider`: API provayder identifikatori (`openai`, `anthropic`, `google`/`gemini`, `groq`, va boshqalar)
- `model`: model identifikatorini almashtirish
- `profile` / `preferredProfile`: autentifikatsiya profilini tanlash

**CLI yozuvi** (`type: "cli"`):

- `command`: ishga tushiriladigan bajariluvchi fayl
- `args`: shablonli argumentlar (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` va boshqalarni qo‘llab-quvvatlaydi)

**Umumiy maydonlar:**

- `capabilities`: ixtiyoriy ro‘yxat (`image`, `audio`, `video`). Standart qiymatlar: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: har bir yozuv uchun alohida sozlamalar.
- Xatolik yuz bersa, keyingi yozuvga o‘tiladi.

Provayder autentifikatsiyasi standart tartib bo‘yicha amalga oshiriladi: auth profillar → env o‘zgaruvchilari → `models.providers.*.apiKey`.

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

- `model`: yaratilgan sub-agentlar uchun standart model. Agar ko‘rsatilmagan bo‘lsa, sub-agentlar chaqiruvchining modelini meros qilib oladi.
- Har bir sub-agent uchun vosita siyosati: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Maxsus provayderlar va base URL manzillari

OpenClaw pi-coding-agent model katalogidan foydalanadi. Maxsus provayderlarni config faylidagi `models.providers` orqali yoki `~/.openclaw/agents/<agentId>/agent/models.json` da qo‘shing.

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

- Maxsus autentifikatsiya ehtiyojlari uchun `authHeader: true` va `headers` dan foydalaning.
- Agent config root papkasini `OPENCLAW_AGENT_DIR` (yoki `PI_CODING_AGENT_DIR`) orqali almashtiring.

### Provayder misollari

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

Cerebras uchun `cerebras/zai-glm-4.7`; Z.AI bevosita uchun `zai/glm-4.7` dan foydalaning.

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

`OPENCODE_API_KEY` (yoki `OPENCODE_ZEN_API_KEY`) ni o‘rnating. Qisqa buyruq: `openclaw onboard --auth-choice opencode-zen`.

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

`ZAI_API_KEY` ni o‘rnating. `z.ai/*` va `z-ai/*` qabul qilinadigan aliaslar hisoblanadi. Qisqa buyruq: `openclaw onboard --auth-choice zai-api-key`.

- Umumiy endpoint: `https://api.z.ai/api/paas/v4`
- Coding endpoint (standart): `https://api.z.ai/api/coding/paas/v4`
- Umumiy endpoint uchun base URL ni almashtirish orqali maxsus provayderni belgilang.

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

Xitoy endpointi uchun: `baseUrl: "https://api.moonshot.cn/v1"` yoki `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic bilan mos, ichki provayder. Qisqa buyruq: `openclaw onboard --auth-choice kimi-code-api-key`.

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

Base URL da `/v1` bo‘lmasligi kerak (Anthropic klienti uni avtomatik qo‘shadi). Qisqa buyruq: `openclaw onboard --auth-choice synthetic-api-key`.

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

`MINIMAX_API_KEY` ni o‘rnating. Qisqa buyruq: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) ga qarang. Qisqacha: MiniMax M2.1 ni kuchli apparat vositalarida LM Studio Responses API orqali ishga tushiring; zaxira uchun host qilingan modellarni merge holatida saqlang.

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

- `allowBundled`: faqat bundle qilingan skills uchun ixtiyoriy allowlist (managed/workspace skills ta’sirlanmaydi).
- `entries.<skillKey>`.enabled: false\` — skill bundle qilingan yoki o‘rnatilgan bo‘lsa ham uni o‘chiradi.
- `entries.<skillKey>`.apiKey\`: asosiy env o‘zgaruvchisini e’lon qilgan skills uchun qulaylik.

---

## Plaginlar

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

- `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` va `plugins.load.paths` dan yuklandi.
- **Config o‘zgarishlari gateway’ni qayta ishga tushirishni talab qiladi.**
- `allow`: ixtiyoriy allowlist (faqat ro‘yxatdagi pluginlar yuklanadi). `deny` ustunlik qiladi.

Qarang: [Plugins](/tools/plugin).

---

## Brauzer

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

- `evaluateEnabled: false` `act:evaluate` va `wait --fn` ni o‘chiradi.
- Remote profillar faqat ulanish uchun (ishga tushirish/to‘xtatish/qayta tiklash o‘chirilgan).
- Avto-aniqlash tartibi: agar Chromium asosidagi bo‘lsa — default brauzer → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Control xizmati: faqat loopback (port `gateway.port` dan olinadi, standart `18791`).

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

- `seamColor`: native ilova UI bezagi uchun urg‘u rangi (Talk Mode pufagi rangi va boshqalar).
- `assistant`: Control UI identifikatsiyasini almashtirish. Faol agent identifikatsiyasiga qaytadi.

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

- `mode`: `local` (gateway’ni ishga tushirish) yoki `remote` (masofaviy gateway’ga ulanish). `local` bo‘lmaguncha Gateway ishga tushirishni rad etadi.
- `port`: WS + HTTP uchun yagona multiplekslangan port. Ustuvorlik: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (standart), `lan` (`0.0.0.0`), `tailnet` (faqat Tailscale IP), yoki `custom`.
- **Auth**: standart bo‘yicha talab qilinadi. Loopback’dan tashqari bind’lar umumiy token/parolni talab qiladi. Onboarding ustasi standart bo‘yicha token yaratadi.
- `auth.mode: "trusted-proxy"`: autentifikatsiyani identity-aware reverse proxy’ga topshiradi va `gateway.trustedProxies` dagi identifikatsiya sarlavhalariga ishonadi (qarang: [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: `true` bo‘lsa, Tailscale Serve identifikatsiya sarlavhalari auth talabini qondiradi (`tailscale whois` orqali tekshiriladi). `tailscale.mode = "serve"` bo‘lsa, standart qiymat `true`.
- `auth.rateLimit`: muvaffaqiyatsiz auth urinishlari uchun ixtiyoriy cheklov. Har bir mijoz IP va har bir auth sohasi bo‘yicha qo‘llanadi (shared-secret va device-token alohida hisoblanadi). Bloklangan urinishlar `429` + `Retry-After` qaytaradi.
  - `auth.rateLimit.exemptLoopback` standart bo‘yicha `true`; agar localhost trafigini ham cheklamoqchi bo‘lsangiz (`test` muhitlari yoki qat’iy proxy sozlamalari uchun), `false` qilib qo‘ying.
- `tailscale.mode`: `serve` (faqat tailnet, loopback bind) yoki `funnel` (ommaviy, auth talab qilinadi).
- `remote.transport`: `ssh` (standart) yoki `direct` (ws/wss). `direct` uchun `remote.url` `ws://` yoki `wss://` bo‘lishi kerak.
- `gateway.remote.token` faqat masofaviy CLI chaqiruvlari uchun; lokal gateway auth’ni yoqmaydi.
- `trustedProxies`: TLS’ni yakunlaydigan reverse proxy IP manzillari. Faqat o‘zingiz boshqaradigan proxy’larni kiriting.
- `gateway.tools.deny`: HTTP `POST /tools/invoke` uchun bloklangan qo‘shimcha asbob nomlari (standart deny ro‘yxatini kengaytiradi).
- `gateway.tools.allow`: standart HTTP deny ro‘yxatidan asbob nomlarini olib tashlaydi.

</Accordion>

### OpenAI bilan mos endpointlar

- Chat Completions: standart bo‘yicha o‘chirilgan. `gateway.http.endpoints.chatCompletions.enabled: true` bilan yoqing.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Responses URL-input mustahkamlash:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Ko‘p instansiyali izolyatsiya

Bitta xostda noyob portlar va state papkalari bilan bir nechta gateway ishga tushiring:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Qulay flaglar: `--dev` (`~/.openclaw-dev` + `19001` portidan foydalanadi), `--profile <name>` (`~/.openclaw-<name>` dan foydalanadi).

[Multiple Gateways](/gateway/multiple-gateways) ga qarang.

---

## Hooklar

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

Auth: `Authorization: Bearer <token>` yoki `x-openclaw-token: <token>`.

**Endpointlar:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
  - `sessionKey` so‘rov payloadidan faqat `hooks.allowRequestSessionKey=true` bo‘lganda qabul qilinadi (standart: `false`).
- `POST /hooks/<name>` → `hooks.mappings` orqali aniqlanadi

<Accordion title="Mapping details">

- `match.path` `/hooks` dan keyingi sub-path bilan mos keladi (masalan, `/hooks/gmail` → `gmail`).
- `match.source` umumiy pathlar uchun payload maydoni bilan mos keladi.
- `{{messages[0].subject}}` kabi shablonlar payloaddan o‘qiladi.
- `transform` hook amalini qaytaruvchi JS/TS moduliga ishora qilishi mumkin.
  - `transform.module` nisbiy yo‘l bo‘lishi va `hooks.transformsDir` ichida qolishi kerak (mutlaq yo‘llar va traversal rad etiladi).
- `agentId` ma’lum bir agentga yo‘naltiradi; noma’lum IDlar standartga qaytadi.
- `allowedAgentIds`: aniq yo‘naltirishni cheklaydi (`*` yoki ko‘rsatilmagan = hammasiga ruxsat, `[]` = hammasi taqiqlanadi).
- `defaultSessionKey`: `sessionKey` aniq ko‘rsatilmagan hook agent ishga tushirishlari uchun ixtiyoriy qat’iy session kaliti.
- `allowRequestSessionKey`: `/hooks/agent` chaqiruvchilariga `sessionKey` ni o‘rnatishga ruxsat beradi (standart: `false`).
- `allowedSessionKeyPrefixes`: aniq `sessionKey` qiymatlari (so‘rov + mapping) uchun ixtiyoriy prefiks allowlist, masalan ` ["hook:"]`.
- `deliver: true` yakuniy javobni kanalga yuboradi; `channel` standart bo‘yicha `last`.
- `model` ushbu hook ishga tushirish uchun LLM ni almashtiradi (agar model katalogi o‘rnatilgan bo‘lsa, ruxsat etilgan bo‘lishi kerak).

</Accordion>

### Gmail integratsiyasi

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

- Gateway sozlanganda yuklanish vaqtida avtomatik ravishda `gog gmail watch serve` ni ishga tushiradi. O‘chirish uchun `OPENCLAW_SKIP_GMAIL_WATCHER=1` ni o‘rnating.
- Gateway bilan birga alohida `gog gmail watch serve` ishga tushirmang.

---

## Canvas xosti

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // yoki OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway porti ostida HTTP orqali agent tahrirlashi mumkin bo‘lgan HTML/CSS/JS va A2UI ni taqdim etadi:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Faqat lokal: `gateway.bind: "loopback"` (standart) qilib qoldiring.
- Non-loopback bindlarda: canvas marshrutlari boshqa Gateway HTTP interfeyslari kabi Gateway autentifikatsiyasini (token/parol/trusted-proxy) talab qiladi.
- Node WebView’lar odatda auth sarlavhalarini yubormaydi; node juftlangan va ulanganidan so‘ng, Gateway xususiy IP fallback’ni ruxsat beradi, shunda node canvas/A2UI’ni URL’ga maxfiy ma’lumotlarni oshkor qilmasdan yuklay oladi.
- Taqdim etilayotgan HTML’ga live-reload mijozini qo‘shadi.
- Bo‘sh bo‘lsa, boshlang‘ich `index.html` faylini avtomatik yaratadi.
- Shuningdek, A2UI’ni `/__openclaw__/a2ui/` manzilida taqdim etadi.
- O‘zgarishlar Gateway’ni qayta ishga tushirishni talab qiladi.
- Katta kataloglar yoki `EMFILE` xatolari uchun live reload’ni o‘chirib qo‘ying.

---

## Kashf etish

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

- `minimal` (standart): TXT yozuvlaridan `cliPath` va `sshPort` chiqarib tashlanadi.
- `full`: `cliPath` va `sshPort` kiritiladi.
- Xost nomi sukut bo‘yicha `openclaw`. `OPENCLAW_MDNS_HOSTNAME` bilan almashtiring.

### Keng hudud (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` ostida unicast DNS-SD zonasi yoziladi. Tarmoqlararo kashf etish uchun DNS server (CoreDNS tavsiya etiladi) va Tailscale split DNS bilan birga ishlating.

Sozlash: `openclaw dns setup --apply`.

---

## Muhit

### `env` (inline muhit o‘zgaruvchilari)

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

- Inline muhit o‘zgaruvchilari faqat jarayon muhitida kalit mavjud bo‘lmaganda qo‘llaniladi.
- `.env` fayllari: joriy ish katalogidagi `.env` va `~/.openclaw/.env` (ikkalasi ham mavjud o‘zgaruvchilarni ustiga yozmaydi).
- `shellEnv`: login shell profilingizdan yetishmayotgan kutilgan kalitlarni import qiladi.
- To‘liq ustuvorlik tartibi uchun [Environment](/help/environment) ga qarang.

### Muhit o‘zgaruvchilarini almashtirish

Istalgan konfiguratsiya satrida muhit o‘zgaruvchilariga `${VAR_NAME}` orqali murojaat qiling:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Faqat katta harfli nomlar mos keladi: `[A-Z_][A-Z0-9_]*`.
- Mavjud bo‘lmagan yoki bo‘sh o‘zgaruvchilar konfiguratsiya yuklanayotganda xatoga olib keladi.
- Literal `${VAR}` uchun `$${VAR}` dan foydalaning.
- `$include` bilan ishlaydi.

---

## Autentifikatsiya xotirasi

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

- Har bir agent uchun autentifikatsiya profillari `<agentDir>/auth-profiles.json` faylida saqlanadi.
- Eski OAuth maʼlumotlari `~/.openclaw/credentials/oauth.json` dan import qilinadi.
- Qarang: [OAuth](/concepts/oauth).

---

## Jurnallash

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

- Standart log fayli: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Barqaror yo‘l uchun `logging.file` ni sozlang.
- `--verbose` ishlatilganda `consoleLevel` `debug` ga ko‘tariladi.

---

## Ustoz (Wizard)

CLI ustozlari (`onboard`, `configure`, `doctor`) tomonidan yoziladigan metamaʼlumotlar:

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

## Identifikatsiya

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

macOS onboarding yordamchisi tomonidan yoziladi. Standart qiymatlar quyidagilardan hosil qilinadi:

- `messages.ackReaction` `identity.emoji` dan olinadi (bo‘lmasa 👀 ishlatiladi)
- `mentionPatterns` `identity.name`/`identity.emoji` dan olinadi
- `avatar` quyidagilarni qabul qiladi: workspace-relative path, `http(s)` URL yoki `data:` URI

---

## Bridge (eskirgan, olib tashlangan)

Joriy buildlarda TCP bridge endi mavjud emas. Tugunlar Gateway WebSocket orqali ulanadi. `bridge.*` kalitlari endi konfiguratsiya sxemasining bir qismi emas (olib tashlanmaguncha validatsiya muvaffaqiyatsiz bo‘ladi; `openclaw doctor --fix` nomaʼlum kalitlarni olib tashlashi mumkin).

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
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: yakunlangan cron sessiyalarini o‘chirishdan oldin qancha vaqt saqlash kerakligi. Standart: `24h`.

Qarang: [Cron Jobs](/automation/cron-jobs).

---

## Media model shablon o‘zgaruvchilari

Shablon joy to‘ldirgichlari `tools.media.*.models[].args` ichida kengaytiriladi:

| O‘zgaruvchi        | Tavsif                                                                              |
| ------------------ | ----------------------------------------------------------------------------------- |
| `{{Body}}`         | To‘liq kiruvchi xabar matni                                                         |
| `{{RawBody}}`      | Xom matn (tarix/jo‘natuvchi qobig‘isiz)                          |
| `{{BodyStripped}}` | Guruh mentionlarisiz tozalangan matn                                                |
| `{{From}}`         | Yuboruvchi identifikatori                                                           |
| `{{To}}`           | Qabul qiluvchi identifikatori                                                       |
| `{{MessageSid}}`   | Kanal xabar identifikatori                                                          |
| `{{SessionId}}`    | Joriy sessiya UUID                                                                  |
| `{{IsNewSession}}` | Yangi sessiya yaratilganda `"true"`                                                 |
| `{{MediaUrl}}`     | Kiruvchi media pseudo-URL                                                           |
| `{{MediaPath}}`    | Mahalliy media yo‘li                                                                |
| `{{MediaType}}`    | Media turi (image/audio/document/…)                              |
| `{{Transcript}}`   | Audio transkripti                                                                   |
| `{{Prompt}}`       | CLI yozuvlari uchun aniqlangan media prompt                                         |
| `{{MaxChars}}`     | CLI yozuvlari uchun aniqlangan maksimal chiqish belgilar soni                       |
| `{{ChatType}}`     | `"direct"` yoki `"group"`                                                           |
| `{{GroupSubject}}` | Guruh mavzusi (imkon qadar)                                      |
| `{{GroupMembers}}` | Guruh a’zolari ko‘rinishi (imkon qadar)                          |
| `{{SenderName}}`   | Yuboruvchining ko‘rsatiladigan ismi (imkon qadar)                |
| `{{SenderE164}}`   | Yuboruvchining telefon raqami (imkon qadar)                      |
| `{{Provider}}`     | Provayder ko‘rsatmasi (whatsapp, telegram, discord va boshqalar) |

---

## Config includes (`$include`)

Konfiguratsiyani bir nechta faylga ajrating:

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

**Birlashtirish xatti-harakati:**

- Bitta fayl: o‘z ichiga olgan obyektni almashtiradi.
- Fayllar ro‘yxati: tartib bo‘yicha chuqur birlashtiriladi (keyingisi oldingisini ustidan yozadi).
- Bir xil darajadagi kalitlar: include’dan keyin birlashtiriladi (include qilingan qiymatlarni ustidan yozadi).
- Ichma-ich include’lar: 10 darajagacha.
- Yo‘llar: nisbiy (include qilayotgan faylga nisbatan), absolyut yoki `../` ota papkaga havolalar.
- Xatolar: mavjud bo‘lmagan fayllar, parse xatolari va aylana include’lar uchun aniq xabarlar.

---

_Tegishli: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

