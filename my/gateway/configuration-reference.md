---
title: "Configuration ကိုးကားချက်"
description: "~/.openclaw/openclaw.json အတွက် field တစ်ခုချင်းစီ အပြည့်အစုံ ကိုးကားချက်"
---

# Configuration ကိုးကားချက်

`~/.openclaw/openclaw.json` တွင် ရရှိနိုင်သော field များအားလုံး။ လုပ်ငန်းအခြေပြု အကျဉ်းချုပ်ကို ကြည့်ရန် [Configuration](/gateway/configuration) ကို ကြည့်ပါ။

Config format သည် **JSON5** (comments နှင့် trailing commas ခွင့်ပြုထားသည်) ဖြစ်သည်။ field များအားလုံးသည် မဖြစ်မနေ မလိုအပ်ပါ — OpenClaw သည် မဖော်ပြထားပါက လုံခြုံသော မူလတန်ဖိုးများကို အသုံးပြုသည်။

---

## Channels

channel တစ်ခုချင်းစီသည် ၎င်း၏ config section ရှိပါက အလိုအလျောက် စတင်မည် (`enabled: false` မဟုတ်လျှင်)။

### DM နှင့် group ဝင်ရောက်ခွင့်

channel များအားလုံးသည် DM policies နှင့် group policies များကို ထောက်ပံ့သည်:

| DM policy                                 | အပြုအမူ                                                                                            |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------- |
| `pairing` (မူလတန်ဖိုး) | မသိသော ပေးပို့သူများသည် တစ်ကြိမ်သာ အသုံးပြုနိုင်သော pairing code ကို ရရှိမည်; owner မှ အတည်ပြုရမည် |
| `allowlist`                               | `allowFrom` (သို့မဟုတ် paired allow store) ထဲရှိ ပေးပို့သူများသာ ခွင့်ပြုမည်    |
| `open`                                    | ဝင်လာသော DMs အားလုံးကို ခွင့်ပြုမည် (`allowFrom: ["*"]` လိုအပ်သည်)              |
| `disabled`                                | အဝင် DM များအားလုံးကို လျစ်လျူရှုပါ                                                                |

| အဖွဲ့ မူဝါဒ                              | အပြုအမူ                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `allowlist` (default) | သတ်မှတ်ထားသော allowlist နှင့် ကိုက်ညီသော အဖွဲ့များသာ                                             |
| `open`                                   | အဖွဲ့ allowlist များကို ကျော်လွှားမည် (mention-gating သည် ဆက်လက် သက်ရောက်မည်) |
| `disabled`                               | အဖွဲ့/ခန်းမ မက်ဆေ့ခ်ျများအားလုံးကို ပိတ်မည်                                                      |

<Note>
`channels.defaults.groupPolicy` သည် provider ၏ `groupPolicy` မသတ်မှတ်ထားပါက မူလတန်ဖိုးအဖြစ် သတ်မှတ်ပေးသည်။
Pairing ကုဒ်များသည် ၁ နာရီအကြာတွင် သက်တမ်းကုန်ဆုံးမည်။ စောင့်ဆိုင်းနေသော DM pairing တောင်းဆိုမှုများကို **channel တစ်ခုလျှင် ၃ ခုအထိသာ** ကန့်သတ်ထားသည်။
Slack/Discord တွင် အထူး fallback ရှိသည်။ ၎င်းတို့၏ provider အပိုင်း လုံးဝ မရှိပါက runtime group policy သည် `open` သို့ သတ်မှတ်နိုင်သည် (startup သတိပေးချက်နှင့်အတူ)။
</Note>

### WhatsApp

WhatsApp သည် gateway ၏ web channel (Baileys Web) မှတစ်ဆင့် လည်ပတ်သည်။ ချိတ်ဆက်ထားသော session ရှိပါက အလိုအလျောက် စတင်မည်။

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

- အထွက် command များသည် `default` account ရှိပါက ၎င်းကို မူလအဖြစ် အသုံးပြုမည်။ မရှိပါက စီထားသော account id များအနက် ပထမဆုံးတစ်ခုကို အသုံးပြုမည်။
- အဟောင်း single-account Baileys auth dir ကို `openclaw doctor` မှ `whatsapp/default` သို့ ပြောင်းရွှေ့ပေးသည်။
- Per-account overrides: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

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

- Bot token: `channels.telegram.botToken` သို့မဟုတ် `channels.telegram.tokenFile` ကို အသုံးပြုပါ၊ မူလ account အတွက် `TELEGRAM_BOT_TOKEN` ကို fallback အဖြစ် အသုံးပြုနိုင်သည်။
- `configWrites: false` သည် Telegram မှ စတင်သော config ရေးသားမှုများ (supergroup ID ပြောင်းရွှေ့ခြင်းများ၊ `/config set|unset`) ကို ပိတ်ဆို့သည်။
- Telegram stream preview များသည် `sendMessage` + `editMessageText` ကို အသုံးပြုသည် (direct နှင့် group chat နှစ်မျိုးလုံးတွင် လုပ်ဆောင်နိုင်သည်)။
- ပြန်လည်ကြိုးစားမှု မူဝါဒ: [ပြန်လည်ကြိုးစားမှု မူဝါဒ](/concepts/retry) ကို ကြည့်ပါ။

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

- Token: `channels.discord.token` ကို အသုံးပြုပါ၊ မူလ account အတွက် `DISCORD_BOT_TOKEN` ကို fallback အဖြစ် အသုံးပြုနိုင်သည်။
- ပို့ဆောင်ရန် target အဖြစ် `user:<id>` (DM) သို့မဟုတ် `channel:<id>` (guild channel) ကို အသုံးပြုပါ။ ဂဏန်းသာ ပါသော ID များကို လက်မခံပါ။
- Guild slug များသည် အက္ခရာအသေးဖြင့် ရေးပြီး space များကို `-` ဖြင့် အစားထိုးထားသည်။ channel key များသည် slug ပြောင်းထားသော နာမည်ကို အသုံးပြုသည် (`#` မပါဝင်)။ Guild ID များကို ဦးစားပေး အသုံးပြုပါ။
- Bot မှရေးသားသော မက်ဆေ့ခ်ျများကို မူလအနေဖြင့် လျစ်လျူရှုထားသည်။ `allowBots: true` သတ်မှတ်ပါက ၎င်းတို့ကို ဖွင့်ပေးမည် (ကိုယ်ပိုင် မက်ဆေ့ခ်ျများကိုတော့ ဆက်လက် စစ်ထုတ်ထားမည်)။
- `maxLinesPerMessage` (မူလ 17) သည် စာလုံးရေ 2000 အောက်ရှိသော်လည်း လိုင်းများ များပြားသော မက်ဆေ့ခ်ျများကို ခွဲပေးသည်။
- `channels.discord.ui.components.accentColor` သည် Discord components v2 container များအတွက် accent color ကို သတ်မှတ်ပေးသည်။

**Reaction notification modes:** `off` (မရှိ), `own` (bot ၏ မက်ဆေ့ချ်များ၊ ပုံမှန်), `all` (မက်ဆေ့ချ်အားလုံး), `allowlist` (`guilds.<id>` မှ.users\` ပေါ်ရှိ မက်ဆေ့ချ်အားလုံး)။

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

- Service account JSON ကို inline (`serviceAccount`) သို့မဟုတ် ဖိုင်အခြေပြု (`serviceAccountFile`) အဖြစ် အသုံးပြုနိုင်သည်။
- Env fallback များ៖ `GOOGLE_CHAT_SERVICE_ACCOUNT` သို့မဟုတ် `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`။
- ပို့ဆောင်ရန် target များအတွက် `spaces/<spaceId>` သို့မဟုတ် `users/<userId|email>` ကို အသုံးပြုပါ။

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

- **Socket mode** သည် `botToken` နှင့် `appToken` နှစ်ခုလုံး လိုအပ်သည် (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` ကို default account env fallback အဖြစ် အသုံးပြုနိုင်သည်)။
- **HTTP mode** သည် `botToken` နှင့် `signingSecret` (root သို့မဟုတ် account တစ်ခုချင်းစီအလိုက်) လိုအပ်သည်။
- `configWrites: false` သတ်မှတ်ပါက Slack မှ စတင်သော config ပြင်ဆင်ရေးသားမှုများကို တားဆီးသည်။
- ပို့ဆောင်ရန် target များအတွက် `user:<id>` (DM) သို့မဟုတ် `channel:<id>` ကို အသုံးပြုပါ။

