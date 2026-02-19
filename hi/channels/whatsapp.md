---
summary: "WhatsApp (वेब चैनल) एकीकरण: लॉगिन, इनबॉक्स, उत्तर, मीडिया और संचालन"
read_when:
  - WhatsApp/वेब चैनल व्यवहार या इनबॉक्स रूटिंग पर काम करते समय
title: "WhatsApp"
---

# WhatsApp (वेब चैनल)

स्थिति: WhatsApp Web (Baileys) के माध्यम से प्रोडक्शन-रेडी। Gateway लिंक्ड सत्र(सत्रों) का स्वामित्व रखता है।

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    अज्ञात प्रेषकों के लिए डिफ़ॉल्ट DM नीति पेयरिंग है。
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    क्रॉस-चैनल डायग्नोस्टिक्स और रिपेयर प्लेबुक्स。
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    पूर्ण चैनल कॉन्फ़िग पैटर्न और उदाहरण。
  
</Card>
</CardGroup>

## त्वरित सेटअप

<Steps>
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    किसी विशिष्ट खाते के लिए:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    ```
    पेयरिंग अनुरोध 1 घंटे के बाद समाप्त हो जाते हैं। लंबित अनुरोध प्रत्येक चैनल पर अधिकतम 3 तक सीमित हैं।
    ```

  
</Step>
</Steps>

<Note>
OpenClaw संभव होने पर WhatsApp को एक अलग नंबर पर चलाने की सलाह देता है। (चैनल मेटाडेटा और ऑनबोर्डिंग फ्लो उस सेटअप के लिए अनुकूलित हैं, लेकिन व्यक्तिगत-नंबर सेटअप भी समर्थित हैं।)
</Note>

## डिप्लॉयमेंट पैटर्न

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    यह सबसे स्वच्छ संचालन मोड है:

    ````
    - OpenClaw के लिए अलग WhatsApp पहचान
    - अधिक स्पष्ट DM allowlists और रूटिंग सीमाएँ
    - स्वयं-चैट भ्रम की कम संभावना
    
    न्यूनतम नीति पैटर्न:
    
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
    ````

  
</Accordion>

  <Accordion title="Personal-number fallback">
    ऑनबोर्डिंग व्यक्तिगत-नंबर मोड का समर्थन करता है और स्वयं-चैट के अनुकूल बेसलाइन लिखता है:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    वर्तमान OpenClaw चैनल आर्किटेक्चर में मैसेजिंग प्लेटफ़ॉर्म चैनल WhatsApp Web-आधारित (`Baileys`) है।

    ```
    बिल्ट-इन chat-channel रजिस्ट्री में कोई अलग Twilio WhatsApp मैसेजिंग चैनल नहीं है।
    ```

  
</Accordion>
</AccordionGroup>

## रनटाइम मॉडल

- Gateway WhatsApp socket और reconnect लूप का स्वामित्व रखता है।
- आउटबाउंड भेजने के लिए लक्ष्य खाते के लिए एक सक्रिय WhatsApp listener आवश्यक है।
- Status और broadcast चैट्स को अनदेखा किया जाता है (`@status`, `@broadcast`)।
- Direct चैट्स DM सत्र नियमों का उपयोग करती हैं (`session.dmScope`; डिफ़ॉल्ट `main` DMs को agent के main सत्र में समेट देता है)।
- Group सत्र अलग-थलग रहते हैं (`agent:<agentId>:whatsapp:group:<jid>`)।

