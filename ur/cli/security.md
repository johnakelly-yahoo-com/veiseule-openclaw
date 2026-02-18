---
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


