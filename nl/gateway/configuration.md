---
summary: "Configuratie-overzicht: veelvoorkomende taken, snelle installatie en links naar de volledige referentie"
read_when:
  - OpenClaw voor de eerste keer instellen
  - Op zoek naar veelvoorkomende configuratiepatronen
  - Navigeren naar specifieke config-secties
title: "Configuratie"
---

# Configuratie 🔧

OpenClaw leest een optionele **JSON5**-config uit `~/.openclaw/openclaw.json` (commentaar + afsluitende komma’s toegestaan).

Als het bestand ontbreekt, gebruikt OpenClaw veilige-achtige standaardwaarden (ingebedde Pi-agent + per-afzender sessies + werkruimte `~/.openclaw/workspace`). Meestal heb je alleen een config nodig om:

- Verbind kanalen en bepaal wie de bot berichten mag sturen
- Stel modellen, tools, sandboxing of automatisering (cron, hooks) in
- Optimaliseer sessies, media, netwerkinstellingen of UI

Zie de [volledige referentie](/gateway/configuration-reference) voor elk beschikbaar veld.

<Tip>
Alle configuratieopties voor ~/.openclaw/openclaw.json met voorbeelden
</Tip>

## Minimale config (aanbevolen startpunt)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Env-var substitutie in config

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # volledige installatiewizard
openclaw configure     # configuratiewizard
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    De Gateway stelt een JSON Schema-weergave van de config beschikbaar via `config.schema` voor UI-editors.
    De Control UI rendert een formulier vanuit dit schema, met een **Raw JSON**-editor als ontsnappingsluik.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (of `OPENCLAW_CONFIG_PATH`) De Gateway bewaakt het bestand en past wijzigingen automatisch toe (zie [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Strikte configvalidatie

<Warning>
OpenClaw accepteert alleen configuraties die volledig overeenkomen met het schema. Onbekende sleutels, onjuist gevormde typen of ongeldige waarden zorgen ervoor dat de Gateway **weigert te starten** om veiligheidsredenen. De enige uitzondering op rootniveau is `$schema` (string), zodat editors JSON Schema-metadata kunnen koppelen.
</Warning>

Wanneer validatie faalt:

- De Gateway start niet.
- Alleen diagnostische opdrachten zijn toegestaan (bijvoorbeeld: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Voer `openclaw doctor` uit om de exacte problemen te zien.
- Voer `openclaw doctor --fix` (of `--yes`) uit om migraties/reparaties toe te passen.

## Veelvoorkomende taken

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">Elk kanaal heeft een eigen configuratiesectie onder `channels.<provider>`. Zie de speciale kanaalpagina voor installatiestappen:

    ```
    {
      channels: {
        whatsapp: {
          groepbeleid: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groepsbeleid: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signaal: {
          groupPolicy: "allowlist",
          groepstoelating van: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groepAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org. om"],
        },
        discord: {
          groepbeleid: "allowlist",
          gilden: {
            GUILD_ID: {
              kanalen: { help: { allow: true } },
            },
          },
        },
        slack: {
          groepbeleid: "allowlist",
          kanalen: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">Stel het primaire model en optionele fallback-modellen in:

    ```
    {
      agents: {
        defaults: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              commando: "my-cli",
              args: ["--json"],
              uitvoer: "json",
              modelArg: "--model",
              sessionArg: "--session",
              sessiemodus: "Bestaat",
              systemPromptArg: "--system",
              systemPromptWhen: "eerste",
              afbeelding: "--afbeelding",
              afbeeldingsmodus: "herhalen",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">Openbare DM's: `channels.mattermost.dmPolicy="open"` plus `channels.mattermost.allowFrom=["*"]`.

    ```
    - `"pairing"` (standaard): onbekende afzenders krijgen een eenmalige koppelcode ter goedkeuring
    - `"allowlist"`: alleen afzenders in `allowFrom` (of de gekoppelde allow store)
    - `"open"`: sta alle inkomende DM’s toe (vereist `allowFrom: ["*"]`)
    - `"disabled"`: negeer alle DM’s
    
    Gebruik voor groepen `groupPolicy` + `groupAllowFrom` of kanaalspecifieke allowlists.
    
    Zie de [volledige referentie](/gateway/configuration-reference#dm-and-group-access) voor details per kanaal.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Groepsberichten vereisen standaard een **vermelding**. `agents.defaults.subagents` configureert sub-agent standaard:

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

  <Accordion title="Configure sessions and resets">Gesprek sessie isolatie:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // aanbevolen voor meerdere gebruikers
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (gedeeld) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Zie [Session Management](/concepts/session) voor scope, identiteitskoppelingen en verzendbeleid.
    - Zie de [volledige referentie](/gateway/configuration-reference#session) voor alle velden.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">Voer agentsessies uit in geïsoleerde Docker-containers:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">```json5
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
    - `every`: duurstring (`30m`, `2h`). Stel `0m` in om uit te schakelen.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Zie [Heartbeat](/gateway/heartbeat) voor de volledige handleiding.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Zie [Cron jobs](/automation/cron-jobs) voor het functieoverzicht en CLI-voorbeelden.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Schakel een eenvoudig HTTP webhook eindpunt in op de Gateway HTTP-server.

    ```
    {
      hooks: {
        enabled: true
        token: "Shared-secret",
        pad: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/. penklauw/hooks",
        toewijzingen: [
          {
            match: { path: "gmail" },
            actie: "agent",
            wakeModus: "nu",
            naam: "Gmail",
            sessieKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "Van: {{messages[0].from}}\nBetreft: {{messages[0].subject}}\n{{messages[0].snippet}}",
            levering: waar,
            kanaal: "laat",
            model: "openai/gpt-5. -mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Voer meerdere geïsoleerde agenten uit (gescheiden werkruimte, `agentDir`, sessies) binnen één Gateway.

    ```
    {
      agents: {
        lijst: [
          { id: "home", standaard: waar, workspace: "~/. penclaw/workspace-home" },
          { id: "work", workspace: "~/. penclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
      kanalen: {
        whatsapp: {
          accounts: {
            persoonlijk: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Splits je config in meerdere bestanden met behulp van de `$include`-directive. Dit is handig voor:

    ```
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

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Config hot reload)

De Gateway kijkt naar `~/.openclaw/openclaw.json` (of `OPENCLAW_CONFIG_PATH`) en past de wijzigingen automatisch toe.

### Herlaadmodi

| Modus:                                                                         | Legacy gedrag:                                                                                                                        |
| ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (standaard)                                                    | Past veilige wijzigingen direct toe zonder onderbreking. Start automatisch opnieuw voor kritieke wijzigingen.         |
| `{provider}`                                                                                   | Past alleen veilige wijzigingen direct toe. Logt een waarschuwing wanneer een herstart nodig is — je regelt dit zelf. |
| modus: **unset** (behandeld als "niet automatisch starten") | `restart`: herstart de Gateway bij elke wijziging van de configuratie.                                                |
| **Inline substitutie:**                                                        | Schakelt bestandsbewaking uit. Wijzigingen worden van kracht bij de volgende handmatige herstart.                     |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Wat wordt direct toegepast vs wat vereist een herstart

De meeste velden worden direct toegepast zonder downtime. In `hybrid`-modus worden wijzigingen die een herstart vereisen automatisch afgehandeld.

| Categorie                            | Velden                                                                                                                   | Herstart nodig?                    |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — alle ingebouwde en extensiekanalen                                   | Nee                                |
| Agent & modellen | `model`: per agent standaard model, overschrijft `agents.defaults.model` voor die agent. | Nee                                |
| Automatisering                       | `hooks`, `cron`, `agent.heartbeat`                                                                                       | Nee                                |
| messages                             | `messages.inbound`                                                                                                       | Nee                                |
| Tools & media    | `tools`, `browser`, `skills`, `audio`, `talk`                                                                            | Nee                                |
| Schema + UI-hints                    | `ui`, `logging`, `identity`, `bindings`                                                                                  | Nee                                |
| Gateway-server                       | `gateway` (port/bind/auth/control UI/tailscale)                                                       | **Vermeld types:** |
| Infrastructuur                       | `discovery`, `canvasHost`, `plugins`                                                                                     | **Regels:**        |

<Note>
`gateway.reload` en `gateway.remote` zijn uitzonderingen — het wijzigen hiervan veroorzaakt **geen** herstart.
</Note>

## Gedeeltelijke updates (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Gebruik `config.apply` om de volledige config te valideren + weg te schrijven en de Gateway in één stap te herstarten.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Gebruik `config.patch` om een gedeeltelijke update samen te voegen met de bestaande config zonder
onverwante sleutels te overschrijven. Het past JSON merge patch-semantiek toe:

    ````
    - Objecten worden recursief samengevoegd
    - `null` verwijdert een sleutel
    - Arrays worden vervangen
    
    Params:
    
    - `raw` (string) — JSON5 met alleen de sleutels die moeten worden gewijzigd
    - `baseHash` (verplicht) — config-hash van `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — hetzelfde als `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Omgevingsvariabelen

OpenClaw leest omgevingsvariabelen van het bovenliggende proces plus:

- `.env` uit de huidige werkdirectory (indien aanwezig)
- een globale fallback `.env` uit `~/.openclaw/.env` (oftewel `$OPENCLAW_STATE_DIR/.env`)

Geen van beide `.env`-bestanden overschrijft bestaande env vars. Je kunt ook inline env vars in de config opgeven.

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

<Accordion title="Shell env import (optional)">Opt-in gemak: indien ingeschakeld en geen van de verwachte sleutels nog is ingesteld,
start OpenClaw je login-shell en importeert alleen de ontbrekende verwachte sleutels (nooit overschrijven).

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

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Je kunt omgevingsvariabelen direct refereren in elke config-stringwaarde met de
`${VAR_NAME}`-syntaxis.

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

Regels:

- Alleen hoofdletter-env-varnamen worden gematcht: `[A-Z_][A-Z0-9_]*`
- Ontbrekende/lege variabelen veroorzaken een fout tijdens het laden
- Escapen met `$${VAR}` om een letterlijke `${VAR}` uit te voeren
- Werkt met `$include` (ingesloten bestanden krijgen ook substitutie)
- Inline vervanging: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Zie [/environment](/help/environment) voor volledige prioriteit en bronnen.

## Volledige referentie

**Nieuw met configuratie?** Bekijk de gids [Configuration Examples](/gateway/configuration-examples) voor volledige voorbeelden met gedetailleerde uitleg!

---

Voorbeeld (provider/model-specifieke allowlist):

