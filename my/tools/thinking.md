---
title: "ဆင်ခြင်စဉ်းစားမှု အဆင့်များ"
---

# ဆင်ခြင်စဉ်းစားမှု အဆင့်များ (/think directives)

## ဘာလုပ်ပေးသလဲ

- မည်သည့် ဝင်လာသော စာကိုယ် (body) မဆို အတွင်းတွင် ထည့်သွင်းရေးနိုင်သော ညွှန်ကြားချက်: `/t <level>`, `/think:<level>`, သို့မဟုတ် `/thinking <level>`။
- အဆင့်များ (အမည်တူများ): `off | minimal | low | medium | high | xhigh` (GPT-5.2 + Codex မော်ဒယ်များအတွက်သာ)
  - minimal → “စဉ်းစား”
  - low → “နက်နက်ရှိုင်းရှိုင်း စဉ်းစား”
  - medium → “ပိုမိုနက်နက်ရှိုင်းရှိုင်း စဉ်းစား”
  - high → “ultrathink” (အများဆုံး ဘတ်ဂျက်)
  - xhigh → “ultrathink+” (GPT-5.2 + Codex မော်ဒယ်များအတွက်သာ)
  - `x-high`, `x_high`, `extra-high`, `extra high`, နှင့် `extra_high` တို့ကို `xhigh` သို့ မാപ്പ်လုပ်ထားသည်။
  - `highest`, `max` တို့ကို `high` သို့ မാപ്പ်လုပ်ထားသည်။
- Provider မှတ်ချက်များ:
  - Z.AI (`zai/*`) သည် binary thinking (`on`/`off`) ကိုသာ ပံ့ပိုးသည်။ `off` မဟုတ်သော အဆင့်အားလုံးကို `on` အဖြစ် သတ်မှတ်ပြီး (`low` သို့ map လုပ်သည်)။

## ဖြေရှင်းသတ်မှတ်မှု အစီအစဉ်

1. မက်ဆေ့ချ်အပေါ်ရှိ Inline ညွှန်ကြားချက် (ထိုမက်ဆေ့ချ်တစ်ခုတည်းအတွက်သာ သက်ရောက်သည်)။
2. ဆက်ရှင် အစားထိုးသတ်မှတ်မှု (ညွှန်ကြားချက်တစ်ခုတည်းပါသော မက်ဆေ့ချ်ပို့ခြင်းဖြင့် သတ်မှတ်သည်)။
3. အပြည်ပြည်ဆိုင်ရာ မူလသတ်မှတ်ချက် (config ထဲရှိ `agents.defaults.thinkingDefault`)။
4. Fallback: ဆင်ခြင်စဉ်းစားနိုင်သော မော်ဒယ်များအတွက် low; မဟုတ်ပါက off။

## ဆက်ရှင် မူလသတ်မှတ်ချက် သတ်မှတ်ခြင်း

- ညွှန်ကြားချက်သာ ပါသော မက်ဆေ့ချ်တစ်စောင် ပို့ပါ (အလွတ်နေရာများ ခွင့်ပြုသည်)၊ ဥပမာ `/think:medium` သို့မဟုတ် `/t high`။
- လက်ရှိ ဆက်ရှင်အတွက် (မူလအားဖြင့် ပို့သူတစ်ဦးချင်းစီအလိုက်) ဆက်လက် သက်ရောက်နေမည်ဖြစ်ပြီး `/think:off` ဖြင့် သို့မဟုတ် ဆက်ရှင် အလုပ်မလုပ်ချိန် reset ဖြစ်ပါက ရှင်းလင်းသွားမည်။
- အတည်ပြုစာပြန်ကို ပို့ပေးသည် (`Thinking level set to high.` / `Thinking disabled.`)။ အဆင့်မမှန်ကန်ပါက (ဥပမာ `/thinking big`) hint တစ်ခုနှင့်အတူ command ကို ပယ်ချပြီး session state ကို မပြောင်းလဲဘဲ ထားရှိသည်။
- လက်ရှိ ဆင်ခြင်စဉ်းစားမှု အဆင့်ကို ကြည့်ရန် အကြောင်းပြန်မပါဘဲ `/think` (သို့မဟုတ် `/think:`) ကို ပို့ပါ။

## အေးဂျင့်အလိုက် အသုံးချခြင်း

- **Embedded Pi**: ဖြေရှင်းပြီးသား အဆင့်ကို in-process Pi agent runtime သို့ ပေးပို့သည်။

## Verbose ညွှန်ကြားချက်များ (/verbose သို့မဟုတ် /v)

