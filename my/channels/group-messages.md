---
summary: "WhatsApp အုပ်စု မက်ဆေ့ချ် ကိုင်တွယ်ပုံဆိုင်ရာ အပြုအမူနှင့် ဖွဲ့စည်းပြင်ဆင်မှု (mentionPatterns ကို surface များအကြား မျှဝေထားသည်)"
read_when:
  - အုပ်စု မက်ဆေ့ချ် စည်းမျဉ်းများ သို့မဟုတ် mention များကို ပြောင်းလဲသောအခါ
title: "အုပ်စု မက်ဆေ့ချ်များ"
---

# အုပ်စု မက်ဆေ့ချ်များ (WhatsApp web channel)

ရည်ရွယ်ချက်: Clawd ကို WhatsApp အုပ်စုများထဲတွင် ပါဝင်စေပြီး ping လုပ်ခံရသည့်အချိန်မှသာ နိုးထစေခြင်း၊ ထို့အပြင် ထို thread ကို ကိုယ်ရေးကိုယ်တာ DM ဆက်ရှင်နှင့် ခွဲထားခြင်း။

မှတ်ချက် - `agents.list[].groupChat.mentionPatterns` ကို ယခုအခါ Telegram/Discord/Slack/iMessage တွင်လည်း အသုံးပြုနေပြီး၊ ဤစာတမ်းသည် WhatsApp အတွက်သာ သက်ဆိုင်သည့် လုပ်ဆောင်ပုံကို အဓိကထား ဖော်ပြထားပါသည်။ Multi-agent setup များအတွက် `agents.list[].groupChat.mentionPatterns` ကို agent တစ်ခုချင်းစီအလိုက် သတ်မှတ်ပါ (သို့မဟုတ် `messages.groupChat.mentionPatterns` ကို global fallback အဖြစ် အသုံးပြုနိုင်သည်)။

## အကောင်အထည်ဖော်ပြီးသား အရာများ (2025-12-03)

