---
summary: "ဥပမာများနှင့်အတူ ~/.openclaw/openclaw.json အတွက် ဖွဲ့စည်းပြင်ဆင်မှု ရွေးချယ်စရာများ အားလုံး"
read_when:
  - ဖွဲ့စည်းပြင်ဆင်မှု အကွက်များ ထည့်သွင်းခြင်း သို့မဟုတ် ပြင်ဆင်ခြင်း
  - အသုံးများသော configuration ပုံစံများကို ရှာဖွေနေပါသလား
  - သတ်မှတ်ထားသော config အပိုင်းများသို့ သွားရန်
title: "ဖွဲ့စည်းပြင်ဆင်ခြင်း"
---

# ဖွဲ့စည်းပြင်ဆင်ခြင်း 🔧

OpenClaw သည် `~/.openclaw/openclaw.json` မှ **JSON5** ဖွဲ့စည်းပြင်ဆင်မှု (မှတ်ချက်များ + နောက်ဆုံး ကော်မာများ ခွင့်ပြု) ကို ရွေးချယ်နိုင်သည့် အနေဖြင့် ဖတ်ရှုသည်။

ဖိုင်မရှိပါက OpenClaw သည် လုံခြုံသော default တန်ဖိုးများကို အသုံးပြုမည်။ Config တစ်ခု ထည့်သွင်းရသော အကြောင်းရင်းများ:

