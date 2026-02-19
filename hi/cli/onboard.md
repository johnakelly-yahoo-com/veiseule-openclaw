---
summary: "`openclaw onboard` के लिए CLI संदर्भ (इंटरैक्टिव ऑनबोर्डिंग विज़ार्ड)"
read_when:
  - आपको Gateway, वर्कस्पेस, प्रमाणीकरण, चैनल और Skills के लिए मार्गदर्शित सेटअप चाहिए
title: "onboard"
---

# `openclaw onboard`

इंटरैक्टिव ऑनबोर्डिंग विज़ार्ड (स्थानीय या दूरस्थ Gateway सेटअप)।

## संबंधित मार्गदर्शिकाएँ

- CLI ऑनबोर्डिंग हब: [Onboarding Wizard (CLI)](/start/wizard)
- CLI ऑनबोर्डिंग संदर्भ: [CLI Onboarding Reference](/start/wizard-cli-reference)
- CLI स्वचालन: [CLI Automation](/start/wizard-cli-automation)
- macOS ऑनबोर्डिंग: [Onboarding (macOS App)](/start/onboarding)
- macOS ऑनबोर्डिंग: [Onboarding (macOS App)](/start/onboarding)

## Examples

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Flow नोट्स:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` non-interactive mode में वैकल्पिक है। यदि इसे नहीं दिया गया है, तो onboarding `CUSTOM_API_KEY` की जाँच करता है।

Non-interactive Z.AI endpoint विकल्प:

नोट: `--auth-choice zai-api-key` अब आपकी key के लिए सर्वोत्तम Z.AI endpoint को स्वतः पहचानता है (सामान्य API को `zai/glm-5` के साथ प्राथमिकता देता है)।
यदि आप विशेष रूप से GLM Coding Plan endpoints चाहते हैं, तो `zai-coding-global` या `zai-coding-cn` चुनें।

```bash
# Promptless endpoint selection
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Other Z.AI endpoint choices:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow नोट्स:

- `quickstart`: न्यूनतम प्रॉम्प्ट्स, Gateway टोकन स्वतः जनरेट करता है।
- `manual`: पोर्ट/बाइंड/प्रमाणीकरण के लिए पूर्ण प्रॉम्प्ट्स ( `advanced` का उपनाम)।
- सबसे तेज़ पहली चैट: `openclaw dashboard` (कंट्रोल UI, चैनल सेटअप नहीं)।
- Custom Provider: किसी भी OpenAI या Anthropic compatible endpoint से कनेक्ट करें,
  जिसमें सूचीबद्ध न किए गए hosted providers भी शामिल हैं। स्वतः पहचान के लिए Unknown का उपयोग करें।

## Common follow-up commands

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` does not imply non-interactive mode. Use `--non-interactive` for scripts.
</Note>
