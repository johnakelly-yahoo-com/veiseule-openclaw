---
title: "Signal"
---

# Signal (signal-cli)

स्थिति: बाहरी CLI इंटीग्रेशन। Gateway `signal-cli` से HTTP JSON-RPC + SSE के माध्यम से बात करता है।

## त्वरित सेटअप (शुरुआती)

1. बॉट के लिए **एक अलग Signal नंबर** उपयोग करें (अनुशंसित)।
2. `signal-cli` इंस्टॉल करें (Java आवश्यक)।
3. बॉट डिवाइस लिंक करें और डेमन शुरू करें:
   - `signal-cli link -n "OpenClaw"`
4. OpenClaw को विन्यस्त करें और Gateway शुरू करें।

न्यूनतम विन्यास:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## यह क्या है

- `signal-cli` के माध्यम से Signal चैनल (एंबेडेड libsignal नहीं)।
- निर्धारक रूटिंग: उत्तर हमेशा Signal पर ही वापस जाते हैं।
- DMs एजेंट के मुख्य सत्र को साझा करते हैं; समूह अलग-थलग होते हैं (`agent:<agentId>:signal:group:<groupId>`)।

## Config writes

डिफ़ॉल्ट रूप से, Signal को `/config set|unset` द्वारा ट्रिगर किए गए config अपडेट लिखने की अनुमति है (इसके लिए `commands.config: true` आवश्यक है)।

इसे अक्षम करने के लिए:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## नंबर मॉडल (महत्वपूर्ण)

- Gateway एक **Signal डिवाइस** ( `signal-cli` खाता) से जुड़ता है।
- यदि आप बॉट को **अपने व्यक्तिगत Signal खाते** पर चलाते हैं, तो यह आपके अपने संदेशों को अनदेखा करेगा (लूप सुरक्षा)।
- “मैं बॉट को टेक्स्ट करता हूँ और वह जवाब देता है” के लिए **अलग बॉट नंबर** का उपयोग करें।

## सेटअप (त्वरित मार्ग)

1. `signal-cli` इंस्टॉल करें (Java आवश्यक)।
2. एक बॉट खाता लिंक करें:
   - `signal-cli link -n "OpenClaw"` फिर Signal में QR स्कैन करें।
3. Signal को विन्यस्त करें और Gateway शुरू करें।

उदाहरण:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

मल्टी-अकाउंट सपोर्ट: प्रति-अकाउंट कॉन्फ़िग और वैकल्पिक `name` के साथ `channels.signal.accounts` का उपयोग करें। साझा पैटर्न के लिए [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) देखें।

## बाहरी डेमन मोड (httpUrl)

यदि आप `signal-cli` को स्वयं प्रबंधित करना चाहते हैं (धीमे JVM कोल्ड स्टार्ट, कंटेनर इनिट, या साझा CPU), तो डेमन को अलग से चलाएँ और OpenClaw को उसकी ओर इंगित करें:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

यह OpenClaw के अंदर ऑटो-स्पॉन और स्टार्टअप वेट को छोड़ देता है। ऑटो-स्पॉन के दौरान धीमी शुरुआत के लिए `channels.signal.startupTimeoutMs` सेट करें।

## प्रवेश नियंत्रण (DMs + समूह)

DMs:

- डिफ़ॉल्ट: `channels.signal.dmPolicy = "pairing"`।
- अज्ञात प्रेषकों को एक पेयरिंग कोड मिलता है; स्वीकृति तक संदेश अनदेखे रहते हैं (कोड 1 घंटे बाद समाप्त हो जाते हैं)।
- स्वीकृति के लिए:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Signal DMs के लिए पेयरिंग डिफ़ॉल्ट टोकन एक्सचेंज है। विवरण: [Pairing](/channels/pairing)
- UUID-केवल प्रेषक (`sourceUuid` से) को `channels.signal.allowFrom` में `uuid:<id>` के रूप में संग्रहीत किया जाता है।

समूह:

- `channels.signal.groupPolicy = open | allowlist | disabled`।
- `channels.signal.groupAllowFrom` यह नियंत्रित करता है कि समूहों में कौन ट्रिगर कर सकता है जब `allowlist` सेट हो।

## यह कैसे काम करता है (व्यवहार)

- `signal-cli` एक डेमन के रूप में चलता है; Gateway SSE के माध्यम से इवेंट पढ़ता है।
- इनबाउंड संदेशों को साझा चैनल एनवेलप में सामान्यीकृत किया जाता है।
- उत्तर हमेशा उसी नंबर या समूह पर रूट होते हैं।

## मीडिया + सीमाएँ

- आउटबाउंड टेक्स्ट को `channels.signal.textChunkLimit` तक चंक्स में विभाजित किया जाता है (डिफ़ॉल्ट 4000)।
- वैकल्पिक न्यूलाइन चंकिंग: लंबाई चंकिंग से पहले खाली पंक्तियों (अनुच्छेद सीमाएँ) पर विभाजित करने के लिए `channels.signal.chunkMode="newline"` सेट करें।
- अटैचमेंट समर्थित (base64 `signal-cli` से फ़ेच किया जाता है)।
- डिफ़ॉल्ट मीडिया सीमा: `channels.signal.mediaMaxMb` (डिफ़ॉल्ट 8)।
- मीडिया डाउनलोड छोड़ने के लिए `channels.signal.ignoreAttachments` का उपयोग करें।
- ग्रुप इतिहास संदर्भ `channels.signal.historyLimit` (या `channels.signal.accounts.*.historyLimit`) का उपयोग करता है, और `messages.groupChat.historyLimit` पर फ़ॉलबैक करता है। अक्षम करने के लिए `0` सेट करें (डिफ़ॉल्ट 50)।

## टाइपिंग + रीड रसीदें

