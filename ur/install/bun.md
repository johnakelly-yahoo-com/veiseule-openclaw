---
title: "Bun (تجرباتی)"
---

# Bun (تجرباتی)

مقصد: اس ریپو کو **Bun** کے ساتھ چلانا (اختیاری، WhatsApp/Telegram کے لیے سفارش نہیں کی جاتی)
اور pnpm ورک فلو سے انحراف کیے بغیر۔

38. ⚠️ **Gateway runtime کے لیے تجویز نہیں کیا جاتا** (WhatsApp/Telegram بگز)۔ Use Node for production.

## حالت

- Bun TypeScript کو براہِ راست چلانے کے لیے ایک اختیاری لوکل رن ٹائم ہے (`bun run …`, `bun --watch …`)۔
- `pnpm` بلڈز کے لیے بطورِ طے شدہ ہے اور مکمل طور پر معاون رہتا ہے (اور کچھ ڈاکس ٹولنگ کے ذریعے استعمال بھی ہوتا ہے)۔
- Bun `pnpm-lock.yaml` استعمال نہیں کر سکتا اور اسے نظرانداز کرے گا۔

## انسٹال کریں

بطورِ طے شدہ:

```sh
bun install
```

نوٹ: `bun.lock`/`bun.lockb` کو gitignore کیا گیا ہے، اس لیے کسی بھی صورت میں ریپوزٹری میں غیر ضروری تبدیلیاں نہیں ہوں گی۔ اگر آپ _کوئی لاک فائل تحریر نہ ہو_ چاہتے ہیں:

```sh
bun install --no-save
```

## بلڈ / ٹیسٹ (Bun)

```sh
bun run build
bun run vitest run
```

## Bun lifecycle اسکرپٹس (بطورِ طے شدہ مسدود)

Bun انحصارات کے لائف سائیکل اسکرپٹس کو بلاک کر سکتا ہے جب تک کہ انہیں واضح طور پر قابلِ اعتماد قرار نہ دیا جائے (`bun pm untrusted` / `bun pm trust`)۔
For this repo, the commonly blocked scripts are not required:

- `@whiskeysockets/baileys` `preinstall`: Node کے major ورژن >= 20 کی جانچ (ہم Node 22+ چلاتے ہیں)۔
- `protobufjs` `postinstall`: غیر مطابقت پذیر ورژن اسکیمز کے بارے میں انتباہات جاری کرتی ہیں (کوئی بلڈ آرٹیفیکٹس نہیں)۔

اگر آپ کو کسی حقیقی رن ٹائم مسئلے کا سامنا ہو جس کے لیے ان اسکرپٹس کی ضرورت ہو، تو انہیں صراحتاً قابلِ اعتماد بنائیں:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## اہم نکات

- Some scripts still hardcode pnpm (e.g. `docs:build`, `ui:*`, `protocol:check`). 39. فی الحال انہیں pnpm کے ذریعے چلائیں۔

