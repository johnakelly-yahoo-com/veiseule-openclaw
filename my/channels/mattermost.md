---
title: "Mattermost"
---

# Mattermost (plugin)

အခြေအနေ - plugin (bot token + WebSocket events) မှတစ်ဆင့် ထောက်ပံ့ထားသည်။ Channels၊ groups နှင့် DMs များကို ထောက်ပံ့ထားသည်။
Mattermost is a self-hostable team messaging platform; see the official site at
[mattermost.com](https://mattermost.com) for product details and downloads.

## Plugin လိုအပ်သည်

Mattermost ကို plugin အဖြစ် ပေးပို့ထားပြီး core install နှင့် မပါဝင်ပါ။

CLI ဖြင့် ထည့်သွင်းတပ်ဆင်ခြင်း (npm registry):

```bash
openclaw plugins install @openclaw/mattermost
```

Local checkout (git repo မှ လည်ပတ်နေစဉ်):

```bash
openclaw plugins install ./extensions/mattermost
```

configure/onboarding အတွင်း Mattermost ကို ရွေးချယ်ပြီး git checkout ကို တွေ့ရှိပါက
OpenClaw သည် local install လမ်းကြောင်းကို အလိုအလျောက် ပေးပါမည်။

အသေးစိတ်: [Plugins](/tools/plugin)

## အမြန် စတင်သတ်မှတ်ခြင်း

1. Mattermost plugin ကို ထည့်သွင်းတပ်ဆင်ပါ။
2. Mattermost bot အကောင့်တစ်ခု ဖန်တီးပြီး **bot token** ကို ကူးယူပါ။
3. Mattermost **base URL** ကို ကူးယူပါ (ဥပမာ `https://chat.example.com`)။
4. OpenClaw ကို ဖွဲ့စည်းပြင်ဆင်ပြီး Gateway ကို စတင်ပါ။

အနည်းဆုံး ဖွဲ့စည်းပြင်ဆင်မှု:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
    },
  },
}
```

## Environment variables (မူလအကောင့်)

env vars ကို အသုံးပြုလိုပါက Gateway ဟို့စ် ပေါ်တွင် အောက်ပါအရာများကို သတ်မှတ်ပါ:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

Env vars များသည် **မူလ** အကောင့် (`default`) အတွက်သာ သက်ဆိုင်ပါသည်။ အခြားအကောင့်များသည် config တန်ဖိုးများကို အသုံးပြုရမည်ဖြစ်သည်။

## Chat မုဒ်များ

Mattermost သည် DMs များကို အလိုအလျောက် ပြန်လည်တုံ့ပြန်ပါသည်။ Channel အပြုအမူကို `chatmode` ဖြင့် ထိန်းချုပ်ပါသည်-

- `oncall` (default): ချန်နယ်များတွင် @mention လုပ်ထားသောအခါသာ တုံ့ပြန်ပါသည်။
- `onmessage`: ချန်နယ်မက်ဆေ့ချ် အားလုံးကို တုံ့ပြန်ပါသည်။
- `onchar`: မက်ဆေ့ချ်သည် trigger prefix ဖြင့် စတင်သောအခါ တုံ့ပြန်ပါသည်။

Config ဥပမာ:

```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"],
    },
  },
}
```

မှတ်ချက်များ:

- `onchar` သည် ထင်ရှားသော @mention များကို ဆက်လက် တုံ့ပြန်ပါသည်။
- `channels.mattermost.requireMention` ကို legacy configs များအတွက် လိုက်နာပေးသော်လည်း `chatmode` ကို အသုံးပြုရန် ပိုမို အကြံပြုပါသည်။

## အသုံးပြုခွင့် ထိန်းချုပ်မှု (DMs)

- Default: `channels.mattermost.dmPolicy = "pairing"` (မသိသော ပို့သူများသည် pairing code ရရှိပါမည်)။
- အတည်ပြုရန်:
  - `openclaw pairing list mattermost`
  - `openclaw pairing approve mattermost <CODE>`
- Public DMs: `channels.mattermost.dmPolicy="open"` နှင့်အတူ `channels.mattermost.allowFrom=["*"]`။

## Channels (အုပ်စုများ)

- Default: `channels.mattermost.groupPolicy = "allowlist"` (mention-gated)။
- `channels.mattermost.groupAllowFrom` ဖြင့် ပို့သူများကို allowlist သတ်မှတ်ပါ (user IDs သို့မဟုတ် `@username`)။
- Open channels: `channels.mattermost.groupPolicy="open"` (mention-gated)။

## Targets for outbound delivery

`openclaw message send` သို့မဟုတ် cron/webhooks နှင့်အတူ အောက်ပါ target format များကို အသုံးပြုပါ-

- ချန်နယ်အတွက် `channel:<id>`
- DM အတွက် `user:<id>`
- DM အတွက် `@username` (Mattermost API ဖြင့် ဖြေရှင်းထားသည်)

ID အလွတ်များကို ချန်နယ်များအဖြစ် သတ်မှတ်ပါသည်။

## Multi-account

Mattermost သည် `channels.mattermost.accounts` အောက်တွင် အကောင့်များစွာကို ပံ့ပိုးပါသည်-

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" },
      },
    },
  },
}
```

## Troubleshooting

- ချန်နယ်များတွင် တုံ့ပြန်မှု မရှိပါက: ဘော့တ်ကို ချန်နယ်အတွင်း ထည့်ထားပြီး mention လုပ်ထားကြောင်း (oncall) သေချာပါစေ၊ trigger prefix (onchar) ကို အသုံးပြုပါ၊ သို့မဟုတ် `chatmode: "onmessage"` ကို သတ်မှတ်ပါ။
- Auth အမှားများ: bot token၊ base URL နှင့် အကောင့်ကို ဖွင့်ထားခြင်းရှိမရှိ စစ်ဆေးပါ။
- Multi-account ပြဿနာများ: env vars များသည် `default` အကောင့်အတွက်သာ သက်ရောက်ပါသည်။

