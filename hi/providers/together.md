---
summary: "Together AI सेटअप (auth + मॉडल चयन)"
read_when:
  - आप OpenClaw के साथ Together AI का उपयोग करना चाहते हैं
  - आपको API key environment variable या CLI auth विकल्प की आवश्यकता है
---

# Together AI

[Together AI](https://together.ai) एकीकृत API के माध्यम से Llama, DeepSeek, Kimi और अन्य सहित अग्रणी ओपन-सोर्स मॉडलों तक पहुंच प्रदान करता है।

- प्रोवाइडर: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-संगत

## त्वरित प्रारंभ

1. API key सेट करें (सिफारिश: इसे Gateway के लिए संग्रहीत करें):

```bash
openclaw onboard --auth-choice together-api-key
```

2. एक डिफ़ॉल्ट मॉडल सेट करें:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## नॉन-इंटरैक्टिव उदाहरण

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

यह `together/moonshotai/Kimi-K2.5` को डिफ़ॉल्ट मॉडल के रूप में सेट करेगा।

## पर्यावरण नोट

यदि Gateway एक daemon (launchd/systemd) के रूप में चलता है, तो सुनिश्चित करें कि `TOGETHER_API_KEY`
उस प्रक्रिया के लिए उपलब्ध है (उदाहरण के लिए, `~/.clawdbot/.env` में या
`env.shellEnv` के माध्यम से)।

## उपलब्ध मॉडल

Together AI कई लोकप्रिय ओपन-सोर्स मॉडलों तक पहुंच प्रदान करता है:

- **GLM 4.7 Fp8** - 200K कॉन्टेक्स्ट विंडो के साथ डिफ़ॉल्ट मॉडल
- **Llama 3.3 70B Instruct Turbo** - तेज़, कुशल निर्देश पालन
- **Llama 4 Scout** - इमेज समझ के साथ विज़न मॉडल
- **Llama 4 Maverick** - उन्नत विज़न और रीजनिंग
- **DeepSeek V3.1** - शक्तिशाली कोडिंग और रीजनिंग मॉडल
- **DeepSeek R1** - उन्नत रीजनिंग मॉडल
- **Kimi K2 Instruct** - 262K कॉन्टेक्स्ट विंडो के साथ उच्च-प्रदर्शन मॉडल

सभी मॉडल मानक चैट कम्प्लीशन का समर्थन करते हैं और OpenAI API के साथ संगत हैं।

