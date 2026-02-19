---
summary: "Tlon/Urbit کی معاونت کی حیثیت، صلاحیتیں، اور کنفیگریشن"
read_when:
  - Tlon/Urbit چینل کی خصوصیات پر کام کرتے وقت
title: "Tlon"
---

# Tlon (plugin)

Tlon is a decentralized messenger built on Urbit. OpenClaw connects to your Urbit ship and can
respond to DMs and group chat messages. Group replies require an @ mention by default and can
be further restricted via allowlists.

Status: supported via plugin. DMs, group mentions, thread replies, and text-only media fallback
(URL appended to caption). Reactions, polls, and native media uploads are not supported.

## پلگ اِن درکار ہے

Tlon ایک پلگ ان کے طور پر فراہم کیا جاتا ہے اور کور انسٹال کے ساتھ شامل نہیں ہوتا۔

CLI کے ذریعے انسٹال کریں (npm رجسٹری):

```bash
openclaw plugins install @openclaw/tlon
```

مقامی چیک آؤٹ (جب git ریپو سے چلایا جا رہا ہو):

```bash
openclaw plugins install ./extensions/tlon
```

تفصیلات: [Plugins](/tools/plugin)

## سیٹ اپ

1. Tlon پلگ ان انسٹال کریں۔
2. اپنے ship کا URL اور لاگ اِن کوڈ جمع کریں۔
3. `channels.tlon` کنفیگر کریں۔
4. gateway کو ری اسٹارٹ کریں۔
5. بوٹ کو DM کریں یا کسی گروپ چینل میں اس کا ذکر کریں۔

کم از کم کنفیگ (سنگل اکاؤنٹ):

```json5
{
  channels: {
    tlon: {
      enabled: true,
      ship: "~sampel-palnet",
      url: "https://your-ship-host",
      code: "lidlut-tabwed-pillex-ridrup",
    },
  },
}
```

Private/LAN ship URLs (ایڈوانسڈ):

ڈیفالٹ طور پر، OpenClaw اس پلگ اِن کے لیے نجی/اندرونی ہوسٹ نیمز اور IP رینجز کو بلاک کرتا ہے (SSRF ہارڈننگ)۔
اگر آپ کا ship URL نجی نیٹ ورک پر ہے (مثال کے طور پر `http://192.168.1.50:8080` یا `http://localhost:8080`)،
آپ کو واضح طور پر اجازت دینی ہوگی:

```json5
{
  channels: {
    tlon: {
      allowPrivateNetwork: true,
    },
  },
}
```

## Group channels

Auto-discovery is enabled by default. You can also pin channels manually:

```json5
{
  channels: {
    tlon: {
      groupChannels: ["chat/~host-ship/general", "chat/~host-ship/support"],
    },
  },
}
```

DM اجازت فہرست (خالی = سب کی اجازت):

```json5
{
  channels: {
    tlon: {
      autoDiscoverChannels: false,
    },
  },
}
```

## Access control

DM اجازت فہرست (خالی = سب کی اجازت):

```json5
{
  channels: {
    tlon: {
      dmAllowlist: ["~zod", "~nec"],
    },
  },
}
```

انہیں `openclaw message send` یا cron ڈیلیوری کے ساتھ استعمال کریں:

```json5
{
  channels: {
    tlon: {
      defaultAuthorizedShips: ["~zod"],
      authorization: {
        channelRules: {
          "chat/~host-ship/general": {
            mode: "restricted",
            allowedShips: ["~zod", "~nec"],
          },
          "chat/~host-ship/announcements": {
            mode: "open",
          },
        },
      },
    },
  },
}
```

## Delivery targets (CLI/cron)

انہیں `openclaw message send` یا cron ڈیلیوری کے ساتھ استعمال کریں:

- DM: `~sampel-palnet` یا `dm/~sampel-palnet`
- Group: `chat/~host-ship/channel` یا `group:~host-ship/channel`

## Notes

- گروپس میں جواب دینے کے لیے mention درکار ہے (مثلاً `~your-bot-ship`)۔
- تھریڈ جوابات: اگر آنے والا پیغام کسی تھریڈ میں ہو تو OpenClaw اسی تھریڈ میں جواب دیتا ہے۔
- میڈیا: `sendMedia` متن + URL پر فال بیک کرتا ہے (native اپلوڈ نہیں)۔
