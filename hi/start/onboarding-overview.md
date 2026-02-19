---
summary: "OpenClaw onboarding विकल्पों और प्रवाहों का अवलोकन"
read_when:
  - एक onboarding पथ चुनना
  - नया environment सेट अप करना
title: "Onboarding अवलोकन"
sidebarTitle: "Onboarding अवलोकन"
---

# Onboarding अवलोकन

OpenClaw कई onboarding पथों का समर्थन करता है, यह इस पर निर्भर करता है कि Gateway कहाँ चल रहा है
और आप providers को किस प्रकार कॉन्फ़िगर करना पसंद करते हैं।

## अपना onboarding पथ चुनें

- **CLI wizard** macOS, Linux और Windows (WSL2 के माध्यम से) के लिए।
- **macOS app** Apple silicon या Intel Macs पर मार्गदर्शित पहली रन के लिए।

## CLI onboarding wizard

टर्मिनल में wizard चलाएँ:

```bash
openclaw onboard
```

जब आप Gateway, workspace,
channels और skills पर पूर्ण नियंत्रण चाहते हैं, तब CLI wizard का उपयोग करें। Docs:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS app onboarding

जब आप macOS पर पूर्णतः मार्गदर्शित सेटअप चाहते हैं, तब OpenClaw app का उपयोग करें। Docs:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

यदि आपको ऐसा endpoint चाहिए जो सूचीबद्ध नहीं है, जिसमें वे hosted providers भी शामिल हैं
जो मानक OpenAI या Anthropic APIs प्रदान करते हैं, तो
CLI wizard में **Custom Provider** चुनें। आपसे निम्न कार्य करने के लिए कहा जाएगा:

- OpenAI-compatible, Anthropic-compatible, या **Unknown** (auto-detect) चुनें।
- एक base URL और API key दर्ज करें (यदि provider द्वारा आवश्यक हो)।
- एक model ID और वैकल्पिक alias प्रदान करें।
- एक Endpoint ID चुनें ताकि कई custom endpoints साथ में मौजूद रह सकें।

विस्तृत चरणों के लिए, ऊपर दिए गए CLI onboarding docs का पालन करें।

