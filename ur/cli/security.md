---
summary: "CLI کے لیے `openclaw security` کا حوالہ (عام سکیورٹی خامیوں کی آڈٹ اور درستگی)"
read_when:
  - آپ کنفیگ/اسٹیٹ پر فوری سکیورٹی آڈٹ چلانا چاہتے ہوں
  - آپ محفوظ “fix” تجاویز (chmod، ڈیفالٹس کو سخت کرنا) لاگو کرنا چاہتے ہوں
title: "سکیورٹی"
---

# `openclaw security`

سکیورٹی کے اوزار (آڈٹ + اختیاری درستگیاں)۔

متعلقہ:

- سکیورٹی گائیڈ: [سکیورٹی](/gateway/security)

## آڈٹ

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

10. آڈٹ اس وقت خبردار کرتا ہے جب متعدد DM بھیجنے والے مرکزی سیشن شیئر کریں اور **محفوظ DM موڈ** کی سفارش کرتا ہے: `session.dmScope="per-channel-peer"` (یا ملٹی اکاؤنٹ چینلز کے لیے `per-account-channel-peer`) مشترکہ ان باکسز کے لیے۔
11. یہ اس وقت بھی خبردار کرتا ہے جب چھوٹے ماڈلز (`<=300B`) سینڈ باکسنگ کے بغیر اور ویب/براؤزر ٹولز فعال ہونے کے ساتھ استعمال کیے جائیں۔
    webhook ingress کے لیے، یہ اس وقت وارننگ دیتا ہے جب `hooks.defaultSessionKey` سیٹ نہ ہو، جب درخواست کے `sessionKey` اووررائیڈز فعال ہوں، اور جب اووررائیڈز `hooks.allowedSessionKeyPrefixes` کے بغیر فعال ہوں۔
    یہ اس وقت بھی وارننگ دیتا ہے جب sandbox Docker سیٹنگز کنفیگر ہوں لیکن sandbox موڈ بند ہو، جب `gateway.nodes.denyCommands` غیر مؤثر pattern جیسے/نامعلوم اندراجات استعمال کرے، جب گلوبل `tools.profile="minimal"` کو agent tool پروفائلز اووررائیڈ کریں، اور جب انسٹال شدہ extension plugin tools نرم tool پالیسی کے تحت قابل رسائی ہو سکتے ہوں۔

