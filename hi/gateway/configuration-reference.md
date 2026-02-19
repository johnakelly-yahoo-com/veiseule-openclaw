---
title: "कॉन्फ़िगरेशन संदर्भ"
description: "~/.openclaw/openclaw.json के लिए पूर्ण फ़ील्ड-दर-फ़ील्ड संदर्भ"
---

# कॉन्फ़िगरेशन संदर्भ

`~/.openclaw/openclaw.json` में उपलब्ध प्रत्येक फ़ील्ड। कार्य-उन्मुख अवलोकन के लिए, देखें [Configuration](/gateway/configuration)।

कॉन्फ़िग फ़ॉर्मेट **JSON5** है (comments + trailing commas की अनुमति है)। सभी फ़ील्ड वैकल्पिक हैं — OpenClaw छोड़े जाने पर सुरक्षित डिफ़ॉल्ट्स का उपयोग करता है।

---

## चैनल्स

जब इसकी कॉन्फ़िग सेक्शन मौजूद होती है तो प्रत्येक चैनल स्वतः शुरू हो जाता है (जब तक `enabled: false` न हो)।

### DM और समूह एक्सेस

सभी चैनल DM नीतियों और समूह नीतियों का समर्थन करते हैं:

| DM नीति                                 | व्यवहार                                                                         |
| --------------------------------------- | ------------------------------------------------------------------------------- |
| `pairing` (डिफ़ॉल्ट) | अज्ञात प्रेषकों को एक बार का pairing कोड मिलता है; मालिक को अनुमोदन करना होगा   |
| `allowlist`                             | केवल `allowFrom` (या paired allow store) में मौजूद प्रेषक    |
| `open`                                  | सभी इनबाउंड DMs की अनुमति दें (आवश्यक है `allowFrom: ["*"]`) |
| `disabled`                              | सभी इनबाउंड DMs को अनदेखा करें                                                  |

| समूह नीति                                | व्यवहार                                                                               |
| ---------------------------------------- | ------------------------------------------------------------------------------------- |
| `allowlist` (default) | केवल कॉन्फ़िगर की गई allowlist से मेल खाने वाले समूह                                  |
| `open`                                   | ग्रुप allowlists को बायपास करें (mention-gating अभी भी लागू रहेगा) |
| `disabled`                               | सभी समूह/रूम संदेशों को अवरुद्ध करें                                                  |

<Note>
`channels.defaults.groupPolicy` डिफ़ॉल्ट सेट करता है जब किसी provider का `groupPolicy` सेट नहीं होता है।
पेयरिंग कोड 1 घंटे बाद समाप्त हो जाते हैं। लंबित DM पेयरिंग अनुरोध **प्रति चैनल 3** तक सीमित हैं।
Slack/Discord में एक विशेष फ़ॉलबैक है: यदि उनका provider सेक्शन पूरी तरह से अनुपस्थित है, तो runtime group policy `open` पर सेट हो सकती है (स्टार्टअप चेतावनी के साथ)।
</Note>

### WhatsApp

WhatsApp gateway के web चैनल (Baileys Web) के माध्यम से चलता है। जब लिंक किया हुआ सेशन मौजूद होता है तो यह अपने आप शुरू हो जाता है।

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blue ticks (self-chat मोड में false)
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

- आउटबाउंड कमांड डिफ़ॉल्ट रूप से `default` अकाउंट का उपयोग करते हैं यदि वह मौजूद हो; अन्यथा पहले कॉन्फ़िगर किए गए अकाउंट id (sorted) का।
- Legacy single-account Baileys auth dir को `openclaw doctor` द्वारा `whatsapp/default` में माइग्रेट किया जाता है।
- प्रति-अकाउंट ओवरराइड्स: `channels.whatsapp.accounts.<id> .sendReadReceipts`, `channels.whatsapp.accounts.<id> .dmPolicy`, `channels.whatsapp.accounts.<id> .allowFrom`.

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
          systemPrompt: "उत्तर संक्षिप्त रखें।",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "विषय पर बने रहें।",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git बैकअप" },
        { command: "generate", description: "एक छवि बनाएँ" },
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

- Bot token: `channels.telegram.botToken` या `channels.telegram.tokenFile`, डिफ़ॉल्ट अकाउंट के लिए फ़ॉलबैक के रूप में `TELEGRAM_BOT_TOKEN`।
- `configWrites: false` Telegram द्वारा आरंभ की गई config writes (supergroup ID माइग्रेशन, `/config set|unset`) को अवरुद्ध करता है।
- Telegram stream previews `sendMessage` + `editMessageText` का उपयोग करते हैं (direct और group chats में काम करता है)।
- Retry policy: देखें [Retry policy](/concepts/retry)।

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
              systemPrompt: "केवल संक्षिप्त उत्तर।",
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

- Token: `channels.discord.token`, डिफ़ॉल्ट अकाउंट के लिए फ़ॉलबैक के रूप में `DISCORD_BOT_TOKEN`।
- डिलीवरी टार्गेट के लिए `user:<id>` (DM) या `channel:<id>` (guild channel) का उपयोग करें; केवल संख्यात्मक IDs स्वीकार नहीं किए जाते।
- Guild slugs लोअरकेस होते हैं और स्पेस को `-` से बदला जाता है; चैनल keys slugged नाम का उपयोग करते हैं (`#` के बिना)। Guild IDs को प्राथमिकता दें।
- Bot द्वारा लिखे गए संदेश डिफ़ॉल्ट रूप से अनदेखा किए जाते हैं। `allowBots: true` उन्हें सक्षम करता है (अपने संदेश अभी भी फ़िल्टर होते हैं)।
- `maxLinesPerMessage` (डिफ़ॉल्ट 17) लंबे संदेशों को 2000 अक्षरों से कम होने पर भी विभाजित करता है।
- `channels.discord.ui.components.accentColor` Discord components v2 कंटेनरों के लिए accent color सेट करता है।

**Reaction notification modes:** `off` (कोई नहीं), `own` (बॉट के संदेश, डिफ़ॉल्ट), `all` (सभी संदेश), `allowlist` ( `guilds.<id> .users` में मौजूद उपयोगकर्ताओं से सभी संदेशों पर)।

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

