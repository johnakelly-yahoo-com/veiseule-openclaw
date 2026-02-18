---
title: "အောက်ခံ အေးဂျင့်များ"
---

# Sub-Agents

40. Sub-agent များသည် အဓိက စကားဝိုင်းကို မတားဆီးဘဲ နောက်ခံလုပ်ငန်းများကို လုပ်ဆောင်နိုင်စေသည်။ 41. Sub-agent တစ်ခုကို စတင်ဖန်တီးသောအခါ၊ ၎င်းသည် ကိုယ်ပိုင် သီးခြား session အတွင်း လည်ပတ်ပြီး အလုပ်ကို ပြီးမြောက်သည့်အခါ ရလဒ်ကို chat သို့ ပြန်ကြားပေးသည်။

42. **အသုံးပြုနိုင်သော နမူနာများ:**

- 43. အဓိက agent က မေးခွန်းများကို ဆက်လက် ဖြေဆိုနေစဉ် ခေါင်းစဉ်တစ်ခုကို သုတေသန ပြုလုပ်ခြင်း
- 44. ကြာရှည်သော လုပ်ငန်းများကို တပြိုင်နက် လုပ်ဆောင်ခြင်း (web scraping, code analysis, file processing)
- 45. Multi-agent ဖွဲ့စည်းပုံတွင် အထူးပြု agent များသို့ လုပ်ငန်းများကို ခွဲဝေပေးခြင်း

## အမြန်စတင်ရန်

46. Sub-agent များကို အသုံးပြုရန် အလွယ်ဆုံးနည်းလမ်းမှာ agent ကို သဘာဝကျကျ တောင်းဆိုခြင်း ဖြစ်သည်:

> 47. "နောက်ဆုံး Node.js release notes ကို သုတေသနလုပ်ရန် sub-agent တစ်ခု စတင်ဖန်တီးပါ"

48. Agent သည် နောက်ကွယ်တွင် `sessions_spawn` tool ကို ခေါ်ယူမည်ဖြစ်သည်။ 49. Sub-agent က အလုပ်ပြီးဆုံးသည့်အခါ ၎င်း၏ တွေ့ရှိချက်များကို သင့် chat ထဲသို့ ကြေညာပေးမည်ဖြစ်သည်။

50. ရွေးချယ်စရာများကိုလည်း တိတိကျကျ သတ်မှတ်နိုင်ပါသည်:

> ယနေ့အတွက် ဆာဗာ လော့ဂ်များကို ခွဲခြမ်းစိတ်ဖြာရန် sub-agent တစ်ခုကို စတင်ပါ။
> 2. gpt-5.2 ကို အသုံးပြုပြီး ၅ မိနစ် အချိန်ကန့်သတ် သတ်မှတ်ပါ။ အဓိက agent သည် task ဖော်ပြချက်နှင့်အတူ `sessions_spawn` ကို ခေါ်ပါသည်။

## အလုပ်လုပ်ပုံ

<Steps>
  <Step title="Main agent spawns">
    ခေါ်ဆိုမှုသည် **non-blocking** ဖြစ်သည် — အဓိက agent သည် `{ status: "accepted", runId, childSessionKey }` ကို ချက်ချင်း ပြန်လည်ရရှိပါသည်။ သီးခြားခွဲထားသော session အသစ်တစ်ခုကို (`agent:
:subagent:
`) သတ်မှတ်ထားသော `subagent` queue lane ပေါ်တွင် ဖန်တီးပါသည်။
  </Step>
  <Step title="Sub-agent runs in the background">sub-agent သည် အလုပ်ပြီးဆုံးသောအခါ၊ ၎င်း၏ တွေ့ရှိချက်များကို တောင်းဆိုသူ chat သို့ ပြန်လည်ကြေညာပါသည်။<agentId>အဓိက agent သည် သဘာဝဘာသာစကားဖြင့် အကျဉ်းချုပ်တစ်ခုကို တင်ပါသည်။<uuid>sub-agent session ကို မိနစ် ၆၀ အကြာတွင် အလိုအလျောက် archive လုပ်ပါသည် (ပြင်ဆင်နိုင်သည်)။</Step>
  <Step title="Result is announced">
    When the sub-agent finishes, it announces its findings back to the requester chat. sub-agent တစ်ခုချင်းစီတွင် ၎င်း၏ **ကိုယ်ပိုင်** context နှင့် token အသုံးပြုမှု ရှိပါသည်။
  </Step>
  <Step title="Session is archived">
    ကုန်ကျစရိတ် လျှော့ချရန် sub-agent များအတွက် စျေးသက်သာသော model ကို သတ်မှတ်ပါ — အောက်တွင်ရှိသော [Setting a Default Model](#setting-a-default-model) ကို ကြည့်ပါ။ sub-agent များသည် ပြင်ဆင်မှု မလိုအပ်ဘဲ အလိုအလျောက် အလုပ်လုပ်ပါသည်။
  </Step>
</Steps>

<Tip>
Model: ပစ်မှတ် agent ၏ ပုံမှန် model ရွေးချယ်မှု (`subagents.model` ကို မသတ်မှတ်ထားပါက) Thinking: sub-agent အတွက် override မရှိပါ (`subagents.thinking` ကို မသတ်မှတ်ထားပါက)
</Tip>

## Configuration

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

### Setting a Default Thinking Level

{
agents: {
defaults: {
subagents: {
thinking: "low",
},
},
},
}

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### multi-agent စနစ်တစ်ခုတွင် agent တစ်ခုချင်းစီအလိုက် sub-agent default များကို သတ်မှတ်နိုင်ပါသည်:

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### တစ်ချိန်တည်းတွင် လည်ပတ်နိုင်သော sub-agent အရေအတွက်ကို ထိန်းချုပ်ပါ:

{
agents: {
defaults: {
subagents: {
maxConcurrent: 4, // default: 8
},
},
},
}

```json5
sub-agent များသည် အဓိက agent queue နှင့် သီးခြားဖြစ်သော dedicated queue lane (`subagent`) ကို အသုံးပြုပါသည်၊ ထို့ကြောင့် sub-agent လည်ပတ်မှုများသည် inbound reply များကို မတားဆီးပါ။
```

### Concurrency

Auto-Archive

```json5
sub-agent session များကို ပြင်ဆင်နိုင်သော ကာလအကြာတွင် အလိုအလျောက် archive လုပ်ပါသည်:
```

{
agents: {
defaults: {
subagents: {
archiveAfterMinutes: 120, // default: 60
},
},
},
}

### Archive လုပ်ခြင်းသည် transcript ကို `*.deleted.
` ဟု အမည်ပြောင်းပါသည် (တူညီသော folder) — transcripts များကို ဖျက်မပစ်ဘဲ ထိန်းသိမ်းထားပါသည်။

auto-archive timer များသည် best-effort ဖြစ်ပြီး gateway ကို ပြန်လည်စတင်ပါက မပြီးသေးသော timer များ ပျောက်ဆုံးနိုင်ပါသည်။

```json5
The `sessions_spawn` Tool
```

<Note>ဤသည်မှာ sub-agent များကို ဖန်တီးရန် agent က ခေါ်သုံးသော tool ဖြစ်ပါသည်။<timestamp>Parameter `task`
</Note>

## _(လိုအပ်သည်)_

sub-agent က လုပ်ဆောင်ရမည့် အလုပ်အရာ

### ပါရာမီတာများ

| —                                            | Type                     | ပုံမှန်                                                                        | Description                                                                                       |
| -------------------------------------------- | ------------------------ | ------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| ခွဲခြားသတ်မှတ်ရန် အတိုချုံး အမည်             | string                   | `agentId`                                                                      | _(ခေါ်ဆိုသူ၏ agent)_                                                           |
| `label`                                      | string                   | မတူညီသော agent id အောက်တွင် spawn လုပ်ရန် (ခွင့်ပြုထားရမည်) | _(ရွေးချယ်နိုင်သည်)_                                                           |
| ဤ sub-agent အတွက် model ကို override လုပ်ရန် | string                   | `thinking`                                                                     | _(ရွေးချယ်နိုင်သည်)_                                                           |
| `model`                                      | string                   | _(optional)_                                                | Override the model for this sub-agent                                                             |
| `thinking`                                   | string                   | _(optional)_                                                | Override thinking level (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds`                          | number                   | `0` (no limit)                                              | Abort the sub-agent after N seconds                                                               |
| `cleanup`                                    | `"delete"` \\| `"keep"` | `"keep"`                                                                       | `"delete"` archives immediately after announce                                                    |

### Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

1. Explicit `model` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.model`
3. Global default: `agents.defaults.subagents.model`
4. Target agent’s normal model resolution for that new session

Thinking level is resolved in this order:

1. Explicit `thinking` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.thinking`
3. Global default: `agents.defaults.subagents.thinking`
4. Otherwise no sub-agent-specific thinking override is applied

<Note>
Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
</Note>

### Cross-Agent Spawning

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>
Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.
</Tip>

## Managing Sub-Agents (`/subagents`)

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

| Command                                    | ဖော်ပြချက်                                                        |
| ------------------------------------------ | ----------------------------------------------------------------- |
| `/subagents list`                          | List all sub-agent runs (active and completed) |
| `/subagents stop <id\\|#\\|all>`         | Stop a running sub-agent                                          |
| `/subagents log <id\\|#> [limit] [tools]` | View sub-agent transcript                                         |
| `/subagents info <id\\|#>`                | Show detailed run metadata                                        |
| `/subagents send <id\\|#> <message>`      | Send a message to a running sub-agent                             |

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">
    ```
    /subagents list
    ```

    ````
    ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">
    ```
    /subagents info 1
    ```

    ````
    ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">
    ```
    /subagents log 1 10
    ```

    ````
    Shows the last 10 messages from the sub-agent's transcript. Add `tools` to include tool call messages:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Sends a message into the running sub-agent's session and waits up to 30 seconds for a reply.
    ```

  </Accordion>
</AccordionGroup>

## Announce (How Results Come Back)

When a sub-agent finishes, it goes through an **announce** step:

1. The sub-agent's final reply is captured
2. A summary message is sent to the main agent's session with the result, status, and stats
3. The main agent posts a natural-language summary to your chat

Announce reply များသည် ရရှိနိုင်ပါက thread/topic routing (Slack threads, Telegram topics, Matrix threads) ကို ထိန်းသိမ်းထားပါသည်။

### Announce Stats

Each announce includes a stats line with:

- Runtime duration
- Token အသုံးပြုမှု (input/output/စုစုပေါင်း)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- ၁။ စက်ရှင် ကီး၊ စက်ရှင် အိုင်ဒီ၊ နှင့် ထရန့်စခရစ် လမ်းကြောင်း

### ၂။ အခြေအနေ ကြေညာခြင်း

၃။ ကြေညာချက်တွင် လည်ပတ်မှုရလဒ်မှ ဆင်းသက်လာသော အခြေအနေကို ပါဝင်ပါသည် (မော်ဒယ်ထုတ်လွှတ်မှုမှ မဟုတ်ပါ):

- ၄။ **အောင်မြင်စွာ ပြီးစီးခြင်း** (`ok`) — လုပ်ငန်းကို ပုံမှန်အတိုင်း ပြီးစီးခဲ့သည်
- ၅။ **အမှား** — လုပ်ငန်း မအောင်မြင်ပါ (အမှား အသေးစိတ်များကို မှတ်စုများတွင် ဖော်ပြထားသည်)
- ၆။ **အချိန်ကျော်လွန်ခြင်း** — လုပ်ငန်းသည် `runTimeoutSeconds` ကို ကျော်လွန်ခဲ့သည်
- ၇။ **မသိရှိနိုင်သော** — အခြေအနေကို သတ်မှတ်၍ မရပါ

<Tip>
၈။ အသုံးပြုသူအတွက် ကြေညာရန် မလိုအပ်ပါက main-agent ၏ summarize အဆင့်သည် `NO_REPLY` ကို ပြန်ပေးနိုင်ပြီး မည်သည့်အရာမျှ မတင်ပို့ပါ။
၉။ ဤအချက်သည် agent-to-agent announce flow (`sessions_send`) တွင် အသုံးပြုသော `ANNOUNCE_SKIP` နှင့် မတူပါ။
</Tip>

## ၁၀။ ကိရိယာ မူဝါဒ

၁၁။ မူလအနေဖြင့် sub-agent များသည် အန္တရာယ်ရှိသော်လည်း သို့မဟုတ် နောက်ခံလုပ်ငန်းများအတွက် မလိုအပ်သော ငြင်းပယ်ထားသည့် ကိရိယာအစုမှ လွဲ၍ **ကိရိယာအားလုံး** ကို ရရှိပါသည်:

<AccordionGroup>
  <Accordion title="Default denied tools">၁၂။ 
    | ငြင်းပယ်ထားသော ကိရိယာ | အကြောင်းပြချက် |
    |-------------|--------|
    | `sessions_list` | စက်ရှင် စီမံခန့်ခွဲမှု — main agent က စီမံခန့်ခွဲသည် |
    | `sessions_history` | စက်ရှင် စီမံခန့်ခွဲမှု — main agent က စီမံခန့်ခွဲသည် |
    | `sessions_send` | စက်ရှင် စီမံခန့်ခွဲမှု — main agent က စီမံခန့်ခွဲသည် |
    | `sessions_spawn` | Nested fan-out မရှိပါ (sub-agent များသည် sub-agent များကို မဖန်တီးနိုင်ပါ) |
    | `gateway` | စနစ် အုပ်ချုပ်ရေး — sub-agent မှ အသုံးပြုပါက အန္တရာယ်ရှိ |
    | `agents_list` | စနစ် အုပ်ချုပ်ရေး |
    | `whatsapp_login` | အပြန်အလှန် သတ်မှတ်ခြင်း — လုပ်ငန်းတစ်ခု မဟုတ်ပါ |
    | `session_status` | အခြေအနေ/အချိန်ဇယား — main agent က ညှိနှိုင်းသည် |
    | `cron` | အခြေအနေ/အချိန်ဇယား — main agent က ညှိနှိုင်းသည် |
    | `memory_search` | လိုအပ်သော အချက်အလက်ကို spawn prompt ထဲတွင် ပို့ပေးပါ |
    | `memory_get` | လိုအပ်သော အချက်အလက်ကို spawn prompt ထဲတွင် ပို့ပေးပါ |</Accordion>
</AccordionGroup>

### ၁၃။ Sub-Agent ကိရိယာများကို စိတ်ကြိုက် ပြင်ဆင်ခြင်း

၁၄။ Sub-agent ကိရိယာများကို ထပ်မံ ကန့်သတ်နိုင်ပါသည်:

```json5
၁၅။ {
  tools: {
    subagents: {
      tools: {
        // deny သည် allow ထက် အမြဲ အနိုင်ရသည်
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

၁၆။ Sub-agent များကို **သတ်မှတ်ထားသော ကိရိယာများသာ** အသုံးပြုစေရန် ကန့်သတ်ရန်:

```json5
၁၇။ {
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // သတ်မှတ်ထားပါက deny သည် အနိုင်ရဆဲ ဖြစ်သည်
      },
    },
  },
}
```

<Note>
၁၈။ စိတ်ကြိုက် deny အချက်အလက်များကို မူလ deny စာရင်းထဲသို့ **ထပ်ပေါင်းထည့်သည်**။ ၁၉။ `allow` ကို သတ်မှတ်ထားပါက ထိုကိရိယာများသာ ရရှိနိုင်ပါသည် (မူလ deny စာရင်းသည် ထပ်ဆင့် အကျုံးဝင်ပါသည်)။
</Note>

## အတည်ပြုခြင်း

Sub-agent authentication ကို session type မဟုတ်ဘဲ **agent id** အပေါ် အခြေခံ၍ ဖြေရှင်းပါသည်–

- ၂၀။ Auth store ကို ရည်မှန်းထားသော agent ၏ `agentDir` မှ ဖတ်ယူပါသည်
- ၂၁။ Main agent ၏ auth profile များကို **fallback** အဖြစ် ပေါင်းထည့်ပါသည် (အပြိုင်ဖြစ်ပါက agent profile များက အနိုင်ရသည်)
- ၂၂။ ပေါင်းစည်းခြင်းသည် additive ဖြစ်ပြီး — main profile များကို fallback အဖြစ် အမြဲ ရရှိနိုင်ပါသည်

<Note>၂၃။ 
Sub-agent တစ်ခုချင်းစီအလိုက် အပြည့်အဝ သီးခြား auth ကို လက်ရှိတွင် မထောက်ပံ့သေးပါ။</Note>

## ၂၄။ Context နှင့် System Prompt

၂၅။ Sub-agent များသည် main agent ထက် လျော့နည်းသော system prompt ကို လက်ခံရရှိပါသည်:

- ၂၆။ **ပါဝင်သည်:** Tooling, Workspace, Runtime အပိုင်းများနှင့် `AGENTS.md` နှင့် `TOOLS.md`
- ၂၇။ **မပါဝင်ပါ:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

၂၈။ Sub-agent သည် သတ်မှတ်ထားသော လုပ်ငန်းကိုသာ အာရုံစိုက်ရန်၊ ပြီးစီးအောင်လုပ်ရန်၊ main agent အဖြစ် မလုပ်ဆောင်ရန် ညွှန်ကြားသည့် လုပ်ငန်းအာရုံစိုက် system prompt တစ်ခုကိုလည်း လက်ခံရရှိပါသည်။

## ၂၉။ Sub-Agent များကို ရပ်တန့်ခြင်း

| ၃၀။ နည်းလမ်း               | ၃၁။ အကျိုးသက်ရောက်မှု                                                                              |
| -------------------------- | -------------------------------------------------------------------------------------------------- |
| ၃၂။ ချတ်အတွင်း `/stop`     | ၃၃။ Main session ကို **နှင့်** ၎င်းမှ ဖန်တီးထားသော လည်ပတ်နေသော sub-agent အားလုံးကို ဖျက်သိမ်းပါသည် |
| ၃၄။ `/subagents stop <id>` | ၃၅။ Main session ကို မထိခိုက်ဘဲ သတ်မှတ်ထားသော sub-agent တစ်ခုကို ရပ်တန့်ပါသည်                      |
| ၃၆။ `runTimeoutSeconds`    | ၃၇။ သတ်မှတ်ထားသော အချိန်ပြီးဆုံးသည့်အခါ sub-agent လည်ပတ်မှုကို အလိုအလျောက် ဖျက်သိမ်းပါသည်          |

<Note>
၃၈။ `runTimeoutSeconds` သည် စက်ရှင်ကို အလိုအလျောက် archive မလုပ်ပါ။ ၃၉။ ပုံမှန် archive timer လည်ပတ်သည့်အချိန်အထိ စက်ရှင်သည် ဆက်လက် ရှိနေပါသည်။
</Note>

## ၄၀။ အပြည့်အစုံ ဖွဲ့စည်းမှု ဥပမာ

<Accordion title="Complete sub-agent configuration">၄၁။ 
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops သည် "main" အောက်တွင် sub-agent များကို ဖန်တီးနိုင်သည်
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agent များသည် browser ကို အသုံးမပြုနိုင်ပါ
      },
    },
  },
}
```</Accordion>

## ကန့်သတ်ချက်များ

<Warning>
၄၂။ - **အကောင်းဆုံး ကြိုးစားမှုဖြင့် ကြေညာခြင်း:** gateway ပြန်လည်စတင်ပါက မပြီးဆုံးသေးသော ကြေညာမှု လုပ်ငန်းများ ပျောက်ဆုံးသွားနိုင်ပါသည်။
၄၃။ - **Nested spawning မရှိပါ:** Sub-agent များသည် ၎င်းတို့၏ sub-agent များကို မဖန်တီးနိုင်ပါ။
၄၄။ - **မျှဝေထားသော အရင်းအမြစ်များ:** Sub-agent များသည် gateway process ကို မျှဝေအသုံးပြုကြပါသည်; `maxConcurrent` ကို လုံခြုံရေး ထိန်းကွပ်ချက်အဖြစ် အသုံးပြုပါ။
၄၅။ - **Auto-archive သည် best-effort ဖြစ်သည်:** gateway ပြန်လည်စတင်ပါက မပြီးဆုံးသေးသော archive timer များ ပျောက်ဆုံးသွားနိုင်ပါသည်။
</Warning>

## ဆက်စပ်အကြောင်းအရာများ

- ၄၆။ [Session Tools](/concepts/session-tool) — `sessions_spawn` နှင့် အခြား session tool များ၏ အသေးစိတ်
- ၄၇။ [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — agent အလိုက် ကိရိယာ ကန့်သတ်ခြင်းနှင့် sandboxing
- ၄၈။ [Configuration](/gateway/configuration) — `agents.defaults.subagents` ကိုးကားချက်
- ၄၉။ [Queue](/concepts/queue) — `subagent` lane မည်သို့ လုပ်ဆောင်သည်
