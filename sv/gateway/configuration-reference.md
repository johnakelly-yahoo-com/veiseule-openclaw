---
title: "Konfigurationsreferens"
description: "Fullständig fält-för-fält-referens för ~/.openclaw/openclaw.json"
---

# Konfigurationsreferens

Varje fält som finns i `~/.openclaw/openclaw.json`. För en uppgiftsorienterad översikt, se [Configuration](/gateway/configuration).

Konfigurationsformatet är **JSON5** (kommentarer + avslutande kommatecken tillåtna). Alla fält är valfria — OpenClaw använder säkra standardvärden när de utelämnas.

---

## Kanaler

Varje kanal startar automatiskt när dess konfigurationssektion finns (om inte `enabled: false`).

### DM- och gruppåtkomst

Alla kanaler stöder DM-policyer och gruppolicyer:

| DM-policy                               | Beteende                                                                         |
| --------------------------------------- | -------------------------------------------------------------------------------- |
| `pairing` (standard) | Okända avsändare får en engångskod för parkoppling; ägaren måste godkänna        |
| `allowlist`                             | Endast avsändare i `allowFrom` (eller parkopplad allow-lista) |
| `open`                                  | Tillåt alla inkommande DM (kräver `allowFrom: ["*"]`)         |
| `disabled`                              | Ignorera alla inkommande DM                                                      |

| Gruppolicy                                | Beteende                                                                                 |
| ----------------------------------------- | ---------------------------------------------------------------------------------------- |
| `allowlist` (standard) | Endast grupper som matchar den konfigurerade allowlistan                                 |
| `open`                                    | Kringgå gruppers tillåtelselistor (mention-gating gäller fortfarande) |
| `disabled`                                | Blockera alla grupp-/rumsmeddelanden                                                     |

<Note>
`channels.defaults.groupPolicy` anger standardvärdet när en providers `groupPolicy` inte är satt.
Parningskoder upphör att gälla efter 1 timme. Väntande DM-parningsförfrågningar är begränsade till **3 per kanal**.
Slack/Discord har en särskild reservlösning: om deras provider-sektion saknas helt kan runtime group policy lösas till `open` (med en startvarning).
</Note>

### WhatsApp

WhatsApp körs via gatewayens webbkanal (Baileys Web). Den startar automatiskt när en länkad session finns.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blå bockar (false i self-chat-läge)
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

- Utgående kommandon använder kontot `default` om det finns; annars det första konfigurerade konto-id:t (sorterat).
- Äldre Baileys-autentiseringskatalog för enskilt konto migreras av `openclaw doctor` till `whatsapp/default`.
- Per-konto-åsidosättningar: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>`.allowFrom\`.

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
          systemPrompt: "Håll svaren korta.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Håll dig till ämnet.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git-säkerhetskopiering" },
        { command: "generate", description: "Skapa en bild" },
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

- Bot-token: `channels.telegram.botToken` eller `channels.telegram.tokenFile`, med `TELEGRAM_BOT_TOKEN` som reserv för standardkontot.
- `configWrites: false` blockerar Telegram-initierade konfigurationsändringar (supergroup-ID-migreringar, `/config set|unset`).
- Telegram stream-förhandsvisningar använder `sendMessage` + `editMessageText` (fungerar i direkt- och gruppchattar).
- Retry-policy: se [Retry policy](/concepts/retry).

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
              systemPrompt: "Endast korta svar.",
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

- Token: `channels.discord.token`, med `DISCORD_BOT_TOKEN` som reserv för standardkontot.
- Använd `user:<id>` (DM) eller `channel:<id>` (guild-kanal) för leveransmål; rena numeriska ID:n avvisas.
- Guild-sluggar är gemener med mellanslag ersatta av `-`; kanalnycklar använder det slugifierade namnet (utan `#`). Föredra guild-ID:n.
- Meddelanden skrivna av boten ignoreras som standard. `allowBots: true` aktiverar dem (egna meddelanden filtreras fortfarande bort).
- `maxLinesPerMessage` (standard 17) delar upp långa meddelanden även när de är under 2000 tecken.
- `channels.discord.ui.components.accentColor` anger accentfärgen för Discord components v2-containrar.

**Lägen för reaktionsnotiser:** `off` (inga), `own` (botens meddelanden, standard), `all` (alla meddelanden), `allowlist` (från `guilds.<id>`.users\` på alla meddelanden).

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

- Service account-JSON: inline (`serviceAccount`) eller filbaserad (`serviceAccountFile`).
- Miljövariabel-reserver: `GOOGLE_CHAT_SERVICE_ACCOUNT` eller `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Använd `spaces/<spaceId>` eller `users/<userId|email>` för leveransmål.

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

- **Socket mode** kräver både `botToken` och `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` för standardkontots miljövariabel‑fallback).
- **HTTP mode** kräver `botToken` samt `signingSecret` (på rotnivå eller per konto).
- `configWrites: false` blockerar konfigurationsändringar som initieras från Slack.
- Använd `user:<id>` (DM) eller `channel:<id>` för leveransmål.