- Service account JSON: inline (`serviceAccount`) या फ़ाइल-आधारित (`serviceAccountFile`).
- Env fallback: `GOOGLE_CHAT_SERVICE_ACCOUNT` या `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- डिलीवरी लक्ष्य के लिए `spaces/<spaceId>` या `users/<userId|email>` का उपयोग करें।

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

- **Socket mode** के लिए `botToken` और `appToken` दोनों आवश्यक हैं (डिफ़ॉल्ट अकाउंट env fallback के लिए `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`).
- **HTTP mode** के लिए `botToken` के साथ `signingSecret` (root पर या प्रति-अकाउंट) आवश्यक है।
- `configWrites: false` Slack द्वारा शुरू किए गए config लिखने को अवरुद्ध करता है।
- डिलीवरी लक्ष्य के लिए `user:<id>` (DM) या `channel:<id>` का उपयोग करें।

**Reaction notification modes:** `off`, `own` (डिफ़ॉल्ट), `all`, `allowlist` ( `reactionAllowlist` से ).

**Thread session isolation:** `thread.historyScope` प्रति-थ्रेड (डिफ़ॉल्ट) या पूरे चैनल में साझा। `thread.inheritParent` नए थ्रेड्स में parent चैनल ट्रांसक्रिप्ट की कॉपी करता है।

| Action group | डिफ़ॉल्ट | नोट्स                          |
| ------------ | -------- | ------------------------------ |
| reactions    | सक्रिय   | React + प्रतिक्रियाओं की सूची  |
| messages     | सक्रिय   | पढ़ें/भेजें/संपादित करें/हटाएँ |
| pins         | सक्रिय   | पिन/अनपिन/सूची                 |
| memberInfo   | सक्रिय   | सदस्य जानकारी                  |
| emojiList    | सक्रिय   | कस्टम emoji सूची               |

### Mattermost

Mattermost एक plugin के रूप में उपलब्ध है: `openclaw plugins install @openclaw/mattermost`.

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

Chat modes: `oncall` (@-mention पर उत्तर दें, डिफ़ॉल्ट), `onmessage` (हर संदेश पर), `onchar` (ट्रिगर प्रीफ़िक्स से शुरू होने वाले संदेश).

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

**Reaction notification modes:** `off`, `own` (डिफ़ॉल्ट), `all`, `allowlist` ( `reactionAllowlist` से ).

### iMessage

OpenClaw `imsg rpc` (stdio पर JSON-RPC) को स्पॉन करता है। कोई daemon या port आवश्यक नहीं है।

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

- Messages DB के लिए Full Disk Access आवश्यक है।
- `chat_id:<id>` targets को प्राथमिकता दें। चैट्स की सूची देखने के लिए `imsg chats --limit 20` का उपयोग करें।
- `cliPath` एक SSH wrapper की ओर संकेत कर सकता है; attachment को SCP से प्राप्त करने के लिए `remoteHost` सेट करें।

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### मल्टी-अकाउंट (सभी channels)

प्रति channel कई accounts चलाएँ (प्रत्येक का अपना `accountId` हो):

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

- `default` का उपयोग तब किया जाता है जब `accountId` छोड़ा जाता है (CLI + routing)।
- Env tokens केवल **default** account पर लागू होते हैं।
- बेस channel settings सभी accounts पर लागू होती हैं, जब तक कि प्रति account उन्हें override न किया जाए।
- हर account को अलग agent से route करने के लिए `bindings[].match.accountId` का उपयोग करें।

### ग्रुप चैट mention gating

ग्रुप संदेशों में डिफ़ॉल्ट रूप से **mention आवश्यक** होता है (metadata mention या regex patterns)। यह WhatsApp, Telegram, Discord, Google Chat, और iMessage ग्रुप चैट्स पर लागू होता है।

**Mention के प्रकार:**

- **Metadata mentions**: प्लेटफ़ॉर्म के मूल @-mentions। WhatsApp self-chat mode में अनदेखा किया जाता है।
- **Text patterns**: `agents.list[].groupChat.mentionPatterns` में regex patterns। हमेशा जाँचे जाते हैं।
- Mention gating केवल तब लागू होता है जब detection संभव हो (native mentions या कम से कम एक pattern)।

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

`messages.groupChat.historyLimit` ग्लोबल डिफ़ॉल्ट सेट करता है। Channels इसे `channels.<channel>
.historyLimit` (या प्रति-account) से override कर सकते हैं। अक्षम करने के लिए `0` सेट करें।

#### DM history limits

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

Resolution: per-DM override → provider डिफ़ॉल्ट → कोई सीमा नहीं (सभी सुरक्षित रहते हैं)।

समर्थित: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`।

#### Self-chat mode

Self-chat mode सक्षम करने के लिए `allowFrom` में अपना स्वयं का नंबर शामिल करें (native @-mentions को अनदेखा करता है, केवल text patterns का उत्तर देता है):

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

### Commands (चैट command हैंडलिंग)

```json5
{
  commands: {
    native: "auto", // जब समर्थित हो तो native commands रजिस्टर करें
    text: true, // चैट संदेशों में /commands पार्स करें
    bash: false, // ! की अनुमति दें (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // /config की अनुमति दें
    debug: false, // /debug की अनुमति दें
    restart: false, // /restart + gateway restart tool की अनुमति दें
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Text commands **standalone** संदेश होने चाहिए जिनकी शुरुआत `/` से हो।
- `native: "auto"` Discord/Telegram के लिए native commands चालू करता है, Slack को बंद रखता है।
- प्रति channel override करें: `channels.discord.commands.native` (bool या `"auto"`)। `false` पहले से रजिस्टर किए गए कमांड्स को साफ़ करता है।
- `channels.telegram.customCommands` अतिरिक्त Telegram बॉट मेनू प्रविष्टियाँ जोड़ता है।
- `bash: true` `! <cmd>` को होस्ट शेल के लिए सक्षम करता है। इसके लिए `tools.elevated.enabled` सक्षम होना चाहिए और प्रेषक `tools.elevated.allowFrom.<channel>` में होना चाहिए।\`.
- `config: true` `/config` को सक्षम करता है (जो `openclaw.json` को पढ़ता/लिखता है)।
- `channels.<provider>`.configWrites\` प्रति चैनल कॉन्फ़िगरेशन बदलावों को नियंत्रित करता है (डिफ़ॉल्ट: true)।
- `allowFrom` प्रत्येक प्रदाता के लिए अलग होता है। जब यह सेट होता है, तो यह **एकमात्र** प्राधिकरण स्रोत होता है (चैनल allowlists/pairing और `useAccessGroups` को नज़रअंदाज़ किया जाता है)।
- `useAccessGroups: false` तब, जब `allowFrom` सेट नहीं है, कमांड्स को access-group नीतियों को बायपास करने की अनुमति देता है।

</Accordion>

---

## एजेंट डिफ़ॉल्ट्स

### `agents.defaults.workspace`

डिफ़ॉल्ट: `~/.openclaw/workspace`।

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

वैकल्पिक रिपॉज़िटरी रूट, जो सिस्टम प्रॉम्प्ट की Runtime पंक्ति में दिखाया जाता है। यदि सेट नहीं है, तो OpenClaw workspace से ऊपर की ओर खोजकर इसे स्वतः पहचान लेता है।

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

workspace bootstrap फ़ाइलों (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`) के स्वचालित निर्माण को अक्षम करता है।

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

