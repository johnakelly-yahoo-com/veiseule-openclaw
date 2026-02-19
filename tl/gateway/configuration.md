---
summary: "Lahat ng opsyon sa configuration para sa ~/.openclaw/openclaw.json na may mga halimbawa"
read_when:
  - Pagdaragdag o pagbabago ng mga field ng config
  - Naghahanap ng karaniwang pattern ng configuration
  - Pag-navigate sa mga partikular na seksyon ng config
title: "Konpigurasyon"
---

# Konpigurasyon 🔧

Binabasa ng OpenClaw ang opsyonal na **JSON5** config mula sa `~/.openclaw/openclaw.json` (pinapayagan ang mga komento + trailing commas).

Kung nawawala ang file, gagamit ang OpenClaw ng ligtas na mga default. Mga karaniwang dahilan para magdagdag ng config:

- limitahan kung sino ang puwedeng mag‑trigger ng bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, atbp.)
- kontrolin ang mga group allowlist + asal ng pag‑mention (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- i‑customize ang mga prefix ng mensahe (`messages`)

Tingnan ang [buong reference](/gateway/configuration-reference) para sa lahat ng available na field.

<Tip>
**Bago sa configuration?** Magsimula sa `openclaw onboard` para sa interactive na setup, o tingnan ang gabay na [Configuration Examples](/gateway/configuration-examples) para sa kumpletong copy-paste na mga config.
</Tip>

## Minimal na config

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Pag-edit ng config

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # buong setup wizard
    openclaw configure     # config wizard
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
    Buksan ang [http://127.0.0.1:18789](http://127.0.0.1:18789) at gamitin ang tab na **Config**.
    Ang Control UI ay nagre-render ng form mula sa config schema, na may **Raw JSON** editor bilang alternatibong opsyon.
  
</Tab>
  <Tab title="Direct edit">
    Direktang i-edit ang `~/.openclaw/openclaw.json`. Binabantayan ng Gateway ang file at awtomatikong inilalapat ang mga pagbabago (tingnan ang [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Schema + mga UI hint

<Warning>
Tumatanggap lamang ang OpenClaw ng mga configuration na ganap na tumutugma sa schema. Ang mga hindi kilalang key, maling uri ng data, o invalid na halaga ay magdudulot sa Gateway na **tumangging mag-start**. Ang tanging exception sa root level ay `$schema` (string), upang makapag-attach ang mga editor ng JSON Schema metadata.
</Warning>

Maaaring mag‑register ang mga channel plugin at extension ng schema + mga UI hint para sa kanilang config, kaya nananatiling schema‑driven ang mga setting ng channel sa iba’t ibang app nang walang hard‑coded na form.

- Hindi nagbo-boot ang Gateway
- Tanging mga diagnostic command lang ang gumagana (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Patakbuhin ang `openclaw doctor` upang makita ang eksaktong mga isyu
- Patakbuhin ang `openclaw doctor --fix` (o `--yes`) upang ilapat ang mga pag-aayos

## I‑apply + i‑restart (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Bawat channel ay may sariling seksyon ng config sa ilalim ng `channels.<provider>`. Tingnan ang nakalaang pahina ng channel para sa mga hakbang sa setup:

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
    
    Lahat ng channel ay may iisang pattern ng DM policy:
    
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
    Itakda ang pangunahing model at mga opsyonal na fallback:


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
    
    - `agents.defaults.models` ang nagtatakda ng model catalog at nagsisilbing allowlist para sa `/model`.
    - Gumagamit ang model refs ng `provider/model` na format (hal. `anthropic/claude-opus-4-6`).
    - Tingnan ang [Models CLI](/concepts/models) para sa pagpapalit ng model sa chat at ang [Model Failover](/concepts/model-failover) para sa auth rotation at fallback behavior.
    - Para sa custom/self-hosted na mga provider, tingnan ang [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) sa reference.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    Kinokontrol ang DM access kada channel sa pamamagitan ng `dmPolicy`:


    ```
    - `"pairing"` (default): ang mga hindi kilalang sender ay makakatanggap ng one-time pairing code para maaprubahan
    - `"allowlist"`: tanging mga sender sa `allowFrom` (o sa paired allow store)
    - `"open"`: payagan ang lahat ng papasok na DM (nangangailangan ng `allowFrom: ["*"]`)
    - `"disabled"`: balewalain ang lahat ng DM
    
    Para sa mga group, gamitin ang `groupPolicy` + `groupAllowFrom` o mga channel-specific na allowlist.
    
    Tingnan ang [buong reference](/gateway/configuration-reference#dm-and-group-access) para sa mga detalyeng per-channel.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Ang mga mensahe sa grupo ay default na **nangangailangan ng pagbanggit**. I-configure ang mga pattern bawat agent:

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
    
    - **Metadata mentions**: native @-mentions (WhatsApp tap-to-mention, Telegram @bot, atbp.)
    - **Text patterns**: regex patterns sa `mentionPatterns`
    - Tingnan ang [full reference](/gateway/configuration-reference#group-chat-mention-gating) para sa per-channel overrides at self-chat mode.
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">    Kinokontrol ng mga session ang pagpapatuloy at paghihiwalay ng mga pag-uusap:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // inirerekomenda para sa multi-user
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (shared) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Tingnan ang [Session Management](/concepts/session) para sa scoping, identity links, at send policy.
    - Tingnan ang [full reference](/gateway/configuration-reference#session) para sa lahat ng field.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    Patakbuhin ang mga agent session sa mga nakahiwalay na Docker container:

    ```
    scripts/sandbox-setup.sh
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">    ```json5
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

  <Accordion title="Configure cron jobs">    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">    I-enable ang HTTP webhook endpoints sa Gateway:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">    Magpatakbo ng maraming nakahiwalay na agent na may magkakahiwalay na workspace at session:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">    Gamitin ang `$include` upang ayusin ang malalaking config:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Config hot reload

Binabantayan ng Gateway ang `~/.openclaw/openclaw.json` at awtomatikong ina-apply ang mga pagbabago — hindi kailangan ng manu-manong restart para sa karamihan ng setting.

### Pag‑hawak ng error

| Mode                                      | Behavior                                                                                                                                                               |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | Agarang ina-apply ang mga ligtas na pagbabago. Awtomatikong nagre-restart para sa mga kritikal na pagbabago.                           |
| **`hot`**                                 | Ina-apply lamang nang live ang mga ligtas na pagbabago. Naglalagay ng babala sa log kapag kailangan ng restart — ikaw ang bahala rito. |
| **`restart`**                             | Nagre-restart ang Gateway sa anumang pagbabago ng config, ligtas man o hindi.                                                                          |
| **`off`**                                 | Hindi pinapagana ang file watching. Magkakabisa ang mga pagbabago sa susunod na manu-manong restart.                                   |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Ano ang maaaring i-hot-apply at ano ang nangangailangan ng restart

Karamihan sa mga field ay naia-apply nang live nang walang downtime. Sa `hybrid` mode, ang mga pagbabagong nangangailangan ng restart ay awtomatikong hinahawakan.

| Category                           | Fields                                                                                     | Kailangan ng restart? |
| ---------------------------------- | ------------------------------------------------------------------------------------------ | --------------------- |
| Channels                           | `channels.*`, `web` (WhatsApp) — lahat ng built-in at extension channel | Hindi                 |
| Agent & models | `agent`, `agents`, `models`, `routing`                                                     | Hindi                 |
| Awtomasyon                         | `hooks`, `cron`, `agent.heartbeat`                                                         | Hindi                 |
| Mga session at mensahe             | `session`, `messages`                                                                      | Hindi                 |
| Mga tool at media                  | `tools`, `browser`, `skills`, `audio`, `talk`                                              | Hindi                 |
| UI at iba pa                       | `ui`, `logging`, `identity`, `bindings`                                                    | Hindi                 |
| Gateway server                     | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                    | **Oo**                |
| Imprastraktura                     | `discovery`, `canvasHost`, `plugins`                                                       | **Oo**                |

<Note>
`gateway.reload` at `gateway.remote` ay mga exception — ang pagbabago sa mga ito ay **hindi** magti-trigger ng restart.
</Note>

## Mga env var + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Vina-validate + isinusulat ang buong config at nire-restart ang Gateway sa isang hakbang.


    ````
    <Warning>
    `config.apply` pinapalitan ang **buong config**. Gamitin ang `config.patch` para sa bahagyang mga update, o `openclaw config set` para sa iisang key.
    
</Warning>
    
    Params:
    
    - `raw` (string) — JSON5 payload para sa buong config
    - `baseHash` (optional) — config hash mula sa `config.get` (kinakailangan kapag may umiiral na config)
    - `sessionKey` (optional) — session key para sa post-restart wake-up ping
    - `note` (optional) — tala para sa restart sentinel
    - `restartDelayMs` (optional) — delay bago ang restart (default 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # kunin ang payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    Pinagsasama ang bahagyang update sa umiiral na config (JSON merge patch semantics):


    ````
    - Ang mga object ay recursive na pinagsasama
    - Ang `null` ay nagtatanggal ng key
    - Ang mga array ay pinapalitan
    
    Params:
    
    - `raw` (string) — JSON5 na may mga key lang na babaguhin
    - `baseHash` (kinakailangan) — config hash mula sa `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — pareho sa `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Mga environment variable

Binabasa ng OpenClaw ang mga env var mula sa parent process kasama ang:

- `.env` mula sa kasalukuyang working directory (kung mayroon)
- `~/.openclaw/.env` (global fallback)

Walang alinman sa file ang nag-o-override ng mga umiiral na env var. Maaari ka ring magtakda ng inline na env var sa config:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  Kapag naka-enable at hindi nakatakda ang mga inaasahang key, pinapatakbo ng OpenClaw ang iyong login shell at ini-import lamang ang mga nawawalang key:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var equivalent: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  I-refer ang mga env var sa anumang string value ng config gamit ang `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Mga Patakaran:

- Mga uppercase na pangalan lamang ang tinutugma: `[A-Z_][A-Z0-9_]*`
- Ang nawawala o walang laman na mga var ay magdudulot ng error sa oras ng pag-load
- I-escape gamit ang `$${VAR}` para sa literal na output
- Gumagana sa loob ng `$include` na mga file
- Inline na pagpapalit: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Tingnan ang [Environment](/help/environment) para sa kumpletong precedence at mga pinagmulan.

## Buong sanggunian

Para sa kumpletong sanggunian na nakaayos bawat field, tingnan ang **[Configuration Reference](/gateway/configuration-reference)**.

---

Legacy OAuth imports:

