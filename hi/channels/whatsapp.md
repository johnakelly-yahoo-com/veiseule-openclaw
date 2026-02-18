---
title: "WhatsApp"
---

# WhatsApp (वेब चैनल)

Status: WhatsApp Web via Baileys only. Gateway owns the session(s).

## त्वरित सेटअप (शुरुआती)

1. यदि संभव हो तो **अलग फोन नंबर** का उपयोग करें (अनुशंसित)।
2. `~/.openclaw/openclaw.json` में WhatsApp को कॉन्फ़िगर करें।
3. QR कोड स्कैन करने के लिए `openclaw channels login` चलाएँ (Linked Devices)।
4. Gateway प्रारंभ करें।

न्यूनतम विन्यास:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## लक्ष्य

- एक Gateway प्रक्रिया में कई WhatsApp खाते (मल्टी-अकाउंट)।
- निर्धारक रूटिंग: उत्तर WhatsApp पर ही लौटें, कोई मॉडल रूटिंग नहीं।
- मॉडल को उद्धृत उत्तर समझने के लिए पर्याप्त संदर्भ मिले।

## कॉन्फ़िग लिखावट

डिफ़ॉल्ट रूप से, WhatsApp को `/config set|unset` द्वारा ट्रिगर किए गए कॉन्फ़िग अपडेट लिखने की अनुमति होती है (इसके लिए `commands.config: true` आवश्यक है)।

अक्षम करने के लिए:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## आर्किटेक्चर (कौन क्या नियंत्रित करता है)

- **Gateway** Baileys सॉकेट और इनबॉक्स लूप का स्वामी है।
- **CLI / macOS ऐप** Gateway से बात करते हैं; Baileys का प्रत्यक्ष उपयोग नहीं।
- **सक्रिय लिस्नर** आउटबाउंड भेजने के लिए आवश्यक है; अन्यथा भेजना तुरंत विफल हो जाता है।

## फोन नंबर प्राप्त करना (दो मोड)

WhatsApp requires a real mobile number for verification. VoIP and virtual numbers are usually blocked. There are two supported ways to run OpenClaw on WhatsApp:

### समर्पित नंबर (अनुशंसित)

Use a **separate phone number** for OpenClaw. Best UX, clean routing, no self-chat quirks. Ideal setup: **spare/old Android phone + eSIM**. Leave it on Wi‑Fi and power, and link it via QR.

**WhatsApp Business:** You can use WhatsApp Business on the same device with a different number. Great for keeping your personal WhatsApp separate — install WhatsApp Business and register the OpenClaw number there.

**नमूना विन्यास (समर्पित नंबर, सिंगल-यूज़र allowlist):**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**Pairing mode (optional):**
If you want pairing instead of allowlist, set `channels.whatsapp.dmPolicy` to `pairing`. Unknown senders get a pairing code; approve with:
`openclaw pairing approve whatsapp <code>`

### व्यक्तिगत नंबर (फ़ॉलबैक)

Quick fallback: run OpenClaw on **your own number**. Message yourself (WhatsApp “Message yourself”) for testing so you don’t spam contacts. Expect to read verification codes on your main phone during setup and experiments. **Must enable self-chat mode.**
When the wizard asks for your personal WhatsApp number, enter the phone you will message from (the owner/sender), not the assistant number.

