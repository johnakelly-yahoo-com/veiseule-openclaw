---
summary: "Zalo Personal प्लगइन: zca-cli के माध्यम से QR लॉगिन + मैसेजिंग (प्लगइन इंस्टॉल + चैनल विन्यास + CLI + टूल)"
read_when:
  - आप OpenClaw में Zalo Personal (अनौपचारिक) समर्थन चाहते हैं
  - आप zalouser प्लगइन को विन्यस्त या विकसित कर रहे हैं
title: "Zalo Personal प्लगइन"
---

# Zalo Personal (प्लगइन)

`zca-cli` का उपयोग करते हुए, एक सामान्य Zalo उपयोगकर्ता खाते को स्वचालित करने के लिए प्लगइन के माध्यम से OpenClaw के लिए Zalo Personal समर्थन।

> **चेतावनी:** अनौपचारिक ऑटोमेशन के कारण खाता निलंबित/प्रतिबंधित किया जा सकता है। अपने जोखिम पर उपयोग करें।

## नामकरण

Channel id `zalouser` है ताकि यह स्पष्ट हो कि यह एक **व्यक्तिगत Zalo उपयोगकर्ता खाते** (अनौपचारिक) को स्वचालित करता है। संभावित भविष्य के आधिकारिक Zalo API इंटीग्रेशन के लिए हम `zalo` को सुरक्षित रखते हैं।

## यह कहाँ चलता है

यह प्लगइन **Gateway (गेटवे) प्रक्रिया के भीतर** चलता है।

यदि आप एक दूरस्थ Gateway का उपयोग करते हैं, तो इसे **Gateway चलाने वाली मशीन** पर इंस्टॉल/विन्यस्त करें, फिर Gateway को पुनः आरंभ करें।

## इंस्टॉल करें

### Option A: npm से इंस्टॉल करें

```bash
openclaw plugins install @openclaw/zalouser
```

इसके बाद Gateway को पुनः आरंभ करें।

### Option B: स्थानीय फ़ोल्डर से इंस्टॉल करें (dev)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

इसके बाद Gateway को पुनः आरंभ करें।

## पूर्वापेक्षा: zca-cli

Gateway मशीन पर `zca` का `PATH` पर होना आवश्यक है:

```bash
zca --version
```

## कॉन्फ़िगरेशन

चैनल विन्यास `channels.zalouser` के अंतर्गत रहता है (`plugins.entries.*` के अंतर्गत नहीं):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## एजेंट टूल

टूल नाम: `zalouser`

क्रियाएँ: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`
