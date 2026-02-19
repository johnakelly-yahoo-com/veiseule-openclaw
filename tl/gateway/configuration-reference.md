---
title: "Sanggunian sa Configuration"
description: "Kumpletong field-by-field na sanggunian para sa ~/.openclaw/openclaw.json"
---

# Sanggunian sa Configuration

Bawat field na available sa `~/.openclaw/openclaw.json`. Para sa task-oriented na overview, tingnan ang [Configuration](/gateway/configuration).

Ang format ng config ay **JSON5** (pinapayagan ang mga comment + trailing commas). Lahat ng field ay opsyonal — gumagamit ang OpenClaw ng ligtas na mga default kapag hindi tinukoy.

---

## Mga Channel

Awtomatikong nagsisimula ang bawat channel kapag umiiral ang config section nito (maliban kung `enabled: false`).

### Access sa DM at group

Sinusuportahan ng lahat ng channel ang mga DM policy at group policy:

| DM policy                              | Pag-uugali                                                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `pairing` (default) | Ang mga hindi kilalang sender ay makakatanggap ng one-time pairing code; dapat aprubahan ng owner |
| `allowlist`                            | Tanging mga sender sa `allowFrom` (o paired allow store)                       |
| `open`                                 | Payagan ang lahat ng papasok na DM (nangangailangan ng `allowFrom: ["*"]`)     |
| `disabled`                             | Huwag pansinin ang lahat ng papasok na DM                                                         |

| Patakaran ng grupo                       | Pag-uugali                                                                                 |
| ---------------------------------------- | ------------------------------------------------------------------------------------------ |
| `allowlist` (default) | Mga grupo lamang na tumutugma sa naka-configure na allowlist                               |
| `open`                                   | Laktawan ang mga allowlist ng grupo (umiiral pa rin ang mention-gating) |
| `disabled`                               | Harangin ang lahat ng mensahe sa grupo/room                                                |

<Note>
Itinatakda ng `channels.defaults.groupPolicy` ang default kapag hindi nakatakda ang `groupPolicy` ng isang provider.
Ang mga pairing code ay nag-e-expire pagkalipas ng 1 oras. Ang mga nakabinbing DM pairing request ay limitado sa **3 bawat channel**.
May espesyal na fallback ang Slack/Discord: kung ganap na nawawala ang kanilang provider section, maaaring maging `open` ang runtime group policy (na may babala sa pagsisimula).
</Note>

### WhatsApp

Ang WhatsApp ay tumatakbo sa pamamagitan ng web channel ng gateway (Baileys Web). Awtomatiko itong nagsisimula kapag may naka-link na session.

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

- Ang mga outbound command ay gumagamit ng account na `default` bilang default kung mayroon; kung wala, ang unang naka-configure na account id (ayon sa pagkakasunod-sunod).
- Ang legacy na single-account Baileys auth dir ay inililipat ng `openclaw doctor` papunta sa `whatsapp/default`.
- Mga override kada account: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

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

- Bot token: `channels.telegram.botToken` o `channels.telegram.tokenFile`, na may `TELEGRAM_BOT_TOKEN` bilang fallback para sa default na account.
- Pinipigilan ng `configWrites: false` ang mga config write na pinasimulan ng Telegram (mga migration ng supergroup ID, `/config set|unset`).
- Gumagamit ang Telegram stream previews ng `sendMessage` + `editMessageText` (gumagana sa direct at group chats).
- Patakaran sa retry: tingnan ang [Retry policy](/concepts/retry).

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

- Token: `channels.discord.token`, na may `DISCORD_BOT_TOKEN` bilang fallback para sa default na account.
- Gamitin ang `user:<id>` (DM) o `channel:<id>` (guild channel) para sa mga delivery target; hindi tinatanggap ang mga simpleng numeric ID.
- Ang mga guild slug ay lowercase at ang mga espasyo ay pinapalitan ng `-`; ginagamit ng mga channel key ang slugged na pangalan (walang `#`). Mas mainam na gamitin ang mga guild ID.
- Ang mga mensaheng ginawa ng bot ay hindi pinapansin bilang default. Pinapagana ng `allowBots: true` ang mga ito (sinasala pa rin ang sariling mga mensahe).
- Hinahati ng `maxLinesPerMessage` (default 17) ang mahahabang mensahe kahit na mas mababa sa 2000 na karakter.
- Itinatakda ng `channels.discord.ui.components.accentColor` ang accent color para sa mga container ng Discord components v2.

**Mga mode ng notification ng reaksyon:** `off` (wala), `own` (mga mensahe ng bot, default), `all` (lahat ng mensahe), `allowlist` (mula sa `guilds.<id>``.users` sa lahat ng mensahe).

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

