---
summary: "वेक और पृथक एजेंट रन के लिए वेबहुक इनग्रेस"
read_when:
  - वेबहुक एंडपॉइंट जोड़ते या बदलते समय
  - बाहरी सिस्टमों को OpenClaw से जोड़ते समय
title: "वेबहुक्स"
---

# वेबहुक्स

Gateway बाहरी ट्रिगर्स के लिए एक छोटा HTTP वेबहुक एंडपॉइंट एक्सपोज़ कर सकता है।

## सक्षम करें

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

टिप्पणियाँ:

- `hooks.token` आवश्यक है जब `hooks.enabled=true`।
- `hooks.path` का डिफ़ॉल्ट मान `/hooks` है।

## प्रमाणीकरण

हर अनुरोध में हुक टोकन शामिल होना चाहिए। हेडर्स को प्राथमिकता दें:

- `Authorization: Bearer <token>` (अनुशंसित)
- `x-openclaw-token: <token>`
- `?token=<token>` (अप्रचलित; एक चेतावनी लॉग करता है और भविष्य के किसी प्रमुख रिलीज़ में हटा दिया जाएगा)

## एंडपॉइंट्स

### `POST /hooks/wake`

पेलोड:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **आवश्यक** (string): इवेंट का विवरण (उदाहरण: "नया ईमेल प्राप्त हुआ")।
- `mode` वैकल्पिक (`now` | `next-heartbeat`): क्या तुरंत हार्टबीट ट्रिगर करना है (डिफ़ॉल्ट `now`) या अगली आवधिक जाँच का इंतज़ार करना है।

Effect:

- **मुख्य** सत्र के लिए एक सिस्टम इवेंट कतारबद्ध करता है
- यदि `mode=now`, तो तुरंत हार्टबीट ट्रिगर करता है

### `POST /hooks/agent`

पेलोड:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **आवश्यक** (string): एजेंट द्वारा प्रोसेस किया जाने वाला प्रॉम्प्ट या संदेश।
- `name` वैकल्पिक (string): हुक के लिए मानव-पठनीय नाम (उदाहरण: "GitHub"), जिसे सत्र सारांशों में उपसर्ग के रूप में उपयोग किया जाता है।
- `agentId` वैकल्पिक (string): इस हुक को किसी विशिष्ट agent तक रूट करें। अज्ञात ID डिफ़ॉल्ट एजेंट पर वापस चले जाते हैं। सेट होने पर, हुक रेज़ॉल्व किए गए एजेंट के वर्कस्पेस और कॉन्फ़िगरेशन का उपयोग करके चलता है।
- `sessionKey` वैकल्पिक (string): एजेंट के सेशन की पहचान के लिए उपयोग की जाने वाली कुंजी। डिफ़ॉल्ट रूप से यह फ़ील्ड अस्वीकृत होती है जब तक कि `hooks.allowRequestSessionKey=true` सेट न हो।
- `deliver` वैकल्पिक (boolean): यदि `true` है, तो एजेंट का उत्तर मैसेजिंग चैनल पर भेजा जाएगा। डिफ़ॉल्ट `true` है। जो प्रतिक्रियाएँ केवल हार्टबीट स्वीकृतियाँ होती हैं, उन्हें अपने आप छोड़ दिया जाता है।
- `deliver` वैकल्पिक (boolean): यदि `true` है, तो एजेंट का उत्तर मैसेजिंग चैनल पर भेजा जाएगा। डिफ़ॉल्ट `true` है। जो प्रतिक्रियाएँ केवल हार्टबीट स्वीकृतियाँ होती हैं, उन्हें अपने आप छोड़ दिया जाता है।
- `channel` वैकल्पिक (string): डिलीवरी के लिए मैसेजिंग चैनल। इनमें से एक: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. `last` पर डिफ़ॉल्ट होता है।
- `to` वैकल्पिक (string): चैनल के लिए प्राप्तकर्ता पहचानकर्ता (उदा., WhatsApp/Signal के लिए फ़ोन नंबर, Telegram के लिए चैट ID, Discord/Slack/Mattermost (plugin) के लिए चैनल ID, MS Teams के लिए conversation ID)। मुख्य सत्र में अंतिम प्राप्तकर्ता पर डिफ़ॉल्ट होता है।
- `model` वैकल्पिक (string): मॉडल ओवरराइड (उदा., `anthropic/claude-3-5-sonnet` या कोई alias)। यदि प्रतिबंधित हो, तो इसे अनुमत मॉडल सूची में होना चाहिए।
- `timeoutSeconds` वैकल्पिक (number): एजेंट रन की अधिकतम अवधि सेकंड में।
- `timeoutSeconds` वैकल्पिक (number): एजेंट रन की अधिकतम अवधि सेकंड में।

