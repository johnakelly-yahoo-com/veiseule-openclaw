---
title: "Konfiguration"
---

# Konfiguration 🔧

OpenClaw læser en valgfri **JSON5**-konfiguration fra `~/.openclaw/openclaw.json` (kommentarer + afsluttende kommaer er tilladt).

Hvis filen mangler, bruger OpenClaw sikker standard (indlejret Pi-agent + per-afsendersessioner + arbejdsområde `~/.openclaw/workspace`). Du har normalt kun brug for en config til:

- begrænse hvem der kan trigge botten (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, osv.)
- styre gruppetilladelseslister + mention-adfærd (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- tilpasse beskedpræfikser (`messages`)
- sætte agentens workspace (`agents.defaults.workspace` eller `agents.list[].workspace`)
- finjustere standardindstillingerne for den indlejrede agent (`agents.defaults`) og session-adfærd (`session`)
- sætte identitet pr. agent (`agents.list[].identity`)

> **Ny i konfiguration?** Tjek guiden [Configuration Examples](/gateway/configuration-examples) for komplette eksempler med detaljerede forklaringer!

## Streng konfigurationsvalidering

OpenClaw accepterer kun konfigurationer, der fuldt ud matcher ordningen.
Ukendte nøgler, misdannede typer eller ugyldige værdier gør, at porten **nægter at starte** af sikkerhedshensyn.

Når valideringen fejler:

- Gateway starter ikke.
- Kun diagnostiske kommandoer er tilladt (for eksempel: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Kør `openclaw doctor` for at se de præcise problemer.
- Kør `openclaw doctor --fix` (eller `--yes`) for at anvende migreringer/reparationer.

Doctor skriver aldrig ændringer, medmindre du eksplicit tilvælger `--fix`/`--yes`.

## Skema + UI-hints

Gateway udsætter en JSON Schema repræsentation af config via `config.schema` for UI redaktører.
Den Control UI gør en form fra dette skema, med en \*\* Raw JSON\*\* editor som en undslippe luge.

Kanal-plugins og udvidelser kan registrere skema + UI-hints for deres konfiguration, så kanalindstillinger
forbliver skemadrevne på tværs af apps uden hardcodede formularer.

Hints (labels, gruppering, følsomme felter) leveres sammen med skemaet, så klienter kan rendere
bedre formularer uden at hardcode viden om konfigurationen.

## Anvend + genstart (RPC)

Brug `config.apply` for at validere + skrive den fulde konfiguration og genstarte Gateway i et trin.
Det skriver en genstart sentinel og pings den sidste aktive session efter Gateway kommer tilbage.

Advarsel: `config.apply` erstatter **hele konfigurationen**. Hvis du kun ønsker at ændre nogle få nøgler, så brug
`config.patch` eller `openclaw config set`. Hold en sikkerhedskopi af `~/.openclaw/openclaw.json`.

Parametre:

- `raw` (string) — JSON5-payload for hele konfigurationen
- `baseHash` (valgfri) — konfigurations-hash fra `config.get` (påkrævet når en konfiguration allerede findes)
- `sessionKey` (valgfri) — nøgle for senest aktive session til wake-up ping
- `note` (valgfri) — note, der inkluderes i genstarts-sentinellen
- `restartDelayMs` (valgfri) — forsinkelse før genstart (standard 2000)

Eksempel (via `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Delvise opdateringer (RPC)

Brug `config.patch` til at flette en delvis opdatering i den eksisterende config uden at tilberede
ikke-relaterede nøgler. Det gælder JSON merge patch semantiske:

- objekter merges rekursivt
- `null` sletter en nøgle
- arrays erstattes
  Ligesom `config.apply` validerer den, skriver konfigurationen, gemmer en genstarts-sentinel og planlægger
  Gateway-genstarten (med en valgfri wake når `sessionKey` er angivet).

Parametre:

- `raw` (string) — JSON5-payload, der kun indeholder de nøgler, der skal ændres
- `baseHash` (påkrævet) — konfigurations-hash fra `config.get`
- `sessionKey` (valgfri) — nøgle for senest aktive session til wake-up ping
- `note` (valgfri) — note, der inkluderes i genstarts-sentinellen
- `restartDelayMs` (valgfri) — forsinkelse før genstart (standard 2000)

Eksempel:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Minimal konfiguration (anbefalet startpunkt)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Byg standardbilledet én gang med:

```bash
scripts/sandbox-setup.sh
```

## Self-chat-tilstand (anbefalet til gruppestyring)

For at forhindre botten i at svare på WhatsApp @-mentions i grupper (kun svare på specifikke tekst-triggere):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## Konfigurations-inkluderinger (`$include`)

Opdel din config i flere filer ved hjælp af `$include` -direktivet. Dette er nyttigt for:

- Organisering af store konfigurationer (fx agentdefinitioner pr. klient)
- Deling af fælles indstillinger på tværs af miljøer
- Adskillelse af følsomme konfigurationer

### Grundlæggende brug

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### Merge-adfærd

- **Enkelt fil**: Erstatter objektet, der indeholder `$include`
- **Array af filer**: Deep-merger filer i rækkefølge (senere filer overskriver tidligere)
- **Med søskendenøgler**: Søskendenøgler merges efter includes (overskriver inkluderede værdier)
- **Søskendenøgler + arrays/primitiver**: Ikke understøttet (inkluderet indhold skal være et objekt)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Indlejrede includes

Inkluderede filer kan selv indeholde `$include`-direktiver (op til 10 niveauer dybt):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### Stiopløsning

- **Relative stier**: Opløses relativt til den inkluderende fil
- **Absolutte stier**: Bruges som de er
- **Overordnede mapper**: `../`-referencer virker som forventet

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### Fejlhåndtering

- **Manglende fil**: Klar fejl med den opløste sti
- **Parse-fejl**: Viser hvilken inkluderet fil der fejlede
- **Cirkulære includes**: Detekteres og rapporteres med include-kæde

### Eksempel: Multi-klient juridisk opsætning

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## Almindelige indstillinger

### Miljøvariabler + `.env`

OpenClaw læser miljøvariabler fra forældreprocessen (shell, launchd/systemd, CI, osv.).

Derudover indlæser den:

- `.env` fra den aktuelle arbejdsmappe (hvis til stede)
- en global fallback `.env` fra `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Ingen af `.env`-filerne overskriver eksisterende miljøvariabler.

Du kan også give inline env vars i config. Disse anvendes kun, hvis
-processen env mangler nøglen (samme ikke-overordnede regel):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

Se [/environment](/help/environment) for fuld præcedens og kilder.

### `env.shellEnv` (valgfri)

Opt-in bekvemmelighed: hvis aktiveret og ingen af de forventede nøgler er sat endnu, OpenClaw kører din login-shell og importerer kun de manglende forventede nøgler (aldrig tilsidesætter).
Dette giver effektivt din shell-profil.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Miljøvariabel-ækvivalent:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### Substitution af miljøvariabler i konfigurationen

Du kan referere miljøvariabler direkte i en hvilken som helst konfigurationsstrengværdi ved hjælp af
`${VAR_NAME}` syntaks. Variabler erstattes ved konfigurationstid, før validering.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**Regler:**

- Kun store bogstaver i env-var-navne matches: `[A-Z_][A-Z0-9_]*`
- Manglende eller tomme env-vars giver fejl ved konfigurationsindlæsning
- Escap med `$${VAR}` for at outputte en bogstavelig `${VAR}`
- Virker med `$include` (inkluderede filer får også substitution)

**Inline-substitution:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### Auth-lagring (OAuth + API-nøgler)

OpenClaw gemmer **per-agent** auth profiler (OAuth + API-nøgler) i:

- `<agentDir>/auth-profiles.json` (standard: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

Se også: [/concepts/oauth](/concepts/oauth)

Legacy OAuth-importer:

- `~/.openclaw/credentials/oauth.json` (eller `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

Den indlejrede Pi-agent vedligeholder en runtime-cache i:

- `<agentDir>/auth.json` (administreres automatisk; redigér ikke manuelt)

Legacy agent-mappe (før multi-agent):

- `~/.openclaw/agent/*` (migreres af `openclaw doctor` til `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Overrides:

- OAuth-mappe (kun legacy import): `OPENCLAW_OAUTH_DIR`
- Agent-mappe (standard agent-root override): `OPENCLAW_AGENT_DIR` (foretrukken), `PI_CODING_AGENT_DIR` (legacy)

Ved første brug importerer OpenClaw `oauth.json`-poster til `auth-profiles.json`.

### `auth`

Valgfri metadata for auth profiler. Dette gør **ikke** butik hemmeligheder det kortlægger
profil-ID'er til en udbyder + tilstand (og valgfri e-mail) og definerer udbyder
rotation rækkefølge, der anvendes til failover.

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

### `agents.list[].identity`

Valgfri per-agent identitet brugt til standard og UX. Dette er skrevet af macOS onboarding assistent.

Hvis sat, afleder OpenClaw standarder (kun når du ikke har sat dem eksplicit):

- `messages.ackReaction` fra den **aktive agent**s `identity.emoji` (falder tilbage til 👀)
- `agents.list[].groupChat.mentionPatterns` fra agentens `identity.name`/`identity.emoji` (så “@Samantha” virker i grupper på tværs af Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
- `identity.avatar` accepterer en arbejdsrumsrelativ billedsti eller en ekstern URL/data URL. Lokale filer skal leve inde i agentens arbejdsområde.

`identity.avatar` accepterer:

- Workspace-relativ sti (skal forblive inden for agentens workspace)
- `http(s)`-URL
- `data:`-URI

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

### `wizard`

Metadata skrevet af CLI-opsætningsguides (`onboard`, `configure`, `doctor`).

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

### `logging`

- Standard logfil: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- Hvis du vil have en stabil sti, så sæt `logging.file` til `/tmp/openclaw/openclaw.log`.
- Konsol-output kan justeres separat via:
  - `logging.consoleLevel` (standard `info`, øges til `debug` når `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- Værktøjssammendrag kan redigeres for at undgå læk af hemmeligheder:
  - `logging.redactSensitive` (`off` | `tools`, standard: `tools`)
  - `logging.redactPatterns` (array af regex-strenge; overskriver standarder)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

Styrer hvordan WhatsApp direkte chats (DM’er) håndteres:

- `"pairing"` (standard): ukendte afsendere får en parringskode; ejeren skal godkende
- `"allowlist"`: tillad kun afsendere i `channels.whatsapp.allowFrom` (eller parret tilladelseslager)
- `"open"`: tillad alle indgående DM’er (**kræver** at `channels.whatsapp.allowFrom` inkluderer `"*"`)
- `"disabled"`: ignorér alle indgående DM’er

Parringskoder udløber efter 1 time; botten sender kun en parringskode, når en ny anmodning oprettes. Afventende DM-parringsanmodninger er som standard begrænset til **3 pr. kanal**

Parringsgodkendelser:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

Allowlist of E.164 phone numre, der kan udløse WhatsApp auto-svar (**DMs kun**).
Hvis tom og `channels.whatsapp.dmPolicy="parring"`, ukendte afsendere vil modtage en parringskode.
For grupper, brug `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

Kontrollerer, om indgående WhatsApp beskeder er markeret som læse (blå flåter). Standard: `sand`.

Self-chat-tilstand springer altid læsekvitteringer over, selv når aktiveret.

Per-account tilsidesættelse: \`channels.whatsapp.accounts.<id>.sendReadReceipts«.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (multi-konto)

Kør flere WhatsApp-konti i én gateway:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

Noter:

- Udgående kommandoer bruger som standard konto `default` hvis til stede; ellers den første konfigurerede konto-id (sorteret).
- Den legacy single-konto Baileys auth-mappe migreres af `openclaw doctor` til `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Kør flere konti pr. kanal (hver konto har sin egen `accountId` og valgfri `name`):

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

Noter:

- `default` bruges, når `accountId` udelades (CLI + routing).
- Env-tokens gælder kun for **standard**kontoen.
- Basiskanalindstillinger (gruppepolitik, omtale gating, etc.) anvende på alle konti, medmindre tilsidesættes pr. konto.
- Brug `bindings[].match.accountId` til at route hver konto til forskellige agents.defaults.

### Mention-gating i gruppechats (`agents.list[].groupChat` + `messages.groupChat`)

Grupper beskeder som standard **kræver navn** (enten metadata omtale eller regex mønstre). Gælder WhatsApp, Telegram, Discord, Google Chat og iMessage gruppechats.

**Mention-typer:**

- **Metadata omtaler**: Native platform @-mentions (f.eks. WhatsApp tap-to-mention). Ignoreret i WhatsApp selv-chat tilstand (se `channels.whatsapp.allowFrom`).
- **Tekstmønstre**: Regex mønstre defineret i `agents.list[].groupChat.mentionPatterns`. Kontroller altid uanset selv-chat-tilstand.
- Mention-gating håndhæves kun, når mention-detektion er mulig (native mentions eller mindst én `mentionPattern`).

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

`messages.groupChat.historyLimit` sætter den globale standard for gruppe-historik-kontekst. Kanaler kan tilsidesætte med `kanaler.<channel>.historyLimit` (eller 'kanaler).&lt;channel&gt;.accounts.\*.historyLimit`for multi-konto). Sæt`0\` for at deaktivere historik wrapping.

#### DM-historikgrænser

DM samtaler bruger sessionsbaseret historie forvaltes af agent. Du kan begrænse antallet af bruger-drejninger pr. DM-session:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

Opløsningsrækkefølge:

1. Per-DM tilsidesættelse: `kanaler.<provider>.dms[userId].historyLimit`
2. Leverandør standard: \`kanaler.<provider>.dmHistoryLimit«
3. Ingen grænse (al historik bevares)

Understøttede udbydere: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Override pr. agent (har forrang når sat, selv `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

Nævn gating defaults live per channel (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Når `*.groups` er indstillet, virker det også som en gruppe tilladte; inkludere `"*"` for at tillade alle grupper.

For kun at svare på specifikke tekst-triggere (ignorere native @-mentions):

```json5
{
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### Gruppepolitik (pr. kanal)

Brug `channels.*.groupPolicy` til at styre, om gruppe-/rum-beskeder overhovedet accepteres:

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

Noter:

- `"open"`: grupper omgår tilladelseslister; mention-gating gælder stadig.
- `"disabled"`: bloker alle gruppe-/rum-beskeder.
- `"allowlist"`: tillad kun grupper/rum, der matcher den konfigurerede tilladelsesliste.
- `channels.defaults.groupPolicy` sætter standarden, når en udbyders `groupPolicy` er usat.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams bruger `groupAllowFrom` (fallback: eksplicit `allowFrom`).
- Discord/Slack bruger kanal-tilladelseslister (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
- Gruppe-DM’er (Discord/Slack) styres stadig af `dm.groupEnabled` + `dm.groupChannels`.
- Standard er `groupPolicy: "allowlist"` (medmindre overskrevet af `channels.defaults.groupPolicy`); hvis ingen tilladelsesliste er konfigureret, blokeres gruppebeskeder.

### Multi-agent routing (`agenter.list` + `bindinger`)

Kør flere isolerede agenter (separat arbejdsområde, `agentDir`, sessioner) inde i en Gateway.
Indgående beskeder sendes til en agent via bindinger.

- `agenter.list[]`: per-agent tilsidesættelser.
  - `id`: stabil agent id (påkrævet).
  - `default`: valgfri; når flere er indstillet, er de første gevinster og en advarsel logget.
    Hvis ingen er angivet, er **første indgang** i listen standardagent.
  - `navn`: vis navn for agenten.
  - `workspace`: standard `~/.openclaw/workspace-<agentId>` (for `main`, falder tilbage til `agents.defaults.workspace`).
  - `agentDir`: standard `~/.openclaw/agents/<agentId>/agent`.
  - `model`: per-agent standardmodel, tilsidesætter `agents.defaults.model` for denne agent.
    - strengformular: `"provider/model"`, tilsidesætter kun `agents.defaults.model.primary`
    - object form: `{ primary, fallbacks }` (fallbacks tilsidesætte `agents.defaults.model.fallbacks`; `[]` deaktiverer globale tilbagefald for denne agent)
  - `identity`: per-agent navn/theme/emoji (bruges til at nævne mønstre + ack reaktioner).
  - `groupChat`: per-agent mention-gating (`mentionPatterns`).
  - `sandbox`: per-agent sandkasse config (tilsidesætter `agents.defaults.sandbox`).
    - `mode`: `"off"` ¤ `"non-main"` ● `"all"`
    - `workspaceAccess`: `"none"` ¤ `"ro"` ¤ `"rw"`
    - `scope`: `"session"` ¤ `"agent"` ● `"delt"`
    - `workspaceRoot`: brugerdefineret sandkasse arbejdsområde rod
    - `docker`: per-agent docker tilsidesættelser (f.eks. `image`, `network`, `env`, `setupCommand`, grænser; ignoreret når `scope: "shared"`)
    - `browser`: per-agent sandboxed browser overrides (ignoreres når `scope: "shared"`)
    - `prune`: per-agent sandkasse beskæring tilsidesættelser (ignoreret ved `scope: "shared"`)
  - `subagenter`: per-agent sub-agent defaults.
    - `allowAgents`: allowlist of agent ids for `sessions_spawn` from this agent (`["*"]` = allow any; default: only same agent)
  - `tools`: per-agent værktøj restriktioner (anvendes før sandkasse værktøj politik).
    - `profil`: base tool profil (anvendes før tillad/nægt)
    - `allow`: matrix af tilladte værktøjsnavne
    - `deny`: array af nægtede værktøjsnavne (benægte vinder)
- `agents.defaults`: shared agent defaults (model, arbejdsområde, sandkasse, etc.).
- `bindings[]`: ruter indgående beskeder til en `agentId`.
  - `match.channel` (påkrævet)
  - `match.accountId` (valgfri; `*` = enhver konto; udeladt = standardkonto)
  - `match.peer` (valgfri; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (valgfri; kanalspecifikt)

Deterministisk match rækkefølge:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, no peer/guild/team)
5. `match.accountId: "*"` (channel-wide, no peer/guild/team)
6. standard agent (`agents.list[].default`, ellers første liste post, ellers `"main"`)

Inden for hvert matchningstrin vinder den første matchende post i 'bindinger'.

#### Adgangsprofiler pr. agent (multi-agent)

Hver agent kan bære sin egen sandkasse + værktøjspolitik. Brug dette til at blande adgang
niveauer i én gateway:

- **Fuld adgang** (personlig agent)
- **Læs-kun** værktøjer + arbejdsområde
- **Ingen adgang til filsystemet** (kun besked/sessionsværktøjer)

Se [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for forrang og
yderligere eksempler.

Fuld adgang (ingen sandkasse):

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

Skrivebeskyttet værktøj + skrivebeskyttet arbejdsområde:

```json5
{
  agenter: {
    liste: [
      {
        id: "family",
        arbejdsområde: "~/. benclaw/workspace-family",
        sandbox: {
          tilstand: "all",
          anvendelsesområde: "agent"
          workspaceAccess: "ro",
        },
        værktøjer: {
          tillader: [
            "læst",
            "sessions_list"
            "sessions_history"
            "sessions_send"
            "sessions_spawn"
            "session_status"
          ]
          benægt: ["skriv", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

Ingen adgang til filsystemet (besked / session værktøjer aktiveret):

```json5
{
  agenter: {
    liste: [
      {
        id: "public",
        arbejdsområde: "~/. benclaw/workspace-public",
        sandbox: {
          tilstand: "all",
          anvendelsesområde: "agent"
          workspaceAccess: "none",
        },
        værktøjer: {
          tillader: [
            "sessions_list",
            "sessions_history"
            "sessions_send"
            "sessions_spawn"
            "session_status"
            "whatsapp"
            "telegram"
            "slack"
            "discord"
            "gateway"
          ]
          benægt: [
            "læst"
            "skriv"
            "redigér"
            "apply_patch"
            "exec"
            "proces"
            "browser"
            "canvas"
            "noder"
            "cron"
            "gateway"
            "billede"
          ],
        },
      },
    ],
  },
}
```

Eksempel: to WhatsApp-konti → to agenter:

```json5
{
  agenter: {
    liste: [
      { id: "home", default: true, arbejdsrum: "~/. penclaw/workspace-home" },
      { id: "work", workspace: "~/. penclaw/workspace-work" },
    ],
  },
  bindinger: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp" accountId: "biz" } },
  ]
  kanaler: {
    whatsapp: {
      konti: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (valgfrit)

Agent-til-agent besked er fravalg:

```json5
{
  værktøjer: {
    agentToAgent: {
      aktiveret: false,
      tillader: ["home", "work"],
    },
  },
}
```

### `messages.queue`

Kontrollerer, hvordan indgående beskeder opfører sig, når en agent kører er allerede aktiv.

```json5
{
  meddelelser: {
    kø: {
      tilstand: "collect", // steer - followup - up - samle - steer-backlog (steer+backlog ok) - interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarisk", // gammeldags ny, opsummerer
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect"
        billede: "collect",
        webchat: "collect"
      },
    },
  },
}
```

### `messages.inbound`

Debounce hurtige indgående beskeder fra den **samme afsender** så flere back-to-back
beskeder bliver en enkelt agent tur. Debouncing er scoped per kanal + conversation
og bruger den seneste meddelelse til svar tråde / IDs.

```json5
{
  meddelelser: {
    indbundet: {
      debounceMs: 2000, // 0 deaktiverer
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      },
    },
  },
}
```

Noter:

- Debounce batches **text-only** beskeder; medie/vedhæftede filer flush straks.
- Kontrolkommandoer (f.eks. `/kø`, `/new`) bypass debouncing, så de forbliver standalone.

### `commands` (chat kommando håndtering)

Kontrollerer, hvordan chatkommandoer er aktiveret på tværs af konnektorer.

```json5
{
  kommandoer: {
    indfødt: "auto", // registrere indfødte kommandoer når understøttet (auto)
    tekst: true, // parse skråstreg kommandoer i chatbeskeder
    bash: false, // tillad ! (Alias: /bash) (kun vært; kræver værktøjer. levated allowlists)
    bashForegroundMs: 2000, // bash forgrundsvindue (0 baggrunde straks)
    config: false, // tillad /config (skriver til disk)
    debug: false, // tillad /debug (runtime-only overrides)
    genstart: false, // tillad /genstart + gateway genstart værktøj
    useAccessGroups: true, // håndhæve access-group allowlists/policies for kommandoer
  },
}
```

Noter:

- Tekst kommandoer skal sendes som en **standalone** besked og bruge de ledende `/` (ingen almindelig tekst aliaser).
- `commands.text: false` deaktiverer parsing chatbeskeder for kommandoer.
- `commands.native: "auto"` (default) slår lokale kommandoer til for Discord/Telegram og efterlader Slack off; ikke-understøttede kanaler forbliver kun tekst-kun.
- Set `commands.native: trueřfalse` at tvinge alle, eller tilsidesætte per kanal med `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool eller `"auto"`). `false` rydder tidligere registrerede kommandoer på Discord/Telegram ved opstart; Slack kommandoer håndteres i Slack appen.
- `channels.telegram.customCommands` tilføjer ekstra Telegram bot menu poster. Navne er normaliseret; konflikter med indfødte kommandoer ignoreres.
- `commands.bash: true` aktiverer `! <cmd>` til at køre vært shell kommandoer (`/bash <cmd>` fungerer også som et alias). Kræver `tools.elevated.enabled` og tillad afsenderen i `tools.elevated.allowFrom.<channel>`.
- `commands.bashForegroundMs` styrer hvor længe bash venter før baggrund. Mens et bash job kører, nyt `! <cmd>` anmodninger afvises (en ad gangen).
- `commands.config: true` muliggør `/config` (reads/write `openclaw.json`).
- `kanaler.<provider>.configWrites` gates config mutationer initieret af denne kanal (standard: true). Dette gælder for `/config sæt, unset` plus provider-specifikke auto-migrationer (Telegram supergroup ID ændringer, Slack kanal ID ændringer).
- `commands.debug: true` muliggør `/debug` (runtime-only overrides).
- `commands.restart: true` aktiverer `/restart` og gateway-værktøjets genstart handling.
- `commands.useAccessGroups: false` tillader kommandoer at omgå access-group allowlists/policies.
- Slash kommandoer og direktiver er kun hædret for **autoriserede afsendere**. Authorization is derived from
  channel allowlists/pairing plus `commands.useAccessGroups`.

### `web` (WhatsApp web-kanal runtime)

WhatsApp kører gennem gatewayens web-kanal (Baileys Web). Det starter automatisk, når en linket session eksisterer.
Sæt `web.enabled: false` for at holde den slukket som standard.

```json5
{
  web: {
    aktiveret: sandt,
    hjerteslagSekunder: 60,
    genforbindelse: {
      initialMs: 2000,
      maxMs: 120000,
      faktor: 1. ,
      jitter: 0. ,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram` (bot transport)

OpenClaw starter Telegram kun når en `channels.telegram` config sektion eksisterer. Bot token er løst fra `channels.telegram.botToken` (eller `channels.telegram.tokenFile`), med `TELEGRAM_BOT_TOKEN` som en fallback for standardkontoen.
Indstil `channels.telegram.enabled: false` for at deaktivere automatisk opstart.
Multi-konto support lever under `channels.telegram.accounts` (se multikonto afsnittet ovenfor). Env tokens gælder kun for standardkontoen.
Sæt `channels.telegram.configWrites: false` for at blokere Telegram-initieret config skriver (herunder supergruppe-ID migrationer og `/config sæt unset`).

```json5
{
  kanaler: {
    telegram: {
      aktiveret: true,
      botToken: "your-bot-token"
      dmPolicy: "parring", // parring - tilladelse:Åbent afbrudt
      tilladtFra: ["tg:123456789"], // valgfri; "open" kræver ["*"]
      grupper: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFra: ["@admin"],
          systemPrompt: "Hold svar kort. ,
          emner: {
            "99": {
              requireMention: false,
              færdigheder: ["søgning"],
              systemPrompt: "Bliv ved emnet. ,
            },
          },
        },
      },
      customCommands: [
        { command: "backup", beskrivelse: "Git backup" },
        { kommando: "generate", beskrivelse: "Opret et billede" },
      ],
      historyLimit: 50, // include sidste N gruppe beskeder som kontekst (0 disables)
      replyToMode: "først" // sluk for første
      linkPreview: true, // skift udgående link previews
      streamMode: "partial", // sluk - delvis - blok (udkast streaming; adskilt fra blokstreaming)
      udkastChunk: {
        // valgfri kun for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph " // paragraf nylinje- sætning
      },
      handlinger: { reactions: true, sendMessage: true }, / værktøj handling gates (falske deaktiverer)
      reaktionMeddelelser: "egen" // sluk af... egen - alle
      medierMaxMb: 5,
      retry: {
        // udgående genforsøg politik
        forsøg: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0. ,
      },
      netværk: {
        // transport overrides
        autoSelectFamilie: false,
      },
      proxy: "sokker5://localhost:9050",
      webhookUrl: "https://example om/telegram-webhook", // kræver webhookSecret
      webhookSecret: "hemmelig"
      webhookSti: "/telegram-webhook",
    },
  },
}
```

Udkast til streaming noter:

- Bruger Telegram `sendMessageDraft` (kladde boble, ikke en reel besked).
- Kræver **private chat-emner** (message_thread_id i DMs; bot har emner aktiveret).
- `/ræsonnement stream` streams ræsonnement i udkastet, derefter sender det endelige svar.
  Forsøg politik standarder og adfærd er dokumenteret i [Prøv politik] (/concepts/retry).

### `channels.discord` (bot transport)

Konfigurere Discord bot ved at indstille bot token og valgfri gating:
Multi-konto support lever under `channels.discord.accounts` (se multikonto afsnittet ovenfor). Env tokens gælder kun for standardkontoen.

```json5
{
  kanaler: {
    discord: {
      aktiveret: true,
      token: "your-bot-token"
      mediaMaxMb: 8, // klemme indgående mediestørrelse
      tillader Bots: false, // Tillad bot-forfattede beskeder
      handlinger: {
        // værktøj handling gates (falske deaktiverer)
        reaktioner: sandt,
        klistermærker: true,
        meningsmålinger: true,
        tilladelser: sand,
        meddelelser: sand,
        tråde: sand,
        pins: true,
        søgning: true,
        medlemInfo: true,
        rolleInfo: true,
        roller: false,
        channelInfo: true,
        voiceStatus: sandt,
        begivenheder: sand,
        moderation: false,
      },
      replyToMode: "off", // sluk for første
      dm: {
        aktiveret: true, // deaktiver alle DMs når falsk
        politik: "parring", // parring - tillad - åben - åben - deaktiveret
        allowFra: ["1234567890", "steipete"], // valgfri DM allowlist ("open" kræver ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // valgfri gruppe DM tilladt liste
      },
      guilds: {
        "123456789012345678": {
          // guild id (foretrukket) eller slug
          slug: "friends-of-openclaw"
          kravOmtale: falsk, // per-guild default
          reactionNotifications: "own", // sluk
          brugere: ["987654321098765432"], // valgfri per-guild bruger allowlist
          kanaler: {
            general: { allow: true },
            hjælp: {
              tillader: sand,
              kravOmtale: sandt,
              brugere: ["987654321098765432"],
              færdigheder: ["dokumenter"],
              systemPrompt: "Kun korte svar. ,
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000 // valgfri udgående tekst chunk size (chars)
      chunkMode: "length", / / valgfri chunking tilstand (længden jævnt newline)
      maksLinjerPerMessage: 17, // bløde maks linjer pr. besked (Discord UI klipning)
      genforsøg: {
        // outbound retry policy
        forsøg: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0. ,
      },
    },
  },
}
```

OpenClaw starter Discord kun når en `channels.discord` config sektion eksisterer. Tokenen er løst fra `channels.discord.token`, med `DISCORD_BOT_TOKEN` som en fallback for standardkontoen (medmindre `channels.discord.enabled` er `false`). Brug `user:<id>` (DM) eller `kanal:<id>` (guild channel) når du angiver leveringsmål for cron/CLI kommandoer; bare numeriske id'er er tvetydige og afvist.
Guild snegle er små bogstaver med mellemrum erstattet af `-`; kanal nøgler bruger den træge kanal navn (ingen ledende `#`). Foretræk guild ids som nøgler for at undgå at omdøbe tvetydighed.
Bot-forfattede beskeder ignoreres som standard. Aktiver med `channels.discord.allowBots` (egne beskeder filtreres stadig for at forhindre selvsvarsløkker).
Reaktion notifikation tilstande:

- `off`: ingen reaktions-events.
- `own`: reaktioner på bottens egne beskeder (standard).
- `all`: alle reaktioner på alle beskeder.
- `allowlist`: reaktioner fra `guilds.<id>.users` på alle beskeder (tomme liste deaktiverer).
  Outbound tekst er chunked af `channels.discord.textChunkLimit` (standard 2000). Sæt `channels.discord.chunkMode="newline"` til at opdele på tomme linjer (afsnit grænser) før længde chunking. Discord klienter kan klippe meget høje beskeder, så `channels.discord.maxLinesPerMessage` (standard 17) opdeler lange multi-line svar, selv når under 2000 tegn.
  Forsøg politik standarder og adfærd er dokumenteret i [Prøv politik] (/concepts/retry).

### `channels.googlechat` (Chat API webhook)

Google Chat kører over HTTP webhooks med app-level auth (servicekonto).
Multi-konto support lever under `channels.googlechat.accounts` (se multikonto afsnittet ovenfor). Env vars gælder kun for standardkontoen.

```json5
{
  kanaler: {
    googlechat: {
      aktiveret: true,
      serviceAccountFile: "/path/to/service-konto. son",
      publikumType: "app-url", // app-url ja-projektnummer
      publikum: "https://gateway.example om/googlechat",
      webhookPath: "/googlechat",
      botBruger: "users/1234567890", // valgfri forbedrer detektion
      dm: {
        aktiveret: true
        politik: "parring" // parringstilstand allowlist Fælled open - disabled
        allowFra: ["brugere/1234567890"], // valgfri; "open" kræver ["*"]
      },
      groupPolicy: "allowlist"
      grupper: {
        "mellemrum/AAAA": { allow: true, requireMention: true }
      },
      handlinger: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Noter:

- Tjenesteydelseskonto JSON kan være inline (`serviceAccount`) eller fil-baseret (`serviceAccountFile`).
- Env fallbacks for standardkontoen: `GOOGLE_CHAT_SERVICE_ACCOUNT` eller `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- `audienceType` + `audience` skal matche Chat app ‘ s webhook auth config.
- Brug `spaces/<spaceId>` eller `users/<userId|email>` ved indstilling af leveringsmål.

### `channels.slack` (sokkeltilstand)

Slack kører i Socket Mode og kræver både en bot token og app token:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // optional; "open" requires ["*"]
        groupEnabled: false,
        groupChannels: ["G123"],
      },
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
      historyLimit: 50, // include last N channel/group messages as context (0 disables)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
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

Multi-konto support lever under `channels.slack.accounts` (se multikonto afsnittet ovenfor). Env tokens gælder kun for standardkontoen.

OpenClaw starter Slack når udbyderen er aktiveret og begge tokens er indstillet (via config eller `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Brug `user:<id>` (DM) eller `kanal:<id>` når du angiver leveringsmål for cron/CLI kommandoer.
Sæt `channels.slack.configWrites: false` for at blokere Slack-initieret config skriver (herunder kanal-ID migrationer og `/config sæt, unset`).

Bot-forfattede beskeder ignoreres som standard. Aktiver med `channels.slack.allowBots` eller `channels.slack.channels.<id>.allowBots`.

Reaktion notifikation tilstande:

- `off`: ingen reaktions-events.
- `own`: reaktioner på bottens egne beskeder (standard).
- `all`: alle reaktioner på alle beskeder.
- `allowlist`: reaktioner fra `channels.slack.reactionAllowlist` på alle beskeder (tomme liste disables).

Tråd session isolation:

- `channels.slack.thread.historyScope` styrer om trådhistorikken er per-thread (`thread`, default) eller delt på tværs af kanalen (`kanal`).
- `channels.slack.thread.inheritParent` styrer om nye trådssessioner arver den overordnede kanaludskrift (standard: false).

Slack action grupper (gate `slack` værktøj handlinger):

| Handlingsgruppe | Standard | Noter                       |
| --------------- | -------- | --------------------------- |
| reactions       | enabled  | Reagér + list reaktioner    |
| messages        | enabled  | Læs/send/redigér/slet       |
| pins            | enabled  | Pin/afpin/list              |
| memberInfo      | enabled  | Medlemsinfo                 |
| emojiList       | enabled  | Brugerdefineret emoji-liste |

### `channels.mattermost` (bot token)

Mattermost leveres som et plugin og er ikke inkluderet i kerneinstallationen.
Installer det først: `openclaw plugins installere @openclaw/mattermost` (eller `./extensions/mattermost` fra en git checkout).

Mattermost kræver en bot token plus base URL til din server:

```json5
{
  kanaler: {
    mattermost: {
      aktiveret: true,
      botToken: "mm-token",
      baseUrl: "https://chat. xample. om",
      dmPolicy: "parring",
      chatmode: "oncall", // oncall tirrsel onmesse, onchar
      oncharPrefixes: [">", "! ],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

OpenClaw starter mest når kontoen er konfigureret (bot token + base URL) og aktiveret. Token + base URL er løst fra `channels.mattermost.botToken` + `channels.mattermost.baseUrl` eller `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` for standardkontoen (medmindre `channels.mattermost.enabled` er `false`).

Chat tilstande:

- `oncall` (standard): svar kun på kanalbeskeder når @mentioned.
- `onmessage`: svar på hver kanalbesked.
- `onchar`: svarer, når en besked starter med et trigger præfiks (`channels.mattermost.oncharPrefixes`, standard `[">", "!"]`).

Adgangskontrol:

- Standard DMs: `channels.mattermost.dmPolicy="pairing"` (ukendte afsendere får en parringskode).
- Offentlige DMs: `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.
- Grupper: `channels.mattermost.groupPolicy="allowlist"` som standard (mention-gated). Brug `channels.mattermost.groupAllowFrom` for at begrænse afsendere.

Multi-konto support lever under `channels.mattermost.accounts` (se multikonto afsnittet ovenfor). Env vars gælder kun for standardkontoen.
Brug `kanal:<id>` eller `user:<id>` (eller `@username`) ved angivelse af leveringsmål; bare id'er behandles som kanalid'er.

### `channels.signal` (signal-cli)

Signal reaktioner kan udsende system hændelser (delt reaktion værktøj):

```json5
{
  kanaler: {
    signal: {
      reaktionMeddelelser: "own", // sluk af... egen - alle - tilladt liste
      reaktionTillad: ["+15551234567" "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // omfatter sidste N-gruppemeddelelser som kontekst (0 deaktiverer)
    },
  },
}
```

Reaktion notifikation tilstande:

- `off`: ingen reaktions-events.
- `own`: reaktioner på bottens egne beskeder (standard).
- `all`: alle reaktioner på alle beskeder.
- `allowlist`: reaktioner fra `channels.signal.reactionAllowlist` på alle beskeder (tomme liste disables).

### `channels.imessage` (imsg CLI)

OpenClaw spawns `imsg rpc` (JSON-RPC over stdio). Ingen dæmon eller port påkrævet.

```json5
{
  kanaler: {
    imessage: {
      aktiveret: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat. b",
      remoteHost: "user@gateway-host", // SCP for eksterne vedhæftede filer, når du bruger SSH wrapper
      dmPolicy: "pairing", // parring - tillad - åben - åben - deaktiveret
      allowFra: ["+15555550123", "user@eksempel. om", "chat_id:123"],
      historyLimit: 50, // omfatter sidste N gruppe beskeder som kontekst (0 disables)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

Multi-konto support liv under `channels.imessage.accounts` (se multikonto afsnittet ovenfor).

Noter:

- Kræver fuld diskadgang til meddelelserne DB.
- Den første afsendelse vil spørge om Beskeder automatisering tilladelse.
- Foretræk `chat_id:<id>` mål. Brug `imsg chats --limit 20` til at liste chats.
- `channels.imessage.cliPath` kan pege på en wrapper script (fx `ssh` til en anden Mac, der kører `imsg rpc`); bruge SSH nøgler til at undgå adgangskodeprompter.
- For eksterne SSH wrappers, sæt `channels.imessage.remoteHost` for at hente vedhæftede filer via SCP når `includeAttachments` er aktiveret.

Eksempel-wrapper:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

Sætter den **single global arbejdsområde-mappe** der bruges af agenten til filoperationer.

Standard: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Hvis `agents.defaults.sandbox` er aktiveret, kan ikke-hovedsessioner tilsidesætte dette med deres
egne per-scope arbejdsområder under `agents.defaults.sandbox.workspaceRoot`.

### `agents.defaults.repoRoot`

Valgfri repository rod, der skal vises i systempromptens Runtime linje. Hvis frakoblet, forsøger OpenClaw
at detektere en `.git` mappe ved at gå opad fra arbejdsområdet (og nuværende
arbejdsmappe). Stien skal eksistere for at blive brugt.

```json5
{
  agenter: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Deaktiverer automatisk oprettelse af arbejdsrummet bootstrap filer (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, og `BOOTSTRAP.md`).

Brug dette til pre-seeded implementeringer, hvor dit arbejdsområde filer kommer fra en repo.

```json5
{
  agenter: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Maks tegn i hvert arbejdsområde bootstrap fil injiceres i systemet prompt
før trunkering. Standard: `20000`.

Når en fil overskrider denne grænse, logger OpenClaw en advarsel og injicerer en afkortet
hoved/hale med en markør.

```json5
{
  agenter: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

Sætter brugerens tidszone for **systemprompt-kontekst** (ikke for tidsstempler i
beskedkonvolutter). Hvis deaktiveret, bruger OpenClaw værtstidszonen på runtime.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Styrer **tidsformatet** vist i systempromptens sektion Nuværende dato og tid.
Standard: `auto` (OS præference).

```json5
{
  agenter: { defaults: { timeFormat: "auto" } }, // auto ¤ 12 ¤ 24
}
```

### `meddelelser`

Kontrollerer indgående / udgående præfikser og valgfri ack reaktioner.
Se [Messages](/concepts/messages) for kø, sessioner og streaming kontekst.

```json5
{
  meddelelser: {
    responsePrefix: "🦞", // eller "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterSvar: false,
  },
}
```

`responsePrefix` anvendes til **alle udgående svar** (værktøjs resuméer, blok
streaming, endelige svar) på tværs af kanaler, medmindre de allerede er til stede.

Tilsidesættelser kan konfigureres pr. kanal og pr. konto:

- \`kanaler.<channel>.responsePrefix«
- \`kanaler.<channel>.accounts.<id>.responsePrefix«

Opløsningsrækkefølge (mest specifik vinder):

1. \`kanaler.<channel>.accounts.<id>.responsePrefix«
2. \`kanaler.<channel>.responsePrefix«
3. `messages.responsePrefix`

Semantisk:

- `undefined` falder igennem til det næste niveau.
- `""` deaktiverer eksplicit præfikset og stopper kaskaden.
- `"auto"` afleder `[{identity.name}]` for den dirigerede agent.

Overskrivninger gælder for alle kanaler, herunder udvidelser, og for hver udgående svar slags.

Hvis `messages.responsePrefix` er deaktiveret, anvendes der ikke noget præfiks som standard. WhatsApp self-chat
svar er undtagelsen: de standard til `[{identity.name}]` når angivet, ellers
`[openclaw]`, så samme telefon samtaler forbliver læselige.
Set it to `"auto"` to derive `[{identity.name}]` for the routed agent (when set). (Automatic Copy)

#### Skabelon variabler

Strengen `responsePrefix` kan indeholde skabelonvariabler, der løser dynamisk:

| Variabel          | Beskrivelse                  | Eksempel                                         |
| ----------------- | ---------------------------- | ------------------------------------------------ |
| `{model}`         | Kort modelnavn               | `claude-opus-4-6`, `gpt-4o`                      |
| `{modelFull}`     | Identifikator for fuld model | `anthropic/claude-opus-4-6`                      |
| `{provider}`      | Navn på udbyder              | `antropi`, `openai`                              |
| `{thinkingLevel}` | Nuværende tankegang niveau   | `high`, `low`, `off`                             |
| `{identity.name}` | Agentens identitet navn      | (samme som `"auto"` tilstand) |

Variablerne er ufølsomme (`{MODEL}` = `{model}`). `{think}` er et alias for `{thinkingLevel}`.
Uløste variabler forbliver som bogstavelig tekst.

```json5
{
  meddelelser: {
    responsePrefix: "[{model} ● think:{thinkingLevel}]",
  },
}
```

Eksempel output: `[claude-opus-4-6 ţthink:high] Her er mit svar...`

WhatsApp indgående præfiks er konfigureret via `channels.whatsapp.messagePrefix` (forældet:
`messages.messagePrefix`). Standard forbliver **uændret**: `"[openclaw]"` når
`channels.whatsapp.allowFrom` er tom, ellers `""` (ingen præfiks). Når du bruger
`"[openclaw]"`, vil OpenClaw i stedet bruge `[{identity.name}]` når den dirigerede
agent har `identity.name` sæt.

`ackReaction` sender en bedste indsats emoji reaktion til at anerkende indgående meddelelser
på kanaler, der understøtter reaktioner (Slack/Discord/Telegram/Google Chat). Defaults to the
active agent’s `identity.emoji` when set, otherwise `"👀"`. Sæt den til `""` for at deaktivere.

`ackReactionScope` kontrollerer, når reaktionerne skyder:

- `group-mentions` (standard): kun når en gruppe/rum kræver nævner **og** botten blev nævnt
- `gruppe-all`: alle gruppe/værelses beskeder
- `direkte`: kun direkte beskeder
- `alle`: alle beskeder

`removeAckAfterReply` fjerner bot’s ack reaktion efter et svar er sendt
(Slack/Discord/Telegram/Google Chat kun). Standard: `falsk`.

#### `messages.tts`

Aktiver tekst-til-tale for udgående svar. Når tændt, OpenClaw genererer lyd
ved hjælp af ElevenLabs eller OpenAI og knytter den til svar. Telegram bruger Opus
stemmenoter; andre kanaler sender MP3 lyd.

```json5
{
  meddelelser: {
    tts: {
      auto: "altid", // slukket - altid - indadgående - mærket
      -tilstand: "endelig", // finale... alle (omfatter værktøj/blok replies)
      udbyder: "elevenlabs",
      resuméModel: "openai/gpt-4. -mini",
      modelTilladninger: {
        enabled: true,
      },
      maxTextLængde: 4000,
      timeoutMs: 30000,
      prefsPath: "~/. penclaw/settings/tts. son",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api. levenlabs. o",
        voiceId: "voice_id"
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalisering: "auto",
        languageCode: "en",
        stemmeIndstillinger: {
          stabilitet: 0. ,
          similarityBoost: 0. 5,
          stil: 0. ,
          useSpeakerBoost: true,
          hastighed: 1. ,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        stemme: "alloy"
      },
    },
  },
}
```

Noter:

- `messages.tts.auto` styrer auto-TTS (`off`, `always`, `inbound`, `tagged`).
- `/tts off¤ altid,inbound-tagged` sætter per-session auto mode (overrides config).
- `messages.tts.enabled` er legacy; læge migrerer det til `messages.tts.auto`.
- `prefsPath` gemmer lokale tilsidesættelser (provider/limit/summarize).
- `maxTextLength` er en hård hætte til TTS input; resuméer er afkortet til at passe.
- `summaryModel` tilsidesætter `agents.defaults.model.primary` for auto-resumé.
  - Accepterer `provider/model` eller et alias fra `agents.defaults.models`.
- `modelOverrides` muliggør model-drevne tilsidesættelser som `[[tts:...]]` tags (på som standard).
- `/tts limit` og `/tts summary` kontrol per bruger opsummering indstillinger.
- `apiKey` værdier falder tilbage til `ELEVENLABS_API_KEY`/`XI_API_KEY` og `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` tilsidesætter ElevenLabs API base URL.
- `elevenlabs.voiceSettings` understøtter `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost`, og `speed` (0,5..2.0).

### `talk`

Standard for Talk mode (macOS/iOS/Android). Stemme-ID'er falder tilbage til `ELEVENLABS_VOICE_ID` eller `SAG_VOICE_ID` når de ikke er angivet.
`apiKey` falder tilbage til `ELEVENLABS_API_KEY` (eller gatewayens shell profil) når den er frakoblet.
`voiceAliases` lader Tal direktiver bruger venlige navne (fx `"stemme":"Clawd"`).

```json5
{
  tale: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128"
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

Styrer den indlejrede agent runtime (model/tænkning/verbose/timeouts).
`agents.defaults.models` definerer den konfigurerede model katalog (og fungerer som den tilladte liste for `/model`).
`agents.defaults.model.primary` sætter standardmodellen; `agents.defaults.model.fallbacks` er globale failovers.
`agents.defaults.imageModel` er valgfri og bruges kun **hvis den primære model mangler billede input**.
Hver `agents.defaults.models` post kan omfatte:

- `alias` (valgfri genvej i modellen, f.eks. `/opus`).
- `params` (valgfri udbyder-specifikke API-params passeret til modelanmodningen).

`params` anvendes også til streaming kører (indlejret agent + komprimering). Understøttede nøgler i dag: `temperatur`, `maxTokens`. Disse sammenflette med call-time muligheder; opkalds-leverede værdier vinder. `temperatur` er en avanceret drejeknap – lad være frakoblet, medmindre du kender modellens standarder og har brug for en ændring.

Eksempel:

```json5
{
  agenter: {
    defaults: {
      modeller: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5. ": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x modeller aktiverer automatisk tænkemåde, medmindre du:

- sæt `-- thinking off`, eller
- definere `agents.defaults.models["zai/<model>"].params.thinking` selv.

OpenClaw også skibe et par indbyggede alias shorthands. Standarder gælder kun, når model
allerede er til stede i `agents.defaults.models`:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

Hvis du selv indstiller det samme aliasnavn (case-insensitive), vinder din værdi (standardværdier tilsidesætter aldrig).

Eksempel: Opus 4.6 primær med MiniMax M2.1 fallback (hosted MiniMax):

```json5
{
  agenter: {
    defaults: {
      modeller: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2. ": { alias: "minimax" },
      },
      model: {
        primær: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2. "],
      },
    },
  },
}
```

MiniMax auth: sæt `MINIMAX_API_KEY` (env) eller konfigurér `models.providers.minimax`.

#### `agents.defaults.cliBackends` (CLI fallback)

Valgfrie CLI- backends til fallback- kørsel (ingen værktøjs opkald). Disse er nyttige som en
backup sti, når API-udbydere mislykkes. Billede gennemkørsel er understøttet når du konfigurerer
et `imageArg` der accepterer filstier.

Noter:

- CLI backends er \*\*text-first \*\*; værktøjer er altid deaktiverede.
- Sessioner understøttes når `sessionArg` er indstillet; sessions-id'er er fortsatte per backend.
- For `claude-cli`, standardindstillinger er wired i. Tilsidesæt kommandostien hvis PATH er minimal
  (launchd/systemd).

Eksempel:

```json5
{
  agenter: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          kommando: "/opt/homebrew/bin/claude",
        },
        "min-kli": {
          kommando: "min-kli"
          args: ["--json"],
          output: "json"
          modelArg: "--model",
          sessionArg: "--session",
          sessionTilstand: "eksisterende"
          systemPromptArg: "--system",
          systemPromptWhen: "først"
          billedeArg: "--image",
          billedeTilstand: "gentag"
        },
      },
    },
  },
}
```

```json5
{
  agenter: {
    defaults: {
      modeller: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4. ": {
          alias: "GLM"
          params: {
            tænkning: {
              type: "aktiveret",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primær: "anthropic/claude-opus-4-6"
        tilbagefald: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3. -70b-instruct:free",
        ],
      },
      billedeModel: {
        primær: "openrouter/qwen/qwen-2. -vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2. -flash-vision:fri"],
      },
      tænkningStandard: "low",
      verboseStandard: "off",
      elevatedStandard: "on",
      timeoutSekunder: 600,
      mediaMaxMb: 5,
      hjerteslag: {
        hver: "30m",
        mål: "last",
      },
      maxConcurrent: 3,
      subagenter: {
        model: "minimax/MiniMax-M2. ",
        maxConcurrent: 1,
        arkivAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800
        oprydningsmidler: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (tool-result beskæring)

`agents.defaults.contextPruning` svesker **gamle værktøjsresultater** fra in-memory konteksten lige før en anmodning sendes til LLM.
Det ændrer **ikke** sessionshistorikken på disken (`*.jsonl` forbliver komplet).

Dette er beregnet til at reducere token brug for chatty agenter, der akkumulerer store værktøj udgange over tid.

Højt niveau:

- Rør aldrig bruger/assistent beskeder.
- Beskytter de sidste 'keepLastAssistants' assisterende beskeder (ingen værktøjsresultater efter dette punkt beskæres).
- Beskytter bootstrap præfiks (intet før den første brugerbesked beskæres).
- Tilstande:
  - `adaptive`: resultater af overdimensionerede værktøjer (hold hoved/hale), når det estimerede kontekstforhold krydser `softTrimRatio`.
    Så rydder hårdt de ældste kvalificerede værktøj resultater, når det anslåede kontekstforhold krydser `hardClearRatio` **og**
    der er nok prunable tool-result bulk (`minPrunableToolChars`).
  - `aggressive`: altid erstatter støtteberettigede værktøj resultater før cutoff med `hardClear.placeholder` (ingen forholdet kontroller).

Blød vs hård beskæring (hvilke ændringer i den sammenhæng, der sendes til LLM):

- **Soft-trim**: kun for _oversized_ værktøj resultater. Holder begyndelsen + ende og indsætter `...` i midten.
  - Før: `toolResult("…meget lang output…")`
  - Efter: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **Hard-clear**: erstatter hele værktøjets resultat med pladsholderen.
  - Før: `toolResult("…meget lang output…")`
  - Efter: `toolResultat ("[Gamle værktøj resultat indhold ryddet]")`

Noter / aktuelle begrænsninger:

- Værktøjsresultater der indeholder **billedblokke springes over** (aldrig trimmet/ryddet) lige nu.
- Det estimerede “kontekstforhold” er baseret på **karakterer** (omtrentlig), ikke eksakte tokens.
- Hvis sessionen ikke indeholder mindst `keepLastAssistants` assisterende beskeder endnu, springes beskæring over.
- I `aggressive` tilstand, `hardClear.enabled` ignoreres (støtteberettigede værktøj resultater er altid erstattet med `hardClear.placeholder`).

Standard (adaptive):

```json5
{
  agenter: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

At deaktivere:

```json5
{
  agenter: { defaults: { contextPruning: { mode: "off" } } },
}
```

Standarder (når `tilstand` er `"adaptive"` eller `"aggressive"`):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (kun adaptiv)
- `hardClearRatio`: `0,5` (kun adaptiv)
- `minPrunableToolChars`: `50000` (kun adaptiv)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (kun adaptiv)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Eksempel (aggressiv, minimal):

```json5
{
  agenter: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

Eksempel (adaptiv justeret):

```json5
{
  agenter: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0. ,
        hardClearRatio: 0. ,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { aktiveret: true, pladsholder: "[Gamle værktøj resultat indhold ryddet]" },
        // Valgfri: Begræns beskæring til specifikke værktøjer (benægte vindere; understøtter "*" wildcards)
        værktøjer: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

Se [/concepts/session-pruning](/concepts/session-pruning) for adfærdsdetaljer.

#### `agents.defaults.compaction` (reserve headroom + memory flush)

`agents.defaults.compaction.mode` vælger komprimeringsstrategien. Defaults to `default`; set `safeguard` to enable chunked summarization for very long histories. Se [/concepts/compaction](/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` håndhæver et minimum `reserveTokens`
værdi for Pi komprimering (standard: `20000`). Sæt den til `0` for at deaktivere gulvet.

`agents.defaults.compaction.memoryFlush` kører en \*\*lydløs \*\* agentisk tur før
auto-komprimering, instruere modellen til at gemme holdbare minder på disken (f.eks.
`memory/YYYY-MM-DD.md`). Det udløser, når sessionen token estimat krydser en
blød tærskel under komprimeringsgrænsen.

Ældre standard:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: indbyggede standardindstillinger med `NO_REPLY`
- Bemærk: hukommelse flush springes over, når sessionen arbejdsområde er skrivebeskyttet
  (`agents.defaults.sandbox.workspaceAccess: "ro"` eller `"ingen"`).

Eksempel (tunet):

```json5
{
  agenter: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000
        hukommelseFlush: {
          aktiveret: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nærmer sig komprimering. Gem holdbare minder nu.",
          prompt: "Skriv eventuelle varige noter til hukommelse/ÅÅÅÅ-MM-DD. d; svar med NO_REPLY hvis intet at gemme. ,
        },
      },
    },
  },
}
```

Blokér streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (standard fra).

- Kanal tilsidesættelser: `*.blockStreaming` (og per-konto varianter) for at tvinge blok streaming til/fra.
  Ikke-Telegram kanaler kræver en eksplicit `*.blockStreaming: true` for at aktivere blok svar.

- `agents.defaults.blockStreamingBreak`: `"text_end"` eller `"message_end"` (standard: text_end).

- `agents.defaults.blockStreamingChunk`: blød chunking for streamede blokke. Standard er
  800–1200 tegn, foretrækker afsnit pauser (`\n\n`), derefter newlines, derefter sætninger.
  Eksempel:

  ```json5
  {
    agenter: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: flette streamede blokke, før du sender.
  Defaults to `{ idleMs: 1000 }` and inherits `minChars` from `blockStreamingChunk`
  with `maxChars` capped to the channel text limit . Signal/Slack/Discord/Google Chat default
  til `minChars: 1500` medmindre tilsidesættes.
  Kanal tilsidesættelser: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  channels.googlechat.blockStreamingCoalesce\`
  (og per-konto variants).

- `agents.defaults.humanDelay`: randomiseret pause mellem **blokér svar** efter den første.
  Tilstande: `off` (standard), `natural` (800–2500ms), `custom` (brug `minMs`/`maxMs`).
  Per-agent tilsidesættelse: `agents.list[].humanDelay`.
  Eksempel:

  ```json5
  {
    agenter: { defaults: { humanForsinkelse: { mode: "natural" } } },
  }
  ```

  Se [/concepts/streaming](/concepts/streaming) for adfærd + chunking detaljer.

Typende indikatorer:

- `agents.defaults.typingMode`: `"aldrig", "instant", "tænkning", "message"`. Defaults to
  `instant` for direct chats / omtaler and `message` for unmentioned group chats.
- `session.typingMode`: per-session tilsidesætte for tilstanden.
- `agents.defaults.typingIntervalSeconds`: hvor ofte maskinskrivningssignalet opdateres (standard: 6s).
- `session.typingIntervalSeconds`: per-session tilsidesætte for opdateringsintervallet.
  Se [/concepts/typing-indicators](/concepts/typing-indicators) for adfærdsdetaljer.

`agents.defaults.model.primary` skal indstilles som `provider/model` (f.eks. `anthropic/claude-opus-4-6`).
Aliaser kommer fra `agents.defaults.models.*.alias` (fx `Opus`).
Hvis du udelader udbyderen, antager OpenClaw i øjeblikket `antropisk` som en midlertidig
udfasning fallback.
Z.AI modeller er tilgængelige som `zai/<model>` (fx `zai/glm-4.7`) og kræver
`ZAI_API_KEY` (eller arv `Z_AI_API_KEY`) i miljøet.

`agents.defaults.heartbeat` konfigurerer periodiske hjerteslag løb:

- `every`: varighed streng (`ms`, `s`, `m`, `h`); standard enhed minutter. Standard:
  `30m`. Sæt `0m` til deaktiveret.
- `model`: valgfri tilsidesættelse model for hjerteslag kører (`udbyder/model`).
- `includeReasoning`: når `true`, hjerteslag vil også levere den separate `Reasoning:` meddelelse, når den er tilgængelig (samme form som `/ræsonnement på`). Standard: `falsk`.
- `session`: valgfri session nøgle til at kontrollere, hvilken session hjerteslag kører i. Standard: `main`.
- `to`: valgfri modtager tilsidesættelse (kanal-specifik id, f.eks. E.164 for WhatsApp, chat-id for Telegram).
- `target`: valgfri leveringskanal (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Standard: `sidst`.
- `prompt`: valgfri tilsidesættelse for hjerteslag krop (standard: `Læs HEARTBEAT.md hvis det findes (arbejdsområde kontekst). Følg den nøje. Udsæt eller gentag ikke gamle opgaver fra tidligere chats. Hvis intet behøver opmærksomhed, besvar HEARTBEAT_OK.`). Overskrivninger sendes ordret, inkludere en `Læs HEARTBEAT.md` linje, hvis du stadig vil have filen læst.
- `ackMaxChars`: max tegn tilladt efter `HEARTBEAT_OK` før levering (standard: 300).

Per-agent heartbeats:

- Sæt `agents.list[].heartbeat` for at aktivere eller tilsidesætte hjerteslag indstillinger for en bestemt agent.
- Hvis indtastning af stoffer definerer 'hjerteslag', **kun disse midler** køre hjerteslag; standard
  bliver den fælles baseline for disse stoffer.

Hjertebanken kører fuld agent drejer. Kortere intervaller brænde flere tokens; vær opmærksom
af `hver`, hold `HEARTBEAT.md` lille, og/eller vælg en billigere `model`.

`tools.exec` konfigurerer baggrunds exec standard:

- `backgroundMs`: tid før auto-baggrund (ms, standard 10000)
- `timeoutSec`: auto-kill efter denne runtime (sekunder, standard 1800)
- `oprensninger`: hvor lang tid der skal holdes færdige sessioner i hukommelsen (ms, standard 1800000)
- `notifyOnExit`: kø en systembegivenhed + anmodning hjerteslag, når backgrounded exec afslutter (standard true)
- `applyPatch.enabled`: enable experimental `apply_patch` (OpenAI/OpenAI Codex only; default false)
- `applyPatch.allowModels`: valgfri tilladt liste over model-id'er (f.eks. `gpt-5.2` eller `openai/gpt-5.2`)
  Bemærk: `applyPatch` er kun under `tools.exec`.

`tools.web` konfigurerer websøgning + hent værktøjer:

- `tools.web.search.enabled` (standard: sand, når nøglen er til stede)
- `tools.web.search.apiKey` (anbefalet: sæt via `openclaw configure --section web`, eller brug `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults` (1–10, standard 5)
- `tools.web.search.timeoutSeconds` (standard 30)
- `tools.web.search.cacheTtlMinutes` (standard 15)
- `tools.web.fetch.enabled` (standard true)
- `tools.web.fetch.maxChars` (standard 50000)
- `tools.web.fetch.maxCharsCap` (standard 50000; klemmer maxChars fra config/tool opkald)
- `tools.web.fetch.timeoutSeconds` (standard 30)
- `tools.web.fetch.cacheTtlMinutes` (standard 15)
- `tools.web.fetch.userAgent` (valgfri override)
- `tools.web.fetch.readability` (standard true; deaktivere kun for at bruge grundlæggende HTML-oprydning)
- `tools.web.fetch.firecrawl.enabled` (standard sand, når en API-nøgle er sat)
- `tools.web.fetch.firecrawl.apiKey` (valgfri; standard `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (default [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (standard true)
- `tools.web.fetch.firecrawl.maxAgeMs` (valgfri)
- `tools.web.fetch.firecrawl.timeoutSeconds` (valgfri)

`tools.media` konfigurerer indgående medieforståelse (billede/audio/video):

- `tools.media.models`: liste over delte modeller (kapacitet-tagged; bruges efter per-cap lister).
- `tools.media.concurrency`: max samtidige funktioner kører (standard 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `aktiveret`: opt-out-kontakt (standard sand, når modeller er konfigureret).
  - `prompt`: valgfri prompt tilsidesættelse (billede / video tilføje et `maxChars` vink automatisk).
  - `maxChars`: max output tegn (standard 500 for billede/video; unset for lyd).
  - `maxBytes`: max mediestørrelse til at sende (standard: billede 10MB, lyd 20MB, video 50MB).
  - `timeoutSeconds`: anmodning timeout (standard: billede 60s, audio 60s, video 120s).
  - `language`: valgfrit lydvink.
  - `attachments`: attachment policy (`mode`, `maxAttachments`, `prefer`).
  - `scope`: valgfri gating (første match vinder) med `match.channel`, `match.chatType`, eller `match.keyPrefix`.
  - `models`: bestilt liste over modelindgange; fejl eller oversize medier falder tilbage til den næste indgang.
- Hver `model[]` indgang:
  - Indgang til udbyder ('type: "udbyder"' eller udeladt):
    - `provider`: API udbyder id (`openai`, `anthropic`, `google`/`gemini`, `groq`, etc).
    - `model`: model id tilsidesættelse (kræves for billede; standard er `gpt-4o-mini-transcribe`/`whisper-large v3-turbo` for lydudbydere, og `gemini-3-flash-preview` for video).
    - `profile` / `preferredProfile`: auth profile selection.
  - CLI post (`type: "cli"`):
    - `kommando`: eksekverbar til at køre.
    - `args`: skabelonerede args (understøtter `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, etc).
  - `capabilities `: valgfri liste (`image`, `audio`, `video`) til at gate en delt post. Standarder når udeladt: `openai`/`anthropic`/`minimax` → billede, `google` → image+audio+video, `groq` → lyd.
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` kan tilsidesættes pr. post.

Hvis ingen modeller er konfigureret (eller `aktiveret: falsk`), er forståelsen sprunget over; modellen modtager stadig de originale vedhæftede filer.

Udbyderen auth følger standard model auth order (auth profiles, env vars like `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, or `models.providers.*.apiKey`).

Eksempel:

```json5
{
  værktøjer: {
    media: {
      lyd: {
        aktiveret: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          regler: [{ handling: "allow", match: { chatType: "direkte" } }],
        },
        modeller: [
          { udbyder: "openai", model: "gpt-4o-mini-transcribe" },
          { typ: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        aktiveret: sandt,
        maxBytes: 52428800,
        modeller: [{ leverandør: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

`agents.defaults.subagents` konfigurerer underagent standard:

- `model`: standardmodel for opfostrede underagenter (streng eller `{ primary, fallbacks }`). Hvis udeladt, arver subagenter opkalderens model, medmindre den tilsidesættes pr. agent eller pr. opkald.
- `maxConcurrent`: max samtidige underagent kører (standard 1)
- `archiveAfterMinutes`: auto-archive sub-agent sessioner efter N minutter (standard 60; sæt `0` til deaktiveret)
- Per-subagent værktøjspolitik: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (benægte vinder)

`tools.profile` sætter et **base tool allowlist** før `tools.allow`/`tools.deny`:

- `minimal`: kun `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: ingen begrænsning (samme som ikke sat)

Per-agent tilsidesættelse: `agents.list[].tools.profile`.

Eksempel (kun messaging som standard, tillad også Slack- og Discord-værktøjer):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Eksempel (coding-profil, men afvis exec/process overalt):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` lader dig **yderligere begrænse** værktøjer for bestemte udbydere (eller en enkelt `udbyder/model`).
Per-agent tilsidesættelse: `agents.list[].tools.byProvider`.

Ordre: basisprofil → udbyderprofil → Tillad/benægte politikker.
Leverandørnøgler accepterer enten `provider` (f.eks. `google-antigravity`) eller `provider/model`
(f.eks. `openai/gpt-5.2`).

Eksempel (bevar global coding-profil, men minimale værktøjer for Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Eksempel (udbyder/model-specifik tilladelse):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` configure a global tool allow/deny policy (benægte vine).
Matchning er versalfølsom og understøtter `*` jokertegn (`"*"` betyder alle værktøjer).
Dette anvendes, selv når Docker-sandkassen er **slukket**.

Eksempel (deaktiver browser/lærred overalt):

```json5
{
  værktøjer: { benægt: ["browser", "canvas"] },
}
```

Værktøjsgrupper (shorthands) arbejde i **global** og **per-agent** værktøjspolitikker:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: alle indbyggede OpenClaw-værktøjer (ekskluderer udbyder-plugins)

`tools.elevated` kontrol forhøjet (vært) exec adgang:

- `aktiveret`: tillad forhøjet tilstand (standard sand)
- `allowFrom`: per-kanal tilladte lister (tom = deaktiveret)
  - `whatsapp`: E.164 numre
  - `telegram`: chat id'er eller brugernavne
  - `discord`: brugernavne eller brugernavne (falder tilbage til `channels.discord.dm.allowFrom` hvis udeladt)
  - `signal`: E.164 tal
  - `imessage`: håndtag/chat id
  - `webchat`: sessions-id eller brugernavne

Eksempel:

```json5
{
  værktøjer: {
    højde: {
      aktiveret: true,
      allowFra: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

Tilsidesæt peragent (yderligere begrænsning):

```json5
{
  agenter: {
    liste: [
      {
        id: "family",
        -værktøjer: {
          højde: { enabled: false },
        },
      },
    ],
  },
}
```

Noter:

- `tools.elevated` er den globale baseline. `agents.list[].tools.elevated` kan kun yderligere begrænse (begge skal tillade).
- `/forhøjet onřoffřy beder full` butikker stat per session nøgle; inline direktiver gælder for en enkelt besked.
- Forhøjet `exec` kører på værten og omgår sandboxing.
- Værktøjspolitik gælder stadig. Hvis `exec` nægtes, kan forhøjet ikke anvendes.

`agents.defaults.maxConcurrent` sætter det maksimale antal indlejrede agent kørsler, der kan
udføre parallelt på tværs af sessioner. Hver session er stadig serialiseret (en kør
pr. sessionsnøgle ad gangen). Standard: 1.

### `agents.defaults.sandbox`

Valgfri \*\* Docker sandboxing\*\* for den indlejrede agent. Beregnet til ikke-vigtigste
sessioner, så de ikke kan få adgang til dit værtssystem.

Detaljer: [Sandboxing](/gateway/sandboxing)

Standardindstillinger (hvis aktiveret):

- anvendelsesområde: "agent"\` (en beholder + arbejdsområde pr. agent)
- Debians bogorm-slanke billede
- Agent arbejdsrum adgang: `workspaceAccess: "none"` (standard)
  - `"ingen"`: brug et sandkasse-arbejdsområde pr. skop under `~/.openclaw/sandkasser`
- `"ro"`: behold arbejdsområdet i sandkassen på `/arbejdsområdet`, og monter agenten skrivebeskyttet ved `/agent` (deaktiverer `skriv `/`edit`/`apply_patch`)
  - `"rw"`: montere agenten arbejdsområde læst/skriv på `/workspace`
- auto-prune: idle > 24 t ELLER alder > 7 dage
- værktøjspolitik: Tillad kun `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (afvis gevinster)
  - konfigurere via `tools.sandbox.tools`, tilsidesætte per-agent via `agents.list[].tools.sandbox.tools`
  - værktøjsgruppe shorthands understøttet i sandkasse politik: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (se [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- valgfri sandboxed browser (Chromium + CDP, noVNC observatør)
- hærdende knapper: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Advarsel: `scope: "delt"` betyder en delt container og delt arbejdsområde. Ingen
cross-session isolation. Brug `scope: "session"` for per-session isolation.

Legacy: `perSession` understøttes stadig (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` kører **en gang** efter beholderen er oprettet (inde i beholderen via `sh -lc`).
For pakkeinstallationer skal du sørge for netværks egress, en skrivbar root FS, og en root-bruger.

```json5
{
  agenter: {
    defaults: {
      sandbox: {
        mode: "non-main", // sluk - ikke - hoved - alle
        rækkevidde: "agent" // sessionsmåde - agent shared (agent er standard)
        workspaceAccess: "none", // Ingen - ... ro - rw
        workspaceRoot: "~/. penclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-"
          workdir: "/workspace"
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp" "/run"],
          -netværk: "intet"
          bruger: "1000:1000"
          capDrop: ["ALL"],
          da: { LANG: "C. TF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq"
          // Per-agent tilsidesætte (multi-agent): agenter. ist[].sandbox.docker.
          pidsLimit: 256,
          hukommelse: "1g"
          memorySwap: "2g"
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp. son",
          apparmorProfile: "openclaw-sandbox"
          dns: ["1. .1.1", "8.8.8. "],
          ekstraværter: ["internal.service:10.0.0. "],
          bind: ["/var/run/docker.sock:/var/run/docker. ock", "/home/user/source:/source:rw"],
        },
        browser: {
          aktiveret: false,
          billede: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-"
          cdpPort: 9222,
          vncPort: 5900
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          tilladtControlUrls: ["http://10. .0.42:18791"],
          tilladtControlHosts: ["browser.lab.local", "10.0.0. 2"],
          tilladtControlPorts: [18791],
          autoStart: sand
          autoStartTimeout: 12000,
        },
        besked: {
          idleHours: 24, // 0 deaktiverer inaktiv beskæring
          maxAgeDays: 7, // 0 deaktiverer maks-age-beskæring
        },
      },
    },
  },
  værktøjer: {
    sandkasse: {
      værktøjer: {
        tillader: [
          "exec",
          "proces",
          "læst"
          "skriv",
          "edit"
          "apply_patch",
          "sessions_list"
          "sessions_history",
          "sessions_send"
          "sessions_spawn",
          "session_status"
        ],
        nægte: ["browser" "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

Byg standard sandkasse billede én gang med:

```bash
scripts/sandbox-setup.sh
```

Bemærk: sandkasse containere standard til `network: "none"`; sæt `agents.defaults.sandbox.docker.network`
til `"bridge"` (eller dit brugerdefinerede netværk), hvis agenten har brug for udgående adgang.

Bemærk: Indgående vedhæftede filer iscenesættes ind i det aktive arbejdsområde på 'media/inbound/\*'. Med `workspaceAccess: "rw"`, betyder det, at filer skrives ind i agentens arbejdsområde.

Bemærk: `docker.binds` monterer yderligere værtsmapper; globale og per-agent binds er flettet.

Byg det valgfrie browserbillede med:

```bash
scripts/sandbox-browser-setup.sh
```

Når `agents.defaults.sandbox.browser.enabled=true`, browseren værktøj bruger en sandkasse
Chromium instans (CDP). Hvis noVNC er aktiveret (standard når headless=false),
er noVNC URL injiceret i systemprompten, så agenten kan henvise til det.
Dette kræver ikke `browser.enabled` i hovedkonfigurationen; sandkassen kontrol
URL injiceres pr. session.

`agents.defaults.sandbox.browser.allowHostControl` (default: false) tillader
sandboxed sessions at eksplicit målrette **vært** browser control server
via browser-værktøjet (`target: "host"`). Efterlad dette, hvis du ønsker streng
sandkasse isolation.

Tilladslister til fjernbetjening:

- `tilladtControlUrls`: nøjagtige kontrol-URLer tilladt for `mål: "custom"`.
- `allowedControlHosts`: værtsnavne tilladt (kun værtsnavn ingen port).
- `allowedControlPorts`: ports tilladte (defaults: http=80, https=443).
  Standard: alle tilladte lister er ikke sat (ingen begrænsning). `allowHostControl` standard til false.

### `models` (brugerdefinerede udbydere + base URLs)

OpenClaw bruger **pi-coding-agent** modelkataloget. Du kan tilføje brugerdefinerede udbydere
(LiteLLM, lokale OpenAI-kompatible servere, Antropiske fuldmagter osv.) ved at skrive
`~/.openclaw/agents/<agentId>/agent/models.json` eller ved at definere det samme skema i din
OpenClaw config under `models.providers`.
Overblik over udbyderen + eksempler: [/concepts/model-providers](/concepts/model-providers).

Når `models.providers` er til stede, skriver OpenClaw, sammenfletter en `models.json` i
`~/.openclaw/agents/<agentId>/agent/` ved opstart:

- standard opførsel: **merge** (holder eksisterende udbydere, tilsidesættelser på navn)
- set `models.mode: "replace"` for at overskrive filindholdet

Vælg modellen via `agents.defaults.model.primary` (provider/model).

```json5
{
  agenter: {
    defaults: {
      model: { primary: "custom-proxy/llama-3. -8b" },
      -modeller: {
        "custom-proxy/llama-3. -8b": {},
      },
    },
  },
  modeller: {
    tilstand: "merge",
    Institutioner: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1"
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        modeller: [
          {
            id: "llama-3. -8b",
            : "Llama 3. 8B",
            ræsonnement: falsk,
            input: ["tekst"],
            omkostninger: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### OpenCode Zen (multi-model proxy)

OpenCode Zen er en multi-model gateway med per-model endepunkter. OpenClaw bruger
den indbyggede `opencode` udbyder fra pi-ai; sæt `OPENCODE_API_KEY` (eller
`OPENCODE_ZEN_API_KEY`) fra [https://opencode.ai/auth](https://opencode.ai/auth).

Noter:

- Model nægter at anvende 'opencode/&lt;modelId&gt;(eksempel: 'opencode/claude-opus-4-6').
- Hvis du aktiverer en tilladt via `agents.defaults.models`, tilføj hver model, du planlægger at bruge.
- Genvej: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agenter: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      modeller: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### Z.AI (GLM-4.7) — support for udbyder alias

Z.AI modeller er tilgængelige via den indbyggede `zai` udbyder. Sæt `ZAI_API_KEY`
i dit miljø og referer modellen efter udbyder/model.

Genvej: `openclaw onboard -- auth-choice zai-api-key`.

```json5
{
  agenter: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Noter:

- `z.ai/*` og `z-ai/*` accepteres aliaser og normaliseres til `zai/*`.
- Hvis `ZAI_API_KEY` mangler, vil anmodninger til `zai/*` mislykkes med en auth fejl på runtime.
- Eksempel fejl: `Ingen API-nøgle fundet for udbyder "zai".`
- Z.AI's generelle API-endepunkt er `https://api.z.ai/api/paas/v4`. GLM-kodning
  anmoder om brug det dedikerede kodningsendepunkt `https://api.z.ai/api/coding/paas/v4`.
  Den indbyggede `zai` udbyder bruger kodnings-endepunktet. Hvis du har brug for det generelle
  endepunkt, skal du definere en brugerdefineret udbyder i `models.providers` med base URL
  tilsidesætte (se afsnittet brugerdefinerede udbydere ovenfor).
- Brug en falsk pladsholder i docs/configs; aldrig begå virkelige API-nøgler.

### Moonshot AI (Kimi)

Brug Moonshot's OpenAI-kompatible endepunkt:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-... },
  agenter: {
    defaults: {
      model: { primary: "moonshot/kimi-k2. " },
      modeller: { "moonshot/kimi-k2. ": { alias: "Kimi K2. " } },
    },
  },
  modeller: {
    mode: "merge",
    Institutioner: {
      måneskot: {
        baseUrl: "https://api. oonshot. i/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        modeller: [
          {
            id: "kimi-k2. ",
            navn: "Kimi K2. ",
            ræsonnement: falsk,
            input: ["tekst"],
            omkostninger: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Noter:

- Sæt `MOONSHOT_API_KEY` i miljøet eller brug `openclaw onboard --auth-choice moonshot-api-key`.
- Model ref: `moonshot/kimi-k2.5`.
- For det kinesiske endepunkt, enten:
  - Kør `openclaw onboard --auth-choice moonshot-api-key-cn` (guiden vil sætte `https://api.moonshot.cn/v1`), eller
  - Sæt manuelt `baseUrl: "https://api.moonshot.cn/v1"` i `models.providers.moonshot`.

### Kimi Coding

Brug Moonshot AI's Kimi Coding endpoint (Antropisk kompatibel, indbygget udbyder):

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

Noter:

- Sæt `KIMI_API_KEY` i miljøet eller brug `openclaw onboard -- auth-choice kimi-code-api-key`.
- Model ref: `kimi-coding/k2p5`.

### Syntetisk (Antropisk-kompatibel)

Brug Synthetic's Antropiske kompatible endepunkt:

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

Noter:

- Sæt `SYNTHETIC_API_KEY` eller brug `openclaw onboard --auth-choice synthetic-api-key`.
- Model ref: `syntetisk/hf:MiniMaxAI/MiniMax-M2.1`.
- Base URL skal udelade `/v1` fordi den antropiske klient tilføjer det.

### Lokale modeller (LM Studio) — anbefalet opsætning

Se [/gateway/local-models](/gateway/local-models) for den aktuelle lokale vejledning. TL;DR: Kør MiniMax M2.1 via LM Studio svar API på alvorlig hardware; holde hostede modeller fusioneret til fallback.

### MiniMax M2.1

Brug MiniMax M2.1 direkte uden LM Studio:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2. " },
    modeller: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2. ": { alias: "Minimax" },
    },
  },
  modeller: {
    tilstand: "merge",
    Institutioner: {
      minimax: {
        baseUrl: "https://api. inimax. o/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "antropisk-beskeder",
        modeller: [
          {
            id: "MiniMax-M2. ",
            navn: "MiniMax M2. ",
            ræsonnement: falsk,
            input: ["tekst"],
            // Priser: Opdatering i modeller. søn hvis du har brug for nøjagtig omkostningssporing.
            omkostninger: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Noter:

- Sæt `MINIMAX_API_KEY` miljøvariabel eller brug `openclaw ombord --auth-choice minimax-api`.
- Tilgængelig model: `MiniMax-M2.1` (standard).
- Opdater priser i `models.json` hvis du har brug for nøjagtig omkostningssporing.

### Cerebraer (GLM 4. 6 / 4. 7)

Brug Cerebras via deres OpenAI-kompatible endepunkt:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-... },
  agenter: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4. ",
        fallbacks: ["cerebras/zai-glm-4. "],
      },
      modeller: {
        "cerebras/zai-glm-4. ": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4. (Cerebras)" },
      },
    },
  },
  modeller: {
    tilstand: "merge",
    Institutioner: {
      cerebras: {
        baseUrl: "https://api. erebras. i/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        modeller: [
          { id: "zai-glm-4. ", navn: "GLM 4. (Cerebras)" },
          { id: "zai-glm-4,6", navn: "GLM 4. (Cerebras)" },
        ],
      },
    },
  },
}
```

Noter:

- Brug `cerebras/zai-glm-4.7` til Cerebras; brug `zai/glm-4.7` til Z.AI direkte.
- Sæt `CEREBRAS_API_KEY` i miljøet eller config.

Noter:

- Understøttede API'er: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- Brug `authHeader: true` + `headers` for brugerdefinerede auth behov.
- Tilsidesæt agent config rod med `OPENCLAW_AGENT_DIR` (eller `PI_CODING_AGENT_DIR`)
  hvis du vil have `models.json` gemt andetsteds (standard: `~/.openclaw/agents/main/agent`).

### `session`

Styrer session scoping, reset politik, reset triggers, og hvor session butikken er skrevet.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Standard er allerede pr. agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // Du kan tilsidesætte med {agentId}-templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direkte chats samles til agent:<agentId>:<mainKey> (standard: "main").
    mainKey: "main",
    agentToAgent: {
      // Maks. ping-pong-svaromgange mellem anmoder/mål (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

Felter:

- `mainKey`: direct-chat bucket nøgle (standard: `"main"`). Nyttigt, når du ønsker at “omdøbe” den primære DM tråd uden at ændre `agentId`.
  - Sandbox note: `agents.defaults.sandbox.mode: "non-main"` bruger denne nøgle til at detektere hovedsessionen. Enhver sessionsnøgle der ikke matcher `mainKey` (grupper / kanaler) er sandboxed.
- `dmScope`: hvordan DM sessioner er grupperet (standard: `"main"`).
  - `main`: alle DMs deler hovedsessionen for kontinuitet.
  - `per-peer`: isolere DMs af afsender-id på tværs af kanaler.
  - `per-kanal-peer`: isolere DMs pr. kanal + afsender (anbefales til multi-bruger indbakker).
  - `per-account-channel-peer`: isolere DMs pr. konto + kanal + afsender (anbefales til multi-konto indbakker).
  - Sikker DM-tilstand (anbefalet): sæt `session.dmScope: "per-channel-peer"` når flere personer kan DM botten (delte indbakker, multi-person tillader, eller `dmPolicy: "open"`).
- `identityLinks`: kort kanoniske id'er til udbyder-prefixed peers, så den samme person deler en DM-session på tværs af kanaler, når du bruger `per-peer`, `per-channel-peer`, eller `per-account-channel-peer`.
  - Eksempel: \`alice: ["telegram:123456789", "discord:987654321012345678"]«.
- `reset`: primær nulstillingspolitik. Defaults to daily resets at 4:00 AM lokal tid på gateway værten.
  - `mode`: `daily` eller `idle` (standard: `daily` når `reset` er til stede).
  - `atTime`: lokal time (0-23) for den daglige nulstillingsgrænse.
  - `idleMinutes`: glidende tomgangsvindue på få minutter. Når dagligt + inaktiv begge er konfigureret, alt efter hvad der udløber første gevinst.
- `resetByType`: pr.-session overrides for `direct`, `group` og `thread`. Legacy-`dm`-nøglen accepteres som et alias for `direct`.
  - Hvis du kun indstille arven `session.idleMinutes` uden nogen `reset`/`resetByType`, OpenClaw forbliver i tomgangstilstand for bagudkompatibilitet.
- `heartbeatIdleMinutes`: valgfri tomgang for hjerteslag checks (daglig nulstilling gælder stadig, når aktiveret).
- `agentToAgent.maxPingPongTurns`: max reply-back sving mellem requester/target (0–5, standard 5).
- `sendPolicy.default`: `allow` eller `deny` fallback når ingen regel matcher.
- `sendPolicy.rules[]`: match by `channel`, `chatType` (`directţgroupţroom`), eller `keyPrefix` (f.eks `cron:`). Første benægter vinder; ellers tillades.

### `færdigheder` (færdigheder config)

Kontroller bundtet tilladelse, installere præferencer, ekstra færdighed mapper, og per-færdighed
tilsidesættelser. Gælder for **bundtede** færdigheder og `~/.openclaw/skills` (arbejdsområde færdigheder
vinder stadig på navn konflikter).

Felter:

- `allowBundled`: valgfri tilladt liste for \*\*bundtede \*\* færdigheder. Hvis angivet, kun disse
  bundtede færdigheder er kvalificerede (administrerede/arbejdsområde færdigheder upåvirket).
- `load.extraDirs`: ekstra Skill-mapper, der skal scannes (laveste præcedens).
- `install.preferBrew`: foretræk brew-installatører, når de er tilgængelige (standard: true).
- `install.nodeManager`: node installeringsprogram præference (`npm` ● `pnpm` ● `yarn`, default: npm).
- `poster.<skillKey>`: per-skill config overrides.

Per-færdigheds felter:

- `enabled`: sæt `false` for at deaktivere en Skill, selv hvis den er bundtet/installeret.
- `env`: miljøvariabler, der injiceres til agent-kørslen (kun hvis de ikke allerede er sat).
- `apiKey`: valgfri bekvemmelighed for færdigheder, der erklærer en primær env var (fx `nano-banana-pro` → `GEMINI_API_KEY`).

Eksempel:

```json5
{
  færdigheder: {
    tilladt: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projekter/oss/some-skill-pack/skills"],
    },
    installation: {
      preferBrew: true,
      nodeManager: "npm",
    },
    indgange: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        da: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins` (udvidelser)

Kontroller plugin opdagelse, tillad / nægt, og per-plugin konfiguration. Plugins er indlæst
fra `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus alle
`plugins.load.paths` poster. **Konfigurationsændringer kræver en genstart af gateway.**
Se [/plugin](/tools/plugin) for fuldt forbrug.

Felter:

- `aktiveret`: master toggle for plugin indlæsning (standard: true).
- `allow`: valgfri tilladt liste over plugin ids; når angivet, kun listet plugins belastning.
- `deny`: valgfri benægtelse af plugin id'er (benægte vinder).
- `load.paths`: ekstra plugin-filer eller mapper der skal indlæses (absolut eller `~`).
- `poster.<pluginId>`: per-plugin overskrivninger.
  - `aktiveret`: sæt `false` til at deaktivere.
  - `config`: plugin-specifikt config objekt (valideret af plugin'et, hvis angivet).

Eksempel:

```json5
{
  plugins: {
    aktiveret: true,
    tillad: ["voice-call"],
    indlæs: {
      stier: ["~/Projects/oss/voice-call-extension"],
    },
    indgange: {
      "voice-call": {
        aktiveret: true,
        config: {
          udbyder: "twilio"
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw kan starte en \*\* dedikeret, isoleret\*\* Chrome/Brave/Edge/Chromium eksempel for openclaw og udsætte en lille loopback kontroltjeneste.
Profiler kan pege på en **fjern** Chrom-baseret browser via `profiler.<name>.cdpUrl`. Eksterne
profiler er kun vedhæftede (start/stop/reset er deaktiveret).

`browser.cdpUrl` rester for ældre single-profile configs og som base
ordning/vært for profiler, der kun angiver `cdpPort`.

Standardindstillinger:

- aktiveret: `sand`
- evaluateEnabled: `true` (sæt `false` for at deaktivere `act:evaluate` og `wait --fn`)
- kontroltjeneste: kun loopback (port afledt af `gateway.port`, standard `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profil)
- profilfarve: `#FF4500` (hummer-orange)
- Bemærk: Kontrol-serveren er startet af den kørende gateway (OpenClaw.app menubar, eller `openclaw gateway`).
- Auto-registrere rækkefølge: standard browser, hvis Chrom-baserede; ellers Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    aktiveret: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127. .0. :18792", // legacy single-profile tilsidesætte
    standardProfil: "chrome"
    -profiler: {
      openclaw: { cdpPort: 18800, farve: "#FF4500" },
      arbejde: { cdpPort: 18801, farve: "#0066CC" },
      fjern: { cdpUrl: "http://10. .0.42:9222", farve: "#00AA00" },
    },
    farve: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser. pp/Contents/MacOS/Brave Browser",
    // attachOnly: false, // sæt sand, når tunneling af en ekstern CDP til localhost
  },
}
```

### `ui` (Udseende)

Valgfri accent farve, der anvendes af de indfødte apps til UI krom (f.eks. Talk Mode boble tint).

Hvis frakoblet klienter falder tilbage til en dæmpet lyseblå.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB eller #RRGGBB)
    // Valgfri: Kontrol UI assistent identitet tilsidesættelse.
    // Hvis deaktiveret, bruger kontrolbrugergrænsefladen den aktive agent identitet (config eller IDENTITY. d).
    assistent: {
      navn: "OpenClaw"
      avatar: "CB" // emoji, kort tekst, eller billede URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Brug `gateway.mode` til udtrykkeligt at erklære, om denne maskine skal køre Gateway.

Standardindstillinger:

- **unset** (behandles som “start ikke”)
- bind: `loopback`
- port: `18789` (enkelt port for WS + HTTP)

```json5
{
  gateway: {
    -tilstand: "local", // eller "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { aktiveret: true, basePath: "/openclaw" }
    // auth: { mode: "token" token: "your- token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" ¤ "serve" ¤ "funnel" }
  },
}
```

Kontrol UI base sti:

- `gateway.controlUi.basePath` indstiller URL-præfikset hvor kontrol-UI serveres.
- Eksempler: `"/ui"`, `"/openclaw"`, \`"/apps/openclaw"«.
- Standard: rod (`/`) (uændret).
- `gateway.controlUi.root` sætter filsystemroden for Control UI aktiver (standard: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` tillader token-only auth for Control UI, når
  enhed identitet er udeladt (typisk over HTTP). Standard: `falsk`. Foretræk HTTPS
  (Tailscale Serve) eller `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` deaktiverer enhedsidentitetstjek for
  Control UI (kun token/password). Standard: `falsk`. Kun brudglas.

Relaterede docs:

- [Control UI](/web/control-ui)
- [Weboversigt](/web)
- [Tailscale](/gateway/tailscale)
- [Fjernadgang](/gateway/remote)

Betroede proxies:

- `gateway.trustedProxies`: liste over reverse proxy IP'er, der afslutter TLS foran Gateway.
- Når en forbindelse kommer fra en af disse IP'er, OpenClaw bruger 'x-forwarded-for' (eller 'x-real-ip') til at bestemme klientens IP til lokal parringskontrol og HTTP auth/lokal kontrol.
- Kun liste fuldgyldige fuldmagter og sikre, at de **overskriver** indkommende `x-forwarded-for`.

Noter:

- `openclaw gateway` nægter at starte, medmindre `gateway.mode` er sat til `local` (eller du passerer overskrivningen flag).
- `gateway.port` styrer den enkelt multiplexed port der bruges til WebSocket + HTTP (kontrol UI, kroge, A2UI).
- OpenAI Chat Completions endepunkt: **deaktiveret som standard**; aktiver med `gateway.http.endpoints.chatCompletions.enabled: true`.
- Præsentation: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > standard `18789`.
- Gateway auth kræves som standard (token / adgangskode eller Tailscale Serve identitet). Non-loopback binds kræver en delt token/password.
- Onboarding guiden genererer som standard en gateway token (selv på loopback).
- `gateway.remote.token` er **kun** for eksterne CLI opkald; det aktiverer ikke lokal gateway auth. `gateway.token` ignoreres.

Auth og haleskala:

- `gateway.auth.mode` angiver kravene til håndtryk (`token` eller `password`). Når det ikke er angivet, antages token auth.
- `gateway.auth.token` gemmer den delte token for token auth (bruges af CLI på samme maskine).
- Når `gateway.auth.mode` er indstillet, er kun denne metode accepteret (plus valgfri Tailscale headers).
- `gateway.auth.password` kan indstilles her, eller via `OPENCLAW_GATEWAY_PASSWORD` (anbefales).
- `gateway.auth.allowTailscale` tillader Tailscale Serve identitet headers
  (`tailscale-user-login`) at tilfredsstille auth når anmodningen ankommer på loopback
  med `x-forwarded-for`, `x-forwarded-proto`, og `x-forwarded-host`. OpenClaw
  verificerer identiteten ved at løse 'x-forwarded-for'-adressen via
  'tailscale whois', før du accepterer den. Når `true`, Serveres anmodninger behøver ikke
  en token/password; sæt `false` for at kræve eksplicitte legitimationsoplysninger. Standard er
  `true` når `tailscale.mode = "serve"` og auth mode er ikke `password`.
- `gateway.tailscale.mode: "serve"` bruger Tailscale Server (kun halenet, loopback bind).
- `gateway.tailscale.mode: "tragt"` udsætter instrumentbrættet offentligt; kræver auth.
- `gateway.tailscale.resetOnExit` nulstiller Serve/Tragt config på nedlukning.

Ekstern klient standard (CLI):

- `gateway.remote.url` sætter standard Gateway WebSocket URL til CLI opkald, når `gateway.mode = "remote"`.
- `gateway.remote.transport` vælger macOS fjerntransport (`ssh` default, `direct` for ws/wss). Når `direct`, `gateway.remote.url` skal være `ws://` eller `wss://`. `ws://host` standard port `18789`.
- `gateway.remote.token` leverer token til fjernopkald (lad være fravalgt for ingen auth).
- `gateway.remote.password` leverer adgangskoden til fjernopkald (lad være fravalgt for ingen auth).

macOS app opførsel:

- OpenClaw.app ure `~/.openclaw/openclaw.json` og skifter tilstande live når `gateway.mode` eller `gateway.remote.url` ændringer.
- Hvis `gateway.mode` ikke er angivet, men `gateway.remote.url` er angivet, behandler macOS appen den som fjerntilstand.
- Når du ændrer forbindelsestilstand i macOS-appen, skriver den `gateway.mode` (og `gateway.remote.url` + `gateway.remote.transport` i fjerntilstand) tilbage til konfigurationsfilen.

```json5
{
  gateway: {
    tilstand: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Direkte transporteksempel (macOS app):

```json5
{
  gateway: {
    tilstand: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (Config hot reload)

Gateway ure `~/.openclaw/openclaw.json` (eller `OPENCLAW_CONFIG_PATH`) og anvender ændringer automatisk.

Tilstande:

- `hybrid` (standard): hot-apply safe changes; genstart Gateway for kritiske ændringer.
- `hot`: Anvend kun hot-safe ændringer; log når genstart er påkrævet.
- `restart`: genstart Gateway ved enhver konfigurationsændring.
- `off`: Deaktivér genindlæsning af genladning.

```json5
{
  gateway: {
    reload: {
      tilstand: "hybrid",
      debounceMs: 300,
    },
  },
}
```

#### Hot reload matrix (filer + impact)

Filer set:

- `~/.openclaw/openclaw.json` (eller `OPENCLAW_CONFIG_PATH`)

Varmtilført (ingen fuld genstart af gateway):

- `hooks` (webhook auth/path/mappings) + `hooks.gmail` (Gmail watcher genstartet)
- `browser` (genstart af browserkontrolserver)
- `cron` (cron service genstart + concurrency opdatering)
- `agents.defaults.heartbeat` (hjerteslag løberen genstart)
- `web` (WhatsApp web-kanal genstart)
- `telegram`, `discord`, `signal`, `imessage` (kanal genstarter)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `guide` (dynamiske læser)

Kræver fuld Gateway genstart:

- `gateway` (port/bind/auth/control UI/tailscale)
- `bridge` (legacy)
- `discovery`
- `canvasHost`
- `plugins`
- Enhver ukendt/ikke-understøttet konfigurationssti (standard genstart for sikkerhed)

### Multi-instans isolering

For at køre flere gateways på en vært (for redundans eller en redning bot), isolere per instans tilstand + config og bruge unikke havne:

- `OPENCLAW_CONFIG_PATH` (pr. instans config)
- `OPENCLAW_STATE_DIR` (session/creds)
- `agents.defaults.workspace` (memories)
- `gateway.port` (unik pr. eksempel)

Bekvemmelighedsflag (CLI):

- `openclaw --dev …` → bruger `~/.openclaw-dev` + skift porte fra base `19001`
- `openclaw --profile <name> …` → uses `~/.openclaw-<name>` (port via config/env/flags)

Se [Gateway runbook](/gateway) for den afledte port mapping (gateway/browser/canvas).
Se [Flere gateways](/gateway/multiple-gateways) for browser/CDP port isolation detaljer.

Eksempel:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (Gateway webhooks)

Aktiver et simpelt HTTP webhook endpoint på Gateway HTTP serveren.

Standardindstillinger:

- aktiveret: `falsk`
- sti: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    aktiveret: true,
    token: "shared-secret",
    -sti: "/hooks",
    -forudindstillinger: ["gmail"],
    -transformsDir: "~/. penclaw/hooks",
    kortlægninger: [
      {
        match: { path: "gmail" },
        handling: "agent"
        wakeMode: "now",
        navn: "Gmail"
        sessionKey: "hook:gmail:{{messages[0].id}}",
        beskedskabelon: "Fra: {{messages[0].from}}\nOm: {{messages[0].subject}}\n{{messages[0].snippet}}"
        levering: sand,
        kanal: "sidste"
        model: "openai/gpt-5. -mini",
      },
    ],
  },
}
```

Anmodninger skal indeholde krogtoken:

- `Autorisation: Bearer <token>` **eller**
- `x-openclaw-token: <token>`

Endepunkter:

- `POST /hooks/wake` → `{ text , mode?: "now"¤ "next-heartbeat" }`
- `POST /hooks/agent` → `{ besked, navn?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
- `POST /hooks/<name>` → løst via `hooks.mappings`

`/hooks/agent` altid sende et resumé i hovedsessionen (og kan eventuelt udløse et øjeblikkeligt hjerteslag via `wakeMode: "now"`).

Kortlægning noter:

- `match.path` matcher understien efter `/hooks` (f.eks. `/hooks/gmail` → `gmail`).
- `match.source` matcher et nyttelast felt (f.eks. `{ kilde: "gmail" }`), så du kan bruge en generisk `/hooks/ingest` sti.
- Skabeloner som `{{messages[0].subject}}` læst fra nyttelasten.
- `transform` kan pege på et JS/TS modul, der returnerer en krog handling.
- `deliver: true` sender det endelige svar til en kanal; `kanal` standard er `last` (falder tilbage til WhatsApp).
- Hvis der ikke er nogen forudgående leveringsrute, sæt `kanal` + `til` eksplicit (kræves for Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams).
- `model` tilsidesætter LLM for denne hook run (`provider/model` eller alias; skal være tilladt, hvis `agents.defaults.models` er sat).

Gmail helper config (bruges af `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      konto: "openclaw@gmail. om",
      -emne: "projects/<project-id>/topics/gog-gmail-watch",
      -abonnement: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127. .0.1:18789/hooks/gmail",
      inkludererKrop: sandt,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      tjener: { bind: "127. .0. ", port: 8788, sti: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Valgfri: brug en billigere model til Gmail-krog behandling
      // Falls tilbage til agenter. efaults.model. allbacks, derefter primær, på auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3. -70b-instruct:free",
      // Valgfri: standard tankegang niveau for Gmail hooks
      tænkning: "off"
    },
  },
}
```

Tilsidesæt model til Gmail hooks:

- `hooks.gmail.model` angiver en model til brug for Gmail hook behandling (standard session primær).
- Accepterer `provider/model` refs eller aliaser fra `agents.defaults.models`.
- Falder tilbage til `agents.defaults.model.fallbacks`, derefter `agents.defaults.model.primary`, på auth/rate-limit/timeouts.
- If `agents.defaults.models` is set, include hooks model in the allowlist.
- Ved opstart, advarer om den konfigurerede model ikke er i modelkataloget eller tilladelseslisten.
- `hooks.gmail.thinking` sætter standard tænkning niveau for Gmail hooks og er tilsidesat af per-hook `thinking`.

Gateway auto-start:

- Hvis `hooks.enabled=true` og `hooks.gmail.account` er indstillet, Gateway starter
  `gog gmail watch serve` ved boot og auto-fornyelse af uret.
- Sæt `OPENCLAW_SKIP_GMAIL_WATCHER=1` for at deaktivere auto-start (til manuel kørsel).
- Undgå at køre en separat `gog gmail ur serve` sammen med Gateway; det vil
  mislykkes med `lytte tcp 127.0.0.1:8788: bind: adresse, der allerede er i brug`.

Bemærk: når `tailscale.mode` er tændt, OpenClaw standard `serve.path` til `/` så
Tailscale kan proxy `/gmail-pubsub` korrekt (det striber set-path præfikset).
Hvis du har brug for backend til at modtage den præfikserede sti, sæt
`hooks.gmail.tailscale.target` til en fuld URL (og indstil `serve.path`).

### `canvasHost` (LAN/tailnet Canvas filserver + live reload)

Gateway tjener en mappe af HTML/CSS/JS over HTTP, så iOS/Android noder kan simpelthen `canvas.navigate` til det.

Standard root: `~/. penclaw/workspace/canvas`  
Standard port: `18793` (valgt for at undgå openclaw browser CDP port `18792`)  
Serveren lytter på **gateway bind vært** (LAN eller Tailnet) så knuder kan nå det.

Serveren:

- serverer filer under `canvasHost.root`
- injicerer en lille live-reload klient i serveret HTML
- overvåger mappen og udsendelserne genindlæses over et WebSocket endpoint på `/__openclaw__/ws`
- auto-opretter en starter `index.html` når mappen er tom (så du ser noget med det samme)
- tjener også A2UI på `/__openclaw__/a2ui/` og annonceres til noder som `canvasHostUrl`
  (altid bruges af noder til Canvas/A2UI)

Deaktivér levende genindlæsning (og fil kigger), hvis mappen er stor, eller du trykker på `EMFILE`:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

Ændringer til `canvasHost.*` kræver en genstart af gateway (konfiguration genindlæsning vil genstarte).

Deaktivér med:

- config: `canvasHost: { enabled: false }`
- Env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (ældre TCP-bro, fjernet)

Aktuelle builds indeholder ikke længere TCP-broens lytter; `bridge.*` config nøgler ignoreres.
Knuder forbinde over Gateway WebSocket. Dette afsnit opbevares til historisk reference.

Ældre adfærd:

- Gateway kunne udsætte en simpel TCP bro for knudepunkter (iOS/Android), typisk på port `18790`.

Standardindstillinger:

- aktiveret: `sand`
- port: \`18790«
- bind: `lan` (bindes til `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (kan nås på enhver grænseflade, herunder LAN/Wi‐Fi og Tailscale)
- `tailnet`: bindes kun til maskinens Tailscale IP (anbefales til Wien,London)
- `loopback`: `127.0.0.1` (lokalt)
- `auto`: foretrække tailnet IP hvis til stede, ellers `lan`

TLS:

- `bridge.tls.enabled`: Aktiver TLS for broforbindelser (TLS-kun, når aktiveret).
- `bridge.tls.autoGenerate`: generere et selvsigneret cert når ingen cert/key er til stede (standard: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM stier til brocertifikatet + privat nøgle.
- `bridge.tls.caPath`: valgfri PEM CA bundle (brugerdefinerede rødder eller fremtidige mTLS).

Når TLS er aktiveret, reklamerer Gateway `bridgeTls=1` og `bridgeTlsSha256` i opdagelsen TXT
registrerer, så knudepunkter kan fastgøre certifikatet. Manuelle forbindelser bruger tillid til første-brug, hvis der endnu ikke gemmes
-fingeraftryk.
Auto-genererede certs kræver `openssl` på PATH; hvis generering mislykkes, vil broen ikke starte.

```json5
{
  bro: {
    aktiveret: true,
    port: 18790,
    bind: "tailnet"
    tls: {
      aktiveret: true,
      // Anvendelser ~ /. penclaw/bridge/tls/bridge-{cert,key}. em when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/. penclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS broadcast-tilstand)

Kontrollerer LAN mDNS-opdagelse udsendelser (`_openclaw-gw._tcp`).

- `minimal` (standard): omit `cliPath` + `sshPort` fra TXT-poster
- `full`: omfatter `cliPath` + `sshPort` i TXT-optegnelser
- `off`: deaktiver kun mDNS-udsendelser
- Værtsnavn: Standard til `openclaw` (reklamerer `openclaw.local`). Tilsidesæt med `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  opdagelse: { mdns: { mode: "minimal" } },
}
```

### `opdagelse.wideArea` (Wide-Area Bonjour / unicast DNS-SD)

Når aktiveret, skriver Gateway en unicast DNS-SD zone for `_openclaw-gw._tcp` under `~/.openclaw/dns/` ved hjælp af den konfigurerede opdagelse domæne (eksempel: `openclaw.internal.`).

For at få iOS/Android til at opdage på tværs af netværk (Wien,London), parre dette med:

- en DNS-server på gateway vært betjener dit valgte domæne (CoreDNS anbefales)
- Skræddersy **split DNS**, så klienter løser det domæne via gateway DNS-serveren

Engangs setup hjælper (gateway host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Variabler for mediemodel

Skabelon pladsholdere er udvidet i `tools.media.*.models[].args` og `tools.media.models[].args` (og eventuelle fremtidige skabelonerede argumentfelter).

\| Variable           | Beskrivelse                                                                    |
\| ------------------ | ------------------------------------------------------------------------------ | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
\| `{{Body}}`         | Fuldt indgående beskedindhold                                                   |
\| `{{RawBody}}`      | Rå indgående beskedindhold (ingen historik/afsender-wrappere; bedst til kommandoparsning) |
\| `{{BodyStripped}}` | Indhold med gruppenævnelser fjernet (bedste standard for agenter)              |
\| `{{From}}`         | Afsender-id (E.164 for WhatsApp; kan variere pr. kanal)                         |
\| `{{To}}`           | Destinations-id                                                                 |
\| `{{MessageSid}}`   | Kanalbesked-id (når tilgængelig)                                                |
\| `{{SessionId}}`    | Aktuel sessions-UUID                                                           |
\| `{{IsNewSession}}` | `"true"` når en ny session blev oprettet                                      |
\| `{{MediaUrl}}`     | Indgående medie pseudo-URL (hvis til stede)                                    |
\| `{{MediaPath}}`    | Lokal mediesti (hvis downloadet)                                               |
\| `{{MediaType}}`    | Medietype (billede/lyd/dokument/…)                                             |
\| `{{Transcript}}`   | Lydtransskription (når aktiveret)                                               |
\| `{{Prompt}}`       | Opløst medieprompt for CLI-indgange                                            |
\| `{{MaxChars}}`     | Opløst maks. antal outputtegn for CLI-indgange                                 |
\| `{{ChatType}}`     | `"direct"` eller `"group"`                                                  |
\| `{{GroupSubject}}` | Gruppeemne (bedste bud)                                                         |
\| `{{GroupMembers}}` | Forhåndsvisning af gruppemedlemmer (bedste bud)                                |
\| `{{SenderName}}`   | Afsenders visningsnavn (bedste bud)                                             |
\| `{{SenderE164}}`   | Afsenders telefonnummer (bedste bud)                                           |
\| `{{Provider}}`     | Udbyder-hint (whatsapp | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (Gateway scheduler)

Cron er en Gateway-ejet scheduler for wakeups og planlagte job. Se [Cron job](/automation/cron-jobs) for funktionen oversigt og CLI eksempler.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_Næste: [Agent Runtime](/concepts/agent)_ 🦞
