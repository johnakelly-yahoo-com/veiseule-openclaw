---
title: "SOUL Evil ہُک"
---

# SOUL Evil ہُک

SOUL Evil hook دوران **injected** `SOUL.md` مواد کو `SOUL_EVIL.md` سے تبدیل کر دیتا ہے
a purge window or by random chance. It does **not** modify files on disk.

## یہ کیسے کام کرتا ہے

جب `agent:bootstrap` چلتا ہے، تو hook میموری میں `SOUL.md` کے مواد کو تبدیل کر سکتا ہے
before the system prompt is assembled. If `SOUL_EVIL.md` is missing or empty,
OpenClaw logs a warning and keeps the normal `SOUL.md`.

سب-ایجنٹ رنز میں اپنے بوٹسٹریپ فائلز میں `SOUL.md` شامل نہیں ہوتا، اس لیے اس ہُک کا سب-ایجنٹس پر کوئی اثر نہیں پڑتا۔

## فعال کریں

```bash
openclaw hooks enable soul-evil
```

پھر کنفیگ سیٹ کریں:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

ایجنٹ ورک اسپیس روٹ میں `SOUL_EVIL.md` بنائیں (بالکل `SOUL.md` کے ساتھ)۔

## اختیارات

- `file` (string): متبادل SOUL فائل نام (بطورِ طے شدہ: `SOUL_EVIL.md`)
- `chance` (number 0–1): ہر رن میں `SOUL_EVIL.md` استعمال کرنے کا رینڈم امکان
- `purge.at` (HH:mm): روزانہ purge شروع ہونے کا وقت (24-گھنٹے گھڑی)
- `purge.duration` (duration): ونڈو کی مدت (مثلاً `30s`، `10m`، `1h`)

**ترجیح:** purge ونڈو کو رینڈم امکان پر فوقیت حاصل ہے۔

**ٹائم زون:** اگر `agents.defaults.userTimezone` سیٹ ہو تو اسے استعمال کیا جاتا ہے؛ بصورتِ دیگر ہوسٹ ٹائم زون۔

## نوٹس

- ڈسک پر کوئی فائل لکھی یا ترمیم نہیں کی جاتی۔
- اگر بوٹسٹریپ فہرست میں `SOUL.md` شامل نہ ہو تو ہُک کچھ نہیں کرتا۔

## یہ بھی دیکھیں

- [ہکس](/automation/hooks)


