---
title: "အောက်ခံ အေးဂျင့်များ"
---

# Sub-agents

Sub-agent များသည် ရှိပြီးသား agent run တစ်ခုမှ နောက်ခံအနေဖြင့် ဖန်တီးထုတ်လုပ်ထားသော agent run များဖြစ်သည်။ ၎င်းတို့သည် ကိုယ်ပိုင် session (`agent:<agentId>:subagent:<uuid>`) အတွင်း လည်ပတ်ပြီး ပြီးဆုံးသည့်အခါ ရလဒ်ကို တောင်းဆိုသူ chat channel သို့ **announce** ပြုလုပ်၍ ပြန်လည်ကြေညာပါသည်။

## Slash command

လက်ရှိ **session** အတွက် sub-agent run များကို စစ်ဆေးရန် သို့မဟုတ် ထိန်းချုပ်ရန် `/subagents` ကို အသုံးပြုပါ–

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` သည် run metadata များ (status, timestamps, session id, transcript path, cleanup) ကို ပြသပါသည်။

အဓိက ရည်ရွယ်ချက်များ–

- အဓိက run ကို မတားဆီးဘဲ “research / long task / slow tool” လုပ်ငန်းများကို တပြိုင်နက် လုပ်ဆောင်နိုင်ရန်။
- မူလအတိုင်း သီးခြားထားရှိခြင်း (session ခွဲထားခြင်း + optional sandboxing)။
- tool surface ကို အလွဲသုံးစား မဖြစ်စေရန် — sub-agent များသည် မူလအတိုင်း session tools မရရှိပါ။
- orchestrator pattern များအတွက် nesting depth ကို စိတ်ကြိုက် သတ်မှတ်နိုင်ရန်။

ကုန်ကျစရိတ်မှတ်ချက်– sub-agent တစ်ခုချင်းစီတွင် ကိုယ်ပိုင် context နှင့် token အသုံးပြုမှု ရှိပါသည်။ အလုပ်ကြီးများ သို့မဟုတ် ထပ်ခါတလဲလဲ လုပ်ဆောင်ရသော task များအတွက် sub-agent များတွင် စျေးသက်သာသော model ကို သတ်မှတ်ပြီး main agent ကို အရည်အသွေးမြင့် model ပေါ်တွင် ဆက်လက်ထားရှိနိုင်ပါသည်။ ၎င်းကို `agents.defaults.subagents.model` သို့မဟုတ် agent အလိုက် override များဖြင့် ပြင်ဆင်နိုင်ပါသည်။

## Tool

`sessions_spawn` ကို အသုံးပြုပါ–

- Sub-agent run ကို စတင်ပါသည် (`deliver: false`, global lane: `subagent`)
- ထို့နောက် announce အဆင့်ကို လည်ပတ်ပြီး announce reply ကို တောင်းဆိုသူ chat channel သို့ တင်ပို့ပါသည်
- Default model– သင် `agents.defaults.subagents.model` (သို့မဟုတ် per-agent `agents.list[].subagents.model`) မသတ်မှတ်ထားပါက caller မှ ဆက်ခံပါသည်။ သို့သော် `sessions_spawn.model` ကို တိတိကျကျ သတ်မှတ်ပါက ၎င်းက ဦးစားပေးပါသည်။
- Default thinking– သင် `agents.defaults.subagents.thinking` (သို့မဟုတ် per-agent `agents.list[].subagents.thinking`) မသတ်မှတ်ထားပါက caller မှ ဆက်ခံပါသည်။ သို့သော် `sessions_spawn.thinking` ကို တိတိကျကျ သတ်မှတ်ပါက ၎င်းက ဦးစားပေးပါသည်။

Tool parameters–

- `task` (လိုအပ်သည်)
- `label?` (ရွေးချယ်နိုင်သည်)
- `agentId?` (ရွေးချယ်နိုင်သည်; ခွင့်ပြုထားပါက အခြား agent id အောက်တွင် spawn လုပ်နိုင်သည်)
- `model?` (ရွေးချယ်နိုင်သည်; sub-agent model ကို override လုပ်သည်; မမှန်ကန်သော value များကို skip လုပ်ပြီး tool result ထဲတွင် warning ဖြင့် default model ပေါ်တွင် ဆက်လက် လည်ပတ်ပါသည်)
- `thinking?` (ရွေးချယ်နိုင်သည်; sub-agent run အတွက် thinking level ကို override လုပ်သည်)
- `runTimeoutSeconds?` (default `0`; သတ်မှတ်ပါက N seconds အကြာတွင် sub-agent run ကို abort လုပ်ပါသည်)
- `cleanup?` (`delete|keep`, default `keep`)

Allowlist–

- `agents.list[].subagents.allowAgents`– `agentId` မှတဆင့် target လုပ်နိုင်သော agent id စာရင်း (`["*"]` သည် မည်သည့် agent မဆို ခွင့်ပြုသည်)။ Default– တောင်းဆိုသူ agent တစ်ခုတည်းသာ။

Discovery–

- `sessions_spawn` အတွက် လက်ရှိ ခွင့်ပြုထားသော agent id များကို ကြည့်ရန် `agents_list` ကို အသုံးပြုပါ။

Auto-archive–

- Sub-agent session များကို `agents.defaults.subagents.archiveAfterMinutes` (default: 60) အကြာတွင် အလိုအလျောက် archive လုပ်ပါသည်။
- Archive လုပ်ရာတွင် `sessions.delete` ကို အသုံးပြုပြီး transcript ကို `*.deleted.<timestamp>` ဟု အမည်ပြောင်းပါသည် (တူညီသော folder)။
- `cleanup: "delete"` သည် announce ပြီးချင်း ချက်ချင်း archive လုပ်ပါသည် (transcript ကို rename လုပ်ပြီး ဆက်လက် ထိန်းသိမ်းထားပါသည်)။
- Auto-archive သည် best-effort ဖြစ်ပြီး gateway ပြန်လည်စတင်ပါက pending timer များ ပျောက်ဆုံးနိုင်ပါသည်။
- `runTimeoutSeconds` သည် auto-archive မလုပ်ပါ; run ကိုသာ ရပ်တန့်ပါသည်။ Session သည် auto-archive အထိ ဆက်လက် ရှိနေပါသည်။
- Auto-archive သည် depth-1 နှင့် depth-2 session များအတွက် တူညီစွာ အသုံးပြုပါသည်။

## Nested Sub-Agents

မူလအတိုင်း sub-agent များသည် ၎င်းတို့၏ sub-agent များကို မဖန်တီးနိုင်ပါ (`maxSpawnDepth: 1`)။ `maxSpawnDepth: 2` ဟု သတ်မှတ်ပါက nesting အဆင့် တစ်ဆင့် ခွင့်ပြုနိုင်ပြီး **orchestrator pattern** (main → orchestrator sub-agent → worker sub-sub-agents) ကို အသုံးပြုနိုင်ပါသည်။

### How to enable

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // allow sub-agents to spawn children (default: 1)
        maxChildrenPerAgent: 5, // max active children per agent session (default: 5)
        maxConcurrent: 8, // global concurrency lane cap (default: 8)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Main agent                                    | Always                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orchestrator when depth 2 allowed) | Only if `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                   | Never                        |

