---
title: "ဖွဲ့စည်းပြင်ဆင်ခြင်း"
---

# ဖွဲ့စည်းပြင်ဆင်ခြင်း 🔧

OpenClaw သည် `~/.openclaw/openclaw.json` မှ **JSON5** ဖွဲ့စည်းပြင်ဆင်မှု (မှတ်ချက်များ + နောက်ဆုံး ကော်မာများ ခွင့်ပြု) ကို ရွေးချယ်နိုင်သည့် အနေဖြင့် ဖတ်ရှုသည်။

If the file is missing, OpenClaw uses safe-ish defaults (embedded Pi agent + per-sender sessions + workspace `~/.openclaw/workspace`). You usually only need a config to:

- ဘော့ကို လှုံ့ဆော်နိုင်သူများကို ကန့်သတ်ရန် (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` စသည်)
- အုပ်စု allowlist များနှင့် mention အပြုအမူကို ထိန်းချုပ်ရန် (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- မက်ဆေ့ချ် prefix များကို စိတ်ကြိုက်ပြင်ဆင်ရန် (`messages`)
- agent ၏ အလုပ်ခွင်ကို သတ်မှတ်ရန် (`agents.defaults.workspace` သို့မဟုတ် `agents.list[].workspace`)
- ထည့်သွင်းထားသော agent ၏ မူလတန်ဖိုးများ (`agents.defaults`) နှင့် ဆက်ရှင် အပြုအမူ (`session`) ကို ချိန်ညှိရန်
- အေးဂျင့်တစ်ခုချင်းစီအလိုက် အထောက်အထားကို သတ်မှတ်ရန် (`agents.list[].identity`)

> **ဖွဲ့စည်းပြင်ဆင်ခြင်း အသစ်ဖြစ်ပါသလား?** အသေးစိတ်ရှင်းလင်းချက်များပါဝင်သည့် ပြည့်စုံသော ဥပမာများအတွက် [Configuration Examples](/gateway/configuration-examples) လမ်းညွှန်ကို ကြည့်ပါ။

## တင်းကျပ်သော ဖွဲ့စည်းပြင်ဆင်မှု စစ်ဆေးခြင်း

OpenClaw only accepts configurations that fully match the schema.
Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start** for safety.

စစ်ဆေးမှု မအောင်မြင်သည့်အခါ—

- Gateway မဖွင့်ပါ။
- ရောဂါရှာဖွေရေး အမိန့်များသာ ခွင့်ပြုသည် (ဥပမာ—`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`)။
- ပြဿနာများကို တိတိကျကျ ကြည့်ရန် `openclaw doctor` ကို ပြုလုပ်ပါ။
- ပြောင်းလဲမှု/ပြုပြင်မှုများကို လုပ်ဆောင်ရန် `openclaw doctor --fix` (သို့မဟုတ် `--yes`) ကို အသုံးပြုပါ။

Doctor သည် သင်က အတည်ပြု၍ `--fix`/`--yes` ကို ရွေးချယ်မထားပါက မည်သည့် ပြောင်းလဲမှုကိုမျှ မရေးသားပါ။

## Schema + UI အညွှန်းများ

The Gateway exposes a JSON Schema representation of the config via `config.schema` for UI editors.
The Control UI renders a form from this schema, with a **Raw JSON** editor as an escape hatch.

ချန်နယ် ပလဂင်များနှင့် တိုးချဲ့မှုများသည် ၎င်းတို့၏ ဖွဲ့စည်းပြင်ဆင်မှုအတွက် schema + UI အညွှန်းများကို မှတ်ပုံတင်နိုင်ပြီး၊ အက်ပ်များအနှံ့တွင် schema ကို အခြေခံထားသည့် ဆက်တင်များကို hard-coded ဖောင်များ မလိုအပ်ဘဲ ထိန်းသိမ်းနိုင်သည်။

အညွှန်းများ (တံဆိပ်များ၊ အုပ်စုခွဲခြင်း၊ အရေးကြီး အကွက်များ) ကို schema နှင့်အတူ ပို့ပေးသဖြင့် ကလိုင်းယင့်များသည် ဖွဲ့စည်းပြင်ဆင်မှု အကြောင်းအရာကို hard-code မလုပ်ဘဲ ပိုမိုကောင်းမွန်သော ဖောင်များကို ပြသနိုင်သည်။

## အသုံးချ + ပြန်လည်စတင်ခြင်း (RPC)

Use `config.apply` to validate + write the full config and restart the Gateway in one step.
It writes a restart sentinel and pings the last active session after the Gateway comes back.

Warning: `config.apply` replaces the **entire config**. If you want to change only a few keys,
use `config.patch` or `openclaw config set`. Keep a backup of `~/.openclaw/openclaw.json`.

ပါရာမီတာများ—

- `raw` (string) — ဖွဲ့စည်းပြင်ဆင်မှု အပြည့်အစုံအတွက် JSON5 payload
- `baseHash` (ရွေးချယ်နိုင်) — `config.get` မှ config hash (ဖွဲ့စည်းပြင်ဆင်မှု ရှိပြီးသားဖြစ်လျှင် လိုအပ်)
- `sessionKey` (ရွေးချယ်နိုင်) — wake-up ping အတွက် နောက်ဆုံး အသုံးပြုနေသည့် ဆက်ရှင် ကီး
- `note` (ရွေးချယ်နိုင်) — restart sentinel တွင် ထည့်သွင်းမည့် မှတ်ချက်
- `restartDelayMs` (ရွေးချယ်နိုင်) — ပြန်လည်စတင်မည့် အချိန် နှောင့်နှေးမှု (မူလ 2000)

ဥပမာ (`gateway call` မှတစ်ဆင့်)—

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## အစိတ်အပိုင်း အပ်ဒိတ်များ (RPC)

1. ဆက်စပ်မှုမရှိသော key များကို မဖျက်ဆီးဘဲ ရှိပြီးသား config ထဲသို့ အပိုင်းလိုက် အပြောင်းအလဲကို ပေါင်းထည့်ရန် `config.patch` ကို အသုံးပြုပါ။ 2. ၎င်းသည် JSON merge patch အဓိပ္ပါယ်ဖော်ပြချက်များကို အသုံးချပါသည်။

- object များကို recursive ပေါင်းစည်းသည်
- `null` သည် ကီးကို ဖျက်သည်
- array များကို အစားထိုးသည်  
  `config.apply` ကဲ့သို့ပင် စစ်ဆေးခြင်း၊ ရေးသားခြင်း၊ restart sentinel သိမ်းဆည်းခြင်းနှင့် Gateway ပြန်လည်စတင်မှုကို အချိန်ဇယားချသည် (`sessionKey` ပေးထားလျှင် wake လုပ်နိုင်သည်)။

ပါရာမီတာများ—

- `raw` (string) — ပြောင်းလဲမည့် ကီးများသာ ပါဝင်သည့် JSON5 payload
- `baseHash` (လိုအပ်) — `config.get` မှ config hash
- `sessionKey` (ရွေးချယ်နိုင်) — wake-up ping အတွက် နောက်ဆုံး ဆက်ရှင် ကီး
- `note` (ရွေးချယ်နိုင်) — restart sentinel အတွက် မှတ်ချက်
- `restartDelayMs` (ရွေးချယ်နိုင်) — ပြန်လည်စတင်မည့် အချိန် နှောင့်နှေးမှု (မူလ 2000)

ဥပမာ—

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## အနည်းဆုံး ဖွဲ့စည်းပြင်ဆင်မှု (အကြံပြုထားသော စတင်ချက်)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

မူလ image ကို တစ်ကြိမ် တည်ဆောက်ရန်—

```bash
scripts/sandbox-setup.sh
```

## ကိုယ်တိုင်-ချက် မုဒ် (အုပ်စု ထိန်းချုပ်မှုအတွက် အကြံပြု)

အုပ်စုများတွင် WhatsApp @-mentions များကို ဘော့က မတုံ့ပြန်စေရန် (တိကျသော စာသား trigger များသာ တုံ့ပြန်စေရန်)—

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

## Config Includes (`$include`)

3. `$include` ညွှန်ကြားချက်ကို အသုံးပြုပြီး သင့် config ကို ဖိုင်အများအပြားအဖြစ် ခွဲထားနိုင်ပါသည်။ 4. ဤအရာသည် အောက်ပါအတွက် အသုံးဝင်ပါသည်။

- ကြီးမားသော ဖွဲ့စည်းပြင်ဆင်မှုများကို စီမံရန် (ဥပမာ—client တစ်ခုချင်းစီအလိုက် agent သတ်မှတ်ချက်များ)
- ပတ်ဝန်းကျင်များအကြား အများသုံး ဆက်တင်များကို မျှဝေရန်
- အရေးကြီးသော ဖွဲ့စည်းပြင်ဆင်မှုများကို ခွဲထားရန်

### အခြေခံ အသုံးပြုနည်း

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

### ပေါင်းစည်းမှု အပြုအမူ

- **ဖိုင်တစ်ဖိုင်**: `$include` ပါဝင်သည့် object ကို အစားထိုးသည်
- **ဖိုင် array**: အစဉ်လိုက် Deep-merge လုပ်သည် (နောက်ပိုင်းဖိုင်များက အရင်ဖိုင်များကို အစားထိုး)
- **Sibling keys ပါရှိလျှင်**: include ပြီးနောက် sibling keys များကို ပေါင်းစည်းသည် (include တန်ဖိုးများကို အစားထိုး)
- **Sibling keys + arrays/primitives**: မထောက်ပံ့ပါ (include လုပ်သော အကြောင်းအရာသည် object ဖြစ်ရမည်)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Nested includes

Include လုပ်ထားသော ဖိုင်များတွင်လည်း `$include` ညွှန်ကြားချက်များ ပါဝင်နိုင်သည် (အများဆုံး အဆင့် 10)—

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### လမ်းကြောင်း ဖြေရှင်းခြင်း

- **Relative paths**: include လုပ်သော ဖိုင်၏ လမ်းကြောင်းအပေါ် မူတည်၍ ဖြေရှင်းသည်
- **Absolute paths**: အတိုင်းအတာမပြောင်းဘဲ အသုံးပြုသည်
- **Parent directories**: `../` ကိုးကားချက်များ အလုပ်လုပ်သည်

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### အမှား ကိုင်တွယ်ခြင်း

- **ဖိုင် မရှိပါ**: ဖြေရှင်းထားသော လမ်းကြောင်းနှင့်အတူ အမှားကို ထုတ်ပြသည်
- **Parse အမှား**: include လုပ်ထားသော မည်သည့်ဖိုင် မအောင်မြင်သည်ကို ပြသသည်
- **Circular includes**: include ချိတ်ဆက်စဉ်ကို ထောက်လှမ်းပြီး အစီရင်ခံသည်

### ဥပမာ—ဖောက်သည်အများအတွက် ဥပဒေရေးရာ စနစ်တကျ ပြင်ဆင်မှု

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

## Common options

### 5. Env vars + `.env`

6. OpenClaw သည် မိခင် process (shell, launchd/systemd, CI စသည်) မှ env vars များကို ဖတ်ယူပါသည်။

7. ထို့အပြင် ၎င်းသည် အောက်ပါတို့ကိုလည်း load လုပ်ပါသည်။

- 8. လက်ရှိ working directory ထဲမှ `.env` (ရှိပါက)
- 9. `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`) မှ global fallback `.env`

10. `.env` ဖိုင်နှစ်ခုလုံးသည် ရှိပြီးသား env vars များကို override မလုပ်ပါ။

11. config ထဲတွင် inline env vars များကိုလည်း ပေးနိုင်ပါသည်။ 12. process env တွင် key မရှိပါကသာ (override မလုပ်သည့် စည်းမျဉ်းတူညီစွာဖြင့်) အသုံးချပါသည်။

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

13. အပြည့်အစုံ precedence နှင့် sources များအတွက် [/environment](/help/environment) ကို ကြည့်ပါ။

### 14. `env.shellEnv` (optional)

15. Opt-in အဆင်ပြေမှုအဖြစ်—ဖွင့်ထားပြီး မျှော်မှန်းထားသော key များထဲမှ မည်သည့် key မှ မသတ်မှတ်ရသေးပါက—OpenClaw သည် သင့် login shell ကို run လုပ်ပြီး လိုအပ်နေသည့် key များကိုသာ import လုပ်ပါသည် (ဘယ်တော့မှ override မလုပ်ပါ)။
16. ၎င်းသည် သင့် shell profile ကို source လုပ်သကဲ့သို့ ဖြစ်ပါသည်။

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

17. Env var နှင့်ညီမျှသော အရာ။

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### Config အတွင်း Env var အစားထိုးခြင်း

18. `${VAR_NAME}` syntax ကို အသုံးပြုပြီး မည်သည့် config string value မဆိုအတွင်း environment variables များကို တိုက်ရိုက် ကိုးကားနိုင်ပါသည်။ Variables များကို config load လုပ်ချိန်တွင် validation မလုပ်မီ အစားထိုးပါသည်။

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

21. **စည်းမျဉ်းများ:**

- 22. အကြီးစာလုံး env var အမည်များကိုသာ ကိုက်ညီစေပါသည်: `[A-Z_][A-Z0-9_]*`
- မရှိသော သို့မဟုတ် အလွတ် env vars များသည် config load လုပ်ချိန်တွင် error ဖြစ်စေပါသည်။
- 24. စာသားအဖြစ် `${VAR}` ကို ထုတ်လိုပါက `$${VAR}` ဖြင့် escape လုပ်ပါ။
- 25. `$include` နှင့်အတူ အလုပ်လုပ်ပါသည် (include လုပ်ထားသော ဖိုင်များလည်း substitution ရရှိပါသည်)။

26. **Inline substitution:**

```json5
27. {
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### 28. Auth storage (OAuth + API keys)

29. OpenClaw သည် **agent တစ်ခုချင်းစီအလိုက်** auth profiles (OAuth + API keys) များကို အောက်ပါနေရာတွင် သိမ်းဆည်းပါသည်။

- 30. `<agentDir>/auth-profiles.json` (default: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

31. ထို့အပြင် [/concepts/oauth](/concepts/oauth) ကို ကြည့်ပါ။

32. Legacy OAuth imports:

- 33. `~/.openclaw/credentials/oauth.json` (သို့မဟုတ် `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

34. Embedded Pi agent သည် runtime cache ကို အောက်ပါနေရာတွင် ထိန်းသိမ်းထားပါသည်။

- 35. `<agentDir>/auth.json` (အလိုအလျောက် စီမံခန့်ခွဲထားသည်; လက်ဖြင့် မတည်းဖြတ်ပါနှင့်)

36. Legacy agent dir (multi-agent မတိုင်မီ):

- 37. `~/.openclaw/agent/*` (`openclaw doctor` ဖြင့် `~/.openclaw/agents/<defaultAgentId>/agent/*` သို့ migrate လုပ်ထားသည်)

38. Overrides:

- 39. OAuth dir (legacy import အတွက်သာ): `OPENCLAW_OAUTH_DIR`
- 40. Agent dir (default agent root override): `OPENCLAW_AGENT_DIR` (ဦးစားပေး), `PI_CODING_AGENT_DIR` (legacy)

41. ပထမဆုံး အသုံးပြုချိန်တွင် OpenClaw သည် `oauth.json` entries များကို `auth-profiles.json` ထဲသို့ import လုပ်ပါသည်။

### `auth`

42. Auth profiles အတွက် optional metadata။ 43. ၎င်းသည် လျှို့ဝှက်ချက်များကို **မသိမ်းဆည်းပါ**; profile IDs များကို provider + mode (နှင့် optional email) သို့ ချိတ်ဆက်ပေးပြီး failover အတွက် အသုံးပြုမည့် provider rotation order ကို သတ်မှတ်ပါသည်။

```json5
44. {
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

45. Defaults နှင့် UX အတွက် အသုံးပြုသည့် optional per-agent identity။ 46. ၎င်းကို macOS onboarding assistant မှ ရေးသားပါသည်။

47. သတ်မှတ်ထားပါက (သင် ကိုယ်တိုင် အထူးသတ်မှတ်မထားသေးသောအခါတွင်သာ) OpenClaw သည် defaults များကို ဆင်းသက်တွက်ချက်ပါသည်။

- 48. **active agent** ၏ `identity.emoji` မှ `messages.ackReaction` ကို ယူပါသည် (မရှိပါက 👀 သို့ fallback လုပ်ပါသည်)။
- 49. Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp တစ်လျှောက် group များတွင် “@Samantha” ကဲ့သို့ အလုပ်လုပ်စေရန် agent ၏ `identity.name`/`identity.emoji` မှ `agents.list[].groupChat.mentionPatterns` ကို ယူပါသည်။
- 50. `identity.avatar` သည် workspace-relative image path သို့မဟုတ် remote URL/data URL ကို လက်ခံပါသည်။ Local files must live inside the agent workspace.

`identity.avatar` accepts:

- Workspace-relative path (must stay within the agent workspace)
- `http(s)` URL
- `data:` URI

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

Metadata written by CLI wizards (`onboard`, `configure`, `doctor`).

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

- Default log file: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- If you want a stable path, set `logging.file` to `/tmp/openclaw/openclaw.log`.
- Console output ကို အောက်ပါအတိုင်း သီးခြား ချိန်ညှိနိုင်ပါသည်။
  - `logging.consoleLevel` (defaults to `info`, bumps to `debug` when `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- Tool summaries can be redacted to avoid leaking secrets:
  - `logging.redactSensitive` (`off` | `tools`, default: `tools`)
  - `logging.redactPatterns` (array of regex strings; overrides defaults)

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

Controls how WhatsApp direct chats (DMs) are handled:

- `"pairing"` (default): unknown senders get a pairing code; owner must approve
- `"allowlist"`: only allow senders in `channels.whatsapp.allowFrom` (or paired allow store)
- `"open"`: allow all inbound DMs (**requires** `channels.whatsapp.allowFrom` to include `"*"`)
- `"disabled"`: ignore all inbound DMs

Pairing codes expire after 1 hour; the bot only sends a pairing code when a new request is created. Pending DM pairing requests များကို ပုံမှန်အားဖြင့် **channel တစ်ခုလျှင် 3 ခု** အထိသာ ခွင့်ပြုထားပါသည်။

Pairing approvals:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

Allowlist of E.164 phone numbers that may trigger WhatsApp auto-replies (**DMs only**).
If empty and `channels.whatsapp.dmPolicy="pairing"`, unknown senders will receive a pairing code.
For groups, use `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

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

Inbound WhatsApp မက်ဆေ့ချ်များကို read (blue ticks) အဖြစ် အမှတ်အသားပြုမည်ကို ထိန်းချုပ်ပါသည်။ Default: `true`.

Self-chat mode always skips read receipts, even when enabled.

Per-account override: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (multi-account)

Run multiple WhatsApp accounts in one gateway:

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

မှတ်ချက်များ —

- Outbound commands default to account `default` if present; otherwise the first configured account id (sorted).
- The legacy single-account Baileys auth dir is migrated by `openclaw doctor` into `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Run multiple accounts per channel (each account has its own `accountId` and optional `name`):

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

မှတ်ချက်များ —

- `default` is used when `accountId` is omitted (CLI + routing).
- Env tokens only apply to the **default** account.
- Base channel settings (group policy, mention gating, etc.) apply to all accounts unless overridden per account.
- `bindings[].match.accountId` ကို အသုံးပြုပြီး account တစ်ခုချင်းစီကို မတူညီသော `agents.defaults` သို့ route လုပ်ပါ။

### Group chat mention gating (`agents.list[].groupChat` + `messages.groupChat`)

Group message များသည် default အနေဖြင့် **mention လိုအပ်သည်** (metadata mention သို့မဟုတ် regex pattern များ)။ WhatsApp, Telegram, Discord, Google Chat နှင့် iMessage group chats များတွင် အသုံးချပါသည်။

**Mention အမျိုးအစားများ:**

- **Metadata mentions**: Platform အလိုက် native @-mention များ (ဥပမာ WhatsApp tap-to-mention)။ WhatsApp self-chat mode တွင် ignore လုပ်ပါသည် (`channels.whatsapp.allowFrom` ကို ကြည့်ပါ)။
- **Text patterns**: `agents.list[].groupChat.mentionPatterns` တွင် သတ်မှတ်ထားသော Regex pattern များ။ Self-chat mode မည်သို့ဖြစ်စေ အမြဲစစ်ဆေးပါသည်။
- Mention detection ပြုလုပ်နိုင်သောအခါ (native mentions သို့မဟုတ် `mentionPattern` အနည်းဆုံးတစ်ခု ရှိသောအခါ) တွင်သာ mention gating ကို အတည်ပြုလုပ်ဆောင်ပါသည်။

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

`messages.groupChat.historyLimit` သည် group history context အတွက် global default ကို သတ်မှတ်ပါသည်။ Channels များသည် `channels.<channel>` ဖြင့် override လုပ်နိုင်ပါသည်`.historyLimit` (multi-account အတွက် `channels.<channel>``.accounts.*.historyLimit`)။ `0` ကို သတ်မှတ်ပါက history wrapping ကို disable လုပ်ပါသည်။

#### DM history limit များ

DM conversation များသည် agent မှ စီမံခန့်ခွဲသော session-based history ကို အသုံးပြုပါသည်။ DM session တစ်ခုချင်းစီတွင် ထိန်းသိမ်းထားမည့် user turn အရေအတွက်ကို ကန့်သတ်နိုင်ပါသည်။

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // DM sessions ကို user turn 30 ခုအထိ ကန့်သတ်
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

Resolution အစဉ်လိုက်:

1. Per-DM override: `channels.<provider>``.dms[userId].historyLimit`
2. Provider default: `channels.<provider>``.dmHistoryLimit`
3. Limit မရှိပါ (history အားလုံးကို ထိန်းသိမ်းထားသည်)။

Supported provider များ: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`။

Per-agent override (သတ်မှတ်ထားပါက precedence ယူပါသည်၊ `[]` ဖြစ်သော်လည်း)။

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

Mention gating default များသည် channel အလိုက် သတ်မှတ်ထားပါသည် (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`)။ `*.groups` ကို သတ်မှတ်ထားပါက group allowlist အဖြစ်လည်း လုပ်ဆောင်ပါသည်; group အားလုံးကို ခွင့်ပြုရန် `"*"` ကို ထည့်ပါ။

သတ်မှတ်ထားသော text trigger များကိုသာ **တုံ့ပြန်ရန်** (native @-mention များကို ignore လုပ်ရန်):

```json5
{
  channels: {
    whatsapp: {
      // Self-chat mode ကို enable လုပ်ရန် သင့်ကိုယ်ပိုင် number ကို ထည့်ပါ (native @-mention များကို ignore လုပ်သည်)။
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // ဒီ text pattern များသာ response ကို trigger လုပ်ပါမည်
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### Group policy (channel အလိုက်)

`channels.*.groupPolicy` ကို အသုံးပြုပြီး group/room message များကို လက်ခံမလားကို ထိန်းချုပ်ပါ။

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

မှတ်ချက်များ-

- `"open"`: groups များသည် allowlist ကို ကျော်လွန်နိုင်သည်; mention-gating သည် ဆက်လက် အသုံးပြုပါသည်။
- `"disabled"`: group/room message အားလုံးကို ပိတ်ထားသည်။
- `"allowlist"`: သတ်မှတ်ထားသော allowlist နှင့် ကိုက်ညီသော group/room များကိုသာ ခွင့်ပြုသည်။
- `channels.defaults.groupPolicy` သည် provider ၏ `groupPolicy` ကို မသတ်မှတ်ထားသောအခါ default ကို သတ်မှတ်ပါသည်။
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams များသည် `groupAllowFrom` ကို အသုံးပြုပါသည် (fallback: explicit `allowFrom`)။
- Discord/Slack များသည် channel allowlist များကို အသုံးပြုပါသည် (`channels.discord.guilds.*.channels`, `channels.slack.channels`)။
- Group DM များ (Discord/Slack) သည် `dm.groupEnabled` + `dm.groupChannels` ဖြင့် ဆက်လက် ထိန်းချုပ်ပါသည်။
- Default သည် `groupPolicy: "allowlist"` ဖြစ်ပါသည် (`channels.defaults.groupPolicy` ဖြင့် override မလုပ်ထားပါက); allowlist မသတ်မှတ်ထားပါက group message များကို ပိတ်ထားပါသည်။

### Multi-agent routing (`agents.list` + `bindings`)

Gateway တစ်ခုအတွင်း agent များစွာကို သီးခြားခွဲထားပြီး (workspace, `agentDir`, sessions များ သီးခြား) run လုပ်နိုင်ပါသည်။
Inbound မက်ဆေ့ချ်များကို bindings များမှတဆင့် agent တစ်ဦးထံသို့ route လုပ်ပါသည်။

- `agents.list[]`: agent အလိုက် override များ။
  - `id`: stable agent id (မဖြစ်မနေ လိုအပ်သည်)။
  - `default`: optional ဖြစ်ပါသည်; အများအပြား သတ်မှတ်ထားပါက ပထမတစ်ခုကို အသုံးပြုပြီး warning ကို log တွင် မှတ်တမ်းတင်ပါသည်။
    If none are set, the **first entry** in the list is the default agent.
  - `name`: display name for the agent.
  - `workspace`: default `~/.openclaw/workspace-<agentId>` (for `main`, falls back to `agents.defaults.workspace`).
  - `agentDir`: default `~/.openclaw/agents/<agentId>/agent`.
  - `model`: per-agent default model, overrides `agents.defaults.model` for that agent.
    - string form: `"provider/model"`, overrides only `agents.defaults.model.primary`
    - object form: `{ primary, fallbacks }` (fallbacks override `agents.defaults.model.fallbacks`; `[]` disables global fallbacks for that agent)
  - `identity`: per-agent name/theme/emoji (used for mention patterns + ack reactions).
  - `groupChat`: per-agent mention-gating (`mentionPatterns`).
  - `sandbox`: per-agent sandbox config (overrides `agents.defaults.sandbox`).
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: custom sandbox workspace root
    - `docker`: per-agent docker overrides (e.g. `image`, `network`, `env`, `setupCommand`, limits; ignored when `scope: "shared"`)
    - `browser`: per-agent sandboxed browser overrides (ignored when `scope: "shared"`)
    - `prune`: per-agent sandbox pruning overrides (ignored when `scope: "shared"`)
  - `subagents`: per-agent sub-agent defaults.
    - `allowAgents`: allowlist of agent ids for `sessions_spawn` from this agent (`["*"]` = allow any; default: only same agent)
  - `tools`: per-agent tool restrictions (applied before sandbox tool policy).
    - `profile`: base tool profile (applied before allow/deny)
    - `allow`: array of allowed tool names
    - `deny`: array of denied tool names (deny wins)
- `agents.defaults`: shared agent defaults (model, workspace, sandbox, etc.).
- `bindings[]`: routes inbound messages to an `agentId`.
  - `match.channel` (required)
  - `match.accountId` (optional; `*` = any account; omitted = default account)
  - `match.peer` (optional; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (optional; channel-specific)

Deterministic match order:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (exact, no peer/guild/team)
5. `match.accountId: "*"` (channel-wide, no peer/guild/team)
6. default agent (`agents.list[].default`, else first list entry, else `"main"`)

Within each match tier, the first matching entry in `bindings` wins.

#### Agent တစ်ခုချင်းစီအလိုက် ဝင်ရောက်ခွင့် ပရိုဖိုင်များ (multi-agent)

Each agent can carry its own sandbox + tool policy. Use this to mix access
levels in one gateway:

- **အပြည့်အဝ အသုံးပြုခွင့်** (personal agent)
- **Read-only** tools + workspace
- **No filesystem access** (messaging/session tools only)

See [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) for precedence and
additional examples.

Full access (no sandbox):

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

Read-only tools + read-only workspace:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
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

No filesystem access (messaging/session tools enabled):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
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

Example: two WhatsApp accounts → two agents:

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
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (ရွေးချယ်နိုင်)

Agent များအကြား မက်ဆေ့ချ်ပို့ခြင်းသည် opt‑in ဖြစ်ပါသည်။

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

### `messages.queue`

Agent run တစ်ခု အလုပ်လုပ်နေပြီးသား ဖြစ်သည့်အခါ ဝင်လာသော မက်ဆေ့ချ်များ၏ အပြုအမူကို ထိန်းချုပ်သည်။

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

**ပို့သူတူညီသူ** မှ အလျင်အမြန် ဝင်လာသော မက်ဆေ့ချ်များကို debounce လုပ်ပြီး ဆက်တိုက်ပို့သော မက်ဆေ့ချ်များကို agent turn တစ်ခုအဖြစ် ပေါင်းစည်းသည်။ Debouncing သည် channel + conversation အလိုက် ခွဲခြားထားပြီး အဖြေပြန်ပို့ရန် threading/IDs အတွက် နောက်ဆုံး မက်ဆေ့ချ်ကို အသုံးပြုသည်။

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

မှတ်ချက်များ-

- **စာသားသာ ပါသော** မက်ဆေ့ချ် အစုများကိုသာ debounce လုပ်သည်။ မီဒီယာ/attachment များသည် ချက်ချင်း flush လုပ်မည်။
- Control commands (ဥပမာ `/queue`, `/new`) များသည် debounce ကို ကျော်လွှားပြီး သီးသန့် မက်ဆေ့ချ်အဖြစ် ရှိနေမည်။

### `commands` (chat command ကို ကိုင်တွယ်ခြင်း)

Connectors အားလုံးတွင် chat commands များကို မည်သို့ enable လုပ်မည်ကို ထိန်းချုပ်ပါသည်။

```json5
{
  commands: {
    native: "auto", // register native commands when supported (auto)
    text: true, // parse slash commands in chat messages
    bash: false, // allow ! (alias: /bash) (host-only; requires tools.elevated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false, // allow /config (writes to disk)
    debug: false, // allow /debug (runtime-only overrides)
    restart: false, // allow /restart + gateway restart tool
    useAccessGroups: true, // enforce access-group allowlists/policies for commands
  },
}
```

မှတ်ချက်များ-

- Text command များကို **သီးသန့်** မက်ဆေ့ချ်အဖြစ် ပို့ရပြီး ရှေ့တွင် `/` ကို အသုံးပြုရမည် (plain-text alias မရှိပါ)။
- `commands.text: false` သည် chat မက်ဆေ့ချ်များထဲမှ command များကို parse မလုပ်စေရန် ပိတ်ထားသည်။
- `commands.native: "auto"` (default) သည် Discord/Telegram အတွက် native command များကို ဖွင့်ပြီး Slack ကို ပိတ်ထားသည်။ မထောက်ပံ့သော channel များသည် text-only အဖြစ် ဆက်လက်ရှိနေမည်။
- `commands.native: true|false` ကို သတ်မှတ်၍ အားလုံးကို အတင်းအကျပ် ဖွင့်/ပိတ်နိုင်ပြီး၊ `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool သို့မဟုတ် `"auto"`) ဖြင့် channel အလိုက် override လုပ်နိုင်သည်။ `false` သတ်မှတ်ပါက startup အချိန်တွင် Discord/Telegram တွင် ယခင် register လုပ်ထားသော command များကို ဖျက်ရှင်းမည်။ Slack command များကို Slack app ထဲတွင် စီမံခန့်ခွဲသည်။
- `channels.telegram.customCommands` သည် Telegram bot menu entry အသစ်များကို ထည့်ပေါင်းသည်။ နာမည်များကို normalize လုပ်ထားပြီး native command များနှင့် ပဋိပက္ခ ဖြစ်ပါက လျစ်လျူရှုမည်။
- `commands.bash: true` သည် `! <cmd>` ကို ဖွင့်ပြီး host shell command များကို run နိုင်စေသည် (`/bash <cmd>` ကို alias အဖြစ်လည်း အသုံးပြုနိုင်သည်)။ `tools.elevated.enabled` ကို လိုအပ်ပြီး ပို့သူကို `tools.elevated.allowFrom.<channel>` တွင် allowlist လုပ်ထားရမည်။ .`commands.bashForegroundMs` သည် bash ကို background သို့ မပြောင်းမီ စောင့်ဆိုင်းချိန်ကို ထိန်းချုပ်သည်။
- bash job တစ်ခု အလုပ်လုပ်နေစဉ် `! <cmd>` တောင်းဆိုမှု အသစ်များကို ပယ်ချမည် (တစ်ကြိမ်လျှင် တစ်ခုသာ)။ `commands.config: true` သည် `/config` ကို ဖွင့်ပြီး (`openclaw.json` ကို ဖတ်/ရေး) ခွင့်ပြုသည်။ `channels.<provider>`
- `.configWrites` သည် ထို channel မှ စတင်သော config ပြောင်းလဲမှုများကို ကန့်သတ်သည် (default: true)။
- ဤသည်သည် `/config set|unset` နှင့် provider-specific auto-migration များ (Telegram supergroup ID ပြောင်းလဲမှု၊ Slack channel ID ပြောင်းလဲမှု) ကို သက်ရောက်သည်။`commands.debug: true` သည် `/debug` ကို ဖွင့်သည် (runtime-only override များ)။ `commands.restart: true` သည် `/restart` နှင့် gateway tool restart action ကို ဖွင့်သည်။
- `commands.useAccessGroups: false` သည် access-group allowlist/policy များကို ကျော်လွှားပြီး command များကို ခွင့်ပြုသည်။
- Slash command များနှင့် directive များကို **ခွင့်ပြုထားသော ပို့သူများ** အတွက်သာ လက်ခံသည်။
- Authorization သည် channel allowlist/pairing နှင့် `commands.useAccessGroups` ပေါ်မူတည်၍ သတ်မှတ်သည်။
- `web` (WhatsApp web channel runtime) WhatsApp သည် gateway ၏ web channel (Baileys Web) မှတစ်ဆင့် လည်ပတ်သည်။

### Link လုပ်ထားသော session ရှိပါက အလိုအလျောက် စတင်မည်။

`web.enabled: false` ကို သတ်မှတ်ပါက default အနေဖြင့် ပိတ်ထားမည်။ {
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
`channels.telegram` (bot transport)

```json5
`channels.telegram` config section ရှိမှသာ OpenClaw သည် Telegram ကို စတင်မည်။
```

### Bot token ကို `channels.telegram.botToken` (သို့မဟုတ် `channels.telegram.tokenFile`) မှ ရယူပြီး default account အတွက် `TELEGRAM_BOT_TOKEN` ကို fallback အဖြစ် အသုံးပြုသည်။

`channels.telegram.enabled: false` ကို သတ်မှတ်ပါက အလိုအလျောက် စတင်ခြင်းကို ပိတ်မည်။ Multi-account support ကို `channels.telegram.accounts` အောက်တွင် ထားရှိသည် (အထက်ပါ multi-account section ကို ကြည့်ပါ)။
Env token များသည် default account အတွက်သာ သက်ရောက်သည်။
`channels.telegram.configWrites: false` ကို သတ်မှတ်ပါက Telegram မှ စတင်သော config write များ (supergroup ID migration နှင့် `/config set|unset` အပါအဝင်) ကို ပိတ်ဆို့မည်။ Env tokens only apply to the default account.
Set `channels.telegram.configWrites: false` to block Telegram-initiated config writes (including supergroup ID migrations and `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

Draft streaming ဆိုင်ရာ မှတ်စုများ:

- Telegram `sendMessageDraft` ကို အသုံးပြုသည် (draft bubble ဖြစ်ပြီး တကယ့် message မဟုတ်ပါ)။
- **private chat topics** လိုအပ်သည် (DM များတွင် message_thread_id; bot တွင် topics ဖွင့်ထားရမည်)။
- `/reasoning stream` သည် reasoning ကို draft ထဲသို့ stream လုပ်ပြီး နောက်ဆုံးအဖြေကို ပို့ပေးသည်။
  Retry policy ၏ default တန်ဖိုးများနှင့် အပြုအမူများကို [Retry policy](/concepts/retry) တွင် မှတ်တမ်းတင်ထားသည်။

### `channels.discord` (bot transport)

Discord bot ကို bot token နှင့် optional gating များကို သတ်မှတ်ခြင်းဖြင့် configure လုပ်ပါ။
Multi-account support သည် `channels.discord.accounts` အောက်တွင် ရှိသည် (အထက်ပါ multi-account အပိုင်းကို ကြည့်ပါ)။ Env tokens များသည် default account အတွက်သာ သက်ဆိုင်ပါသည်။

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
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
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw သည် `channels.discord` config အပိုင်း ရှိမှသာ Discord ကို စတင်ပါသည်။ Token ကို `channels.discord.token` မှ ရယူပြီး default account အတွက် `DISCORD_BOT_TOKEN` ကို fallback အဖြစ် အသုံးပြုပါသည် (`channels.discord.enabled` သည် `false` မဖြစ်ပါက)။ cron/CLI command များအတွက် delivery target ကို သတ်မှတ်ရာတွင် `user:<id>` (DM) သို့မဟုတ် `channel:<id>` (guild channel) ကို အသုံးပြုပါ။ အရေအတွက်သက်သက် ID များသည် မရှင်းလင်းသဖြင့် ပယ်ချခံရပါမည်။
Guild slug များသည် အက္ခရာအသေးဖြစ်ပြီး space များကို `-` ဖြင့် အစားထိုးထားသည်; channel key များသည် slugged channel name ကို အသုံးပြုသည် (`#` မပါ)။ အမည်ပြောင်းလဲမှုကြောင့် မရှင်းလင်းမှု မဖြစ်စေရန် guild id များကို key အဖြစ် အသုံးပြုရန် အကြံပြုပါသည်။
Bot ကိုယ်တိုင်ရေးသားသော message များကို default အနေဖြင့် လစ်လျူရှုပါသည်။ `channels.discord.allowBots` ဖြင့် ဖွင့်နိုင်ပါသည် (ကိုယ်ပိုင် message များကိုတော့ self-reply loop မဖြစ်စေရန် ဆက်လက် filter လုပ်ထားသည်)။
Reaction notification mode များ:

- `off`: reaction events မရှိ။
- `own`: ဘော့တ်၏ ကိုယ်ပိုင် မက်ဆေ့ချ်များပေါ်ရှိ reactions (default)။
- `all`: မက်ဆေ့ချ်အားလုံးပေါ်ရှိ reactions အားလုံး။
- `allowlist`: `guilds.<id>
  .users` မှ reaction များကို message အားလုံးတွင် ခွင့်ပြုသည် (စာရင်းလွတ်လျှင် ပိတ်ထားသည်)။Outbound text ကို `channels.discord.textChunkLimit` (default 2000) အရ chunk ခွဲပို့ပါသည်။
  `channels.discord.chunkMode="newline"` ကို သတ်မှတ်ပါက အရှည်အလိုက် chunk မခွဲမီ blank line (paragraph boundary) များအတိုင်း ခွဲပါသည်။ Discord client များတွင် အလွန်ရှည်သော message များကို clip လုပ်နိုင်သဖြင့် `channels.discord.maxLinesPerMessage` (default 17) သည် 2000 chars အောက်ဖြစ်သော်လည်း multi-line reply များကို ခွဲပို့ပါသည်။ Retry policy ၏ default တန်ဖိုးများနှင့် အပြုအမူများကို [Retry policy](/concepts/retry) တွင် မှတ်တမ်းတင်ထားသည်။
  `channels.googlechat` (Chat API webhook)

### Google Chat သည် app-level auth (service account) ဖြင့် HTTP webhook များပေါ်တွင် လည်ပတ်ပါသည်။

Multi-account support သည် `channels.googlechat.accounts` အောက်တွင် ရှိသည် (အထက်ပါ multi-account အပိုင်းကို ကြည့်ပါ)။
Env vars များသည် default account အတွက်သာ သက်ဆိုင်ပါသည်။ {
channels: {
googlechat: {
enabled: true,
serviceAccountFile: "/path/to/service-account.json",
audienceType: "app-url", // app-url | project-number
audience: "https://gateway.example.com/googlechat",
webhookPath: "/googlechat",
botUser: "users/1234567890", // optional; improves mention detection
dm: {
enabled: true,
policy: "pairing", // pairing | allowlist | open | disabled
allowFrom: ["users/1234567890"], // optional; "open" requires ["\*"]
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

```json5
Service account JSON ကို inline (`serviceAccount`) သို့မဟုတ် ဖိုင်အခြေခံ (`serviceAccountFile`) အဖြစ် ထည့်နိုင်ပါသည်။
```

မှတ်ချက်များ-

- Default account အတွက် Env fallback များမှာ `GOOGLE_CHAT_SERVICE_ACCOUNT` သို့မဟုတ် `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE` ဖြစ်သည်။
- `audienceType` နှင့် `audience` သည် Chat app ၏ webhook auth config နှင့် ကိုက်ညီရမည်။
- Delivery target များကို သတ်မှတ်ရာတွင် `spaces/<spaceId>` သို့မဟုတ် `users/<userId|email>` ကို အသုံးပြုပါ။
- `channels.slack` (socket mode)

### Slack သည် Socket Mode ဖြင့် လည်ပတ်ပြီး bot token နှင့် app token နှစ်ခုစလုံး လိုအပ်ပါသည်။

{
channels: {
slack: {
enabled: true,
botToken: "xoxb-...",
appToken: "xapp-...",
dm: {
enabled: true,
policy: "pairing", // pairing | allowlist | open | disabled
allowFrom: ["U123", "U456", "_"], // optional; "open" requires ["_"]
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

```json5
Multi-account support သည် `channels.slack.accounts` အောက်တွင် ရှိသည် (အထက်ပါ multi-account အပိုင်းကို ကြည့်ပါ)။
```

Env token များသည် default account အတွက်သာ သက်ဆိုင်ပါသည်။ Provider ကို enable လုပ်ထားပြီး token နှစ်ခုလုံး (config သို့မဟုတ် `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`) သတ်မှတ်ထားပါက OpenClaw သည် Slack ကို စတင်ပါသည်။

cron/CLI command များအတွက် delivery target ကို သတ်မှတ်ရာတွင် `user:<id>` (DM) သို့မဟုတ် `channel:<id>` ကို အသုံးပြုပါ။ `channels.slack.configWrites: false` ကို သတ်မှတ်ပါက Slack မှ စတင်သော config write များကို (channel ID migration နှင့် `/config set|unset` အပါအဝင်) တားဆီးပါသည်။
Bot ကိုယ်တိုင်ရေးသားသော message များကို default အနေဖြင့် လစ်လျူရှုပါသည်။

`channels.slack.allowBots` သို့မဟုတ် `channels.slack.channels.<id>
.allowBots` ဖြင့် ဖွင့်နိုင်ပါသည်။ Reaction notification mode များ:`allowlist`: message အားလုံးတွင် `channels.slack.reactionAllowlist` မှ reaction များကို ခွင့်ပြုသည် (စာရင်းလွတ်လျှင် ပိတ်ထားသည်)။

Thread session ခွဲခြားမှု:

- `off`: reaction events မရှိ။
- `own`: ဘော့တ်၏ ကိုယ်ပိုင် မက်ဆေ့ချ်များပေါ်ရှိ reactions (default)။
- `all`: မက်ဆေ့ချ်အားလုံးပေါ်ရှိ reactions အားလုံး။
- `channels.slack.thread.historyScope` သည် thread history ကို per-thread (`thread`, default) သို့မဟုတ် channel တစ်ခုလုံးအတွက် မျှဝေထားခြင်း (`channel`) ကို ထိန်းချုပ်ပါသည်။

`channels.slack.thread.inheritParent` သည် thread အသစ်များတွင် parent channel transcript ကို အမွေဆက်ခံမည်/မမည် ကို ထိန်းချုပ်ပါသည် (default: false)။

- Slack action group များ (`slack` tool action များကို gate လုပ်ရန်):
- `channels.slack.thread.inheritParent` controls whether new thread sessions inherit the parent channel transcript (default: false).

Slack action groups (gate `slack` tool actions):

| Action group | Default | မှတ်ချက်များ           |
| ------------ | ------- | ---------------------- |
| reactions    | enabled | React + list reactions |
| messages     | enabled | Read/send/edit/delete  |
| pins         | enabled | Pin/unpin/list         |
| memberInfo   | enabled | Member info            |
| emojiList    | enabled | Custom emoji list      |

### `channels.mattermost` (bot token)

Mattermost ကို plugin အဖြစ် ပေးပို့ထားပြီး core install နှင့် မပါဝင်ပါ။
အရင်ဆုံး ထည့်သွင်းပါ: `openclaw plugins install @openclaw/mattermost` (သို့မဟုတ် git checkout မှ `./extensions/mattermost`)။

Mattermost သည် bot token နှင့် သင့် server အတွက် base URL ကို လိုအပ်ပါသည်:

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

OpenClaw သည် account ကို configure လုပ်ထားပြီး (bot token + base URL) enabled ဖြစ်ပါက Mattermost ကို စတင်အသုံးပြုပါသည်။ Token + base URL ကို default account အတွက် `channels.mattermost.botToken` + `channels.mattermost.baseUrl` သို့မဟုတ် `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` မှ resolve လုပ်ပါသည် (`channels.mattermost.enabled` ကို `false` မထားပါက)။

Chat modes:

- `oncall` (default): @mention လုပ်ထားသောအခါတွင်သာ channel message များကို တုံ့ပြန်ပါသည်။
- `onmessage`: ချန်နယ်မက်ဆေ့ချ် အားလုံးကို တုံ့ပြန်ပါသည်။
- `onchar`: message သည် trigger prefix (`channels.mattermost.oncharPrefixes`, default `[">", "!"]`) ဖြင့် စတင်သောအခါ တုံ့ပြန်ပါသည်။

Access control:

- Default DMs: `channels.mattermost.dmPolicy="pairing"` (မသိသော sender များကို pairing code ပေးပါသည်)။
- Public DMs: `channels.mattermost.dmPolicy="open"` နှင့်အတူ `channels.mattermost.allowFrom=["*"]`။
- Groups: default အနေဖြင့် `channels.mattermost.groupPolicy="allowlist"` (mention ဖြင့်သာ ဝင်ရောက်ခွင့်) ဖြစ်ပါသည်။ Sender များကို ကန့်သတ်ရန် `channels.mattermost.groupAllowFrom` ကို အသုံးပြုပါ။

Multi-account support ကို `channels.mattermost.accounts` အောက်တွင် ထားရှိပါသည် (အထက်ပါ multi-account အပိုင်းကို ကြည့်ပါ)။ Env vars များသည် default account အတွက်သာ သက်ရောက်ပါသည်။
Delivery target များကို သတ်မှတ်ရာတွင် `channel:<id>` သို့မဟုတ် `user:<id>` (သို့မဟုတ် `@username`) ကို အသုံးပြုပါ။ id ကိုသာ ပေးပါက channel id အဖြစ် သတ်မှတ်ပါသည်။

### `channels.signal` (signal-cli)

Signal reactions များသည် system events များကို ထုတ်ပေးနိုင်ပါသည် (shared reaction tooling)။

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // include last N group messages as context (0 disables)
    },
  },
}
```

Reaction notification modes:

- `off`: reaction events မရှိ။
- `own`: ဘော့တ်၏ ကိုယ်ပိုင် မက်ဆေ့ချ်များပေါ်ရှိ reactions (default)။
- `all`: မက်ဆေ့ချ်အားလုံးပေါ်ရှိ reactions အားလုံး။
- `allowlist`: `channels.signal.reactionAllowlist` ထဲရှိ reactions များကို မက်ဆေ့ချ်အားလုံးတွင် ခွင့်ပြုပါသည် (အလွတ်စာရင်းဖြစ်ပါက ပိတ်ထားသည်)။

### `channels.imessage` (imsg CLI)

OpenClaw သည် `imsg rpc` ကို spawn လုပ်ပါသည် (stdio ပေါ်တွင် JSON-RPC)။ Daemon သို့မဟုတ် port မလိုအပ်ပါ။

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SSH wrapper အသုံးပြုသောအခါ remote attachment များအတွက် SCP
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // နောက်ဆုံး group message N ခုကို context အဖြစ် ထည့်သွင်းပါသည် (0 ဆိုပါက ပိတ်ပါသည်)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

Multi-account support ကို `channels.imessage.accounts` အောက်တွင် ထားရှိပါသည် (အထက်ပါ multi-account အပိုင်းကို ကြည့်ပါ)။

မှတ်ချက်များ-

- Messages DB ကို Full Disk Access လိုအပ်ပါသည်။
- ပထမဆုံး message ပို့ရာတွင် Messages automation permission ကို မေးမြန်းပါလိမ့်မည်။
- `chat_id:<id>` target များကို ဦးစားပေး အသုံးပြုပါ။ Chat များကို စာရင်းကြည့်ရန် `imsg chats --limit 20` ကို အသုံးပြုပါ။
- `channels.imessage.cliPath` ကို wrapper script တစ်ခု (ဥပမာ `ssh` ဖြင့် `imsg rpc` ကို run လုပ်သော အခြား Mac သို့) ကိုညွှန်ပြနိုင်ပါသည်။ password prompt မဖြစ်စေရန် SSH keys ကို အသုံးပြုပါ။
- Remote SSH wrapper များအတွက် `includeAttachments` ကို enabled လုပ်ထားပါက attachment များကို SCP ဖြင့် fetch လုပ်ရန် `channels.imessage.remoteHost` ကို သတ်မှတ်ပါ။

Wrapper ဥပမာ:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

Agent သည် file operation များအတွက် အသုံးပြုသော **single global workspace directory** ကို သတ်မှတ်ပါသည်။

Default: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

`agents.defaults.sandbox` ကို enabled လုပ်ထားပါက main မဟုတ်သော session များသည် `agents.defaults.sandbox.workspaceRoot` အောက်ရှိ scope အလိုက် workspace များဖြင့် override လုပ်နိုင်ပါသည်။

### `agents.defaults.repoRoot`

System prompt ၏ Runtime line တွင် ပြသရန် optional repository root ဖြစ်ပါသည်။ မသတ်မှတ်ထားပါက OpenClaw သည် workspace (နှင့် လက်ရှိ working directory) မှ အပေါ်ဘက်သို့ `.git` directory ကို ရှာဖွေကြိုးစားပါသည်။ အသုံးပြုရန် path သည် ရှိပြီးသား ဖြစ်ရပါမည်။

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Workspace bootstrap file များ (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, နှင့် `BOOTSTRAP.md`) ကို အလိုအလျောက် ဖန်တီးခြင်းကို ပိတ်ပါသည်။

Workspace file များကို repo မှ ကြိုတင် ထည့်သွင်းထားသော deployment များအတွက် အသုံးပြုပါ။

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Truncation မလုပ်မီ system prompt ထဲသို့ ထည့်သွင်းမည့် workspace bootstrap file တစ်ခုစီ၏ အများဆုံး character အရေအတွက် ဖြစ်ပါသည်။ Default: `20000`.

File တစ်ခုသည် ဤကန့်သတ်ချက်ကို ကျော်လွန်ပါက OpenClaw သည် warning ကို log လုပ်ပြီး marker ပါသော truncated head/tail ကို ထည့်သွင်းပါသည်။

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

**System prompt context** အတွက် အသုံးပြုသူ၏ timezone ကို သတ်မှတ်ပါသည် (message envelope များရှိ timestamp များအတွက် မဟုတ်ပါ)။ မသတ်မှတ်ထားပါက OpenClaw သည် runtime အချိန်တွင် host timezone ကို အသုံးပြုပါသည်။

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

စနစ် prompt ရဲ့ Current Date & Time အပိုင်းမှာ ပြသမယ့် **အချိန် ဖော်မတ်** ကို ထိန်းချုပ်သည်။
မူလသတ်မှတ်ချက်: `auto` (OS ဦးစားပေးမှုအတိုင်း)။

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `messages`

အဝင်/အထွက် prefix များနှင့် ရွေးချယ်နိုင်သော ack reaction များကို ထိန်းချုပ်သည်။
queueing, sessions နှင့် streaming context အတွက် [Messages](/concepts/messages) ကို ကြည့်ပါ။

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` ကို **ထွက်သည့် reply အားလုံး** (tool summaries, block streaming, final replies) တွင် channel အားလုံးအနှံ့ အသုံးချမည်ဖြစ်ပြီး မရှိပြီးသားဖြစ်ပါကသာ အသုံးချမည်။

Override များကို channel အလိုက်၊ account အလိုက် သတ်မှတ်နိုင်သည်။

- `channels.<channel>``.responsePrefix`
- `channels.<channel>``.accounts.<id>``.responsePrefix`

ဆုံးဖြတ် အစဉ် (အသေးစိတ်ဆုံးက အနိုင်ရ) —

1. `channels.<channel>``.accounts.<id>``.responsePrefix`
2. `channels.<channel>``.responsePrefix`
3. `messages.responsePrefix`

အဓိပ္ပါယ်ဖွင့်ဆိုချက်များ:

- `undefined` ဖြစ်ပါက နောက်အဆင့်သို့ ဆက်လက် လွှဲချသွားသည်။
- `""` သည် prefix ကို တိတိကျကျ ပိတ်ထားပြီး cascade ကို ရပ်တန့်စေသည်။
- `"auto"` သည် route လုပ်ထားသော agent အတွက် `[{identity.name}]` ကို ထုတ်ယူအသုံးပြုသည်။

Override များသည် extension များအပါအဝင် channel အားလုံးနှင့် ထွက်သည့် reply အမျိုးအစားအားလုံးအတွက် သက်ရောက်သည်။

`messages.responsePrefix` ကို မသတ်မှတ်ထားပါက မူလအနေဖြင့် prefix ကို မအသုံးပြုပါ။ WhatsApp ကို ကိုယ်တိုင်နှင့် စကားပြောသည့် chat တွင် reply များမှာ ခြွင်းချက်ဖြစ်ပြီး: သတ်မှတ်ထားပါက `[{identity.name}]` ကို မူလသုံးမည်၊ မဟုတ်ပါက `[openclaw]` ကို အသုံးပြုမည်ဖြစ်သဖြင့် ဖုန်းတစ်လုံးတည်း စကားပြောခြင်းများကို ဖတ်ရလွယ်ကူစေသည်။
သတ်မှတ်ထားသောအခါ route လုပ်ထားသော agent အတွက် `[{identity.name}]` ကို ထုတ်ယူရန် `"auto"` ဟု သတ်မှတ်ပါ။

#### Template variables

`responsePrefix` string အတွင်းတွင် dynamic အဖြစ် ဖြေရှင်းပေးမည့် template variables များကို ထည့်သွင်းနိုင်သည်။

| Variable          | Description                  | Example                                        |
| ----------------- | ---------------------------- | ---------------------------------------------- |
| `{model}`         | မော်ဒယ်အမည် အတိုကောက်        | `claude-opus-4-6`, `gpt-4o`                    |
| `{modelFull}`     | မော်ဒယ် အပြည့်အစုံ အမှတ်အသား | `anthropic/claude-opus-4-6`                    |
| `{provider}`      | Provider အမည်                | `anthropic`, `openai`                          |
| `{thinkingLevel}` | လက်ရှိ thinking level        | `high`, `low`, `off`                           |
| `{identity.name}` | Agent identity အမည်          | (`"auto"` mode နှင့် တူသည်) |

Variable များသည် case-insensitive ဖြစ်သည် (`{MODEL}` = `{model}`)။ `{think}` သည် `{thinkingLevel}` အတွက် alias ဖြစ်သည်။
မဖြေရှင်းနိုင်သော variable များကို literal စာသားအဖြစ် ကျန်ရှိစေမည်။

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

ဥပမာ output: `[claude-opus-4-6 | think:high] Here's my response...`

WhatsApp inbound prefix is configured via `channels.whatsapp.messagePrefix` (deprecated:
`messages.messagePrefix`). Default stays **unchanged**: `"[openclaw]"` when
`channels.whatsapp.allowFrom` is empty, otherwise `""` (no prefix). When using
`"[openclaw]"`, OpenClaw will instead use `[{identity.name}]` when the routed
agent has `identity.name` set.

`ackReaction` sends a best-effort emoji reaction to acknowledge inbound messages
on channels that support reactions (Slack/Discord/Telegram/Google Chat). Defaults to the
active agent’s `identity.emoji` when set, otherwise `"👀"`. Set it to `""` to disable.

`ackReactionScope` controls when reactions fire:

- `group-mentions` (default): only when a group/room requires mentions **and** the bot was mentioned
- `group-all`: all group/room messages
- `direct`: direct messages only
- `all`: all messages

`removeAckAfterReply` removes the bot’s ack reaction after a reply is sent
(Slack/Discord/Telegram/Google Chat only). Default: `false`.

#### `messages.tts`

Enable text-to-speech for outbound replies. When on, OpenClaw generates audio
using ElevenLabs or OpenAI and attaches it to responses. Telegram uses Opus
voice notes; other channels send MP3 audio.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
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

မှတ်ချက်များ-

- `messages.tts.auto` controls auto‑TTS (`off`, `always`, `inbound`, `tagged`).
- `/tts off|always|inbound|tagged` sets the per‑session auto mode (overrides config).
- `messages.tts.enabled` is legacy; doctor migrates it to `messages.tts.auto`.
- `prefsPath` stores local overrides (provider/limit/summarize).
- `maxTextLength` is a hard cap for TTS input; summaries are truncated to fit.
- `summaryModel` overrides `agents.defaults.model.primary` for auto-summary.
  - Accepts `provider/model` or an alias from `agents.defaults.models`.
- `modelOverrides` enables model-driven overrides like `[[tts:...]]` tags (on by default).
- `/tts limit` and `/tts summary` control per-user summarization settings.
- `apiKey` values fall back to `ELEVENLABS_API_KEY`/`XI_API_KEY` and `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` overrides the ElevenLabs API base URL.
- `elevenlabs.voiceSettings` supports `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost`, and `speed` (0.5..2.0).

### `talk`

Defaults for Talk mode (macOS/iOS/Android). Voice IDs fall back to `ELEVENLABS_VOICE_ID` or `SAG_VOICE_ID` when unset.
`apiKey` falls back to `ELEVENLABS_API_KEY` (or the gateway’s shell profile) when unset.
`voiceAliases` lets Talk directives use friendly names (e.g. `"voice":"Clawd"`).

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

### `agents.defaults`

Controls the embedded agent runtime (model/thinking/verbose/timeouts).
`agents.defaults.models` defines the configured model catalog (and acts as the allowlist for `/model`).
`agents.defaults.model.primary` sets the default model; `agents.defaults.model.fallbacks` are global failovers.
`agents.defaults.imageModel` is optional and is **only used if the primary model lacks image input**.
Each `agents.defaults.models` entry can include:

- `alias` (optional model shortcut, e.g. `/opus`).
- `params` (optional provider-specific API params passed through to the model request).

`params` is also applied to streaming runs (embedded agent + compaction). Supported keys today: `temperature`, `maxTokens`. These merge with call-time options; caller-supplied values win. `temperature` is an advanced knob—leave unset unless you know the model’s defaults and need a change.

ဥပမာ —

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x models automatically enable thinking mode unless you:

- 1. `--thinking off` ကို သတ်မှတ်ပါ၊ သို့မဟုတ်
- 2. `agents.defaults.models["zai/<model>"].params.thinking` ကို သင်ကိုယ်တိုင် သတ်မှတ်ပါ။

3. OpenClaw တွင် အတွင်းထည့်သွင်းထားသော alias shorthand အချို့ကိုလည်း ပါဝင်ပို့ဆောင်ထားသည်။ 4. Default များသည် မော်ဒယ်ကို `agents.defaults.models` ထဲတွင် ရှိပြီးသား ဖြစ်သောအခါမှသာ အသက်ဝင်သည်။

- 5. `opus` -> `anthropic/claude-opus-4-6`
- 6. `sonnet` -> `anthropic/claude-sonnet-4-5`
- 7. `gpt` -> `openai/gpt-5.2`
- 8. `gpt-mini` -> `openai/gpt-5-mini`
- 9. `gemini` -> `google/gemini-3-pro-preview`
- 10. `gemini-flash` -> `google/gemini-3-flash-preview`

11. သင်ကိုယ်တိုင် alias အမည်တူ (case-insensitive) ကို သတ်မှတ်ထားပါက သင့်တန်ဖိုးက ဦးစားပေး အနိုင်ရသည် (default များသည် မည်သည့်အခါမှ override မလုပ်ပါ)။

12. ဥပမာ- Opus 4.6 ကို primary အဖြစ် အသုံးပြုပြီး MiniMax M2.1 ကို fallback အဖြစ် (hosted MiniMax) သတ်မှတ်ခြင်း-

```json5
13. {
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
    },
  },
}
```

14. MiniMax auth- `MINIMAX_API_KEY` ကို (env) သတ်မှတ်ပါ သို့မဟုတ် `models.providers.minimax` ကို configure လုပ်ပါ။

#### 15. `agents.defaults.cliBackends` (CLI fallback)

16. tool call မပါဝင်သော text-only fallback run များအတွက် Optional CLI backend များ။ 17. API provider များ ပျက်ကွက်သည့်အခါ backup လမ်းကြောင်းအဖြစ် အသုံးဝင်သည်။ 18. file path များကို လက်ခံသော `imageArg` ကို configure လုပ်ထားပါက image pass-through ကို ထောက်ပံ့ပေးသည်။

မှတ်ချက်များ-

- 19. CLI backend များသည် **text-first** ဖြစ်ပြီး tool များကို အမြဲတမ်း ပိတ်ထားသည်။
- 20. `sessionArg` ကို သတ်မှတ်ထားပါက session များကို ထောက်ပံ့ပေးပြီး session id များကို backend တစ်ခုချင်းစီအလိုက် သိမ်းဆည်းထားသည်။
- 21. `claude-cli` အတွက် default များကို ကြိုတင် ချိတ်ဆက်ထားပြီးသား ဖြစ်သည်။ 22. PATH သေးငယ်နေပါက (launchd/systemd) command path ကို override လုပ်ပါ။

ဥပမာ —

```json5
23. {
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

```json5
24. {
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
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
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### 25. `agents.defaults.contextPruning` (tool-result pruning)

26. `agents.defaults.contextPruning` သည် LLM သို့ request ပို့မီ အချိန်တွင် in-memory context ထဲမှ **ဟောင်းသော tool result များ** ကို ဖြတ်တောက် ဖယ်ရှားပေးသည်။
27. disk ပေါ်ရှိ session history ကို မပြင်ဆင်ပါ (`*.jsonl` သည် အပြည့်အစုံ ရှိနေဆဲ ဖြစ်သည်)။

28. အချိန်ကြာလာသည်နှင့်အမျှ tool output ကြီးများ စုဆောင်းလာသော chatty agent များအတွက် token အသုံးပြုမှု လျော့ချရန် ရည်ရွယ်ထားသည်။

29. အဆင့်မြင့် အကျဉ်းချုပ်-

- 30. user/assistant message များကို မည်သည့်အခါမှ မထိပါ။
- 31. နောက်ဆုံး `keepLastAssistants` assistant message များကို ကာကွယ်ထားသည် (ထိုအချက်အပြီးရှိ tool result များကို မဖြတ်တောက်ပါ)။
- 32. bootstrap prefix ကို ကာကွယ်ထားသည် (ပထမ user message မတိုင်မီရှိ အရာအားလုံးကို မဖြတ်တောက်ပါ)။
- 33. Modes-
  - 34. `adaptive`- ခန့်မှန်းထားသော context ratio သည် `softTrimRatio` ကို ကျော်လွန်သည့်အခါ oversized tool result များကို soft-trim လုပ်သည် (head/tail ကို ထိန်းသိမ်းထားသည်)။
        35. ထို့နောက် ခန့်မှန်းထားသော context ratio သည် `hardClearRatio` ကို ကျော်လွန်ပြီး **နှင့်** ဖြတ်တောက်နိုင်သော tool-result အစုအဝေး လုံလောက်ပါက (`minPrunableToolChars`) အဟောင်းဆုံး eligible tool result များကို hard-clear လုပ်သည်။
  - 36. `aggressive`- cutoff မတိုင်မီရှိ eligible tool result များကို ratio စစ်ဆေးခြင်းမရှိဘဲ `hardClear.placeholder` ဖြင့် အမြဲ အစားထိုးသည်။

37. Soft vs hard pruning (LLM သို့ ပို့သော context ထဲတွင် ဘာတွေ ပြောင်းလဲသလဲ)-

- 38. **Soft-trim**- _oversized_ tool result များအတွက်သာ အသုံးပြုသည်။ 39. အစပိုင်း + အဆုံးပိုင်းကို ထိန်းသိမ်းထားပြီး အလယ်တွင် `...` ထည့်သွင်းသည်။
  - 40. Before: `toolResult("…very long output…")`
  - 41. After: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- 42. **Hard-clear**- tool result အပြည့်အစုံကို placeholder ဖြင့် အစားထိုးသည်။
  - 43. Before: `toolResult("…very long output…")`
  - 44. After: `toolResult("[Old tool result content cleared]")`

45. မှတ်စုများ / လက်ရှိ ကန့်သတ်ချက်များ-

- 46. **image block ပါဝင်သော tool result များကို လောလောဆယ် ကျော်သွားသည်** (မဖြတ်တောက်/မရှင်းလင်းပါ)။
- 47. ခန့်မှန်းထားသော “context ratio” သည် token အတိအကျ မဟုတ်ဘဲ **character အရေအတွက်** အပေါ် အခြေခံထားသည်။
- 48. session တွင် `keepLastAssistants` assistant message အရေအတွက် မပြည့်မီပါက pruning ကို မလုပ်ပါ။
- 49. `aggressive` mode တွင် `hardClear.enabled` ကို မစဉ်းစားပါ (eligible tool result များကို အမြဲ `hardClear.placeholder` ဖြင့် အစားထိုးသည်)။

50. Default (adaptive)-

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

ပိတ်ရန်:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

`mode` သည် `"adaptive"` သို့မဟုတ် `"aggressive"` ဖြစ်သောအခါ မူလတန်ဖိုးများ:

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (adaptive အတွက်သာ)
- `hardClearRatio`: `0.5` (adaptive အတွက်သာ)
- `minPrunableToolChars`: `50000` (adaptive အတွက်သာ)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (adaptive အတွက်သာ)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

ဥပမာ (aggressive, အနည်းဆုံး):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

ဥပမာ (adaptive ကို ချိန်ညှိထားသည်):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // Optional: restrict pruning to specific tools (deny wins; supports "*" wildcards)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

အပြုအမူ အသေးစိတ်များအတွက် [/concepts/session-pruning](/concepts/session-pruning) ကို ကြည့်ပါ။

#### `agents.defaults.compaction` (headroom ထားရှိခြင်း + memory flush)

`agents.defaults.compaction.mode` သည် compaction အကျဉ်းချုပ် ပြုလုပ်နည်းဗျူဟာကို ရွေးချယ်သည်။ မူလအားဖြင့် `default` ဖြစ်ပြီး၊ အလွန်ရှည်လျားသော history များအတွက် chunked summarization ကို ဖွင့်ရန် `safeguard` ကို သတ်မှတ်ပါ။ [/concepts/compaction](/concepts/compaction) ကို ကြည့်ပါ။

`agents.defaults.compaction.reserveTokensFloor` သည် Pi compaction အတွက် အနည်းဆုံး `reserveTokens`
တန်ဖိုးကို အတည်ပြုသတ်မှတ်ပေးသည် (မူလတန်ဖိုး: `20000`)။ floor ကို ပိတ်ရန် `0` ဟု သတ်မှတ်ပါ။

`agents.defaults.compaction.memoryFlush` သည် auto-compaction မတိုင်မီ **တိတ်ဆိတ်သော** agentic turn တစ်ကြိမ် လုပ်ဆောင်ပြီး
model ကို disk ပေါ်တွင် အကြာကြီးအသုံးဝင်မည့် memory များ သိမ်းဆည်းရန် ညွှန်ကြားသည် (ဥပမာ `memory/YYYY-MM-DD.md`)။ session token ခန့်မှန်းတန်ဖိုးသည် compaction ကန့်သတ်ချက်အောက်ရှိ soft threshold ကို ကျော်လွန်သောအခါ အလုပ်လုပ်စေသည်။

Legacy မူလတန်ဖိုးများ:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: `NO_REPLY` ပါဝင်သည့် built-in မူလတန်ဖိုးများ
- မှတ်ချက်: session workspace သည် read-only ဖြစ်သောအခါ memory flush ကို ကျော်သွားမည်
  (`agents.defaults.sandbox.workspaceAccess: "ro"` သို့မဟုတ် `"none"`)။

ဥပမာ (ချိန်ညှိထားသည်):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
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

Block streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (မူလအခြေအနေ ပိတ်ထားသည်)။

- Channel override များ: block streaming ကို ဖွင့်/ပိတ် အတင်းအကျပ်လုပ်ရန် `*.blockStreaming` (နှင့် per-account မျိုးကွဲများ)။
  Telegram မဟုတ်သော channel များတွင် block reply များကို ဖွင့်ရန် `*.blockStreaming: true` ကို သတ်မှတ်ရန် လိုအပ်သည်။

- `agents.defaults.blockStreamingBreak`: `"text_end"` သို့မဟုတ် `"message_end"` (မူလ: text_end)။

- `agents.defaults.blockStreamingChunk`: streamed block များအတွက် soft chunking။ မူလအားဖြင့်
  800–1200 အက္ခရာများ ဖြစ်ပြီး၊ paragraph break (`\n\n`) ကို ဦးစားပေးကာ၊ ထို့နောက် newline၊ ထို့နောက် sentence များကို အသုံးပြုသည်။
  ဥပမာ —

  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: ပို့မည့်အခါ streamed block များကို ပေါင်းစည်းသည်။
  မူလတန်ဖိုးမှာ `{ idleMs: 1000 }` ဖြစ်ပြီး `blockStreamingChunk` မှ `minChars` ကို အမွေဆက်ခံကာ
  `maxChars` ကို channel စာသားကန့်သတ်ချက်အထိ ကန့်သတ်ထားသည်။ Signal/Slack/Discord/Google Chat တွင်
  override မလုပ်ပါက `minChars: 1500` ကို မူလအဖြစ် သတ်မှတ်ထားသည်။
  Channel override များ: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (နှင့် per-account မျိုးကွဲများ)။

- `agents.defaults.humanDelay`: ပထမ block ပြန်စာပြီးနောက် **block reply** များအကြား ကျပန်းနားချိန်။
  Mode များ: `off` (မူလ), `natural` (800–2500ms), `custom` (`minMs`/`maxMs` ကို အသုံးပြု)။
  Agent တစ်ခုချင်းစီအလိုက် override: `agents.list[].humanDelay`။
  ဥပမာ —

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  အပြုအမူနှင့် chunking အသေးစိတ်များအတွက် [/concepts/streaming](/concepts/streaming) ကို ကြည့်ပါ။

Typing indicator များ:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`။ မူလအားဖြင့်
  `instant` ကို direct chat / mention များအတွက် သုံးပြီး `message` ကို mention မလုပ်ထားသော group chat များအတွက် သုံးသည်။
- `session.typingMode`: mode အတွက် session အလိုက် override။
- `agents.defaults.typingIntervalSeconds`: typing signal ကို refresh လုပ်သည့် အကြိမ်နှုန်း (မူလ: 6s)။
- `session.typingIntervalSeconds`: refresh interval အတွက် session အလိုက် override။
  See [/concepts/typing-indicators](/concepts/typing-indicators) for behavior details.

`agents.defaults.model.primary` should be set as `provider/model` (e.g. `anthropic/claude-opus-4-6`).
Aliases come from `agents.defaults.models.*.alias` (e.g. `Opus`).
If you omit the provider, OpenClaw currently assumes `anthropic` as a temporary
deprecation fallback.
Z.AI models are available as `zai/<model>` (e.g. `zai/glm-4.7`) and require
`ZAI_API_KEY` (or legacy `Z_AI_API_KEY`) in the environment.

`agents.defaults.heartbeat` configures periodic heartbeat runs:

- `every`: duration string (`ms`, `s`, `m`, `h`); default unit minutes. Default:
  `30m`. Set `0m` to disable.
- `model`: optional override model for heartbeat runs (`provider/model`).
- `includeReasoning`: when `true`, heartbeats will also deliver the separate `Reasoning:` message when available (same shape as `/reasoning on`). Default: `false`.
- `session`: optional session key to control which session the heartbeat runs in. Default: `main`.
- `to`: optional recipient override (channel-specific id, e.g. E.164 for WhatsApp, chat id for Telegram).
- `target`: optional delivery channel (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). Default: `last`.
- `prompt`: optional override for the heartbeat body (default: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Overrides are sent verbatim; include a `Read HEARTBEAT.md` line if you still want the file read.
- `ackMaxChars`: max chars allowed after `HEARTBEAT_OK` before delivery (default: 300).

Per-agent heartbeats:

- Set `agents.list[].heartbeat` to enable or override heartbeat settings for a specific agent.
- If any agent entry defines `heartbeat`, **only those agents** run heartbeats; defaults
  become the shared baseline for those agents.

Heartbeats run full agent turns. Shorter intervals burn more tokens; be mindful
of `every`, keep `HEARTBEAT.md` tiny, and/or choose a cheaper `model`.

`tools.exec` configures background exec defaults:

- `backgroundMs`: time before auto-background (ms, default 10000)
- `timeoutSec`: auto-kill after this runtime (seconds, default 1800)
- `cleanupMs`: how long to keep finished sessions in memory (ms, default 1800000)
- `notifyOnExit`: enqueue a system event + request heartbeat when backgrounded exec exits (default true)
- `applyPatch.enabled`: enable experimental `apply_patch` (OpenAI/OpenAI Codex only; default false)
- `applyPatch.allowModels`: optional allowlist of model ids (e.g. `gpt-5.2` or `openai/gpt-5.2`)
  Note: `applyPatch` is only under `tools.exec`.

`tools.web` configures web search + fetch tools:

- `tools.web.search.enabled` (default: true when key is present)
- `tools.web.search.apiKey` (recommended: set via `openclaw configure --section web`, or use `BRAVE_API_KEY` env var)
- `tools.web.search.maxResults` (1–10, default 5)
- `tools.web.search.timeoutSeconds` (မူလတန်ဖိုး 30)
- `tools.web.search.cacheTtlMinutes` (မူလတန်ဖိုး 15)
- `tools.web.fetch.enabled` (default true)
- `tools.web.fetch.maxChars` (မူလတန်ဖိုး 50000)
- `tools.web.fetch.maxCharsCap` (default 50000; clamps maxChars from config/tool calls)
- `tools.web.fetch.timeoutSeconds` (မူလတန်ဖိုး 30)
- `tools.web.fetch.cacheTtlMinutes` (မူလတန်ဖိုး 15)
- `tools.web.fetch.userAgent` (optional override)
- `tools.web.fetch.readability` (default true; disable to use basic HTML cleanup only)
- `tools.web.fetch.firecrawl.enabled` (default true when an API key is set)
- `tools.web.fetch.firecrawl.apiKey` (optional; defaults to `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (default [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (default true)
- `tools.web.fetch.firecrawl.maxAgeMs` (optional)
- `tools.web.fetch.firecrawl.timeoutSeconds` (optional)

`tools.media` configures inbound media understanding (image/audio/video):

- `tools.media.models`: shared model list (capability-tagged; used after per-cap lists).
- `tools.media.concurrency`: max concurrent capability runs (default 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: opt-out ခလုတ် (မော်ဒယ်များကို ပြင်ဆင်ထားပါက ပုံမှန်အားဖြင့် true ဖြစ်သည်)။
  - `prompt`: ရွေးချယ်နိုင်သော prompt အစားထိုး (image/video တွင် `maxChars` အညွှန်းကို အလိုအလျောက် ထည့်ပေါင်းသည်)။
  - `maxChars`: ထုတ်လွှင့်နိုင်သော စာလုံးအများဆုံး (image/video အတွက် ပုံမှန် 500; audio အတွက် မသတ်မှတ်ထား)။
  - `maxBytes`: ပို့ရန် ခွင့်ပြုသော မီဒီယာအရွယ်အစားအများဆုံး (ပုံမှန်: image 10MB, audio 20MB, video 50MB)။
  - `timeoutSeconds`: တောင်းဆိုမှု အချိန်ကန့်သတ် (ပုံမှန်: image 60s, audio 60s, video 120s)။
  - `language`: audio အတွက် ရွေးချယ်နိုင်သော ဘာသာစကား အညွှန်း။
  - `attachments`: ပူးတွဲဖိုင် မူဝါဒ (`mode`, `maxAttachments`, `prefer`)။
  - `scope`: ရွေးချယ်နိုင်သော ကန့်သတ်ခြင်း (ပထမကိုက်ညီမှုသာ အနိုင်ရ) `match.channel`, `match.chatType`, သို့မဟုတ် `match.keyPrefix` ဖြင့် သတ်မှတ်နိုင်သည်။
  - `models`: မော်ဒယ် ထည့်သွင်းချက်များကို အစီအစဉ်လိုက် စာရင်းပြုစုထားခြင်း; မအောင်မြင်ပါက သို့မဟုတ် မီဒီယာအရွယ်အစားကြီးလွန်းပါက နောက်တစ်ခုသို့ ပြန်လည်အသုံးပြုသည်။
- မော်ဒယ် `models[]` တစ်ခုချင်းစီအတွက်:
  - Provider entry (`type: "provider"` သို့မဟုတ် မထည့်လည်းရ):
    - `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq`, စသည်)။
    - `model`: မော်ဒယ် id အစားထိုး (image အတွက် မဖြစ်မနေလိုအပ်; audio providers အတွက် ပုံမှန် `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo`, video အတွက် `gemini-3-flash-preview`)။
    - `profile` / `preferredProfile`: အတည်ပြုရေး profile ရွေးချယ်မှု။
  - CLI entry (`type: "cli"`):
    - `command`: လုပ်ဆောင်ရန် executable။
    - `args`: template ပါသော args (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, စသည်တို့ကို ပံ့ပိုးသည်)။
  - `capabilities`: ရွေးချယ်နိုင်သော စာရင်း (`image`, `audio`, `video`) ကို မျှဝေထားသော entry ကို ကန့်သတ်ရန် အသုံးပြုသည်။ မထည့်ထားပါက ပုံမှန်များ: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio။
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` ကို entry တစ်ခုချင်းစီအလိုက် အစားထိုးနိုင်သည်။

မော်ဒယ်များ မသတ်မှတ်ထားပါက (သို့မဟုတ် `enabled: false`) နားလည်မှုကို ကျော်လွှားသည်; သို့သော် မော်ဒယ်သည် မူလ attachments များကို ဆက်လက် လက်ခံရရှိမည်ဖြစ်သည်။

Provider အတည်ပြုရေးသည် စံ မော်ဒယ် အတည်ပြုအစီအစဉ်ကို လိုက်နာသည် (auth profiles, `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY` ကဲ့သို့သော env vars, သို့မဟုတ် `models.providers.*.apiKey`)။

ဥပမာ:

```json5
{
  tools: {
    media: {
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

`agents.defaults.subagents` သည် sub-agent ပုံမှန်တန်ဖိုးများကို ပြင်ဆင်သတ်မှတ်သည်:

- `model`: ဖန်တီးထားသော sub-agents များအတွက် ပုံမှန် မော်ဒယ် (string သို့မဟုတ် `{ primary, fallbacks }`)။ မထည့်ထားပါက sub-agents များသည် ခေါ်ဆိုသူ၏ မော်ဒယ်ကို ဆက်ခံအသုံးပြုမည်ဖြစ်ပြီး agent သို့မဟုတ် call အလိုက် အစားထိုးထားခြင်းမရှိလျှင် ဖြစ်သည်။
- `maxConcurrent`: တစ်ပြိုင်နက် sub-agent အများဆုံး လုပ်ဆောင်နိုင်သည့် အရေအတွက် (ပုံမှန် 1)
- `archiveAfterMinutes`: မိနစ် N ပြီးနောက် sub-agent session များကို အလိုအလျောက် archive ပြုလုပ်ခြင်း (ပုံမှန် 60; ပိတ်ရန် `0` သတ်မှတ်ပါ)
- Subagent တစ်ခုချင်းစီအလိုက် tool မူဝါဒ: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny သည် အနိုင်ရ)

`tools.profile` သည် `tools.allow`/`tools.deny` မတိုင်မီ **အခြေခံ tool allowlist** ကို သတ်မှတ်သည်:

- `minimal`: `session_status` သာလျှင်
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: ကန့်သတ်ချက်မရှိ (မသတ်မှတ်ထားသကဲ့သို့)

Agent အလိုက် အစားထိုး: `agents.list[].tools.profile`။

ဥပမာ (မက်ဆေ့ချ်ပို့ခြင်းသာ မူလသတ်မှတ်ထားပြီး Slack + Discord tools များကိုလည်း ခွင့်ပြုရန်):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

ဥပမာ (coding profile ဖြစ်သော်လည်း exec/process ကို နေရာတိုင်းတွင် ပိတ်ပင်ရန်):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` သည် provider သီးသန့် (သို့မဟုတ် provider/model တစ်ခုတည်း) အတွက် tools များကို **ထပ်မံ ကန့်သတ်** ခွင့်ပြုသည်။
Agent အလိုက် အစားထိုး: `agents.list[].tools.byProvider`။

အစီအစဉ်: အခြေခံ profile → provider profile → allow/deny မူဝါဒများ။
Provider keys များသည် `provider` (ဥပမာ `google-antigravity`) သို့မဟုတ် `provider/model`
(ဥပမာ `openai/gpt-5.2`) ကို လက်ခံသည်။

ဥပမာ (ကမ္ဘာလုံးဆိုင်ရာ coding profile ကို ထားရှိပြီး Google Antigravity အတွက် tools အနည်းဆုံးသာ):

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

ဥပမာ (provider/model အထူးသတ်မှတ် allowlist):

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

`tools.allow` / `tools.deny` သည် ကမ္ဘာလုံးဆိုင်ရာ tool allow/deny မူဝါဒကို ပြင်ဆင်သတ်မှတ်သည် (deny သည် အနိုင်ရ)။
ကိုက်ညီမှုသည် case-insensitive ဖြစ်ပြီး `*` wildcard များကို ပံ့ပိုးသည် (`"*"` သည် tools အားလုံးကို ဆိုလိုသည်)။
Docker sandbox ကို **ပိတ်ထား** သော်လည်း ဤအရာကို အသုံးချနေဆဲဖြစ်သည်။

ဥပမာ (နေရာအားလုံးတွင် browser/canvas ကို ပိတ်ရန်):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Tool အုပ်စုများ (shorthands) သည် **global** နှင့် **per-agent** tool မူဝါဒများတွင် အလုပ်လုပ်သည်:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: OpenClaw built-in tools အားလုံး (provider plugins မပါဝင်)

`tools.elevated` သည် မြှင့်တင်ထားသော (host) exec access ကို ထိန်းချုပ်သည်:

- `enabled`: elevated mode ကို ခွင့်ပြုရန် (ပုံမှန် true)
- `allowFrom`: channel အလိုက် allowlist များ (အလွတ် = ပိတ်ထား)
  - `whatsapp`: E.164 ဖုန်းနံပါတ်များ
  - `telegram`: chat ids သို့မဟုတ် usernames
  - `discord`: user ids သို့မဟုတ် usernames (`channels.discord.dm.allowFrom` ကို မထည့်ထားပါက fallback အသုံးပြုသည်)
  - `signal`: E.164 ဖုန်းနံပါတ်များ
  - `imessage`: handles/chat ids
  - `webchat`: session ids or usernames

ဥပမာ —

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

Per-agent override (further restrict):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

မှတ်ချက်များ-

- `tools.elevated` is the global baseline. `agents.list[].tools.elevated` can only further restrict (both must allow).
- `/elevated on|off|ask|full` သည် session key အလိုက် state ကို သိမ်းဆည်းပါသည်; inline directives များသည် မက်ဆေ့ချ်တစ်ခုတည်းအတွက်သာ သက်ရောက်ပါသည်။
- Elevated `exec` runs on the host and bypasses sandboxing.
- Tool policy still applies; if `exec` is denied, elevated cannot be used.

`agents.defaults.maxConcurrent` sets the maximum number of embedded agent runs that can
execute in parallel across sessions. Each session is still serialized (one run
per session key at a time). Default: 1.

### `agents.defaults.sandbox`

Optional **Docker sandboxing** for the embedded agent. Intended for non-main
sessions so they cannot access your host system.

Details: [Sandboxing](/gateway/sandboxing)

Defaults (if enabled):

- scope: `"agent"` (agent တစ်ခုလျှင် container တစ်ခု + workspace တစ်ခု)
- Debian bookworm-slim based image
- agent workspace access: `workspaceAccess: "none"` (default)
  - `"none"`: use a per-scope sandbox workspace under `~/.openclaw/sandboxes`
- `"ro"`: keep the sandbox workspace at `/workspace`, and mount the agent workspace read-only at `/agent` (disables `write`/`edit`/`apply_patch`)
  - `"rw"`: mount the agent workspace read/write at `/workspace`
- auto-prune: idle > 24 နာရီ သို့မဟုတ် age > 7 ရက်
- tool policy: allow only `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (deny wins)
  - configure via `tools.sandbox.tools`, override per-agent via `agents.list[].tools.sandbox.tools`
  - tool group shorthands supported in sandbox policy: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (see [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- optional sandboxed browser (Chromium + CDP, noVNC observer)
- hardening knobs: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Warning: `scope: "shared"` means a shared container and shared workspace. No
cross-session isolation. Use `scope: "session"` for per-session isolation.

Legacy: `perSession` is still supported (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` runs **once** after the container is created (inside the container via `sh -lc`).
For package installs, ensure network egress, a writable root FS, and a root user.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
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
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
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
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
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

Build the default sandbox image once with:

```bash
scripts/sandbox-setup.sh
```

Note: sandbox containers default to `network: "none"`; set `agents.defaults.sandbox.docker.network`
to `"bridge"` (or your custom network) if the agent needs outbound access.

Note: inbound attachments are staged into the active workspace at `media/inbound/*`. With `workspaceAccess: "rw"`, that means files are written into the agent workspace.

Note: `docker.binds` mounts additional host directories; global and per-agent binds are merged.

Build the optional browser image with:

```bash
scripts/sandbox-browser-setup.sh
```

When `agents.defaults.sandbox.browser.enabled=true`, the browser tool uses a sandboxed
Chromium instance (CDP). If noVNC is enabled (default when headless=false),
the noVNC URL is injected into the system prompt so the agent can reference it.
This does not require `browser.enabled` in the main config; the sandbox control
URL is injected per session.

`agents.defaults.sandbox.browser.allowHostControl` (default: false) allows
sandboxed sessions to explicitly target the **host** browser control server
via the browser tool (`target: "host"`). Leave this off if you want strict
sandbox isolation.

Allowlists for remote control:

- `allowedControlUrls`: exact control URLs permitted for `target: "custom"`.
- `allowedControlHosts`: hostnames permitted (hostname only, no port).
- `allowedControlPorts`: ports permitted (defaults: http=80, https=443).
  ပုံမှန်သတ်မှတ်ချက်များ: allowlist အားလုံးကို မသတ်မှတ်ထားပါ (ကန့်သတ်ချက်မရှိ)။ `allowHostControl` ၏ default တန်ဖိုးမှာ false ဖြစ်ပါသည်။

### `models` (custom provider များ + base URL များ)

OpenClaw သည် **pi-coding-agent** model catalog ကို အသုံးပြုပါသည်။ Custom provider များကို ထည့်သွင်းနိုင်ပါသည်
(LiteLLM, local OpenAI-compatible server များ, Anthropic proxy များ စသည်ဖြင့်) အောက်ပါနေရာတွင် ရေးသားခြင်းဖြင့်
`~/.openclaw/agents/<agentId>/agent/models.json` သို့မဟုတ် သင်၏
OpenClaw config အတွင်းရှိ `models.providers` အောက်တွင် အလားတူ schema ကို သတ်မှတ်ခြင်းဖြင့်။
Provider တစ်ခုချင်းစီအလိုက် အကျဉ်းချုပ် + ဥပမာများ: [/concepts/model-providers](/concepts/model-providers)။

`models.providers` ပါရှိနေပါက OpenClaw သည် startup အချိန်တွင် `models.json` ကို
`~/.openclaw/agents/<agentId>/agent/` အောက်သို့ ရေးသား/ပေါင်းစည်းပါသည်။

- ပုံမှန်အပြုအမူ: **merge** (ရှိပြီးသား provider များကို ထိန်းသိမ်းထားပြီး နာမည်အပေါ်မူတည်၍ override လုပ်သည်)
- ဖိုင်အကြောင်းအရာအားလုံးကို အစားထိုးလိုပါက `models.mode: "replace"` ဟု သတ်မှတ်ပါ။

`agents.defaults.model.primary` (provider/model) မှတဆင့် model ကို ရွေးချယ်ပါ။

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3.1-8b" },
      models: {
        "custom-proxy/llama-3.1-8b": {},
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
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

### OpenCode Zen (multi-model proxy)

OpenCode Zen သည် မော်ဒယ်တစ်ခုချင်းစီအလိုက် endpoint များပါရှိသော multi-model gateway တစ်ခုဖြစ်သည်။ OpenClaw သည်
pi-ai မှ built-in `opencode` provider ကို အသုံးပြုပါသည်; [https://opencode.ai/auth](https://opencode.ai/auth) မှ `OPENCODE_API_KEY` (သို့မဟုတ်
`OPENCODE_ZEN_API_KEY`) ကို သတ်မှတ်ပါ။

မှတ်ချက်များ-

- Model reference များတွင် `opencode/<modelId>` ကို အသုံးပြုပါသည် (ဥပမာ: `opencode/claude-opus-4-6`)။
- `agents.defaults.models` မှတဆင့် allowlist ကို ဖွင့်ထားပါက သင်အသုံးပြုရန် စီစဉ်ထားသော model တစ်ခုချင်းစီကို ထည့်ပါ။
- Shortcut: `openclaw onboard --auth-choice opencode-zen`။

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

### Z.AI (GLM-4.7) — provider alias အထောက်အပံ့

Z.AI model များကို built-in `zai` provider မှတဆင့် အသုံးပြုနိုင်ပါသည်။ Environment အတွင်း `ZAI_API_KEY` ကို သတ်မှတ်ပြီး provider/model ဖြင့် model ကို reference လုပ်ပါ။

Shortcut: `openclaw onboard --auth-choice zai-api-key`။

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

မှတ်ချက်များ-

- `z.ai/*` နှင့် `z-ai/*` ကို alias အဖြစ် လက်ခံပြီး `zai/*` သို့ normalize လုပ်ပါသည်။
- `ZAI_API_KEY` မရှိပါက `zai/*` သို့ request များသည် runtime တွင် auth error ဖြင့် မအောင်မြင်ပါ။
- ဥပမာ error: `No API key found for provider "zai".`
- Z.AI ၏ အထွေထွေ API endpoint သည် `https://api.z.ai/api/paas/v4` ဖြစ်သည်။ GLM coding request များသည် သီးသန့် Coding endpoint ဖြစ်သော `https://api.z.ai/api/coding/paas/v4` ကို အသုံးပြုပါသည်။
  Built-in `zai` provider သည် Coding endpoint ကို အသုံးပြုပါသည်။ အထွေထွေ endpoint ကို လိုအပ်ပါက `models.providers` အတွင်း custom provider ကို သတ်မှတ်ပြီး base URL ကို override လုပ်ပါ (အထက်ပါ custom providers အပိုင်းကို ကြည့်ပါ)။
- Docs/config များတွင် fake placeholder ကို အသုံးပြုပါ; အမှန်တကယ် API key များကို ဘယ်တော့မှ commit မလုပ်ပါနှင့်။

### Moonshot AI (Kimi)

Moonshot ၏ OpenAI-compatible endpoint ကို အသုံးပြုပါ:

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

မှတ်ချက်များ-

- Environment အတွင်း `MOONSHOT_API_KEY` ကို သတ်မှတ်ပါ သို့မဟုတ် `openclaw onboard --auth-choice moonshot-api-key` ကို အသုံးပြုပါ။
- Model reference: `moonshot/kimi-k2.5`။
- China endpoint အတွက် အောက်ပါအတိုင်းလုပ်ဆောင်နိုင်ပါသည်:
  - `openclaw onboard --auth-choice moonshot-api-key-cn` ကို run လုပ်ပါ (wizard သည် `https://api.moonshot.cn/v1` ကို သတ်မှတ်ပေးမည်) သို့မဟုတ်
  - `models.providers.moonshot` အတွင်း `baseUrl: "https://api.moonshot.cn/v1"` ကို လက်ဖြင့် သတ်မှတ်ပါ။

### Kimi Coding

Moonshot AI ၏ Kimi Coding endpoint (Anthropic-compatible, built-in provider) ကို အသုံးပြုပါ:

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

မှတ်ချက်များ-

- Environment အတွင်း `KIMI_API_KEY` ကို သတ်မှတ်ပါ သို့မဟုတ် `openclaw onboard --auth-choice kimi-code-api-key` ကို အသုံးပြုပါ။
- Model reference: `kimi-coding/k2p5`။

### Synthetic (Anthropic-compatible)

Synthetic ၏ Anthropic-compatible endpoint ကို အသုံးပြုပါ:

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

မှတ်ချက်များ-

- `SYNTHETIC_API_KEY` ကို သတ်မှတ်ပါ သို့မဟုတ် `openclaw onboard --auth-choice synthetic-api-key` ကို အသုံးပြုပါ။
- Model reference: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`။
- Anthropic client သည် `/v1` ကို အလိုအလျောက် ထည့်ပေးသောကြောင့် Base URL တွင် `/v1` ကို မထည့်ပါနှင့်။

### Local model များ (LM Studio) — အကြံပြုထားသော setup

လက်ရှိ local အတွက် လမ်းညွှန်ချက်များကို [/gateway/local-models](/gateway/local-models) တွင် ကြည့်ပါ။ 1. TL;DR: အားကောင်းတဲ့ ဟာ့ဒ်ဝဲပေါ်에서 LM Studio Responses API ကို အသုံးပြုပြီး MiniMax M2.1 ကို chạy လုပ်ပါ; fallback အတွက် hosted models ကို merge လုပ်ထားပါ။

### MiniMax M2.1

2. LM Studio မသုံးဘဲ MiniMax M2.1 ကို တိုက်ရိုက် အသုံးပြုပါ:

```json5
3. {
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
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
            // Pricing: update in models.json if you need exact cost tracking.
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

မှတ်ချက်များ-

- 4. `MINIMAX_API_KEY` environment variable ကို သတ်မှတ်ပါ သို့မဟုတ် `openclaw onboard --auth-choice minimax-api` ကို အသုံးပြုပါ။
- 5. ရရှိနိုင်သော မော်ဒယ်: `MiniMax-M2.1` (default)။
- 6. တိကျတဲ့ ကုန်ကျစရိတ် ခြေရာခံရန် လိုအပ်ပါက `models.json` ထဲမှာ pricing ကို update လုပ်ပါ။

### 7. Cerebras (GLM 4.6 / 4.7)

8. Cerebras ကို သူတို့ရဲ့ OpenAI-compatible endpoint မှတဆင့် အသုံးပြုပါ:

```json5
9. {
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

မှတ်ချက်များ-

- 10. Cerebras အတွက် `cerebras/zai-glm-4.7` ကို အသုံးပြုပါ; Z.AI ကို တိုက်ရိုက် အသုံးပြုလျှင် `zai/glm-4.7` ကို အသုံးပြုပါ။
- 11. `CEREBRAS_API_KEY` ကို environment သို့မဟုတ် config ထဲမှာ သတ်မှတ်ပါ။

မှတ်ချက်များ-

- 12. ပံ့ပိုးထားသော APIs: `openai-completions`, `openai-responses`, `anthropic-messages`,
      `google-generative-ai`
- 13. စိတ်ကြိုက် auth လိုအပ်ချက်များအတွက် `authHeader: true` + `headers` ကို အသုံးပြုပါ။
- 14. `models.json` ကို အခြားနေရာတွင် သိမ်းဆည်းလိုပါက `OPENCLAW_AGENT_DIR` (သို့မဟုတ် `PI_CODING_AGENT_DIR`) ဖြင့် agent config root ကို override လုပ်နိုင်ပါသည် (default: `~/.openclaw/agents/main/agent`)။

### `session`

15. session scope, reset policy, reset triggers နှင့် session store ကို ဘယ်မှာ ရေးသားမလဲကို ထိန်းချုပ်ပါသည်။

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
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

Fields —

- 17. `mainKey`: direct-chat bucket key (default: `"main"`)။ 18. `agentId` ကို မပြောင်းဘဲ primary DM thread ကို “အမည်ပြောင်း” လုပ်ချင်တဲ့အခါ အသုံးဝင်ပါသည်။
  - 19. Sandbox မှတ်ချက်: `agents.defaults.sandbox.mode: "non-main"` သည် main session ကို ခွဲခြားသိရန် ဒီ key ကို အသုံးပြုပါသည်။ 20. `mainKey` နဲ့ မကိုက်ညီတဲ့ session key မည်သည့်အရာမဆို (groups/channels) ကို sandbox လုပ်ထားပါသည်။
- 21. `dmScope`: DM sessions ကို ဘယ်လို group လုပ်မလဲ (default: `"main"`)။
  - 22. `main`: ဆက်လက်ညှိနှိုင်းမှု ရရှိစေရန် DM အားလုံးက main session ကို မျှဝေပါသည်။
  - 23. `per-peer`: channel များအနှံ့ sender id အလိုက် DM ကို ခွဲထားပါသည်။
  - 24. `per-channel-peer`: channel + sender အလိုက် DM ကို ခွဲထားပါသည် (multi-user inboxes အတွက် အကြံပြုထားသည်)။
  - 25. `per-account-channel-peer`: account + channel + sender အလိုက် DM ကို ခွဲထားပါသည် (multi-account inboxes အတွက် အကြံပြုထားသည်)။
  - 26. လုံခြုံသော DM mode (အကြံပြု): လူအများက bot ကို DM လုပ်နိုင်သည့်အခါ (`shared inboxes`, multi-person allowlists သို့မဟုတ် `dmPolicy: "open"`) `session.dmScope: "per-channel-peer"` ကို သတ်မှတ်ပါ။
- 27. `identityLinks`: canonical ids ကို provider-prefix ပါသော peers များနဲ့ map လုပ်ပေးပြီး `per-peer`, `per-channel-peer`, သို့မဟုတ် `per-account-channel-peer` ကို အသုံးပြုသောအခါ channel များအနှံ့ လူတစ်ဦးတည်းက DM session တစ်ခုကို မျှဝေနိုင်စေပါသည်။
  - 28. ဥပမာ: `alice: ["telegram:123456789", "discord:987654321012345678"]`။
- 29. `reset`: အဓိက reset policy။ 30. gateway host ရဲ့ local time အရ မနက် 4:00 နာရီမှာ နေ့စဉ် reset လုပ်ခြင်းကို default အဖြစ် သတ်မှတ်ထားပါသည်။
  - 31. `mode`: `daily` သို့မဟုတ် `idle` (`reset` ရှိပါက default သည် `daily`)။
  - 32. `atHour`: နေ့စဉ် reset boundary အတွက် local hour (0-23)။
  - 33. `idleMinutes`: sliding idle window ကို မိနစ်ဖြင့် သတ်မှတ်ပါသည်။ 34. daily + idle နှစ်ခုစလုံးကို သတ်မှတ်ထားပါက အရင်ဆုံး သက်တမ်းကုန်တဲ့ အရာက အနိုင်ရပါသည်။
- `resetByType`: `direct`, `group`, နှင့် `thread` အတွက် session တစ်ခုချင်းစီအလိုက် override များ။ Legacy `dm` key ကို `direct` ၏ alias အဖြစ် လက်ခံပါသည်။
  - 36. legacy `session.idleMinutes` ကိုသာ သတ်မှတ်ပြီး `reset`/`resetByType` မရှိပါက backward compatibility အတွက် OpenClaw သည် idle-only mode အဖြစ် ဆက်လက် လုပ်ဆောင်ပါသည်။
- 37. `heartbeatIdleMinutes`: heartbeat စစ်ဆေးမှုများအတွက် optional idle override (enable ဖြစ်ပါက daily reset သည် ဆက်လက် သက်ရောက်ပါသည်)။
- 38. `agentToAgent.maxPingPongTurns`: requester/target အကြား reply-back turns အများဆုံး (0–5, default 5)။
- 39. `sendPolicy.default`: rule မကိုက်ညီပါက fallback အဖြစ် `allow` သို့မဟုတ် `deny`။
- 40. `sendPolicy.rules[]`: `channel`, `chatType` (`direct|group|room`), သို့မဟုတ် `keyPrefix` (ဥပမာ `cron:`) အလိုက် match လုပ်ပါသည်။ 41. ပထမဆုံး deny က အနိုင်ရပါသည်; မဟုတ်ပါက allow ဖြစ်ပါသည်။

### 42. `skills` (skills config)

43. bundled allowlist, install preferences, extra skill folders နှင့် per-skill override များကို ထိန်းချုပ်ပါသည်။ 44. **bundled** skills နှင့် `~/.openclaw/skills` ကို သက်ရောက်ပါသည် (workspace skills များက အမည် တူညီပါက အနိုင်ရပါသည်)။

Fields —

- 45. `allowBundled`: **bundled** skills များအတွက်သာ optional allowlist။ 46. သတ်မှတ်ထားပါက အဆိုပါ bundled skills များသာ eligible ဖြစ်ပြီး (managed/workspace skills များကို မသက်ရောက်ပါ)။
- `load.extraDirs`: စကင်လုပ်ရန် ထည့်သွင်းပေးထားသော skill ဒိုင်ရက်ထရီများ (အနိမ့်ဆုံး ဦးစားပေးအဆင့်)။
- `install.preferBrew`: ရနိုင်ပါက brew installer များကို ဦးစားပေးအသုံးပြုခြင်း (မူလသတ်မှတ်ချက်: true)။
- 47. `install.nodeManager`: node installer preference (`npm` | `pnpm` | `yarn`, default: npm)။
- 48. `entries.<skillKey>`49. \`: per-skill config override များ။

Skill တစ်ခုချင်းစီအတွက် fields များ:

- `enabled`: bundled/installed ဖြစ်နေသော်လည်း skill တစ်ခုကို ပိတ်ရန် `false` ကို သတ်မှတ်နိုင်သည်။
- `env`: agent ကို run လုပ်စဉ် ထည့်သွင်းပေးမည့် environment variables (မတိုင်မီ သတ်မှတ်ထားပြီးသား မဟုတ်ပါကသာ)။
- 50. `apiKey`: primary env var ကို ကြေညာထားသော skills များအတွက် optional convenience (ဥပမာ `nano-banana-pro` → `GEMINI_API_KEY`)။

ဥပမာ —

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins` (extensions)

Controls plugin discovery, allow/deny, and per-plugin config. Plugins are loaded
from `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus any
`plugins.load.paths` entries. **Config changes require a gateway restart.**
See [/plugin](/tools/plugin) for full usage.

Fields —

- `enabled`: master toggle for plugin loading (default: true).
- `allow`: optional allowlist of plugin ids; when set, only listed plugins load.
- `deny`: optional denylist of plugin ids (deny wins).
- `load.paths`: extra plugin files or directories to load (absolute or `~`).
- `entries.<pluginId>`: per-plugin overrides.
  - `enabled`: set `false` to disable.
  - `config`: plugin-specific config object (validated by the plugin if provided).

ဥပမာ —

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw can start a **dedicated, isolated** Chrome/Brave/Edge/Chromium instance for openclaw and expose a small loopback control service.
Profiles can point at a **remote** Chromium-based browser via `profiles.<name>.cdpUrl`. Remote
profiles are attach-only (start/stop/reset are disabled).

`browser.cdpUrl` remains for legacy single-profile configs and as the base
scheme/host for profiles that only set `cdpPort`.

Defaults:

- enabled: `true`
- evaluateEnabled: `true` (set `false` to disable `act:evaluate` and `wait --fn`)
- control service: loopback only (port derived from `gateway.port`, default `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profile)
- profile color: `#FF4500` (lobster-orange)
- Note: the control server is started by the running gateway (OpenClaw.app menubar, or `openclaw gateway`).
- Auto-detect order: default browser if Chromium-based; otherwise Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // set true when tunneling a remote CDP to localhost
  },
}
```

### `ui` (Appearance)

Optional accent color used by the native apps for UI chrome (e.g. Talk Mode bubble tint).

If unset, clients fall back to a muted light-blue.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Control UI assistant identity override.
    // If unset, the Control UI uses the active agent identity (config or IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, or image URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Use `gateway.mode` to explicitly declare whether this machine should run the Gateway.

Defaults:

- mode: **unset** (treated as “do not auto-start”)
- bind: `loopback`
- port: `18789` (single port for WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

Control UI base path:

- `gateway.controlUi.basePath` sets the URL prefix where the Control UI is served.
- Examples: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- Default: root (`/`) (unchanged).
- `gateway.controlUi.root` sets the filesystem root for Control UI assets (default: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` allows token-only auth for the Control UI when
  device identity is omitted (typically over HTTP). Default: `false`. Prefer HTTPS
  (Tailscale Serve) or `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` disables device identity checks for the
  Control UI (token/password only). Default: `false`. Break-glass only.

ဆက်စပ်စာရွက်စာတမ်းများ—

- [Control UI](/web/control-ui)
- [Web overview](/web)
- [Tailscale](/gateway/tailscale)
- [Remote access](/gateway/remote)

Trusted proxies-

- `gateway.trustedProxies`: list of reverse proxy IPs that terminate TLS in front of the Gateway.
- When a connection comes from one of these IPs, OpenClaw uses `x-forwarded-for` (or `x-real-ip`) to determine the client IP for local pairing checks and HTTP auth/local checks.
- Only list proxies you fully control, and ensure they **overwrite** incoming `x-forwarded-for`.

မှတ်ချက်များ —

- `openclaw gateway` refuses to start unless `gateway.mode` is set to `local` (or you pass the override flag).
- `gateway.port` controls the single multiplexed port used for WebSocket + HTTP (control UI, hooks, A2UI).
- OpenAI Chat Completions endpoint: **disabled by default**; enable with `gateway.http.endpoints.chatCompletions.enabled: true`.
- Precedence: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > default `18789`.
- Gateway auth is required by default (token/password or Tailscale Serve identity). Non-loopback binds require a shared token/password.
- The onboarding wizard generates a gateway token by default (even on loopback).
- `gateway.remote.token` is **only** for remote CLI calls; it does not enable local gateway auth. `gateway.token` is ignored.

Auth and Tailscale:

- `gateway.auth.mode` sets the handshake requirements (`token` or `password`). When unset, token auth is assumed.
- `gateway.auth.token` stores the shared token for token auth (used by the CLI on the same machine).
- When `gateway.auth.mode` is set, only that method is accepted (plus optional Tailscale headers).
- `gateway.auth.password` can be set here, or via `OPENCLAW_GATEWAY_PASSWORD` (recommended).
- `gateway.auth.allowTailscale` allows Tailscale Serve identity headers
  (`tailscale-user-login`) to satisfy auth when the request arrives on loopback
  with `x-forwarded-for`, `x-forwarded-proto`, and `x-forwarded-host`. OpenClaw
  verifies the identity by resolving the `x-forwarded-for` address via
  `tailscale whois` before accepting it. When `true`, Serve requests do not need
  a token/password; set `false` to require explicit credentials. Defaults to
  `true` when `tailscale.mode = "serve"` and auth mode is not `password`.
- `gateway.tailscale.mode: "serve"` uses Tailscale Serve (tailnet only, loopback bind).
- `gateway.tailscale.mode: "funnel"` exposes the dashboard publicly; requires auth.
- `gateway.tailscale.resetOnExit` resets Serve/Funnel config on shutdown.

Remote client defaults (CLI):

- `gateway.remote.url` sets the default Gateway WebSocket URL for CLI calls when `gateway.mode = "remote"`.
- `gateway.remote.transport` selects the macOS remote transport (`ssh` default, `direct` for ws/wss). When `direct`, `gateway.remote.url` must be `ws://` or `wss://`. `ws://host` defaults to port `18789`.
- `gateway.remote.token` supplies the token for remote calls (leave unset for no auth).
- `gateway.remote.password` supplies the password for remote calls (leave unset for no auth).

macOS app behavior:

- OpenClaw.app watches `~/.openclaw/openclaw.json` and switches modes live when `gateway.mode` or `gateway.remote.url` changes.
- If `gateway.mode` is unset but `gateway.remote.url` is set, the macOS app treats it as remote mode.
- When you change connection mode in the macOS app, it writes `gateway.mode` (and `gateway.remote.url` + `gateway.remote.transport` in remote mode) back to the config file.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Direct transport example (macOS app):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

Modes:

- `hybrid` (default): hot-apply safe changes; restart the Gateway for critical changes.
- `hot`: only apply hot-safe changes; log when a restart is required.
- `restart`: restart the Gateway on any config change.
- `off`: disable hot reload.

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

#### Hot reload matrix (files + impact)

Files watched:

- `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`)

Hot-applied (no full gateway restart):

- `hooks` (webhook အတည်ပြုခြင်း/auth၊ path၊ mappings) + `hooks.gmail` (Gmail watcher ကို ပြန်လည်စတင်ထားသည်)
- `browser` (browser ထိန်းချုပ်ရေး server ကို ပြန်လည်စတင်)
- `cron` (cron service ကို ပြန်လည်စတင် + concurrency အပ်ဒိတ်)
- `agents.defaults.heartbeat` (heartbeat runner ကို ပြန်လည်စတင်)
- `web` (WhatsApp web channel ကို ပြန်လည်စတင်)
- `telegram`, `discord`, `signal`, `imessage` (channel များကို ပြန်လည်စတင်)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (dynamic reads)

Gateway ကို အပြည့်အဝ ပြန်လည်စတင်ရန် လိုအပ်သည်:

- `gateway` (port/bind/auth/control UI/tailscale)
- `bridge` (legacy)
- `discovery`
- `canvasHost`
- `plugins`
- မသိရှိသော/မထောက်ပံ့ထားသော config path မည်သည့်အရာမဆို (လုံခြုံရေးအတွက် default အနေဖြင့် restart လုပ်မည်)

### Multi-instance isolation

host တစ်ခုတည်းပေါ်တွင် gateway များစွာကို chạy လိုပါက (redundancy သို့မဟုတ် rescue bot အတွက်) instance တစ်ခုချင်းစီအလိုက် state + config ကို ခွဲထားပြီး port မတူအောင် အသုံးပြုပါ:

- `OPENCLAW_CONFIG_PATH` (instance တစ်ခုချင်းစီအတွက် config)
- `OPENCLAW_STATE_DIR` (sessions/creds)
- `agents.defaults.workspace` (memories)
- `gateway.port` (instance တစ်ခုချင်းစီအတွက် သီးသန့်)

အဆင်ပြေမှု flags (CLI):

- `openclaw --dev …` → `~/.openclaw-dev` ကို အသုံးပြုပြီး base `19001` မှ ports ကို ရွှေ့ထားသည်
- `openclaw --profile <name> …` → `~/.openclaw-<name>` ကို အသုံးပြုသည် (port ကို config/env/flags ဖြင့် သတ်မှတ်)

derived port mapping (gateway/browser/canvas) အတွက် [Gateway runbook](/gateway) ကို ကြည့်ပါ။
browser/CDP port isolation အသေးစိတ်အတွက် [Multiple gateways](/gateway/multiple-gateways) ကို ကြည့်ပါ။

ဥပမာ:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (Gateway webhooks)

Gateway HTTP server ပေါ်တွင် ရိုးရှင်းသော HTTP webhook endpoint တစ်ခုကို ဖွင့်ပါ။

မူလတန်ဖိုးများ—

- enabled: `false`
- path: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
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

Request များတွင် hook token ကို ထည့်သွင်းရပါမည်:

- `Authorization: Bearer <token>` **သို့မဟုတ်**
- `x-openclaw-token: <token>`

Endpoints:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\`
- `POST /hooks/<name>` → `hooks.mappings` ဖြင့် resolve လုပ်သည်

`/hooks/agent` သည် အမြဲတမ်း main session ထဲသို့ summary တစ်ခုကို post လုပ်ပြီး (`wakeMode: "now"` ဖြင့် immediate heartbeat ကို ရွေးချယ်၍ trigger လုပ်နိုင်သည်)။

Mapping ဆိုင်ရာ မှတ်စုများ:

- `match.path` သည် `/hooks` အပြီးရှိ sub-path ကို ကိုက်ညီစေသည် (ဥပမာ `/hooks/gmail` → `gmail`)။
- `match.source` သည် payload field တစ်ခုကို ကိုက်ညီစေသည် (ဥပမာ `{ source: "gmail" }`)၊ ထို့ကြောင့် generic `/hooks/ingest` path ကို အသုံးပြုနိုင်သည်။
- `{{messages[0].subject}}` ကဲ့သို့သော template များသည် payload မှ ဖတ်ယူသည်။
- `transform` သည် hook action ကို ပြန်ပေးသော JS/TS module တစ်ခုကို ညွှန်းနိုင်သည်။
- `deliver: true` သည် နောက်ဆုံး reply ကို channel တစ်ခုသို့ ပို့သည်; `channel` ၏ default သည် `last` ဖြစ်ပြီး (WhatsApp သို့ fallback ဖြစ်သည်)။
- ယခင် delivery route မရှိပါက `channel` + `to` ကို ထင်ရှားစွာ သတ်မှတ်ပါ (Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams အတွက် မဖြစ်မနေ လိုအပ်သည်)။
- `model` သည် ဤ hook run အတွက် LLM ကို override လုပ်သည် (`provider/model` သို့မဟုတ် alias; `agents.defaults.models` ကို သတ်မှတ်ထားပါက ခွင့်ပြုထားရပါမည်)။

Gmail helper config (`openclaw webhooks gmail setup` / `run` မှ အသုံးပြုသည်):

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

      // Optional: use a cheaper model for Gmail hook processing
      // Falls back to agents.defaults.model.fallbacks, then primary, on auth/rate-limit/timeout
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // Optional: default thinking level for Gmail hooks
      thinking: "off",
    },
  },
}
```

Gmail hooks အတွက် model override:

- `hooks.gmail.model` သည် Gmail hook processing အတွက် အသုံးပြုမည့် model ကို သတ်မှတ်သည် (default အနေဖြင့် session primary ကို အသုံးပြုသည်)။
- Accepts `provider/model` refs or aliases from `agents.defaults.models`.
- Falls back to `agents.defaults.model.fallbacks`, then `agents.defaults.model.primary`, on auth/rate-limit/timeouts.
- If `agents.defaults.models` is set, include the hooks model in the allowlist.
- At startup, warns if the configured model is not in the model catalog or allowlist.
- `hooks.gmail.thinking` sets the default thinking level for Gmail hooks and is overridden by per-hook `thinking`.

Gateway auto-start:

- If `hooks.enabled=true` and `hooks.gmail.account` is set, the Gateway starts
  `gog gmail watch serve` on boot and auto-renews the watch.
- Set `OPENCLAW_SKIP_GMAIL_WATCHER=1` to disable the auto-start (for manual runs).
- Avoid running a separate `gog gmail watch serve` alongside the Gateway; it will
  fail with `listen tcp 127.0.0.1:8788: bind: address already in use`.

Note: when `tailscale.mode` is on, OpenClaw defaults `serve.path` to `/` so
Tailscale can proxy `/gmail-pubsub` correctly (it strips the set-path prefix).
If you need the backend to receive the prefixed path, set
`hooks.gmail.tailscale.target` to a full URL (and align `serve.path`).

### `canvasHost` (LAN/tailnet Canvas file server + live reload)

The Gateway serves a directory of HTML/CSS/JS over HTTP so iOS/Android nodes can simply `canvas.navigate` to it.

Default root: `~/.openclaw/workspace/canvas`  
Default port: `18793` (chosen to avoid the openclaw browser CDP port `18792`)  
The server listens on the **gateway bind host** (LAN or Tailnet) so nodes can reach it.

The server:

- serves files under `canvasHost.root`
- injects a tiny live-reload client into served HTML
- watches the directory and broadcasts reloads over a WebSocket endpoint at `/__openclaw__/ws`
- auto-creates a starter `index.html` when the directory is empty (so you see something immediately)
- also serves A2UI at `/__openclaw__/a2ui/` and is advertised to nodes as `canvasHostUrl`
  (always used by nodes for Canvas/A2UI)

Disable live reload (and file watching) if the directory is large or you hit `EMFILE`:

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

Changes to `canvasHost.*` require a gateway restart (config reload will restart).

ပိတ်ရန်-

- config: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (legacy TCP bridge, removed)

Current builds no longer include the TCP bridge listener; `bridge.*` config keys are ignored.
Nodes connect over the Gateway WebSocket. This section is kept for historical reference.

Legacy behavior:

- The Gateway could expose a simple TCP bridge for nodes (iOS/Android), typically on port `18790`.

Defaults:

- enabled: `true`
- port: `18790`
- bind: `lan` (binds to `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (reachable on any interface, including LAN/Wi‑Fi and Tailscale)
- `tailnet`: bind only to the machine’s Tailscale IP (recommended for Vienna ⇄ London)
- `loopback`: `127.0.0.1` (local only)
- `auto`: prefer tailnet IP if present, else `lan`

TLS:

- `bridge.tls.enabled`: enable TLS for bridge connections (TLS-only when enabled).
- `bridge.tls.autoGenerate`: generate a self-signed cert when no cert/key are present (default: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: PEM paths for the bridge certificate + private key.
- `bridge.tls.caPath`: optional PEM CA bundle (custom roots or future mTLS).

When TLS is enabled, the Gateway advertises `bridgeTls=1` and `bridgeTlsSha256` in discovery TXT
records so nodes can pin the certificate. Manual connections use trust-on-first-use if no
fingerprint is stored yet.
Auto-generated certs require `openssl` on PATH; if generation fails, the bridge will not start.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // Uses ~/.openclaw/bridge/tls/bridge-{cert,key}.pem when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS broadcast mode)

Controls LAN mDNS discovery broadcasts (`_openclaw-gw._tcp`).

- `minimal` (default): omit `cliPath` + `sshPort` from TXT records
- `full`: include `cliPath` + `sshPort` in TXT records
- `off`: disable mDNS broadcasts entirely
- Hostname: defaults to `openclaw` (advertises `openclaw.local`). Override with `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS‑SD)

When enabled, the Gateway writes a unicast DNS-SD zone for `_openclaw-gw._tcp` under `~/.openclaw/dns/` using the configured discovery domain (example: `openclaw.internal.`).

To make iOS/Android discover across networks (Vienna ⇄ London), pair this with:

- a DNS server on the gateway host serving your chosen domain (CoreDNS is recommended)
- Tailscale **split DNS** so clients resolve that domain via the gateway DNS server

One-time setup helper (gateway host):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Media model template variables

Template placeholders are expanded in `tools.media.*.models[].args` and `tools.media.models[].args` (and any future templated argument fields).

\| Variable           | Description                                                                     |
\| ------------------ | ------------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | --- |
\| `{{Body}}`         | Full inbound message body                                                       |
\| `{{RawBody}}`      | Raw inbound message body (no history/sender wrappers; best for command parsing) |
\| `{{BodyStripped}}` | Body with group mentions stripped (best default for agents)                     |
\| `{{From}}`         | Sender identifier (E.164 for WhatsApp; may differ per channel)                  |
\| `{{To}}`           | Destination identifier                                                          |
\| `{{MessageSid}}`   | Channel message id (when available)                                             |
\| `{{SessionId}}`    | Current session UUID                                                            |
\| `{{IsNewSession}}` | `"true"` when a new session was created                                         |
\| `{{MediaUrl}}`     | Inbound media pseudo-URL (if present)                                           |
\| `{{MediaPath}}`    | Local media path (if downloaded)                                                |
\| `{{MediaType}}`    | Media type (image/audio/document/…)                                             |
\| `{{Transcript}}`   | Audio transcript (when enabled)                                                 |
\| `{{Prompt}}`       | Resolved media prompt for CLI entries                                           |
\| `{{MaxChars}}`     | Resolved max output chars for CLI entries                                       |
\| `{{ChatType}}`     | `"direct"` or `"group"`                                                         |
\| `{{GroupSubject}}` | Group subject (best effort)                                                     |
\| `{{GroupMembers}}` | Group members preview (best effort)                                             |
\| `{{SenderName}}`   | Sender display name (best effort)                                               |
\| `{{SenderE164}}`   | Sender phone number (best effort)                                               |
\| `{{Provider}}`     | Provider hint (whatsapp                                                         | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (Gateway scheduler)

Cron is a Gateway-owned scheduler for wakeups and scheduled jobs. See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_Next: [Agent Runtime](/concepts/agent)_ 🦞