- အဆင့်များ: `on` (minimal) | `full` | `off` (မူလသတ်မှတ်ချက်)။
- ညွှန်ကြားချက်သာ ပါသော မက်ဆေ့ချ်သည် ဆက်ရှင် verbose ကို ဖွင့်/ပိတ် ပြောင်းလဲပြီး `Verbose logging enabled.` / `Verbose logging disabled.` ဖြင့် ပြန်ကြားမည်ဖြစ်သည်; မမှန်ကန်သော အဆင့်များတွင် အခြေအနေ မပြောင်းဘဲ အညွှန်းသာ ပြန်ပေးမည်။
- `/verbose off` သည် ဆက်ရှင် အစားထိုးသတ်မှတ်မှုကို သိမ်းဆည်းထားသည်; Sessions UI တွင် `inherit` ကို ရွေးချယ်ခြင်းဖြင့် ရှင်းလင်းနိုင်သည်။
- Inline ညွှန်ကြားချက်သည် ထိုမက်ဆေ့ချ်တစ်ခုတည်းအတွက်သာ သက်ရောက်သည်; အခြားအခါများတွင် ဆက်ရှင်/အပြည်ပြည်ဆိုင်ရာ မူလသတ်မှတ်ချက်များ သက်ရောက်သည်။
- လက်ရှိ verbose အဆင့်ကို ကြည့်ရန် အကြောင်းပြန်မပါဘဲ `/verbose` (သို့မဟုတ် `/verbose:`) ကို ပို့ပါ။
- verbose ကို ဖွင့်ထားသောအခါ structured tool results များကို ထုတ်ပေးသော agents (Pi, အခြား JSON agents) သည် tool call တစ်ခုချင်းစီကို metadata-only message အဖြစ် သီးခြားပို့ပေးသည်။ ရရှိနိုင်ပါက `<emoji> <tool-name>: <arg>` (path/command) ပုံစံဖြင့် prefix ထည့်ပေးသည်။ ဤ tool summary များကို tool စတင်ချိန်တွင် ချက်ချင်း (separate bubbles အဖြစ်) ပို့ပေးပြီး streaming deltas အဖြစ် မပို့ပါ။
- verbose ကို `full` ဟု သတ်မှတ်ထားပါက tool outputs များကို ပြီးဆုံးပြီးနောက်တွင်လည်း forward ပို့ပေးသည် (separate bubble အဖြစ်၊ လုံခြုံသော အရှည်အတိုင်းအတာအထိ ဖြတ်တောက်ထားသည်)။ Run တစ်ခု လုပ်ဆောင်နေစဉ်အတွင်း `/verbose on|full|off` ကို ပြောင်းလဲပါက နောက်ဆက်တွဲ tool bubbles များသည် setting အသစ်ကို လိုက်နာမည်ဖြစ်သည်။

## ဆင်ခြင်စဉ်းစားမှု မြင်နိုင်မှု (/reasoning)

- အဆင့်များ: `on|off|stream`။
- ညွှန်ကြားချက်သာ ပါသော မက်ဆေ့ချ်သည် ပြန်ကြားချက်များတွင် thinking blocks ကို ပြသ/မပြသ ပြောင်းလဲပေးသည်။
- ဖွင့်ထားပါက reasoning ကို `Reasoning:` ဖြင့် အစပြုထားသော **သီးသန့် မက်ဆေ့ချ်** အဖြစ် ပို့ပေးမည်။
- `stream` (Telegram သာ): ပြန်ကြားချက်ကို ထုတ်လုပ်နေစဉ် Telegram draft bubble ထဲသို့ reasoning ကို stream လုပ်ပြီး နောက်ဆုံး အဖြေကို reasoning မပါဘဲ ပို့မည်။
- အမည်တူ: `/reason`။
- လက်ရှိ reasoning အဆင့်ကို ကြည့်ရန် အကြောင်းပြန်မပါဘဲ `/reasoning` (သို့မဟုတ် `/reasoning:`) ကို ပို့ပါ။

## ဆက်စပ်အကြောင်းအရာများ

- Elevated mode စာရွက်စာတမ်းများကို [Elevated mode](/tools/elevated) တွင် ကြည့်နိုင်သည်။

## Heartbeat များ

- Heartbeat probe body သည် သတ်မှတ်ထားသော heartbeat prompt ဖြစ်သည် (default: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`)။ Heartbeat message အတွင်းရှိ inline directives များကို ပုံမှန်အတိုင်း အသုံးပြုနိုင်သည် (သို့သော် heartbeats မှ session defaults များကို မပြောင်းလဲပါနှင့်)။
- Heartbeat ပို့ခြင်းသည် ပုံမှန်အားဖြင့် နောက်ဆုံး payload ကိုသာ ပို့သည်။ သီးခြား `Reasoning:` message ကိုလည်း (ရရှိနိုင်ပါက) ပို့လိုပါက `agents.defaults.heartbeat.includeReasoning: true` သို့မဟုတ် agent တစ်ခုချင်းစီအတွက် `agents.list[].heartbeat.includeReasoning: true` ဟု သတ်မှတ်ပါ။

## Web chat UI

- Web chat ၏ thinking selector သည် စာမျက်နှာ ဖွင့်ချိန်တွင် inbound session store/config မှ သိမ်းဆည်းထားသော ဆက်ရှင် အဆင့်ကို တိုက်ဆိုင်စေပါသည်။
- အခြားအဆင့်ကို ရွေးချယ်ပါက နောက်မက်ဆေ့ချ်တစ်စောင်အတွက်သာ သက်ရောက်မည် (`thinkingOnce`)။ ပို့ပြီးနောက် selector သည် သိမ်းဆည်းထားသော ဆက်ရှင် အဆင့်သို့ ပြန်လည် ခုန်သွားမည်။
- ဆက်ရှင် မူလသတ်မှတ်ချက်ကို ပြောင်းလဲရန် ယခင်ကလို `/think:<level>` ညွှန်ကြားချက်ကို ပို့ပါ; နောက်တစ်ကြိမ် reload ပြုလုပ်ပြီးနောက် selector တွင် ထင်ဟပ်ပြမည်။