### Announce chain

ရလဒ်များသည် အဆင့်လိုက် ပြန်လည်စီးဆင်းပါသည်–

1. Depth-2 worker ပြီးဆုံးသည် → ၎င်း၏ မိဘ (depth-1 orchestrator) ထံ announce ပြုလုပ်သည်
2. Depth-1 orchestrator သည် announce ကို လက်ခံပြီး ရလဒ်များကို စုစည်းကာ ပြီးဆုံးသည် → main ထံ announce ပြုလုပ်သည်
3. Main agent သည် announce ကို လက်ခံပြီး အသုံးပြုသူထံ ပို့ဆောင်ပါသည်

အဆင့်တစ်ခုချင်းစီသည် ၎င်း၏ တိုက်ရိုက် children များမှ announce များကိုသာ မြင်တွေ့နိုင်ပါသည်။

### Tool policy by depth

- **Depth 1 (orchestrator, when `maxSpawnDepth >= 2`)**: ၎င်း၏ children များကို စီမံခန့်ခွဲနိုင်ရန် `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` ကို ရရှိပါသည်။ အခြား session/system tools များကို မရရှိပါ။
- **Depth 1 (leaf, when `maxSpawnDepth == 1`)**: Session tools မရှိပါ (လက်ရှိ default behavior)။
- **Depth 2 (leaf worker)**: Session tools မရှိပါ — depth 2 တွင် `sessions_spawn` ကို အမြဲ ငြင်းပယ်ပါသည်။ ထပ်မံ children မဖန်တီးနိုင်ပါ။

