---
title: "OpenProse"
---

# OpenProse

OpenProse एक पोर्टेबल, markdown-first वर्कफ़्लो फ़ॉर्मेट है जो AI सत्रों के ऑर्केस्ट्रेशन के लिए उपयोग किया जाता है। OpenClaw में यह एक प्लगइन के रूप में उपलब्ध है, जो OpenProse skill pack के साथ `/prose` स्लैश कमांड इंस्टॉल करता है। प्रोग्राम `.prose` फ़ाइलों में होते हैं और स्पष्ट नियंत्रण प्रवाह के साथ कई sub-agents को प्रारंभ कर सकते हैं।

आधिकारिक साइट: [https://www.prose.md](https://www.prose.md)

## यह क्या कर सकता है

- स्पष्ट समानांतरता के साथ बहु-एजेंट अनुसंधान + संश्लेषण।
- दोहराने योग्य, अनुमोदन-सुरक्षित वर्कफ़्लो (कोड समीक्षा, घटना ट्रायेज, सामग्री पाइपलाइंस)।
- पुन: उपयोग योग्य `.prose` प्रोग्राम जिन्हें आप समर्थित एजेंट रनटाइम्स में चला सकते हैं।

## इंस्टॉल + सक्षम करें

Bundled प्लगइन्स डिफ़ॉल्ट रूप से अक्षम होते हैं। OpenProse सक्षम करें:

```bash
openclaw plugins enable open-prose
```

प्लगइन सक्षम करने के बाद Gateway को पुनः प्रारंभ करें।

डेव/लोकल चेकआउट: `openclaw plugins install ./extensions/open-prose`

संबंधित दस्तावेज़: [Plugins](/tools/plugin), [Plugin manifest](/plugins/manifest), [Skills](/tools/skills).

## स्लैश कमांड

OpenProse `/prose` को उपयोगकर्ता द्वारा चलाए जाने योग्य skill कमांड के रूप में पंजीकृत करता है। यह OpenProse VM निर्देशों की ओर रूट करता है और अंदरूनी रूप से OpenClaw टूल्स का उपयोग करता है।

सामान्य कमांड:

```
/prose help
/prose run <file.prose>
/prose run <handle/slug>
/prose run <https://example.com/file.prose>
/prose compile <file.prose>
/prose examples
/prose update
```

## उदाहरण: एक सरल `.prose` फ़ाइल

```prose
# Research + synthesis with two agents running in parallel.

input topic: "What should we research?"

agent researcher:
  model: sonnet
  prompt: "You research thoroughly and cite sources."

agent writer:
  model: opus
  prompt: "You write a concise summary."

parallel:
  findings = session: researcher
    prompt: "Research {topic}."
  draft = session: writer
    prompt: "Summarize {topic}."

session "Merge the findings + draft into a final answer."
context: { findings, draft }
```

## फ़ाइल स्थान

OpenProse आपके वर्कस्पेस में `.prose/` के अंतर्गत स्टेट रखता है:

```
.prose/
├── .env
├── runs/
│   └── {YYYYMMDD}-{HHMMSS}-{random}/
│       ├── program.prose
│       ├── state.md
│       ├── bindings/
│       └── agents/
└── agents/
```

उपयोगकर्ता-स्तर के स्थायी एजेंट यहाँ रहते हैं:

```
~/.prose/agents/
```

## स्टेट मोड्स

OpenProse कई स्टेट बैकएंड्स का समर्थन करता है:

- **filesystem** (डिफ़ॉल्ट): `.prose/runs/...`
- **in-context**: क्षणिक, छोटे प्रोग्रामों के लिए
- **sqlite** (प्रायोगिक): `sqlite3` बाइनरी की आवश्यकता
- **postgres** (प्रायोगिक): `psql` और एक कनेक्शन स्ट्रिंग की आवश्यकता

टिप्पणियाँ:

- sqlite/postgres वैकल्पिक हैं और प्रायोगिक हैं।
- postgres क्रेडेंशियल्स उप-एजेंट लॉग्स में प्रवाहित होते हैं; एक समर्पित, न्यूनतम-अधिकार DB का उपयोग करें।

## रिमोट प्रोग्राम

`/prose run <handle/slug>` का समाधान `https://p.prose.md/<handle>/<slug>` पर होता है।
Direct URLs are fetched as-is. This uses the `web_fetch` tool (or `exec` for POST).

## OpenClaw रनटाइम मैपिंग

OpenProse प्रोग्राम OpenClaw प्रिमिटिव्स से मैप होते हैं:

| OpenProse अवधारणा         | OpenClaw टूल     |
| ------------------------- | ---------------- |
| सत्र प्रारंभ करें / Task टूल | `sessions_spawn` |
| फ़ाइल पढ़ना/लिखना         | `read` / `write` |
| वेब फ़ेच                  | `web_fetch`      |

यदि आपकी टूल allowlist इन टूल्स को ब्लॉक करती है, तो OpenProse प्रोग्राम विफल हो जाएंगे। अधिक जानकारी के लिए [Skills config](/tools/skills-config) देखें।

## सुरक्षा + अनुमोदन

`.prose` फ़ाइलों को कोड की तरह मानें। चलाने से पहले समीक्षा करें। साइड इफ़ेक्ट्स को नियंत्रित करने के लिए OpenClaw टूल allowlists और approval gates का उपयोग करें।

निर्धारक, अनुमोदन-गेटेड वर्कफ़्लो के लिए, [Lobster](/tools/lobster) से तुलना करें।

