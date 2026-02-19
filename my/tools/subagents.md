---
summary: "Sub-agents: တောင်းဆိုသူ၏ ချတ်ချန်နယ်သို့ ရလဒ်များကို ကြေညာပြန်လည်ပို့သည့် သီးခြားခွဲထားသော agent run များကို spawn ပြုလုပ်ခြင်း"
read_when:
  - agent မှတစ်ဆင့် နောက်ခံ/အပြိုင် လုပ်ဆောင်မှုများ လိုအပ်သည့်အခါ
  - sessions_spawn သို့မဟုတ် sub-agent tool policy ကို ပြောင်းလဲနေသည့်အခါ
title: "Sub-Agents"
---

# Sub-agents

Sub-agents များသည် လက်ရှိ agent run တစ်ခုမှ စတင်ဖန်တီးထားသော background agent runs များဖြစ်သည်။ ၎င်းတို့သည် ကိုယ်ပိုင် session (`agent:<agentId>:subagent:<uuid>`) အတွင်း လည်ပတ်ပြီး ပြီးဆုံးသည့်အခါ ရလဒ်ကို တောင်းဆိုသူ chat channel သို့ **announce** ပြုလုပ်မည်။

## Slash command

**လက်ရှိ session** အတွက် sub-agent runs များကို စစ်ဆေးရန် သို့မဟုတ် ထိန်းချုပ်ရန် `/subagents` ကို အသုံးပြုပါ:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` သည် run metadata (status, timestamps, session id, transcript path, cleanup) ကို ပြသသည်။

အဓိက ရည်ရွယ်ချက်များ:

- အဓိက run ကို မတားဆီးဘဲ "research / long task / slow tool" အလုပ်များကို parallel ပြုလုပ်ရန်။
- Sub-agents များကို default အနေဖြင့် သီးခြားထားရန် (session ခွဲခြားခြင်း + optional sandboxing)။
- tool surface ကို အလွဲအသုံးချရန် ခက်ခဲစေရန် ထိန်းသိမ်းထားရန် — sub-agents များသည် default အနေဖြင့် session tools မရရှိပါ။
- orchestrator ပုံစံများအတွက် စီမံခန့်ခွဲနိုင်သော nesting depth ကို ထောက်ပံ့ပါ။

ကုန်ကျစရိတ် မှတ်ချက် - sub-agent တစ်ခုချင်းစီတွင် ၎င်းတို့၏ **ကိုယ်ပိုင်** context နှင့် token အသုံးပြုမှု ရှိပါသည်။ အလုပ်များလွန်ကဲသော သို့မဟုတ် ထပ်ခါတလဲလဲ လုပ်ဆောင်ရသော tasks များအတွက်၊ sub-agents များတွင် ကုန်ကျစရိတ်သက်သာသော model ကို သတ်မှတ်ပြီး သင်၏ main agent ကို အရည်အသွေးမြင့် model ဖြင့် ထားပါ။
ဤကို `agents.defaults.subagents.model` သို့မဟုတ် agent တစ်ခုချင်းစီအလိုက် override ပြုလုပ်၍ configure လုပ်နိုင်ပါသည်။

## Tool

`sessions_spawn` ကို အသုံးပြုပါ -

- sub-agent run တစ်ခုကို စတင်ပါသည် (`deliver: false`, global lane: `subagent`)
- ထို့နောက် announce step ကို လုပ်ဆောင်ပြီး announce reply ကို requester chat channel သို့ ပို့စ်တင်ပါသည်။
- မူရင်း model - `agents.defaults.subagents.model` (သို့မဟုတ် agent တစ်ခုချင်းစီအလိုက် `agents.list[].subagents.model`) ကို သတ်မှတ်ထားခြင်းမရှိပါက caller မှ အမွေဆက်ခံမည်ဖြစ်သည်။ သို့သော် `sessions_spawn.model` ကို တိတိကျကျ သတ်မှတ်ထားပါက ၎င်းကို ဦးစားပေးမည်ဖြစ်သည်။
- မူရင်း thinking - `agents.defaults.subagents.thinking` (သို့မဟုတ် agent တစ်ခုချင်းစီအလိုက် `agents.list[].subagents.thinking`) ကို သတ်မှတ်ထားခြင်းမရှိပါက caller မှ အမွေဆက်ခံမည်ဖြစ်သည်။ သို့သော် `sessions_spawn.thinking` ကို တိတိကျကျ သတ်မှတ်ထားပါက ၎င်းကို ဦးစားပေးမည်ဖြစ်သည်။

Max concurrent: 8 မူလတန်ဖိုးများ—

- Auto-archive: မိနစ် ၆၀ အကြာ
- Setting a Default Model
- token ကုန်ကျစရိတ်ကို လျှော့ချရန် sub-agent များအတွက် စျေးသက်သာသော model ကို အသုံးပြုပါသည်:
- {
  agents: {
  defaults: {
  subagents: {
  model: "minimax/MiniMax-M2.1",
  },
  },
  },
  }
- `thinking?` (ရွေးချယ်နိုင်သည် - sub-agent run အတွက် thinking level ကို override လုပ်နိုင်သည်)
- `runTimeoutSeconds?` (မူရင်း `0` - သတ်မှတ်ထားပါက N စက္ကန့်အပြီးတွင် sub-agent run ကို ရပ်တန့်မည်)
- `cleanup?` (`delete|keep`, မူရင်း `keep`)

Allowlist -

- `agents.list[].subagents.allowAgents` - `agentId` ဖြင့် သတ်မှတ်နိုင်သော agent id များစာရင်း (`["*"]` ဖြင့် မည်သည့် agent မဆို ခွင့်ပြုနိုင်သည်)။ မူရင်း - requester agent တစ်ခုတည်းသာ။

Discovery -

- `sessions_spawn` အတွက် လက်ရှိ ခွင့်ပြုထားသော agent id များကို ကြည့်ရှုရန် `agents_list` ကို အသုံးပြုပါ။

Auto-archive -

- Sub-agent sessions များကို `agents.defaults.subagents.archiveAfterMinutes` (မူရင်း - 60) အပြီးတွင် အလိုအလျောက် archive လုပ်ပါသည်။
- Archive ပြုလုပ်ရာတွင် `sessions.delete` ကို အသုံးပြုပြီး transcript ကို `*.deleted.<timestamp>` ဟု အမည်ပြောင်းပါသည်\` (တူညီသော folder)။
- `cleanup: "delete"` သည် announce ပြီးနောက် ချက်ချင်း archive လုပ်ပါသည် (transcript ကို အမည်ပြောင်း၍ ဆက်လက်သိမ်းဆည်းထားသည်)။
- Auto-archive သည် best-effort အခြေခံဖြစ်ပြီး gateway ပြန်လည်စတင်ပါက စောင့်ဆိုင်းနေသော timer များ ပျောက်ဆုံးနိုင်ပါသည်။
- `runTimeoutSeconds` သည် auto-archive မလုပ်ပါ - run ကိုသာ ရပ်တန့်စေပါသည်။ Session သည် auto-archive မတိုင်မီအထိ ဆက်လက်တည်ရှိနေပါမည်။
- Auto-archive သည် depth-1 နှင့် depth-2 sessions များတွင် တူညီစွာ အသုံးချပါသည်။

## Nested Sub-Agents

မူရင်းအားဖြင့် sub-agents များသည် ၎င်းတို့၏ ကိုယ်ပိုင် sub-agents များကို spawn မလုပ်နိုင်ပါ (`maxSpawnDepth: 1`)။ `maxSpawnDepth: 2` ဟု သတ်မှတ်ခြင်းဖြင့် nesting တစ်ဆင့်ကို ဖွင့်နိုင်ပြီး **orchestrator pattern** ကို ခွင့်ပြုပါသည် - main → orchestrator sub-agent → worker sub-sub-agents။

### ဖွင့်ရန် နည်းလမ်း

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // sub-agents များက child များကို spawn လုပ်နိုင်ရန် ခွင့်ပြုသည် (မူရင်း: 1)
        maxChildrenPerAgent: 5, // agent session တစ်ခုလျှင် အများဆုံး active children အရေအတွက် (မူရင်း: 5)
        maxConcurrent: 8, // global concurrency lane အများဆုံးကန့်သတ်ချက် (မူရင်း: 8)
      },
    },
  },
}
```