### Per-agent spawn limit

Agent session တစ်ခုချင်းစီ (မည်သည့် depth မဆို) သည် တစ်ချိန်တည်းတွင် အများဆုံး `maxChildrenPerAgent` (default: 5) active children ရှိနိုင်ပါသည်။ ၎င်းသည် orchestrator တစ်ခုမှ runaway fan-out ဖြစ်ပေါ်ခြင်းကို ကာကွယ်ပါသည်။

### Cascade stop

Depth-1 orchestrator ကို ရပ်တန့်ပါက ၎င်း၏ depth-2 children များအားလုံးကိုလည်း အလိုအလျောက် ရပ်တန့်ပါသည်–

- Main chat ထဲတွင် `/stop` ပို့ပါက depth-1 agents များအားလုံးကို ရပ်တန့်ပြီး ၎င်းတို့၏ depth-2 children များသို့ cascade လုပ်ပါသည်။
- `/subagents kill <id>` သည် သတ်မှတ်ထားသော sub-agent တစ်ခုကို ရပ်တန့်ပြီး ၎င်း၏ children များသို့ cascade လုပ်ပါသည်။
- `/subagents kill all` သည် တောင်းဆိုသူအတွက် sub-agents များအားလုံးကို ရပ်တန့်ပြီး cascade လုပ်ပါသည်။

## Authentication

Sub-agent authentication ကို session type မဟုတ်ဘဲ **agent id** အပေါ် အခြေခံ၍ ဖြေရှင်းပါသည်–

- Sub-agent session key သည် `agent:<agentId>:subagent:<uuid>` ဖြစ်ပါသည်။
- Auth store ကို ထို agent ၏ `agentDir` မှ load လုပ်ပါသည်။
- Main agent ၏ auth profile များကို **fallback** အဖြစ် ပေါင်းထည့်ပါသည်; conflict ဖြစ်ပါက agent profile များက ဦးစားပေးပါသည်။

မှတ်ချက်– merge သည် additive ဖြစ်သောကြောင့် main profile များကို fallback အဖြစ် အမြဲ ရရှိနိုင်ပါသည်။ Agent အလိုက် အပြည့်အဝ သီးခြား auth ကို လက်ရှိတွင် မထောက်ပံ့သေးပါ။

## Announce

Sub-agent များသည် announce အဆင့်မှတဆင့် ရလဒ်များကို ပြန်လည်ကြေညာပါသည်–

- Announce အဆင့်သည် sub-agent session အတွင်း (requester session မဟုတ်ဘဲ) လည်ပတ်ပါသည်။
- Sub-agent က တိတိကျကျ `ANNOUNCE_SKIP` ဟု ပြန်ပါက မည်သည့်အရာမျှ မတင်ပို့ပါ။
- မဟုတ်ပါက announce reply ကို follow-up `agent` call (`deliver=true`) ဖြင့် requester chat channel သို့ တင်ပို့ပါသည်။
- Announce reply များသည် ရရှိနိုင်ပါက thread/topic routing (Slack threads, Telegram topics, Matrix threads) ကို ထိန်းသိမ်းပါသည်။
- Announce message များကို တည်ငြိမ်သော template ဖြင့် စံပြုလုပ်ထားပါသည်–
  - `Status:` သည် run outcome (`success`, `error`, `timeout`, သို့မဟုတ် `unknown`) မှ ဆင်းသက်လာပါသည်။
  - `Result:` announce အဆင့်မှ summary content (မရှိပါက `(not available)`)။
  - `Notes:` error အသေးစိတ်များနှင့် အခြား အသုံးဝင်သော အချက်အလက်များ။
