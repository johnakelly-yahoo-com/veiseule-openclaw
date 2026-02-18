---
title: "doctor"
---

# `openclaw doctor`

Gateway और चैनलों के लिए स्वास्थ्य जाँच + त्वरित सुधार।

संबंधित:

- समस्या-निवारण: [Troubleshooting](/gateway/troubleshooting)
- सुरक्षा ऑडिट: [Security](/gateway/security)

## उदाहरण

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

नोट्स:

- इंटरैक्टिव प्रॉम्प्ट्स (जैसे keychain/OAuth फिक्स) केवल तभी चलते हैं जब stdin एक TTY हो और `--non-interactive` **सेट न** किया गया हो। हेडलेस रन (cron, Telegram, बिना टर्मिनल) प्रॉम्प्ट्स को छोड़ देंगे।
- `--fix` (`--repair` का उपनाम) `~/.openclaw/openclaw.json.bak` में एक बैकअप लिखता है और अज्ञात विन्यास कुंजियों को हटा देता है, प्रत्येक हटाने को सूचीबद्ध करते हुए।

## macOS: `launchctl` के env ओवरराइड्स

यदि आपने पहले `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (या `...PASSWORD`) चलाया था, तो वह मान आपकी विन्यास फ़ाइल को ओवरराइड करता है और लगातार “unauthorized” त्रुटियों का कारण बन सकता है।

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```
