---
title: "Konfigurationsreference"
description: "Komplet felt-for-felt-reference for ~/.openclaw/openclaw.json"
---

# Konfigurationsreference

Alle tilgængelige felter i `~/.openclaw/openclaw.json`. For en opgaveorienteret oversigt, se [Configuration](/gateway/configuration).

Konfigurationsformatet er **JSON5** (kommentarer + trailing commas er tilladt). Alle felter er valgfrie — OpenClaw bruger sikre standardværdier, når de udelades.

---

## Kanaler

Hver kanal starter automatisk, når dens konfigurationssektion findes (medmindre `enabled: false`).

### DM- og gruppeadgang

Alle kanaler understøtter DM-politikker og gruppepolitikker:

| DM-politik                              | Adfærd                                                                    |
| --------------------------------------- | ------------------------------------------------------------------------- |
| `pairing` (standard) | Ukendte afsendere får en engangs-parringskode; ejeren skal godkende       |
| `allowlist`                             | Kun afsendere i `allowFrom` (eller parret allow-lager) |
| `open`                                  | Tillad alle indgående DMs (kræver `allowFrom: ["*"]`)  |
| `disabled`                              | Ignorer alle indgående DMs                                                |

| Gruppepolitik                             | Adfærd                                                                   |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| `allowlist` (standard) | Kun grupper, der matcher den konfigurerede allowlist                     |
| `open`                                    | Omgå gruppe-allowlists (mention-gating gælder stadig) |
| `disabled`                                | Bloker alle gruppe-/rumbeskeder                                          |

<Note>
`channels.defaults.groupPolicy` angiver standarden, når en providers `groupPolicy` ikke er sat.
Parringskoder udløber efter 1 time. Afventende DM-parringsanmodninger er begrænset til **3 pr. kanal**.
Slack/Discord har en særlig fallback: hvis deres provider-sektion mangler helt, kan runtime group policy blive sat til `open` (med en advarsel ved opstart).
</Note>

### WhatsApp

WhatsApp kører via gatewayens webkanal (Baileys Web). Den starter automatisk, når der findes en tilknyttet session.

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

- Udgående kommandoer bruger som standard kontoen `default`, hvis den findes; ellers det første konfigurerede konto-id (sorteret).
- Ældre single-account Baileys auth-dir migreres af `openclaw doctor` til `whatsapp/default`.
- Per-konto-tilsidesættelser: `channels.whatsapp.accounts.<id> .sendReadReceipts`, `channels.whatsapp.accounts.<id> .dmPolicy`, `channels.whatsapp.accounts.<id> .allowFrom`.

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
          systemPrompt: "Hold svar korte.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Hold dig til emnet.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git-backup" },
        { command: "generate", description: "Opret et billede" },
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

- Bot-token: `channels.telegram.botToken` eller `channels.telegram.tokenFile`, med `TELEGRAM_BOT_TOKEN` som fallback for standardkontoen.
- `configWrites: false` blokerer Telegram-initierede konfigurationsskrivninger (supergruppe-ID-migreringer, `/config set|unset`).
- Telegram stream previews bruger `sendMessage` + `editMessageText` (fungerer i direkte chats og gruppechats).
- Retry-politik: se [Retry policy](/concepts/retry).

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
              systemPrompt: "Kun korte svar.",
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

- Token: `channels.discord.token`, med `DISCORD_BOT_TOKEN` som fallback for standardkontoen.
- Brug `user:<id>` (DM) eller `channel:<id>` (guild-kanal) som leveringsmål; rene numeriske ID’er afvises.
- Guild-slugs er med små bogstaver, og mellemrum erstattes med `-`; kanalnøgler bruger det slugificerede navn (uden `#`). Foretræk guild-ID’er.
- Beskeder skrevet af botten ignoreres som standard. `allowBots: true` aktiverer dem (egne beskeder filtreres stadig).
- `maxLinesPerMessage` (standard 17) opdeler lange beskeder, selv når de er under 2000 tegn.
- `channels.discord.ui.components.accentColor` angiver accentfarven for Discord components v2-containere.

**Reaktionsnotifikationstilstande:** `off` (ingen), `own` (bottens beskeder, standard), `all` (alle beskeder), `allowlist` (fra `guilds.<id> .users` på alle beskeder).

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