## एक्सेस नियंत्रण और सक्रियण

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` direct चैट एक्सेस को नियंत्रित करता है:

    ```
    - `pairing` (डिफ़ॉल्ट)
    - `allowlist`
    - `open` (इसके लिए `allowFrom` में `"*"` शामिल होना आवश्यक है)
    - `disabled`
    
    `allowFrom` E.164-शैली के नंबर स्वीकार करता है (आंतरिक रूप से normalized)।
    
    मल्टी-अकाउंट ओवरराइड: `channels.whatsapp.accounts.<id>.dmPolicy` (और `allowFrom`) उस खाते के लिए चैनल-स्तर डिफ़ॉल्ट पर प्राथमिकता लेते हैं।
    
    रनटाइम व्यवहार विवरण:
    
    - pairings को channel allow-store में persist किया जाता है और कॉन्फ़िगर किए गए `allowFrom` के साथ मर्ज किया जाता है
    - यदि कोई allowlist कॉन्फ़िगर नहीं है, तो लिंक किया गया self नंबर डिफ़ॉल्ट रूप से अनुमति प्राप्त होता है
    - आउटबाउंड `fromMe` DMs कभी भी auto-paired नहीं होते
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Group एक्सेस के दो स्तर हैं:

    ```
    1. **Group membership allowlist** (`channels.whatsapp.groups`)
       - यदि `groups` छोड़ा गया है, तो सभी groups पात्र हैं
       - यदि `groups` मौजूद है, तो यह group allowlist की तरह कार्य करता है (`"*"` अनुमति है)
    
    2. **Group sender policy** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: sender allowlist को बायपास करता है
       - `allowlist`: sender को `groupAllowFrom` (या `*`) से मेल खाना चाहिए
       - `disabled`: सभी group inbound को ब्लॉक करता है
    
    Sender allowlist fallback:
    
    - यदि `groupAllowFrom` सेट नहीं है, तो रनटाइम उपलब्ध होने पर `allowFrom` पर वापस जाता है
    
    नोट: यदि बिल्कुल भी `channels.whatsapp` ब्लॉक मौजूद नहीं है, तो रनटाइम group-policy fallback प्रभावी रूप से `open` होता है।
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    डिफ़ॉल्ट रूप से Group replies में उल्लेख (mention) आवश्यक होता है।

    ```
    Mention डिटेक्शन में शामिल हैं:
    
    - bot पहचान के स्पष्ट WhatsApp mentions
    - कॉन्फ़िगर किए गए mention regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - bot को reply करने की अप्रत्यक्ष पहचान (reply sender bot पहचान से मेल खाता है)
    
    सत्र-स्तर सक्रियण कमांड:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` सत्र स्थिति को अपडेट करता है (ग्लोबल कॉन्फ़िग नहीं)। यह owner-gated है।
    ```

  
</Tab>
</Tabs>

## व्यक्तिगत-नंबर और स्वयं-चैट व्यवहार

वैश्विक रूप से अक्षम करें:

- स्वयं-चैट टर्न के लिए read receipts को छोड़ें
- mention-JID auto-trigger व्यवहार को अनदेखा करें जो अन्यथा आपको स्वयं को ping कर सकता है
- यदि `messages.responsePrefix` सेट नहीं है, तो स्वयं-चैट replies डिफ़ॉल्ट रूप से `[{identity.name}]` या `[openclaw]` होंगी

## मैसेज normalization और संदर्भ

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    इनकमिंग WhatsApp संदेशों को साझा inbound envelope में wrap किया जाता है।

    ````
    यदि कोई quoted reply मौजूद है, तो संदर्भ इस रूप में जोड़ा जाता है:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Reply metadata फ़ील्ड्स भी उपलब्ध होने पर भरे जाते हैं (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164)।
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    केवल-मीडिया इनबाउंड संदेशों को निम्न placeholders के साथ normalize किया जाता है:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Location और contact payloads को रूटिंग से पहले टेक्स्ट संदर्भ में normalize किया जाता है।
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Groups के लिए, बिना प्रोसेस किए गए संदेशों को buffer किया जा सकता है और जब bot अंततः ट्रिगर होता है तो संदर्भ के रूप में inject किया जा सकता है।

    ```
    - डिफ़ॉल्ट सीमा: `50`
    - कॉन्फ़िग: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` अक्षम करता है
    
    Injection markers:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    स्वीकार किए गए इनबाउंड WhatsApp संदेशों के लिए डिफ़ॉल्ट रूप से read receipts सक्षम होते हैं।

    ````
    वैश्विक रूप से अक्षम करें:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    प्रति-अकाउंट ओवरराइड:
    
    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```
    
    स्वयं-चैट टर्न में, वैश्विक रूप से सक्षम होने पर भी read receipts को छोड़ दिया जाता है।
    ````

  
</Accordion>
</AccordionGroup>

## डिलीवरी, चंकिंग, और मीडिया

<AccordionGroup>
  <Accordion title="Text chunking">
    - डिफ़ॉल्ट चंक सीमा: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - `newline` मोड पैराग्राफ़ सीमाओं (खाली पंक्तियाँ) को प्राथमिकता देता है, फिर लंबाई-सुरक्षित चंकिंग पर वापस जाता है
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - इमेज, वीडियो, ऑडियो (PTT वॉइस-नोट), और दस्तावेज़ पेलोड का समर्थन करता है
    - वॉइस-नोट संगतता के लिए `audio/ogg` को `audio/ogg; codecs=opus` में पुनर्लिखा जाता है
    - वीडियो भेजते समय `gifPlayback: true` के माध्यम से एनिमेटेड GIF प्लेबैक समर्थित है
    - मल्टी-मीडिया रिप्लाई पेलोड भेजते समय कैप्शन पहले मीडिया आइटम पर लागू किए जाते हैं
    - मीडिया स्रोत HTTP(S), `file://`, या लोकल पाथ हो सकता है
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - इनबाउंड मीडिया सेव सीमा: `channels.whatsapp.mediaMaxMb` (डिफ़ॉल्ट `50`)
    - ऑटो-रिप्लाई के लिए आउटबाउंड मीडिया सीमा: `agents.defaults.mediaMaxMb` (डिफ़ॉल्ट `5MB`)
    - इमेज को सीमा में फिट करने के लिए स्वतः अनुकूलित (रिसाइज़/क्वालिटी समायोजन) किया जाता है
    - मीडिया भेजने में विफलता होने पर, पहले आइटम के लिए फॉलबैक टेक्स्ट चेतावनी भेजी जाती है ताकि प्रतिक्रिया चुपचाप ड्रॉप न हो
  