- ဘော့ကို လှုံ့ဆော်နိုင်သူများကို ကန့်သတ်ရန် (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` စသည်)
- အုပ်စု allowlist များနှင့် mention အပြုအမူကို ထိန်းချုပ်ရန် (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- မက်ဆေ့ချ် prefix များကို စိတ်ကြိုက်ပြင်ဆင်ရန် (`messages`)

ရရှိနိုင်သော field အားလုံးအတွက် [full reference](/gateway/configuration-reference) ကို ကြည့်ပါ။

<Tip>
**Configuration အသစ်လား?** Interactive setup အတွက် `openclaw onboard` ကို စတင်အသုံးပြုပါ၊ သို့မဟုတ် အပြည့်အစုံ copy‑paste config များအတွက် [Configuration Examples](/gateway/configuration-examples) လမ်းညွှန်ကို ကြည့်ပါ။
</Tip>

## အနည်းဆုံး config

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Config ကို ပြင်ဆင်ခြင်း

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # full setup wizard
openclaw configure     # config wizard
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    Open [http://127.0.0.1:18789](http://127.0.0.1:18789) ကို ဖွင့်ပြီး **Config** tab ကို အသုံးပြုပါ။
    Control UI သည် config schema အပေါ် အခြေခံပြီး form တစ်ခုကို render လုပ်ပေးပြီး၊ လိုအပ်ပါက **Raw JSON** editor ကို အသုံးပြုနိုင်သည်။
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` ကို တိုက်ရိုက် ပြင်ဆင်ပါ။ Gateway သည် ဖိုင်ကို စောင့်ကြည့်ပြီး ပြောင်းလဲမှုများကို အလိုအလျောက် အသုံးချမည် (ကြည့်ရန် [hot reload](#config-hot-reload))။
  
</Tab>
</Tabs>

## Schema + UI အညွှန်းများ

<Warning>
OpenClaw only accepts configurations that fully match the schema. မသိသော key များ၊ type မမှန်ကန်မှုများ၊ သို့မဟုတ် တန်ဖိုးမမှန်မှုများရှိပါက Gateway သည် **စတင်မလုပ်ဆောင်ပါ**။ Root-level တွင် တစ်ခုတည်းသော ခြွင်းချက်မှာ `$schema` (string) ဖြစ်ပြီး editor များအတွက် JSON Schema metadata ချိတ်ဆက်ရန် အသုံးပြုနိုင်သည်။
</Warning>

ချန်နယ် ပလဂင်များနှင့် တိုးချဲ့မှုများသည် ၎င်းတို့၏ ဖွဲ့စည်းပြင်ဆင်မှုအတွက် schema + UI အညွှန်းများကို မှတ်ပုံတင်နိုင်ပြီး၊ အက်ပ်များအနှံ့တွင် schema ကို အခြေခံထားသည့် ဆက်တင်များကို hard-coded ဖောင်များ မလိုအပ်ဘဲ ထိန်းသိမ်းနိုင်သည်။

- Gateway စတင်မလုပ်ဆောင်ပါ
- Diagnostic command များသာ အလုပ်လုပ်သည် (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- ပြဿနာအသေးစိတ်ကို ကြည့်ရန် `openclaw doctor` ကို run ပါ။
- ပြင်ဆင်မှုများ အလိုအလျောက် လုပ်ဆောင်ရန် `openclaw doctor --fix` (သို့မဟုတ် `--yes`) ကို run ပါ။

## အသုံးချ + ပြန်လည်စတင်ခြင်း (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">Channel တစ်ခုချင်းစီတွင် `channels.` အောက်တွင် ကိုယ်ပိုင် config အပိုင်းရှိသည်။<provider>`commands.bashForegroundMs` သည် bash ကို background သို့ မပြောင်းမီ စောင့်ဆိုင်းချိန်ကို ထိန်းချုပ်သည်။ Setup လုပ်ဆောင်ရန် အောက်ပါ channel စာမျက်နှာကို ကြည့်ပါ:

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
    
    Channel အားလုံးတွင် တူညီသော DM policy pattern ကို အသုံးပြုသည်:
    
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

  <Accordion title="Choose and configure models">Primary model နှင့် optional fallback များကို သတ်မှတ်ပါ:

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
    
    - `agents.defaults.models` သည် model catalog ကို သတ်မှတ်ပြီး `/model` အတွက် allowlist အဖြစ် လုပ်ဆောင်သည်။
    - Model reference များသည် `provider/model` format ကို အသုံးပြုသည် (ဥပမာ `anthropic/claude-opus-4-6`)။
    - Chat အတွင်း model ပြောင်းရန် [Models CLI](/concepts/models) ကို ကြည့်ပါ၊ authentication rotation နှင့် fallback behavior အတွက် [Model Failover](/concepts/model-failover) ကို ကြည့်ပါ။
    - Custom/self-hosted provider များအတွက် reference ထဲရှိ [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) ကို ကြည့်ပါ။
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">DM access ကို channel တစ်ခုချင်းစီအလိုက် `dmPolicy` ဖြင့် ထိန်းချုပ်သည်:

    ```
    - `"pairing"` (default): မသိသော sender များကို အတည်ပြုရန် တစ်ကြိမ်သာ အသုံးပြုနိုင်သော pairing code ပေးသည်
    - `"allowlist"`: `allowFrom` (သို့မဟုတ် paired allow store) ထဲရှိ sender များသာ ခွင့်ပြုသည်
    - `"open"`: ဝင်လာသော DM အားလုံးကို ခွင့်ပြုသည် (`allowFrom: ["*"]` လိုအပ်သည်)
    - `"disabled"`: DM အားလုံးကို လျစ်လျူရှုသည်
    
    Group များအတွက် `groupPolicy` + `groupAllowFrom` သို့မဟုတ် channel အလိုက် allowlist များကို အသုံးပြုပါ။
    
    Channel အလိုက် အသေးစိတ်များအတွက် [full reference](/gateway/configuration-reference#dm-and-group-access) ကို ကြည့်ပါ။
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    အုပ်စုစာများသည် ပုံမှန်အားဖြင့် **mention လိုအပ်သည်** ဖြစ်သည်။ Agent တစ်ခုချင်းစီအတွက် pattern များကို သတ်မှတ်ပါ:

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
    
    - **Metadata mentions**: native @-mentions (WhatsApp tap-to-mention, Telegram @bot စသည်)
    - **Text patterns**: `mentionPatterns` အတွင်းရှိ regex pattern များ
    - Channel တစ်ခုချင်းစီအလိုက် override များနှင့် self-chat mode အတွက် [full reference](/gateway/configuration-reference#group-chat-mention-gating) ကိုကြည့်ပါ။
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">    Sessions များသည် စကားပြောဆက်လက်တည်ရှိမှုနှင့် သီးခြားခွဲခြားမှုကို ထိန်းချုပ်သည်:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // multi-user အတွက် အကြံပြုထားသည်
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (မျှဝေထားသည်) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Scope သတ်မှတ်မှုများ၊ identity links နှင့် send policy အတွက် [Session Management](/concepts/session) ကိုကြည့်ပါ။
    - Field အားလုံးအတွက် [full reference](/gateway/configuration-reference#session) ကိုကြည့်ပါ။
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    Agent session များကို သီးခြားခွဲထားသော Docker container များအတွင်း run လုပ်ပါ:

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

  <Accordion title="Set up webhooks (hooks)">Gateway ပေါ်တွင် HTTP webhook endpoint များကို ဖွင့်ပါ:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">သီးခြား workspace များနှင့် session များပါဝင်သော isolated agent များကို များစွာ run လုပ်ပါ:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">အရွယ်အစားကြီးမားသော config များကို စနစ်တကျ စုစည်းရန် `$include` ကို အသုံးပြုပါ:

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

Gateway သည် `~/.openclaw/openclaw.json` ကို စောင့်ကြည့်ပြီး ပြောင်းလဲမှုများကို အလိုအလျောက် အသုံးချသည် — setting အများစုအတွက် လက်ဖြင့် restart လုပ်ရန် မလိုအပ်ပါ။

### အမှား ကိုင်တွယ်ခြင်း

| Mode                                      | အပြုအမူ                                                                                                                          |
| ----------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (ပုံမှန်) | လုံခြုံသော ပြောင်းလဲမှုများကို ချက်ချင်း hot-apply လုပ်သည်။ အရေးကြီးသော ပြောင်းလဲမှုများအတွက် အလိုအလျောက် restart လုပ်သည်။       |
| **`hot`**                                 | လုံခြုံသော ပြောင်းလဲမှုများကိုသာ hot-apply လုပ်သည်။ Restart လိုအပ်သည့်အခါ သတိပေးစာကို log ထုတ်ပေးသည် — ကိုယ်တိုင် ကိုင်တွယ်ရမည်။ |
| **`restart`**                             | Config ပြောင်းလဲမှု မည်သည့်အမျိုးအစားမဆို (လုံခြုံ/မလုံခြုံ) Gateway ကို restart လုပ်သည်။                     |
| **`off`**                                 | File watching ကို ပိတ်ထားသည်။ ပြောင်းလဲမှုများသည် နောက်တစ်ကြိမ် လက်ဖြင့် restart လုပ်သည့်အခါမှ အကျိုးသက်ရောက်မည်။                |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Hot-apply လုပ်နိုင်သောအရာများနှင့် restart လိုအပ်သောအရာများ

Field အများစုသည် downtime မရှိဘဲ hot-apply လုပ်နိုင်သည်။ `hybrid` mode တွင် restart လိုအပ်သော ပြောင်းလဲမှုများကို အလိုအလျောက် ကိုင်တွယ်သည်။

| အမျိုးအစား                       | Fields                                                                                       | Restart လိုအပ်ပါသလား? |
| -------------------------------- | -------------------------------------------------------------------------------------------- | --------------------- |
| Channels                         | `channels.*`, `web` (WhatsApp) — built-in နှင့် extension channel အားလုံး | မလိုအပ်ပါ             |
| Agent နှင့် model များ           | `agent`, `agents`, `models`, `routing`                                                       | မလိုအပ်ပါ             |
| အလိုအလျောက်လုပ်ဆောင်မှု          | `hooks`, `cron`, `agent.heartbeat`                                                           | မရှိ                  |
| Session များ နှင့် မက်ဆေ့ချ်များ | `session`, `messages`                                                                        | မရှိ                  |
| Tools နှင့် မီဒီယာ               | `tools`, `browser`, `skills`, `audio`, `talk`                                                | မရှိ                  |
| UI နှင့် အခြားအရာများ            | `ui`, `logging`, `identity`, `bindings`                                                      | မရှိ                  |
| Gateway ဆာဗာ                     | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                      | **ဟုတ်သည်**           |
| Infrastructure                   | `discovery`, `canvasHost`, `plugins`                                                         | **ဟုတ်သည်**           |

<Note>
`gateway.reload` နှင့် `gateway.remote` သည် ချွင်းချက်များဖြစ်သည် — ၎င်းတို့ကို ပြောင်းလဲပါက restart မဖြစ်ပါ။
</Note>

## Config RPC (programmatic updates)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Config အပြည့်အစုံကို စစ်ဆေးအတည်ပြုပြီး ရေးသားသိမ်းဆည်းကာ Gateway ကို တစ်ဆင့်တည်းဖြင့် restart ပြန်လုပ်ပါသည်။


    ````
    <Warning>
    `config.apply` သည် **config တစ်ခုလုံးကို** အစားထိုးပါသည်။ အပိုင်းပိုင်းပြင်ဆင်ရန် `config.patch` ကို အသုံးပြုပါ၊ သို့မဟုတ် key တစ်ခုချင်းစီအတွက် `openclaw config set` ကို အသုံးပြုပါ။
    
</Warning>
    
    Params:
    
    - `raw` (string) — config တစ်ခုလုံးအတွက် JSON5 payload
    - `baseHash` (optional) — `config.get` မှ ရရှိသော config hash (config ရှိပြီးသားဖြစ်ပါက လိုအပ်သည်)
    - `sessionKey` (optional) — restart ပြီးနောက် wake-up ping အတွက် session key
    - `note` (optional) — restart sentinel အတွက် မှတ်ချက်
    - `restartDelayMs` (optional) — restart မလုပ်မီ နှောင့်နှေးချိန် (မူလတန်ဖိုး 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # payload.hash ကို ရယူပါ
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    ရှိပြီးသား config ထဲသို့ အပိုင်းလိုက် update ကို ပေါင်းစည်းပါသည် (JSON merge patch semantics):


    ````
    - Objects များကို recursive ပေါင်းစည်းသည်
    - `null` သည် key ကို ဖျက်ပစ်သည်
    - Arrays များကို အစားထိုးသည်
    
    Params:
    
    - `raw` (string) — ပြောင်းလဲလိုသော key များသာ ပါဝင်သော JSON5
    - `baseHash` (required) — `config.get` မှ ရရှိသော config hash
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` နှင့် တူညီသည်
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## ပတ်ဝန်းကျင် variable များ

OpenClaw သည် parent process မှ env vars များအပြင် အောက်ပါအရာများမှလည်း ဖတ်ယူပါသည်:

- Env var နှင့်ညီမျှသော အရာ။
- `~/.openclaw/.env` (global fallback)

ဖိုင်နှစ်ခုစလုံးသည် ရှိပြီးသား env vars များကို override မလုပ်ပါ။ config ထဲတွင်လည်း inline env vars များကို သတ်မှတ်နိုင်သည်:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  ဖွင့်ထားပြီး မျှော်လင့်ထားသော key များ မသတ်မှတ်ထားပါက OpenClaw သည် သင့် login shell ကို run ပြီး လိုအပ်နေသော key များကိုသာ import လုပ်ပါသည်:


```json5
20. {
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

Env var နှင့်ညီမျှသော အရာ: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  မည်သည့် config string value မဆိုအတွင်း `${VAR_NAME}` ဖြင့် env vars များကို ကိုးကားနိုင်သည်:


```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

စည်းမျဉ်းများ:

- အက္ခရာကြီးသာ ကိုက်ညီသည်: `[A-Z_][A-Z0-9_]*`
- မရှိသော သို့မဟုတ် အလွတ်ဖြစ်သော vars များသည် load လုပ်စဉ်တွင် error ဖြစ်စေသည်
- စာသားအဖြစ် ထုတ်လိုပါက `$${VAR}` ဖြင့် escape လုပ်ပါ
- `$include` ဖိုင်များအတွင်းတွင်လည်း အသုံးပြုနိုင်သည်
- Inline အစားထိုးခြင်း: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

အပြည့်အစုံ ဦးစားပေးအစီအစဉ်နှင့် ရင်းမြစ်များအတွက် [Environment](/help/environment) ကို ကြည့်ပါ။

## အပြည့်အစုံ ကိုးကားချက်

အကွက်တိုင်းအလိုက် အသေးစိတ်ကိုးကားချက်အတွက် **[Configuration Reference](/gateway/configuration-reference)** ကို ကြည့်ပါ။

---

_ဆက်စပ်: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
