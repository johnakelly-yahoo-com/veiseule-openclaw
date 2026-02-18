---
title: "ဖွဲ့စည်းပြင်ဆင်ခြင်း"
---

# ဖွဲ့စည်းပြင်ဆင်ခြင်း

OpenClaw သည် `~/.openclaw/openclaw.json` မှ ရွေးချယ်နိုင်သော <Tooltip tip="JSON5 supports comments and trailing commas">**JSON5**</Tooltip> ဖွဲ့စည်းပြင်ဆင်မှုကို ဖတ်ရှုပါသည်။

ဖိုင်မရှိပါက OpenClaw သည် လုံခြုံသော မူလတန်ဖိုးများကို အသုံးပြုပါသည်။ ဖွဲ့စည်းပြင်ဆင်မှု ထည့်သွင်းရန် ပုံမှန်အကြောင်းရင်းများမှာ —

- Channel များကို ချိတ်ဆက်ပြီး ဘော့ကို မည်သူများ မက်ဆေ့ချ် ပို့နိုင်သည်ကို ထိန်းချုပ်ရန်  
- Model များ၊ tools များ၊ sandboxing သို့မဟုတ် automation (cron, hooks) ကို သတ်မှတ်ရန်  
- Sessions, media, networking သို့မဟုတ် UI ကို ချိန်ညှိရန်  

ရရှိနိုင်သော field များအားလုံးအတွက် [Configuration Reference](/gateway/configuration-reference) ကို ကြည့်ပါ။

<Tip>
**ဖွဲ့စည်းပြင်ဆင်မှု အသစ်ဖြစ်ပါသလား?** Interactive setup အတွက် `openclaw onboard` ကို စတင်အသုံးပြုပါ သို့မဟုတ် copy‑paste အပြည့်အစုံ configs များအတွက် [Configuration Examples](/gateway/configuration-examples) လမ်းညွှန်ကို ကြည့်ပါ။
</Tip>

