---
summary: "CLI حوالہ برائے `openclaw cron` (شیڈول بنانا اور پس منظر میں جابز چلانا)"
read_when:
  - آپ کو شیڈول شدہ جابز اور ویک اپس درکار ہوں
  - آپ cron کی عمل درآمد اور لاگز کی جانچ کر رہے ہوں
title: "cron"
---

# `openclaw cron`

Gateway شیڈیولر کے لیے cron جابز کا نظم کریں۔

متعلقہ:

- کرون جابز: [کرون جابز](/automation/cron-jobs)

مشورہ: مکمل کمانڈ سطح کے لیے `openclaw cron --help` چلائیں۔

نوٹ: علیحدہ `cron add` جابز بطورِ ڈیفالٹ `--announce` ڈلیوری ہوتی ہیں۔ برقرار رکھنے کے لیے `--no-deliver` استعمال کریں
output internal. `--deliver` remains as a deprecated alias for `--announce`.

نوٹ: ایک بار چلنے والی (`--at`) جابز کامیابی کے بعد بطورِ ڈیفالٹ حذف ہو جاتی ہیں۔ انہیں برقرار رکھنے کے لیے `--keep-after-run` استعمال کریں۔

نوٹ: بار بار چلنے والی جابز اب مسلسل غلطیوں کے بعد ایکسپونینشل ری ٹرائی بیک آف استعمال کرتی ہیں (30s → 1m → 5m → 15m → 60m)، پھر اگلی کامیاب رن کے بعد معمول کے شیڈول پر واپس آ جاتی ہیں۔

## عام ترامیم

پیغام بدلے بغیر ڈیلیوری کی ترتیبات اپڈیٹ کریں:

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

علیحدہ جاب کے لیے ڈیلیوری غیر فعال کریں:

```bash
openclaw cron edit <job-id> --no-deliver
```

کسی مخصوص چینل میں اعلان کریں:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```
