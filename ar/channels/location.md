---
summary: "تحليل موقع القنوات الواردة (Telegram وWhatsApp) وحقول السياق"
read_when:
  - إضافة أو تعديل تحليل موقع القنوات
  - استخدام حقول سياق الموقع في مطالبات الوكيل أو الأدوات
title: "تحليل موقع القنوات"
---

# تحليل موقع القنوات

يقوم OpenClaw بتوحيد المواقع المشتركة من قنوات الدردشة إلى:

- نص قابل للقراءة البشرية يُضاف إلى نص الرسالة الواردة، و
- حقول مُهيكلة ضمن حمولة سياق الردّ التلقائي.

المدعوم حاليًا:

- **Telegram** (دبابيس الموقع + الأماكن + المواقع المباشرة)
- **WhatsApp** (locationMessage + liveLocationMessage)
- **Matrix** (`m.location` مع `geo_uri`)

## تنسيق النص

وتصبح المواضع خطوطا ودية دون أقواس معقوفة:

- دبوس:
  - `📍 48.858844, 2.294351 ±12m`
- مكان مُسمّى:
  - `📍 Eiffel Tower — Champ de Mars, Paris (48.858844, 2.294351 ±12m)`
- المشاركة المباشرة:
  - `🛰 Live location: 48.858844, 2.294351 ±12m`

إذا تضمنت القناة تعليقًا/تسمية توضيحية، فيُضاف في السطر التالي:

```
📍 48.858844, 2.294351 ±12m
Meet here
```

## حقول السياق

عند وجود موقع، تُضاف هذه الحقول إلى `ctx`:

- `LocationLat` (number)
- `LocationLon` (number)
- `LocationAccuracy` (number، بالأمتار؛ اختياري)
- `LocationName` (string؛ اختياري)
- `LocationAddress` (string؛ اختياري)
- `LocationSource` (`pin | place | live`)
- `LocationIsLive` (boolean)

## ملاحظات القنوات

- **Telegram**: تُطابِق الأماكن `LocationName/LocationAddress`؛ وتستخدم المواقع المباشرة `live_period`.
- **WhatsApp**: يُضاف `locationMessage.comment` و`liveLocationMessage.caption` كسطر التسمية التوضيحية.
- **Matrix**: يُحلَّل `geo_uri` كموقع دبوس؛ يتم تجاهل الارتفاع ويكون `LocationIsLive` دائمًا false.

