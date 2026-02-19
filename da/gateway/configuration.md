---
summary: "Alle konfigurationsmuligheder for ~/.openclaw/openclaw.json med eksempler"
read_when:
  - Tilføjelse eller ændring af konfigurationsfelter
  - Leder du efter almindelige konfigurationsmønstre
  - Navigerer til specifikke konfigurationssektioner
title: "Konfiguration"
---

# Konfiguration 🔧

OpenClaw læser en valgfri **JSON5**-konfiguration fra `~/.openclaw/openclaw.json` (kommentarer + afsluttende kommaer er tilladt).

Hvis filen mangler, bruger OpenClaw sikre standardindstillinger. Almindelige grunde til at tilføje en konfiguration:

- begrænse hvem der kan trigge botten (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, osv.)
- styre gruppetilladelseslister + mention-adfærd (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- tilpasse beskedpræfikser (`messages`)

Se den [fulde reference](/gateway/configuration-reference) for alle tilgængelige felter.

<Tip>
**Ny i konfiguration?** Start med `openclaw onboard` for interaktiv opsætning, eller se guiden [Configuration Examples](/gateway/configuration-examples) for komplette copy-paste-konfigurationer.
</Tip>

## Minimal konfiguration

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Redigering af konfiguration

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # fuld opsætningsguide
    openclaw configure     # konfigurationsguide
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    Åbn [http://127.0.0.1:18789](http://127.0.0.1:18789) og brug fanen **Config**.
    Control UI gengiver en formular ud fra konfigurationsskemaet med en **Raw JSON**-editor som nødudgang.
  
</Tab>
  <Tab title="Direct edit">
    Redigér `~/.openclaw/openclaw.json` direkte. Gateway overvåger filen og anvender ændringer automatisk (se [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Skema + UI-hints

<Warning>
OpenClaw accepterer kun konfigurationer, der fuldt ud matcher ordningen. Ukendte nøgler, ugyldige typer eller ugyldige værdier får Gateway til at **nægte at starte**. Den eneste undtagelse på rodniveau er `$schema` (string), så editorer kan tilknytte JSON Schema-metadata.
</Warning>

Kanal-plugins og udvidelser kan registrere skema + UI-hints for deres konfiguration, så kanalindstillinger
forbliver skemadrevne på tværs af apps uden hardcodede formularer.

- Gateway starter ikke
- Kun diagnostiske kommandoer virker (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Kør `openclaw doctor` for at se de præcise problemer
- Kør `openclaw doctor --fix` (eller `--yes`) for at anvende rettelser

## Anvend + genstart (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Hver kanal har sin egen konfigurationssektion under `channels.<provider>`. Se den dedikerede kanalside for opsætningstrin:

    ````
    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`
    
    Alle kanaler deler det samme DM-politikmønster:
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // kun for allowlist/open
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    Angiv den primære model og valgfrie fallback-modeller:

    ````
    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```
    
    - `agents.defaults.models` definerer modelkataloget og fungerer som allowlist for `/model`.
    - Modelreferencer bruger formatet `provider/model` (f.eks. `anthropic/claude-opus-4-6`).
    - Se [Models CLI](/concepts/models) for at skifte model i chat og [Model Failover](/concepts/model-failover) for auth-rotation og fallback-adfærd.
    - For brugerdefinerede/selvhostede providere, se [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) i referencen.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM-adgang styres pr. kanal via `dmPolicy`:

    ```
    - `"pairing"` (standard): ukendte afsendere får en engangskode til godkendelse
    - `"allowlist"`: kun afsendere i `allowFrom` (eller den parrede allow-liste)
    - `"open"`: tillad alle indgående DMs (kræver `allowFrom: ["*"]`)
    - `"disabled"`: ignorer alle DMs
    
    For grupper bruges `groupPolicy` + `groupAllowFrom` eller kanalspecifikke allowlister.
    
    Se den [fulde reference](/gateway/configuration-reference#dm-and-group-access) for detaljer pr. kanal.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Gruppemeddelelser kræver som standard **omtale**. Konfigurer mønstre pr. agent:

    ````
    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
    
    - **Metadata-omtaler**: native @-omtaler (WhatsApp tryk-for-at-omtale, Telegram @bot osv.)
    - **Tekstmønstre**: regex-mønstre i `mentionPatterns`
    - Se [fuld reference](/gateway/configuration-reference#group-chat-mention-gating) for kanal-specifikke overrides og self-chat-tilstand.
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Sessioner styrer samtalekontinuitet og isolering:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // anbefalet til flerbruger
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (delt) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Se [Session Management](/concepts/session) for scope, identitetslinks og afsendelsespolitik.
    - Se [fuld reference](/gateway/configuration-reference#session) for alle felter.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Kør agent-sessioner i isolerede Docker-containere:

    ```
    scripts/sandbox-setup.sh
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    ```
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

  
</Accordion>

  <Accordion title="Configure cron jobs">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    ```
    Se [Cron job](/automation/cron-jobs) for funktionen oversigt og CLI eksempler.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">
    Aktivér HTTP webhook-endpoints på Gateway:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">
    Kør flere isolerede agenter med separate workspaces og sessioner:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">
    Brug `$include` til at organisere store konfigurationer:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Hot reload af konfiguration

Gateway overvåger `~/.openclaw/openclaw.json` og anvender ændringer automatisk — ingen manuel genstart nødvendig for de fleste indstillinger.

### Fejlhåndtering

| Tilstand                                   | Adfærd                                                                                                                                           |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`hybrid`** (standard) | Anvender sikre ændringer med det samme. Genstarter automatisk ved kritiske ændringer.                            |
| **`hot`**                                  | Anvender kun sikre ændringer med det samme. Logger en advarsel, når en genstart er nødvendig — du håndterer den. |
| **`restart`**                              | Genstarter Gateway ved enhver konfigurationsændring, sikker eller ej.                                                            |
| **`off`**                                  | Deaktiverer filovervågning. Ændringer træder i kraft ved næste manuelle genstart.                                |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Hvad anvendes med det samme, og hvad kræver en genstart

De fleste felter anvendes med det samme uden nedetid. I `hybrid`-tilstand håndteres ændringer, der kræver genstart, automatisk.

| Kategori                               | Felter                                                                                   | Genstart nødvendig? |
| -------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------- |
| Kanaler                                | `channels.*`, `web` (WhatsApp) — alle indbyggede og udvidelseskanaler | Nej                 |
| Agent og modeller                      | `agent`, `agents`, `models`, `routing`                                                   | Nej                 |
| Automatisering                         | `hooks`, `cron`, `agent.heartbeat`                                                       | Nej                 |
| Sessioner og beskeder                  | `session`, `messages`                                                                    | Nej                 |
| Værktøjer & medier | `tools`, `browser`, `skills`, `audio`, `talk`                                            | Nej                 |
| UI & diverse       | `ui`, `logging`, `identity`, `bindings`                                                  | Nej                 |
| Gateway-server                         | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                  | **Ja**              |
| Infrastruktur                          | `discovery`, `canvasHost`, `plugins`                                                     | **Ja**              |

<Note>
`gateway.reload` og `gateway.remote` er undtagelser — ændringer af dem udløser **ikke** en genstart.
</Note>

## Miljøvariabler + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Validerer + skriver hele konfigurationen og genstarter Gateway i ét trin.


    ````
    <Warning>
    `config.apply` erstatter **hele konfigurationen**. Brug `config.patch` til delvise opdateringer eller `openclaw config set` til enkelte nøgler.
    
</Warning>
    
    Params:
    
    - `raw` (string) — JSON5-payload for hele konfigurationen
    - `baseHash` (valgfri) — config-hash fra `config.get` (påkrævet når konfigurationen findes)
    - `sessionKey` (valgfri) — session key til post-genstarts wake-up ping
    - `note` (valgfri) — note til genstarts-sentinel
    - `restartDelayMs` (valgfri) — forsinkelse før genstart (standard 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # hent payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    Fletter en delvis opdatering ind i den eksisterende konfiguration (JSON merge patch-semantik):


    ````
    - Objekter flettes rekursivt
    - `null` sletter en nøgle
    - Arrays erstattes
    
    Params:
    
    - `raw` (string) — JSON5 med kun de nøgler, der skal ændres
    - `baseHash` (påkrævet) — config-hash fra `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — samme som `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Miljøvariabler

OpenClaw læser miljøvariabler fra parent-processen samt:

- `.env` fra den aktuelle arbejdsmappe (hvis til stede)
- `~/.openclaw/.env` (global fallback)

Ingen af filerne overskriver eksisterende miljøvariabler. Du kan også angive inline-miljøvariabler i konfigurationen:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  Hvis aktiveret og de forventede nøgler ikke er angivet, kører OpenClaw din login-shell og importerer kun de manglende nøgler:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Miljøvariabel-ækvivalent: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  Referér til miljøvariabler i enhver strengværdi i konfigurationen med `${VAR_NAME}`:


```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Regler:

- Kun navne med store bogstaver matches: `[A-Z_][A-Z0-9_]*`
- Manglende/tomme variabler giver en fejl ved indlæsning
- Escap med `$${VAR}` for bogstaveligt output
- Virker også i `$include`-filer
- Inline-substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Se [Environment](/help/environment) for fuld præcedens og kilder.

## Fuld reference

For den komplette felt-for-felt-reference, se **[Configuration Reference](/gateway/configuration-reference)**.

---

Legacy OAuth-importer:

