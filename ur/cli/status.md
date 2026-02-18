---
title: "اسٹیٹس"
---

# `openclaw status`

چینلز اور سیشنز کے لیے تشخیصی معلومات۔

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

نوٹس:

- `--deep` لائیو پروبز چلاتا ہے (WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal)۔
- آؤٹ پٹ میں متعدد ایجنٹس کی کنفیگریشن کی صورت میں فی ایجنٹ سیشن اسٹورز شامل ہوتے ہیں۔
- جائزہ میں Gateway اور نوڈ ہوسٹ سروس کی انسٹال/رن ٹائم اسٹیٹس (جہاں دستیاب ہو) شامل ہوتی ہے۔
- جائزہ میں اپ ڈیٹ چینل اور git SHA (سورس چیک آؤٹس کے لیے) شامل ہوتے ہیں۔
- اپ ڈیٹ کی معلومات جائزہ میں ظاہر ہوتی ہیں؛ اگر کوئی اپ ڈیٹ دستیاب ہو تو اسٹیٹس `openclaw update` چلانے کا اشارہ پرنٹ کرتا ہے (دیکھیے [Updating](/install/updating))۔