**Reaction notification modes:** `off`, `own` (ပုံမှန်), `all`, `allowlist` (`reactionAllowlist` မှ)။

**Thread session isolation:** `thread.historyScope` သည် thread တစ်ခုချင်းစီအလိုက် (ပုံမှန်) သို့မဟုတ် channel အတွင်း မျှဝေအသုံးပြုနိုင်သည်။ `thread.inheritParent` သည် parent channel transcript ကို thread အသစ်များသို့ ကူးယူပေးသည်။

| Action group | ပုံမှန် | မှတ်စုများ                               |
| ------------ | ------- | ---------------------------------------- |
| reactions    | enabled | React လုပ်ရန် + reactions စာရင်းကြည့်ရန် |
| messages     | enabled | ဖတ်ရန်/ပို့ရန်/ပြင်ဆင်ရန်/ဖျက်ရန်        |
| pins         | enabled | Pin လုပ်ရန်/ဖြုတ်ရန်/စာရင်းကြည့်ရန်      |
| memberInfo   | enabled | Member အချက်အလက်                         |
| emojiList    | enabled | စိတ်ကြိုက် emoji စာရင်း                  |

### Mattermost

Mattermost ကို plugin အဖြစ် ထည့်သွင်းအသုံးပြုသည်: `openclaw plugins install @openclaw/mattermost`။

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

Chat mode များ៖ `oncall` (@-mention ပြုလုပ်သောအခါ တုံ့ပြန်သည်၊ ပုံမှန်), `onmessage` (မက်ဆေ့ချ်တိုင်းတွင် တုံ့ပြန်သည်), `onchar` (သတ်မှတ်ထားသော trigger prefix ဖြင့် စတင်သော မက်ဆေ့ချ်များတွင် တုံ့ပြန်သည်)။

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

**တုံ့ပြန်မှု အသိပေးချက် မုဒ်များ:** `off`, `own` (မူလသတ်မှတ်ချက်), `all`, `allowlist` (`reactionAllowlist` မှ).

### iMessage

OpenClaw သည် `imsg rpc` (stdio မှတဆင့် JSON-RPC) ကို စတင်ဖွင့်လှစ်သည်။ daemon သို့မဟုတ် port မလိုအပ်ပါ။

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

- Messages DB အတွက် Full Disk Access လိုအပ်သည်။
- `chat_id:<id>` ကို ဦးစားပေး အသုံးပြုပါ။ chat များကို စာရင်းပြုစုရန် `imsg chats --limit 20` ကို အသုံးပြုပါ။
- `cliPath` သည် SSH wrapper ကို ညွှန်ပြနိုင်သည်; attachment များကို SCP ဖြင့် ရယူရန် `remoteHost` ကို သတ်မှတ်ပါ။

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Multi-account (channel အားလုံး)

channel တစ်ခုချင်းစီအတွက် account များစွာကို လည်ပတ်နိုင်သည် (တစ်ခုချင်းစီတွင် ကိုယ်ပိုင် `accountId` ရှိသည်):

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

- `accountId` မသတ်မှတ်ထားပါက `default` ကို အသုံးပြုမည် (CLI + routing).
- Env token များသည် **default** account အတွက်သာ သက်ဆိုင်သည်။
- အခြေခံ channel ဆက်တင်များသည် account တစ်ခုချင်းစီအလိုက် မပြောင်းလဲထားလျှင် account အားလုံးအတွက် သက်ဆိုင်သည်။
- account တစ်ခုချင်းစီကို agent မတူညီစေရန် လမ်းကြောင်းခွဲရန် `bindings[].match.accountId` ကို အသုံးပြုပါ။

### Group chat mention စစ်ဆေးခြင်း

Group မက်ဆေ့ချ်များသည် မူလအနေဖြင့် **mention လိုအပ်သည်** (metadata mention သို့မဟုတ် regex pattern များ)။ WhatsApp, Telegram, Discord, Google Chat နှင့် iMessage group chat များတွင် သက်ဆိုင်သည်။

**Mention အမျိုးအစားများ:**

- **Metadata mentions**: ပလက်ဖောင်း၏ မူရင်း @-mention များ။ WhatsApp self-chat mode တွင် လျစ်လျူရှုသည်။
- **စာသား pattern များ**: `agents.list[].groupChat.mentionPatterns` အတွင်းရှိ Regex pattern များ။ အမြဲစစ်ဆေးသည်။
- (မူရင်း mention များ သို့မဟုတ် pattern အနည်းဆုံးတစ်ခု ရှိသည့်အခါ) ရှာဖွေတွေ့ရှိနိုင်သည့်အခြေအနေတွင်သာ mention စစ်ဆေးခြင်းကို လိုက်နာစေသည်။

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

`messages.groupChat.historyLimit` သည် global မူလသတ်မှတ်ချက်ကို သတ်မှတ်သည်။ Channels များသည် `channels.<channel>` ဖြင့် override လုပ်နိုင်သည်။ `.historyLimit` (သို့မဟုတ် account တစ်ခုချင်းစီအလိုက်)။ ပိတ်ရန် `0` ဟု သတ်မှတ်ပါ။

#### DM history ကန့်သတ်ချက်များ

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

ဖြေရှင်းပုံ: per-DM override → provider မူလသတ်မှတ်ချက် → ကန့်သတ်ချက်မရှိ (အားလုံး သိမ်းဆည်းထားမည်)။