### Depth အဆင့်များ

| Depth | Session key ပုံစံ                            | Role                                                                      | Spawn လုပ်နိုင်ပါသလား?        |
| ----- | -------------------------------------------- | ------------------------------------------------------------------------- | ----------------------------- |
| 0     | `agent:&lt;id&gt;:main`                            | Main agent                                                                | အမြဲတမ်း                      |
| 1     | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (depth 2 ကို ခွင့်ပြုထားသောအခါ orchestrator) | `maxSpawnDepth >= 2` ဖြစ်မှသာ |
| 2     | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                            | မည်သည့်အခါမျှ မဖြစ်ပါ         |

### Archive လုပ်ခြင်းသည် transcript ကို \`\*.deleted.

\` ဟု အမည်ပြောင်းပါသည် (တူညီသော folder) — transcripts များကို ဖျက်မပစ်ဘဲ ထိန်းသိမ်းထားပါသည်။

1. Depth-2 worker သည် အလုပ်ပြီးဆုံးပါက → ၎င်း၏ မိဘ (depth-1 orchestrator) ထံ announce လုပ်သည်
2. Depth-1 orchestrator သည် announce ကို လက်ခံရရှိပြီး၊ ရလဒ်များကို ပေါင်းစည်းကာ အလုပ်ပြီးဆုံးပြီးနောက် → main ထံ announce လုပ်သည်
3. Main agent သည် announce ကို လက်ခံရရှိပြီး အသုံးပြုသူထံ ပို့ဆောင်သည်

အဆင့်တိုင်းသည် ၎င်း၏ တိုက်ရိုက် ကလေးများမှ announce များကိုသာ မြင်နိုင်သည်။

### Depth အလိုက် Tool policy

- **Depth 1 (orchestrator, `maxSpawnDepth >= 2` ဖြစ်သည့်အခါ)**: ၎င်း၏ ကလေးများကို စီမံခန့်ခွဲနိုင်ရန် `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` ကို ရရှိသည်။ အခြား session/system tools များကို မခွင့်ပြုထားပါ။
- **Depth 1 (leaf, `maxSpawnDepth == 1` ဖြစ်သည့်အခါ)**: session tools မရှိပါ (လက်ရှိ default အပြုအမူ)။
- **Depth 2 (leaf worker)**: session tools မရှိပါ — depth 2 တွင် `sessions_spawn` ကို အမြဲ မခွင့်ပြုပါ။ နောက်ထပ် ကလေးများကို မဖန်တီးနိုင်ပါ။

### Agent တစ်ခုချင်းစီအလိုက် spawn ကန့်သတ်ချက်

Agent session တစ်ခုစီ (မည်သည့် depth မဆို) သည် တစ်ချိန်တည်းတွင် အများဆုံး `maxChildrenPerAgent` (default: 5) active children များသာ ရှိနိုင်သည်။ ၎င်းသည် orchestrator တစ်ခုမှ မထိန်းချုပ်နိုင်သော fan-out ဖြစ်ပေါ်မှုကို တားဆီးပေးသည်။

### Cascade ရပ်တန့်ခြင်း

Depth-1 orchestrator ကို ရပ်တန့်လိုက်ပါက ၎င်း၏ depth-2 ကလေးများအားလုံးကိုလည်း အလိုအလျောက် ရပ်တန့်စေသည်:

- Main chat တွင် `/stop` ကို အသုံးပြုပါက depth-1 agents များအားလုံးကို ရပ်တန့်စေပြီး ၎င်းတို့၏ depth-2 ကလေးများထံ cascade လုပ်သည်။
- `/subagents kill <id>` သည် သတ်မှတ်ထားသော sub-agent ကို ရပ်တန့်စေပြီး ၎င်း၏ ကလေးများထံ cascade လုပ်သည်။
- `/subagents kill all` သည် တောင်းဆိုသူအတွက် sub-agents အားလုံးကို ရပ်တန့်စေပြီး cascade လုပ်သည်။

## Authentication

Thinking level is resolved in this order:

- Explicit `thinking` parameter in the `sessions_spawn` call
- Per-agent config: `agents.list[].subagents.thinking`
- Global default: `agents.defaults.subagents.thinking`

မှတ်ချက် - merge သည် additive ဖြစ်သောကြောင့် main profiles များကို fallback အဖြစ် အမြဲ အသုံးပြုနိုင်သည်။ Agent တစ်ခုချင်းစီအတွက် အပြည့်အဝ ခွဲထုတ်ထားသော auth ကို ယခုအချိန်တွင် မပံ့ပိုးသေးပါ။

## Cross-Agent Spawning

Sub-agents များသည် announce အဆင့်မှတစ်ဆင့် ပြန်လည်အစီရင်ခံသည်:

- Announce အဆင့်သည် sub-agent session အတွင်းတွင် လည်ပတ်သည် (requester session မဟုတ်ပါ)။
- Sub-agent က `ANNOUNCE_SKIP` ဟု တိတိကျကျ ပြန်ကြားပါက မည်သည့်အရာမျှ မတင်ပို့ပါ။
- မဟုတ်ပါက announce အကြောင်းပြန်ချက်ကို follow-up `agent` call (`deliver=true`) မှတစ်ဆင့် requester chat channel သို့ တင်ပို့သည်။
- Announce reply များသည် ရရှိနိုင်ပါက thread/topic routing (Slack threads, Telegram topics, Matrix threads) ကို ထိန်းသိမ်းထားပါသည်။
- Announce မက်ဆေ့ချ်များကို တည်ငြိမ်သော template တစ်ခုအဖြစ် စံပြုထားသည်:
  - `Status:` ကို run outcome (`success`, `error`, `timeout`, သို့မဟုတ် `unknown`) မှ ဆင်းသက်လာသည်။
  - `Result:` announce အဆင့်မှ summary အကြောင်းအရာ (မရှိပါက `(not available)`)။
  - `Notes:` error အသေးစိတ်များနှင့် အခြား အသုံးဝင်သော အချက်အလက်များ။
- `Status` ကို model output မှ ခန့်မှန်းထားခြင်း မဟုတ်ပါ; runtime outcome signals များမှ ရရှိသည်။

Announce payload များတွင် (wrap လုပ်ထားသော်လည်း) အဆုံးတွင် stats လိုင်းတစ်ကြောင်း ပါဝင်သည်:

- Runtime (ဥပမာ `runtime 5m12s`)
- Token အသုံးပြုမှု (input/output/စုစုပေါင်း)
- model စျေးနှုန်းကို သတ်မှတ်ထားသောအခါ ခန့်မှန်းကုန်ကျစရိတ် (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` နှင့် transcript လမ်းကြောင်း (အဓိက agent သည် `sessions_history` မှတစ်ဆင့် history ကို ရယူနိုင်ရန် သို့မဟုတ် disk ပေါ်ရှိ ဖိုင်ကို စစ်ဆေးနိုင်ရန်)

