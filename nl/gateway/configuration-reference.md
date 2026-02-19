---
title: "Configuratiereferentie"
description: "Volledige veld-voor-veld referentie voor ~/.openclaw/openclaw.json"
---

# Configuratiereferentie

Elk veld beschikbaar in `~/.openclaw/openclaw.json`. Voor een taakgerichte overzicht, zie [Configuration](/gateway/configuration).

Het configuratieformaat is **JSON5** (commentaar + afsluitende komma's toegestaan). Alle velden zijn optioneel — OpenClaw gebruikt veilige standaardwaarden wanneer ze zijn weggelaten.

---

## Kanalen

Elk kanaal start automatisch wanneer de configuratiesectie bestaat (tenzij `enabled: false`).

### DM- en groepstoegang

Alle kanalen ondersteunen DM-beleid en groepsbeleid:

| DM-beleid                                | Gedrag                                                                                |
| ---------------------------------------- | ------------------------------------------------------------------------------------- |
| `pairing` (standaard) | Onbekende afzenders krijgen een eenmalige koppelingscode; de eigenaar moet goedkeuren |
| `allowlist`                              | Alleen afzenders in `allowFrom` (of gekoppelde allow store)        |
| `open`                                   | Sta alle inkomende DM's toe (vereist `allowFrom: ["*"]`)           |
| `disabled`                               | Negeer alle inkomende DM's                                                            |

| Groepsbeleid                               | Gedrag                                                                             |
| ------------------------------------------ | ---------------------------------------------------------------------------------- |
| `allowlist` (standaard) | Alleen groepen die overeenkomen met de geconfigureerde allowlist                   |
| `open`                                     | Omzeil groeps-allowlists (mention-gating blijft van toepassing) |
| `disabled`                                 | Blokkeer alle groeps-/roomberichten                                                |

<Note>
`channels.defaults.groupPolicy` stelt de standaard in wanneer de `groupPolicy` van een provider niet is ingesteld.
Koppelcodes verlopen na 1 uur. Openstaande DM-koppelverzoeken zijn beperkt tot **3 per kanaal**.
Slack/Discord hebben een speciale fallback: als hun providersectie volledig ontbreekt, kan het runtime-groepsbeleid worden ingesteld op `open` (met een opstartwaarschuwing).
</Note>

### WhatsApp

WhatsApp draait via het webkanaal van de gateway (Baileys Web). Het start automatisch wanneer er een gekoppelde sessie bestaat.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // blauwe vinkjes (false in self-chat-modus)
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

- Uitgaande opdrachten gebruiken standaard het account `default` indien aanwezig; anders het eerste geconfigureerde account-id (gesorteerd).
- De verouderde single-account Baileys-authmap wordt door `openclaw doctor` gemigreerd naar `whatsapp/default`.
- Per-account overrides: `channels.whatsapp.accounts.<id> .sendReadReceipts`, `channels.whatsapp.accounts.<id> .dmPolicy`, `channels.whatsapp.accounts.<id> .allowFrom`.

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
          systemPrompt: "Houd antwoorden beknopt.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Blijf bij het onderwerp.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git-back-up" },
        { command: "generate", description: "Maak een afbeelding" },
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

- Bot-token: `channels.telegram.botToken` of `channels.telegram.tokenFile`, met `TELEGRAM_BOT_TOKEN` als fallback voor het standaardaccount.
- `configWrites: false` blokkeert door Telegram geïnitieerde config-schrijfacties (supergroup-ID-migraties, `/config set|unset`).
- Telegram stream-previews gebruiken `sendMessage` + `editMessageText` (werkt in directe en groepschats).
- Retrybeleid: zie [Retry policy](/concepts/retry).

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
              systemPrompt: "Alleen korte antwoorden.",
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

- Token: `channels.discord.token`, met `DISCORD_BOT_TOKEN` als fallback voor het standaardaccount.
- Gebruik `user:<id>` (DM) of `channel:<id>` (guildkanaal) als bezorgdoel; losse numerieke ID's worden geweigerd.
- Guild-slugs zijn lowercase waarbij spaties worden vervangen door `-`; kanaalsleutels gebruiken de geslugde naam (zonder `#`). Geef de voorkeur aan guild-ID's.
- Door bots geschreven berichten worden standaard genegeerd. `allowBots: true` schakelt ze in (eigen berichten worden nog steeds gefilterd).
- `maxLinesPerMessage` (standaard 17) splitst lange berichten, zelfs wanneer ze onder 2000 tekens blijven.
- `channels.discord.ui.components.accentColor` stelt de accentkleur in voor Discord components v2-containers.

**Modi voor reactiemeldingen:** `off` (geen), `own` (berichten van de bot, standaard), `all` (alle berichten), `allowlist` (van `guilds.<id>`.users\` op alle berichten).

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

- Serviceaccount-JSON: inline (`serviceAccount`) of op basis van een bestand (`serviceAccountFile`).
- Omgevingsvariabele-fallbacks: `GOOGLE_CHAT_SERVICE_ACCOUNT` of `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Gebruik `spaces/<spaceId>` of `users/<userId|email>` als afleverdoelen.

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

- **Socket-modus** vereist zowel `botToken` als `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` voor de standaard account-omgevingsfallback).
- **HTTP-modus** vereist `botToken` plus `signingSecret` (op root-niveau of per account).
- `configWrites: false` blokkeert door Slack geïnitieerde config-schrijfacties.
- Gebruik `user:<id>` (DM) of `channel:<id>` als afleverdoelen.