- Service account JSON: inline (`serviceAccount`) o file-based (`serviceAccountFile`).
- Mga fallback sa env: `GOOGLE_CHAT_SERVICE_ACCOUNT` o `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Gamitin ang `spaces/<spaceId>` o `users/<userId|email>` para sa mga target ng pagpapadala.

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

- **Socket mode** ay nangangailangan ng parehong `botToken` at `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` para sa default na account env fallback).
- **HTTP mode** ay nangangailangan ng `botToken` kasama ang `signingSecret` (sa root o per-account).
- Pinipigilan ng `configWrites: false` ang mga config write na pinasimulan ng Slack.
- Gamitin ang `user:<id>` (DM) o `channel:<id>` para sa mga target ng pagpapadala.

**Mga mode ng notification ng reaksyon:** `off`, `own` (default), `all`, `allowlist` (mula sa `reactionAllowlist`).

**Isolation ng session sa thread:** Ang `thread.historyScope` ay per-thread (default) o naka-share sa buong channel. Kinokopya ng `thread.inheritParent` ang transcript ng parent channel papunta sa mga bagong thread.

| Action group | Default | Mga Tala                              |
| ------------ | ------- | ------------------------------------- |
| reactions    | enabled | Mag-react + ilista ang mga reaksyon   |
| messages     | enabled | Magbasa/magpadala/mag-edit/mag-delete |
| pins         | enabled | Mag-pin/mag-unpin/maglista            |
| memberInfo   | enabled | Impormasyon ng miyembro               |
| emojiList    | enabled | Listahan ng custom emoji              |

### Mattermost

Ang Mattermost ay ipinapadala bilang isang plugin: `openclaw plugins install @openclaw/mattermost`.

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

Mga chat mode: `oncall` (tumugon sa @-mention, default), `onmessage` (bawat mensahe), `onchar` (mga mensaheng nagsisimula sa trigger prefix).

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

**Mga mode ng notipikasyon ng reaksyon:** `off`, `own` (default), `all`, `allowlist` (mula sa `reactionAllowlist`).

### iMessage

Ang OpenClaw ay naglulunsad ng `imsg rpc` (JSON-RPC sa pamamagitan ng stdio). Hindi kailangan ng daemon o port.

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

- Nangangailangan ng Full Disk Access sa Messages DB.
- Mas mainam na gamitin ang mga target na `chat_id:<id>`. Gamitin ang `imsg chats --limit 20` upang ilista ang mga chat.
- Maaaring ituro ng `cliPath` sa isang SSH wrapper; itakda ang `remoteHost` para sa pagkuha ng attachment sa pamamagitan ng SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Multi-account (lahat ng channel)

Magpatakbo ng maraming account bawat channel (bawat isa ay may sariling `accountId`):

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

- Ginagamit ang `default` kapag hindi tinukoy ang `accountId` (CLI + routing).
- Nalalapat lamang ang mga Env token sa **default** na account.
- Ang mga base setting ng channel ay nalalapat sa lahat ng account maliban kung may override kada account.
- Gamitin ang `bindings[].match.accountId` upang i-route ang bawat account sa ibang agent.

### Pag-gate ng pagbanggit sa group chat

Bilang default, ang mga mensahe sa group ay **nangangailangan ng pagbanggit** (metadata mention o mga regex pattern). Nalalapat sa mga group chat ng WhatsApp, Telegram, Discord, Google Chat, at iMessage.

**Mga uri ng pagbanggit:**

- **Metadata mentions**: Native na @-mentions ng platform. Hindi ito isinasaalang-alang sa WhatsApp self-chat mode.
- **Mga text pattern**: Mga regex pattern sa `agents.list[].groupChat.mentionPatterns`. Laging sinusuri.
- Ipinapatupad lamang ang mention gating kapag posible ang detection (native mentions o may hindi bababa sa isang pattern).

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

Itinatakda ng `messages.groupChat.historyLimit` ang global default. Maaaring mag-override ang mga channel gamit ang `channels.<channel>`.historyLimit`(o kada account). Itakda sa`0\` upang i-disable.

#### Mga limitasyon ng DM history

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

Resolusyon: override kada DM → default ng provider → walang limitasyon (lahat ay naka-retain).

Sinusuportahan: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat mode

Isama ang sarili mong numero sa `allowFrom` upang paganahin ang self-chat mode (hindi papansinin ang native @-mentions, tutugon lamang sa mga text pattern):

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

### Mga Command (paghawak ng chat command)

```json5
{
  commands: {
    native: "auto", // magrehistro ng native commands kapag suportado
    text: true, // i-parse ang /commands sa mga mensahe sa chat
    bash: false, // payagan ang ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // payagan ang /config
    debug: false, // payagan ang /debug
    restart: false, // payagan ang /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Ang mga text command ay dapat na **standalone** na mga mensahe na may nauunang `/`.
- Ang `native: "auto"` ay nag-o-on ng native commands para sa Discord/Telegram, at iniiwang naka-off ang Slack.
- Mag-override bawat channel: `channels.discord.commands.native` (bool o `"auto"`). Ang `false` ay nag-aalis ng mga dating nairehistrong command.
- Ang `channels.telegram.customCommands` ay nagdadagdag ng karagdagang Telegram bot menu entries.
- Pinapagana ng `bash: true` ang `! <cmd>` para sa host shell. Kinakailangan ang `tools.elevated.enabled` at ang sender ay nasa `tools.elevated.allowFrom.<channel>`.
- Pinapagana ng `config: true` ang `/config` (nagbabasa/nagsusulat sa `openclaw.json`).
- `channels.<provider>Ang `.configWrites\` ay kumokontrol sa config mutations bawat channel (default: true).
- Ang `allowFrom` ay nakatakda bawat provider. Kapag naka-set, ito ang **tanging** pinagmumulan ng authorization (ang channel allowlists/pairing at `useAccessGroups` ay hindi isasaalang-alang).
- Ang `useAccessGroups: false` ay nagpapahintulot sa mga command na lampasan ang access-group policies kapag ang `allowFrom` ay hindi naka-set.

</Accordion>

---

## Mga default ng Agent

### `agents.defaults.workspace`

Default: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Opsyonal na repository root na ipinapakita sa Runtime line ng system prompt. Kung hindi naka-set, awtomatikong dini-detect ng OpenClaw sa pamamagitan ng paglalakad pataas mula sa workspace.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Hindi pinapagana ang awtomatikong paglikha ng mga workspace bootstrap file (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Pinakamataas na bilang ng mga karakter bawat workspace bootstrap file bago ito putulin. Default: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Pinakamataas na kabuuang bilang ng mga karakter na ini-inject sa lahat ng workspace bootstrap file. Default: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Timezone para sa konteksto ng system prompt (hindi kasama ang message timestamps). Gumagamit ng host timezone kung walang nakatakda.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Format ng oras sa system prompt. Default: `auto` (kagustuhan ng OS).

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

- `model.primary`: format na `provider/model` (hal. `anthropic/claude-opus-4-6`). Kung hindi mo ilalagay ang provider, ipagpapalagay ng OpenClaw na `anthropic` (deprecated).
- `models`: ang naka-configure na model catalog at allowlist para sa `/model`. Ang bawat entry ay maaaring maglaman ng `alias` (shortcut) at `params` (provider-specific: `temperature`, `maxTokens`).
- `imageModel`: gagamitin lamang kung walang image input ang primary model.
- `maxConcurrent`: pinakamataas na sabayang agent runs sa lahat ng session (ang bawat session ay serialized pa rin). Default: 1.

**Built-in alias shorthands** (naaangkop lamang kapag ang model ay nasa `agents.defaults.models`):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Ang iyong mga naka-configure na alias ay laging mananaig kaysa sa defaults.

Ang mga modelong Z.AI GLM-4.x ay awtomatikong nag-e-enable ng thinking mode maliban kung itatakda mo ang `--thinking off` o ikaw mismo ang magde-define ng `agents.defaults.models["zai/<model>"].params.thinking`.

### `agents.defaults.cliBackends`

Opsyonal na CLI backends para sa text-only fallback runs (walang tool calls). Kapaki-pakinabang bilang backup kapag pumalya ang mga API provider.

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

- Ang mga CLI backend ay text-first; palaging naka-disable ang tools.
- Sinusuportahan ang sessions kapag nakatakda ang `sessionArg`.
- Sinusuportahan ang image pass-through kapag tumatanggap ng file paths ang `imageArg`.

### `agents.defaults.heartbeat`

Mga pana-panahong heartbeat run.

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

- `every`: string ng tagal (ms/s/m/h). Default: `30m`.
- Bawat agent: itakda ang `agents.list[].heartbeat`. Kapag may kahit isang agent na nagtakda ng `heartbeat`, **ang mga agent na iyon lamang** ang magpapatakbo ng heartbeats.
- Ang heartbeats ay nagpapatakbo ng buong agent turns — mas maiikling pagitan ay mas maraming tokens ang nagagamit.

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

- `mode`: `default` o `safeguard` (chunked summarization para sa mahahabang history). Tingnan ang [Compaction](/concepts/compaction).
- `memoryFlush`: tahimik na agentic turn bago ang auto-compaction upang mag-imbak ng pangmatagalang memories. Nilalaktawan kapag read-only ang workspace.

### `agents.defaults.contextPruning`

Tinatanggal ang **lumang tool results** mula sa in-memory context bago ipadala sa LLM. **Hindi** nito binabago ang session history sa disk.

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

- `mode: "cache-ttl"` ay nagpapagana ng pruning passes.
- Kinokontrol ng `ttl` kung gaano kadalas maaaring tumakbo muli ang pruning (pagkatapos ng huling cache touch).
- Ang pruning ay unang nagsasagawa ng soft-trim sa malalaking tool results, pagkatapos ay hard-clear sa mas lumang tool results kung kinakailangan.

**Soft-trim** ay pinapanatili ang simula + dulo at naglalagay ng `...` sa gitna.

**Hard-clear** ay pinapalitan ang buong tool result ng placeholder.

Mga Tala:

- Ang mga image block ay hindi kailanman tina-trim o kina-clear.
- Ang mga ratio ay batay sa bilang ng character (tinatayang), hindi eksaktong bilang ng token.
- Kung mas kaunti sa `keepLastAssistants` na assistant messages ang umiiral, nilalaktawan ang pruning.

</Accordion>

Tingnan ang [Session Pruning](/concepts/session-pruning) para sa detalye ng behavior.

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

- Ang mga non-Telegram channel ay nangangailangan ng tahasang `*.blockStreaming: true` upang paganahin ang block replies.
- Channel overrides: `channels.<channel> .blockStreamingCoalesce` (at mga per-account na variant). Signal/Slack/Discord/Google Chat default `minChars: 1500`.
- `humanDelay`: random na paghinto sa pagitan ng mga block reply. `natural` = 800–2500ms. Per-agent override: `agents.list[].humanDelay`.

Tingnan ang [Streaming](/concepts/streaming) para sa detalye ng behavior + chunking.

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

- Mga default: `instant` para sa direct chats/mentions, `message` para sa group chats na walang mention.
- Per-session overrides: `session.typingMode`, `session.typingIntervalSeconds`.

Tingnan ang [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Opsyonal na **Docker sandboxing** para sa embedded agent. Tingnan ang [Sandboxing](/gateway/sandboxing) para sa kumpletong gabay.

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

**Access sa workspace:**

- `none`: per-scope na sandbox workspace sa ilalim ng `~/.openclaw/sandboxes`
- `ro`: sandbox workspace sa `/workspace`, agent workspace na naka-mount bilang read-only sa `/agent`
- `rw`: agent workspace na naka-mount bilang read/write sa `/workspace`

**Scope:**

- `session`: per-session na container + workspace
- `agent`: isang container + workspace bawat agent (default)
- `shared`: shared na container at workspace (walang cross-session isolation)

**`setupCommand`** tumatakbo nang isang beses pagkatapos malikha ang container (sa pamamagitan ng `sh -lc`). Nangangailangan ng network egress, writable root, at root user.

**Ang mga container ay naka-default sa `network: "none"`** — itakda sa `"bridge"` kung kailangan ng agent ng outbound access.

**Ang mga inbound attachment** ay inilalagay sa `media/inbound/*` sa aktibong workspace.

**Ang `docker.binds`** ay nagmo-mount ng karagdagang host directories; pinagsasama ang global at per-agent binds.

**Sandboxed browser** (`sandbox.browser.enabled`): Chromium + CDP sa isang container. Ang noVNC URL ay ini-inject sa system prompt. Hindi nito kailangan ang `browser.enabled` sa pangunahing config.

- `allowHostControl: false` (default) hinaharangan ang mga sandboxed session na i-target ang host browser.
- `sandbox.browser.binds` nagmo-mount ng karagdagang host directories sa sandbox browser container lamang. Kapag naka-set (kasama ang `[]`), pinapalitan nito ang `docker.binds` para sa browser container.

</Accordion>

I-build ang mga image:

```bash
scripts/sandbox-setup.sh           # main sandbox image
scripts/sandbox-browser-setup.sh   # optional browser image
```

### `agents.list` (per-agent na overrides)

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

- `id`: matatag na agent id (kinakailangan).
- `default`: kapag marami ang naka-set, ang una ang mananaig (may warning na ila-log). Kung walang naka-set, ang unang entry sa listahan ang magiging default.
- `model`: ang string form ay nag-o-override lamang sa `primary`; ang object form na `{ primary, fallbacks }` ay nag-o-override sa pareho (`[]` ay hindi pinapagana ang global fallbacks).
- `identity.avatar`: path na naka-relative sa workspace, `http(s)` URL, o `data:` URI.
- Ang `identity` ay kumukuha ng mga default: `ackReaction` mula sa `emoji`, `mentionPatterns` mula sa `name`/`emoji`.
- `subagents.allowAgents`: allowlist ng mga agent id para sa `sessions_spawn` (`["*"]` = kahit alin; default: kaparehong agent lamang).

---

## Multi-agent routing

Magpatakbo ng maraming hiwa-hiwalay na agent sa loob ng isang Gateway. Tingnan ang [Multi-Agent](/concepts/multi-agent).

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

### Mga binding match field

- `match.channel` (kinakailangan)
- `match.accountId` (opsyonal; `*` = anumang account; kapag hindi isinama = default na account)
- `match.peer` (opsyonal; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (opsyonal; partikular sa channel)

**Deterministikong pagkakasunod-sunod ng match:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (eksakto, walang peer/guild/team)
5. `match.accountId: "*"` (buong channel)
6. Default na agent

Sa loob ng bawat tier, ang unang tumugmang entry sa `bindings` ang mananaig.

### Mga profile ng access bawat agent

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

Tingnan ang [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) para sa mga detalye ng precedence.

---

## Session

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

- **`dmScope`**: kung paano pinagsasama-sama ang mga DM.
  - `main`: lahat ng DM ay nagbabahagi ng main session.
  - `per-peer`: ihiwalay batay sa sender id sa iba’t ibang channel.
  - `per-channel-peer`: ihiwalay bawat channel + sender (inirerekomenda para sa mga multi-user inbox).
  - `per-account-channel-peer`: ihiwalay bawat account + channel + sender (inirerekomenda para sa multi-account).
- **`identityLinks`**: i-map ang canonical ids sa mga peer na may provider prefix para sa pagbabahagi ng session sa iba’t ibang channel.
- **`reset`**: pangunahing patakaran ng pag-reset. `daily` ay nagre-reset sa `atHour` lokal na oras; ang `idle` ay nagre-reset pagkatapos ng `idleMinutes`. Kapag parehong naka-configure, kung alin ang unang mag-expire ang masusunod.
- **`resetByType`**: mga override bawat uri (`direct`, `group`, `thread`). Ang legacy na `dm` ay tinatanggap bilang alias para sa `direct`.
- **`mainKey`**: legacy na field. Ang runtime ay palaging gumagamit na ngayon ng `"main"` para sa pangunahing direct-chat bucket.
- **`sendPolicy`**: mag-match ayon sa `channel`, `chatType` (`direct|group|channel`, na may legacy na `dm` alias), `keyPrefix`, o `rawKeyPrefix`. Unang deny ang mananaig.
- **`maintenance`**: ang `warn` ay nagbibigay-babala sa aktibong session kapag may eviction; ang `enforce` ay nagpapatupad ng pruning at rotation.

</Accordion>

---

## Mga Mensahe

```json5
{
  messages: {
    responsePrefix: "🦞", // o "auto"
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

### Prefix ng tugon

Mga override bawat channel/account: `channels.<channel>``.responsePrefix`, `channels.<channel>``.accounts.<id>``.responsePrefix`.

Resolusyon (pinaka-tiyak ang mananaig): account → channel → global. `""` hindi pinapagana at itinitigil ang cascade. `"auto"` kinukuha mula sa `[{identity.name}]`.

**Mga template variable:**

| Variable          | Paglalarawan                    | Halimbawa                               |
| ----------------- | ------------------------------- | --------------------------------------- |
| `{model}`         | Maikling pangalan ng model      | `claude-opus-4-6`                       |
| `{modelFull}`     | Buong identifier ng model       | `anthropic/claude-opus-4-6`             |
| `{provider}`      | Pangalan ng provider            | `anthropic`                             |
| `{thinkingLevel}` | Kasalukuyang antas ng pag-iisip | `high`, `low`, `off`                    |
| `{identity.name}` | Pangalan ng identity ng agent   | (pareho ng `"auto"`) |

Hindi case-sensitive ang mga variable. `{think}` ay alias ng `{thinkingLevel}`.

### Ack reaction

- Default ay `identity.emoji` ng aktibong agent, kung wala ay `"👀"`. Itakda sa `""` para huwag paganahin.
- Mga override kada channel: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`.
- Pagkakasunod ng resolution: account → channel → `messages.ackReaction` → fallback sa identity.
- Saklaw: `group-mentions` (default), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: inaalis ang ack pagkatapos ng reply (Slack/Discord/Telegram/Google Chat lamang).

### Inbound debounce

Pinagsasama ang sunod-sunod at mabilis na text-only na mensahe mula sa parehong sender sa iisang turn ng agent. Ang media/attachments ay agad na pinoproseso. Ang mga control command ay hindi saklaw ng debouncing.

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

- `auto` ang kumokontrol sa auto-TTS. `/tts off|always|inbound|tagged` override sa bawat session.
- Ino-override ng `summaryModel` ang `agents.defaults.model.primary` para sa auto-summary.
- Ang mga API key ay babalik sa `ELEVENLABS_API_KEY`/`XI_API_KEY` at `OPENAI_API_KEY`.

---

## Usapan

Mga default para sa Talk mode (macOS/iOS/Android).

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

- Ang mga Voice ID ay babalik sa `ELEVENLABS_VOICE_ID` o `SAG_VOICE_ID`.
- Ang `apiKey` ay babalik sa `ELEVENLABS_API_KEY`.
- Pinapayagan ng `voiceAliases` na gumamit ang Talk directives ng mga madaling tandaan na pangalan.

---

## Mga Tool

### Mga profile ng tool

Itinatakda ng `tools.profile` ang batayang allowlist bago ang `tools.allow`/`tools.deny`:

| Profile     | Kasama ang                                                                                |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | `session_status` lamang                                                                   |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Walang limitasyon (kapareho kapag hindi nakatakda)                     |

### Mga grupo ng tool

| Grupo              | Mga Tool                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` ay tinatanggap bilang alias para sa `exec`) |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Lahat ng built-in tools (hindi kasama ang provider plugins)           |

### `tools.allow` / `tools.deny`

Pangkalahatang patakaran ng pagpayag/pagtanggi ng tool (mas nangingibabaw ang deny). Hindi case-sensitive, sumusuporta sa `*` wildcards. Ina-apply kahit naka-off ang Docker sandbox.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Mas lalo pang paghihigpit ng mga tool para sa partikular na providers o models. Ayos ng pagkakasunod: base profile → provider profile → allow/deny.

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

Kinokontrol ang elevated (host) exec access:

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

- Ang per-agent override (`agents.list[].tools.elevated`) ay maaari lamang magpatupad ng mas mahigpit na paghihigpit.
- Ang `/elevated on|off|ask|full` ay nagtatala ng estado kada session; ang inline directives ay nalalapat lamang sa iisang mensahe.
- Ang elevated `exec` ay tumatakbo sa host at nilalampasan ang sandboxing.

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
        apiKey: "brave_api_key", // o BRAVE_API_KEY env
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

Nagko-configure ng pag-unawa sa papasok na media (image/audio/video):

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

**Provider entry** (`type: "provider"` o hindi tinukoy):

- `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, atbp.)
- `model`: override ng model id
- `profile` / `preferredProfile`: pagpili ng auth profile

**CLI entry** (`type: "cli"`):

- `command`: executable na tatakbuhin
- `args`: templated na args (sumusuporta sa `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, atbp.)

**Common fields:**

- `capabilities`: opsyonal na listahan (`image`, `audio`, `video`). Mga default: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: mga override kada entry.
- Ang mga pagkabigo ay lilipat sa susunod na entry.

Ang provider auth ay sumusunod sa karaniwang pagkakasunod: auth profiles → env vars → `models.providers.*.apiKey`.

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

- `model`: default na model para sa mga spawn na sub-agent. Kung hindi ito isasama, mamanahin ng mga sub-agent ang model ng tumawag.
- Patakaran sa tool bawat sub-agent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Mga custom provider at base URL

Ginagamit ng OpenClaw ang pi-coding-agent model catalog. Magdagdag ng mga custom provider sa pamamagitan ng `models.providers` sa config o sa `~/.openclaw/agents/<agentId>/agent/models.json`.

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

- Gamitin ang `authHeader: true` + `headers` para sa mga custom na pangangailangan sa auth.
- I-override ang root ng agent config gamit ang `OPENCLAW_AGENT_DIR` (o `PI_CODING_AGENT_DIR`).

### Mga halimbawa ng provider

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

Gamitin ang `cerebras/zai-glm-4.7` para sa Cerebras; `zai/glm-4.7` para sa direktang Z.AI.

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

Itakda ang `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`). Shortcut: `openclaw onboard --auth-choice opencode-zen`.

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

Itakda ang `ZAI_API_KEY`. `z.ai/*` at `z-ai/*` ay mga tinatanggap na alias. Shortcut: `openclaw onboard --auth-choice zai-api-key`.

- Pangkalahatang endpoint: `https://api.z.ai/api/paas/v4`
- Endpoint para sa coding (default): `https://api.z.ai/api/coding/paas/v4`
- Para sa pangkalahatang endpoint, magtakda ng custom provider na may override sa base URL.

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

Para sa China endpoint: `baseUrl: "https://api.moonshot.cn/v1"` o `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic-compatible, built-in na provider. Shortcut: `openclaw onboard --auth-choice kimi-code-api-key`.

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

Ang base URL ay hindi dapat isama ang `/v1` (idinadagdag ito ng Anthropic client). Shortcut: `openclaw onboard --auth-choice synthetic-api-key`.

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

Itakda ang `MINIMAX_API_KEY`. Shortcut: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Tingnan ang [Mga Local Model](/gateway/local-models). Sa madaling sabi: patakbuhin ang MiniMax M2.1 sa pamamagitan ng LM Studio Responses API sa malakas na hardware; panatilihing naka-merge ang mga hosted model bilang fallback.

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

- `allowBundled`: opsyonal na allowlist para lamang sa mga bundled skills (hindi apektado ang managed/workspace skills).
- `entries.<skillKey>``.enabled: false` hindi pinapagana ang isang skill kahit ito ay bundled/installed.
- `entries.<skillKey>``.apiKey`: kaginhawaan para sa mga skill na nagdedeklara ng pangunahing env var.

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

- Nilo-load mula sa `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, at `plugins.load.paths`.
- **Ang mga pagbabago sa config ay nangangailangan ng pag-restart ng gateway.**
- `allow`: opsyonal na allowlist (ang mga nakalista lamang na plugins ang ilo-load). `deny` ang masusunod.

Tingnan ang [Plugins](/tools/plugin).

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

- `evaluateEnabled: false` hindi pinapagana ang `act:evaluate` at `wait --fn`.
- Ang mga remote profile ay attach-only (hindi pinapagana ang start/stop/reset).
- Auto-detect na pagkakasunod-sunod: default na browser kung Chromium-based → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Control service: loopback lamang (ang port ay nagmumula sa `gateway.port`, default `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, maikling teksto, image URL, o data URI
    },
  },
}
```

- `seamColor`: accent color para sa native app UI chrome (kulay ng Talk Mode bubble, atbp.).
- `assistant`: override ng pagkakakilanlan ng Control UI. Bumabalik sa aktibong agent identity.

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
      // password: "your-password", // o OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // para sa mode=trusted-proxy; tingnan ang /gateway/trusted-proxy-auth
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
      // Karagdagang /tools/invoke HTTP denies
      deny: ["browser"],
      // Alisin ang mga tool mula sa default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (patakbuhin ang gateway) o `remote` (kumonekta sa remote gateway). Hindi magsisimula ang Gateway maliban kung `local`.
- `port`: iisang multiplexed port para sa WS + HTTP. Precedence: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (default), `lan` (`0.0.0.0`), `tailnet` (Tailscale IP lamang), o `custom`.
- **Auth**: kinakailangan bilang default. Ang mga non-loopback bind ay nangangailangan ng shared token/password. Ang onboarding wizard ay gumagawa ng token bilang default.
- `auth.mode: "trusted-proxy"`: i-delegate ang auth sa isang identity-aware reverse proxy at pagkatiwalaan ang identity headers mula sa `gateway.trustedProxies` (tingnan ang [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: kapag `true`, ang Tailscale Serve identity headers ay sapat para sa auth (nave-verify sa pamamagitan ng `tailscale whois`). Default ay `true` kapag `tailscale.mode = "serve"`.
- `auth.rateLimit`: opsyonal na limiter para sa mga nabigong auth. Nalalapat kada client IP at kada auth scope (ang shared-secret at device-token ay sinusubaybayan nang hiwalay). Ang mga na-block na pagtatangka ay nagbabalik ng `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` ay naka-default sa `true`; itakda sa `false` kung nais mong isailalim din sa rate limit ang localhost traffic (para sa mga test setup o mahigpit na proxy deployment).
- `tailscale.mode`: `serve` (tailnet lamang, loopback bind) o `funnel` (pampubliko, nangangailangan ng auth).
- `remote.transport`: `ssh` (default) o `direct` (ws/wss). Para sa `direct`, ang `remote.url` ay dapat `ws://` o `wss://`.
- Ang `gateway.remote.token` ay para lamang sa mga remote CLI call; hindi nito pinapagana ang lokal na gateway auth.
- `trustedProxies`: mga IP ng reverse proxy na nagtatapos ng TLS. Ilista lamang ang mga proxy na ikaw ang may kontrol.
- `gateway.tools.deny`: mga karagdagang pangalan ng tool na hinaharang para sa HTTP `POST /tools/invoke` (pinapalawak ang default deny list).
- `gateway.tools.allow`: alisin ang mga pangalan ng tool mula sa default HTTP deny list.

</Accordion>

### Mga endpoint na compatible sa OpenAI

- Chat Completions: hindi pinagana bilang default. Paganahin gamit ang `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Pagpapatibay ng URL-input ng Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Multi-instance isolation

Magpatakbo ng maraming gateway sa iisang host na may natatanging ports at state dirs:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Mga convenience flag: `--dev` (gumagamit ng `~/.openclaw-dev` + port na `19001`), `--profile <name>` (gumagamit ng `~/.openclaw-<name>`).

Tingnan ang [Multiple Gateways](/gateway/multiple-gateways).

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

Auth: `Authorization: Bearer <token>` o `x-openclaw-token: <token>`.

**Mga Endpoint:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - Ang `sessionKey` mula sa request payload ay tinatanggap lamang kapag `hooks.allowRequestSessionKey=true` (default: `false`).
- `POST /hooks/<name>` → nireresolba sa pamamagitan ng `hooks.mappings`

<Accordion title="Mapping details">

- Ang `match.path` ay tumutugma sa sub-path pagkatapos ng `/hooks` (hal. `/hooks/gmail` → `gmail`).
- Ang `match.source` ay tumutugma sa isang field ng payload para sa mga generic na path.
- Ang mga template tulad ng `{{messages[0].subject}}` ay nagbabasa mula sa payload.
- Ang `transform` ay maaaring tumukoy sa isang JS/TS module na nagbabalik ng hook action.
  - Ang `transform.module` ay dapat relative path at mananatili sa loob ng `hooks.transformsDir` (ang absolute path at traversal ay tatanggihan).
- Ang `agentId` ay nagra-route sa isang partikular na agent; ang hindi kilalang ID ay babalik sa default.
- `allowedAgentIds`: nililimitahan ang tahasang pagra-route (`*` o hindi tinukoy = payagan lahat, `[]` = tanggihan lahat).
- `defaultSessionKey`: opsyonal na nakapirming session key para sa mga hook agent run na walang tahasang `sessionKey`.
- `allowRequestSessionKey`: pahintulutan ang mga tumatawag sa `/hooks/agent` na magtakda ng `sessionKey` (default: `false`).
- `allowedSessionKeyPrefixes`: opsyonal na allowlist ng prefix para sa tahasang `sessionKey` na mga value (request + mapping), hal. `["hook:"]`.
- `deliver: true` nagpapadala ng huling tugon sa isang channel; ang `channel` ay default na `last`.
- Ang `model` ay nag-o-override ng LLM para sa hook run na ito (dapat pinapayagan kung may naka-set na model catalog).

</Accordion>

### Integrasyon ng Gmail

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

- Awtomatikong sini-start ng Gateway ang `gog gmail watch serve` sa pag-boot kapag naka-configure. Itakda ang `OPENCLAW_SKIP_GMAIL_WATCHER=1` para i-disable.
- Huwag magpatakbo ng hiwalay na `gog gmail watch serve` kasabay ng Gateway.

---

## Canvas host

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Nagse-serve ng agent-editable na HTML/CSS/JS at A2UI sa pamamagitan ng HTTP sa ilalim ng Gateway port:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Lokal lamang: panatilihin ang `gateway.bind: "loopback"` (default).
- Mga non-loopback bind: ang mga canvas route ay nangangailangan ng Gateway auth (token/password/trusted-proxy), katulad ng iba pang Gateway HTTP surfaces.
- Karaniwang hindi nagpapadala ng auth headers ang mga Node WebView; kapag na-pair at nakakonekta na ang isang node, pinapayagan ng Gateway ang private-IP fallback upang ma-load ng node ang canvas/A2UI nang hindi naglalantad ng mga lihim sa mga URL.
- Nag-iinject ng live-reload client sa sine-serve na HTML.
- Awtomatikong gumagawa ng panimulang `index.html` kapag walang laman.
- Nagse-serve rin ng A2UI sa `/__openclaw__/a2ui/`.
- Ang mga pagbabago ay nangangailangan ng pag-restart ng gateway.
- I-disable ang live reload para sa malalaking direktoryo o kapag may `EMFILE` na error.

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

- `minimal` (default): hindi isinasama ang `cliPath` + `sshPort` sa TXT records.
- `full`: isama ang `cliPath` + `sshPort`.
- Ang hostname ay default na `openclaw`. I-override gamit ang `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Sumusulat ng unicast DNS-SD zone sa ilalim ng `~/.openclaw/dns/`. Para sa cross-network discovery, ipares sa isang DNS server (inirerekomenda ang CoreDNS) + Tailscale split DNS.

Setup: `openclaw dns setup --apply`.

---

## Environment

### `env` (inline na env vars)

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

- Ang mga inline na env var ay inilalapat lamang kung wala ang key sa process env.
- Mga `.env` file: CWD `.env` + `~/.openclaw/.env` (walang alinman ang nag-o-override ng umiiral na vars).
- `shellEnv`: ini-import ang mga nawawalang inaasahang key mula sa iyong login shell profile.
- Tingnan ang [Environment](/help/environment) para sa buong precedence.

### Pagpapalit ng env var

I-refer ang env vars sa anumang config string gamit ang `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Tanging uppercase na mga pangalan ang tinutugma: `[A-Z_][A-Z0-9_]*`.
- Magkakaroon ng error sa pag-load ng config kapag kulang o walang laman ang vars.
- I-escape gamit ang `$${VAR}` para sa literal na `${VAR}`.
- Gumagana kasama ang `$include`.

---

## Imbakan ng auth

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

- Ang mga auth profile kada agent ay nakaimbak sa `<agentDir>/auth-profiles.json`.
- Ini-import ang legacy OAuth mula sa `~/.openclaw/credentials/oauth.json`.
- Tingnan ang [OAuth](/concepts/oauth).

---

## Paglo-log

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

- Default na log file: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Itakda ang `logging.file` para sa isang stable na path.
- Ang `consoleLevel` ay tumataas sa `debug` kapag may `--verbose`.

---

## Wizard

Metadata na isinulat ng mga CLI wizard (`onboard`, `configure`, `doctor`):

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

## Pagkakakilanlan

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

Isinulat ng macOS onboarding assistant. Nagmula ang mga default:

- `messages.ackReaction` mula sa `identity.emoji` (fallback sa 👀)
- `mentionPatterns` mula sa `identity.name`/`identity.emoji`
- Tinatanggap ng `avatar`: workspace-relative na path, `http(s)` URL, o `data:` URI

---

## Bridge (legacy, inalis na)

Ang kasalukuyang mga build ay hindi na kasama ang TCP bridge. Ang mga node ay kumokonekta sa pamamagitan ng Gateway WebSocket. Ang mga `bridge.*` key ay hindi na bahagi ng config schema (magfa-fail ang validation hangga’t hindi inaalis; maaaring alisin ng `openclaw doctor --fix` ang mga unknown key).

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

- `sessionRetention`: gaano katagal itatago ang mga natapos na cron session bago i-prune. Default: `24h`.

Tingnan ang [Cron Jobs](/automation/cron-jobs).

---

## Mga template variable ng media model

Mga template placeholder na ine-expand sa `tools.media.*.models[].args`:

| Variable           | Paglalarawan                                                                                  |
| ------------------ | --------------------------------------------------------------------------------------------- |
| `{{Body}}`         | Buong katawan ng papasok na mensahe                                                           |
| `{{RawBody}}`      | Hilaw na katawan (walang history/sender wrappers)                          |
| `{{BodyStripped}}` | Katawan na tinanggalan ng mga pagbanggit sa grupo                                             |
| `{{From}}`         | Identifier ng nagpadala                                                                       |
| `{{To}}`           | Identifier ng destinasyon                                                                     |
| `{{MessageSid}}`   | ID ng mensahe sa channel                                                                      |
| `{{SessionId}}`    | UUID ng kasalukuyang session                                                                  |
| `{{IsNewSession}}` | `"true"` kapag may bagong session na nalikha                                                  |
| `{{MediaUrl}}`     | Pseudo-URL ng papasok na media                                                                |
| `{{MediaPath}}`    | Lokal na path ng media                                                                        |
| `{{MediaType}}`    | Uri ng media (image/audio/document/…)                                      |
| `{{Transcript}}`   | Transcript ng audio                                                                           |
| `{{Prompt}}`       | Na-resolve na media prompt para sa mga entry ng CLI                                           |
| `{{MaxChars}}`     | Na-resolve na maximum na bilang ng output characters para sa mga entry ng CLI                 |
| `{{ChatType}}`     | `"direct"` or `"group"`                                                                       |
| `{{GroupSubject}}` | Paksa ng grupo (ayon sa pinakamahusay na pagsisikap)                       |
| `{{GroupMembers}}` | Preview ng mga miyembro ng grupo (ayon sa pinakamahusay na pagsisikap)     |
| `{{SenderName}}`   | Display name ng nagpadala (ayon sa pinakamahusay na pagsisikap)            |
| `{{SenderE164}}`   | Numero ng telepono ng nagpadala (ayon sa pinakamahusay na pagsisikap)      |
| `{{Provider}}`     | Pahiwatig ng provider (whatsapp, telegram, discord, atbp.) |

---

## Kasama sa config ang (`$include`)

Hatiin ang config sa maraming file:

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

**Gawi ng pag-merge:**

- Isang file: pinapalitan ang nakapaloob na object.
- Array ng mga file: deep-merged ayon sa pagkakasunod-sunod (ang mas huli ang nangingibabaw sa nauna).
- Mga sibling key: minemerge pagkatapos ng includes (pinapalitan ang mga halagang mula sa include).
- Mga nested include: hanggang 10 antas ang lalim.
- Mga path: relative (mula sa file na nag-include), absolute, o `../` na reference sa parent.
- Mga error: malinaw na mensahe para sa nawawalang file, parse error, at circular include.

---

_Kaugnay: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
