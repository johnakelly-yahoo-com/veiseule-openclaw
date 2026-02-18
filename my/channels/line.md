---
title: LINE
---

# LINE (plugin)

LINE သည် LINE Messaging API မှတဆင့် OpenClaw နှင့် ချိတ်ဆက်သည်။ plugin သည် webhook အဖြစ် လည်ပတ်သည်။
receiver on the gateway and uses your channel access token + channel secret for
authentication.

အခြေအနေ - plugin မှတဆင့် ပံ့ပိုးထားသည်။ တိုက်ရိုက်မက်ဆေ့ချ်များ၊ အုပ်စုစကားပြောများ၊ မီဒီယာ၊ တည်နေရာများ၊ Flex
messages, template messages, and quick replies are supported. Reactions and threads
are not supported.

## Plugin လိုအပ်သည်

LINE ပလပ်ဂင်ကို ထည့်သွင်းတပ်ဆင်ပါ:

```bash
openclaw plugins install @openclaw/line
```

Local checkout (git repo မှ လည်ပတ်စေသောအခါ):

```bash
openclaw plugins install ./extensions/line
```

## တပ်ဆင်ခြင်း

1. LINE Developers အကောင့်တစ်ခု ဖန်တီးပြီး Console ကို ဖွင့်ပါ:
   [https://developers.line.biz/console/](https://developers.line.biz/console/)
2. Provider တစ်ခုကို ဖန်တီးပါ (သို့မဟုတ် ရွေးချယ်ပါ) နှင့် **Messaging API** channel တစ်ခု ထည့်ပါ။
3. channel settings မှ **Channel access token** နှင့် **Channel secret** ကို ကူးယူပါ။
4. Messaging API settings တွင် **Use webhook** ကို ဖွင့်ပါ။
5. webhook URL ကို သင့် Gateway endpoint သို့ သတ်မှတ်ပါ (HTTPS လိုအပ်သည်):

```
https://gateway-host/line/webhook
```

gateway သည် LINE ၏ webhook အတည်ပြုမှု (GET) နှင့် ဝင်လာသော အဖြစ်အပျက်များ (POST) ကို တုံ့ပြန်သည်။
If you need a custom path, set `channels.line.webhookPath` or
`channels.line.accounts.<id>.webhookPath` and update the URL accordingly.

## ပြင်ဆင်သတ်မှတ်ခြင်း

အနည်းဆုံး ဖွဲ့စည်းပြင်ဆင်မှု:

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing",
    },
  },
}
```

Env vars (default account အတွက်သာ):

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

Token/secret ဖိုင်များ:

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt",
    },
  },
}
```

အကောင့်များ အများအပြား:

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing",
        },
      },
    },
  },
}
```

## ဝင်ရောက်အသုံးပြုခွင့် ထိန်းချုပ်မှု

တိုက်ရိုက်မက်ဆေ့ချ်များသည် ပုံမှန်အားဖြင့် pairing လုပ်ခြင်းကို သတ်မှတ်ထားသည်။ မသိသေးသော ပို့ဆောင်သူများသည် pairing code တစ်ခုကို လက်ခံရရှိပြီး ၎င်းတို့၏
messages are ignored until approved.

```bash
openclaw pairing list line
openclaw pairing approve line <CODE>
```

Allowlists နှင့် မူဝါဒများ:

- `channels.line.dmPolicy`: `pairing | allowlist | open | disabled`
- `channels.line.allowFrom`: DM မက်ဆေ့ချ်များအတွက် allowlist ပြုလုပ်ထားသော LINE user ID များ
- `channels.line.groupPolicy`: `allowlist | open | disabled`
- `channels.line.groupAllowFrom`: အုပ်စုများအတွက် allowlist ပြုလုပ်ထားသော LINE user ID များ
- Per-group overrides: `channels.line.groups.<groupId>.allowFrom`

LINE IDs are case-sensitive. Valid IDs look like:

- User: `U` + 32 hex chars
- Group: `C` + 32 hex chars
- Room: `R` + 32 hex chars

## Message behavior

- စာသားများကို အက္ခရာ 5000 လုံးစီအလိုက် ခွဲထုတ်ပို့ပါသည်။
- Markdown ဖော်မတ်ကို ဖယ်ရှားပြီး code blocks နှင့် tables များကို ဖြစ်နိုင်သလောက် Flex
  ကတ်များအဖြစ် ပြောင်းလဲပါသည်။
- Streaming တုံ့ပြန်ချက်များကို buffer လုပ်ထားပြီး agent လုပ်ဆောင်နေစဉ် LINE သည် loading
  animation ပါသော chunk အပြည့်အစုံများကို လက်ခံရရှိပါသည်။
- မီဒီယာ ဒေါင်းလုဒ်များကို `channels.line.mediaMaxMb` ဖြင့် ကန့်သတ်ထားပါသည် (default 10)။

## Channel data (rich messages)

quick replies၊ တည်နေရာများ၊ Flex ကတ်များ သို့မဟုတ် template
မက်ဆေ့ချ်များ ပို့ရန် `channelData.line` ကို အသုံးပြုပါ။

```json5
{
  text: "Here you go",
  channelData: {
    line: {
      quickReplies: ["Status", "Help"],
      location: {
        title: "Office",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125,
      },
      flexMessage: {
        altText: "Status card",
        contents: {
          /* Flex payload */
        },
      },
      templateMessage: {
        type: "confirm",
        text: "Proceed?",
        confirmLabel: "Yes",
        confirmData: "yes",
        cancelLabel: "No",
        cancelData: "no",
      },
    },
  },
}
```

LINE ပလပ်ဂင်တွင် Flex မက်ဆေ့ချ် preset များအတွက် `/card` command ကိုလည်း ထည့်သွင်းပေးထားပါသည်:

```
/card info "Welcome" "Thanks for joining!"
```

## Troubleshooting

- **Webhook verification မအောင်မြင်ပါက:** webhook URL သည် HTTPS ဖြစ်ကြောင်းနှင့်
  `channelSecret` သည် LINE console နှင့် ကိုက်ညီကြောင်း သေချာစေပါ။
- **Inbound events မရှိပါက:** webhook path သည် `channels.line.webhookPath` နှင့် ကိုက်ညီကြောင်း
  နှင့် Gateway ကို LINE မှ ရောက်ရှိနိုင်ကြောင်း အတည်ပြုပါ။
- **Media download အမှားများ:** မီဒီယာသည် default ကန့်သတ်ချက်ကို ကျော်လွန်ပါက
  `channels.line.mediaMaxMb` ကို တိုးမြှင့်ပါ။

