---
summary: "Gateway، نوڈز، اور کینوس ہوسٹ کس طرح جڑتے ہیں۔"
read_when:
  - آپ Gateway کے نیٹ ورکنگ ماڈل کا مختصر منظر چاہتے ہیں
title: "نیٹ ورک ماڈل"
---

زیادہ تر آپریشنز Gateway (`openclaw gateway`) کے ذریعے گزرتے ہیں، جو ایک واحد طویل عرصے تک چلنے والا
عمل ہے جو چینل کنکشنز اور WebSocket کنٹرول پلین کی ملکیت رکھتا ہے۔

## بنیادی اصول

- 44. فی ہوسٹ ایک گیٹ وے کی سفارش کی جاتی ہے۔ 45. WhatsApp Web سیشن کا مالک ہونے کی اجازت صرف اسی پروسس کو ہے۔ 46. ریسکیو بوٹس یا سخت آئسولیشن کے لیے، آئسولیٹڈ پروفائلز اور پورٹس کے ساتھ متعدد گیٹ ویز چلائیں۔ 47. دیکھیں [Multiple gateways](/gateway/multiple-gateways)۔
- 48. پہلے لوپ بیک: گیٹ وے WS کا ڈیفالٹ `ws://127.0.0.1:18789` ہے۔ 49. وزرڈ بطورِ ڈیفالٹ ایک گیٹ وے ٹوکن بناتا ہے، حتیٰ کہ لوپ بیک کے لیے بھی۔ 50. ٹیل نیٹ ایکسس کے لیے، `openclaw gateway --bind tailnet --token ...` چلائیں کیونکہ نان-لوپ بیک بائنڈز کے لیے ٹوکنز درکار ہوتے ہیں۔
- Nodes connect to the Gateway WS over LAN, tailnet, or SSH as needed. The legacy TCP bridge is deprecated.
- Canvas host کو Gateway HTTP سرور کے ذریعے **اسی پورٹ** پر فراہم کیا جاتا ہے جس پر Gateway چل رہا ہوتا ہے (ڈیفالٹ `18789`):
  - `/__openclaw__/canvas/`
  - `/__openclaw__/a2ui/`
    جب `gateway.auth` کنفیگر کیا گیا ہو اور Gateway لوپ بیک سے آگے بائنڈ ہو، تو یہ روٹس Gateway auth کے ذریعے محفوظ ہوتے ہیں (لوپ بیک ریکویسٹس مستثنیٰ ہیں)۔ [Gateway configuration](/gateway/configuration) (`canvasHost`, `gateway`) دیکھیں۔
- Remote use is typically SSH tunnel or tailnet VPN. See [Remote access](/gateway/remote) and [Discovery](/gateway/discovery).
