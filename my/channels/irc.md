---
title: IRC
description: OpenClaw ကို IRC channels နှင့် direct messages များသို့ ချိတ်ဆက်ပါ။
---

OpenClaw ကို ရိုးရာ channels (`#room`) များနှင့် direct messages များတွင် အသုံးပြုလိုသောအခါ IRC ကို အသုံးပြုပါ။
IRC သည် extension plugin အဖြစ် ပါရှိသော်လည်း `channels.irc` အောက်ရှိ main config တွင် ပြင်ဆင်သတ်မှတ်ရပါသည်။

## အမြန်စတင်ရန်

1. `~/.openclaw/openclaw.json` တွင် IRC config ကို ဖွင့်ပါ။
2. အနည်းဆုံး အောက်ပါတို့ကို သတ်မှတ်ပါ-

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. gateway ကို စတင်/ပြန်စတင်ပါ-

```bash
openclaw gateway run
```

## လုံခြုံရေး မူလတန်ဖိုးများ

- `channels.irc.dmPolicy` ၏ မူလတန်ဖိုးမှာ `"pairing"` ဖြစ်သည်။
- `channels.irc.groupPolicy` ၏ မူလတန်ဖိုးမှာ `"allowlist"` ဖြစ်သည်။
- `groupPolicy="allowlist"` ကို အသုံးပြုပါက ခွင့်ပြုထားသော channels များကို သတ်မှတ်ရန် `channels.irc.groups` ကို ပြင်ဆင်ပါ။
- ရည်ရွယ်ချက်ရှိရှိ plaintext transport ကို လက်ခံလိုခြင်း မရှိပါက TLS (`channels.irc.tls=true`) ကို အသုံးပြုပါ။

## အသုံးပြုခွင့် ထိန်းချုပ်မှု

IRC channels များအတွက် သီးခြား “gates” နှစ်ခု ရှိသည်-

1. **Channel access** (`groupPolicy` + `groups`): bot သည် channel တစ်ခုမှ မက်ဆေ့ခ်ျများကို လက်ခံမလား မလက်ခံမလား။
2. **Sender access** (`groupAllowFrom` / per-channel `groups["#channel"].allowFrom`): ထို channel အတွင်း bot ကို မည်သူများက trigger လုပ်ခွင့်ရှိသနည်း။

Config keys:

- DM allowlist (DM sender access): `channels.irc.allowFrom`
- Group sender allowlist (channel sender access): `channels.irc.groupAllowFrom`
- Per-channel controls (channel + sender + mention rules): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` သည် မပြင်ဆင်ထားသော channels များကို ခွင့်ပြုသည် (**မူလအတိုင်း mention-gated ဖြစ်နေဆဲဖြစ်သည်**)

Allowlist entries များတွင် nick သို့မဟုတ် `nick!user@host` ပုံစံများကို အသုံးပြုနိုင်သည်။

### အများဆုံး တွေ့ရသော ပြဿနာ: `allowFrom` သည် DMs အတွက်သာ ဖြစ်ပြီး channels များအတွက် မဟုတ်ပါ

အောက်ပါကဲ့သို့သော logs များကို တွေ့ပါက-

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…၎င်းသည် **group/channel** မက်ဆေ့ခ်ျများအတွက် sender ကို ခွင့်မပြုထားခြင်းကို ဆိုလိုသည်။ အောက်ပါနည်းလမ်းများထဲမှ တစ်ခုဖြင့် ပြင်ဆင်နိုင်သည်-

- `channels.irc.groupAllowFrom` ကို သတ်မှတ်ခြင်း (channels အားလုံးအတွက် global) သို့မဟုတ်
- per-channel sender allowlists ကို သတ်မှတ်ခြင်း- `channels.irc.groups["#channel"].allowFrom`

ဥပမာ (`#tuirc-dev` တွင် မည်သူမဆို bot နှင့် စကားပြောနိုင်ရန် ခွင့်ပြုခြင်း):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Reply triggering (mentions)

Channel ကို ခွင့်ပြုထားပြီးသားဖြစ်သော်လည်း (`groupPolicy` + `groups`) နှင့် sender ကိုလည်း ခွင့်ပြုထားပါက OpenClaw သည် group contexts တွင် မူလအတိုင်း **mention-gating** ကို အသုံးပြုသည်။