</Accordion>
</AccordionGroup>

## स्वीकृति प्रतिक्रियाएँ

**विन्यास:**

```json5
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

- इनबाउंड स्वीकार होते ही तुरंत भेजी जाती हैं (प्री-रिप्लाई)
- `direct` (boolean, डिफ़ॉल्ट: `true`): डायरेक्ट/DM चैट्स में प्रतिक्रियाएँ भेजें।
- `group` (string, डिफ़ॉल्ट: `"mentions"`): समूह चैट व्यवहार:
- WhatsApp में `channels.whatsapp.ackReaction` का उपयोग होता है (यहाँ लेगेसी `messages.ackReaction` उपयोग नहीं किया जाता)

## मल्टी-अकाउंट और क्रेडेंशियल्स

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - अकाउंट आईडी `channels.whatsapp.accounts` से प्राप्त होती हैं
    - डिफ़ॉल्ट अकाउंट चयन: यदि `default` मौजूद हो तो वही, अन्यथा पहला कॉन्फ़िगर किया गया अकाउंट आईडी (सॉर्टेड)
    - लुकअप के लिए अकाउंट आईडी आंतरिक रूप से सामान्यीकृत की जाती हैं
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - वर्तमान ऑथ पाथ: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - बैकअप फ़ाइल: `creds.json.bak`
    - `~/.openclaw/credentials/` में लेगेसी डिफ़ॉल्ट ऑथ अभी भी डिफ़ॉल्ट-अकाउंट फ्लो के लिए पहचाना/माइग्रेट किया जाता है
  
</Accordion>

  <Accordion title="Logout behavior">    `openclaw channels logout --channel whatsapp [--account <id>]` उस अकाउंट के लिए WhatsApp ऑथ स्टेट साफ़ करता है।

    ```
    लेगेसी ऑथ डायरेक्टरी में, `oauth.json` सुरक्षित रखा जाता है जबकि Baileys ऑथ फ़ाइलें हटा दी जाती हैं।
    ```

  
</Accordion>
</AccordionGroup>

## सीमाएँ

- आउटबाउंड टेक्स्ट को `channels.whatsapp.textChunkLimit` तक चंक किया जाता है (डिफ़ॉल्ट 4000)।
- वैकल्पिक नई‑लाइन चंकिंग: लंबाई चंकिंग से पहले खाली लाइनों (पैराग्राफ सीमाएँ) पर विभाजित करने के लिए `channels.whatsapp.chunkMode="newline"` सेट करें।
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- इनबाउंड मीडिया सेव `channels.whatsapp.mediaMaxMb` द्वारा सीमित (डिफ़ॉल्ट 50 MB)।

## आउटबाउंड भेजना (टेक्स्ट + मीडिया)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    लक्षण: चैनल स्टेटस लिंक नहीं दिखा रहा है।

    ````
    समाधान:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    लक्षण: लिंक किया गया अकाउंट बार-बार डिस्कनेक्ट हो रहा है या पुनः कनेक्ट करने की कोशिश कर रहा है।

    ````
    समाधान:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    यदि आवश्यक हो, तो `channels login` के साथ पुनः लिंक करें।
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    जब लक्ष्य अकाउंट के लिए कोई सक्रिय gateway listener मौजूद नहीं होता, तो आउटबाउंड सेंड तुरंत विफल हो जाते हैं।

    ```
    सुनिश्चित करें कि gateway चल रहा है और अकाउंट लिंक किया गया है।
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    इस क्रम में जाँच करें:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist प्रविष्टियाँ
    - मेंशन गेटिंग (`requireMention` + मेंशन पैटर्न)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway रनटाइम में Node का उपयोग होना चाहिए। स्थिर WhatsApp/Telegram gateway संचालन के लिए Bun को असंगत के रूप में चिह्नित किया गया है।
  
</Accordion>
</AccordionGroup>

## कॉन्फ़िगरेशन संदर्भ संकेत

प्राथमिक संदर्भ:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

उच्च-प्रभाव वाले WhatsApp फ़ील्ड्स:

- एक्सेस: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- डिलीवरी: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- मल्टी-अकाउंट: `accounts.<id>.enabled`, `accounts.<id>.authDir`, अकाउंट-स्तरीय ओवरराइड्स
- ऑपरेशन्स: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- सेशन व्यवहार: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## संबंधित

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