- Servicekonto-JSON: inline (`serviceAccount`) eller filbaseret (`serviceAccountFile`).
- Env-fallbacks: `GOOGLE_CHAT_SERVICE_ACCOUNT` eller `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Brug `spaces/<spaceId>` eller `users/<userId|email>` som leveringsmål.

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

- **Socket-tilstand** kræver både `botToken` og `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` for standardkontoens env-fallback).
- **HTTP-tilstand** kræver `botToken` samt `signingSecret` (på roden eller pr. konto).
- `configWrites: false` blokerer Slack-initierede konfigurationsskrivninger.
- Brug `user:<id>` (DM) eller `channel:<id>` som leveringsmål.

**Tilstande for reaktionsnotifikationer:** `off`, `own` (standard), `all`, `allowlist` (fra `reactionAllowlist`).

**Tråd-sessionsisolering:** `thread.historyScope` er pr. tråd (standard) eller delt på tværs af kanalen. `thread.inheritParent` kopierer den overordnede kanals transskription til nye tråde.

| Handlingsgruppe | Standard  | Noter                             |
| --------------- | --------- | --------------------------------- |
| reaktioner      | aktiveret | Reagér + vis reaktioner           |
| beskeder        | aktiveret | Læs/send/redigér/slet             |
| fastgørelser    | aktiveret | Fastgør/løsn/vis liste            |
| memberInfo      | aktiveret | Medlemsinfo                       |
| emojiList       | aktiveret | Liste over brugerdefinerede emoji |

### Mattermost

Mattermost leveres som et plugin: `openclaw plugins install @openclaw/mattermost`.

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

Chat-tilstande: `oncall` (svar ved @-omtale, standard), `onmessage` (hver besked), `onchar` (beskeder, der starter med trigger-præfiks).

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

**Tilstande for reaktionsnotifikationer:** `off`, `own` (standard), `all`, `allowlist` (fra `reactionAllowlist`).

### iMessage

OpenClaw starter `imsg rpc` (JSON-RPC over stdio). Ingen daemon eller port påkrævet.

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

- Kræver fuld diskadgang til Messages-databasen.
- Foretræk `chat_id:<id>`-mål. Brug `imsg chats --limit 20` til at liste chats.
- `cliPath` kan pege på en SSH-wrapper; angiv `remoteHost` for at hente vedhæftninger via SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Flere konti (alle kanaler)

Kør flere konti pr. kanal (hver med sin egen `accountId`):

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

- `default` bruges, når `accountId` udelades (CLI + routing).
- Env-tokens gælder kun for **default**-kontoen.
- Basis kanalindstillinger gælder for alle konti, medmindre de tilsidesættes pr. konto.
- Brug `bindings[].match.accountId` til at route hver konto til en anden agent.

### Omtale-krav i gruppechat

Gruppemeddelelser kræver som standard **omtale** (metadata-omtale eller regex-mønstre). Gælder for WhatsApp-, Telegram-, Discord-, Google Chat- og iMessage-gruppechats.

**Omtaletyper:**

- **Metadata-omtaler**: Platformens indbyggede @-omtaler. Ignoreres i WhatsApp self-chat-tilstand.
- **Tekstmønstre**: Regex-mønstre i `agents.list[].groupChat.mentionPatterns`. Kontrolleres altid.
- Omtale-krav håndhæves kun, når registrering er mulig (indbyggede omtaler eller mindst ét mønster).

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

`messages.groupChat.historyLimit` angiver den globale standard. Kanaler kan tilsidesætte med `channels.<channel> .historyLimit` (eller pr. konto). Sæt `0` for at deaktivere.

#### Grænser for DM-historik

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

Opløsning: pr.-DM-tilsidesættelse → udbyderens standard → ingen grænse (alt gemmes).

Understøttet: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat-tilstand

Inkludér dit eget nummer i `allowFrom` for at aktivere self-chat-tilstand (ignorerer indbyggede @-omtaler, svarer kun på tekstmønstre):

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

### Kommandoer (håndtering af chatkommandoer)

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

- Tekstkommandoer skal være **selvstændige** beskeder, der starter med `/`.
- `native: "auto"` aktiverer indbyggede kommandoer for Discord/Telegram, men lader Slack være slået fra.
- Tilsidesæt pr. kanal: `channels.discord.commands.native` (bool eller `"auto"`). `false` rydder tidligere registrerede kommandoer.
- `channels.telegram.customCommands` tilføjer ekstra Telegram-botmenuindgange.
- `bash: true` aktiverer `! <cmd>` til værtens shell. Kræver `tools.elevated.enabled` og at afsenderen er i `tools.elevated.allowFrom.<channel>`.
- `config: true` aktiverer `/config` (læser/skriver `openclaw.json`).
- `channels.<provider> .configWrites` styrer konfigurationsændringer pr. kanal (standard: true).
- `allowFrom` er pr. udbyder. Når den er sat, er den den **eneste** autorisationskilde (kanalens allowlists/parring og `useAccessGroups` ignoreres).
- `useAccessGroups: false` tillader, at kommandoer omgår access-group-politikker, når `allowFrom` ikke er sat.

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

Valgfri repository-rod, der vises på system promptens Runtime-linje. Hvis den ikke er sat, registrerer OpenClaw den automatisk ved at gå opad fra workspace-mappen.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Deaktiverer automatisk oprettelse af workspace-bootstrapfiler (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maksimalt antal tegn pr. workspace-bootstrapfil før afkortning. Standard: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Maksimalt samlet antal tegn indsat på tværs af alle workspace-bootstrapfiler. Standard: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Tidszone for system prompt-kontekst (ikke beskedtidsstempler). Falder tilbage til værtens tidszone.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Tidsformat i system prompt. Standard: `auto` (OS-præference).

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

- `model.primary`: format `provider/model` (f.eks. `anthropic/claude-opus-4-6`). Hvis du udelader provider, antager OpenClaw `anthropic` (forældet).
- `models`: det konfigurerede modelkatalog og allowlist for `/model`. Hver post kan inkludere `alias` (genvej) og `params` (providerspecifik: `temperature`, `maxTokens`).
- `imageModel`: bruges kun, hvis den primære model mangler billedinput.
- `maxConcurrent`: maks. parallelle agentkørsler på tværs af sessioner (hver session er stadig serialiseret). Standard: 1.

**Indbyggede alias-genveje** (gælder kun når modellen er i `agents.defaults.models`):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Dine konfigurerede aliasser har altid forrang over standarderne.

Z.AI GLM-4.x-modeller aktiverer automatisk thinking-tilstand, medmindre du angiver `--thinking off` eller selv definerer `agents.defaults.models["zai/<model>"].params.thinking`.

### `agents.defaults.cliBackends`

Valgfri CLI-backends til tekstbaserede fallback-kørsler (ingen tool calls). Nyttigt som backup, når API-providers fejler.

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

- CLI-backends er tekst-først; tools er altid deaktiveret.
- Sessioner understøttes, når `sessionArg` er angivet.
- Billed-pass-through understøttes, når `imageArg` accepterer filstier.

### `agents.defaults.heartbeat`

Periodiske heartbeat-kørsler.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m deaktiverer
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

- `every`: varighedsstreng (ms/s/m/h). Standard: `30m`.
- Per agent: angiv `agents.list[].heartbeat`. Når en agent definerer `heartbeat`, kører **kun disse agenter** heartbeats.
- Heartbeats kører fulde agent-turns — kortere intervaller bruger flere tokens.

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

- `mode`: `default` eller `safeguard` (opdelt opsummering for lange historikker). Se [Compaction](/concepts/compaction).
- `memoryFlush`: stille agentisk tur før auto-compaction for at gemme vedvarende minder. Springes over, når workspace er skrivebeskyttet.

### `agents.defaults.contextPruning`

Beskærer **gamle værktøjsresultater** fra konteksten i hukommelsen, før den sendes til LLM. Ændrer **ikke** sessionshistorikken på disken.

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

- `mode: "cache-ttl"` aktiverer beskæringskørsler.
- `ttl` styrer, hvor ofte beskæring kan køre igen (efter sidste cache-berøring).
- Beskæring foretager først blød trimning af for store værktøjsresultater og derefter hård rydning af ældre værktøjsresultater, hvis nødvendigt.

**Blød trimning** bevarer begyndelsen + slutningen og indsætter `...` i midten.

**Hård rydning** erstatter hele værktøjsresultatet med pladsholderen.

Bemærk:

- Billedblokke bliver aldrig trimmet/ryddet.
- Forhold er tegnbaserede (omtrentlige), ikke nøjagtige token-antal.
- Hvis der findes færre end `keepLastAssistants` assistentbeskeder, springes beskæring over.

</Accordion>

Se [Session Pruning](/concepts/session-pruning) for detaljer om adfærd.

### Blok-streaming

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

- Ikke-Telegram-kanaler kræver eksplicit `*.blockStreaming: true` for at aktivere blok-svar.
- Kanaloverstyringer: `channels.<channel>.blockStreamingCoalesce` (og varianter pr. konto). Signal/Slack/Discord/Google Chat har som standard `minChars: 1500`.
- `humanDelay`: tilfældig pause mellem blok-svar. `natural` = 800–2500 ms. Overstyring pr. agent: `agents.list[].humanDelay`.

Se [Streaming](/concepts/streaming) for detaljer om adfærd og chunking.

### Skriveindikatorer

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

- Standard: `instant` for direkte chats/omtaler, `message` for gruppechats uden omtale.
- Overstyringer pr. session: `session.typingMode`, `session.typingIntervalSeconds`.

Se [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Valgfri **Docker-sandboxing** for den indlejrede agent. Se [Sandboxing](/gateway/sandboxing) for den fulde vejledning.

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

**Workspace-adgang:**

- `none`: sandbox-workspace pr. scope under `~/.openclaw/sandboxes`
- `ro`: sandbox-arbejdsområde ved `/workspace`, agent-arbejdsområde monteret skrivebeskyttet ved `/agent`
- `rw`: agent-arbejdsområde monteret med læse-/skriveadgang ved `/workspace`

**Omfang:**

- `session`: container + arbejdsområde pr. session
- `agent`: én container + ét arbejdsområde pr. agent (standard)
- `shared`: delt container og arbejdsområde (ingen isolering mellem sessioner)

**`setupCommand`** kører én gang efter oprettelse af containeren (via `sh -lc`). Kræver netværksudgående adgang, skrivbar root og root-bruger.

**Containere bruger som standard `network: "none"`** — sæt til `"bridge"`, hvis agenten har brug for udgående adgang.

**Indgående vedhæftninger** placeres i `media/inbound/*` i det aktive arbejdsområde.

**`docker.binds`** monterer yderligere værtsmapper; globale og agentspecifikke mounts flettes sammen.

**Sandbox-browser** (`sandbox.browser.enabled`): Chromium + CDP i en container. noVNC-URL indsættes i systemprompten. Kræver ikke `browser.enabled` i hovedkonfigurationen.

- `allowHostControl: false` (standard) blokerer sandbox-sessioner fra at målrette værtsbrowseren.
- `sandbox.browser.binds` monterer yderligere værtsmapper kun i sandbox-browsercontaineren. Når den er sat (inklusive `[]`), erstatter den `docker.binds` for browsercontaineren.

</Accordion>

Byg images:

```bash
scripts/sandbox-setup.sh           # main sandbox image
scripts/sandbox-browser-setup.sh   # optional browser image
```

### `agents.list` (agent-specifikke tilsidesættelser)

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

- `id`: stabilt agent-id (påkrævet).
- `default`: når flere er angivet, vinder den første (advarsel logges). Hvis ingen er angivet, er den første post på listen standard.
- `model`: strengformat tilsidesætter kun `primary`; objektformat `{ primary, fallbacks }` tilsidesætter begge (`[]` deaktiverer globale fallbacks).
- `identity.avatar`: sti relativ til arbejdsområde, `http(s)`-URL eller `data:`-URI.
- `identity` afleder standarder: `ackReaction` fra `emoji`, `mentionPatterns` fra `name`/`emoji`.
- `subagents.allowAgents`: allowliste over agent-id’er for `sessions_spawn` (`["*"]` = enhver; standard: kun samme agent).

---

## Multi-agent routing

Kør flere isolerede agenter i én Gateway. Se [Multi-Agent](/concepts/multi-agent).

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

### Binding-matchfelter

- `match.channel` (påkrævet)
- `match.accountId` (valgfri; `*` = enhver konto; udeladt = standardkonto)
- `match.peer` (valgfri; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (valgfri; kanalspecifik)

**Deterministisk matchrækkefølge:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (præcis, ingen peer/guild/team)
5. `match.accountId: "*"` (kanal-dækkende)
6. Standardagent

Inden for hvert niveau vinder den første matchende `bindings`-post.

### Adgangsprofiler pr. agent

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

Se [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for detaljer om prioritet.

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
    mainKey: "main", // legacy (runtime uses altid "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: hvordan DM’er grupperes.
  - `main`: alle DM’er deler hovedsessionen.
  - `per-peer`: isolér efter afsender-id på tværs af kanaler.
  - `per-channel-peer`: isolér pr. kanal + afsender (anbefalet til multi-user indbakker).
  - `per-account-channel-peer`: isolér pr. konto + kanal + afsender (anbefalet til multi-account).
- **`identityLinks`**: kortlæg kanoniske id’er til provider-præfikserede peers for deling af sessioner på tværs af kanaler.
- **`reset`**: primær nulstillingspolitik. `daily` nulstiller ved `atHour` lokal tid; `idle` nulstiller efter `idleMinutes`. Når begge er konfigureret, gælder den der udløber først.
- **`resetByType`**: tilsidesættelser pr. type (`direct`, `group`, `thread`). Legacy `dm` accepteres som alias for `direct`.
- **`mainKey`**: legacy-felt. Runtime bruger nu altid `"main"` til hovedbeholderen for direkte chats.
- **`sendPolicy`**: match efter `channel`, `chatType` (`direct|group|channel`, med legacy `dm` alias), `keyPrefix` eller `rawKeyPrefix`. Første deny vinder.
- **`maintenance`**: `warn` advarer den aktive session ved fjernelse; `enforce` anvender oprydning og rotation.

</Accordion>

---

## Beskeder

```json5
{
  messages: {
    responsePrefix: "🦞", // eller "auto"
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
      debounceMs: 2000, // 0 deaktiverer
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Svarpræfiks

Tilsidesættelser pr. kanal/konto: `channels.<channel> .responsePrefix`, `channels.<channel> .accounts.<id> .responsePrefix`.

Opløsning (mest specifik vinder): konto → kanal → global. `""` deaktiverer og stopper kaskaden. `"auto"` afleder `[{identity.name}]`.

**Skabelonvariabler:**

| Variabel          | Beskrivelse             | Eksempel                                |
| ----------------- | ----------------------- | --------------------------------------- |
| `{model}`         | Kort modelnavn          | `claude-opus-4-6`                       |
| `{modelFull}`     | Fuld modelidentifikator | `anthropic/claude-opus-4-6`             |
| `{provider}`      | Udbydernavn             | `anthropic`                             |
| `{thinkingLevel}` | Nuværende tænkeniveau   | `high`, `low`, `off`                    |
| `{identity.name}` | Agentens identitetsnavn | (samme som `"auto"`) |

Variabler er ikke følsomme over for store og små bogstaver. `{think}` er et alias for `{thinkingLevel}`.

### Ack-reaktion

- Standard er den aktive agents `identity.emoji`, ellers `"👀"`. Sæt til `""` for at deaktivere.
- Per-kanal-tilsidesættelser: `channels.<channel>
   .ackReaction`, `channels.<channel>
  .accounts.<id>
  .ackReaction`.
- Opløsningsrækkefølge: konto → kanal → `messages.ackReaction` → identitetsfallback.
- Omfang: `group-mentions` (standard), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: fjerner ack efter svar (kun Slack/Discord/Telegram/Google Chat).

### Indgående debounce

Samler hurtige tekstbeskeder fra samme afsender i én samlet agenttur. Medier/vedhæftninger sendes med det samme. Kontrolkommandoer omgår debounce.

### TTS (tekst-til-tale)

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

- `auto` styrer automatisk TTS. `/tts off|always|inbound|tagged` tilsidesætter pr. session.
- `summaryModel` tilsidesætter `agents.defaults.model.primary` for automatisk opsummering.
- API-nøgler falder tilbage til `ELEVENLABS_API_KEY`/`XI_API_KEY` og `OPENAI_API_KEY`.

---

## Tal

Standardindstillinger for Talk-tilstand (macOS/iOS/Android).

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

- Voice-ID'er falder tilbage til `ELEVENLABS_VOICE_ID` eller `SAG_VOICE_ID`.
- `apiKey` falder tilbage til `ELEVENLABS_API_KEY`.
- `voiceAliases` gør det muligt for Talk-direktiver at bruge venlige navne.

---

## Værktøjer

### Værktøjsprofiler

`tools.profile` angiver en grundlæggende tilladelsesliste før `tools.allow`/`tools.deny`:

| Profil      | Inkluderer                                                                                |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | kun `session_status`                                                                      |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Ingen begrænsning (samme som ikke angivet)                             |

### Værktøjsgrupper

| Gruppe             | Værktøjer                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` accepteres som et alias for `exec`)         |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Alle indbyggede værktøjer (ekskl. provider-plugins)   |

### `tools.allow` / `tools.deny`

Global tillad/afvis-politik for værktøjer (afvis har forrang). Ikke-følsom for store/små bogstaver, understøtter `*`-wildcards. Anvendes selv når Docker-sandbox er slået fra.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Begræns yderligere værktøjer for specifikke providere eller modeller. Rækkefølge: basisprofil → providerprofil → tillad/afvis.

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

Kontrollerer forhøjet (host) exec-adgang:

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

- Per-agent tilsidesættelse (`agents.list[].tools.elevated`) kan kun begrænse yderligere.
- `/elevated on|off|ask|full` gemmer tilstand pr. session; inline-direktiver gælder kun for en enkelt besked.
- Forhøjet `exec` kører på værten og omgår sandboxing.

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

Konfigurerer forståelse af indgående medier (billede/lyd/video):

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

**Provider-post** (`type: "provider"` eller udeladt):

- `provider`: API-provider-id (`openai`, `anthropic`, `google`/`gemini`, `groq`, osv.)
- `model`: tilsidesættelse af model-id
- `profile` / `preferredProfile`: valg af auth-profil

**CLI-post** (`type: "cli"`):

- `command`: eksekverbar fil der skal køres
- `args`: skabelonbaserede argumenter (understøtter `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, osv.)

**Fælles felter:**

- `capabilities`: valgfri liste (`image`, `audio`, `video`). Standarder: `openai`/`anthropic`/`minimax` → billede, `google` → billede+lyd+video, `groq` → lyd.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: tilsidesættelser pr. post.
- Fejl falder tilbage til næste post.

Provider-auth følger standardrækkefølgen: auth-profiler → miljøvariabler → `models.providers.*.apiKey`.

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

- `model`: standardmodel for oprettede underagenter. Hvis udeladt, arver underagenter kalderens model.
- Værktøjspolitik pr. underagent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Brugerdefinerede providere og base-URL’er

OpenClaw bruger modelkataloget fra pi-coding-agent. Tilføj brugerdefinerede providere via `models.providers` i konfigurationen eller `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (standard) | replace
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

- Brug `authHeader: true` + `headers` til brugerdefinerede autentificeringsbehov.
- Tilsidesæt agentens konfigurationsrod med `OPENCLAW_AGENT_DIR` (eller `PI_CODING_AGENT_DIR`).

### Eksempler på providere

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

Brug `cerebras/zai-glm-4.7` til Cerebras; `zai/glm-4.7` til Z.AI direkte.

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

Sæt `OPENCODE_API_KEY` (eller `OPENCODE_ZEN_API_KEY`). Genvej: `openclaw onboard --auth-choice opencode-zen`.

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

Sæt `ZAI_API_KEY`. `z.ai/*` og `z-ai/*` accepteres som aliaser. Genvej: `openclaw onboard --auth-choice zai-api-key`.

- Generelt endpoint: `https://api.z.ai/api/paas/v4`
- Coding-endpoint (standard): `https://api.z.ai/api/coding/paas/v4`
- For det generelle endpoint skal du definere en brugerdefineret provider med tilsidesættelse af base-URL.

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

For Kina-endpointet: `baseUrl: "https://api.moonshot.cn/v1"` eller `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic-kompatibel, indbygget provider. Genvej: `openclaw onboard --auth-choice kimi-code-api-key`.

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

Base-URL må ikke indeholde `/v1` (Anthropic-klienten tilføjer den). Genvej: `openclaw onboard --auth-choice synthetic-api-key`.

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

Sæt `MINIMAX_API_KEY`. Genvej: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Se [Local Models](/gateway/local-models). Kort sagt: kør MiniMax M2.1 via LM Studio Responses API på kraftig hardware; behold hostede modeller flettet som fallback.

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

- `allowBundled`: valgfri allowlist kun for medfølgende skills (administrerede/workspace-skills påvirkes ikke).
- `entries.<skillKey>`.enabled: false\` deaktiverer en skill, selv hvis den er medfølgende/installeret.
- `entries.<skillKey>.apiKey`: bekvemmelighed for skills, der angiver en primær miljøvariabel.

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

- Indlæses fra `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` samt `plugins.load.paths`.
- **Ændringer i konfigurationen kræver en genstart af gatewayen.**
- `allow`: valgfri allowliste (kun angivne plugins indlæses). `deny` har forrang.

Se [Plugins](/tools/plugin).

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

- `evaluateEnabled: false` deaktiverer `act:evaluate` og `wait --fn`.
- Remote-profiler er kun til tilknytning (start/stop/reset deaktiveret).
- Rækkefølge for automatisk registrering: standardbrowser hvis Chromium-baseret → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Control-tjeneste: kun loopback (port afledt af `gateway.port`, standard `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, kort tekst, billed-URL eller data-URI
    },
  },
}
```

- `seamColor`: accentfarve for native app-UI (Talk Mode-boblefarve osv.).
- `assistant`: tilsidesættelse af Control UI-identitet. Falder tilbage til den aktive agent-identitet.

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
      // password: "your-password", // eller OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; se /gateway/trusted-proxy-auth
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
      // Yderligere /tools/invoke HTTP-deny
      deny: ["browser"],
      // Fjern værktøjer fra standard HTTP-denylisten
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (kør gateway) eller `remote` (forbind til remote gateway). Gateway nægter at starte, medmindre `local` er angivet.
- `port`: enkelt multiplekset port til WS + HTTP. Prioritet: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (standard), `lan` (`0.0.0.0`), `tailnet` (kun Tailscale-IP) eller `custom`.
- **Auth**: påkrævet som standard. Ikke-loopback bindinger kræver en delt token/adgangskode. Onboarding-guiden genererer som standard en token.
- `auth.mode: "trusted-proxy"`: delegér auth til en identitetsbevidst reverse proxy og stol på identitets-headere fra `gateway.trustedProxies` (se [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: når `true`, opfylder Tailscale Serve-identitetsheadere auth (verificeret via `tailscale whois`). Standard er `true`, når `tailscale.mode = "serve"`.
- `auth.rateLimit`: valgfri begrænsning af mislykkede auth-forsøg. Gælder pr. klient-IP og pr. auth-scope (shared-secret og device-token spores uafhængigt). Blokerede forsøg returnerer `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` er som standard `true`; sæt til `false`, når du bevidst ønsker, at localhost-trafik også skal rate-begrænses (til testopsætninger eller stramme proxy-implementeringer).
- `tailscale.mode`: `serve` (kun tailnet, loopback-binding) eller `funnel` (offentlig, kræver auth).
- `remote.transport`: `ssh` (standard) eller `direct` (ws/wss). For `direct` skal `remote.url` være `ws://` eller `wss://`.
- `gateway.remote.token` er kun til fjern-CLI-kald; aktiverer ikke lokal gateway-godkendelse.
- `trustedProxies`: reverse proxy-IP’er, der terminerer TLS. Angiv kun proxies, du kontrollerer.
- `gateway.tools.deny`: ekstra værktøjsnavne, der blokeres for HTTP `POST /tools/invoke` (udvider standard deny-liste).
- `gateway.tools.allow`: fjern værktøjsnavne fra standard HTTP deny-liste.

</Accordion>

### OpenAI-kompatible endpoints

- Chat Completions: deaktiveret som standard. Aktivér med `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Hærdning af URL-input i Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Isolering af flere instanser

Kør flere gateways på én host med unikke porte og state-mapper:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Bekvemmelighedsflag: `--dev` (bruger `~/.openclaw-dev` + port `19001`), `--profile <name>` (bruger `~/.openclaw-<name>`).

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
  - `sessionKey` fra request-payload accepteres kun når `hooks.allowRequestSessionKey=true` (standard: `false`).
- `POST /hooks/<name>` → løses via `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` matcher sub-path efter `/hooks` (f.eks. `/hooks/gmail` → `gmail`).
- `match.source` matcher et payload-felt for generiske paths.
- Skabeloner som `{{messages[0].subject}}` læses fra payloadet.
- `transform` kan pege på et JS/TS-modul, der returnerer en hook-handling.
  - `transform.module` skal være en relativ sti og forblive inden for `hooks.transformsDir` (absolutte stier og traversal afvises).
- `agentId` ruter til en specifik agent; ukendte ID’er falder tilbage til standard.
- `allowedAgentIds`: begrænser eksplicit routing (`*` eller udeladt = tillad alle, `[]` = afvis alle).
- `defaultSessionKey`: valgfri fast sessionsnøgle til hook-agentkørsler uden eksplicit `sessionKey`.
- `allowRequestSessionKey`: tillad at `/hooks/agent`-kaldere sætter `sessionKey` (standard: `false`).
- `allowedSessionKeyPrefixes`: valgfri prefix-allowlist for eksplicitte `sessionKey`-værdier (request + mapping), f.eks. `["hook:"]`.
- `deliver: true` sender det endelige svar til en kanal; `channel` er som standard `last`.
- `model` tilsidesætter LLM for denne hook-kørsel (skal være tilladt, hvis modelkatalog er sat).

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

- Gateway starter automatisk `gog gmail watch serve` ved opstart, når den er konfigureret. Sæt `OPENCLAW_SKIP_GMAIL_WATCHER=1` for at deaktivere.
- Kør ikke en separat `gog gmail watch serve` sammen med Gateway.

---

## Canvas-vært

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // eller OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Serverer agent-redigerbar HTML/CSS/JS og A2UI over HTTP under Gateway-porten:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Kun lokalt: behold `gateway.bind: "loopback"` (standard).
- Ikke-loopback bindings: canvas-ruter kræver Gateway-godkendelse (token/adgangskode/trusted-proxy), ligesom andre Gateway HTTP-flader.
- Node WebViews sender typisk ikke auth-headers; efter at en node er parret og forbundet, tillader Gateway en privat-IP fallback, så noden kan indlæse canvas/A2UI uden at lække hemmeligheder i URL'er.
- Indsætter live-reload-klient i serveret HTML.
- Opretter automatisk en starter-`index.html`, når mappen er tom.
- Serverer også A2UI på `/__openclaw__/a2ui/`.
- Ændringer kræver en genstart af gatewayen.
- Deaktivér live reload for store mapper eller ved `EMFILE`-fejl.

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

- `minimal` (standard): udelad `cliPath` + `sshPort` fra TXT-records.
- `full`: inkluder `cliPath` + `sshPort`.
- Værtsnavn er som standard `openclaw`. Tilsidesæt med `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Skriver en unicast DNS-SD-zone under `~/.openclaw/dns/`. For discovery på tværs af netværk, kombiner med en DNS-server (CoreDNS anbefales) + Tailscale split DNS.

Opsætning: `openclaw dns setup --apply`.

---

## Miljø

### `env` (inline miljøvariabler)

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

- Inline miljøvariabler anvendes kun, hvis procesmiljøet mangler nøglen.
- `.env`-filer: CWD `.env` + `~/.openclaw/.env` (ingen af dem tilsidesætter eksisterende variabler).
- `shellEnv`: importerer manglende forventede nøgler fra din login shell-profil.
- Se [Environment](/help/environment) for fuld prioritering.

### Substitution af miljøvariabler

Referér til miljøvariabler i enhver konfigurationsstreng med `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Kun navne med store bogstaver matches: `[A-Z_][A-Z0-9_]*`.
- Manglende/tomme variabler udløser en fejl ved indlæsning af konfigurationen.
- Escap med `$${VAR}` for en bogstavelig `${VAR}`.
- Fungerer med `$include`.

---

## Godkendelseslagring

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

- Pr.-agent godkendelsesprofiler gemmes i `<agentDir>/auth-profiles.json`.
- Ældre OAuth importeres fra `~/.openclaw/credentials/oauth.json`.
- Se [OAuth](/concepts/oauth).

---

## Logning

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

- Standard logfil: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Sæt `logging.file` for en fast sti.
- `consoleLevel` hæves til `debug` ved brug af `--verbose`.

---

## Guide

Metadata skrevet af CLI-guider (`onboard`, `configure`, `doctor`):

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

Skrevet af macOS onboarding-assistenten. Afleder standardværdier:

- `messages.ackReaction` fra `identity.emoji` (falder tilbage til 👀)
- `mentionPatterns` fra `identity.name`/`identity.emoji`
- `avatar` accepterer: arbejdsområderelativ sti, `http(s)` URL eller `data:` URI

---

## Bridge (forældet, fjernet)

Nuværende builds inkluderer ikke længere TCP bridge. Noder forbinder via Gateway WebSocket. `bridge.*`-nøgler er ikke længere en del af konfigurationsskemaet (validering fejler, indtil de fjernes; `openclaw doctor --fix` kan fjerne ukendte nøgler).

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

- `sessionRetention`: hvor længe afsluttede cron-sessioner gemmes før oprydning. Standard: `24h`.

Se [Cron Jobs](/automation/cron-jobs).

---

## Skabelonvariabler for mediemodel

Skabelonpladsholdere udvidet i `tools.media.*.models[].args`:

| Variabel           | Beskrivelse                                                                        |
| ------------------ | ---------------------------------------------------------------------------------- |
| `{{Body}}`         | Fuld indgående beskedtekst                                                         |
| `{{RawBody}}`      | Rå brødtekst (ingen historik-/afsender-wrappere)                |
| `{{BodyStripped}}` | Brødtekst med gruppementioner fjernet                                              |
| `{{From}}`         | Afsenderidentifikator                                                              |
| `{{To}}`           | Destinationsidentifikator                                                          |
| `{{MessageSid}}`   | Kanalens besked-id                                                                 |
| `{{SessionId}}`    | Nuværende sessions-UUID                                                            |
| `{{IsNewSession}}` | `"true"` når ny session oprettes                                                   |
| `{{MediaUrl}}`     | Pseudo-URL for indgående medie                                                     |
| `{{MediaPath}}`    | Lokal mediest i                                                                    |
| `{{MediaType}}`    | Medietype (image/audio/document/…)                              |
| `{{Transcript}}`   | Lydtransskription                                                                  |
| `{{Prompt}}`       | Løst medie-prompt til CLI-indgange                                                 |
| `{{MaxChars}}`     | Løst maks. antal outputtegn til CLI-indgange                       |
| `{{ChatType}}`     | `"direct"` eller `"group"`                                                         |
| `{{GroupSubject}}` | Gruppens emne (bedste estimat)                                  |
| `{{GroupMembers}}` | Forhåndsvisning af gruppemedlemmer (bedste estimat)             |
| `{{SenderName}}`   | Afsenderens viste navn (bedste estimat)                         |
| `{{SenderE164}}`   | Afsenderens telefonnummer (bedste estimat)                      |
| `{{Provider}}`     | Udbyder-hint (whatsapp, telegram, discord osv.) |

---

## Konfiguration inkluderer (`$include`)

Opdel konfiguration i flere filer:

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

**Fletteadfærd:**

- Enkelt fil: erstatter det indeholdte objekt.
- Array af filer: deep-merged i rækkefølge (senere overskriver tidligere).
- Sideløbende nøgler: flettes efter includes (overskriver inkluderede værdier).
- Indlejrede includes: op til 10 niveauer dybt.
- Stier: relative (til den inkluderende fil), absolutte eller `../` forældre-referencer.
- Fejl: tydelige beskeder for manglende filer, parse-fejl og cirkulære includes.

---

_Relateret: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
