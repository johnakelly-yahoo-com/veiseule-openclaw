---
summary: "CLI کے لیے حوالہ برائے `openclaw voicecall` (voice-call پلگ اِن کی کمانڈ سطح)"
read_when:
  - آپ voice-call پلگ اِن استعمال کرتے ہیں اور CLI کے اندراجی پوائنٹس چاہتے ہیں
  - آپ `voicecall call|continue|status|tail|expose` کے لیے فوری مثالیں چاہتے ہیں
title: "voicecall"
---

# `openclaw voicecall`

`voicecall` ایک plugin کی طرف سے فراہم کردہ کمانڈ ہے۔ یہ صرف اسی صورت میں ظاہر ہوتی ہے جب voice-call plugin انسٹال اور فعال ہو۔

بنیادی دستاویز:

- voice-call پلگ اِن: [وائس کال](/plugins/voice-call)

## عام کمانڈز

```bash
openclaw voicecall status --call-id <id>
openclaw voicecall call --to "+15555550123" --message "Hello" --mode notify
openclaw voicecall continue --call-id <id> --message "Any questions?"
openclaw voicecall end --call-id <id>
```

## ویب ہکس کو ایکسپوز کرنا (Tailscale)

```bash
openclaw voicecall expose --mode serve
openclaw voicecall expose --mode funnel
openclaw voicecall unexpose
```

سیکیورٹی نوٹ: webhook endpoint کو صرف اُن نیٹ ورکس تک محدود رکھیں جن پر آپ کو اعتماد ہو۔ جہاں ممکن ہو، Funnel کے بجائے Tailscale Serve کو ترجیح دیں۔