- **टाइपिंग संकेतक**: OpenClaw `signal-cli sendTyping` के माध्यम से टाइपिंग संकेत भेजता है और उत्तर चलते समय उन्हें रिफ़्रेश करता है।
- **रीड रसीदें**: जब `channels.signal.sendReadReceipts` true हो, OpenClaw अनुमत DMs के लिए रीड रसीदें फ़ॉरवर्ड करता है।
- Signal-cli समूहों के लिए रीड रसीदें उपलब्ध नहीं कराता।

## रिएक्शन्स (message tool)

- `message action=react` का उपयोग `channel=signal` के साथ करें।
- लक्ष्य: प्रेषक E.164 या UUID (पेयरिंग आउटपुट से `uuid:<id>` उपयोग करें; साधारण UUID भी काम करता है)।
- `messageId` उस संदेश का Signal टाइमस्टैम्प है जिस पर आप प्रतिक्रिया दे रहे हैं।
- समूह रिएक्शन्स के लिए `targetAuthor` या `targetAuthorUuid` आवश्यक है।

उदाहरण:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

विन्यास:

- `channels.signal.actions.reactions`: रिएक्शन क्रियाएँ सक्षम/अक्षम करें (डिफ़ॉल्ट true)।
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`।
  - `off`/`ack` एजेंट रिएक्शन्स को अक्षम करता है (message tool `react` त्रुटि देगा)।
  - `minimal`/`extensive` एजेंट रिएक्शन्स सक्षम करता है और मार्गदर्शन स्तर सेट करता है।
- प्रति-अकाउंट ओवरराइड्स: `channels.signal.accounts.<id>`1. .actions.reactions`, `channels.signal.accounts.<id>2. .reactionLevel\`.

## डिलीवरी लक्ष्य (CLI/cron)

- DMs: `signal:+15551234567` (या साधारण E.164)।
- UUID DMs: `uuid:<id>` (या साधारण UUID)।
- समूह: `signal:group:<groupId>`।
- उपयोगकर्ता नाम: `username:<name>` (यदि आपके Signal खाते द्वारा समर्थित हो)।

## समस्या-निवारण

पहले यह सीढ़ी चलाएँ:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

फिर आवश्यकता होने पर DM पेयरिंग स्थिति की पुष्टि करें:

```bash
openclaw pairing list signal
```

सामान्य विफलताएँ:

- डेमन पहुँचे योग्य है लेकिन उत्तर नहीं: खाता/डेमन सेटिंग्स (`httpUrl`, `account`) और receive मोड सत्यापित करें।
- DMs अनदेखे: प्रेषक पेयरिंग स्वीकृति की प्रतीक्षा में है।
- समूह संदेश अनदेखे: समूह प्रेषक/मेंशन गेटिंग डिलीवरी को रोक रही है।

ट्रायेज फ़्लो के लिए: [/channels/troubleshooting](/channels/troubleshooting)।

## विन्यास संदर्भ (Signal)

पूर्ण विन्यास: [Configuration](/gateway/configuration)

प्रदाता विकल्प:

- `channels.signal.enabled`: चैनल स्टार्टअप सक्षम/अक्षम करें।
- `channels.signal.account`: बॉट खाते के लिए E.164।
- `channels.signal.cliPath`: `signal-cli` का पाथ।
- `channels.signal.httpUrl`: पूर्ण डेमन URL (host/port को ओवरराइड करता है)।
- `channels.signal.httpHost`, `channels.signal.httpPort`: डेमन बाइंड (डिफ़ॉल्ट 127.0.0.1:8080)।
- `channels.signal.autoStart`: ऑटो-स्पॉन डेमन (डिफ़ॉल्ट true यदि `httpUrl` अनसेट हो)।
- `channels.signal.startupTimeoutMs`: स्टार्टअप वेट टाइमआउट (ms में, अधिकतम 120000)।
- `channels.signal.receiveMode`: `on-start | manual`।
- `channels.signal.ignoreAttachments`: अटैचमेंट डाउनलोड छोड़ें।
- `channels.signal.ignoreStories`: डेमन से स्टोरीज़ अनदेखी करें।
- `channels.signal.sendReadReceipts`: रीड रसीदें फ़ॉरवर्ड करें।
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (डिफ़ॉल्ट: पेयरिंग)।
- 3. `channels.signal.allowFrom`: DM अनुमति सूची (E.164 या `uuid:<id>`). 4. `open` के लिए `"*"` आवश्यक है। 5. Signal में उपयोगकर्ता नाम नहीं होते; फोन/UUID आईडी का उपयोग करें।
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (डिफ़ॉल्ट: allowlist)।
- `channels.signal.groupAllowFrom`: समूह प्रेषक allowlist।
- `channels.signal.historyLimit`: संदर्भ के रूप में शामिल करने के लिए अधिकतम समूह संदेश (0 अक्षम करता है)।
- 6. `channels.signal.dmHistoryLimit`: उपयोगकर्ता टर्न में DM इतिहास सीमा। 7. प्रति-उपयोगकर्ता ओवरराइड्स: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: आउटबाउंड चंक आकार (अक्षर)।
- `channels.signal.chunkMode`: `length` (डिफ़ॉल्ट) या `newline` ताकि लंबाई चंकिंग से पहले खाली पंक्तियों (अनुच्छेद सीमाएँ) पर विभाजन हो।
- `channels.signal.mediaMaxMb`: इनबाउंड/आउटबाउंड मीडिया सीमा (MB)।

संबंधित वैश्विक विकल्प:

- `agents.list[].groupChat.mentionPatterns` (Signal मूल मेंशन का समर्थन नहीं करता)।
- `messages.groupChat.mentionPatterns` (वैश्विक फ़ॉलबैक)।
- `messages.responsePrefix`।