**Modi voor reactiemeldingen:** `off`, `own` (standaard), `all`, `allowlist` (van `reactionAllowlist`).

**Thread-sessie-isolatie:** `thread.historyScope` is per thread (standaard) of gedeeld binnen het kanaal. `thread.inheritParent` kopieert het bovenliggende kanaaltranscript naar nieuwe threads.

| Actiegroep | Standaard    | Notities                             |
| ---------- | ------------ | ------------------------------------ |
| reactions  | ingeschakeld | Reageren + reacties weergeven        |
| messages   | ingeschakeld | Lezen/verzenden/bewerken/verwijderen |
| pins       | ingeschakeld | Vastpinnen/losmaken/lijst weergeven  |
| memberInfo | ingeschakeld | Lidinformatie                        |
| emojiList  | ingeschakeld | Aangepaste emoji-lijst               |

### Mattermost

Mattermost wordt geleverd als een plugin: `openclaw plugins install @openclaw/mattermost`.

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

Chatmodi: `oncall` (reageer bij @-vermelding, standaard), `onmessage` (elk bericht), `onchar` (berichten die beginnen met een triggerprefix).

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

**Reactiemeldingsmodi:** `off`, `own` (standaard), `all`, `allowlist` (vanuit `reactionAllowlist`).

### iMessage

OpenClaw start `imsg rpc` (JSON-RPC via stdio). Geen daemon of poort vereist.

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

- Vereist Full Disk Access tot de Messages-database.
- Geef de voorkeur aan `chat_id:<id>`-doelen. Gebruik `imsg chats --limit 20` om chats weer te geven.
- `cliPath` kan verwijzen naar een SSH-wrapper; stel `remoteHost` in voor het ophalen van bijlagen via SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Multi-account (alle kanalen)

Voer meerdere accounts per kanaal uit (elk met een eigen `accountId`):

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

- `default` wordt gebruikt wanneer `accountId` is weggelaten (CLI + routing).
- Env-tokens zijn alleen van toepassing op het **default**-account.
- Basis-kanaalinstellingen zijn van toepassing op alle accounts, tenzij per account overschreven.
- Gebruik `bindings[].match.accountId` om elk account naar een andere agent te routeren.

### Mention-gating in groepschats

Groepsberichten vereisen standaard een **vermelding** (metadata-vermelding of regex-patronen). Van toepassing op WhatsApp-, Telegram-, Discord-, Google Chat- en iMessage-groepschats.

**Vermeldingstypen:**

- **Metadata-vermeldingen**: Native platform-@-vermeldingen. Genegeerd in WhatsApp self-chat-modus.
- **Tekstpatronen**: Regex-patronen in `agents.list[].groupChat.mentionPatterns`. Altijd gecontroleerd.
- Mention-gating wordt alleen afgedwongen wanneer detectie mogelijk is (native vermeldingen of ten minste één patroon).

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