Effect:

- एक **पृथक** एजेंट टर्न चलाता है (स्वयं की सत्र कुंजी)
- हमेशा **मुख्य** सत्र में एक सारांश पोस्ट करता है
- यदि `wakeMode=now`, तो तुरंत हार्टबीट ट्रिगर करता है

## Session key नीति (ब्रेकिंग परिवर्तन)

कस्टम hook नाम `hooks.mappings` के माध्यम से resolved किए जाते हैं (कॉन्फ़िगरेशन देखें)। एक mapping
मनमाने payloads को `wake` या `agent` actions में बदल सकती है, वैकल्पिक templates या
code transforms के साथ।

- अनुशंसित: एक स्थिर `hooks.defaultSessionKey` सेट करें और अनुरोध ओवरराइड्स बंद रखें।
- वैकल्पिक: केवल आवश्यकता होने पर अनुरोध ओवरराइड्स की अनुमति दें, और प्रीफ़िक्स को प्रतिबंधित करें।

अनुशंसित कॉन्फ़िगरेशन:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

संगतता कॉन्फ़िगरेशन (लीगेसी व्यवहार):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // दृढ़ता से अनुशंसित
  },
}
```

### `POST /hooks/<name>` (mapped)

कस्टम hook नाम `hooks.mappings` के माध्यम से resolved किए जाते हैं (कॉन्फ़िगरेशन देखें)। एक mapping
मनमाने payloads को `wake` या `agent` actions में बदल सकती है, वैकल्पिक templates या
code transforms के साथ।

Mapping options (summary):

- `hooks.presets: ["gmail"]` बिल्ट-इन Gmail मैपिंग सक्षम करता है।
- `hooks.mappings` आपको विन्यास में `match`, `action`, और टेम्पलेट्स परिभाषित करने देता है।
- `hooks.transformsDir` + `transform.module` कस्टम लॉजिक के लिए एक JS/TS मॉड्यूल लोड करता है।
  - `hooks.transformsDir` (यदि सेट किया गया है) आपके OpenClaw कॉन्फ़िगरेशन डायरेक्टरी के अंतर्गत transforms रूट के भीतर ही रहना चाहिए (आमतौर पर `~/.openclaw/hooks/transforms`)।
  - `transform.module` को प्रभावी transforms डायरेक्टरी के भीतर ही रेज़ॉल्व होना चाहिए (traversal/escape पथ अस्वीकृत हैं)।
- `match.source` का उपयोग एक सामान्य इनजेस्ट एंडपॉइंट रखने के लिए करें (payload-आधारित रूटिंग)।
- TS ट्रांसफ़ॉर्म्स के लिए TS लोडर (उदाहरण: `bun` या `tsx`) या रनटाइम पर प्रीकम्पाइल्ड `.js` आवश्यक है।
- मैपिंग्स पर `deliver: true` + `channel`/`to` सेट करें ताकि उत्तरों को किसी चैट सतह पर रूट किया जा सके
  (`channel` का डिफ़ॉल्ट `last` है और बैकअप के रूप में WhatsApp पर गिरता है)।
- `agentId` हुक को एक विशिष्ट एजेंट की ओर रूट करता है; अज्ञात ID डिफ़ॉल्ट एजेंट पर वापस चले जाते हैं।
- `hooks.allowedAgentIds` स्पष्ट `agentId` रूटिंग को प्रतिबंधित करता है। किसी भी एजेंट की अनुमति देने के लिए इसे छोड़ दें (या `*` शामिल करें)। स्पष्ट `agentId` रूटिंग को अस्वीकार करने के लिए `[]` सेट करें।
- `hooks.defaultSessionKey` तब हुक एजेंट रन के लिए डिफ़ॉल्ट सेशन सेट करता है जब कोई स्पष्ट कुंजी प्रदान नहीं की जाती।
- `hooks.allowRequestSessionKey` यह नियंत्रित करता है कि क्या `/hooks/agent` payloads `sessionKey` सेट कर सकते हैं (डिफ़ॉल्ट: `false`)।
- `hooks.allowedSessionKeyPrefixes` वैकल्पिक रूप से अनुरोध payloads और mappings से स्पष्ट `sessionKey` मानों को प्रतिबंधित करता है।
- `allowUnsafeExternalContent: true` उस हुक के लिए बाहरी सामग्री सुरक्षा रैपर को अक्षम करता है
  (खतरनाक; केवल विश्वसनीय आंतरिक स्रोतों के लिए)।
- `openclaw webhooks gmail setup`, `openclaw webhooks gmail run` के लिए `hooks.gmail` कॉन्फ़िग लिखता है।
  पूर्ण Gmail watch flow के लिए [Gmail Pub/Sub](/automation/gmail-pubsub) देखें।

## Responses

- `/hooks/wake` के लिए `200`
- `/hooks/agent` के लिए `202` (async रन शुरू हुआ)
- प्रमाणीकरण विफलता पर `401`
- उसी क्लाइंट से बार-बार auth विफलताओं के बाद `429` ( `Retry-After` जांचें )
- अमान्य payload पर `400`
- अत्यधिक बड़े payloads पर `413`

## Examples

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Use a different model

उस रन के लिए मॉडल ओवरराइड करने हेतु एजेंट payload (या मैपिंग) में `model` जोड़ें:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

यदि आप `agents.defaults.models` लागू करते हैं, तो सुनिश्चित करें कि ओवरराइड मॉडल उसमें शामिल है।

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Security

- हुक एंडपॉइंट्स को loopback, tailnet, या विश्वसनीय रिवर्स प्रॉक्सी के पीछे रखें।
- एक समर्पित हुक टोकन का उपयोग करें; Gateway प्रमाणीकरण टोकनों का पुनः उपयोग न करें।
- बार-बार auth विफलताओं को brute-force प्रयासों को धीमा करने के लिए प्रति क्लाइंट पते पर rate-limit किया जाता है।
- यदि आप multi-agent routing का उपयोग करते हैं, तो स्पष्ट `agentId` चयन को सीमित करने के लिए `hooks.allowedAgentIds` सेट करें।
- जब तक आपको caller-चयनित सेशन की आवश्यकता न हो, `hooks.allowRequestSessionKey=false` रखें।
- यदि आप अनुरोध `sessionKey` सक्षम करते हैं, तो `hooks.allowedSessionKeyPrefixes` को प्रतिबंधित करें (उदाहरण के लिए, `["hook:"]`)।
- वेबहुक लॉग्स में संवेदनशील कच्चे payloads शामिल करने से बचें।
- Hook payloads को डिफ़ॉल्ट रूप से अविश्वसनीय माना जाता है और सुरक्षा सीमाओं के साथ wrap किया जाता है।
  यदि किसी विशेष hook के लिए इसे अक्षम करना आवश्यक हो, तो उस hook की mapping में
  `allowUnsafeExternalContent: true`
  सेट करें (खतरनाक)।