**Lägen för reaktionsnotiser:** `off`, `own` (standard), `all`, `allowlist` (från `reactionAllowlist`).

**Trådisolering för sessioner:** `thread.historyScope` är per tråd (standard) eller delad över kanalen. `thread.inheritParent` kopierar huvudkanalens konversation till nya trådar.

| Åtgärdsgrupp | Standard  | Anteckningar                |
| ------------ | --------- | --------------------------- |
| reaktioner   | aktiverad | Reagera + lista reaktioner  |
| meddelanden  | aktiverad | Läsa/skicka/redigera/radera |
| nålar        | aktiverad | Fäst/ta bort/lista          |
| medlemsinfo  | aktiverad | Medlemsinformation          |
| emojiLista   | aktiverad | Anpassad emoji‑lista        |

### Mattermost

Mattermost levereras som ett plugin: `openclaw plugins install @openclaw/mattermost`.

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

Chattlägen: `oncall` (svara vid @‑omnämnande, standard), `onmessage` (varje meddelande), `onchar` (meddelanden som börjar med trigger‑prefix).

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

**Lägen för reaktionsnotiser:** `off`, `own` (standard), `all`, `allowlist` (från `reactionAllowlist`).

### iMessage

OpenClaw startar `imsg rpc` (JSON-RPC över stdio). Ingen daemon eller port krävs.

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

- Kräver Full Disk Access till Messages‑databasen.
- Föredra `chat_id:<id>`-mål. Använd `imsg chats --limit 20` för att lista chattar.
- `cliPath` kan peka på en SSH-wrapper; ange `remoteHost` för att hämta SCP-bilagor.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Flera konton (alla kanaler)

Kör flera konton per kanal (var och en med sitt eget `accountId`):

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

- `default` används när `accountId` utelämnas (CLI + routing).
- Miljövariabel-tokens gäller endast för **default**-kontot.
- Grundläggande kanalinställningar gäller för alla konton om de inte åsidosätts per konto.
- Använd `bindings[].match.accountId` för att dirigera varje konto till en annan agent.

### Mention-gating i gruppchatt

Gruppmeddelanden kräver som standard **omnämnande** (metadata-omnämnande eller regex-mönster). Gäller för gruppchattar i WhatsApp, Telegram, Discord, Google Chat och iMessage.

**Typer av omnämnanden:**

- **Metadata-omnämnanden**: Plattformens inbyggda @-omnämnanden. Ignoreras i WhatsApp self-chat-läge.
- **Textmönster**: Regex-mönster i `agents.list[].groupChat.mentionPatterns`. Kontrolleras alltid.
- Mention-gating tillämpas endast när det är möjligt att detektera (inbyggda omnämnanden eller minst ett mönster).

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

`messages.groupChat.historyLimit` anger den globala standardinställningen. Kanaler kan åsidosätta med `channels.<channel>` `.historyLimit` (eller per konto). Sätt `0` för att inaktivera.

#### Gränser för DM-historik

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

Upplösning: per-DM-åsidosättning → leverantörens standard → ingen gräns (allt behålls).

Stöds: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat-läge

Inkludera ditt eget nummer i `allowFrom` för att aktivera self-chat-läge (ignorerar inbyggda @-omnämnanden, svarar endast på textmönster):

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

### Kommandon (hantering av chattkommandon)

