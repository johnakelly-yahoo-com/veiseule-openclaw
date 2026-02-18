---
title: "جوڑی بنانا"
---

# جوڑی بنانا

“Pairing” OpenClaw کا واضح **مالک کی منظوری** کا مرحلہ ہے۔
It is used in two places:

1. **DM جوڑی بنانا** (کون بوٹ سے بات کر سکتا ہے)
2. **نوڈ جوڑی بنانا** (کون سے ڈیوائسز/نوڈز گیٹ وے نیٹ ورک میں شامل ہو سکتے ہیں)

سکیورٹی سیاق: [Security](/gateway/security)

## 1. DM جوڑی بنانا (ان باؤنڈ چیٹ رسائی)

جب کسی چینل کو DM پالیسی `pairing` کے ساتھ کنفیگر کیا جاتا ہے، تو نامعلوم ارسال کنندگان کو ایک مختصر کوڈ ملتا ہے اور آپ کی منظوری تک ان کا پیغام **پروسیس نہیں** کیا جاتا۔

ڈیفالٹ DM پالیسیاں یہاں دستاویزی ہیں: [Security](/gateway/security)

جوڑی بنانے کے کوڈز:

- 8 حروف، بڑے حروف میں، بغیر مبہم حروف (`0O1I`)۔
- **1 گھنٹے بعد میعاد ختم ہو جاتی ہے**۔ بوٹ صرف اسی وقت پیئرنگ پیغام بھیجتا ہے جب نئی درخواست بنائی جاتی ہے (تقریباً ہر بھیجنے والے کے لیے فی گھنٹہ ایک بار).
- زیرِ التوا DM جوڑی بنانے کی درخواستیں بطورِ طے شدہ **ہر چینل پر 3** تک محدود ہیں؛ اضافی درخواستیں اس وقت تک نظرانداز کی جاتی ہیں جب تک کوئی ایک میعاد ختم نہ ہو یا منظور نہ ہو جائے۔

### کسی ارسال کنندہ کی منظوری

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

معاون چینلز: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`۔

### اسٹیٹ کہاں محفوظ ہوتی ہے

`~/.openclaw/credentials/` کے تحت محفوظ:

- زیرِ التوا درخواستیں: `<channel>-pairing.json`
- منظور شدہ اجازت فہرست اسٹور: `<channel>-allowFrom.json`

انہیں حساس سمجھیں (یہ آپ کے اسسٹنٹ تک رسائی کو کنٹرول کرتے ہیں)۔

## 2. نوڈ ڈیوائس جوڑی بنانا (iOS/Android/macOS/ہیڈلیس نوڈز)

Nodes، Gateway سے **devices** کے طور پر `role: node` کے ساتھ جڑتے ہیں۔ Gateway
creates a device pairing request that must be approved.

### Telegram کے ذریعے Pair کریں (iOS کے لیے تجویز کردہ)

اگر آپ `device-pair` plugin استعمال کرتے ہیں، تو آپ پہلی بار ڈیوائس کی pairing مکمل طور پر Telegram سے کر سکتے ہیں:

1. Telegram میں، اپنے بوٹ کو پیغام بھیجیں: `/pair`
2. بوٹ دو پیغامات کے ساتھ جواب دیتا ہے: ایک ہدایتی پیغام اور ایک علیحدہ **setup code** پیغام (جسے Telegram میں آسانی سے کاپی/پیسٹ کیا جا سکتا ہے).
3. اپنے فون پر، OpenClaw iOS ایپ کھولیں → Settings → Gateway.
4. setup code پیسٹ کریں اور کنیکٹ کریں۔
5. واپس Telegram میں: `/pair approve`

The setup code is a base64-encoded JSON payload that contains:

- `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `token`: a short-lived pairing token

Treat the setup code like a password while it is valid.

### نوڈ ڈیوائس کی منظوری

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### نوڈ جوڑی بنانے کی اسٹیٹ کا ذخیرہ

`~/.openclaw/devices/` کے تحت محفوظ:

- `pending.json` (مختصر مدت؛ زیرِ التوا درخواستیں میعاد ختم ہو جاتی ہیں)
- `paired.json` (جوڑی بنے ہوئے ڈیوائسز + ٹوکنز)

### نوٹس

- The legacy `node.pair.*` API (CLI: `openclaw nodes pending/approve`) is a
  separate gateway-owned pairing store. WS nodes still require device pairing.

## متعلقہ دستاویزات

- سکیورٹی ماڈل + پرامپٹ انجیکشن: [Security](/gateway/security)
- محفوظ طریقے سے اپڈیٹ کرنا (رن ڈاکٹر): [Updating](/install/updating)
- چینل کنفیگز:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (لیگیسی): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)


