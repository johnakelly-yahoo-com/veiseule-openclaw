---
title: "چینل لوکیشن پارسنگ"
---

# چینل لوکیشن پارسنگ

OpenClaw چیٹ چینلز سے شیئر کی گئی لوکیشنز کو درج ذیل صورتوں میں معمول پر لاتا ہے:

- ان باؤنڈ باڈی کے آخر میں شامل کیا گیا انسان دوست متن، اور
- خودکار جواب کے سیاق پے لوڈ میں ساختی فیلڈز۔

فی الحال معاونت یافتہ:

- **Telegram** (لوکیشن پنز + وینیوز + لائیو لوکیشنز)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` کے ساتھ `geo_uri`)

## متن کی فارمیٹنگ

لوکیشنز بغیر بریکٹس کے دوستانہ سطور کی صورت میں رینڈر کی جاتی ہیں:

- پن:
  - `📍 48.858844, 2.294351 ±12m`
- نامزد مقام:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- لائیو شیئر:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

اگر چینل میں کیپشن/تبصرہ شامل ہو تو اسے اگلی سطر میں شامل کیا جاتا ہے:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## سیاق کے فیلڈز

جب لوکیشن موجود ہو تو یہ فیلڈز `ctx` میں شامل کیے جاتے ہیں:

- `LocationLat` (عدد)
- `LocationLon` (عدد)
- `LocationAccuracy` (عدد، میٹر؛ اختیاری)
- `LocationName` (سٹرنگ؛ اختیاری)
- `LocationAddress` (سٹرنگ؛ اختیاری)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (بولین)

## چینل نوٹس

- **Telegram**: وینیوز `LocationName/LocationAddress` سے میپ ہوتے ہیں؛ لائیو لوکیشنز `live_period` استعمال کرتی ہیں۔
- **WhatsApp**: `locationMessage.comment` اور `liveLocationMessage.caption` کیپشن سطر کے طور پر شامل کیے جاتے ہیں۔
- **Matrix**: `geo_uri` کو پن لوکیشن کے طور پر پارس کیا جاتا ہے؛ الٹی ٹیوڈ کو نظر انداز کیا جاتا ہے اور `LocationIsLive` ہمیشہ false ہوتا ہے۔