```json5
{
  commands: {
    native: "auto", // registrera inbyggda kommandon när det stöds
    text: true, // tolka /kommandon i chattmeddelanden
    bash: false, // tillåt ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // tillåt /config
    debug: false, // tillåt /debug
    restart: false, // tillåt /restart + omstartsverktyg för gateway
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Textkommandon måste vara **fristående** meddelanden som börjar med `/`.
- `native: "auto"` aktiverar inbyggda kommandon för Discord/Telegram, lämnar Slack avstängt.
- Åsidosätt per kanal: `channels.discord.commands.native` (bool eller `"auto"`). `false` rensar tidigare registrerade kommandon.
- `channels.telegram.customCommands` lägger till extra menyval för Telegram-boten.
- `bash: true` aktiverar `!` <cmd>`för värdskalet. Kräver`tools.elevated.enabled`och avsändare i`tools.elevated.allowFrom.<channel>\`.
- `config: true` aktiverar `/config` (läser/skriver `openclaw.json`).
- `channels.<provider>`.configWrites\` styr konfigurationsändringar per kanal (standard: true).
- `allowFrom` är per leverantör. När den är satt är den den **enda** auktorisationskällan (kanalernas tillåtelselistor/parkoppling och `useAccessGroups` ignoreras).
- `useAccessGroups: false` tillåter kommandon att kringgå åtkomstgruppspolicys när `allowFrom` inte är satt.

</Accordion>

---

## Agentstandarder

### `agents.defaults.workspace`

Standard: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Valfri repository-rot som visas i systempromptens Runtime-rad. Om den inte är satt upptäcker OpenClaw den automatiskt genom att gå uppåt från arbetsytan.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Inaktiverar automatisk skapande av arbetsytans bootstrap-filer (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maximalt antal tecken per bootstrap-fil i arbetsytan innan trunkering. Standard: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Maximalt totalt antal tecken som injiceras över alla bootstrap-filer i arbetsytan. Standard: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Tidszon för systempromptens kontext (inte meddelandets tidsstämplar). Faller tillbaka till värdens tidszon.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Tidsformat i systemprompten. Standard: `auto` (OS-inställning).

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

- `model.primary`: format `provider/model` (t.ex. `anthropic/claude-opus-4-6`). Om du utelämnar leverantören antar OpenClaw `anthropic` (föråldrat).
- `models`: den konfigurerade modellkatalogen och tillåtelselistan för `/model`. Varje post kan innehålla `alias` (genväg) och `params` (leverantörsspecifika: `temperature`, `maxTokens`).
- `imageModel`: används endast om den primära modellen saknar bildinmatning.
- `maxConcurrent`: max antal parallella agentkörningar över sessioner (varje session serialiseras fortfarande). Standard: 1.

**Inbyggda aliasgenvägar** (gäller endast när modellen finns i `agents.defaults.models`):

| Alias          | Modell                          |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Dina konfigurerade alias har alltid företräde framför standardvärdena.

Z.AI GLM-4.x-modeller aktiverar automatiskt thinking-läge om du inte anger `--thinking off` eller själv definierar `agents.defaults.models["zai/<model>"].params.thinking`.

### `agents.defaults.cliBackends`

Valfria CLI-backends för textbaserade fallback-körningar (inga tool-anrop). Användbart som reserv när API-leverantörer fallerar.

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

- CLI-backends är i första hand textbaserade; tools är alltid inaktiverade.
- Sessioner stöds när `sessionArg` är angiven.
- Bildvidarebefordran stöds när `imageArg` accepterar filsökvägar.

### `agents.defaults.heartbeat`

Periodiska heartbeat-körningar.

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

- `every`: tidsangivelse (ms/s/m/h). Standard: `30m`.
- Per agent: ange `agents.list[].heartbeat`. När någon agent definierar `heartbeat` körs **endast dessa agenter** heartbeat.
- Heartbeats kör fullständiga agentvarv — kortare intervaller förbrukar fler tokens.

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

- `mode`: `default` eller `safeguard` (sammanfattning i delar för långa historiker). Se [Compaction](/concepts/compaction).
- `memoryFlush`: tyst agentiskt steg före automatisk komprimering för att lagra beständiga minnen. Hoppas över när arbetsytan är skrivskyddad.

### `agents.defaults.contextPruning`

Rensar **gamla verktygsresultat** från minneskontexten innan den skickas till LLM. Modifierar **inte** sessionshistoriken på disk.

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

- `mode: "cache-ttl"` aktiverar rensningskörningar.
- `ttl` styr hur ofta rensning kan köras igen (efter senaste cache-åtkomst).
- Rensning mjuktrimmar först överdimensionerade verktygsresultat och hårdrensar sedan äldre verktygsresultat vid behov.

**Mjuktrimning** behåller början + slutet och infogar `...` i mitten.

**Hårdrensning** ersätter hela verktygsresultatet med platshållaren.

Obs:

- Bildblock trimmas/rensas aldrig.
- Kvoter är teckenbaserade (ungefärliga), inte exakta token-antal.
- Om färre än `keepLastAssistants` assistentmeddelanden finns hoppar rensningen över.

</Accordion>

Se [Session Pruning](/concepts/session-pruning) för beteendedetaljer.

### Blockstreaming

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

- Kanaler som inte är Telegram kräver explicit `*.blockStreaming: true` för att aktivera block­svar.
- Kanalåsidosättningar: `channels.<channel>`.blockStreamingCoalesce`(och varianter per konto). Signal/Slack/Discord/Google Chat har som standard`minChars: 1500\`.
- `humanDelay`: slumpmässig paus mellan blocksvar. `natural` = 800–2500 ms. Åsidosättning per agent: `agents.list[].humanDelay`.

Se [Streaming](/concepts/streaming) för beteende- och chunkningsdetaljer.

### Skrivindikatorer

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

- Standard: `instant` för direktchattar/omnämnanden, `message` för gruppchattar utan omnämnande.
- Åsidosättningar per session: `session.typingMode`, `session.typingIntervalSeconds`.

Se [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Valfri **Docker-sandboxning** för den inbäddade agenten. Se [Sandboxing](/gateway/sandboxing) för den fullständiga guiden.

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

**Åtkomst till arbetsyta:**

- `none`: sandbox-arbetsyta per scope under `~/.openclaw/sandboxes`
- `ro`: sandbox-arbetsyta på `/workspace`, agentens arbetsyta monterad skrivskyddad på `/agent`
- `rw`: agentens arbetsyta monterad med läs/skriv på `/workspace`

**Omfattning:**

- `session`: container + arbetsyta per session
- `agent`: en container + arbetsyta per agent (standard)
- `shared`: delad container och arbetsyta (ingen isolering mellan sessioner)

**`setupCommand`** körs en gång efter att containern har skapats (via `sh -lc`). Kräver nätverksutgående trafik, skrivbar root och root‑användare.

**Containrar använder som standard `network: "none"`** — sätt till `"bridge"` om agenten behöver utgående åtkomst.

**Inkommande bilagor** placeras i `media/inbound/*` i den aktiva arbetsytan.

**`docker.binds`** monterar ytterligare värdkataloger; globala och agentspecifika binds slås samman.

**Sandboxad webbläsare** (`sandbox.browser.enabled`): Chromium + CDP i en container. noVNC‑URL injiceras i systemprompten. Kräver inte `browser.enabled` i huvudkonfigurationen.

- `allowHostControl: false` (standard) blockerar sandboxade sessioner från att rikta sig mot värdens webbläsare.
- `sandbox.browser.binds` monterar ytterligare värdkataloger endast i sandbox‑webbläsarcontainern. När den är satt (inklusive `[]`) ersätter den `docker.binds` för webbläsarcontainern.

</Accordion>

Bygg images:

```bash
scripts/sandbox-setup.sh           # huvud‑sandbox‑image
scripts/sandbox-browser-setup.sh   # valfri webbläsar‑image
```

### `agents.list` (agentspecifika åsidosättningar)

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
        model: "anthropic/claude-opus-4-6", // eller { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "hjälpsam sengångare",
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

- `id`: stabilt agent‑id (obligatoriskt).
- `default`: när flera är satta vinner den första (varning loggas). Om ingen är satt används första posten i listan som standard.
- `model`: strängform åsidosätter endast `primary`; objektform `{ primary, fallbacks }` åsidosätter båda (`[]` inaktiverar globala fallbacks).
- `identity.avatar`: sökväg relativ till arbetsytan, `http(s)`‑URL eller `data:`‑URI.
- `identity` härleder standardvärden: `ackReaction` från `emoji`, `mentionPatterns` från `name`/`emoji`.
- `subagents.allowAgents`: tillåten lista med agent‑id:n för `sessions_spawn` (`["*"]` = valfri; standard: endast samma agent).

---

## Multi‑agent‑routning

Kör flera isolerade agenter i en och samma Gateway. Se [Multi-Agent](/concepts/multi-agent).

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

### Bindningsfält för matchning

- `match.channel` (obligatoriskt)
- `match.accountId` (valfritt; `*` = valfritt konto; utelämnad = standardkonto)
- `match.peer` (valfritt; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (valfritt; kanalspecifikt)

**Deterministisk matchningsordning:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exakt, ingen peer/guild/team)
5. `match.accountId: "*"` (kanalövergripande)
6. Standardagent

Inom varje nivå vinner den första matchande `bindings`-posten.

### Åtkomstprofiler per agent

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

Se [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) för detaljer om prioritet.

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

- **`dmScope`**: hur DM grupperas.
  - `main`: alla DM delar huvudsessionen.
  - `per-peer`: isolera per avsändar-id över kanaler.
  - `per-channel-peer`: isolera per kanal + avsändare (rekommenderas för inkorgar med flera användare).
  - `per-account-channel-peer`: isolera per konto + kanal + avsändare (rekommenderas för flera konton).
- **`identityLinks`**: mappa kanoniska id:n till leverantörsprefixade parter för delning av sessioner över kanaler.
- **`reset`**: primär återställningspolicy. `daily` återställer vid `atHour` lokal tid; `idle` återställer efter `idleMinutes`. Om båda är konfigurerade gäller den som löper ut först.
- **`resetByType`**: åsidosättningar per typ (`direct`, `group`, `thread`). Äldre `dm` accepteras som alias för `direct`.
- **`mainKey`**: äldre fält. Runtime använder nu alltid `"main"` för huvudbucketen för direktchatt.
- **`sendPolicy`**: matcha efter `channel`, `chatType` (`direct|group|channel`, med äldre `dm` som alias), `keyPrefix` eller `rawKeyPrefix`. Första nekande regeln gäller.
- **`maintenance`**: `warn` varnar den aktiva sessionen vid borttagning; `enforce` tillämpar rensning och rotering.

</Accordion>

---

## Meddelanden

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

### Svarsprefix

Överskrivningar per kanal/konto: `channels.<channel>` `.responsePrefix`, `channels.<channel>` `.accounts.<id>` `.responsePrefix`.

Upplösning (mest specifik gäller): konto → kanal → global. `""` inaktiverar och stoppar kaskaden. `"auto"` härleder `[{identity.name}]`.

**Mallvariabler:**

| Variabel          | Beskrivning                     | Exempel                                 |
| ----------------- | ------------------------------- | --------------------------------------- |
| `{model}`         | Kort modellnamn                 | `claude-opus-4-6`                       |
| `{modelFull}`     | Fullständig modellidentifierare | `anthropic/claude-opus-4-6`             |
| `{provider}`      | Leverantörsnamn                 | `anthropic`                             |
| `{thinkingLevel}` | Nuvarande tänkningsnivå         | `high`, `low`, `off`                    |
| `{identity.name}` | Agentens identitetsnamn         | (samma som `"auto"`) |

Variabler är skiftlägesokänsliga. `{think}` är ett alias för `{thinkingLevel}`.

### Bekräftelsereaktion

- Standard är den aktiva agentens `identity.emoji`, annars `"👀"`. Ange `""` för att inaktivera.
- Per-kanal-åsidosättningar: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`.
- Upplösningsordning: konto → kanal → `messages.ackReaction` → identitets‑fallback.
- Omfattning: `group-mentions` (standard), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: tar bort bekräftelse efter svar (endast Slack/Discord/Telegram/Google Chat).

### Inkommande debounce

Samlar snabba textmeddelanden från samma avsändare till en enda agentomgång. Media/bilagor skickas direkt. Styrkommandon kringgår debounce.

### TTS (text-till-tal)

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

- `auto` styr automatisk TTS. `/tts off|always|inbound|tagged` åsidosätter per session.
- `summaryModel` åsidosätter `agents.defaults.model.primary` för automatisk sammanfattning.
- API-nycklar faller tillbaka till `ELEVENLABS_API_KEY`/`XI_API_KEY` och `OPENAI_API_KEY`.

---

## Talk

Standardinställningar för Talk-läge (macOS/iOS/Android).

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

- Röst-ID:n faller tillbaka på `ELEVENLABS_VOICE_ID` eller `SAG_VOICE_ID`.
- `apiKey` faller tillbaka på `ELEVENLABS_API_KEY`.
- `voiceAliases` gör att Talk-direktiv kan använda användarvänliga namn.

---

## Verktyg

### Verktygsprofiler

`tools.profile` anger en grundläggande tillåtslista före `tools.allow`/`tools.deny`:

| Profil      | Innehåller                                                                                |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | endast `session_status`                                                                   |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Ingen begränsning (samma som ej angiven)                               |

### Verktygsgrupper

| Grupp              | Verktyg                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` accepteras som ett alias för `exec`)        |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Alla inbyggda verktyg (exkluderar leverantörsplugins)                 |

### `tools.allow` / `tools.deny`

Global verktygs‑tillåt/nekande‑policy (nekande vinner). Skiftlägesokänslig, stöder `*`‑wildcards. Tillämpas även när Docker‑sandbox är avstängd.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Begränsa ytterligare verktyg för specifika leverantörer eller modeller. Ordning: basprofil → leverantörsprofil → tillåt/nekande.

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

Styr förhöjd (värd) exec‑åtkomst:

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

- Åsidosättning per agent (`agents.list[].tools.elevated`) kan endast ytterligare begränsa.
- `/elevated on|off|ask|full` sparar tillstånd per session; inline‑direktiv gäller endast för ett enskilt meddelande.
- Förhöjd `exec` körs på värden och kringgår sandboxing.

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
        apiKey: "brave_api_key", // eller BRAVE_API_KEY env
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

Konfigurerar förståelse av inkommande media (bild/ljud/video):

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

**Leverantörspost** (`type: "provider"` eller utelämnad):

- `provider`: API‑leverantörs‑id (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc.)
- `model`: åsidosättning av modell‑id
- `profile` / `preferredProfile`: val av autentiseringsprofil

**CLI‑post** (`type: "cli"`):

- `command`: körbar fil att köra
- `args`: mallbaserade argument (stöder `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc.)

**Gemensamma fält:**

- `capabilities`: valfri lista (`image`, `audio`, `video`). Standardvärden: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: åsidosättningar per post.
- Fel leder till att nästa post används.

Leverantörsautentisering följer standardordning: auth‑profiler → miljövariabler → `models.providers.*.apiKey`.

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

- `model`: standardmodell för skapade underagenter. Om utelämnad ärver underagenter anroparens modell.
- Verktygspolicy per underagent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Anpassade leverantörer och bas-URL:er

OpenClaw använder pi-coding-agent-modellkatalogen. Lägg till anpassade leverantörer via `models.providers` i konfigurationen eller `~/.openclaw/agents/<agentId>/agent/models.json`.

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

- Använd `authHeader: true` + `headers` för anpassade autentiseringsbehov.
- Åsidosätt agentens konfigurationsrot med `OPENCLAW_AGENT_DIR` (eller `PI_CODING_AGENT_DIR`).

### Exempel på leverantörer

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

Använd `cerebras/zai-glm-4.7` för Cerebras; `zai/glm-4.7` för Z.AI direkt.

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

Ange `OPENCODE_API_KEY` (eller `OPENCODE_ZEN_API_KEY`). Genväg: `openclaw onboard --auth-choice opencode-zen`.

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

Ange `ZAI_API_KEY`. `z.ai/*` och `z-ai/*` är accepterade alias. Genväg: `openclaw onboard --auth-choice zai-api-key`.

- Allmän endpoint: `https://api.z.ai/api/paas/v4`
- Kodningsendpoint (standard): `https://api.z.ai/api/coding/paas/v4`
- För den allmänna endpointen, definiera en anpassad leverantör med åsidosatt bas-URL.

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

För China-endpointen: `baseUrl: "https://api.moonshot.cn/v1"` eller `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic-kompatibel, inbyggd leverantör. Genväg: `openclaw onboard --auth-choice kimi-code-api-key`.

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

Bas-URL ska utelämna `/v1` (Anthropic-klienten lägger till det). Genväg: `openclaw onboard --auth-choice synthetic-api-key`.

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

Ange `MINIMAX_API_KEY`. Genväg: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Se [Local Models](/gateway/local-models). TL;DR: kör MiniMax M2.1 via LM Studio Responses API på kraftfull hårdvara; behåll hostade modeller sammanslagna som reserv.

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

- `allowBundled`: valfri allowlist endast för paketerade skills (hanterade/arbetsytans skills påverkas inte).
- `entries.<skillKey>`.enabled: false\` inaktiverar en skill även om den är paketerad/installerad.
- `entries.<skillKey>`.apiKey\`: förenkling för skills som deklarerar en primär miljövariabel.

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

- Laddas från `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, samt `plugins.load.paths`.
- **Konfigurationsändringar kräver en omstart av gatewayen.**
- `allow`: valfri allowlist (endast listade plugins laddas). `deny` har företräde.

Se [Plugins](/tools/plugin).

---

## Webbläsare

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

- `evaluateEnabled: false` inaktiverar `act:evaluate` och `wait --fn`.
- Fjärrprofiler är endast för anslutning (start/stop/reset inaktiverat).
- Automatisk detekteringsordning: standardwebbläsare om Chromium-baserad → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Kontrolltjänst: endast loopback (port härledd från `gateway.port`, standard `18791`).

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

- `seamColor`: accentfärg för det inbyggda appgränssnittet (Talk Mode-bubblans toning m.m.).
- `assistant`: åsidosätt identiteten i Control UI. Faller tillbaka till den aktiva agentens identitet.

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

- `mode`: `local` (kör gateway) eller `remote` (anslut till fjärrgateway). Gatewayen vägrar starta om inte `local`.
- `port`: en enda multiplexad port för WS + HTTP. Prioritet: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (standard), `lan` (`0.0.0.0`), `tailnet` (endast Tailscale-IP) eller `custom`.
- **Auth**: krävs som standard. Bindningar som inte är loopback kräver en delad token/lösenord. Onboarding-guiden genererar en token som standard.
- `auth.mode: "trusted-proxy"`: delegera auth till en identitetsmedveten omvänd proxy och lita på identitetsheaders från `gateway.trustedProxies` (se [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: när `true` uppfyller Tailscale Serve-identitetsheaders auth (verifieras via `tailscale whois`). Standard är `true` när `tailscale.mode = "serve"`.
- `auth.rateLimit`: valfri begränsning av misslyckade autentiseringsförsök. Tillämpas per klient-IP och per auth-omfattning (delad hemlighet och enhetstoken spåras oberoende). Blockerade försök returnerar `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` är som standard `true`; sätt till `false` när du avsiktligt vill att även localhost-trafik ska begränsas (för testmiljöer eller strikta proxyinstallationer).
- `tailscale.mode`: `serve` (endast tailnet, loopback-bindning) eller `funnel` (publik, kräver auth).
- `remote.transport`: `ssh` (standard) eller `direct` (ws/wss). För `direct` måste `remote.url` vara `ws://` eller `wss://`.
- `gateway.remote.token` är endast för fjärranrop via CLI; aktiverar inte lokal gateway-auth.
- `trustedProxies`: IP-adresser för omvänd proxy som terminerar TLS. Lista endast proxies som du kontrollerar.
- `gateway.tools.deny`: extra verktygsnamn som blockeras för HTTP `POST /tools/invoke` (utökar standardlistan för nekade).
- `gateway.tools.allow`: ta bort verktygsnamn från standardlistan för nekade HTTP-anrop.

</Accordion>

### OpenAI-kompatibla endpoints

- Chat Completions: inaktiverad som standard. Aktivera med `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- URL-härdning för Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Isolering av flera instanser

Kör flera gateways på en och samma värd med unika portar och state-kataloger:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Bekvämlighetsflaggor: `--dev` (använder `~/.openclaw-dev` + port `19001`), `--profile <name>` (använder `~/.openclaw-<name>`).

Se [Multiple Gateways](/gateway/multiple-gateways).

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

Auth: `Authorization: Bearer <token>` eller `x-openclaw-token: <token>`.

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` från begärans payload accepteras endast när `hooks.allowRequestSessionKey=true` (standard: `false`).
- `POST /hooks/<name>` → löses via `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` matchar undersökvägen efter `/hooks` (t.ex. `/hooks/gmail` → `gmail`).
- `match.source` matchar ett payload-fält för generiska sökvägar.
- Mallarna som `{{messages[0].subject}}` läses från payloaden.
- `transform` kan peka på en JS/TS-modul som returnerar en hook-åtgärd.
  - `transform.module` måste vara en relativ sökväg och hållas inom `hooks.transformsDir` (absoluta sökvägar och sökvägstraversering avvisas).
- `agentId` dirigerar till en specifik agent; okända ID:n faller tillbaka till standardagenten.
- `allowedAgentIds`: begränsar explicit dirigering (`*` eller utelämnad = tillåt alla, `[]` = neka alla).
- `defaultSessionKey`: valfri fast sessionsnyckel för hook-agentkörningar utan explicit `sessionKey`.
- `allowRequestSessionKey`: tillåt anrop till `/hooks/agent` att ange `sessionKey` (standard: `false`).
- `allowedSessionKeyPrefixes`: valfri prefix‑allowlist för explicita `sessionKey`‑värden (begäran + mappning), t.ex. `["hook:"]`.
- `deliver: true` skickar det slutliga svaret till en kanal; `channel` är som standard `last`.
- `model` åsidosätter LLM för denna hook‑körning (måste vara tillåten om en modellkatalog är angiven).

</Accordion>

### Gmail-integration

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

- Gateway startar automatiskt `gog gmail watch serve` vid uppstart när den är konfigurerad. Sätt `OPENCLAW_SKIP_GMAIL_WATCHER=1` för att inaktivera.
- Kör inte en separat `gog gmail watch serve` parallellt med Gateway.

---

## Canvas-värd

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Serverar agentredigerbar HTML/CSS/JS och A2UI över HTTP via Gateway-porten:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Endast lokalt: behåll `gateway.bind: "loopback"` (standard).
- Icke-loopback-bindningar: canvas-rutter kräver Gateway-autentisering (token/lösenord/trusted-proxy), samma som andra Gateway HTTP-ytor.
- Node WebViews skickar vanligtvis inte autentiseringshuvuden; efter att en nod har parats och anslutits tillåter Gateway en privat-IP-reservlösning så att noden kan läsa in canvas/A2UI utan att läcka hemligheter i URL:er.
- Injicerar live-reload-klient i serverad HTML.
- Skapar automatiskt en start-`index.html` när katalogen är tom.
- Serverar även A2UI på `/__openclaw__/a2ui/`.
- Ändringar kräver en omstart av Gateway.
- Inaktivera live reload för stora kataloger eller vid `EMFILE`-fel.

---

## Upptäckt

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

- `minimal` (standard): utelämnar `cliPath` + `sshPort` från TXT-poster.
- `full`: inkluderar `cliPath` + `sshPort`.
- Värdnamn är som standard `openclaw`. Åsidosätt med `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Skriver en unicast DNS-SD-zon under `~/.openclaw/dns/`. För upptäckt över flera nätverk, kombinera med en DNS-server (CoreDNS rekommenderas) + Tailscale split DNS.

Installation: `openclaw dns setup --apply`.

---

## Miljö

### `env` (inbäddade miljövariabler)

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

- Inbäddade miljövariabler tillämpas endast om processens miljö saknar nyckeln.
- `.env`-filer: CWD `.env` + `~/.openclaw/.env` (ingen av dem åsidosätter befintliga variabler).
- `shellEnv`: importerar saknade förväntade nycklar från din inloggningsskalsprofil.
- Se [Environment](/help/environment) för fullständig prioriteringsordning.

### Substitution av miljövariabler

Referera till miljövariabler i valfri konfigurationssträng med `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Endast versala namn matchas: `[A-Z_][A-Z0-9_]*`.
- Saknade/tomma variabler ger ett fel vid inläsning av konfigurationen.
- Escapa med `$${VAR}` för en bokstavlig `${VAR}`.
- Fungerar med `$include`.

---

## Autentiseringslagring

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

- Per-agent-autentiseringsprofiler lagras i `<agentDir>/auth-profiles.json`.
- Äldre OAuth-importer från `~/.openclaw/credentials/oauth.json`.
- Se [OAuth](/concepts/oauth).

---

## Loggning

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

- Standardloggfil: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Ange `logging.file` för en stabil sökväg.
- `consoleLevel` höjs till `debug` när `--verbose` används.

---

## Guide

Metadata som skrivs av CLI-guider (`onboard`, `configure`, `doctor`):

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

## Identitet

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

Skriven av macOS-onboardingassistenten. Härleder standardvärden:

- `messages.ackReaction` från `identity.emoji` (faller tillbaka till 👀)
- `mentionPatterns` från `identity.name`/`identity.emoji`
- `avatar` accepterar: arbetsyterelativ sökväg, `http(s)`-URL eller `data:`-URI

---

## Bridge (äldre, borttagen)

Nuvarande byggen inkluderar inte längre TCP-bryggan. Noder ansluter via Gateway WebSocket. `bridge.*`-nycklar ingår inte längre i konfigurationsschemat (validering misslyckas tills de tas bort; `openclaw doctor --fix` kan ta bort okända nycklar).

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

- `sessionRetention`: hur länge slutförda cron-sessioner ska behållas innan de rensas bort. Standard: `24h`.

Se [Cron Jobs](/automation/cron-jobs).

---

## Mallvariabler för mediamodell

Mallplatshållare som expanderas i `tools.media.*.models[].args`:

| Variabel           | Beskrivning                                                                                 |
| ------------------ | ------------------------------------------------------------------------------------------- |
| `{{Body}}`         | Fullständig inkommande meddelandetext                                                       |
| `{{RawBody}}`      | Rå text (utan historik-/avsändaromslag)                                  |
| `{{BodyStripped}}` | Brödtext med gruppomnämnanden borttagna                                                     |
| `{{From}}`         | Avsändaridentifierare                                                                       |
| `{{To}}`           | Destinationsidentifierare                                                                   |
| `{{MessageSid}}`   | Kanalens meddelande-id                                                                      |
| `{{SessionId}}`    | Aktuell sessions-UUID                                                                       |
| `{{IsNewSession}}` | `"true"` när en ny session skapas                                                           |
| `{{MediaUrl}}`     | Pseudo-URL för inkommande media                                                             |
| `{{MediaPath}}`    | Lokal sökväg till media                                                                     |
| `{{MediaType}}`    | Mediatyp (bild/ljud/dokument/…)                                          |
| `{{Transcript}}`   | Ljudtranskription                                                                           |
| `{{Prompt}}`       | Löst mediaprompt för CLI-poster                                                             |
| `{{MaxChars}}`     | Löst maxantal tecken för CLI-poster                                                         |
| `{{ChatType}}`     | `"direct"` eller `"group"`                                                                  |
| `{{GroupSubject}}` | Gruppämne (i möjligaste mån)                                             |
| `{{GroupMembers}}` | Förhandsvisning av gruppmedlemmar (i möjligaste mån)                     |
| `{{SenderName}}`   | Avsändarens visningsnamn (i möjligaste mån)                              |
| `{{SenderE164}}`   | Avsändarens telefonnummer (i möjligaste mån)                             |
| `{{Provider}}`     | Leverantörsindikator (whatsapp, telegram, discord, etc.) |

---

## Konfigurationen inkluderar (`$include`)

Dela upp konfigurationen i flera filer:

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

**Sammanfogningsbeteende:**

- En enskild fil: ersätter det omgivande objektet.
- Array av filer: slås samman i ordning (senare skriver över tidigare).
- Syskonnycklar: slås samman efter includes (skriver över inkluderade värden).
- Nästlade includes: upp till 10 nivåer djupa.
- Sökvägar: relativa (till den inkluderande filen), absoluta eller `../`-referenser till föräldrakatalog.
- Fel: tydliga meddelanden för saknade filer, tolkningsfel och cirkulära includes.

---

_Relaterat: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

