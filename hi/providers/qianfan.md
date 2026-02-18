---
title: "Qianfan"
---

# Qianfan प्रदाता मार्गदर्शिका

Qianfan, Baidu का MaaS प्लेटफ़ॉर्म है, जो एक **एकीकृत API** प्रदान करता है जो अनुरोधों को एक ही के पीछे कई मॉडलों तक रूट करता है
endpoint and API key. It is OpenAI-compatible, so most OpenAI SDKs work by switching the base URL.

## पूर्वापेक्षाएँ

1. Qianfan API एक्सेस के साथ एक Baidu Cloud खाता
2. Qianfan कंसोल से एक API कुंजी
3. आपके सिस्टम पर OpenClaw स्थापित होना

## अपनी API कुंजी प्राप्त करना

1. [Qianfan कंसोल](https://console.bce.baidu.com/qianfan/ais/console/apiKey) पर जाएँ
2. एक नया एप्लिकेशन बनाएँ या किसी मौजूदा का चयन करें
3. एक API कुंजी जनरेट करें (फ़ॉर्मेट: `bce-v3/ALTAK-...`)
4. OpenClaw के साथ उपयोग के लिए API कुंजी कॉपी करें

## CLI सेटअप

```bash
openclaw onboard --auth-choice qianfan-api-key
```

## संबंधित दस्तावेज़

- [OpenClaw विन्यास](/gateway/configuration)
- [मॉडल प्रदाता](/concepts/model-providers)
- [एजेंट सेटअप](/concepts/agent)
- [Qianfan API दस्तावेज़](/https://cloud.baidu.com/doc/qianfan-api/s/3m7of64lb)