**नमूना विन्यास (व्यक्तिगत नंबर, self-chat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Self-chat replies default to `[{identity.name}]` when set (otherwise `[openclaw]`)
if `messages.responsePrefix` is unset. Set it explicitly to customize or disable
the prefix (use `""` to remove it).

### नंबर स्रोत सुझाव

- आपके देश के मोबाइल कैरियर से **लोकल eSIM** (सबसे विश्वसनीय)
  - ऑस्ट्रिया: [hot.at](https://www.hot.at)
  - यूके: [giffgaff](https://www.giffgaff.com) — मुफ़्त SIM, कोई अनुबंध नहीं
- **प्रीपेड SIM** — सस्ता, सत्यापन के लिए केवल एक SMS प्राप्त करना पर्याप्त

**बचें:** TextNow, Google Voice, अधिकांश “free SMS” सेवाएँ — WhatsApp इन्हें आक्रामक रूप से ब्लॉक करता है।

**Tip:** The number only needs to receive one verification SMS. After that, WhatsApp Web sessions persist via `creds.json`.

## Twilio क्यों नहीं?

- प्रारंभिक OpenClaw बिल्ड्स में Twilio के WhatsApp Business एकीकरण का समर्थन था।
- WhatsApp Business नंबर व्यक्तिगत सहायक के लिए उपयुक्त नहीं हैं।
- Meta 24‑घंटे की उत्तर विंडो लागू करता है; यदि आपने पिछले 24 घंटों में उत्तर नहीं दिया है, तो बिज़नेस नंबर नए संदेश शुरू नहीं कर सकता।
- उच्च-वॉल्यूम या “चैटी” उपयोग आक्रामक ब्लॉकिंग को ट्रिगर करता है, क्योंकि बिज़नेस अकाउंट दर्जनों व्यक्तिगत सहायक संदेश भेजने के लिए नहीं बने हैं।
- परिणाम: अविश्वसनीय डिलीवरी और बार-बार ब्लॉक, इसलिए समर्थन हटा दिया गया।

## लॉगिन + क्रेडेंशियल्स

- लॉगिन कमांड: `openclaw channels login` (Linked Devices के माध्यम से QR)।
- मल्टी-अकाउंट लॉगिन: `openclaw channels login --account <id>` (`<id>` = `accountId`)।
- डिफ़ॉल्ट अकाउंट (जब `--account` छोड़ा गया हो): यदि मौजूद हो तो `default`, अन्यथा पहला कॉन्फ़िगर किया गया अकाउंट आईडी (क्रमबद्ध)।
- क्रेडेंशियल्स `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` में संग्रहीत।
- बैकअप कॉपी `creds.json.bak` पर (करप्शन होने पर पुनर्स्थापित)।
- लेगेसी संगतता: पुराने इंस्टॉल Baileys फ़ाइलें सीधे `~/.openclaw/credentials/` में संग्रहीत करते थे।
- लॉगआउट: `openclaw channels logout` (या `--account <id>`) WhatsApp ऑथ स्टेट हटाता है (लेकिन साझा `oauth.json` रखता है)।
- लॉगआउटेड सॉकेट => पुनः लिंक करने का निर्देश देने वाली त्रुटि।

## इनबाउंड फ़्लो (DM + समूह)

- WhatsApp इवेंट्स `messages.upsert` (Baileys) से आते हैं।
- शटडाउन पर इनबॉक्स लिस्नर्स को अलग किया जाता है ताकि परीक्षण/रीस्टार्ट में इवेंट हैंडलर्स जमा न हों।
- स्टेटस/ब्रॉडकास्ट चैट्स अनदेखी की जाती हैं।
- डायरेक्ट चैट्स E.164 का उपयोग करती हैं; समूह group JID का।
- **DM नीति**: `channels.whatsapp.dmPolicy` डायरेक्ट चैट एक्सेस नियंत्रित करता है (डिफ़ॉल्ट: `pairing`)।
  - पेयरिंग: अज्ञात प्रेषकों को पेयरिंग कोड मिलता है ( `openclaw pairing approve whatsapp <code>` के माध्यम से अनुमोदन; कोड 1 घंटे बाद समाप्त)।
  - ओपन: `channels.whatsapp.allowFrom` में `"*"` शामिल होना आवश्यक।
  - आपका लिंक किया हुआ WhatsApp नंबर निहित रूप से विश्वसनीय है, इसलिए self संदेश `channels.whatsapp.dmPolicy` और `channels.whatsapp.allowFrom` जाँचों को छोड़ देते हैं।

### व्यक्तिगत-नंबर मोड (फ़ॉलबैक)

यदि आप OpenClaw को **अपने व्यक्तिगत WhatsApp नंबर** पर चलाते हैं, तो `channels.whatsapp.selfChatMode` सक्षम करें (ऊपर दिए नमूने देखें)।

व्यवहार:

- आउटबाउंड DM कभी भी पेयरिंग उत्तर ट्रिगर नहीं करते (संपर्कों को स्पैम से बचाता है)।
- इनबाउंड अज्ञात प्रेषक अभी भी `channels.whatsapp.dmPolicy` का पालन करते हैं।
- self-chat मोड (allowFrom में आपका नंबर शामिल) स्वचालित रीड रसीदें से बचता है और mention JID को अनदेखा करता है।
- गैर‑self‑chat DM के लिए रीड रसीदें भेजी जाती हैं।

## रीड रसीदें

डिफ़ॉल्ट रूप से, Gateway स्वीकार होने पर इनबाउंड WhatsApp संदेशों को पढ़ा हुआ (नीले टिक) चिह्नित करता है।

वैश्विक रूप से अक्षम करें:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

प्रति-अकाउंट अक्षम करें:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

नोट्स:

- self-chat मोड में रीड रसीदें हमेशा छोड़ी जाती हैं।

## WhatsApp FAQ: संदेश भेजना + पेयरिंग

**Will OpenClaw message random contacts when I link WhatsApp?**  
No. Default DM policy is **pairing**, so unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives, or to sends you explicitly trigger (agent/CLI).

**WhatsApp पर पेयरिंग कैसे काम करती है?**  
पेयरिंग अज्ञात प्रेषकों के लिए DM गेट है:

- नए प्रेषक से पहला DM एक छोटा कोड लौटाता है (संदेश प्रोसेस नहीं होता)।
- अनुमोदन करें: `openclaw pairing approve whatsapp <code>` (सूची के लिए `openclaw pairing list whatsapp`)।
- कोड 1 घंटे बाद समाप्त; लंबित अनुरोध प्रति चैनल 3 तक सीमित।

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. DM access control (`dmPolicy`/`allowFrom`) is global per WhatsApp account. See [Multi-Agent Routing](/concepts/multi-agent).

**Why do you ask for my phone number in the wizard?**  
The wizard uses it to set your **allowlist/owner** so your own DMs are permitted. It’s not used for auto-sending. If you run on your personal WhatsApp number, use that same number and enable `channels.whatsapp.selfChatMode`.

## संदेश सामान्यीकरण (मॉडल क्या देखता है)

- `Body` वर्तमान संदेश बॉडी है, एन्वेलप के साथ।

- उद्धृत उत्तर संदर्भ **हमेशा जोड़ा जाता है**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- उत्तर मेटाडेटा भी सेट होता है:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = उद्धृत बॉडी या मीडिया प्लेसहोल्डर
  - `ReplyToSender` = E.164 (जब ज्ञात हो)

- केवल‑मीडिया इनबाउंड संदेश प्लेसहोल्डर का उपयोग करते हैं:
  - `<media:image|video|audio|document|sticker>`

## समूह

- समूह `agent:<agentId>:whatsapp:group:<jid>` सत्रों से मैप होते हैं।
- समूह नीति: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (डिफ़ॉल्ट `allowlist`)।
- सक्रियण मोड:
  - `mention` (डिफ़ॉल्ट): @mention या regex मैच आवश्यक।
  - `always`: हमेशा ट्रिगर।
- `/activation mention|always` केवल owner‑only है और इसे अलग संदेश के रूप में भेजना आवश्यक है।
- Owner = `channels.whatsapp.allowFrom` (या यदि अनसेट हो तो self E.164)।
- **इतिहास इंजेक्शन** (केवल लंबित):
  - हाल के _अप्रोसेस्ड_ संदेश (डिफ़ॉल्ट 50) इसके अंतर्गत डाले जाते हैं:
    `[Chat messages since your last reply - for context]` (जो संदेश पहले से सत्र में हैं, उन्हें पुनः इंजेक्ट नहीं किया जाता)
  - वर्तमान संदेश इसके अंतर्गत:
    `[Current message - respond to this]`
  - प्रेषक प्रत्यय जोड़ा जाता है: `[from: Name (+E164)]`
- समूह मेटाडेटा 5 मिनट के लिए कैश किया जाता है (विषय + प्रतिभागी)।

## उत्तर वितरण (थ्रेडिंग)

- WhatsApp Web मानक संदेश भेजता है (वर्तमान Gateway में उद्धृत उत्तर थ्रेडिंग नहीं)।
- इस चैनल पर रिप्लाई टैग्स अनदेखे किए जाते हैं।

## स्वीकृति प्रतिक्रियाएँ (प्राप्ति पर ऑटो‑रिएक्ट)

WhatsApp can automatically send emoji reactions to incoming messages immediately upon receipt, before the bot generates a reply. This provides instant feedback to users that their message was received.

**विन्यास:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**विकल्प:**

- `emoji` (string): Emoji to use for acknowledgment (e.g., "👀", "✅", "📨"). Empty or omitted = feature disabled.
- `direct` (boolean, डिफ़ॉल्ट: `true`): डायरेक्ट/DM चैट्स में प्रतिक्रियाएँ भेजें।
- `group` (string, डिफ़ॉल्ट: `"mentions"`): समूह चैट व्यवहार:
  - `"always"`: सभी समूह संदेशों पर प्रतिक्रिया (बिना @mention भी)
  - `"mentions"`: केवल तब प्रतिक्रिया जब बॉट @mentioned हो
  - `"never"`: समूहों में कभी प्रतिक्रिया न करें

**प्रति‑अकाउंट ओवरराइड:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**व्यवहार नोट्स:**

- प्रतिक्रियाएँ संदेश प्राप्त होते ही **तुरंत** भेजी जाती हैं, टाइपिंग संकेतकों या बॉट उत्तरों से पहले।
- `requireMention: false` (activation: always) वाले समूहों में, `group: "mentions"` सभी संदेशों पर प्रतिक्रिया करेगा (केवल @mentions नहीं)।
- फायर‑एंड‑फॉरगेट: प्रतिक्रिया विफलताओं को लॉग किया जाता है, लेकिन बॉट के उत्तर देने से नहीं रोकतीं।
- समूह प्रतिक्रियाओं के लिए प्रतिभागी JID स्वतः शामिल किया जाता है।
- WhatsApp `messages.ackReaction` को अनदेखा करता है; इसके बजाय `channels.whatsapp.ackReaction` का उपयोग करें।

## एजेंट टूल (प्रतिक्रियाएँ)

- टूल: `whatsapp` with `react` action (`chatJid`, `messageId`, `emoji`, वैकल्पिक `remove`)।
- वैकल्पिक: `participant` (समूह प्रेषक), `fromMe` (अपने ही संदेश पर प्रतिक्रिया), `accountId` (मल्टी‑अकाउंट)।
- प्रतिक्रिया हटाने का अर्थ: देखें [/tools/reactions](/tools/reactions)।
- टूल गेटिंग: `channels.whatsapp.actions.reactions` (डिफ़ॉल्ट: सक्षम)।

## सीमाएँ

- आउटबाउंड टेक्स्ट को `channels.whatsapp.textChunkLimit` तक चंक किया जाता है (डिफ़ॉल्ट 4000)।
- वैकल्पिक नई‑लाइन चंकिंग: लंबाई चंकिंग से पहले खाली लाइनों (पैराग्राफ सीमाएँ) पर विभाजित करने के लिए `channels.whatsapp.chunkMode="newline"` सेट करें।
- इनबाउंड मीडिया सेव `channels.whatsapp.mediaMaxMb` द्वारा सीमित (डिफ़ॉल्ट 50 MB)।
- आउटबाउंड मीडिया आइटम `agents.defaults.mediaMaxMb` द्वारा सीमित (डिफ़ॉल्ट 5 MB)।

## आउटबाउंड भेजना (टेक्स्ट + मीडिया)

- सक्रिय वेब लिस्नर का उपयोग करता है; Gateway न चलने पर त्रुटि।
- टेक्स्ट चंकिंग: प्रति संदेश अधिकतम 4k ( `channels.whatsapp.textChunkLimit` के माध्यम से कॉन्फ़िगर करने योग्य, वैकल्पिक `channels.whatsapp.chunkMode`)।
- मीडिया:
  - इमेज/वीडियो/ऑडियो/डॉक्यूमेंट समर्थित।
  - ऑडियो PTT के रूप में भेजा जाता है; `audio/ogg` => `audio/ogg; codecs=opus`।
  - कैप्शन केवल पहले मीडिया आइटम पर।
  - मीडिया फ़ेच HTTP(S) और लोकल पाथ्स का समर्थन करता है।
  - एनिमेटेड GIFs: इनलाइन लूपिंग के लिए WhatsApp `gifPlayback: true` के साथ MP4 अपेक्षित करता है।
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send` पैरामीटर में `gifPlayback: true` शामिल

## वॉयस नोट्स (PTT ऑडियो)

WhatsApp ऑडियो को **वॉयस नोट्स** (PTT बबल) के रूप में भेजता है।

- Best results: OGG/Opus. OpenClaw rewrites `audio/ogg` to `audio/ogg; codecs=opus`.
- WhatsApp के लिए `[[audio_as_voice]]` अनदेखा किया जाता है (ऑडियो पहले से ही वॉयस नोट के रूप में जाता है)।

## मीडिया सीमाएँ + अनुकूलन

- डिफ़ॉल्ट आउटबाउंड सीमा: 5 MB (प्रति मीडिया आइटम)।
- ओवरराइड: `agents.defaults.mediaMaxMb`।
- इमेजेज़ को सीमा के भीतर JPEG में स्वतः अनुकूलित किया जाता है (रीसाइज़ + क्वालिटी स्वीप)।
- ओवरसाइज़ मीडिया => त्रुटि; मीडिया उत्तर टेक्स्ट चेतावनी पर फ़ॉलबैक करता है।

## हार्टबीट्स

- **Gateway हार्टबीट** कनेक्शन स्वास्थ्य लॉग करता है (`web.heartbeatSeconds`, डिफ़ॉल्ट 60s)।
- **एजेंट हार्टबीट** प्रति एजेंट (`agents.list[].heartbeat`) या वैश्विक रूप से
  `agents.defaults.heartbeat` के माध्यम से कॉन्फ़िगर किया जा सकता है (जब प्रति‑एजेंट प्रविष्टियाँ सेट न हों तो फ़ॉलबैक)।
  - Uses the configured heartbeat prompt (default: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + `HEARTBEAT_OK` skip behavior.
  - डिलीवरी डिफ़ॉल्ट रूप से अंतिम उपयोग किए गए चैनल पर (या कॉन्फ़िगर किए गए लक्ष्य पर)।

## पुनःकनेक्ट व्यवहार

- बैकऑफ़ नीति: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`।
- यदि maxAttempts पहुँच जाता है, तो वेब मॉनिटरिंग रुक जाती है (degraded)।
- लॉगआउटेड => रोकें और पुनः लिंक की आवश्यकता।

## कॉन्फ़िग त्वरित मानचित्र

- `channels.whatsapp.dmPolicy` (DM नीति: pairing/allowlist/open/disabled)।
- `channels.whatsapp.selfChatMode` (same‑phone सेटअप; बॉट आपका व्यक्तिगत WhatsApp नंबर उपयोग करता है)।
- `channels.whatsapp.allowFrom` (DM allowlist). WhatsApp uses E.164 phone numbers (no usernames).
- `channels.whatsapp.mediaMaxMb` (इनबाउंड मीडिया सेव सीमा)।
- `channels.whatsapp.ackReaction` (संदेश प्राप्ति पर ऑटो‑रिएक्शन: `{emoji, direct, group}`)।
- `channels.whatsapp.accounts.<accountId>.*` (per-account settings + optional `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (per-account inbound media cap).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (per-account ack reaction override).
- `channels.whatsapp.groupAllowFrom` (समूह प्रेषक allowlist)।
- `channels.whatsapp.groupPolicy` (समूह नीति)।
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (group history context; `0` disables).
- `channels.whatsapp.dmHistoryLimit` (DM history limit in user turns). Per-user overrides: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (समूह allowlist + mention गेटिंग डिफ़ॉल्ट; सभी की अनुमति के लिए `"*"` का उपयोग करें)
- `channels.whatsapp.actions.reactions` (WhatsApp टूल प्रतिक्रियाओं को गेट करें)।
- `agents.list[].groupChat.mentionPatterns` (या `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (inbound prefix; per-account: `channels.whatsapp.accounts.<accountId>.messagePrefix`; deprecated: `messages.messagePrefix`)
- `messages.responsePrefix` (आउटबाउंड प्रीफ़िक्स)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (वैकल्पिक ओवरराइड)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (प्रति‑एजेंट ओवरराइड्स)
- `session.*` (scope, idle, store, mainKey)
- `web.enabled` (false होने पर चैनल स्टार्टअप अक्षम)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## लॉग्स + समस्या‑निवारण

- उपप्रणालियाँ: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`।
- लॉग फ़ाइल: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (कॉन्फ़िगर करने योग्य)।
- समस्या‑निवारण मार्गदर्शिका: [Gateway troubleshooting](/gateway/troubleshooting)।

## समस्या‑निवारण (त्वरित)

**लिंक नहीं / QR लॉगिन आवश्यक**

- लक्षण: `channels status` में `linked: false` दिखता है या “Not linked” चेतावनी।
- समाधान: Gateway होस्ट पर `openclaw channels login` चलाएँ और QR स्कैन करें (WhatsApp → Settings → Linked Devices)।

**लिंक्ड लेकिन डिस्कनेक्टेड / पुनःकनेक्ट लूप**

- लक्षण: `channels status` में `running, disconnected` दिखता है या “Linked but disconnected” चेतावनी।
- Fix: `openclaw doctor` (or restart the gateway). If it persists, relink via `channels login` and inspect `openclaw logs --follow`.

**Bun रनटाइम**

- Bun is **not recommended**. WhatsApp (Baileys) and Telegram are unreliable on Bun.
  Run the gateway with **Node**. (See Getting Started runtime note.)