## Tool Policy (sub-agent tools)

မူလအနေဖြင့် sub-agents များသည် **session tools နှင့် system tools များမှအပ tools အားလုံးကို** ရရှိသည်:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2` ဖြစ်သောအခါ depth-1 orchestrator sub-agents များသည် ၎င်းတို့၏ ကလေးများကို စီမံခန့်ခွဲနိုင်ရန် `sessions_spawn`, `subagents`, `sessions_list`, နှင့် `sessions_history` ကို ထပ်မံရရှိမည်ဖြစ်သည်။

config ဖြင့် override လုပ်ရန်:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

Sub-agents များသည် သီးသန့် in-process queue lane ကို အသုံးပြုသည်:

- Lane အမည်: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (မူလတန်ဖိုး `8`)

## ရပ်တန့်ခြင်း

- requester chat တွင် `/stop` ပို့ခြင်းဖြင့် requester session ကို ဖျက်သိမ်းပြီး ၎င်းမှ spawn လုပ်ထားသော active sub-agent runs များအားလုံးကို ရပ်တန့်စေမည်ဖြစ်ပြီး nested children များသို့ပါ ဆက်လက်သက်ရောက်မည်ဖြစ်သည်။
- `/subagents kill <id>` သည် သတ်မှတ်ထားသော sub-agent တစ်ခုကို ရပ်တန့်စေပြီး ၎င်း၏ children များသို့ပါ ဆက်လက်သက်ရောက်မည်ဖြစ်သည်။

## Limitations

- Sub-agent announce သည် **best-effort** အခြေခံဖြစ်သည်။ gateway ပြန်လည်စတင်ပါက pending "announce back" လုပ်ဆောင်မှုများသည် ပျောက်ဆုံးသွားမည်ဖြစ်သည်။
- Sub-agents များသည် တူညီသော gateway process resources များကို မျှဝေသုံးစွဲနေဆဲဖြစ်သောကြောင့် `maxConcurrent` ကို လုံခြုံရေးထိန်းချုပ်ကိရိယာအဖြစ် သတ်မှတ်စဉ်းစားပါ။
- `sessions_spawn` သည် အမြဲ non-blocking ဖြစ်သည် — ၎င်းသည် `{ status: "accepted", runId, childSessionKey }` ကို ချက်ချင်း ပြန်လည်ပေးပို့သည်။
- Sub-agent context တွင် `AGENTS.md` + `TOOLS.md` ကိုသာ inject လုပ်ပေးသည် (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, သို့မဟုတ် `BOOTSTRAP.md` မပါဝင်ပါ)။
- အများဆုံး nesting depth သည် 5 ဖြစ်သည် (`maxSpawnDepth` အတိုင်းအတာ: 1–5)။ အသုံးပြုမှုအများစုအတွက် Depth 2 ကို အကြံပြုပါသည်။
- `maxChildrenPerAgent` သည် session တစ်ခုလျှင် active children အရေအတွက်ကို ကန့်သတ်သည် (မူလတန်ဖိုး: 5, အတိုင်းအတာ: 1–20)။

