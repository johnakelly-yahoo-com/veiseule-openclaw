---
title: "विन्यास"
---

# विन्यास

OpenClaw `~/.openclaw/openclaw.json` से एक वैकल्पिक <Tooltip tip="JSON5 supports comments and trailing commas">**JSON5**</Tooltip> विन्यास पढ़ता है।

यदि फ़ाइल मौजूद नहीं है, तो OpenClaw सुरक्षित डिफ़ॉल्ट्स का उपयोग करता है। कॉन्फ़िग जोड़ने के सामान्य कारण:

- चैनल कनेक्ट करना और यह नियंत्रित करना कि कौन बॉट को संदेश भेज सकता है  
- मॉडल, टूल्स, सैंडबॉक्सिंग, या ऑटोमेशन (cron, hooks) सेट करना  
- सेशंस, मीडिया, नेटवर्किंग, या UI को ट्यून करना  

उपलब्ध सभी फ़ील्ड्स के लिए [पूर्ण संदर्भ](/gateway/configuration-reference) देखें।

<Tip>
**विन्यास में नए हैं?** इंटरैक्टिव सेटअप के लिए `openclaw onboard` से शुरू करें, या पूर्ण कॉपी‑पेस्ट कॉन्फ़िग्स के लिए [Configuration Examples](/gateway/configuration-examples) गाइड देखें।
</Tip>

## न्यूनतम विन्यास

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## कॉन्फ़िग संपादन

<Tabs>
  <Tab title="इंटरैक्टिव विज़ार्ड">
    ```bash
    openclaw onboard       # पूरा सेटअप विज़ार्ड
    openclaw configure     # कॉन्फ़िग विज़ार्ड
    ```
  </Tab>
  <Tab title="CLI (वन-लाइनर्स)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) खोलें और **Config** टैब का उपयोग करें।  
    Control UI कॉन्फ़िग स्कीमा से एक फ़ॉर्म रेंडर करता है, और आपात स्थिति के लिए **Raw JSON** एडिटर भी देता है।
  </Tab>
  <Tab title="सीधा संपादन">
    `~/.openclaw/openclaw.json` को सीधे संपादित करें। Gateway फ़ाइल को मॉनिटर करता है और बदलाव अपने आप लागू करता है (देखें [hot reload](#config-hot-reload))।
  </Tab>
</Tabs>

## सख्त सत्यापन

<Warning>
OpenClaw केवल वही कॉन्फ़िगरेशन स्वीकार करता है जो स्कीमा से पूरी तरह मेल खाते हों। अज्ञात keys, गलत प्रकार, या अमान्य मान होने पर Gateway **स्टार्ट होने से मना कर देता है**। रूट-लेवल पर एकमात्र अपवाद `$schema` (string) है, ताकि एडिटर्स JSON Schema मेटाडेटा संलग्न कर सकें।
</Warning>

जब सत्यापन विफल होता है:

- Gateway बूट नहीं होता  
- केवल डायग्नोस्टिक कमांड्स काम करते हैं (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)  
- सटीक समस्याएँ देखने के लिए `openclaw doctor` चलाएँ  
- मरम्मत लागू करने के लिए `openclaw doctor --fix` (या `--yes`) चलाएँ  

## सामान्य कार्य

<AccordionGroup>
  <Accordion title="एक चैनल सेट करें (WhatsApp, Telegram, Discord, आदि)">
    प्रत्येक चैनल का अपना कॉन्फ़िग सेक्शन `channels.<provider>` के अंतर्गत होता है। सेटअप स्टेप्स के लिए संबंधित चैनल पेज देखें:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    सभी चैनल एक समान DM पॉलिसी पैटर्न साझा करते हैं:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="मॉडल चुनें और कॉन्फ़िगर करें">
    प्राइमरी मॉडल और वैकल्पिक फॉलबैक्स सेट करें:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` मॉडल कैटलॉग परिभाषित करता है और `/model` के लिए allowlist के रूप में कार्य करता है।  
    - मॉडल रेफ़ `provider/model` प्रारूप का उपयोग करते हैं (उदा. `anthropic/claude-opus-4-6`)।  
    - चैट में मॉडल बदलने के लिए [Models CLI](/concepts/models) और ऑथ रोटेशन/फॉलबैक व्यवहार के लिए [Model Failover](/concepts/model-failover) देखें।  
    - कस्टम/सेल्फ-होस्टेड प्रोवाइडर्स के लिए संदर्भ में [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) देखें।  

  </Accordion>

  <Accordion title="कौन बॉट को संदेश भेज सकता है, नियंत्रित करें">
    DM एक्सेस प्रति चैनल `dmPolicy` के माध्यम से नियंत्रित होता है:

    - `"pairing"` (डिफ़ॉल्ट): अज्ञात प्रेषकों को अनुमोदन के लिए एक-बार का पेयरिंग कोड मिलता है  
    - `"allowlist"`: केवल `allowFrom` (या पेयर्ड allow स्टोर) में मौजूद प्रेषक  
    - `"open"`: सभी इनबाउंड DMs की अनुमति (आवश्यक: `allowFrom: ["*"]`)  
    - `"disabled"`: सभी DMs को अनदेखा करें  

    ग्रुप्स के लिए `groupPolicy` + `groupAllowFrom` या चैनल-विशिष्ट allowlists का उपयोग करें।

    विस्तृत जानकारी के लिए [पूर्ण संदर्भ](/gateway/configuration-reference#dm-and-group-access) देखें।

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway `~/.openclaw/openclaw.json` को मॉनिटर करता है और अधिकतर सेटिंग्स के लिए बिना मैनुअल रीस्टार्ट के बदलाव लागू करता है।

### Reload modes

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (default) | सुरक्षित बदलाव तुरंत लागू करता है; क्रिटिकल बदलाव पर स्वतः रीस्टार्ट।              |
| **`hot`**              | केवल सुरक्षित बदलाव लागू करता है; रीस्टार्ट की आवश्यकता होने पर चेतावनी देता है।   |
| **`restart`**          | किसी भी कॉन्फ़िग बदलाव पर Gateway रीस्टार्ट करता है।                                 |
| **`off`**              | फ़ाइल मॉनिटरिंग अक्षम; बदलाव अगली मैनुअल रीस्टार्ट पर लागू होंगे।                  |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

<Note>
`gateway.reload` और `gateway.remote` में बदलाव **रीस्टार्ट ट्रिगर नहीं करते**।
</Note>

## पूर्ण संदर्भ

फ़ील्ड-दर-फ़ील्ड पूर्ण विवरण के लिए देखें **[Configuration Reference](/gateway/configuration-reference)**।

---

_संबंधित: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_