ပံ့ပိုးထားသည်: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`။

#### Self-chat mode

Self-chat mode ကို ဖွင့်ရန် သင့်ဖုန်းနံပါတ်ကို `allowFrom` ထဲတွင် ထည့်ပါ (မူရင်း @-mention များကို လျစ်လျူရှု၍ စာသား pattern များကိုသာ တုံ့ပြန်မည်):

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

### Commands (chat command ကို ကိုင်တွယ်ခြင်း)

```json5
{
  commands: {
    native: "auto", // ပံ့ပိုးထားပါက native command များကို မှတ်ပုံတင်မည်
    text: true, // chat မက်ဆေ့ချ်များထဲရှိ /command များကို parse လုပ်မည်
    bash: false, // ! ကို ခွင့်ပြုမည် (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // /config ကို ခွင့်ပြုမည်
    debug: false, // /debug ကို ခွင့်ပြုမည်
    restart: false, // /restart + gateway restart tool ကို ခွင့်ပြုမည်
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Text command များသည် leading `/` ပါသော **standalone** message များဖြစ်ရမည်။
- `native: "auto"` သည် Discord/Telegram အတွက် native command များကို ဖွင့်ပေးပြီး Slack ကို မဖွင့်ပါ။
- Channel တစ်ခုချင်း override လုပ်ရန်: `channels.discord.commands.native` (bool သို့မဟုတ် `"auto"`)။ `false` သည် ယခင် register လုပ်ထားသော command များကို ရှင်းလင်းဖယ်ရှားပါသည်။
- `channels.telegram.customCommands` သည် Telegram bot menu ထဲသို့ အပို menu entry များ ထည့်သွင်းပေးပါသည်။
- `bash: true` သည် `! <cmd>` ကို host shell အတွက် ဖွင့်ပေးပါသည်။ `tools.elevated.enabled` ကို လိုအပ်ပြီး ပို့သူသည် `tools.elevated.allowFrom.<channel>`.
- `config: true` သည် `/config` ကို ဖွင့်ပေးပြီး (`openclaw.json` ကို ဖတ်/ရေး ပြုလုပ်သည်)။
- `channels.<provider> .configWrites` သည် channel တစ်ခုချင်းအလိုက် config ပြောင်းလဲမှုများကို ထိန်းချုပ်သည် (default: true)။
- `allowFrom` သည် provider တစ်ခုချင်းစီအလိုက် သတ်မှတ်သည်။ သတ်မှတ်ထားပါက ၎င်းသည် တစ်ခုတည်းသော authorization source ဖြစ်ပြီး (channel allowlists/pairing နှင့် `useAccessGroups` ကို လျစ်လျူရှုပါသည်)။
- `useAccessGroups: false` သည် `allowFrom` မသတ်မှတ်ထားပါက command များကို access-group policy များကို ကျော်လွှားခွင့် ပြုပါသည်။

</Accordion>

---

## Agent default များ

### `agents.defaults.workspace`

Default: `~/.openclaw/workspace`။

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

System prompt ၏ Runtime line တွင် ပြသမည့် optional repository root ဖြစ်သည်။ မသတ်မှတ်ထားပါက OpenClaw သည် workspace မှ အပေါ်ဘက်သို့ စစ်ဆေးရှာဖွေကာ အလိုအလျောက် သတ်မှတ်ပေးပါသည်။

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Workspace bootstrap file များ (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`) ကို အလိုအလျောက် ဖန်တီးခြင်းကို ပိတ်ပါသည်။

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Truncation မလုပ်မီ workspace bootstrap file တစ်ခုချင်းစီအတွက် အများဆုံး character အရေအတွက်။ Default: `20000`။

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Workspace bootstrap file များအားလုံးအတွက် စုစုပေါင်း inject လုပ်နိုင်သော အများဆုံး character အရေအတွက်။ Default: `24000`။

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

System prompt context အတွက် timezone (message timestamp မဟုတ်ပါ)။ Host ၏ timezone ကို အစားထိုးအသုံးပြုပါသည်။

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

System prompt အတွင်း အသုံးပြုမည့် အချိန် format။ မူလတန်ဖိုး: `auto` (OS ဦးစားပေးမှု)။

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

- `model.primary`: ပုံစံမှာ `provider/model` ဖြစ်ရမည် (ဥပမာ `anthropic/claude-opus-4-6`)။ provider ကို မသတ်မှတ်ပါက OpenClaw သည် `anthropic` ဟု ယူဆပါမည် (deprecated)။
- `models`: `/model` အတွက် သတ်မှတ်ထားသော model စာရင်းနှင့် allowlist ဖြစ်သည်။ entry တစ်ခုချင်းစီတွင် `alias` (shortcut) နှင့် `params` (provider အလိုက် သတ်မှတ်ချက်များ: `temperature`, `maxTokens`) ပါဝင်နိုင်သည်။
- `imageModel`: primary model တွင် image input မရှိပါကသာ အသုံးပြုသည်။
- `maxConcurrent`: session များအနှံ့ agent ကို တပြိုင်နက် လည်ပတ်နိုင်သည့် အများဆုံးအရေအတွက် (session တစ်ခုချင်းစီအတွင်းတွင်တော့ serialized ဖြစ်နေမည်)။ မူလတန်ဖိုး: 1။

**Built-in alias အတိုကောက်များ** (`agents.defaults.models` တွင် model ပါရှိသည့်အခါသာ သက်ရောက်သည်):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

သင် သတ်မှတ်ထားသော alias များသည် မူလသတ်မှတ်ချက်များထက် အမြဲ ဦးစားပေးမည်ဖြစ်သည်။

Z.AI GLM-4.x model များသည် `--thinking off` ဟု သတ်မှတ်ထားခြင်း သို့မဟုတ် `agents.defaults.models["zai/<model>"].params.thinking` ကို ကိုယ်တိုင် သတ်မှတ်ထားခြင်း မရှိပါက thinking mode ကို အလိုအလျောက် ဖွင့်ပေးမည်ဖြစ်သည်။

### `agents.defaults.cliBackends`

text-only fallback run များအတွက် (tool calls မပါဝင်) ရွေးချယ်နိုင်သော CLI backends များ။ API provider များ ပျက်ကွက်သည့်အခါ backup အဖြစ် အသုံးဝင်သည်။

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

- CLI backend များသည် text ကို ဦးစားပေးပြီး tools များကို အမြဲ ပိတ်ထားသည်။
- `sessionArg` သတ်မှတ်ထားပါက session များကို ပံ့ပိုးပေးသည်။
- `imageArg` သည် file path များကို လက်ခံနိုင်ပါက image pass-through ကို ပံ့ပိုးပေးသည်။

### `agents.defaults.heartbeat`

အချိန်ကာလအလိုက် လည်ပတ်သော heartbeat run များ။

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

- `every`: အချိန်ကြာချိန်ကို ဖော်ပြသော string (ms/s/m/h)။ မူလတန်ဖိုး: `30m`.
- အေးဂျင့်တစ်ခုချင်းစီအတွက် `agents.list[].heartbeat` ကို သတ်မှတ်ပါ။ မည်သည့်အေးဂျင့်မဆို `heartbeat` ကို သတ်မှတ်ထားပါက **ထိုအေးဂျင့်များသာလျှင်** heartbeat ကို လုပ်ဆောင်မည်ဖြစ်သည်။
- Heartbeats များသည် အေးဂျင့်၏ လုပ်ဆောင်မှုအပြည့်အစုံကို လည်ပတ်စေသည် — အချိန်အပိုင်းအခြား ပိုတိုလေလေ token များ ပိုမိုအသုံးပြုမည်ဖြစ်သည်။

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

- `mode` - `default` သို့မဟုတ် `safeguard` (ရှည်လျားသော history များအတွက် အပိုင်းလိုက် အကျဉ်းချုပ်ခြင်း)။ [Compaction](/concepts/compaction) ကို ကြည့်ပါ။
- `memoryFlush` - အလိုအလျောက် compaction မတိုင်မီ တိတ်ဆိတ်သော agentic turn တစ်ခုဖြင့် ရေရှည်သိမ်းဆည်းရန်လိုအပ်သော memory များကို သိမ်းဆည်းခြင်း။ workspace သည် read-only ဖြစ်ပါက ကျော်လွှားမည်။

### `agents.defaults.contextPruning`

LLM သို့ မပို့မီ in-memory context ထဲမှ **tool result အဟောင်းများ** ကို ဖယ်ရှားပေးသည်။ disk ပေါ်ရှိ session history ကို **မပြောင်းလဲပါ**။

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

- `mode: "cache-ttl"` သည် pruning pass များကို ဖွင့်ပေးသည်။
- `ttl` သည် (နောက်ဆုံး cache touch ပြီးနောက်) pruning ကို ထပ်မံ လုပ်ဆောင်နိုင်မည့် အချိန်အကြာကို ထိန်းချုပ်သည်။
- Pruning သည် အရွယ်အစားကြီးမားသော tool result များကို အရင်ဆုံး soft-trim ပြုလုပ်ပြီး၊ လိုအပ်ပါက အဟောင်းများကို hard-clear ပြုလုပ်သည်။

**Soft-trim** သည် အစနှင့် အဆုံးပိုင်းကို ထိန်းသိမ်းထားပြီး အလယ်တွင် `...` ကို ထည့်သွင်းသည်။

**Hard-clear** သည် tool result တစ်ခုလုံးကို placeholder ဖြင့် အစားထိုးသည်။

မှတ်ချက်များ-

- Image block များကို မည်သည့်အခါမျှ trim/clear မလုပ်ပါ။
- Ratio များသည် character အရေအတွက်အပေါ် အခြေခံထားသော (ခန့်မှန်းခြေ) ဖြစ်ပြီး token အရေအတွက် အတိအကျ မဟုတ်ပါ။
- `keepLastAssistants` ထက်နည်းသော assistant message များသာရှိပါက pruning ကို ကျော်လွှားမည်။

</Accordion>

လုပ်ဆောင်ပုံ အသေးစိတ်အတွက် [Session Pruning](/concepts/session-pruning) ကို ကြည့်ပါ။

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

- Non-Telegram channel များတွင် block reply ကို ဖွင့်ရန် `*.blockStreaming: true` ကို သီးသန့် သတ်မှတ်ရမည်။
- Channel override များ- `channels.<channel>``.blockStreamingCoalesce` (နှင့် account တစ်ခုချင်းစီအလိုက် variant များ)။ Signal/Slack/Discord/Google Chat တွင် default `minChars: 1500` ဖြစ်သည်။
- `humanDelay` - block reply များအကြား အချိန်ရပ်နားမှုကို ကျပန်း သတ်မှတ်ပေးသည်။ `natural` = 800–2500ms။ အေးဂျင့်တစ်ခုချင်းစီအတွက် override - `agents.list[].humanDelay`။

လုပ်ဆောင်ပုံနှင့် chunking အသေးစိတ်အတွက် [Streaming](/concepts/streaming) ကို ကြည့်ပါ။

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

- Default အနေဖြင့် direct chat/mention များအတွက် `instant` ဖြစ်ပြီး mention မပါသော group chat များအတွက် `message` ဖြစ်သည်။
- Session တစ်ခုချင်းစီအတွက် override များ- `session.typingMode`, `session.typingIntervalSeconds`။

[Typing Indicators](/concepts/typing-indicators) ကို ကြည့်ပါ။

### `agents.defaults.sandbox`

embedded agent အတွက် ရွေးချယ်နိုင်သော **Docker sandboxing** ဖြစ်သည်။ အသေးစိတ်လမ်းညွှန်အပြည့်အစုံအတွက် [Sandboxing](/gateway/sandboxing) ကို ကြည့်ပါ။

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

**Workspace access:**

- `none`: `~/.openclaw/sandboxes` အောက်ရှိ scope တစ်ခုချင်းစီအလိုက် sandbox workspace
- `ro`: sandbox workspace ကို `/workspace` တွင်ထားပြီး agent workspace ကို `/agent` တွင် read-only အဖြစ် mount လုပ်ထားသည်
- `rw`: agent workspace ကို `/workspace` တွင် read/write အဖြစ် mount လုပ်ထားသည်

**Scope:**

- `session`: session တစ်ခုချင်းစီအတွက် container + workspace
- `agent`: agent တစ်ခုချင်းစီအတွက် container + workspace တစ်ခု (default)
- `shared`: container နှင့် workspace ကို မျှဝေသုံးခြင်း (session များအကြား isolation မရှိပါ)

**`setupCommand`** သည် container ဖန်တီးပြီးနောက် တစ်ကြိမ်သာ (`sh -lc` ဖြင့်) လည်ပတ်သည်။ Network egress လိုအပ်ပြီး root ကို ရေးသားနိုင်ရမည်၊ root user လိုအပ်သည်။

**Containers များ၏ default သည် `network: "none"` ဖြစ်သည်** — agent မှ outbound access လိုအပ်ပါက `"bridge"` ဟု သတ်မှတ်ပါ။

**Inbound attachments** များကို လက်ရှိအသုံးပြုနေသော workspace အတွင်းရှိ `media/inbound/*` ထဲသို့ ထည့်သွင်းထားသည်။

**`docker.binds`** သည် host directory များကို ထပ်မံ mount လုပ်ရန် အသုံးပြုသည်; global နှင့် per-agent binds များကို ပေါင်းစည်းထားသည်။

**Sandboxed browser** (`sandbox.browser.enabled`): container အတွင်းရှိ Chromium + CDP။ noVNC URL ကို system prompt ထဲသို့ ထည့်သွင်းပေးထားသည်။ main config ထဲတွင် `browser.enabled` သတ်မှတ်ရန် မလိုအပ်ပါ။

- `allowHostControl: false` (default) သည် sandboxed sessions များမှ host browser ကို ထိန်းချုပ်ရန် ကြိုးစားမှုကို ပိတ်ပင်ထားသည်။
- `sandbox.browser.binds` သည် host directory များကို sandbox browser container ထဲသို့သာ mount လုပ်သည်။ သတ်မှတ်ထားပါက (`[]` အပါအဝင်) browser container အတွက် `docker.binds` ကို အစားထိုးအသုံးပြုမည်။

</Accordion>

Image များကို build လုပ်ပါ:

```bash
scripts/sandbox-setup.sh           # main sandbox image
scripts/sandbox-browser-setup.sh   # optional browser image
```

### `agents.list` (agent တစ်ခုချင်းစီအလိုက် override များ)

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

- `id`: တည်ငြိမ်သော agent id (လိုအပ်သည်)။
- `default`: အများအပြား သတ်မှတ်ထားပါက ပထမဆုံးတစ်ခုကို အသုံးပြုမည် (warning ကို log ထဲတွင် မှတ်တမ်းတင်မည်)။ မည်သည့်အရာမှ မသတ်မှတ်ထားပါက list ထဲရှိ ပထမဆုံး entry ကို default အဖြစ် သတ်မှတ်မည်။
- `model`: string ပုံစံသည် `primary` ကိုသာ override လုပ်သည်; object ပုံစံ `{ primary, fallbacks }` သည် နှစ်ခုလုံးကို override လုပ်သည် (`[]` သတ်မှတ်ပါက global fallbacks ကို ပိတ်မည်)။
- `identity.avatar`: workspace-relative path၊ `http(s)` URL သို့မဟုတ် `data:` URI။
- `identity` သည် default များကို ဆင်းသက်သတ်မှတ်ပေးသည်: `ackReaction` ကို `emoji` မှ၊ `mentionPatterns` ကို `name`/`emoji` မှ။
- `subagents.allowAgents`: `sessions_spawn` အတွက် ခွင့်ပြုထားသော agent id များ၏ allowlist (`["*"]` = မည်သည့် agent မဆို; default: တူညီသော agent သာ)။

---

## Multi-agent routing

Gateway တစ်ခုအတွင်း သီးခြားထားရှိသော agent များကို အများအပြား လည်ပတ်နိုင်သည်။ [Multi-Agent](/concepts/multi-agent) ကို ကြည့်ပါ။

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

### Binding match fields

- `match.channel` (လိုအပ်သည်)
- `match.accountId` (ရွေးချယ်နိုင်သည်; `*` = မည်သည့် account မဆို; မထည့်ပါက = default account)
- `match.peer` (ရွေးချယ်နိုင်သည်; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (optional; channel-specific)

**တိကျသေချာသော match အစီအစဉ်:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (တိတိကျကျ၊ peer/guild/team မပါ)
5. `match.accountId: "*"` (channel တစ်ခုလုံး)
6. မူလ agent

အဆင့်တစ်ခုချင်းစီအတွင်း ပထမဆုံး ကိုက်ညီသည့် `bindings` entry ကို ဦးစားပေးမည်။

### agent တစ်ခုချင်းစီအတွက် access profile များ

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

ဦးစားပေးစည်းမျဉ်း အသေးစိတ်အတွက် [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) ကို ကြည့်ပါ။

---

## စက်ရှင်

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

- **`dmScope`**: DM များကို မည်သို့ အုပ်စုဖွဲ့မည်ကို သတ်မှတ်သည်။
  - `main`: DM အားလုံးသည် main session ကို မျှဝေသုံးစွဲသည်။
  - `per-peer`: channel များတစ်လျှောက် sender id အလိုက် သီးခြားခွဲထားသည်။
  - `per-channel-peer`: channel + sender အလိုက် သီးခြားခွဲထားသည် (multi-user inbox များအတွက် အကြံပြုသည်)။
  - `per-account-channel-peer`: account + channel + sender အလိုက် သီးခြားခွဲထားသည် (multi-account အတွက် အကြံပြုသည်)။
- **`identityLinks`**: channel များအကြား session မျှဝေရန် canonical id များကို provider-prefix ပါသော peer များနှင့် ချိတ်ဆက်သတ်မှတ်သည်။
- **`reset`**: အဓိက reset မူဝါဒ။ `daily` သည် ဒေသစံအချိန် `atHour` တွင် reset လုပ်သည်; `idle` သည် `idleMinutes` ပြည့်လျှင် reset လုပ်သည်။ နှစ်ခုစလုံး သတ်မှတ်ထားပါက မည်သည့်အရာ အရင်ဆုံး သက်တမ်းကုန်ဆုံးသလဲဆိုသည်ကို ဦးစားပေးမည်။
- **`resetByType`**: အမျိုးအစားအလိုက် override များ (`direct`, `group`, `thread`)။ Legacy `dm` ကို `direct` ၏ အစားထိုးအမည်အဖြစ် လက်ခံသည်။
- **`mainKey`**: legacy field ဖြစ်သည်။ Runtime သည် ယခုအခါ main direct-chat bucket အတွက် အမြဲတမ်း `"main"` ကိုသာ အသုံးပြုသည်။
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`, legacy `dm` alias ပါဝင်), `keyPrefix`, သို့မဟုတ် `rawKeyPrefix` အလိုက် ကိုက်ညီစစ်ဆေးသည်။ ပထမဆုံး deny ကို ဦးစားပေးမည်။
- **`maintenance`**: `warn` သည် session ဖယ်ရှားမည့်အချိန်တွင် သတိပေးမည်; `enforce` သည် pruning နှင့် rotation ကို အတည်ပြု လုပ်ဆောင်မည်။

</Accordion>

---

## မက်ဆေ့ချ်များ

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

### တုံ့ပြန်မှု အစပြုစာသား

channel/account အလိုက် override များ: `channels.<channel>`.responsePrefix`, `channels.<channel>`.accounts.<id>`.responsePrefix\`.

ဖြေရှင်းရာတွင် (အသေးစိတ်ဆုံးကို ဦးစားပေးသည်): account → channel → global။ `""` သည် cascade ကို ပိတ်ပြီး ရပ်တန့်စေသည်။ `"auto"` သည် `[{identity.name}]` ကို အလိုအလျောက် ရယူသတ်မှတ်သည်။

**Template variables:**

| Variable          | ဖော်ပြချက်                    | ဥပမာ                                        |
| ----------------- | ----------------------------- | ------------------------------------------- |
| `{model}`         | မော်ဒယ်အမည် အတိုကောက်         | `claude-opus-4-6`                           |
| `{modelFull}`     | မော်ဒယ် identifier အပြည့်အစုံ | `anthropic/claude-opus-4-6`                 |
| `{provider}`      | Provider အမည်                 | `anthropic`                                 |
| `{thinkingLevel}` | လက်ရှိ စဉ်းစားမှုအဆင့်        | `high`, `low`, `off`                        |
| `{identity.name}` | Agent identity အမည်           | (`"auto"` နှင့် တူညီသည်) |

Variables များသည် အက္ခရာအကြီးအသေး မခွဲခြားပါ။ `{think}` သည် `{thinkingLevel}` ၏ အစားထိုးအမည်ဖြစ်သည်။

### Ack reaction

- မူလအားဖြင့် လက်ရှိ active agent ၏ `identity.emoji` ကို အသုံးပြုသည်၊ မရှိပါက `"👀"` ဖြစ်သည်။ ပိတ်လိုပါက `""` ဟု သတ်မှတ်ပါ။
- Channel အလိုက် override များ - `channels.<channel>
   .ackReaction`, `channels.<channel>
   .accounts.<id>
   .ackReaction`.
- Resolution အစဉ် - account → channel → `messages.ackReaction` → identity fallback။
- Scope - `group-mentions` (မူလသတ်မှတ်ချက်), `group-all`, `direct`, `all`။
- `removeAckAfterReply` - reply ပြန်ပြီးနောက် ack ကို ဖယ်ရှားသည် (Slack/Discord/Telegram/Google Chat တွင်သာ)။

### Inbound debounce

ပို့သူတစ်ဦးတည်းမှ အချိန်တိုအတွင်း ဆက်တိုက်ပို့သော စာသားမက်ဆေ့ချ်များကို agent turn တစ်ခုတည်းအဖြစ် ပေါင်းစည်းပေးသည်။ Media/attachments များကို ချက်ချင်း လုပ်ဆောင်သည်။ Control commands များသည် debouncing ကို မကျော်လွှားဘဲ တိုက်ရိုက် လုပ်ဆောင်သည်။

### TTS (text-to-speech)

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

- `auto` သည် auto-TTS ကို ထိန်းချုပ်သည်။ `/tts off|always|inbound|tagged` ကို session တစ်ခုချင်းအလိုက် override လုပ်နိုင်သည်။
- `summaryModel` သည် auto-summary အတွက် `agents.defaults.model.primary` ကို override လုပ်သည်။
- API keys များသည် `ELEVENLABS_API_KEY`/`XI_API_KEY` နှင့် `OPENAI_API_KEY` သို့ fallback လုပ်သည်။

---

## Talk

Talk mode (macOS/iOS/Android) အတွက် မူလသတ်မှတ်ချက်များ။

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

- Voice ID များသည် `ELEVENLABS_VOICE_ID` သို့မဟုတ် `SAG_VOICE_ID` သို့ fallback လုပ်သည်။
- `apiKey` သည် `ELEVENLABS_API_KEY` သို့ fallback လုပ်သည်။
- `voiceAliases` သည် Talk directives များတွင် အသုံးပြုရန် friendly name များ သတ်မှတ်နိုင်စေသည်။

---

## Tools

### Tool profile များ

`tools.profile` သည် `tools.allow`/`tools.deny` မတိုင်မီ အခြေခံ allowlist တစ်ခုကို သတ်မှတ်ပေးသည်။

| Profile     | ပါဝင်သည်များ                                                                              |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | `session_status` သာလျှင်                                                                  |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | ကန့်သတ်ချက်မရှိ (မသတ်မှတ်ထားသကဲ့သို့ တူညီသည်)                          |

### Tool group များ

| Group              | Tools                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` ကို `exec` အတွက် alias အဖြစ် လက်ခံသည်)      |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | ပါဝင်ပြီးသား tool အားလုံး (provider plugins မပါဝင်ပါ)                 |

### `tools.allow` / `tools.deny`

Global tool allow/deny policy (deny ကို ဦးစားပေးမည်)။ စာလုံးအကြီးအသေး မခွဲခြားပါ၊ `*` wildcards ကို ထောက်ပံ့သည်။ Docker sandbox ကို ပိတ်ထားသော်လည်း အသုံးချမည်။

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

သတ်မှတ်ထားသော provider များ သို့မဟုတ် model များအတွက် tool များကို ထပ်မံ ကန့်သတ်ရန်။ အစဉ်क्रम: base profile → provider profile → allow/deny။

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

elevated (host) exec access ကို ထိန်းချုပ်ရန်:

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

- Per-agent override (`agents.list[].tools.elevated`) သည် ထပ်မံကန့်သတ်ခြင်းသာ ပြုလုပ်နိုင်သည်။
- `/elevated on|off|ask|full` သည် session တစ်ခုချင်းစီအလိုက် အခြေအနေကို သိမ်းဆည်းထားသည်; inline directives များသည် message တစ်ခုတည်းအတွက်သာ သက်ရောက်သည်။
- Elevated `exec` သည် host ပေါ်တွင် လည်ပတ်ပြီး sandboxing ကို ကျော်လွှားသည်။

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
        apiKey: "brave_api_key", // သို့မဟုတ် BRAVE_API_KEY env
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

ဝင်ရောက်လာသော media နားလည်မှု (image/audio/video) ကို ပြင်ဆင်ရန်:

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

**Provider entry** (`type: "provider"` သို့မဟုတ် မသတ်မှတ်ထားပါက):

- `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, စသည်)
- `model`: model id ကို override လုပ်ရန်
- `profile` / `preferredProfile`: auth profile ရွေးချယ်မှု

**CLI entry** (`type: "cli"`):

- `command`: လည်ပတ်ရန် executable
- `args`: template ပြုလုပ်ထားသော args (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` စသည်တို့ကို ထောက်ပံ့သည်)

**Common fields:**

- `capabilities`: ရွေးချယ်နိုင်သော စာရင်း (`image`, `audio`, `video`)။ မူလသတ်မှတ်ချက်များ: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio။
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: entry တစ်ခုချင်းစီအလိုက် override ပြုလုပ်နိုင်သည်။
- မအောင်မြင်ပါက နောက် entry သို့ ပြန်လည် fallback လုပ်မည်။

Provider auth သည် ပုံမှန်အစဉ်အတိုင်း လိုက်နာသည်: auth profiles → env vars → `models.providers.*.apiKey`။

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

- `model`: ဖန်တီးထားသော sub-agent များအတွက် ပုံမှန်အသုံးပြုမည့် model။ မသတ်မှတ်ထားပါက sub-agent များသည် ခေါ်ယူသည့် agent ၏ model ကို အမွေဆက်ခံအသုံးပြုမည်ဖြစ်သည်။
- subagent တစ်ခုချင်းစီအလိုက် tool policy သတ်မှတ်ခြင်း: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`။

---

## စိတ်ကြိုက် providers နှင့် base URL များ

OpenClaw သည် pi-coding-agent model catalog ကို အသုံးပြုသည်။ config ထဲရှိ `models.providers` သို့မဟုတ် `~/.openclaw/agents/<agentId>/agent/models.json` မှတစ်ဆင့် စိတ်ကြိုက် providers များ ထည့်သွင်းနိုင်သည်။

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

- စိတ်ကြိုက် authentication လိုအပ်ချက်များအတွက် `authHeader: true` နှင့် `headers` ကို အသုံးပြုပါ။
- `OPENCLAW_AGENT_DIR` (သို့) `PI_CODING_AGENT_DIR` ဖြင့် agent config root ကို ပြောင်းလဲသတ်မှတ်နိုင်သည်။

### Provider ဥပမာများ

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

Cerebras အတွက် `cerebras/zai-glm-4.7` ကို အသုံးပြုပါ။ Z.AI ကို တိုက်ရိုက်အသုံးပြုရန် `zai/glm-4.7` ကို အသုံးပြုပါ။

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

`OPENCODE_API_KEY` (သို့) `OPENCODE_ZEN_API_KEY` ကို သတ်မှတ်ပါ။ Shortcut: `openclaw onboard --auth-choice opencode-zen`။

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

`ZAI_API_KEY` ကို သတ်မှတ်ပါ။ `z.ai/*` နှင့် `z-ai/*` ကို alias အဖြစ် လက်ခံသည်။ Shortcut: `openclaw onboard --auth-choice zai-api-key`။

- အထွေထွေ endpoint: `https://api.z.ai/api/paas/v4`
- Coding endpoint (ပုံမှန်): `https://api.z.ai/api/coding/paas/v4`
- အထွေထွေ endpoint အတွက် base URL ကို override လုပ်ထားသော custom provider တစ်ခု သတ်မှတ်ပါ။

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

China endpoint အတွက်: `baseUrl: "https://api.moonshot.cn/v1"` သို့မဟုတ် `openclaw onboard --auth-choice moonshot-api-key-cn` ကို အသုံးပြုပါ။

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

Anthropic-compatible ဖြစ်ပြီး built-in provider တစ်ခုဖြစ်သည်။ Shortcut: `openclaw onboard --auth-choice kimi-code-api-key`။

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

Base URL တွင် `/v1` ကို မထည့်ပါနှင့် (Anthropic client က အလိုအလျောက် ထည့်သွင်းပေးသည်)။ Shortcut: `openclaw onboard --auth-choice synthetic-api-key`။

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

`MINIMAX_API_KEY` ကို သတ်မှတ်ပါ။ Shortcut: `openclaw onboard --auth-choice minimax-api`။

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) ကို ကြည့်ပါ။ အကျဉ်းချုပ်: စွမ်းဆောင်ရည်မြင့် hardware ပေါ်တွင် LM Studio Responses API မှတစ်ဆင့် MiniMax M2.1 ကို run လုပ်ပါ။ အစားထိုးရန် fallback အတွက် hosted models များကို merge အနေဖြင့် ထားရှိပါ။

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

- `allowBundled`: bundle လုပ်ထားသော skills များအတွက်သာ ရွေးချယ်ခွင့်ရှိသော allowlist (managed/workspace skills များမပါဝင်ပါ)။
- `entries.<skillKey>`.enabled: false\` သည် skill ကို bundled/installed ဖြစ်နေသော်လည်း ပိတ်ထားစေသည်။
- `entries.<skillKey>`.apiKey\`: အဓိက env var ကို သတ်မှတ်ထားသော skills များအတွက် အဆင်ပြေစေသော option။

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

- `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` နှင့် `plugins.load.paths` မှ load လုပ်သည်။
- **Config ပြောင်းလဲမှုများအတွက် gateway ကို restart ပြန်လုပ်ရန်လိုအပ်သည်။**
- `allow`: ရွေးချယ်ထားသော plugins များသာ load လုပ်ရန် optional allowlist။ `deny` သည် ဦးစားပေး အကျုံးဝင်သည်။

[Plugins](/tools/plugin) ကိုကြည့်ပါ။

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

- `evaluateEnabled: false` သည် `act:evaluate` နှင့် `wait --fn` ကို ပိတ်ထားစေသည်။
- Remote profiles များသည် attach-only ဖြစ်ပြီး (start/stop/reset ကို မပြုလုပ်နိုင်ပါ)။
- Auto-detect အစီအစဉ်: default browser (Chromium-based ဖြစ်ပါက) → Chrome → Brave → Edge → Chromium → Chrome Canary။
- Control service: loopback သာ (port ကို `gateway.port` မှ ရယူပြီး default သည် `18791`)။

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

- `seamColor`: native app UI chrome အတွက် accent color (Talk Mode bubble tint စသည်)။
- `assistant`: Control UI ၏ identity ကို override လုပ်ရန်။ လက်ရှိအသုံးပြုနေသော agent identity ကို fallback အဖြစ် အသုံးပြုသည်။

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

- `mode`: `local` (gateway ကို run လုပ်ရန်) သို့မဟုတ် `remote` (remote gateway သို့ ချိတ်ဆက်ရန်)။ `local` မဟုတ်ပါက Gateway သည် စတင်ရန် ငြင်းဆန်မည်ဖြစ်သည်။
- `port`: WS + HTTP အတွက် တစ်ခုတည်းသော multiplexed port။ ဦးစားပေးအစီအစဉ်: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`။
- `bind`: `auto`, `loopback` (default), `lan` (`0.0.0.0`), `tailnet` (Tailscale IP သာ), သို့မဟုတ် `custom`။
- **Auth**: မူလအားဖြင့် လိုအပ်သည်။ loopback မဟုတ်သော bind များတွင် shared token/password လိုအပ်သည်။ Onboarding wizard သည် မူလအားဖြင့် token ကို ဖန်တီးပေးသည်။
- `auth.mode: "trusted-proxy"`: identity-aware reverse proxy ထံသို့ auth ကို လွှဲပြောင်းပြီး `gateway.trustedProxies` မှ identity headers များကို ယုံကြည်သည် ([Trusted Proxy Auth](/gateway/trusted-proxy-auth) ကိုကြည့်ပါ)။
- `auth.allowTailscale`: `true` ဖြစ်ပါက Tailscale Serve identity headers များကို auth အဖြစ် လက်ခံသည် (`tailscale whois` ဖြင့် စစ်ဆေးသည်)။ `tailscale.mode = "serve"` ဖြစ်သည့်အခါ မူလတန်ဖိုးမှာ `true` ဖြစ်သည်။
- `auth.rateLimit`: failed-auth ကို ကန့်သတ်ရန် optional limiter။ client IP တစ်ခုချင်းစီနှင့် auth scope တစ်ခုချင်းစီအလိုက် သက်ရောက်သည် (shared-secret နှင့် device-token ကို သီးခြားစီ ခြေရာခံသည်)။ ပိတ်ဆို့ထားသော ကြိုးပမ်းမှုများသည် `429` + `Retry-After` ကို ပြန်ပေးသည်။
  - `auth.rateLimit.exemptLoopback` ၏ မူလတန်ဖိုးမှာ `true` ဖြစ်သည်။ localhost traffic ကိုပါ rate limit လုပ်လိုသော (test setup များ သို့မဟုတ် strict proxy deployment များအတွက်) အခြေအနေတွင် `false` ဟု သတ်မှတ်ပါ။
- `tailscale.mode`: `serve` (tailnet သာ၊ loopback bind) သို့မဟုတ် `funnel` (public, auth လိုအပ်သည်)။
- `remote.transport`: `ssh` (မူလတန်ဖိုး) သို့မဟုတ် `direct` (ws/wss)။ `direct` ကိုအသုံးပြုပါက `remote.url` သည် `ws://` သို့မဟုတ် `wss://` ဖြစ်ရမည်။
- `gateway.remote.token` သည် remote CLI call များအတွက်သာ ဖြစ်ပြီး local gateway auth ကို မဖွင့်ပေးပါ။
- `trustedProxies`: TLS ကို terminate လုပ်သော reverse proxy IP များ။ သင်ထိန်းချုပ်ထားသော proxy များကိုသာ စာရင်းပြုစုပါ။
- `gateway.tools.deny`: HTTP `POST /tools/invoke` အတွက် ပိတ်ပင်ထားမည့် tool name များကို ထပ်မံသတ်မှတ်ခြင်း (မူလ deny list ကို တိုးချဲ့သည်)။
- `gateway.tools.allow`: မူလ HTTP deny list မှ tool name များကို ဖယ်ရှားရန်။

</Accordion>

### OpenAI-compatible endpoints

- Chat Completions: မူလအားဖြင့် ပိတ်ထားသည်။ `gateway.http.endpoints.chatCompletions.enabled: true` ဖြင့် ဖွင့်နိုင်သည်။
- Responses API: `gateway.http.endpoints.responses.enabled`။
- Responses URL-input လုံခြုံရေးတိုးမြှင့်ခြင်း:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Multi-instance ခွဲခြားအသုံးပြုခြင်း

Host တစ်ခုတည်းပေါ်တွင် port နှင့် state dir မတူညီစွာ သတ်မှတ်၍ gateway များစွာကို လည်ပတ်နိုင်သည်:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

အဆင်ပြေစေရန် flags များ: `--dev` (`~/.openclaw-dev` + port `19001` ကိုအသုံးပြုသည်), `--profile <name>` (`~/.openclaw-<name>` ကိုအသုံးပြုသည်)။

[Multiple Gateways](/gateway/multiple-gateways) ကိုကြည့်ပါ။

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

Auth: `Authorization: Bearer <token>` သို့မဟုတ် `x-openclaw-token: <token>`။

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` ကို request payload မှ လက်ခံမည်မှာ `hooks.allowRequestSessionKey=true` ဖြစ်သောအခါတွင်သာ ဖြစ်သည် (မူလတန်ဖိုး: `false`)။
- `POST /hooks/<name>` → `hooks.mappings` ဖြင့် ဆုံးဖြတ်သည်

<Accordion title="Mapping details">

- `match.path` သည် `/hooks` ပြီးနောက်ရှိ sub-path ကို ကိုက်ညီစေသည် (ဥပမာ `/hooks/gmail` → `gmail`)။
- `match.source` သည် generic path များအတွက် payload field တစ်ခုနှင့် ကိုက်ညီစေသည်။
- `{{messages[0].subject}}` ကဲ့သို့သော template များသည် payload မှ ဖတ်ယူသည်။
- `transform` သည် hook action ကို ပြန်ပေးသော JS/TS module ကို ညွှန်းနိုင်သည်။
  - `transform.module` သည် relative path ဖြစ်ရမည်ဖြစ်ပြီး `hooks.transformsDir` အတွင်းတွင်သာ ရှိရမည် (absolute path များနှင့် traversal ကို လက်မခံပါ)။
- `agentId` သည် သတ်မှတ်ထားသော agent တစ်ခုသို့ route လုပ်သည်။ မသိရှိသော ID များကို မူလတန်ဖိုးသို့ ပြန်လည်အသုံးပြုမည်။
- `allowedAgentIds`: explicit routing ကို ကန့်သတ်သည် (`*` သို့မဟုတ် မသတ်မှတ်ထားပါက = အားလုံးကို ခွင့်ပြု, `[]` = အားလုံးကို ပိတ်ပင်)။
- `defaultSessionKey`: `sessionKey` ကို မသတ်မှတ်ထားသော hook agent run များအတွက် ရွေးချယ်နိုင်သော fixed session key။
- `allowRequestSessionKey`: `/hooks/agent` ခေါ်ယူသူများအား `sessionKey` သတ်မှတ်ခွင့်ပေးရန် (မူလတန်ဖိုး: `false`)။
- `allowedSessionKeyPrefixes`: သတ်မှတ်ထားသော `sessionKey` တန်ဖိုးများအတွက် (request + mapping) optional prefix allowlist ဖြစ်သည်၊ ဥပမာ `[
  "hook:"
  ]`။
- `deliver: true` ကို သတ်မှတ်ပါက နောက်ဆုံး reply ကို channel တစ်ခုသို့ ပို့မည်ဖြစ်သည်။ `channel` ၏ မူလတန်ဖိုးမှာ `last` ဖြစ်သည်။
- `model` သည် ယခု hook run အတွက် LLM ကို override လုပ်ပေးသည် (model catalog သတ်မှတ်ထားပါက ခွင့်ပြုထားသော model ဖြစ်ရမည်)။

</Accordion>

### Gmail ပေါင်းစည်းမှု

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

- Configure လုပ်ထားပါက Gateway သည် boot တက်ချိန်တွင် `gog gmail watch serve` ကို အလိုအလျောက် စတင်မည်ဖြစ်သည်။ Disable လုပ်ရန် `OPENCLAW_SKIP_GMAIL_WATCHER=1` ကို သတ်မှတ်ပါ။
- Gateway နှင့်အတူ `gog gmail watch serve` ကို သီးခြား မလည်ပတ်ပါနှင့်။

---

## Canvas host

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // သို့မဟုတ် OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway port အောက်ရှိ HTTP မှတစ်ဆင့် agent-editable HTML/CSS/JS နှင့် A2UI ကို ပေးဆောင်သည်။
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Local-only: `gateway.bind: "loopback"` (default) ကို ဆက်လက်ထားပါ။
- Non-loopback bind များတွင် canvas route များသည် အခြား Gateway HTTP surface များကဲ့သို့ Gateway auth (token/password/trusted-proxy) လိုအပ်သည်။
- Node WebViews များသည် ပုံမှန်အားဖြင့် auth header မပို့ကြပါ။ node တစ်ခုကို pair လုပ်ပြီး connect ဖြစ်သည့်နောက် Gateway သည် private-IP fallback ကို ခွင့်ပြုပေးသဖြင့် node သည် URL များအတွင်း secret မဖော်ပြဘဲ canvas/A2UI ကို load လုပ်နိုင်သည်။
- ပေးဆောင်ထားသော HTML ထဲသို့ live-reload client ကို ထည့်သွင်းပေးသည်။
- အလွတ်ဖြစ်နေပါက အခြေခံ `index.html` ကို အလိုအလျောက် ဖန်တီးပေးသည်။
- A2UI ကိုလည်း `/__openclaw__/a2ui/` တွင် ပေးဆောင်သည်။
- ပြောင်းလဲမှုများအတွက် gateway ကို restart လုပ်ရန် လိုအပ်သည်။
- directory အရွယ်အစားကြီးမားပါက သို့မဟုတ် `EMFILE` error များ ဖြစ်ပေါ်ပါက live reload ကို disable လုပ်ပါ။

---

## Discovery

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

- `minimal` (default): TXT record များတွင် `cliPath` နှင့် `sshPort` ကို မထည့်သွင်းပါ။
- `full`: `cliPath` နှင့် `sshPort` ကို ထည့်သွင်းပါ။
- Hostname ၏ မူလတန်ဖိုးမှာ `openclaw` ဖြစ်သည်။ `OPENCLAW_MDNS_HOSTNAME` ဖြင့် override လုပ်နိုင်သည်။

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` အောက်တွင် unicast DNS-SD zone ကို ရေးသားဖန်တီးသည်။ Network မတူညီသည့် အကြား discovery ပြုလုပ်ရန် DNS server (CoreDNS ကို အကြံပြုသည်) နှင့် Tailscale split DNS ကို တွဲဖက်အသုံးပြုပါ။

Setup: `openclaw dns setup --apply`။

---

## Environment

### `env` (inline env vars)

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

- Inline env vars များကို process env တွင် သက်ဆိုင်ရာ key မရှိသည့်အခါတွင်သာ အသုံးပြုမည်ဖြစ်သည်။
- `.env` ဖိုင်များ: CWD `.env` နှင့် `~/.openclaw/.env` (နှစ်ခုစလုံးသည် ရှိပြီးသား vars များကို override မလုပ်ပါ)။
- `shellEnv`: သင့် login shell profile မှ လိုအပ်သည့် key မရှိသေးသော တန်ဖိုးများကို import လုပ်ပေးသည်။
- အပြည့်အစုံ precedence အတွက် [Environment](/help/environment) ကို ကြည့်ပါ။

### Env var အစားထိုးခြင်း

မည်သည့် config string မဆို `${VAR_NAME}` ကို အသုံးပြုပြီး env vars များကို ကိုးကားနိုင်သည်:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- အကြီးစာလုံးသာ ကိုက်ညီသည်: `[A-Z_][A-Z0-9_]*`.
- config ကို load လုပ်စဉ် မရှိသော သို့မဟုတ် အလွတ်ဖြစ်သော var များရှိပါက error ဖြစ်ပေါ်မည်။
- တိတိကျကျ `${VAR}` စာသားအဖြစ် အသုံးပြုလိုပါက `$${VAR}` ဟု escape လုပ်ပါ။
- `$include` နှင့် တွဲဖက်အသုံးပြုနိုင်သည်။

---

## Auth သိမ်းဆည်းမှု

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

- Agent တစ်ခုချင်းစီအတွက် auth profiles များကို `<agentDir>/auth-profiles.json` တွင် သိမ်းဆည်းထားသည်။
- Legacy OAuth ကို `~/.openclaw/credentials/oauth.json` မှ import လုပ်သည်။
- [OAuth](/concepts/oauth) ကို ကြည့်ပါ။

---

## Logging

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

- မူလသတ်မှတ်ထားသော log ဖိုင်: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- တည်ငြိမ်သော path အသုံးပြုရန် `logging.file` ကို သတ်မှတ်ပါ။
- `--verbose` အသုံးပြုသည့်အခါ `consoleLevel` ကို `debug` သို့ မြှင့်တင်သည်။

---

## Wizard

CLI wizard များ (`onboard`, `configure`, `doctor`) မှ ရေးသားထားသော metadata:

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

## Identity

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

macOS onboarding assistant မှ ရေးသားထားသည်။ မူလတန်ဖိုးများကို ဆင်းသက်တွက်ချက်သည်:

- `messages.ackReaction` ကို `identity.emoji` မှ (မရှိပါက 👀 ကို အသုံးပြုမည်)
- `mentionPatterns` ကို `identity.name`/`identity.emoji` မှ ဆင်းသက်တွက်ချက်သည်
- `avatar` သည် အောက်ပါတို့ကို လက်ခံသည်: workspace-relative path, `http(s)` URL, သို့မဟုတ် `data:` URI

---

## Bridge (legacy, ဖယ်ရှားပြီး)

လက်ရှိ build များတွင် TCP bridge မပါဝင်တော့ပါ။ Node များသည် Gateway WebSocket မှတစ်ဆင့် ချိတ်ဆက်သည်။ `bridge.*` keys များသည် config schema ၏ အစိတ်အပိုင်းမဟုတ်တော့ပါ (ဖယ်ရှားမည့်အထိ validation မအောင်မြင်ပါ; `openclaw doctor --fix` ဖြင့် မသိသော keys များကို ဖယ်ရှားနိုင်သည်).

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

- `sessionRetention`: ပြီးဆုံးသွားသော cron sessions များကို ဖျက်ရှားမီ ဘယ်လောက်ကြာ ထိန်းသိမ်းမည်ကို သတ်မှတ်သည်။ မူလတန်ဖိုး: `24h`.

[Cron Jobs](/automation/cron-jobs) ကို ကြည့်ပါ။

---

## Media model template variables

`tools.media.*.models[].args` တွင် template placeholder များကို ဖြန့်ကျက်ထည့်သွင်းသည်:

| Variable           | ဖော်ပြချက်                                                               |
| ------------------ | ------------------------------------------------------------------------ |
| `{{Body}}`         | ဝင်လာသော မက်ဆေ့ချ်၏ အပြည့်အစုံ အကြောင်းအရာ                               |
| `{{RawBody}}`      | မူရင်း အကြောင်းအရာ (history/sender wrappers မပါဝင်ပါ) |
| `{{BodyStripped}}` | အဖွဲ့ mention များကို ဖယ်ရှားထားသော အကြောင်းအရာ                          |
| `{{From}}`         | ပေးပို့သူ အမှတ်အသား                                                      |
| `{{To}}`           | လက်ခံသူ အမှတ်အသား                                                        |
| `{{MessageSid}}`   | ချန်နယ် မက်ဆေ့ချ် ID                                                     |
| `{{SessionId}}`    | လက်ရှိ session UUID                                                      |
| `{{IsNewSession}}` | session အသစ် ဖန်တီးခဲ့ပါက "true"                                         |
| `{{MediaUrl}}`     | ဝင်လာသော မီဒီယာ pseudo-URL                                               |
| `{{MediaPath}}`    | ဒေသတွင်း မီဒီယာ လမ်းကြောင်း                                              |
| `{{MediaType}}`    | မီဒီယာ အမျိုးအစား (image/audio/document/…)            |
| `{{Transcript}}`   | အသံ transcript                                                           |
| `{{Prompt}}`       | CLI entries အတွက် ဖြေရှင်းထားသော media prompt                            |
| `{{MaxChars}}`     | CLI entries အတွက် ဖြေရှင်းထားသော အများဆုံး output စာလုံးအရေအတွက်         |
| `{{ChatType}}`     | "direct" သို့မဟုတ် "group"                                               |
| `{{GroupSubject}}` | အဖွဲ့ ခေါင်းစဉ် (ရနိုင်သလောက်)                        |
| `{{GroupMembers}}` | အဖွဲ့ဝင်များ အကျဉ်းချုပ် (ရနိုင်သလောက်)               |
| `{{SenderName}}`   | ပေးပို့သူ၏ ဖော်ပြအမည် (ရနိုင်သလောက်)                  |
| `{{SenderE164}}`   | ပေးပို့သူ ဖုန်းနံပါတ် (ရနိုင်သလောက်)                  |
| `{{Provider}}`     | Provider hint (whatsapp, telegram, discord, စသည်)     |

---

## Config တွင် (`$include`) ပါဝင်သည်

Config ကို ဖိုင်များစွာအဖြစ် ခွဲရန်:

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

**ပေါင်းစည်းမှု လုပ်ဆောင်ပုံ:**

- ဖိုင်တစ်ခုတည်း - ပါဝင်နေသော object ကို အစားထိုးသည်။
- ဖိုင်အစု (Array) - အစဉ်လိုက် deep-merge လုပ်သည် (နောက်ပိုင်းဖိုင်များက အစောပိုင်းဖိုင်များကို override လုပ်သည်)။
- Sibling keys - includes ပြီးနောက် merge လုပ်သည် (ပါဝင်ထားသော တန်ဖိုးများကို override လုပ်သည်)။
- Nested includes - အများဆုံး အဆင့် 10 ဆင့်အထိ။
- Paths - relative (include လုပ်ထားသော ဖိုင်အပေါ် မူတည်၍), absolute, သို့မဟုတ် `../` parent reference များ။
- Errors - ဖိုင်မတွေ့ရှိခြင်း၊ parse error များနှင့် circular includes များအတွက် ရှင်းလင်းသော message များ ပြသသည်။

---

_Related: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

