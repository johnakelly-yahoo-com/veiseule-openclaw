---
summary: "All configuration options for ~/.openclaw/openclaw.json with examples"
read_when:
  - Adding or modifying config fields
  - Letar du efter vanliga konfigurationsmönster
  - Navigerar till specifika konfigurationsavsnitt
title: "Konfiguration"
---

# Configuration 🔧

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json` (comments + trailing commas allowed).

Om filen saknas använder OpenClaw säkra standardvärden. Vanliga skäl att lägga till en konfiguration:

- restrict who can trigger the bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
- control group allowlists + mention behavior (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- customize message prefixes (`messages`)

Se [fullständig referens](/gateway/configuration-reference) för alla tillgängliga fält.

<Tip>
**Ny inom konfiguration?** Börja med `openclaw onboard` för en interaktiv installation, eller kolla in guiden [Configuration Examples](/gateway/configuration-examples) för kompletta konfigurationer att kopiera och klistra in.
</Tip>

## Minimal konfiguration

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Redigera konfiguration

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # fullständig installationsguide
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
    Öppna [http://127.0.0.1:18789](http://127.0.0.1:18789) och använd fliken **Config**.
    Control UI renderar ett formulär från konfigurationsschemat, med en **Raw JSON**-redigerare som reservutväg.
  
</Tab>
  <Tab title="Direct edit">
    Redigera `~/.openclaw/openclaw.json` direkt. Gateway övervakar filen och tillämpar ändringar automatiskt (se [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Schema + UI hints

<Warning>
OpenClaw accepterar endast konfigurationer som helt matchar schemat. Okända nycklar, felaktiga typer eller ogiltiga värden gör att Gateway **vägrar starta**. Det enda undantaget på rotnivå är `$schema` (sträng), så att redigerare kan koppla JSON Schema-metadata.
</Warning>

When validation fails:

- Gateway startar inte
- Endast diagnostikkommandon fungerar (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Kör `openclaw doctor` för att se exakta problem
- Kör `openclaw doctor --fix` (eller `--yes`) för att tillämpa reparationer

## Apply + restart (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Varje kanal har sitt eget konfigurationsavsnitt under `channels.<provider>`. Se den dedikerade kanalsidan för installationssteg:

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
    
    Alla kanaler delar samma DM-policy-mönster:
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    Ange primär modell och valfria reservmodeller:

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
    
    - `agents.defaults.models` definierar modellkatalogen och fungerar som allowlist för `/model`.
    - Modellreferenser använder formatet `provider/model` (t.ex. `anthropic/claude-opus-4-6`).
    - Se [Models CLI](/concepts/models) för att byta modeller i chatten och [Model Failover](/concepts/model-failover) för autentiseringsrotation och fallback-beteende.
    - För anpassade/självhostade leverantörer, se [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) i referensen.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    DM-åtkomst styrs per kanal via `dmPolicy`:

    ```
    - `"pairing"` (standard): okända avsändare får en engångskod för parkoppling att godkänna
    - `"allowlist"`: endast avsändare i `allowFrom` (eller den parkopplade allowlistan)
    - `"open"`: tillåt alla inkommande DM (kräver `allowFrom: ["*"]`)
    - `"disabled"`: ignorera alla DM
    
    För grupper, använd `groupPolicy` + `groupAllowFrom` eller kanalspecifika allowlistor.
    
    Se [fullständig referens](/gateway/configuration-reference#dm-and-group-access) för detaljer per kanal.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Gruppmeddelanden kräver som standard **omnämnande**. Konfigurera mönster per agent:

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
    
    - **Metadata mentions**: inbyggda @-omnämnanden (WhatsApp tryck-för-att-omnämna, Telegram @bot, etc.)
    - **Textmönster**: regex-mönster i `mentionPatterns`
    - Se [fullständig referens](/gateway/configuration-reference#group-chat-mention-gating) för kanalvisa åsidosättningar och self-chat-läge.
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Sessioner styr samtalskontinuitet och isolering:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // rekommenderas för flera användare
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (delad) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Se [Session Management](/concepts/session) för omfattning, identitetslänkar och sändpolicy.
    - Se [fullständig referens](/gateway/configuration-reference#session) för alla fält.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    Kör agentsessioner i isolerade Docker-containrar:

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
    Se [Cron jobs](/automation/cron-jobs) för funktionen översikt och CLI exempel.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Aktivera HTTP webhook-endpoints på Gateway:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Kör flera isolerade agenter med separata arbetsytor och sessioner:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Använd `$include` för att organisera stora konfigurationer:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Hot reload av konfiguration

Gateway övervakar `~/.openclaw/openclaw.json` och tillämpar ändringar automatiskt — ingen manuell omstart behövs för de flesta inställningar.

### Error handling

| Läge                                       | Beteende                                                                                                                                |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (standard) | Tillämpa säkra ändringar direkt utan omstart. Startar automatiskt om för kritiska ändringar.            |
| **`hot`**                                  | Tillämpa endast säkra ändringar utan omstart. Loggar en varning när en omstart krävs — du hanterar den. |
| **`restart`**                              | Startar om Gateway vid alla konfigurationsändringar, säkra eller inte.                                                  |
| **`off`**                                  | Inaktiverar filövervakning. Ändringar träder i kraft vid nästa manuella omstart.                        |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Vad som tillämpas direkt och vad som kräver omstart

De flesta fält tillämpas direkt utan driftstopp. I `hybrid`-läge hanteras ändringar som kräver omstart automatiskt.

| Kategori                            | Fält                                                                                          | Kräver omstart? |
| ----------------------------------- | --------------------------------------------------------------------------------------------- | --------------- |
| Kanaler                             | `channels.*`, `web` (WhatsApp) — alla inbyggda kanaler och tilläggskanaler | Nej             |
| Agent och modeller                  | `agent`, `agents`, `models`, `routing`                                                        | Nej             |
| Automatisering                      | `hooks`, `cron`, `agent.heartbeat`                                                            | Nej             |
| Sessioner och meddelanden           | `session`, `messages`                                                                         | Nej             |
| Verktyg & media | `tools`, `browser`, `skills`, `audio`, `talk`                                                 | Nej             |
| UI & övrigt     | `ui`, `logging`, `identity`, `bindings`                                                       | Nej             |
| Gateway-server                      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                       | **Ja**          |
| Infrastruktur                       | `discovery`, `canvasHost`, `plugins`                                                          | **Ja**          |

<Note>
`gateway.reload` och `gateway.remote` är undantag — att ändra dem triggar **inte** en omstart.
</Note>

## Env vars + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Validerar + skriver hela konfigurationen och startar om Gateway i ett steg.


    ````
    <Warning>
    `config.apply` ersätter **hela konfigurationen**. Använd `config.patch` för partiella uppdateringar, eller `openclaw config set` för enskilda nycklar.
    
</Warning>
    
    Params:
    
    - `raw` (string) — JSON5‑payload för hela konfigurationen
    - `baseHash` (valfri) — konfigurationshash från `config.get` (krävs när konfiguration finns)
    - `sessionKey` (valfri) — sessionsnyckel för wake‑up‑ping efter omstart
    - `note` (valfri) — anteckning för omstartsmarkören
    - `restartDelayMs` (valfri) — fördröjning före omstart (standard 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    Slår samman en partiell uppdatering med den befintliga konfigurationen (JSON merge patch‑semantik):


    ````
    - Objekt slås samman rekursivt
    - `null` tar bort en nyckel
    - Arrayer ersätts
    
    Params:
    
    - `raw` (string) — JSON5 med endast de nycklar som ska ändras
    - `baseHash` (krävs) — konfigurationshash från `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — samma som `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Miljövariabler

OpenClaw läser miljövariabler från föräldraprocessen samt:

- `.env` from the current working directory (if present)
- `~/.openclaw/.env` (global reserv)

Ingen av filerna åsidosätter befintliga miljövariabler. Du kan också ange inbäddade miljövariabler i konfigurationen:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  Om aktiverat och förväntade nycklar inte är satta kör OpenClaw ditt inloggningsskal och importerar endast de saknade nycklarna:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Motsvarande miljövariabel: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  Referera till miljövariabler i valfri sträng i konfigurationen med `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Regler:

- Endast versala namn matchas: `[A-Z_][A-Z0-9_]*`
- Saknade/tomma variabler ger ett fel vid inläsning
- Escapa med `$${VAR}` för bokstavlig utskrift
- Fungerar i `$include`‑filer
- Inbäddad ersättning: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Se [Environment](/help/environment) för fullständig prioriteringsordning och källor.

## Fullständig referens

För en komplett fält‑för‑fält‑referens, se **[Configuration Reference](/gateway/configuration-reference)**.

---

Legacy OAuth imports:
