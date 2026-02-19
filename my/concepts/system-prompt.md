---
summary: "OpenClaw စနစ်ပရောမ့်တွင် ပါဝင်သည့် အကြောင်းအရာများနှင့် ၎င်းကို မည်သို့ စုစည်းတည်ဆောက်ထားသည်"
read_when:
  - စနစ်ပရောမ့် စာသား၊ ကိရိယာစာရင်း သို့မဟုတ် အချိန်/heartbeat အပိုင်းများကို ပြင်ဆင်သည့်အခါ
  - workspace bootstrap သို့မဟုတ် Skills ထည့်သွင်းပုံအပြုအမူကို ပြောင်းလဲသည့်အခါ
title: "စနစ်ညွှန်ကြားချက်"
---

# စနစ်ညွှန်ကြားချက်

OpenClaw သည် agent run တိုင်းအတွက် custom system prompt တစ်ခု တည်ဆောက်သည်။ prompt သည် **OpenClaw မှ ပိုင်ဆိုင်သော** ဖြစ်ပြီး pi-coding-agent ၏ ပုံမှန် prompt ကို မအသုံးပြုပါ။

ထို prompt ကို OpenClaw က စုစည်းပြီး agent run တစ်ခုချင်းစီထဲသို့ ထည့်သွင်းပေးပါသည်။

## ဖွဲ့စည်းပုံ

Prompt ကို ရည်ရွယ်ချက်ရှိရှိ ကျစ်လစ်အောင် ထားရှိပြီး တိတိကျကျ သတ်မှတ်ထားသော အပိုင်းများကို အသုံးပြုပါသည်–

- **Tooling**: လက်ရှိ ကိရိယာစာရင်း + အကျဉ်းချုပ်ဖော်ပြချက်များ။
- **Safety**: အာဏာရှာဖွေသည့် အပြုအမူများ သို့မဟုတ် oversight ကို ကျော်လွှားရန် ကြိုးပမ်းမှုများကို ရှောင်ရှားရန် အတိုချုပ် သတိပေးချက်။
- **Skills** (ရရှိနိုင်ပါက): လိုအပ်သည့်အချိန်တွင် skill ညွှန်ကြားချက်များကို မည်သို့ load လုပ်ရမည်ကို မော်ဒယ်အား ပြောကြားပေးသည်။
- **OpenClaw Self-Update**: `config.apply` နှင့် `update.run` ကို မည်သို့ လည်ပတ်ရမည်။
- **Workspace**: အလုပ်လုပ်သည့် ဒိုင်ရက်ထရီ (`agents.defaults.workspace`)။
- **Documentation**: OpenClaw docs ၏ local path (repo သို့မဟုတ် npm package) နှင့် ၎င်းတို့ကို မည်သည့်အချိန်တွင် ဖတ်သင့်သည်ကို ဖော်ပြသည်။
- **Workspace Files (injected)**: bootstrap ဖိုင်များကို အောက်တွင် ထည့်သွင်းထားကြောင်း ပြသသည်။
- **Sandbox** (အသုံးပြုထားပါက): sandboxed runtime၊ sandbox paths နှင့် elevated exec ရရှိနိုင်ခြင်း ရှိ/မရှိကို ဖော်ပြသည်။
- **Current Date & Time**: အသုံးပြုသူ၏ local အချိန်၊ timezone နှင့် အချိန်ဖော်ပြပုံစံ။
- **Reply Tags**: ပံ့ပိုးထားသော provider များအတွက် optional reply tag syntax။
- **Heartbeats**: heartbeat prompt နှင့် ack အပြုအမူ။
- **Runtime**: ဟို့စ်၊ OS၊ node၊ model၊ repo root (ရှာဖွေတွေ့ရှိပါက)၊ thinking level (တစ်ကြောင်းတည်း)။
- **Reasoning**: လက်ရှိ မြင်နိုင်မှုအဆင့် + /reasoning toggle အညွှန်း။

system prompt ထဲရှိ လုံခြုံရေး guardrails များသည် အကြံပြုသဘောသာ ဖြစ်သည်။ ၎င်းတို့သည် မော်ဒယ်၏ အပြုအမူကို ညွှန်ပြပေးသော်လည်း မူဝါဒကို အတင်းအကျပ် မအကောင်အထည်ဖော်ပါ။ ခိုင်မာသော အကောင်အထည်ဖော်မှုအတွက် tool policy၊ exec approvals၊ sandboxing နှင့် channel allowlists ကို အသုံးပြုပါ၊ အော်ပရေးတာများသည် ဒီဇိုင်းအရ ယင်းတို့ကို ပိတ်နိုင်သည်။

## ပရոմ့် မုဒ်များ

OpenClaw သည် sub-agent များအတွက် ပိုမိုသေးငယ်သော system prompt များကို ပြန်လည်တင်ဆက်နိုင်သည်။ runtime သည် run တစ်ခုစီအတွက် `promptMode` ကို သတ်မှတ်သည် (အသုံးပြုသူ မျက်နှာပြင်ဆိုင်ရာ config မဟုတ်ပါ):

- `full` (default): အထက်ဖော်ပြပါ အပိုင်းများအားလုံး ပါဝင်သည်။
- `minimal`: sub-agent များအတွက် အသုံးပြုသည်; **Skills**, **Memory Recall**, **OpenClaw  Self-Update**, **Model Aliases**, **User Identity**, **Reply Tags**,
  **Messaging**, **Silent Replies**, နှင့် **Heartbeats** များကို ဖယ်ရှားထားသည်။ ကိရိယာများ၊ **လုံခြုံရေး**၊
  အလုပ်နေရာ (Workspace)၊ Sandbox၊ လက်ရှိ ရက်စွဲနှင့် အချိန် (သိရှိပါက)၊ Runtime နှင့် ထည့်သွင်းထားသော
  context များကို ဆက်လက် အသုံးပြုနိုင်ပါသည်။
- `none`: အခြေခံ identity စာကြောင်းတစ်ကြောင်းသာ ပြန်ပေးသည်။

`promptMode=minimal` ဖြစ်ပါက၊ ထပ်မံ ထည့်သွင်းသည့် prompt များကို **Group Chat Context** အစား **Subagent
Context** ဟု အမည်တပ်ပါသည်။

## လုပ်ငန်းခွင် bootstrap ထည့်သွင်းခြင်း

Bootstrap ဖိုင်များကို ဖြတ်တောက်ပြီး **Project Context** အောက်တွင် ဆက်လက် ထည့်သွင်းထားသဖြင့် မော်ဒယ်သည် identity နှင့် profile context ကို အထူးဖတ်ရန် မလိုဘဲ မြင်နိုင်ပါသည်–

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (အသစ်စက်စက် workspace များတွင်သာ)
- `MEMORY.md` နှင့်/သို့မဟုတ် `memory.md` (workspace ထဲတွင် ရှိပါက; တစ်ခု သို့မဟုတ် နှစ်ခုလုံးကို ထည့်သွင်းနိုင်သည်)

ဤဖိုင်များအားလုံးကို turn တစ်ခုချင်းစီတွင် **context window ထဲသို့ ထည့်သွင်းထားသည်** ဖြစ်ပြီး၊ ထိုကြောင့် token များကို သုံးစွဲပါသည်။ အထူးသဖြင့် `MEMORY.md` ကို တိုတောင်းစွာ ထားပါ — အချိန်ကြာလာသည်နှင့်အမျှ ကြီးထွားလာနိုင်ပြီး context အသုံးပြုမှု များပြားလာခြင်းနှင့် compaction ပိုမို မကြာခဏ ဖြစ်ပေါ်လာနိုင်ပါသည်။

> **မှတ်ချက်:** `memory/*.md` နေ့စဉ်ဖိုင်များကို အလိုအလျောက် ထည့်သွင်းမပေးပါ။ ၎င်းတို့ကို `memory_search` နှင့် `memory_get` tools များမှတစ်ဆင့် လိုအပ်သည့်အချိန်တွင်သာ ရယူအသုံးပြုနိုင်သောကြောင့် မော်ဒယ်က ၎င်းတို့ကို တိတိကျကျ ဖတ်ရှုမထားပါက context window အရွယ်အစားကို မထိခိုက်ပါ။

ဖိုင်အရွယ်အစားကြီးများကို အမှတ်အသားတစ်ခုဖြင့် ဖြတ်တောက်ထားပါသည်။ ဖိုင်တစ်ခုချင်းစီ၏ အများဆုံး အရွယ်အစားကို
`agents.defaults.bootstrapMaxChars` (ပုံမှန်: 20000) ဖြင့် ထိန်းချုပ်ပါသည်။ ဖိုင်များအနှံ့ ထည့်သွင်းထားသော bootstrap content စုစုပေါင်းကို `agents.defaults.bootstrapTotalMaxChars` (မူလတန်ဖိုး: 24000) ဖြင့် ကန့်သတ်ထားသည်။ မရှိသော ဖိုင်များအတွက် ဖိုင်မရှိကြောင်း အတိုချုံးအမှတ်အသားကို ထည့်သွင်းပေးသည်။

Sub-agent session များတွင် `AGENTS.md` နှင့် `TOOLS.md` တို့ကိုသာ ထည့်သွင်းပြီး (အခြား bootstrap ဖိုင်များကို sub-agent context ကို သေးငယ်စေရန် စစ်ထုတ်ထားသည်)။

အသုံးပြုသူ၏ time zone ကို သိရှိပါက system prompt တွင် သီးသန့် **Current Date & Time** အပိုင်း ပါဝင်ပါသည်။ prompt ကို cache-stable ဖြစ်စေရန် ယခုအခါ **time zone** ကိုသာ ထည့်သွင်းထားပြီး (dynamic clock သို့မဟုတ် time format မပါဝင်တော့ပါ)။

ထည့်သွင်းထားသော ဖိုင်တစ်ခုချင်းစီက ဘယ်လောက် ပါဝင်ပံ့ပိုးနေသည်ကို (raw နှင့် injected, truncation, tool schema overhead အပါအဝင်) စစ်ဆေးရန် `/context list` သို့မဟုတ် `/context detail` ကို အသုံးပြုပါ။ [Context](/concepts/context) ကို ကြည့်ပါ။

## Time handling

အသုံးပြုသူ၏ time zone ကို သိရှိပါက system prompt တွင် သီးသန့် **Current Date & Time** အပိုင်း ပါဝင်ပါသည်။ prompt ကို cache-stable ဖြစ်စေရန် ယခုအခါ **time zone** ကိုသာ ထည့်သွင်းထားပြီး (dynamic clock သို့မဟုတ် time format မပါဝင်တော့ပါ)။

အပြည့်အစုံ အပြုအမူအသေးစိတ်ကို [Date & Time](/date-time) တွင် ကြည့်ပါ။

အောက်ပါအတိုင်း configure လုပ်နိုင်ပါသည်–

- `agents.defaults.userTimezone`
- `agents.defaults.timeFormat` (`auto` | `12` | `24`)

အပြည့်အစုံ အပြုအမူအသေးစိတ်ကို [Date & Time](/date-time) တွင် ကြည့်ပါ။

## Skills

အရည်အချင်းပြည့်မီသော skills များ ရှိပါက OpenClaw သည် **available skills list** ကို ချုံ့ထားသောပုံစံဖြင့် (`formatSkillsForPrompt`) ထည့်သွင်းပြီး skill တစ်ခုချင်းစီအတွက် **file path** ကိုလည်း ပါဝင်စေပါသည်။ prompt သည် မော်ဒယ်အား သတ်မှတ်ထားသော တည်နေရာ (workspace, managed, သို့မဟုတ် bundled) တွင်ရှိသော SKILL.md ကို `read` ဖြင့် load လုပ်ရန် ညွှန်ကြားပါသည်။ skills များ မရှိပါက Skills အပိုင်းကို ဖယ်ရှားထားပါသည်။

```
<available_skills>
  <skill>
    <name>...</name>
    <description>...</description>
    <location>...</location>
  </skill>
</available_skills>
```

ဤနည်းလမ်းကြောင့် base prompt ကို သေးငယ်စွာ ထားနိုင်ပြီး လိုအပ်သည့် skill များကိုသာ ထိရောက်စွာ အသုံးပြုနိုင်ပါသည်။

## Documentation

ရရှိနိုင်ပါက system prompt တွင် **Documentation** အပိုင်းပါဝင်ပြီး ဒေသတွင်း OpenClaw docs directory (repo workspace ထဲရှိ `docs/` သို့မဟုတ် bundled npm package docs) ကို ညွှန်ပြသည့်အပြင် public mirror၊ source repo၊ community Discord နှင့် skills ရှာဖွေရန် ClawHub ([https://clawhub.com](https://clawhub.com)) ကိုလည်း မှတ်သားဖော်ပြပါသည်။ prompt သည် OpenClaw ၏ behavior, commands, configuration, သို့မဟုတ် architecture အတွက် local docs ကို ပထမဆုံး ကြည့်ရှုရန်နှင့် ဖြစ်နိုင်ပါက `openclaw status` ကို ကိုယ်တိုင် chạy ရန် ညွှန်ကြားပါသည် (access မရှိသောအခါသာ အသုံးပြုသူကို မေးမြန်းပါသည်)။