ဆိုလိုသည်မှာ `drop channel … ကဲ့သို့သော logs များကို တွေ့နိုင်သည် (missing-mention)` ဟု ပြသမည်ဖြစ်ပြီး မက်ဆေ့ခ်ျအတွင်း bot နှင့် ကိုက်ညီသော mention pattern မပါရှိပါက ထိုသို့ ဖြစ်ပေါ်မည်ဖြစ်သည်။

IRC channel တစ်ခုတွင် **mention မလိုအပ်ဘဲ** bot ကို reply လုပ်စေရန် ထို channel အတွက် mention gating ကို ပိတ်ပါ-

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

သို့မဟုတ် **IRC channel အားလုံးကို** (channel တစ်ခုချင်းစီအလိုက် allowlist မသတ်မှတ်ဘဲ) ခွင့်ပြုပြီး mention မပါဘဲလည်း reply ပြန်နိုင်ရန်:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## လုံခြုံရေး မှတ်ချက် (public channel များအတွက် အကြံပြုထားသည်)

Public channel တွင် `allowFrom: ["*"]` ကို ခွင့်ပြုပါက မည်သူမဆို bot ကို prompt လုပ်နိုင်ပါသည်။
အန္တရာယ် လျှော့ချရန် အဆိုပါ channel အတွက် tools များကို ကန့်သတ်ပါ။

### Channel ထဲရှိ လူတိုင်းအတွက် tools တူညီစွာ သတ်မှတ်ခြင်း

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Sender အလိုက် tools မတူညီစွာ သတ်မှတ်ခြင်း (owner သည် အာဏာပိုရှိသည်)

`toolsBySender` ကို အသုံးပြုပြီး `"*"` အတွက် ပိုမိုတင်းကျပ်သော policy သတ်မှတ်ကာ သင့် nick အတွက် ပိုမိုလျော့ရဲသော policy သတ်မှတ်နိုင်သည်:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

မှတ်ချက်များ:

- `toolsBySender` ၏ key များသည် nick (ဥပမာ `"eigen"`) သို့မဟုတ် ပိုမိုတိကျသော identity ကို ကိုက်ညီစေရန် full hostmask (`"eigen!~eigen@174.127.248.171"`) ဖြစ်နိုင်သည်။
- ကိုက်ညီသော sender policy များအနက် ပထမဆုံး ကိုက်ညီသော policy ကို အသုံးပြုမည်ဖြစ်သည်။ `"*"` သည် wildcard fallback ဖြစ်သည်။

Group access နှင့် mention-gating (နှင့် ၎င်းတို့၏ အပြန်အလှန် သက်ရောက်မှု) အကြောင်း ပိုမိုသိရှိလိုပါက ဤနေရာတွင် ကြည့်ပါ: [/channels/groups](/channels/groups)。

## NickServ

Connect ပြီးနောက် NickServ ဖြင့် identify ပြုလုပ်ရန်:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Connect ပြုလုပ်ချိန်တွင် တစ်ကြိမ်သာ registration ပြုလုပ်ရန် (optional):

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Nick ကို register ပြီးပါက `register` ကို disable လုပ်ပါ၊ ထပ်ခါတလဲလဲ REGISTER ကြိုးပမ်းမှု မဖြစ်စေရန်။

## Environment variables

Default account သည် အောက်ပါအချက်များကို ထောက်ပံ့သည်:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (comma ဖြင့် ခွဲထားသည်)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## ပြဿနာဖြေရှင်းခြင်း

- Bot သည် connect လုပ်နိုင်သော်လည်း channel များတွင် reply မပြန်ပါက `channels.irc.groups` ကို စစ်ဆေးပါ **နှင့်** mention-gating ကြောင့် message များကို ပယ်ချနေခြင်းရှိမရှိ (`missing-mention`) စစ်ဆေးပါ။ Ping မလိုဘဲ reply ပြန်လိုပါက အဆိုပါ channel အတွက် `requireMention:false` ကို သတ်မှတ်ပါ။
- Login မအောင်မြင်ပါက nick ရရှိနိုင်မှုနှင့် server password ကို စစ်ဆေးပါ။
- Custom network တွင် TLS မအောင်မြင်ပါက host/port နှင့် certificate setup ကို စစ်ဆေးပါ။

