---
title: "نیٹ ورک"
---

# نیٹ ورک ہب

یہ ہب اُن بنیادی دستاویزات کو جوڑتا ہے جو بتاتی ہیں کہ OpenClaw
localhost، LAN، اور tailnet کے ذریعے ڈیوائسز کو کیسے کنیکٹ کرتا ہے، جوڑی بناتا ہے، اور محفوظ رکھتا ہے۔

## بنیادی ماڈل

- [Gateway کا فنِ تعمیر](/concepts/architecture)
- [Gateway پروٹوکول](/gateway/protocol)
- [Gateway رن بک](/gateway)
- [ویب سرفیسز + بائنڈ موڈز](/web)

## pairing + شناخت

- [پیئرنگ کا جائزہ (DM + نوڈز)](/channels/pairing)
- [Gateway کی ملکیت والے نوڈ کی پیئرنگ](/gateway/pairing)
- [Devices CLI (پیئرنگ + ٹوکن روٹیشن)](/cli/devices)
- [Pairing CLI (DM منظوری)](/cli/pairing)

مقامی اعتماد:

- مقامی کنیکشنز (loopback یا گیٹ وے ہوسٹ کا اپنا tailnet پتہ) کو
  pairing کے لیے خودکار طور پر منظور کیا جا سکتا ہے تاکہ ایک ہی ہوسٹ پر UX ہموار رہے۔
- غیر مقامی tailnet/LAN کلائنٹس کے لیے اب بھی صریح pairing منظوری درکار ہوتی ہے۔

## ڈسکوری + ٹرانسپورٹس

- [دریافت اور ٹرانسپورٹس](/gateway/discovery)
- [Bonjour / mDNS](/gateway/bonjour)
- [Remote access (SSH)](/gateway/remote)
- [Tailscale](/gateway/tailscale)

## نوڈز + ٹرانسپورٹس

- [Nodes overview](/nodes)
- [Bridge protocol (legacy nodes)](/gateway/bridge-protocol)
- [Node runbook: iOS](/platforms/ios)
- [Node runbook: Android](/platforms/android)

## سکیورٹی

- [Security overview](/gateway/security)
- [Gateway config reference](/gateway/configuration)
- [Troubleshooting](/gateway/troubleshooting)
- [Doctor](/gateway/doctor)