ट्रंकेशन से पहले प्रत्येक workspace bootstrap फ़ाइल के लिए अधिकतम अक्षर। डिफ़ॉल्ट: `20000`।

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

सभी workspace bootstrap फ़ाइलों में सम्मिलित किए जाने वाले कुल अधिकतम अक्षर। डिफ़ॉल्ट: `24000`।

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

सिस्टम प्रॉम्प्ट संदर्भ के लिए टाइमज़ोन (संदेश टाइमस्टैम्प के लिए नहीं)। यदि उपलब्ध नहीं है, तो होस्ट टाइमज़ोन का उपयोग किया जाता है।

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

सिस्टम प्रॉम्प्ट में समय प्रारूप। डिफ़ॉल्ट: `auto` (OS वरीयता)।

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

- `model.primary`: प्रारूप `provider/model` (उदाहरण: `anthropic/claude-opus-4-6`)। यदि आप provider छोड़ देते हैं, तो OpenClaw `anthropic` मान लेता है (deprecated)।
- `models`: `/model` के लिए कॉन्फ़िगर किया गया मॉडल कैटलॉग और allowlist। हर प्रविष्टि में `alias` (शॉर्टकट) और `params` (provider-विशिष्ट: `temperature`, `maxTokens`) शामिल हो सकते हैं।
- `imageModel`: केवल तब उपयोग होता है जब प्राथमिक मॉडल में image input का समर्थन नहीं होता।
- `maxConcurrent`: सेशनों के बीच अधिकतम समानांतर agent रन (हर सेशन फिर भी क्रमबद्ध रहता है)। डिफ़ॉल्ट: 1।

**अंतर्निर्मित alias शॉर्टहैंड** (केवल तब लागू होते हैं जब मॉडल `agents.defaults.models` में हो):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

आपके कॉन्फ़िगर किए गए alias हमेशा डिफ़ॉल्ट पर प्राथमिकता लेते हैं।

Z.AI GLM-4.x मॉडल स्वतः thinking mode सक्षम करते हैं, जब तक आप `--thinking off` सेट नहीं करते या `agents.defaults.models["zai/<model>"].params.thinking` स्वयं परिभाषित नहीं करते।

### `agents.defaults.cliBackends`

केवल-पाठ fallback रन (बिना tool calls) के लिए वैकल्पिक CLI backends। जब API प्रदाता विफल हो जाएँ, तब बैकअप के रूप में उपयोगी।

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

- CLI backends text-first होते हैं; tools हमेशा निष्क्रिय रहते हैं।
- जब `sessionArg` सेट हो, तब सेशन समर्थित होते हैं।
- जब `imageArg` फ़ाइल पथ स्वीकार करता है, तब image pass-through समर्थित होता है।

### `agents.defaults.heartbeat`

आवधिक heartbeat रन।

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

- `every`: अवधि स्ट्रिंग (ms/s/m/h)। डिफ़ॉल्ट: `30m`।
- प्रति-agent: `agents.list[].heartbeat` सेट करें। जब कोई भी agent `heartbeat` परिभाषित करता है, तो **केवल वही agents** heartbeat चलाते हैं।
- Heartbeat पूर्ण agent टर्न चलाते हैं — कम अंतराल अधिक tokens खर्च करते हैं।

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
          systemPrompt: "सेशन compaction के करीब है। स्थायी यादें अभी सहेजें।",
          prompt: "किसी भी स्थायी नोट को memory/YYYY-MM-DD.md में लिखें; यदि सहेजने के लिए कुछ नहीं है तो NO_REPLY से उत्तर दें।",
        },
      },
    },
  },
}
```

- `mode`: `default` या `safeguard` (लंबे इतिहास के लिए खंडित सारांशण)। देखें [Compaction](/concepts/compaction)।
- `memoryFlush`: टिकाऊ यादों को सहेजने के लिए auto-compaction से पहले का साइलेंट agentic टर्न। जब workspace केवल-पढ़ने योग्य (read-only) हो तो इसे छोड़ दिया जाता है।

### `agents.defaults.contextPruning`

LLM को भेजने से पहले इन-मेमोरी कॉन्टेक्स्ट से **पुराने टूल परिणामों** को हटाता है। डिस्क पर मौजूद सेशन इतिहास में **कोई बदलाव नहीं** करता।

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // अवधि (ms/s/m/h), डिफ़ॉल्ट इकाई: मिनट
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[पुरानी टूल परिणाम सामग्री हटाई गई]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` pruning पास को सक्षम करता है।
- `ttl` नियंत्रित करता है कि pruning दोबारा कब चल सकती है (आखिरी cache touch के बाद)।
- Pruning पहले बड़े टूल परिणामों को soft-trim करता है, फिर आवश्यकता होने पर पुराने टूल परिणामों को hard-clear करता है।

**Soft-trim** शुरुआत और अंत को रखता है और बीच में `...` जोड़ता है।

**Hard-clear** पूरे टूल परिणाम को placeholder से बदल देता है।

नोट्स:

- इमेज ब्लॉक्स को कभी भी trim/clear नहीं किया जाता।
- अनुपात कैरेक्टर-आधारित (लगभग) होते हैं, सटीक टोकन गणना नहीं।
- यदि `keepLastAssistants` से कम assistant संदेश मौजूद हों, तो pruning छोड़ दी जाती है।

</Accordion>

व्यवहार विवरण के लिए [Session Pruning](/concepts/session-pruning) देखें।