- Activation modes - `mention` (မူလသတ်မှတ်ချက်) သို့မဟုတ် `always`။ `mention` သည် ping လိုအပ်သည် (WhatsApp ၏ တကယ့် @-mentions များကို `mentionedJids` ဖြင့်၊ regex patterns များဖြင့်၊ သို့မဟုတ် bot ၏ E.164 ကို စာသားအတွင်း မည်သည့်နေရာတွင်မဆို ပါဝင်စေခြင်းဖြင့်)။ `always` သည် message တိုင်းတွင် agent ကို လှုပ်ရှားစေသော်လည်း အဓိပ္ပါယ်ရှိသော တန်ဖိုးထည့်ပေးနိုင်သော အချိန်တွင်သာ ပြန်ကြားသင့်ပြီး၊ မဟုတ်ပါက silent token `NO_REPLY` ကို ပြန်ပေးရမည်။ မူလသတ်မှတ်ချက်များကို config (`channels.whatsapp.groups`) တွင် သတ်မှတ်နိုင်ပြီး group တစ်ခုချင်းစီအလိုက် `/activation` ဖြင့် override လုပ်နိုင်သည်။ `channels.whatsapp.groups` ကို သတ်မှတ်ထားပါက ၎င်းသည် group allowlist အဖြစ်လည်း လုပ်ဆောင်မည် (`"*"` ထည့်သွင်းပါက အားလုံးကို ခွင့်ပြုမည်)။
- Group policy - `channels.whatsapp.groupPolicy` သည် group message များကို လက်ခံမလား (`open|disabled|allowlist`) ကို ထိန်းချုပ်သည်။ `allowlist` သည် `channels.whatsapp.groupAllowFrom` ကို အသုံးပြုသည် (fallback - `channels.whatsapp.allowFrom` ကို တိတိကျကျ သတ်မှတ်ထားပါက အသုံးပြုမည်)။ မူလသတ်မှတ်ချက်မှာ `allowlist` ဖြစ်ပြီး (sender များကို မထည့်မချင်း ပိတ်ထားမည်)။
- Per-group sessions - session key များသည် `agent:<agentId>:whatsapp:group:<jid>` ပုံစံဖြစ်ပြီး ထို့ကြောင့် `/verbose on` သို့မဟုတ် `/think high` ကဲ့သို့သော command များကို (သီးခြား message အဖြစ် ပို့သည့်အခါ) ထို group အတွက်သာ သက်ဆိုင်စေမည်ဖြစ်သည်။ ကိုယ်ရေးကိုယ်တာ DM state ကို မထိခိုက်စေပါ။ Group thread များအတွက် heartbeat များကို မလုပ်ဆောင်ပါ။
- Context injection - **pending-only** group message များ (မူလ 50 ခု) ထဲမှ run မဖြစ်ပေါ်စေခဲ့သော message များကို `[Chat messages since your last reply - for context]` အောက်တွင် prefix အဖြစ် ထည့်သွင်းပြီး၊ run ကို ဖြစ်စေသည့် message ကို `[Current message - respond to this]` အောက်တွင် ထည့်သွင်းပါသည်။ Session ထဲတွင် ရှိပြီးသား message များကို ထပ်မံ ထည့်သွင်းမည်မဟုတ်ပါ။
- Sender surfacing: အုပ်စု batch တစ်ခုချင်းစီ၏ အဆုံးတွင် ယခု `[from: Sender Name (+E164)]` ကို ထည့်သွင်းထားသောကြောင့် Pi သည် ပြောဆိုနေသူကို သိနိုင်ပါသည်။
- Ephemeral/view-once: စာသား/mentions များကို ထုတ်ယူမီ အဲဒီများကို unwrap လုပ်ပါသည်၊ ထို့ကြောင့် အတွင်းရှိ ping များသည် ဆက်လက် trigger ဖြစ်ပါသည်။
- Group system prompt - group session ၏ ပထမဆုံး turn တွင် (နှင့် `/activation` ဖြင့် mode ပြောင်းလဲသည့်အခါတိုင်း) system prompt ထဲသို့ အောက်ပါကဲ့သို့သော စာပိုဒ်တိုကို ထည့်သွင်းပါသည် - `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Metadata မရရှိနိုင်ပါကလည်း ၎င်းသည် group chat ဖြစ်ကြောင်းကို agent အား အသိပေးမည်ဖြစ်သည်။

## Config ဥပမာ (WhatsApp)

WhatsApp သည် text body အတွင်း visual `@` ကို ဖယ်ရှားသည့်အခါတွင်ပါ display-name ping များ အလုပ်လုပ်စေရန် `~/.openclaw/openclaw.json` ထဲသို့ `groupChat` block တစ်ခုကို ထည့်ပါ—

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

မှတ်ချက်များ -

- regex များသည် case-insensitive ဖြစ်ပြီး `@openclaw` ကဲ့သို့သော display-name ping နှင့် `+`/spaces ပါဝင်ခြင်း သို့မဟုတ် မပါဝင်ခြင်းနှင့်အတူ raw number ကိုလည်း ဖုံးလွှမ်းပါသည်။
- WhatsApp သည် လူတစ်ဦးက contact ကို တို့လိုက်သည့်အခါ `mentionedJids` ဖြင့် canonical mentions များကို ဆက်လက်ပို့ပေးနေသဖြင့် number fallback သည် မကြာခဏ မလိုအပ်သော်လည်း အထောက်အကူဖြစ်သော safety net တစ်ခု ဖြစ်ပါသည်။

### Activation command (owner-only)

အုပ်စု chat command ကို အသုံးပြုပါ—

- `/activation mention`
- `/activation always`

Owner နံပါတ် (`channels.whatsapp.allowFrom` မှ သို့မဟုတ် မသတ်မှတ်ထားပါက bot ၏ ကိုယ်ပိုင် E.164) မှသာ ဤ setting ကို ပြောင်းလဲနိုင်သည်။ လက်ရှိ activation mode ကို ကြည့်ရှုရန် group အတွင်း `/status` ကို သီးခြား message အဖြစ် ပို့ပါ။

## အသုံးပြုနည်း

1. OpenClaw ကို လည်ပတ်နေသော သင့် WhatsApp အကောင့်ကို အုပ်စုထဲသို့ ထည့်ပါ။
2. Say `@openclaw …` (or include the number). Only allowlisted senders can trigger it unless you set `groupPolicy: "open"`.
3. agent prompt တွင် မကြာသေးမီ အုပ်စု context နှင့် အဆုံးတွင် `[from: …]` marker ကို ထည့်သွင်းပေးထားသဖြင့် မှန်ကန်သော လူကို ကိုင်တွယ်ဖြေကြားနိုင်ပါသည်။
4. Session-level directives (`/verbose on`, `/think high`, `/new` or `/reset`, `/compact`) apply only to that group’s session; send them as standalone messages so they register. Your personal DM session remains independent.

## စမ်းသပ်ခြင်း / အတည်ပြုခြင်း

- Manual smoke:
  - အုပ်စုထဲတွင် `@openclaw` ping တစ်ခု ပို့ပြီး ပို့သူအမည်ကို ရည်ညွှန်းသော ပြန်ကြားချက် ရှိကြောင်း အတည်ပြုပါ။
  - ဒုတိယ ping တစ်ခု ပို့ပြီး history block ပါဝင်လာကြောင်းနှင့် နောက် turn တွင် ပြန်လည်ရှင်းလင်းသွားကြောင်း စစ်ဆေးပါ။
- Gateway logs ကို စစ်ဆေးပါ (`--verbose` ဖြင့် run လုပ်ပါ)၊ `from: <groupJid>` နှင့် `[from: …]` suffix ကို ပြသထားသော `inbound web message` entries များကို တွေ့ရပါမည်။

## သိထားသင့်သော အချက်များ

- အုပ်စုများအတွက် heartbeats များကို ဆူညံသည့် broadcast များ မဖြစ်စေရန် ရည်ရွယ်ချက်ရှိရှိ ကျော်လွှားထားပါသည်။
- Echo suppression သည် ပေါင်းစည်းထားသော batch string ကို အသုံးပြုပါသည်; mention မပါဘဲ တူညီသော စာသားကို နှစ်ကြိမ်ပို့ပါက ပထမတစ်ကြိမ်သာ ပြန်ကြားချက် ရရှိပါမည်။
- Session store entries များသည် session store (`~/.openclaw/agents/<agentId>/sessions/sessions.json` default) အတွင်း `agent:<agentId>:whatsapp:group:<jid>` အဖြစ် ပေါ်လာပါမည်; entry မရှိခြင်းသည် အုပ်စုမှ run ကို မ trigger လုပ်ရသေးကြောင်းသာ ဆိုလိုပါသည်။
- အုပ်စုများအတွင်း typing indicators များသည် `agents.defaults.typingMode` ကို လိုက်နာပြီး (default: mention မခံရပါက `message`) ဖြစ်ပါသည်။