- `Status` ကို model output မှ ခန့်မှန်းခြင်းမဟုတ်ဘဲ runtime outcome signals မှ ရယူပါသည်။

Announce payload များတွင် အဆုံးတွင် stats line ပါဝင်ပါသည် (wrap လုပ်ထားသော်လည်း ပါဝင်ပါသည်)–

- Runtime (ဥပမာ `runtime 5m12s`)
- Token အသုံးပြုမှု (input/output/total)
- Model pricing ကို `models.providers.*.models[].cost` ဖြင့် configure လုပ်ထားပါက Estimated cost
- `sessionKey`, `sessionId`, နှင့် transcript path (main agent သည် `sessions_history` ဖြင့် history ကို ရယူရန် သို့မဟုတ် disk ပေါ်ရှိ file ကို စစ်ဆေးနိုင်ရန်)

## Tool Policy (sub-agent tools)

မူလအတိုင်း sub-agent များသည် **session tools နှင့် system tools များမှ လွဲ၍ tools အားလုံး** ကို ရရှိပါသည်–

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2` ဖြစ်ပါက depth-1 orchestrator sub-agent များသည် ၎င်းတို့၏ children များကို စီမံခန့်ခွဲနိုင်ရန် `sessions_spawn`, `subagents`, `sessions_list`, နှင့် `sessions_history` ကို ထပ်မံ ရရှိပါသည်။

Config ဖြင့် override လုပ်နိုင်ပါသည်–

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

Sub-agent များသည် သီးခြား in-process queue lane ကို အသုံးပြုပါသည်–

- Lane name: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (default `8`)

## Stopping

- Requester chat ထဲတွင် `/stop` ပို့ပါက requester session ကို abort လုပ်ပြီး ၎င်းမှ spawn လုပ်ထားသော active sub-agent run များအားလုံးကို (nested children များအထိ) ရပ်တန့်ပါသည်။
- `/subagents kill <id>` သည် သတ်မှတ်ထားသော sub-agent တစ်ခုကို ရပ်တန့်ပြီး ၎င်း၏ children များသို့ cascade လုပ်ပါသည်။

## Limitations

- Sub-agent announce သည် **best-effort** ဖြစ်ပါသည်။ Gateway ပြန်လည်စတင်ပါက pending “announce back” လုပ်ငန်းများ ပျောက်ဆုံးနိုင်ပါသည်။
- Sub-agent များသည် တူညီသော gateway process resource များကို မျှဝေအသုံးပြုကြပါသည်; `maxConcurrent` ကို လုံခြုံရေး valve အဖြစ် စဉ်းစားပါ။
- `sessions_spawn` သည် အမြဲ non-blocking ဖြစ်ပြီး `{ status: "accepted", runId, childSessionKey }` ကို ချက်ချင်း ပြန်ပေးပါသည်။
- Sub-agent context သည် `AGENTS.md` + `TOOLS.md` ကိုသာ inject လုပ်ပါသည် (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, သို့မဟုတ် `BOOTSTRAP.md` မပါဝင်ပါ)။
- Maximum nesting depth သည် 5 (`maxSpawnDepth` range: 1–5) ဖြစ်ပါသည်။ အသုံးများသော use case များအတွက် depth 2 ကို အကြံပြုပါသည်။
- `maxChildrenPerAgent` သည် session တစ်ခုလျှင် active children အရေအတွက်ကို ကန့်သတ်ပါသည် (default: 5, range: 1–20)။