`messages.groupChat.historyLimit` stelt de globale standaard in. Kanalen kunnen overschrijven met `channels.<channel>`.historyLimit`(of per account). Stel`0\` in om uit te schakelen.

#### DM-geschiedenislijnen

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

Resolutie: per-DM-override → providerstandaard → geen limiet (alles behouden).

Ondersteund: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Zelf-chatmodus

Voeg je eigen nummer toe aan `allowFrom` om de zelf-chatmodus in te schakelen (negeert native @-vermeldingen, reageert alleen op tekstpatronen):

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

### Commando’s (afhandeling van chatcommando’s)

```json5
{
  commands: {
    native: "auto", // registreer native commando’s indien ondersteund
    text: true, // parse /commando’s in chatberichten
    bash: false, // sta ! toe (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // sta /config toe
    debug: false, // sta /debug toe
    restart: false, // sta /restart + gateway restart-tool toe
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Tekstcommando’s moeten **losstaande** berichten zijn die beginnen met `/`.
- `native: "auto"` schakelt native commando’s in voor Discord/Telegram en laat Slack uit.
- Overschrijf per kanaal: `channels.discord.commands.native` (bool of `"auto"`). `false` wist eerder geregistreerde commando’s.
- `channels.telegram.customCommands` voegt extra Telegram-botmenu-items toe.
- `bash: true` schakelt `!` in <cmd>`voor de hostshell. Vereist`tools.elevated.enabled`en dat de afzender in`tools.elevated.allowFrom.<channel>` staat`.
- `config: true` schakelt `/config` in (leest/schrijft `openclaw.json`).
- `channels.<provider>``.configWrites` regelt configuratiewijzigingen per kanaal (standaard: true).
- `allowFrom` is per provider. Wanneer ingesteld, is dit de **enige** autorisatiebron (kanaal-allowlists/pairing en `useAccessGroups` worden genegeerd).
- `useAccessGroups: false` laat commando’s toegangs­groepbeleid omzeilen wanneer `allowFrom` niet is ingesteld.

</Accordion>

---

## Agent-standaardinstellingen

### `agents.defaults.workspace`

Standaard: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Optionele repository-root die wordt weergegeven in de Runtime-regel van de systeemprompt. Indien niet ingesteld, detecteert OpenClaw deze automatisch door vanaf de workspace omhoog te lopen.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Schakelt het automatisch aanmaken van workspace-bootstrapbestanden uit (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maximaal aantal tekens per workspace-bootstrapbestand vóór afkappen. Standaard: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Maximaal totaal aantal ingevoegde tekens over alle workspace-bootstrapbestanden. Standaard: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Tijdzone voor systeem-promptcontext (niet voor berichttijdstempels). Valt terug op de tijdzone van de host.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Tijdnotatie in de systeem-prompt. Standaard: `auto` (OS-voorkeur).

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

- `model.primary`: indeling `provider/model` (bijv. `anthropic/claude-opus-4-6`). Als je de provider weglaat, gaat OpenClaw uit van `anthropic` (verouderd).
- `models`: de geconfigureerde modelcatalogus en allowlist voor `/model`. Elke invoer kan `alias` (snelkoppeling) en `params` (provider-specifiek: `temperature`, `maxTokens`) bevatten.
- `imageModel`: wordt alleen gebruikt als het primaire model geen afbeeldingsinvoer ondersteunt.
- `maxConcurrent`: maximaal aantal parallelle agent-uitvoeringen over sessies heen (elke sessie blijft geserialiseerd). Standaard: 1.

**Ingebouwde alias-snelkoppelingen** (alleen van toepassing wanneer het model in `agents.defaults.models` staat):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Je geconfigureerde aliassen hebben altijd voorrang op de standaarden.

Z.AI GLM-4.x-modellen schakelen automatisch de denkmodus in, tenzij je `--thinking off` instelt of zelf `agents.defaults.models["zai/<model>"].params.thinking` definieert.

### `agents.defaults.cliBackends`

Optionele CLI-backends voor tekst-only fallback-uitvoeringen (geen tool-aanroepen). Handig als back-up wanneer API-providers uitvallen.

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

- CLI-backends zijn primair tekstgericht; tools zijn altijd uitgeschakeld.
- Sessies worden ondersteund wanneer `sessionArg` is ingesteld.
- Doorgifte van afbeeldingen wordt ondersteund wanneer `imageArg` bestandspaden accepteert.

### `agents.defaults.heartbeat`

Periodieke heartbeat-uitvoeringen.

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

- `every`: duurstring (ms/s/m/h). Standaard: `30m`.
- Per agent: stel `agents.list[].heartbeat` in. Wanneer een agent `heartbeat` definieert, voeren **alleen die agents** heartbeats uit.
- Heartbeats draaien volledige agent-beurten — kortere intervallen verbruiken meer tokens.

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

- `mode`: `default` of `safeguard` (gesegmenteerde samenvatting voor lange geschiedenissen). Zie [Compaction](/concepts/compaction).
- `memoryFlush`: stille agentische beurt vóór automatische compaction om duurzame herinneringen op te slaan. Wordt overgeslagen wanneer de workspace alleen-lezen is.

### `agents.defaults.contextPruning`

Snoeit **oude toolresultaten** uit de in-memory context voordat deze naar de LLM wordt verzonden. Wijzigt de sessiegeschiedenis op schijf **niet**.

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

- `mode: "cache-ttl"` schakelt snoeibeurten in.
- `ttl` bepaalt hoe vaak snoeien opnieuw kan worden uitgevoerd (na de laatste cache-touch).
- Snoeien verkort eerst te grote toolresultaten (soft-trim) en wist daarna indien nodig oudere toolresultaten volledig (hard-clear).

**Soft-trim** behoudt het begin + einde en voegt `...` in het midden in.

**Hard-clear** vervangt het volledige toolresultaat door de placeholder.

Opmerkingen:

- Afbeeldingsblokken worden nooit verkort of gewist.
- Ratio’s zijn gebaseerd op tekens (benadering), geen exacte tokenaantallen.
- Als er minder dan `keepLastAssistants` assistant-berichten zijn, wordt snoeien overgeslagen.

</Accordion>

Zie [Session Pruning](/concepts/session-pruning) voor details over het gedrag.

### Blokstreaming

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

- Niet-Telegram-kanalen vereisen expliciet `*.blockStreaming: true` om blokantwoorden in te schakelen.
- Kanaaloverschrijvingen: `channels.<channel> .blockStreamingCoalesce` (en per-accountvarianten). Signal/Slack/Discord/Google Chat hebben standaard `minChars: 1500`.
- `humanDelay`: willekeurige pauze tussen blokantwoorden. `natural` = 800–2500ms. Per agent overschrijven: `agents.list[].humanDelay`.

Zie [Streaming](/concepts/streaming) voor details over gedrag en chunking.

### Typindicatoren

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

- Standaardinstellingen: `instant` voor directe chats/vermeldingen, `message` voor groepschats zonder vermelding.
- Per-sessie-overschrijvingen: `session.typingMode`, `session.typingIntervalSeconds`.

Zie [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Optionele **Docker-sandboxing** voor de ingebedde agent. Zie [Sandboxing](/gateway/sandboxing) voor de volledige handleiding.

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

**Werkruimtetoegang:**

- `none`: per-scope sandbox-werkruimte onder `~/.openclaw/sandboxes`
- `ro`: sandbox-werkruimte op `/workspace`, agent-werkruimte alleen-lezen gemount op `/agent`
- `rw`: agent-werkruimte lezen/schrijven gemount op `/workspace`

**Scope:**

- `session`: per-sessie container + werkruimte
- `agent`: één container + werkruimte per agent (standaard)
- `shared`: gedeelde container en werkruimte (geen isolatie tussen sessies)

**`setupCommand`** wordt één keer uitgevoerd na het aanmaken van de container (via `sh -lc`). Vereist netwerk-egress, beschrijfbare root en root-gebruiker.

**Containers gebruiken standaard `network: "none"`** — stel in op `"bridge"` als de agent uitgaande toegang nodig heeft.

**Inkomende bijlagen** worden geplaatst in `media/inbound/*` in de actieve werkruimte.

**`docker.binds`** mount extra hostmappen; globale en per-agent binds worden samengevoegd.

**Gesandboxte browser** (`sandbox.browser.enabled`): Chromium + CDP in een container. noVNC-URL wordt in de systeemprompt ingevoegd. Vereist geen `browser.enabled` in de hoofdconfiguratie.

- `allowHostControl: false` (standaard) blokkeert gesandboxte sessies om de hostbrowser aan te sturen.
- `sandbox.browser.binds` mount extra hostmappen uitsluitend in de sandbox-browsercontainer. Wanneer ingesteld (inclusief `[]`), vervangt dit `docker.binds` voor de browsercontainer.

</Accordion>

Bouw images:

```bash
scripts/sandbox-setup.sh           # hoofd-sandboximage
scripts/sandbox-browser-setup.sh   # optioneel browserimage
```

### `agents.list` (per-agent overschrijvingen)

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
        model: "anthropic/claude-opus-4-6", // of { primary, fallbacks }
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

- `id`: stabiele agent-id (verplicht).
- `default`: wanneer meerdere zijn ingesteld, wint de eerste (waarschuwing wordt gelogd). Als er geen is ingesteld, is de eerste vermelding in de lijst de standaard.
- `model`: stringvorm overschrijft alleen `primary`; objectvorm `{ primary, fallbacks }` overschrijft beide (`[]` schakelt globale fallbacks uit).
- `identity.avatar`: werkruimte-relatief pad, `http(s)`-URL of `data:`-URI.
- `identity` leidt standaardwaarden af: `ackReaction` van `emoji`, `mentionPatterns` van `name`/`emoji`.
- `subagents.allowAgents`: allowlist van agent-ids voor `sessions_spawn` (`["*"]` = elke; standaard: alleen dezelfde agent).

---

## Multi-agent routing

Voer meerdere geïsoleerde agents uit binnen één Gateway. Zie [Multi-Agent](/concepts/multi-agent).

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

### Bindende matchvelden

- `match.channel` (verplicht)
- `match.accountId` (optioneel; `*` = elk account; weggelaten = standaardaccount)
- `match.peer` (optioneel; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (optioneel; kanaalspecifiek)

**Deterministische matchvolgorde:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, geen peer/guild/team)
5. `match.accountId: "*"` (kanaalbreed)
6. Standaardagent

Binnen elk niveau wint de eerste overeenkomende `bindings`-vermelding.

### Toegangsprofielen per agent

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

Zie [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) voor details over prioriteit.

---

## Sessie

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

- **`dmScope`**: hoe DM's worden gegroepeerd.
  - `main`: alle DM's delen de hoofdsessie.
  - `per-peer`: isoleren per afzender-id over kanalen heen.
  - `per-channel-peer`: isoleren per kanaal + afzender (aanbevolen voor inboxen met meerdere gebruikers).
  - `per-account-channel-peer`: isoleren per account + kanaal + afzender (aanbevolen voor meerdere accounts).
- **`identityLinks`**: koppel canonieke id's aan provider-voorvoegsel peers voor het delen van sessies over kanalen heen.
- **`reset`**: primair resetbeleid. `daily` reset op `atHour` lokale tijd; `idle` reset na `idleMinutes`. Wanneer beide zijn geconfigureerd, geldt degene die het eerst verloopt.
- **`resetByType`**: overschrijvingen per type (`direct`, `group`, `thread`). Legacy `dm` wordt geaccepteerd als alias voor `direct`.
- **`mainKey`**: legacy veld. De runtime gebruikt nu altijd `"main"` voor de hoofd-bucket van directe chats.
- **`sendPolicy`**: match op `channel`, `chatType` (`direct|group|channel`, met legacy `dm` alias), `keyPrefix` of `rawKeyPrefix`. De eerste weigering (`deny`) wint.
- **`maintenance`**: `warn` waarschuwt de actieve sessie bij verwijdering; `enforce` past opschoning en rotatie toe.

</Accordion>

---

## Berichten

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

### Antwoordvoorvoegsel

Per-kanaal/account-overschrijvingen: `channels.<channel>`.responsePrefix`, `channels.<channel>`.accounts.<id>`.responsePrefix\`.

Resolutie (meest specifieke wint): account → kanaal → globaal. `""` schakelt uit en stopt de cascade. `"auto"` leidt af als `[{identity.name}]`.

**Sjabloonvariabelen:**

| Variabele         | Beschrijving                | Voorbeeld                                   |
| ----------------- | --------------------------- | ------------------------------------------- |
| `{model}`         | Korte modelnaam             | `claude-opus-4-6`                           |
| `{modelFull}`     | Volledige modelidentifier   | `anthropic/claude-opus-4-6`                 |
| `{provider}`      | Providernaam                | `anthropic`                                 |
| `{thinkingLevel}` | Huidig denkniveau           | `high`, `low`, `off`                        |
| `{identity.name}` | Naam van de agentidentiteit | (hetzelfde als `"auto"`) |

Variabelen zijn niet hoofdlettergevoelig. `{think}` is een alias voor `{thinkingLevel}`.

### Ack-reactie

- Standaard de `identity.emoji` van de actieve agent, anders `"👀"`. Stel `""` in om uit te schakelen.
- Per-kanaal-overschrijvingen: `channels.<channel>`.ackReaction`, `channels.<channel>`.accounts.<id>`.ackReaction\`.
- Resolutievolgorde: account → kanaal → `messages.ackReaction` → identity fallback.
- Bereik: `group-mentions` (standaard), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: verwijdert de ack na het antwoord (alleen Slack/Discord/Telegram/Google Chat).

### Inbound debounce

Bundelt snel opeenvolgende tekstberichten van dezelfde afzender tot één enkele agentbeurt. Media/bijlagen worden onmiddellijk doorgestuurd. Besturingsopdrachten omzeilen debouncing.

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

- `auto` regelt auto-TTS. `/tts off|always|inbound|tagged` overschrijft dit per sessie.
- `summaryModel` overschrijft `agents.defaults.model.primary` voor automatische samenvatting.
- API-sleutels vallen terug op `ELEVENLABS_API_KEY`/`XI_API_KEY` en `OPENAI_API_KEY`.

---

## Praten

Standaardinstellingen voor de Praten-modus (macOS/iOS/Android).

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

- Voice-ID's vallen terug op `ELEVENLABS_VOICE_ID` of `SAG_VOICE_ID`.
- `apiKey` valt terug op `ELEVENLABS_API_KEY`.
- `voiceAliases` zorgt ervoor dat Talk-directieven gebruik kunnen maken van vriendelijke namen.

---

## Tools

### Toolprofielen

`tools.profile` stelt een basis-allowlist in vóór `tools.allow`/`tools.deny`:

| Profiel     | Bevat                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | alleen `session_status`                                                                   |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Geen beperking (hetzelfde als niet ingesteld)                          |

### Toolgroepen

| Groep              | Tools                                                                                    |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` wordt geaccepteerd als alias voor `exec`)   |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Alle ingebouwde tools (exclusief provider-plugins)                    |

### `tools.allow` / `tools.deny`

Globaal tool-toestaan/weigeren-beleid (weigeren heeft voorrang). Hoofdletterongevoelig, ondersteunt `*`-wildcards. Wordt toegepast, zelfs wanneer de Docker-sandbox is uitgeschakeld.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Beperkt tools verder voor specifieke providers of modellen. Volgorde: basisprofiel → providerprofiel → allow/deny.

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

Beheert verhoogde (host) exec-toegang:

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

- Per-agent override (`agents.list[].tools.elevated`) kan alleen verder beperken.
- `/elevated on|off|ask|full` slaat de status per sessie op; inline-directieven gelden voor één enkel bericht.
- Verhoogde `exec` wordt uitgevoerd op de host en omzeilt sandboxing.

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
        apiKey: "brave_api_key", // of BRAVE_API_KEY env
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

Configureert inkomend mediabegrip (afbeelding/audio/video):

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

**Provider entry** (`type: "provider"` of weggelaten):

- `provider`: API-provider-id (`openai`, `anthropic`, `google`/`gemini`, `groq`, enz.)
- `model`: model-id-override
- `profile` / `preferredProfile`: selectie van authenticatieprofiel

**CLI entry** (`type: "cli"`):

- `command`: uitvoerbaar bestand om uit te voeren
- `args`: getemplateerde argumenten (ondersteunt `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, enz.)

**Gemeenschappelijke velden:**

- `capabilities`: optionele lijst (`image`, `audio`, `video`). Standaardwaarden: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: overschrijvingen per item.
- Fouten vallen terug op het volgende item.

Provider-authenticatie volgt de standaardvolgorde: auth-profielen → omgevingsvariabelen → `models.providers.*.apiKey`.

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

- `model`: standaardmodel voor aangemaakte subagents. Indien weggelaten erven subagents het model van de aanroeper.
- Toolbeleid per subagent: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Aangepaste providers en basis-URL's

OpenClaw gebruikt de pi-coding-agent modelcatalogus. Voeg aangepaste providers toe via `models.providers` in de configuratie of `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (standaard) | replace
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

- Gebruik `authHeader: true` + `headers` voor aangepaste authenticatiebehoeften.
- Overschrijf de root van de agentconfiguratie met `OPENCLAW_AGENT_DIR` (of `PI_CODING_AGENT_DIR`).

### Voorbeelden van providers

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

Gebruik `cerebras/zai-glm-4.7` voor Cerebras; `zai/glm-4.7` voor Z.AI direct.

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

Stel `OPENCODE_API_KEY` (of `OPENCODE_ZEN_API_KEY`) in. Snelkoppeling: `openclaw onboard --auth-choice opencode-zen`.

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

Stel `ZAI_API_KEY` in. `z.ai/*` en `z-ai/*` zijn geaccepteerde aliassen. Snelkoppeling: `openclaw onboard --auth-choice zai-api-key`.

- Algemeen endpoint: `https://api.z.ai/api/paas/v4`
- Coding-endpoint (standaard): `https://api.z.ai/api/coding/paas/v4`
- Voor het algemene endpoint definieer je een aangepaste provider met een overschreven basis-URL.

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

Voor het China-endpoint: `baseUrl: "https://api.moonshot.cn/v1"` of `openclaw onboard --auth-choice moonshot-api-key-cn`.

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

Anthropic-compatibel, ingebouwde provider. Snelkoppeling: `openclaw onboard --auth-choice kimi-code-api-key`.

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

De basis-URL mag `/v1` niet bevatten (de Anthropic-client voegt dit toe). Snelkoppeling: `openclaw onboard --auth-choice synthetic-api-key`.

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

Stel `MINIMAX_API_KEY` in. Snelkoppeling: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

Zie [Local Models](/gateway/local-models). TL;DR: draai MiniMax M2.1 via de LM Studio Responses API op serieuze hardware; houd gehoste modellen samengevoegd als fallback.

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

- `allowBundled`: optionele allowlist alleen voor gebundelde skills (managed/workspace skills niet beïnvloed).
- `entries.<skillKey>`.enabled: false\` schakelt een skill uit, zelfs als deze gebundeld/geïnstalleerd is.
- `entries.<skillKey>`.apiKey\`: gemakoptie voor skills die een primaire env-var declareren.

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

- Geladen vanuit `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus `plugins.load.paths`.
- **Configuratiewijzigingen vereisen een herstart van de gateway.**
- `allow`: optionele allowlist (alleen vermelde plugins worden geladen). `deny` heeft voorrang.

Zie [Plugins](/tools/plugin).

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

- `evaluateEnabled: false` schakelt `act:evaluate` en `wait --fn` uit.
- Remote-profielen zijn alleen-attach (start/stop/reset uitgeschakeld).
- Auto-detectievolgorde: standaardbrowser indien Chromium-gebaseerd → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Control-service: alleen loopback (poort afgeleid van `gateway.port`, standaard `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, korte tekst, afbeeldings-URL of data-URI
    },
  },
}
```

- `seamColor`: accentkleur voor de native app-UI (Talk Mode-bubbelkleur, enz.).
- `assistant`: overschrijving van de identiteit in de Control UI. Valt terug op de identiteit van de actieve agent.

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
      // password: "your-password", // of OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // voor mode=trusted-proxy; zie /gateway/trusted-proxy-auth
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
      // Aanvullende /tools/invoke HTTP-weigeringen
      deny: ["browser"],
      // Verwijder tools uit de standaard HTTP-weigerlijst
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (gateway uitvoeren) of `remote` (verbinden met externe gateway). Gateway weigert te starten tenzij `local`.
- `port`: enkele gemultiplexte poort voor WS + HTTP. Voorrang: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (standaard), `lan` (`0.0.0.0`), `tailnet` (alleen Tailscale-IP) of `custom`.
- **Auth**: standaard vereist. Non-loopback binds vereisen een gedeelde token/wachtwoord. De onboarding-wizard genereert standaard een token.
- `auth.mode: "trusted-proxy"`: delegeer auth aan een identity-aware reverse proxy en vertrouw identiteitsheaders van `gateway.trustedProxies` (zie [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: wanneer `true`, voldoen Tailscale Serve-identiteitsheaders aan de authenticatie (geverifieerd via `tailscale whois`). Standaard `true` wanneer `tailscale.mode = "serve"`.
- `auth.rateLimit`: optionele limiter voor mislukte authenticatie. Wordt toegepast per client-IP en per auth-scope (shared-secret en device-token worden onafhankelijk bijgehouden). Geblokkeerde pogingen geven `429` + `Retry-After` terug.
  - `auth.rateLimit.exemptLoopback` is standaard `true`; stel in op `false` wanneer je localhost-verkeer ook bewust wilt beperken (voor testopstellingen of strikte proxy-implementaties).
- `tailscale.mode`: `serve` (alleen tailnet, loopback-bind) of `funnel` (publiek, vereist authenticatie).
- `remote.transport`: `ssh` (standaard) of `direct` (ws/wss). Voor `direct` moet `remote.url` `ws://` of `wss://` zijn.
- `gateway.remote.token` is alleen voor externe CLI-aanroepen; schakelt geen lokale gateway-authenticatie in.
- `trustedProxies`: reverse proxy-IP’s die TLS beëindigen. Vermeld alleen proxies die je zelf beheert.
- `gateway.tools.deny`: extra toolnamen geblokkeerd voor HTTP `POST /tools/invoke` (breidt de standaard deny-lijst uit).
- `gateway.tools.allow`: verwijder toolnamen uit de standaard HTTP deny-lijst.

</Accordion>

### OpenAI-compatibele endpoints

- Chat Completions: standaard uitgeschakeld. Inschakelen met `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Responses URL-invoerhardening:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Isolatie van meerdere instanties

Draai meerdere gateways op één host met unieke poorten en state-mappen:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Handige flags: `--dev` (gebruikt `~/.openclaw-dev` + poort `19001`), `--profile <name>` (gebruikt `~/.openclaw-<name>`).

Zie [Multiple Gateways](/gateway/multiple-gateways).

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

Auth: `Authorization: Bearer <token>` of `x-openclaw-token: <token>`.

**Endpoints:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` uit de request-payload wordt alleen geaccepteerd wanneer `hooks.allowRequestSessionKey=true` (standaard: `false`).
- `POST /hooks/<name>` → opgelost via `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` komt overeen met het subpad na `/hooks` (bijv. `/hooks/gmail` → `gmail`).
- `match.source` komt overeen met een payloadveld voor generieke paden.
- Templates zoals `{{messages[0].subject}}` lezen uit de payload.
- `transform` kan verwijzen naar een JS/TS-module die een hook-actie retourneert.
  - `transform.module` moet een relatief pad zijn en binnen `hooks.transformsDir` blijven (absolute paden en padnavigatie worden geweigerd).
- `agentId` routeert naar een specifieke agent; onbekende ID's vallen terug op de standaardinstelling.
- `allowedAgentIds`: beperkt expliciete routering (`*` of weggelaten = alles toestaan, `[]` = alles weigeren).
- `defaultSessionKey`: optionele vaste sessiesleutel voor hook-agentruns zonder expliciete `sessionKey`.
- `allowRequestSessionKey`: sta toe dat `/hooks/agent`-aanroepers `sessionKey` instellen (standaard: `false`).
- `allowedSessionKeyPrefixes`: optionele prefix-allowlist voor expliciete `sessionKey`-waarden (request + mapping), bijv. `["hook:"]`.
- `deliver: true` verstuurt het definitieve antwoord naar een kanaal; `channel` is standaard `last`.
- `model` overschrijft de LLM voor deze hook-run (moet toegestaan zijn als de modelcatalogus is ingesteld).

</Accordion>

### Gmail-integratie

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

- Gateway start automatisch `gog gmail watch serve` bij het opstarten wanneer geconfigureerd. Stel `OPENCLAW_SKIP_GMAIL_WATCHER=1` in om uit te schakelen.
- Voer geen aparte `gog gmail watch serve` uit naast de Gateway.

---

## Canvas-host

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // of OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Levert door de agent bewerkbare HTML/CSS/JS en A2UI via HTTP onder de Gateway-poort:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Alleen lokaal: houd `gateway.bind: "loopback"` (standaard).
- Niet-loopback bindings: canvas-routes vereisen Gateway-authenticatie (token/wachtwoord/trusted-proxy), hetzelfde als andere Gateway-HTTP-oppervlakken.
- Node WebViews sturen doorgaans geen auth-headers mee; nadat een node is gekoppeld en verbonden, staat de Gateway een private-IP fallback toe zodat de node canvas/A2UI kan laden zonder geheimen in URL's te lekken.
- Voegt een live-reloadclient toe aan de geleverde HTML.
- Maakt automatisch een start-`index.html` aan wanneer leeg.
- Levert ook A2UI op `/__openclaw__/a2ui/`.
- Wijzigingen vereisen een herstart van de Gateway.
- Schakel live reload uit voor grote mappen of bij `EMFILE`-fouten.

---

## Detectie

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

- `minimal` (standaard): laat `cliPath` + `sshPort` weg uit TXT-records.
- `full`: voeg `cliPath` + `sshPort` toe.
- Hostnaam is standaard `openclaw`. Overschrijf met `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Schrijft een unicast DNS-SD-zone onder `~/.openclaw/dns/`. Voor detectie over meerdere netwerken, combineer met een DNS-server (CoreDNS aanbevolen) + Tailscale split DNS.

Installatie: `openclaw dns setup --apply`.

---

## Omgeving

### `env` (inline omgevingsvariabelen)

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

- Inline env-vars worden alleen toegepast als de procesomgeving de sleutel mist.
- `.env`-bestanden: CWD `.env` + `~/.openclaw/.env` (geen van beide overschrijft bestaande variabelen).
- `shellEnv`: importeert ontbrekende verwachte sleutels uit het profiel van je login-shell.
- Zie [Environment](/help/environment) voor de volledige prioriteitsvolgorde.

### Env-var-substitutie

Verwijs naar env-vars in elke config-string met `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Alleen hoofdletter-namen worden gematcht: `[A-Z_][A-Z0-9_]*`.
- Ontbrekende/lege variabelen veroorzaken een fout bij het laden van de config.
- Escapen met `$${VAR}` voor een letterlijke `${VAR}`.
- Werkt met `$include`.

---

## Auth-opslag

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

- Auth-profielen per agent opgeslagen in `<agentDir>/auth-profiles.json`.
- Legacy OAuth wordt geïmporteerd uit `~/.openclaw/credentials/oauth.json`.
- Zie [OAuth](/concepts/oauth).

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

- Standaard logbestand: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Stel `logging.file` in voor een stabiel pad.
- `consoleLevel` wordt verhoogd naar `debug` bij `--verbose`.

---

## Wizard

Metadata geschreven door CLI-wizards (`onboard`, `configure`, `doctor`):

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

## Identiteit

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

Geschreven door de macOS-onboardingassistent. Leidt standaardwaarden af:

- `messages.ackReaction` van `identity.emoji` (valt terug op 👀)
- `mentionPatterns` van `identity.name`/`identity.emoji`
- `avatar` accepteert: werkruimte-relatief pad, `http(s)`-URL of `data:`-URI

---

## Bridge (legacy, verwijderd)

Huidige builds bevatten de TCP-bridge niet langer. Nodes verbinden via de Gateway WebSocket. `bridge.*`-sleutels maken geen deel meer uit van het config-schema (validatie mislukt totdat ze zijn verwijderd; `openclaw doctor --fix` kan onbekende sleutels verwijderen).

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

- `sessionRetention`: hoe lang voltooide cron-sessies worden bewaard voordat ze worden opgeschoond. Standaard: `24h`.

Zie [Cron Jobs](/automation/cron-jobs).

---

## Media-modelsjabloonvariabelen

Sjabloonplaatsaanduidingen uitgebreid in `tools.media.*.models[].args`:

| Variabele          | Beschrijving                                                                        |
| ------------------ | ----------------------------------------------------------------------------------- |
| `{{Body}}`         | Volledige inkomende berichttekst                                                    |
| `{{RawBody}}`      | Ruwe berichttekst (zonder geschiedenis-/afzender-wrappers)       |
| `{{BodyStripped}}` | Berichttekst zonder groepsvermeldingen                                              |
| `{{From}}`         | Afzender-ID                                                                         |
| `{{To}}`           | Bestemmings-ID                                                                      |
| `{{MessageSid}}`   | Bericht-ID van het kanaal                                                           |
| `{{SessionId}}`    | Huidige sessie-UUID                                                                 |
| `{{IsNewSession}}` | `"true"` wanneer een nieuwe sessie is aangemaakt                                    |
| `{{MediaUrl}}`     | Pseudo-URL van inkomende media                                                      |
| `{{MediaPath}}`    | Lokaal mediapad                                                                     |
| `{{MediaType}}`    | Mediatype (image/audio/document/…)                               |
| `{{Transcript}}`   | Audiotranscript                                                                     |
| `{{Prompt}}`       | Opgeloste mediaprompt voor CLI-items                                                |
| `{{MaxChars}}`     | Opgelost maximaal aantal uitvoertekens voor CLI-items                               |
| `{{ChatType}}`     | `"direct"` of `"group"`                                                             |
| `{{GroupSubject}}` | Groepsonderwerp (voor zover mogelijk)                            |
| `{{GroupMembers}}` | Voorbeeld van groepsleden (voor zover mogelijk)                  |
| `{{SenderName}}`   | Weergavenaam van afzender (zo goed mogelijk bepaald)             |
| `{{SenderE164}}`   | Telefoonnummer van afzender (zo goed mogelijk bepaald)           |
| `{{Provider}}`     | Providerhint (whatsapp, telegram, discord, enz.) |

---

## Config bevat (`$include`)

Splits config op in meerdere bestanden:

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

**Samenvoeggedrag:**

- Enkel bestand: vervangt het bovenliggende object.
- Array van bestanden: deep-merged in volgorde (latere overschrijven eerdere).
- Sleutelwaarden op hetzelfde niveau: samengevoegd na includes (overschrijven inbegrepen waarden).
- Geneste includes: tot 10 niveaus diep.
- Paden: relatief (ten opzichte van het includende bestand), absoluut of `../`-verwijzingen naar bovenliggende mappen.
- Fouten: duidelijke meldingen voor ontbrekende bestanden, parsefouten en circulaire includes.

---

_Gerelateerd: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