## အနည်းဆုံး ဖွဲ့စည်းပြင်ဆင်မှု

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## ဖွဲ့စည်းပြင်ဆင်မှု ပြင်ဆင်ခြင်း

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # full setup wizard
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
    [http://127.0.0.1:18789](http://127.0.0.1:18789) ကို ဖွင့်ပြီး **Config** tab ကို အသုံးပြုပါ။  
    Control UI သည် config schema မှ form တစ်ခုကို render လုပ်ပေးပြီး **Raw JSON** editor ကိုလည်း ထောက်ပံ့ပေးပါသည်။
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` ကို တိုက်ရိုက် ပြင်ဆင်နိုင်ပါသည်။ Gateway သည် ဖိုင်ကို စောင့်ကြည့်နေပြီး ပြောင်းလဲမှုများကို အလိုအလျောက် အသုံးချပါသည် (ကြည့်ရန် — [hot reload](#config-hot-reload))။
  
</Tab>
</Tabs>

## တင်းကျပ်သော စစ်ဆေးခြင်း

<Warning>
OpenClaw သည် schema နှင့် အပြည့်အဝ ကိုက်ညီသော configuration များကိုသာ လက်ခံပါသည်။ မသိသော key များ၊ မမှန်ကန်သော type များ သို့မဟုတ် invalid တန်ဖိုးများရှိပါက Gateway သည် **စတင်ရန် ငြင်းဆိုမည်** ဖြစ်ပါသည်။ Root-level တွင် `$schema` (string) သာလျှင် ခြွင်းချက်ဖြစ်ပါသည်။
</Warning>

Validation မအောင်မြင်ပါက —

- Gateway မစတင်ပါ  
- Diagnostic commands များသာ အလုပ်လုပ်ပါသည် (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)  
- ပြဿနာများကို ကြည့်ရန် `openclaw doctor` ကို ပြုလုပ်ပါ  
- ပြုပြင်ရန် `openclaw doctor --fix` (သို့မဟုတ် `--yes`) ကို အသုံးပြုပါ  

## အများဆုံး လုပ်ဆောင်ချက်များ

<AccordionGroup>
  <Accordion title="Channel တစ်ခု စတင်သတ်မှတ်ခြင်း (WhatsApp, Telegram, Discord စသည်)">
    Channel တစ်ခုချင်းစီသည် `channels.<provider>` အောက်တွင် ကိုယ်ပိုင် config section ရှိပါသည်။ Setup အဆင့်များအတွက် channel စာမျက်နှာကို ကြည့်ပါ —

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Channel များအားလုံးတွင် DM policy ပုံစံတူ အသုံးပြုပါသည် —

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

  
</Accordion>

  <Accordion title="Model များ ရွေးချယ်ခြင်းနှင့် သတ်မှတ်ခြင်း">
    Primary model နှင့် optional fallbacks များကို သတ်မှတ်ပါ —

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

    - `agents.defaults.models` သည် model catalog ကို သတ်မှတ်ပြီး `/model` အတွက် allowlist အဖြစ်လည်း လုပ်ဆောင်ပါသည်။
    - Model reference များကို `provider/model` ပုံစံဖြင့် အသုံးပြုပါ (ဥပမာ `anthropic/claude-opus-4-6`)။
    - Chat အတွင်း model ပြောင်းရန် [Models CLI](/concepts/models) နှင့် fallback အပြုအမူများအတွက် [Model Failover](/concepts/model-failover) ကို ကြည့်ပါ။
    - Custom/self-hosted providers များအတွက် reference ထဲရှိ [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="ဘော့ကို မည်သူ မက်ဆေ့ချ် ပို့နိုင်သည်ကို ထိန်းချုပ်ခြင်း">
    DM access ကို channel တစ်ခုချင်းစီအလိုက် `dmPolicy` ဖြင့် ထိန်းချုပ်ပါသည် —

    - `"pairing"` (default): မသိသော sender များကို pairing code တစ်ကြိမ် ပေးပြီး အတည်ပြုရန် လိုအပ်ပါသည်  
    - `"allowlist"`: `allowFrom` (သို့မဟုတ် paired allow store) ထဲရှိ sender များသာ ခွင့်ပြုသည်  
    - `"open"`: ဝင်လာသော DM အားလုံးကို ခွင့်ပြုသည် (`allowFrom: ["*"]` လိုအပ်သည်)  
    - `"disabled"`: DM အားလုံးကို လျစ်လျူရှုသည်  

    Group များအတွက် `groupPolicy` + `groupAllowFrom` သို့မဟုတ် channel-specific allowlist များကို အသုံးပြုပါ။

    အသေးစိတ်အတွက် [full reference](/gateway/configuration-reference#dm-and-group-access) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Group chat mention gating သတ်မှတ်ခြင်း">
    Group message များသည် မူလအနေဖြင့် **mention လိုအပ်သည်** ဖြစ်ပါသည်။ Agent အလိုက် pattern များကို သတ်မှတ်နိုင်ပါသည် —

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

    - **Metadata mentions**: Platform ၏ native @-mentions (WhatsApp tap‑to‑mention, Telegram @bot စသည်)  
    - **Text patterns**: `mentionPatterns` အတွင်းရှိ regex pattern များ  
    - Channel override နှင့် self-chat mode အတွက် [full reference](/gateway/configuration-reference#group-chat-mention-gating) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Sessions နှင့် resets သတ်မှတ်ခြင်း">
    Sessions သည် စကားပြောဆက်လက်တည်ရှိမှုနှင့် isolation ကို ထိန်းချုပ်ပါသည် —

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recommended for multi-user
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` (shared) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Scoping နှင့် identity links အတွက် [Session Management](/concepts/session) ကို ကြည့်ပါ။
    - Field အားလုံးအတွက် [full reference](/gateway/configuration-reference#session) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Sandboxing ဖွင့်ခြင်း">
    Agent session များကို Docker container အတွင်း isolation ဖြင့် chạy လုပ်နိုင်ပါသည် —

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

    Image ကို အရင်တည်ဆောက်ပါ — `scripts/sandbox-setup.sh`

    အသေးစိတ်အတွက် [Sandboxing](/gateway/sandboxing) နှင့် [full reference](/gateway/configuration-reference#sandbox) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Heartbeat (periodic check-ins) သတ်မှတ်ခြင်း">
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

    - `every`: duration string (`30m`, `2h`)။ `0m` သတ်မှတ်ပါက ပိတ်မည်။  
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`  
    - အသေးစိတ်အတွက် [Heartbeat](/gateway/heartbeat) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Cron jobs သတ်မှတ်ခြင်း">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    အကျဉ်းချုပ်နှင့် CLI ဥပမာများအတွက် [Cron jobs](/automation/cron-jobs) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Webhooks (hooks) သတ်မှတ်ခြင်း">
    Gateway ပေါ်တွင် HTTP webhook endpoints များကို ဖွင့်ပါ —

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

    Mapping options နှင့် Gmail integration အတွက် [full reference](/gateway/configuration-reference#hooks) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Multi-agent routing သတ်မှတ်ခြင်း">
    Workspace နှင့် session များ သီးခြားထားသော agent များစွာကို chạy လုပ်နိုင်ပါသည် —

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

    Binding rules နှင့် per-agent access profiles များအတွက် [Multi-Agent](/concepts/multi-agent) နှင့် [full reference](/gateway/configuration-reference#multi-agent-routing) ကို ကြည့်ပါ။

  
</Accordion>

  <Accordion title="Config ကို ဖိုင်အများအပြား ခွဲခြင်း ($include)">
    ကြီးမားသော config များကို စုစည်းရန် `$include` ကို အသုံးပြုနိုင်ပါသည် —

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

    - **Single file**: ပါဝင်သည့် object ကို အစားထိုးသည်  
    - **Array of files**: အစဉ်လိုက် deep-merge လုပ်သည် (နောက်ပိုင်းသည် အနိုင်ရ)  
    - **Sibling keys**: include ပြီးနောက် merge လုပ်ပြီး override လုပ်နိုင်သည်  
    - **Nested includes**: အများဆုံး အဆင့် 10 ထိ ထောက်ပံ့သည်  
    - **Relative paths**: include လုပ်သော ဖိုင်အပေါ် မူတည်၍ resolve လုပ်သည်  
    - **Error handling**: ဖိုင်မရှိခြင်း၊ parse error၊ circular include များကို ရှင်းလင်းစွာ ပြသသည်  

  
</Accordion>
</AccordionGroup>

## Config hot reload

Gateway သည် `~/.openclaw/openclaw.json` ကို စောင့်ကြည့်နေပြီး ပြောင်းလဲမှုများကို အလိုအလျောက် အသုံးချပါသည် — များသောအားဖြင့် manual restart မလိုအပ်ပါ။

### Reload modes

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | လုံခြုံသော ပြောင်းလဲမှုများကို ချက်ချင်း အသုံးချပြီး အရေးကြီးသောအရာများအတွက် အလိုအလျောက် restart လုပ်သည်။ |
| **`hot`**              | လုံခြုံသော ပြောင်းလဲမှုများသာ အသုံးချသည်။ Restart လိုအပ်ပါက warning ကိုသာ log လုပ်သည်။ |
| **`restart`**          | Config ပြောင်းလဲမှု မည်သည့်အရာမဆို Gateway ကို restart လုပ်သည်။ |
| **`off`**              | File watching ကို ပိတ်ထားသည်။ နောက်တစ်ကြိမ် manual restart ပြုလုပ်သည့်အခါမှသာ အသက်ဝင်မည်။ |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Hot-apply နှင့် restart လိုအပ်သော အရာများ

Field အများစုသည် downtime မရှိဘဲ hot-apply လုပ်နိုင်ပါသည်။ `hybrid` mode တွင် restart လိုအပ်သော အရာများကို အလိုအလျောက် ကိုင်တွယ်ပေးပါသည်။

| Category            | Fields                                                               | Restart needed? |
| ------------------- | -------------------------------------------------------------------- | --------------- |
| Channels            | `channels.*`, `web` (WhatsApp) — built-in နှင့် extension channels အားလုံး | No              |
| Agent & models      | `agent`, `agents`, `models`, `routing`                               | No              |
| Automation          | `hooks`, `cron`, `agent.heartbeat`                                   | No              |
| Sessions & messages | `session`, `messages`                                                | No              |
| Tools & media       | `tools`, `browser`, `skills`, `audio`, `talk`                        | No              |
| UI & misc           | `ui`, `logging`, `identity`, `bindings`                              | No              |
| Gateway server      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Yes**         |
| Infrastructure      | `discovery`, `canvasHost`, `plugins`                                 | **Yes**         |

<Note>
`gateway.reload` နှင့် `gateway.remote` ကို ပြောင်းလဲခြင်းသည် restart မဖြစ်စေပါ။
</Note>

## Config RPC (programmatic updates)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Config အပြည့်အစုံကို validate + write လုပ်ပြီး Gateway ကို တစ်ကြိမ်တည်း restart လုပ်ပါသည်။

    <Warning>
    `config.apply` သည် **config အားလုံးကို အစားထိုး** ပါသည်။ Partial update အတွက် `config.patch` ကို သုံးပါ သို့မဟုတ် key တစ်ခုချင်းစီအတွက် `openclaw config set` ကို အသုံးပြုပါ။
    
</Warning>

    Params —

    - `raw` (string) — JSON5 payload (config အပြည့်အစုံ)
    - `baseHash` (optional) — `config.get` မှ config hash (ရှိပြီးသားဖြစ်လျှင် လိုအပ်)
    - `sessionKey` (optional) — restart ပြီးနောက် wake-up ping အတွက်
    - `note` (optional) — restart sentinel အတွက် မှတ်ချက်
    - `restartDelayMs` (optional) — restart မတိုင်မီ နှောင့်နှေးချိန် (default 2000)

    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    ရှိပြီးသား config ထဲသို့ partial update ကို merge လုပ်ပါသည် (JSON merge patch semantics) —

    - Objects များကို recursive merge လုပ်သည်  
    - `null` သည် key ကို ဖျက်သည်  
    - Arrays များကို အစားထိုးသည်  

    Params —

    - `raw` (string) — ပြောင်းလဲမည့် key များသာ ပါဝင်သော JSON5
    - `baseHash` (required) — `config.get` မှ config hash
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` နှင့် တူသည်

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  
</Accordion>
</AccordionGroup>

## Environment variables

OpenClaw သည် parent process မှ env vars များကို ဖတ်ယူပြီး ထို့အပြင် —

- လက်ရှိ working directory ထဲရှိ `.env` (ရှိပါက)
- `~/.openclaw/.env` (global fallback)

ဖိုင်နှစ်ခုလုံးသည် ရှိပြီးသား env vars များကို override မလုပ်ပါ။ Config အတွင်း inline env vars များကိုလည်း သတ်မှတ်နိုင်ပါသည် —

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  ဖွင့်ထားပြီး မျှော်မှန်းထားသော key များ မရှိပါက OpenClaw သည် login shell ကို run လုပ်ပြီး မရှိသေးသော key များကိုသာ import လုပ်ပါသည် —

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
  `${VAR_NAME}` ဖြင့် config string value မည်သည့်အရာမဆိုအတွင်း env vars များကို ကိုးကားနိုင်ပါသည် —

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Rules —

- Uppercase အမည်များသာ ကိုက်ညီသည်: `[A-Z_][A-Z0-9_]*`
- မရှိသော သို့မဟုတ် အလွတ် var များသည် load အချိန်တွင် error ဖြစ်စေသည်
- Literal output အတွက် `$${VAR}` ဖြင့် escape လုပ်ပါ
- `$include` ဖိုင်များအတွင်းလည်း အလုပ်လုပ်သည်
- Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

အသေးစိတ် precedence နှင့် sources များအတွက် [Environment](/help/environment) ကို ကြည့်ပါ။

## Full reference

Field တစ်ခုချင်းစီအတွက် အသေးစိတ် reference ကို **[Configuration Reference](/gateway/configuration-reference)** တွင် ကြည့်ပါ။

---

_Related: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

