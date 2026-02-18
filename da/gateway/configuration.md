---
title: "Konfiguration"
---

# Konfiguration

OpenClaw læser en valgfri <Tooltip tip="JSON5 understøtter kommentarer og afsluttende kommaer">**JSON5**</Tooltip>-konfiguration fra `~/.openclaw/openclaw.json`.

Hvis filen mangler, bruger OpenClaw sikre standardindstillinger. Almindelige grunde til at tilføje en config:

- Forbinde kanaler og styre, hvem der kan sende beskeder til botten
- Sætte modeller, værktøjer, sandboxing eller automatisering (cron, hooks)
- Justere sessioner, medier, netværk eller UI

Se [fuld reference](/gateway/configuration-reference) for alle tilgængelige felter.

<Tip>
**Ny i konfiguration?** Start med `openclaw onboard` for interaktiv opsætning, eller se guiden [Configuration Examples](/gateway/configuration-examples) for komplette copy-paste eksempler.
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
  <Tab title="Interaktiv guide">
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
    Control UI genererer en formular fra konfigurationsskemaet med en **Raw JSON**-editor som nødudgang.
  </Tab>
  <Tab title="Direkte redigering">
    Redigér `~/.openclaw/openclaw.json` direkte. Gateway overvåger filen og anvender ændringer automatisk (se [hot reload](#config-hot-reload)).
  </Tab>
</Tabs>

## Streng validering

<Warning>
OpenClaw accepterer kun konfigurationer, der fuldt ud matcher skemaet. Ukendte nøgler, forkerte typer eller ugyldige værdier får Gateway til at **nægte at starte**. Den eneste undtagelse på rodeniveau er `$schema` (string), så editorer kan tilknytte JSON Schema metadata.
</Warning>

Når validering fejler:

- Gateway starter ikke
- Kun diagnostiske kommandoer virker (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Kør `openclaw doctor` for at se de præcise problemer
- Kør `openclaw doctor --fix` (eller `--yes`) for at anvende rettelser

## Almindelige opgaver

<AccordionGroup>
  <Accordion title="Opsæt en kanal (WhatsApp, Telegram, Discord, osv.)">
    Hver kanal har sin egen sektion under `channels.<provider>`. Se den dedikerede kanalside for opsætning:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Alle kanaler deler samme DM-policy-mønster:

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
  </Accordion>

  <Accordion title="Vælg og konfigurer modeller">
    Sæt primær model og eventuelle fallbacks:

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
    - Se [Models CLI](/concepts/models) for at skifte model i chat og [Model Failover](/concepts/model-failover) for fallback-adfærd.
    - For brugerdefinerede/self-hosted udbydere, se [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls).

  </Accordion>

  <Accordion title="Styr hvem der kan skrive til botten">
    DM-adgang styres pr. kanal via `dmPolicy`:

    - `"pairing"` (standard): ukendte afsendere får en engangskode til godkendelse
    - `"allowlist"`: kun afsendere i `allowFrom` (eller parret allow-store)
    - `"open"`: tillad alle indgående DMs (kræver `allowFrom: ["*"]`)
    - `"disabled"`: ignorér alle DMs

    For grupper, brug `groupPolicy` + `groupAllowFrom` eller kanal-specifikke allowlists.

    Se [fuld reference](/gateway/configuration-reference#dm-and-group-access) for detaljer.

  </Accordion>

  <Accordion title="Opsæt mention-gating i gruppechat">
    Gruppebeskeder kræver som standard **mention**. Konfigurer mønstre pr. agent:

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

    - **Metadata-mentions**: native @-mentions (WhatsApp tap-to-mention, Telegram @bot osv.)
    - **Tekstmønstre**: regex-mønstre i `mentionPatterns`
    - Se [fuld reference](/gateway/configuration-reference#group-chat-mention-gating) for overrides pr. kanal og self-chat-tilstand.

  </Accordion>

  <Accordion title="Konfigurer sessioner og nulstillinger">
    Sessioner styrer samtalekontinuitet og isolation:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // anbefalet til multi-bruger
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Se [Session Management](/concepts/session) for scoping og identitetslinks.
    - Se [fuld reference](/gateway/configuration-reference#session) for alle felter.

  </Accordion>

  <Accordion title="Aktivér sandboxing">
    Kør agent-sessioner i isolerede Docker-containere:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Byg image først: `scripts/sandbox-setup.sh`

    Se [Sandboxing](/gateway/sandboxing) for fuld guide og [fuld reference](/gateway/configuration-reference#sandbox) for alle muligheder.

  </Accordion>

  <Accordion title="Opsæt heartbeat (periodiske check-ins)">
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

    - `every`: varighed (`30m`, `2h`). Sæt `0m` for at deaktivere.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Se [Heartbeat](/gateway/heartbeat).

  </Accordion>

  <Accordion title="Konfigurer cron-jobs">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    Se [Cron jobs](/automation/cron-jobs).

  </Accordion>

  <Accordion title="Opsæt webhooks (hooks)">
    Aktivér HTTP webhook-endpoints på Gateway:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    Se [fuld reference](/gateway/configuration-reference#hooks).

  </Accordion>

  <Accordion title="Konfigurer multi-agent routing">
    Kør flere isolerede agenter med separate workspaces og sessioner:

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

    Se [Multi-Agent](/concepts/multi-agent) og [fuld reference](/gateway/configuration-reference#multi-agent-routing).

  </Accordion>

  <Accordion title="Opdel config i flere filer ($include)">
    Brug `$include` til at organisere store configs:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **En fil**: erstatter objektet
    - **Array af filer**: deep-merge i rækkefølge (senere vinder)
    - **Søskendenøgler**: merges efter includes (overskriver)
    - **Nested includes**: op til 10 niveauer
    - **Relative stier**: opløses relativt til inkluderende fil
    - **Fejlhåndtering**: klare fejl ved manglende filer, parse-fejl og cirkulære includes

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway overvåger `~/.openclaw/openclaw.json` og anvender ændringer automatisk — ingen manuel genstart nødvendig for de fleste indstillinger.

### Reload modes

| Mode                   | Adfærd                                                                 |
| ---------------------- | ---------------------------------------------------------------------- |
| **`hybrid`** (standard) | Hot-applier sikre ændringer straks. Genstarter automatisk ved kritiske. |
| **`hot`**              | Hot-applier kun sikre ændringer. Logger advarsel hvis genstart kræves. |
| **`restart`**          | Genstarter Gateway ved enhver config-ændring.                          |
| **`off`**              | Deaktiverer overvågning. Ændringer træder i kraft ved manuel genstart. |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

<Note>
`gateway.reload` og `gateway.remote` er undtagelser — ændringer her udløser **ikke** genstart.
</Note>

## Fuld reference

For komplet felt-for-felt reference, se **[Configuration Reference](/gateway/configuration-reference)**.

---

_Relateret: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