### ब्लॉक स्ट्रीमिंग

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (minMs/maxMs का उपयोग करें)
    },
  },
}
```

- Non-Telegram चैनलों में block replies सक्षम करने के लिए स्पष्ट `*.blockStreaming: true` आवश्यक है।
- चैनल ओवरराइड्स: `channels.<channel>``.blockStreamingCoalesce` (और प्रति-अकाउंट वैरिएंट)। Signal/Slack/Discord/Google Chat में डिफ़ॉल्ट `minChars: 1500` है।
- `humanDelay`: ब्लॉक replies के बीच यादृच्छिक विराम। `natural` = 800–2500ms। प्रति-एजेंट ओवरराइड: `agents.list[].humanDelay`।

व्यवहार और chunking विवरण के लिए [Streaming](/concepts/streaming) देखें।

### टाइपिंग इंडिकेटर्स

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

- डिफ़ॉल्ट्स: direct chats/mentions के लिए `instant`, और बिना mention वाले group chats के लिए `message`।
- प्रति-सेशन ओवरराइड्स: `session.typingMode`, `session.typingIntervalSeconds`।

देखें [Typing Indicators](/concepts/typing-indicators)।

### `agents.defaults.sandbox`

एम्बेडेड एजेंट के लिए वैकल्पिक **Docker sandboxing**। पूरी गाइड के लिए [Sandboxing](/gateway/sandboxing) देखें।

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

- `none`: प्रति-स्कोप सैंडबॉक्स वर्कस्पेस `~/.openclaw/sandboxes` के अंतर्गत
- `ro`: `/workspace` पर सैंडबॉक्स वर्कस्पेस, `/agent` पर एजेंट वर्कस्पेस केवल-पढ़ने योग्य (read-only) माउंटेड
- `rw`: `/workspace` पर एजेंट वर्कस्पेस पढ़ने/लिखने योग्य (read/write) माउंटेड

**Scope:**

- `session`: प्रति-सेशन कंटेनर + वर्कस्पेस
- `agent`: प्रति एजेंट एक कंटेनर + वर्कस्पेस (डिफ़ॉल्ट)
- `shared`: साझा कंटेनर और वर्कस्पेस (सेशनों के बीच कोई आइसोलेशन नहीं)

**`setupCommand`** कंटेनर बनने के बाद एक बार चलता है ( `sh -lc` के माध्यम से ). नेटवर्क इग्रेस, लिखने योग्य रूट, और रूट उपयोगकर्ता की आवश्यकता होती है।

**कंटेनर डिफ़ॉल्ट रूप से `network: "none"` पर सेट होते हैं** — यदि एजेंट को आउटबाउंड एक्सेस चाहिए तो इसे `"bridge"` पर सेट करें।

**इनबाउंड अटैचमेंट्स** सक्रिय वर्कस्पेस में `media/inbound/*` में स्टेज किए जाते हैं।

**`docker.binds`** अतिरिक्त होस्ट डायरेक्टरीज़ माउंट करता है; ग्लोबल और प्रति-एजेंट बाइंड्स को मर्ज किया जाता है।

**Sandboxed browser** (`sandbox.browser.enabled`): कंटेनर में Chromium + CDP। noVNC URL सिस्टम प्रॉम्प्ट में इंजेक्ट किया जाता है। मुख्य कॉन्फ़िग में `browser.enabled` की आवश्यकता नहीं है।

- `allowHostControl: false` (डिफ़ॉल्ट) सैंडबॉक्स सेशनों को होस्ट ब्राउज़र को लक्ष्य बनाने से रोकता है।
- `sandbox.browser.binds` केवल सैंडबॉक्स ब्राउज़र कंटेनर में अतिरिक्त होस्ट डायरेक्टरीज़ माउंट करता है। सेट होने पर ( `[]` सहित ), यह ब्राउज़र कंटेनर के लिए `docker.binds` को प्रतिस्थापित करता है।

</Accordion>

इमेज बनाएं:

```bash
scripts/sandbox-setup.sh           # मुख्य सैंडबॉक्स इमेज
scripts/sandbox-browser-setup.sh   # वैकल्पिक ब्राउज़र इमेज
```

### `agents.list` (प्रति-एजेंट ओवरराइड्स)

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
        model: "anthropic/claude-opus-4-6", // या { primary, fallbacks }
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

- `id`: स्थिर एजेंट आईडी (आवश्यक)।
- `default`: जब कई सेट हों, तो पहला प्रभावी होगा (चेतावनी लॉग की जाती है)। यदि कोई सेट नहीं है, तो सूची की पहली प्रविष्टि डिफ़ॉल्ट होगी।
- `model`: स्ट्रिंग रूप केवल `primary` को ओवरराइड करता है; ऑब्जेक्ट रूप `{ primary, fallbacks }` दोनों को ओवरराइड करता है (`[]` ग्लोबल fallbacks को अक्षम करता है)।
- `identity.avatar`: वर्कस्पेस-सापेक्ष पथ, `http(s)` URL, या `data:` URI।
- `identity` डिफ़ॉल्ट मान प्राप्त करता है: `ackReaction` `emoji` से, `mentionPatterns` `name`/`emoji` से।
- `subagents.allowAgents`: `sessions_spawn` के लिए एजेंट आईडी की अनुमति-सूची (`["*"]` = कोई भी; डिफ़ॉल्ट: केवल वही एजेंट)।

---

## मल्टी-एजेंट रूटिंग

एक Gateway के अंदर कई आइसोलेटेड एजेंट चलाएं। देखें [Multi-Agent](/concepts/multi-agent)।

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

### बाइंडिंग मैच फ़ील्ड्स

- `match.channel` (आवश्यक)
- `match.accountId` (वैकल्पिक; `*` = कोई भी अकाउंट; छोड़ा गया = डिफ़ॉल्ट अकाउंट)
- `match.peer` (वैकल्पिक; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (वैकल्पिक; चैनल-विशिष्ट)

**निर्धारित (Deterministic) मैच क्रम:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (सटीक, कोई peer/guild/team नहीं)
5. `match.accountId: "*"` (पूरे चैनल के लिए)
6. डिफ़ॉल्ट एजेंट

प्रत्येक स्तर के भीतर, सबसे पहले मेल खाने वाली `bindings` एंट्री मान्य होगी।

### प्रति-एजेंट एक्सेस प्रोफ़ाइल

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

विस्तृत प्राथमिकता जानकारी के लिए [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) देखें।

---

## सेशन

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

- **`dmScope`**: DMs को किस प्रकार समूहित किया जाता है।
  - `main`: सभी DMs मुख्य सेशन साझा करते हैं।
  - `per-peer`: सभी चैनलों में प्रेषक id के आधार पर अलग करें।
  - `per-channel-peer`: प्रति चैनल + प्रेषक अलग करें (मल्टी-यूज़र इनबॉक्स के लिए अनुशंसित)।
  - `per-account-channel-peer`: प्रति अकाउंट + चैनल + प्रेषक अलग करें (मल्टी-अकाउंट के लिए अनुशंसित)।
- **`identityLinks`**: क्रॉस-चैनल सेशन साझा करने के लिए canonical ids को provider-prefixed peers से मैप करें।
- **`reset`**: प्राथमिक रीसेट नीति। `daily` `atHour` स्थानीय समय पर रीसेट करता है; `idle` `idleMinutes` के बाद रीसेट करता है। यदि दोनों कॉन्फ़िगर हों, तो जो पहले समाप्त होगा वही लागू होगा।
- **`resetByType`**: प्रति-प्रकार ओवरराइड (`direct`, `group`, `thread`)। पुराना `dm` को `direct` के उपनाम (alias) के रूप में स्वीकार किया जाता है।
- **`mainKey`**: पुराना फ़ील्ड। Runtime अब हमेशा मुख्य direct-chat bucket के लिए `"main"` का उपयोग करता है।
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`, पुराने `dm` alias सहित), `keyPrefix`, या `rawKeyPrefix` के आधार पर मिलान करें। पहला deny मान्य होगा।
- **`maintenance`**: `warn` हटाने (eviction) पर सक्रिय सेशन को चेतावनी देता है; `enforce` pruning और rotation लागू करता है।

</Accordion>

---

## संदेश

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

### रिस्पॉन्स प्रीफ़िक्स

प्रति-चैनल/अकाउंट ओवरराइड्स: `channels.<channel>.responsePrefix`, `channels.<channel>.accounts.<id>.responsePrefix`.

रिज़ॉल्यूशन (सबसे विशिष्ट मान्य होगा): अकाउंट → चैनल → ग्लोबल। `""` अक्षम करता है और cascade रोक देता है। `"auto"` `[{identity.name}]` से व्युत्पन्न करता है।

**टेम्पलेट वेरिएबल्स:**

| चर                | विवरण                 | उदाहरण                              |
| ----------------- | --------------------- | ----------------------------------- |
| `{model}`         | संक्षिप्त मॉडल नाम    | `claude-opus-4-6`                   |
| `{modelFull}`     | पूर्ण मॉडल पहचानकर्ता | `anthropic/claude-opus-4-6`         |
| `{provider}`      | प्रदाता का नाम        | `anthropic`                         |
| `{thinkingLevel}` | वर्तमान सोच स्तर      | `high`, `low`, `off`                |
| `{identity.name}` | एजेंट पहचान नाम       | ("auto" के समान) |

वेरिएबल्स केस-सेंसिटिव नहीं होते हैं। `{think}` , `{thinkingLevel}` का एक उपनाम है।

### Ack प्रतिक्रिया

- डिफ़ॉल्ट रूप से सक्रिय एजेंट का `identity.emoji`, अन्यथा "👀"। अक्षम करने के लिए `""` सेट करें।
- प्रति-चैनल ओवरराइड्स: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`।
- रिज़ॉल्यूशन क्रम: account → channel → `messages.ackReaction` → identity fallback।
- स्कोप: `group-mentions` (डिफ़ॉल्ट), `group-all`, `direct`, `all`।
- `removeAckAfterReply`: उत्तर के बाद ack हटाता है (केवल Slack/Discord/Telegram/Google Chat)।

### इनबाउंड डिबाउंस

एक ही प्रेषक से तेज़ी से आने वाले केवल-टेक्स्ट संदेशों को एक ही एजेंट टर्न में समेकित करता है। मीडिया/अटैचमेंट तुरंत फ्लश होते हैं। कंट्रोल कमांड डिबाउंसिंग को बायपास करते हैं।

### TTS (टेक्स्ट-टू-स्पीच)

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

- `auto` ऑटो-TTS को नियंत्रित करता है। `/tts off|always|inbound|tagged` प्रति सत्र ओवरराइड करता है।
- `summaryModel` ऑटो-समरी के लिए `agents.defaults.model.primary` को ओवरराइड करता है।
- API कुंजियाँ `ELEVENLABS_API_KEY`/`XI_API_KEY` और `OPENAI_API_KEY` पर फॉलबैक करती हैं।

---

## बात करें

Talk मोड के लिए डिफ़ॉल्ट सेटिंग्स (macOS/iOS/Android)।

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

- Voice IDs डिफ़ॉल्ट रूप से `ELEVENLABS_VOICE_ID` या `SAG_VOICE_ID` का उपयोग करते हैं।
- `apiKey` डिफ़ॉल्ट रूप से `ELEVENLABS_API_KEY` का उपयोग करता है।
- `voiceAliases` Talk निर्देशों को उपयोग में आसान नामों का उपयोग करने देता है।

---

## टूल्स

### टूल प्रोफ़ाइल्स

`tools.profile`, `tools.allow`/`tools.deny` से पहले एक बेस allowlist सेट करता है:

| प्रोफ़ाइल   | शामिल हैं                                                                                 |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | केवल `session_status`                                                                     |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | कोई प्रतिबंध नहीं (unset के समान)                                      |

### टूल समूह

| समूह               | टूल्स                                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` को `exec` के लिए एक उपनाम के रूप में स्वीकार किया जाता है) |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                                  |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`                |
| `group:memory`     | `memory_search`, `memory_get`                                                                           |
| `group:web`        | `web_search`, `web_fetch`                                                                               |
| `group:ui`         | `browser`, `canvas`                                                                                     |
| `group:automation` | `cron`, `gateway`                                                                                       |
| `group:messaging`  | `message`                                                                                               |
| `group:nodes`      | `nodes`                                                                                                 |
| `group:openclaw`   | सभी बिल्ट-इन टूल्स (provider plugins शामिल नहीं हैं)                                 |

### `tools.allow` / `tools.deny`

ग्लोबल टूल allow/deny नीति (deny को प्राथमिकता मिलती है)। Case-insensitive, `*` वाइल्डकार्ड का समर्थन करता है। Docker sandbox बंद होने पर भी लागू होता है।

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

विशिष्ट providers या models के लिए टूल्स को और सीमित करता है। क्रम: base profile → provider profile → allow/deny।

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

elevated (host) exec एक्सेस को नियंत्रित करता है:

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

- Per-agent override (`agents.list[].tools.elevated`) केवल और अधिक प्रतिबंध लगा सकता है।
- `/elevated on|off|ask|full` प्रति session स्थिति सहेजता है; inline निर्देश केवल एक संदेश पर लागू होते हैं।
- Elevated `exec` host पर चलता है और sandboxing को बायपास करता है।

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
        apiKey: "brave_api_key", // या BRAVE_API_KEY env
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

इनबाउंड मीडिया समझ (image/audio/video) को कॉन्फ़िगर करता है:

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

**Provider entry** (`type: "provider"` या छोड़ा गया):

- `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, आदि)
- `model`: model id override
- `profile` / `preferredProfile`: auth profile चयन

**CLI entry** (`type: "cli"`):

- `command`: चलाने के लिए executable
- `args`: templated args (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, आदि का समर्थन करता है)

**Common fields:**

- `capabilities`: वैकल्पिक सूची (`image`, `audio`, `video`)। डिफ़ॉल्ट: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio।
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: प्रति-entry overrides।
- विफल होने पर अगली entry पर fallback किया जाता है।

Provider auth मानक क्रम का पालन करता है: auth profiles → env vars → `models.providers.*.apiKey`।

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

- `model`: बनाए गए sub-agents के लिए डिफ़ॉल्ट मॉडल। यदि छोड़ा गया हो, तो sub-agents कॉलर का मॉडल विरासत में लेते हैं।
- प्रति-subagent टूल नीति: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## कस्टम प्रोवाइडर्स और बेस URLs

OpenClaw, pi-coding-agent मॉडल कैटलॉग का उपयोग करता है। `models.providers` को config में या `~/.openclaw/agents/<agentId>/agent/models.json` में जोड़कर कस्टम प्रोवाइडर्स जोड़ें।

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

- कस्टम ऑथ की आवश्यकताओं के लिए `authHeader: true` + `headers` का उपयोग करें।
- `OPENCLAW_AGENT_DIR` (या `PI_CODING_AGENT_DIR`) के साथ agent config root को ओवरराइड करें।

### प्रोवाइडर उदाहरण

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

Cerebras के लिए `cerebras/zai-glm-4.7` का उपयोग करें; Z.AI direct के लिए `zai/glm-4.7`।

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

`OPENCODE_API_KEY` (या `OPENCODE_ZEN_API_KEY`) सेट करें। शॉर्टकट: `openclaw onboard --auth-choice opencode-zen`.

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

`ZAI_API_KEY` सेट करें। `z.ai/*` और `z-ai/*` मान्य उपनाम हैं। शॉर्टकट: `openclaw onboard --auth-choice zai-api-key`.

- सामान्य endpoint: `https://api.z.ai/api/paas/v4`
- Coding endpoint (डिफ़ॉल्ट): `https://api.z.ai/api/coding/paas/v4`
- सामान्य endpoint के लिए, base URL ओवरराइड के साथ एक कस्टम provider परिभाषित करें।

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

China endpoint के लिए: `baseUrl: "https://api.moonshot.cn/v1"` या `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic-संगत, बिल्ट-इन provider। शॉर्टकट: `openclaw onboard --auth-choice kimi-code-api-key`.

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

Base URL में `/v1` शामिल नहीं होना चाहिए (Anthropic client इसे जोड़ देता है)। शॉर्टकट: `openclaw onboard --auth-choice synthetic-api-key`.

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

`MINIMAX_API_KEY` सेट करें। शॉर्टकट: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) देखें। संक्षेप में: गंभीर हार्डवेयर पर LM Studio Responses API के माध्यम से MiniMax M2.1 चलाएँ; fallback के लिए hosted models को merge में रखें।

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

- `allowBundled`: केवल bundled skills के लिए वैकल्पिक allowlist (managed/workspace skills प्रभावित नहीं होते)।
- `entries.<skillKey>`.enabled: false\` एक skill को अक्षम करता है, भले ही वह bundled/installed हो।
- `entries.<skillKey>.apiKey`: उन skills के लिए सुविधा, जो एक primary env var घोषित करती हैं।

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

- `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, और `plugins.load.paths` से लोड किया जाता है।
- **Config में बदलाव के लिए gateway को पुनः प्रारंभ करना आवश्यक है।**
- `allow`: वैकल्पिक allowlist (केवल सूचीबद्ध plugins लोड होंगे)। `deny` को प्राथमिकता मिलती है।

[Plugins](/tools/plugin) देखें।

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

- `evaluateEnabled: false` `act:evaluate` और `wait --fn` को अक्षम करता है।
- Remote profiles केवल attach-only होते हैं (start/stop/reset अक्षम)।
- Auto-detect क्रम: यदि डिफ़ॉल्ट ब्राउज़र Chromium-आधारित है → Chrome → Brave → Edge → Chromium → Chrome Canary।
- Control service: केवल loopback (port `gateway.port` से व्युत्पन्न, डिफ़ॉल्ट `18791`)।

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

- `seamColor`: native app UI chrome के लिए accent रंग (Talk Mode बबल टिंट, आदि)।
- `assistant`: Control UI पहचान ओवरराइड। सक्रिय agent पहचान पर fallback करता है।

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

- `mode`: `local` (gateway चलाएँ) या `remote` (remote gateway से कनेक्ट करें)। Gateway तब तक शुरू होने से इंकार करता है जब तक `local` न हो।
- `port`: WS + HTTP के लिए एकल multiplexed port। प्राथमिकता क्रम: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`।
- `bind`: `auto`, `loopback` (डिफ़ॉल्ट), `lan` (`0.0.0.0`), `tailnet` (केवल Tailscale IP), या `custom`।
- **Auth**: डिफ़ॉल्ट रूप से आवश्यक। Non-loopback bind के लिए साझा token/password आवश्यक है। Onboarding wizard डिफ़ॉल्ट रूप से एक token जनरेट करता है।
- `auth.mode: "trusted-proxy"`: auth को एक identity-aware reverse proxy को सौंपें और `gateway.trustedProxies` से identity headers पर भरोसा करें (देखें [Trusted Proxy Auth](/gateway/trusted-proxy-auth))।
- `auth.allowTailscale`: जब `true` हो, तो Tailscale Serve identity headers auth को संतुष्ट करते हैं ( `tailscale whois` के माध्यम से सत्यापित)। जब `tailscale.mode = "serve"` हो, तो डिफ़ॉल्ट `true` होता है।
- `auth.rateLimit`: वैकल्पिक failed-auth limiter। प्रति client IP और प्रति auth scope पर लागू (shared-secret और device-token को स्वतंत्र रूप से ट्रैक किया जाता है)। अवरुद्ध प्रयास `429` + `Retry-After` लौटाते हैं।
  - `auth.rateLimit.exemptLoopback` डिफ़ॉल्ट रूप से `true` है; यदि आप जानबूझकर localhost ट्रैफ़िक पर भी rate limit लागू करना चाहते हैं (test setups या सख्त proxy deployments के लिए), तो इसे `false` सेट करें।
- `tailscale.mode`: `serve` (केवल tailnet, loopback bind) या `funnel` (सार्वजनिक, auth आवश्यक)।
- `remote.transport`: `ssh` (डिफ़ॉल्ट) या `direct` (ws/wss)।  `direct` के लिए, `remote.url` `ws://` या `wss://` होना चाहिए।
- `gateway.remote.token` केवल remote CLI कॉल्स के लिए है; यह local gateway auth को सक्षम नहीं करता।
- `trustedProxies`: वे reverse proxy IPs जो TLS को terminate करते हैं। केवल उन्हीं proxies को सूचीबद्ध करें जिन्हें आप नियंत्रित करते हैं।
- `gateway.tools.deny`: HTTP `POST /tools/invoke` के लिए अवरुद्ध अतिरिक्त tool नाम (default deny list का विस्तार)।
- `gateway.tools.allow`: default HTTP deny list से tool नाम हटाएँ।

</Accordion>

### OpenAI-compatible endpoints

- Chat Completions: डिफ़ॉल्ट रूप से अक्षम। `gateway.http.endpoints.chatCompletions.enabled: true` के साथ सक्षम करें।
- Responses API: `gateway.http.endpoints.responses.enabled`।
- Responses URL-input सख्तीकरण:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Multi-instance आइसोलेशन

एक ही होस्ट पर अलग-अलग ports और state dirs के साथ कई gateways चलाएँ:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

सुविधाजनक फ़्लैग्स: `--dev` (`~/.openclaw-dev` + port `19001` का उपयोग करता है), `--profile <name>` (`~/.openclaw-<name>` का उपयोग करता है)।

देखें [Multiple Gateways](/gateway/multiple-gateways)।

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

Auth: `Authorization: Bearer <token>` या `x-openclaw-token: <token>`।

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` `}`
  - request payload से `sessionKey` केवल तभी स्वीकार किया जाता है जब `hooks.allowRequestSessionKey=true` हो (डिफ़ॉल्ट: `false`)।
- `POST /hooks/<name>` → `hooks.mappings` के माध्यम से resolve होता है

<Accordion title="Mapping details">

- `match.path` `/hooks` के बाद के sub-path से मेल खाता है (उदा. `/hooks/gmail` → `gmail`)।
- `match.source` generic paths के लिए payload फ़ील्ड से मेल खाता है।
- `{{messages[0].subject}}` जैसे templates payload से पढ़ते हैं।
- `transform` किसी JS/TS module की ओर इंगित कर सकता है जो एक hook action लौटाता है।
  - `transform.module` एक relative path होना चाहिए और `hooks.transformsDir` के भीतर ही रहना चाहिए (absolute paths और traversal अस्वीकार किए जाते हैं)।
- `agentId` किसी विशिष्ट agent को route करता है; अज्ञात IDs default पर वापस जाते हैं।
- `allowedAgentIds`: स्पष्ट routing को सीमित करता है (`*` या छोड़ा गया = सभी की अनुमति, `[]` = सभी अस्वीकृत)।
- `defaultSessionKey`: बिना स्पष्ट `sessionKey` के hook agent runs के लिए वैकल्पिक स्थिर session key।
- `allowRequestSessionKey`: `/hooks/agent` कॉलर्स को `sessionKey` सेट करने की अनुमति देता है (डिफ़ॉल्ट: `false`)।
- `allowedSessionKeyPrefixes`: स्पष्ट `sessionKey` मानों (request + mapping) के लिए वैकल्पिक prefix allowlist, उदा. `["hook:"]`।
- `deliver: true` अंतिम उत्तर को किसी channel पर भेजता है; `channel` का डिफ़ॉल्ट `last` है।
- `model` इस hook run के लिए LLM को override करता है (यदि model catalog सेट है तो यह अनुमति प्राप्त होना चाहिए)।

</Accordion>

### Gmail एकीकरण

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

- कॉन्फ़िगर होने पर Gateway बूट के समय `gog gmail watch serve` को स्वतः प्रारंभ करता है। अक्षम करने के लिए `OPENCLAW_SKIP_GMAIL_WATCHER=1` सेट करें।
- Gateway के साथ अलग से `gog gmail watch serve` न चलाएँ।

---

## Canvas होस्ट

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway पोर्ट के अंतर्गत HTTP पर एजेंट-संपादन योग्य HTML/CSS/JS और A2UI सर्व करता है:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- केवल लोकल: `gateway.bind: "loopback"` (डिफ़ॉल्ट) रखें।
- नॉन-लूपबैक बाइंड: canvas रूट्स के लिए Gateway प्रमाणीकरण (token/password/trusted-proxy) आवश्यक है, अन्य Gateway HTTP सतहों की तरह।
- Node WebViews आमतौर पर auth headers नहीं भेजते; जब कोई node पेयर होकर कनेक्ट हो जाता है, तो Gateway एक private-IP fallback की अनुमति देता है ताकि node URLs में secrets उजागर किए बिना canvas/A2UI लोड कर सके।
- सर्व किए गए HTML में live-reload क्लाइंट इंजेक्ट करता है।
- खाली होने पर प्रारंभिक `index.html` स्वतः बनाता है।
- A2UI को `/__openclaw__/a2ui/` पर भी सर्व करता है।
- परिवर्तनों के लिए gateway को पुनः प्रारंभ करना आवश्यक है।
- बड़ी डायरेक्टरी या `EMFILE` त्रुटियों के लिए live reload अक्षम करें।

---

## डिस्कवरी

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

- `minimal` (डिफ़ॉल्ट): TXT रिकॉर्ड्स से `cliPath` + `sshPort` को हटाता है।
- `full`: `cliPath` + `sshPort` शामिल करता है।
- होस्टनेम डिफ़ॉल्ट रूप से `openclaw` होता है। `OPENCLAW_MDNS_HOSTNAME` से ओवरराइड करें।

### वाइड-एरिया (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` के अंतर्गत एक unicast DNS-SD ज़ोन लिखता है। क्रॉस-नेटवर्क डिस्कवरी के लिए, DNS सर्वर (CoreDNS अनुशंसित) + Tailscale split DNS के साथ पेयर करें।

सेटअप: `openclaw dns setup --apply`।

---

## पर्यावरण

### `env` (इनलाइन env वेरिएबल्स)

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

- इनलाइन env वेरिएबल्स केवल तब लागू होते हैं जब प्रोसेस env में वह key मौजूद न हो।
- `.env` फ़ाइलें: CWD `.env` + `~/.openclaw/.env` (दोनों में से कोई भी मौजूदा वेरिएबल्स को ओवरराइड नहीं करता)।
- `shellEnv`: आपके login shell प्रोफ़ाइल से अपेक्षित लेकिन अनुपस्थित keys इम्पोर्ट करता है।
- पूर्ण प्राथमिकता क्रम के लिए [Environment](/help/environment) देखें।

### Env var प्रतिस्थापन

किसी भी कॉन्फ़िग स्ट्रिंग में `${VAR_NAME}` के साथ env वेरिएबल्स का संदर्भ दें:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- केवल अपरकेस नाम मेल किए गए: `[A-Z_][A-Z0-9_]*`।
- कॉन्फ़िग लोड होने पर गायब/खाली वेरिएबल्स त्रुटि उत्पन्न करते हैं।
- लिटरल `${VAR}` के लिए `$${VAR}` का उपयोग करें।
- `$include` के साथ काम करता है।

---

## Auth स्टोरेज

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

- प्रति-एजेंट auth प्रोफाइल `<agentDir>/auth-profiles.json` में संग्रहीत होते हैं।
- Legacy OAuth को `~/.openclaw/credentials/oauth.json` से आयात किया जाता है।
- [OAuth](/concepts/oauth) देखें।

---

## लॉगिंग

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

- डिफ़ॉल्ट लॉग फ़ाइल: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`।
- स्थिर पाथ के लिए `logging.file` सेट करें।
- `--verbose` होने पर `consoleLevel` `debug` पर चला जाता है।

---

## Wizard

CLI wizards (`onboard`, `configure`, `doctor`) द्वारा लिखा गया मेटाडेटा:

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

## पहचान

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

macOS ऑनबोर्डिंग असिस्टेंट द्वारा लिखा गया। डिफ़ॉल्ट मान निम्न से प्राप्त होते हैं:

- `messages.ackReaction` `identity.emoji` से (अन्यथा 👀 का उपयोग)
- `mentionPatterns` `identity.name`/`identity.emoji` से
- `avatar` स्वीकार करता है: workspace-relative path, `http(s)` URL, या `data:` URI

---

## Bridge (legacy, हटाया गया)

वर्तमान बिल्ड्स में अब TCP bridge शामिल नहीं है। Nodes Gateway WebSocket के माध्यम से कनेक्ट होते हैं। `bridge.*` कुंजियाँ अब कॉन्फ़िग स्कीमा का हिस्सा नहीं हैं (हटाने तक वैलिडेशन विफल होगा; `openclaw doctor --fix` अज्ञात कुंजियों को हटा सकता है)।

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

- `sessionRetention`: पूर्ण हो चुके cron सत्रों को हटाने से पहले कितनी देर तक रखा जाए। डिफ़ॉल्ट: `24h`।

[Cron Jobs](/automation/cron-jobs) देखें।

---

## मीडिया मॉडल टेम्पलेट वेरिएबल्स

`tools.media.*.models[].args` में विस्तारित किए जाने वाले टेम्पलेट प्लेसहोल्डर्स:

| वेरिएबल            | विवरण                                                                 |
| ------------------ | --------------------------------------------------------------------- |
| `{{Body}}`         | पूर्ण इनबाउंड संदेश बॉडी                                              |
| `{{RawBody}}`      | रॉ बॉडी (कोई इतिहास/प्रेषक रैपर नहीं)              |
| `{{BodyStripped}}` | समूह उल्लेख हटाकर बॉडी                                                |
| `{{From}}`         | प्रेषक पहचानकर्ता                                                     |
| `{{To}}`           | गंतव्य पहचानकर्ता                                                     |
| `{{MessageSid}}`   | चैनल संदेश आईडी                                                       |
| `{{SessionId}}`    | वर्तमान सत्र UUID                                                     |
| `{{IsNewSession}}` | नया सत्र बनने पर "true"                                               |
| `{{MediaUrl}}`     | आगमन मीडिया छद्म-URL                                                  |
| `{{MediaPath}}`    | स्थानीय मीडिया पथ                                                     |
| `{{MediaType}}`    | मीडिया प्रकार (image/audio/document/…)             |
| `{{Transcript}}`   | ऑडियो प्रतिलेख                                                        |
| `{{Prompt}}`       | CLI प्रविष्टियों के लिए निर्धारित मीडिया प्रॉम्प्ट                    |
| `{{MaxChars}}`     | CLI प्रविष्टियों के लिए निर्धारित अधिकतम आउटपुट अक्षर                 |
| `{{ChatType}}`     | "direct" या "group"                                                   |
| `{{GroupSubject}}` | समूह विषय (उपलब्ध जानकारी के अनुसार)               |
| `{{GroupMembers}}` | समूह सदस्यों की झलक (उपलब्ध जानकारी के अनुसार)     |
| `{{SenderName}}`   | प्रेषक का प्रदर्शित नाम (उपलब्ध जानकारी के अनुसार) |
| `{{SenderE164}}`   | प्रेषक का फोन नंबर (उपलब्ध जानकारी के अनुसार)      |
| `{{Provider}}`     | प्रदाता संकेत (whatsapp, telegram, discord, आदि)   |

---

## Config includes (`$include`)

कॉन्फ़िग को कई फ़ाइलों में विभाजित करें:

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

**मर्ज व्यवहार:**

- एकल फ़ाइल: जिस ऑब्जेक्ट में शामिल है उसे पूरी तरह बदल देती है।
- फ़ाइलों की ऐरे: क्रम में डीप-मर्ज होती हैं (बाद वाली पहले वाली को ओवरराइड करती है)।
- सिब्लिंग कुंजियाँ: includes के बाद मर्ज होती हैं (शामिल किए गए मानों को ओवरराइड करती हैं)।
- नेस्टेड includes: अधिकतम 10 स्तर तक।
- पथ: सापेक्ष (जिस फ़ाइल में शामिल किया गया है उसके अनुसार), पूर्ण (absolute), या `../` पैरेंट संदर्भ।
- त्रुटियाँ: गुम फ़ाइलों, पार्स त्रुटियों और सर्कुलर includes के लिए स्पष्ट संदेश।

---

_संबंधित: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
