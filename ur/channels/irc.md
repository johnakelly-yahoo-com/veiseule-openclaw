---
title: IRC
description: OpenClaw کو IRC چینلز اور ڈائریکٹ میسجز سے منسلک کریں۔
---

جب آپ چاہتے ہیں کہ OpenClaw کلاسک چینلز (`#room`) اور ڈائریکٹ میسجز میں ہو تو IRC استعمال کریں۔
IRC ایک ایکسٹینشن پلگ اِن کے طور پر آتا ہے، لیکن اسے مین کنفیگ میں `channels.irc` کے تحت ترتیب دیا جاتا ہے۔

## فوری آغاز

1. `~/.openclaw/openclaw.json` میں IRC کنفیگ فعال کریں۔
2. کم از کم یہ سیٹ کریں:

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

3. گیٹ وے شروع/دوبارہ شروع کریں:

```bash
openclaw gateway run
```

## سیکیورٹی کی ڈیفالٹس

- `channels.irc.dmPolicy` کی ڈیفالٹ ویلیو `"pairing"` ہے۔
- `channels.irc.groupPolicy` کی ڈیفالٹ ویلیو `"allowlist"` ہے۔
- جب `groupPolicy="allowlist"` ہو، تو اجازت یافتہ چینلز متعین کرنے کے لیے `channels.irc.groups` سیٹ کریں۔
- جب تک آپ جان بوجھ کر plaintext ٹرانسپورٹ قبول نہ کریں، TLS (`channels.irc.tls=true`) استعمال کریں۔

## رسائی کا کنٹرول

IRC چینلز کے لیے دو الگ “گیٹس” ہیں:

1. **Channel access** (`groupPolicy` + `groups`): آیا بوٹ کسی چینل سے پیغامات قبول کرتا ہے یا نہیں۔
2. **Sender access** (`groupAllowFrom` / فی-چینل `groups["#channel"].allowFrom`): اس چینل کے اندر کون بوٹ کو ٹرگر کر سکتا ہے۔

کنفیگ کیز:

- DM allowlist (DM بھیجنے والے کی رسائی): `channels.irc.allowFrom`
- گروپ بھیجنے والے کی allowlist (چینل بھیجنے والے کی رسائی): `channels.irc.groupAllowFrom`
- فی-چینل کنٹرولز (چینل + بھیجنے والا + مینشن قواعد): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` غیر کنفیگرڈ چینلز کی اجازت دیتا ہے (**پھر بھی ڈیفالٹ کے طور پر mention-gated رہتا ہے**)

Allowlist اندراجات میں nick یا `nick!user@host` فارمز استعمال کیے جا سکتے ہیں۔

### عام غلطی: `allowFrom` DMs کے لیے ہے، چینلز کے لیے نہیں

اگر آپ کو لاگز میں یہ نظر آئے:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…تو اس کا مطلب ہے کہ **گروپ/چینل** پیغامات کے لیے بھیجنے والے کو اجازت نہیں تھی۔ اسے یوں درست کریں:

- یا تو `channels.irc.groupAllowFrom` سیٹ کریں (تمام چینلز کے لیے عالمی طور پر)،
- یا فی-چینل بھیجنے والے کی allowlists سیٹ کریں: `channels.irc.groups["#channel"].allowFrom`

مثال (`#tuirc-dev` میں کسی کو بھی بوٹ سے بات کرنے کی اجازت دینے کے لیے):

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

## جواب ٹرگر کرنا (mentions)

اگرچہ کوئی چینل (`groupPolicy` + `groups` کے ذریعے) مجاز ہو اور بھیجنے والا بھی مجاز ہو، پھر بھی OpenClaw گروپ سیاق و سباق میں ڈیفالٹ کے طور پر **mention-gating** استعمال کرتا ہے۔

اس کا مطلب ہے کہ آپ کو لاگز میں `drop channel …` نظر آ سکتا ہے `(missing-mention)` جب تک کہ پیغام میں ایسا mention پیٹرن شامل نہ ہو جو بوٹ سے میچ کرے۔

کسی IRC چینل میں بوٹ سے **mention کے بغیر** جواب دلوانے کے لیے، اس چینل کے لیے mention gating غیر فعال کریں:

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

یا تمام IRC چینلز کو **مکمل طور پر** اجازت دینے کے لیے (فی چینل allowlist کے بغیر) اور پھر بھی مینشن کے بغیر جواب دینے کے لیے:

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

## سیکیورٹی نوٹ (عوامی چینلز کے لیے تجویز کردہ)

اگر آپ کسی عوامی چینل میں `allowFrom: ["*"]` کی اجازت دیتے ہیں تو کوئی بھی بوٹ کو پرامپٹ کر سکتا ہے۔
خطرہ کم کرنے کے لیے، اس چینل کے ٹولز کو محدود کریں۔

### چینل میں سب کے لیے یکساں ٹولز

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

### ہر بھیجنے والے کے لیے مختلف ٹولز (مالک کو زیادہ اختیارات ملتے ہیں)

`toolsBySender` استعمال کریں تاکہ `"*"` پر زیادہ سخت پالیسی اور اپنے نک پر نسبتاً نرم پالیسی لاگو کی جا سکے:

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

نوٹس:

- `toolsBySender` کی keys ایک نک (مثلاً `"eigen"`) یا مکمل ہوسٹ ماسک (`"eigen!~eigen@174.127.248.171"`) ہو سکتی ہیں تاکہ شناخت کی زیادہ مضبوط مطابقت حاصل ہو سکے۔
- سب سے پہلے میچ ہونے والی sender پالیسی لاگو ہوتی ہے؛ `"*"` بطور وائلڈ کارڈ fallback استعمال ہوتا ہے۔

گروپ رسائی اور مینشن گیٹنگ (اور ان کے باہمی تعامل) کے بارے میں مزید معلومات کے لیے دیکھیں: [/channels/groups](/channels/groups).

## NickServ

کنیکٹ ہونے کے بعد NickServ کے ساتھ شناخت کرنے کے لیے:

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

کنیکٹ پر اختیاری ایک بار رجسٹریشن:

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

نک رجسٹر ہونے کے بعد بار بار REGISTER کی کوششوں سے بچنے کے لیے `register` کو غیر فعال کر دیں۔

## ماحولیاتی ویری ایبلز

ڈیفالٹ اکاؤنٹ درج ذیل کو سپورٹ کرتا ہے:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (کوما سے جدا شدہ)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## خرابیوں کا ازالہ

- اگر بوٹ کنیکٹ ہو جاتا ہے لیکن چینلز میں کبھی جواب نہیں دیتا، تو `channels.irc.groups` **اور** یہ چیک کریں کہ کیا مینشن گیٹنگ پیغامات کو ڈراپ کر رہی ہے (`missing-mention`)۔ اگر آپ چاہتے ہیں کہ یہ بغیر پنگ کے جواب دے، تو اس چینل کے لیے `requireMention:false` سیٹ کریں۔
- اگر لاگ اِن ناکام ہو جائے تو نک کی دستیابی اور سرور پاس ورڈ کی تصدیق کریں۔
- اگر کسی کسٹم نیٹ ورک پر TLS ناکام ہو جائے تو host/port اور سرٹیفکیٹ سیٹ اپ کی تصدیق کریں۔

