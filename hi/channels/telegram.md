---
summary: "Telegram बॉट समर्थन की स्थिति, क्षमताएँ और विन्यास"
read_when:
  - Telegram फीचर्स या वेबहुक्स पर काम करते समय
title: "Telegram"
---

# Telegram (Bot API)

स्थिति: grammY के माध्यम से बॉट DMs + ग्रुप्स के लिए प्रोडक्शन-रेडी। Long polling डिफ़ॉल्ट मोड है; webhook मोड वैकल्पिक है.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Telegram के लिए डिफ़ॉल्ट DM नीति pairing है.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    क्रॉस-चैनल डायग्नोस्टिक्स और रिपेयर प्लेबुक्स.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    पूर्ण चैनल कॉन्फ़िगरेशन पैटर्न और उदाहरण.
  
</Card>
</CardGroup>

## त्वरित सेटअप

<Steps>
  <Step title="Create the bot token in BotFather">
    Telegram खोलें और **@BotFather** के साथ चैट करें (सुनिश्चित करें कि हैंडल बिल्कुल `@BotFather` है).
  

    ```
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",
        },
      },
    }
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (केवल डिफ़ॉल्ट अकाउंट)।
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Pairing कोड 1 घंटे के बाद समाप्त हो जाते हैं.
    ```

  
</Step>

  <Step title="Add the bot to a group">बॉट को अपने समूह में जोड़ें, फिर अपने एक्सेस मॉडल के अनुसार `channels.telegram.groups` और `groupPolicy` सेट करें।
</Step>
</Steps>

<Note>
टोकन रेज़ोल्यूशन का क्रम अकाउंट-अवेयर होता है। व्यवहार में, कॉन्फ़िग वैल्यूज़ env fallback पर प्राथमिकता लेती हैं, और `TELEGRAM_BOT_TOKEN` केवल डिफ़ॉल्ट अकाउंट पर लागू होता है।
</Note>

## Telegram साइड सेटिंग्स

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Telegram बॉट्स में डिफ़ॉल्ट रूप से **Privacy Mode** सक्षम होता है, जो उन्हें मिलने वाले समूह संदेशों को सीमित करता है।

    ```
    यदि बॉट को सभी समूह संदेश देखने हैं, तो इनमें से एक करें:
    
    - `/setprivacy` के माध्यम से privacy mode अक्षम करें, या
    - बॉट को समूह का एडमिन बनाएं।
    
    Privacy mode टॉगल करने के बाद, प्रत्येक समूह में बॉट को हटाकर फिर से जोड़ें ताकि Telegram बदलाव लागू कर सके।
    ```

  
</Accordion>

  <Accordion title="Group permissions">एडमिन स्टेटस Telegram समूह सेटिंग्स में नियंत्रित होता है।

    ```
    एडमिन बॉट्स सभी समूह संदेश प्राप्त करते हैं, जो हमेशा सक्रिय समूह व्यवहार के लिए उपयोगी है।
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - समूह जोड़ने की अनुमति/अस्वीकार करने के लिए `/setjoingroups`
    - समूह दृश्यता व्यवहार के लिए `/setprivacy`
    ```

  
</Accordion>
</AccordionGroup>

## एक्सेस कंट्रोल और सक्रियण

<Tabs>
  <Tab title="DM policy">`channels.telegram.dmPolicy` डायरेक्ट मैसेज एक्सेस को नियंत्रित करता है:

    ```
    थर्ड-पार्टी विधि (कम निजी): `@userinfobot` या `@getidsbot`।
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    दो स्वतंत्र नियंत्रण हैं:
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">1. **कौन से समूह अनुमत हैं** (`channels.telegram.groups`)
   - कोई `groups` कॉन्फ़िग नहीं: सभी समूह अनुमत
   - `groups` कॉन्फ़िग किया गया: allowlist की तरह कार्य करता है (स्पष्ट IDs या `"*"`)

2. **समूहों में कौन से प्रेषक अनुमत हैं** (`channels.telegram.groupPolicy`)
   - `open`
   - `allowlist` (डिफ़ॉल्ट)
   - `disabled`

`groupAllowFrom` का उपयोग समूह प्रेषक फ़िल्टरिंग के लिए होता है। यदि सेट नहीं है, तो Telegram `allowFrom` पर fallback करता है।
`groupAllowFrom` प्रविष्टियाँ संख्यात्मक Telegram user IDs होनी चाहिए।

उदाहरण: एक विशिष्ट समूह में किसी भी सदस्य को अनुमति दें:

    ```
    डिफ़ॉल्ट रूप से समूह उत्तरों के लिए मेंशन आवश्यक है।
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">मेंशन निम्न में से किसी से आ सकता है:

- मूल `@botusername` मेंशन, या
- निम्न में परिभाषित मेंशन पैटर्न:
  - `agents.list[].groupChat.mentionPatterns`
  - `messages.groupChat.mentionPatterns`

सेशन-स्तरीय कमांड टॉगल:

- `/activation always`
- `/activation mention`

ये केवल सेशन स्टेट को अपडेट करते हैं। स्थायित्व के लिए कॉन्फ़िग का उपयोग करें।

स्थायी कॉन्फ़िग उदाहरण:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false },
          },
        },
      },
    }
    ```

```json5
समूह चैट ID प्राप्त करना:

- किसी समूह संदेश को `@userinfobot` / `@getidsbot` पर फ़ॉरवर्ड करें
- या `openclaw logs --follow` से `chat.id` पढ़ें
- या Bot API `getUpdates` का निरीक्षण करें
```

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## Telegram का स्वामित्व gateway प्रक्रिया के पास होता है।

- रूटिंग निर्धारक (deterministic) है: Telegram से आने वाले संदेशों के उत्तर वापस Telegram पर ही जाते हैं (मॉडल चैनल नहीं चुनता)।
- इनबाउंड संदेश reply metadata और media placeholders के साथ साझा चैनल envelope में सामान्यीकृत होते हैं।
- समूह सेशन समूह ID के आधार पर अलग-अलग रखे जाते हैं।
- Forum topics में `:topic:<threadId>` जोड़ा जाता है ताकि विषय अलग-अलग रहें। DM संदेश `message_thread_id` ले जा सकते हैं; OpenClaw उन्हें thread-aware सेशन कीज़ के साथ रूट करता है और उत्तरों के लिए thread ID संरक्षित रखता है।
- Long polling, grammY runner का उपयोग करते हुए, प्रति-चैट/प्रति-थ्रेड अनुक्रमण के साथ किया जाता है।
- कुल runner sink concurrency, `agents.defaults.maxConcurrent` का उपयोग करती है। Telegram Bot API में read-receipt समर्थन नहीं है (`sendReadReceipts` लागू नहीं होता)।
- फ़ीचर संदर्भ

## OpenClaw आंशिक उत्तरों को स्ट्रीम कर सकता है, जिसके लिए वह एक अस्थायी Telegram संदेश भेजता है और टेक्स्ट आते ही उसे संपादित करता है।

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">आवश्यकता:

- `channels.telegram.streamMode` `"off"` नहीं होना चाहिए (डिफ़ॉल्ट: `"partial"`)

मोड्स:

- `off`: कोई लाइव प्रीव्यू नहीं
- `partial`: आंशिक टेक्स्ट से बार-बार प्रीव्यू अपडेट
- `block`: `channels.telegram.draftChunk` का उपयोग करते हुए खंडित प्रीव्यू अपडेट

`streamMode: "block"` के लिए `draftChunk` डिफ़ॉल्ट:

- `minChars: 200`
- `maxChars: 800`
- `breakPreference: "paragraph"`

`maxChars` को `channels.telegram.textChunkLimit` द्वारा सीमित किया जाता है।

यह डायरेक्ट चैट और समूह/टॉपिक्स दोनों में काम करता है।

केवल टेक्स्ट उत्तरों के लिए, OpenClaw उसी प्रीव्यू संदेश को बनाए रखता है और अंत में उसी स्थान पर अंतिम संपादन करता है (कोई दूसरा संदेश नहीं)।

जटिल उत्तरों (उदाहरण के लिए media payloads) के लिए, OpenClaw सामान्य अंतिम डिलीवरी पर fallback करता है और फिर प्रीव्यू संदेश को साफ़ कर देता है।

`streamMode` ब्लॉक स्ट्रीमिंग से अलग है। जब Telegram के लिए ब्लॉक स्ट्रीमिंग स्पष्ट रूप से सक्षम होती है, तो OpenClaw डबल-स्ट्रीमिंग से बचने के लिए प्रीव्यू स्ट्रीम को छोड़ देता है।

केवल Telegram के लिए reasoning स्ट्रीम:

- `/reasoning stream` जनरेशन के दौरान reasoning को लाइव प्रीव्यू में भेजता है
- अंतिम उत्तर reasoning टेक्स्ट के बिना भेजा जाता है

    ```
    आउटबाउंड टेक्स्ट Telegram `parse_mode: "HTML"` का उपयोग करता है।
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">- Markdown-जैसा टेक्स्ट Telegram-सुरक्षित HTML में रेंडर किया जाता है।
- कच्चा मॉडल HTML, Telegram parse विफलताओं को कम करने के लिए escape किया जाता है।
- यदि Telegram parsed HTML को अस्वीकार करता है, तो OpenClaw साधारण टेक्स्ट के रूप में पुनः प्रयास करता है।

Link previews डिफ़ॉल्ट रूप से सक्षम होते हैं और `channels.telegram.linkPreview: false` से अक्षम किए जा सकते हैं।

    ```
    Telegram कमांड मेनू पंजीकरण स्टार्टअप पर `setMyCommands` के साथ संभाला जाता है।
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">नेटिव कमांड डिफ़ॉल्ट:

- `commands.native: "auto"` Telegram के लिए नेटिव कमांड सक्षम करता है

कस्टम कमांड मेनू प्रविष्टियाँ जोड़ें:

    ```
    इनलाइन कीबोर्ड स्कोप कॉन्फ़िगर करें:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    {
      channels: {
        telegram: {
          groups: {
            "-1001234567890": { requireMention: false }, // always respond in this group
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Inline buttons">प्रति-अकाउंट ओवरराइड:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Per-account override:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    Scopes:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (डिफ़ॉल्ट)
    
    Legacy `capabilities: ["inlineButtons"]` को `inlineButtons: "all"` पर मैप किया जाता है।
    
    संदेश क्रिया उदाहरण:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    कॉलबैक क्लिक एजेंट को टेक्स्ट के रूप में पास किए जाते हैं:
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">    Telegram टूल क्रियाओं में शामिल हैं:

    ```
    - `sendMessage` (`to`, `content`, वैकल्पिक `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    चैनल संदेश क्रियाएँ उपयोग में आसान उपनाम प्रदान करती हैं (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`)।
    
    नियंत्रण गेटिंग:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (डिफ़ॉल्ट: अक्षम)
    
    रिएक्शन हटाने की सेमांटिक्स: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">    Telegram जनरेट किए गए आउटपुट में स्पष्ट रिप्लाई थ्रेडिंग टैग्स का समर्थन करता है:

    ```
    - `[[reply_to_current]]` ट्रिगर करने वाले संदेश का उत्तर देता है
    - `[[reply_to:<id>]]` किसी विशिष्ट Telegram संदेश ID का उत्तर देता है
    
    `channels.telegram.replyToMode` हैंडलिंग नियंत्रित करता है:
    
    - `off` (डिफ़ॉल्ट)
    - `first`
    - `all`
    
    नोट: `off` निहित रिप्लाई थ्रेडिंग को अक्षम करता है। स्पष्ट `[[reply_to_*]]` टैग्स अभी भी मान्य रहेंगे।
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">    फ़ोरम सुपरग्रुप्स:

    ```
    - टॉपिक सेशन कुंजियाँ `:topic:<threadId>` जोड़ती हैं
    - रिप्लाई और टाइपिंग टॉपिक थ्रेड को लक्षित करते हैं
    - टॉपिक कॉन्फ़िग पथ:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    सामान्य टॉपिक (`threadId=1`) विशेष-स्थिति:
    
    - संदेश भेजते समय `message_thread_id` शामिल नहीं किया जाता (`sendMessage(...thread_id=1)` को Telegram अस्वीकार करता है)
    - टाइपिंग क्रियाओं में अभी भी `message_thread_id` शामिल रहता है
    
    टॉपिक इनहेरिटेंस: टॉपिक प्रविष्टियाँ समूह सेटिंग्स को इनहेरिट करती हैं जब तक कि ओवरराइड न किया जाए (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`)।
    
    टेम्पलेट कॉन्टेक्स्ट में शामिल हैं:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM थ्रेड व्यवहार:
    
    - `message_thread_id` वाले निजी चैट DM रूटिंग बनाए रखते हैं, लेकिन थ्रेड-अवेयर सेशन कुंजियों/रिप्लाई लक्ष्यों का उपयोग करते हैं।
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">    ### ऑडियो संदेश

    ```
    Telegram वॉइस नोट्स और ऑडियो फ़ाइलों में अंतर करता है।
    
    - डिफ़ॉल्ट: ऑडियो फ़ाइल व्यवहार
    - एजेंट के उत्तर में `[[audio_as_voice]]` टैग जोड़कर वॉइस-नोट के रूप में भेजने के लिए बाध्य करें
    
    संदेश क्रिया उदाहरण:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### वीडियो संदेश
    
    Telegram वीडियो फ़ाइलों और वीडियो नोट्स में अंतर करता है।
    
    संदेश क्रिया उदाहरण:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    वीडियो नोट्स कैप्शन का समर्थन नहीं करते; दिया गया संदेश टेक्स्ट अलग से भेजा जाता है।
    
    ### स्टिकर्स
    
    इनबाउंड स्टिकर हैंडलिंग:
    
    - स्थिर WEBP: डाउनलोड और प्रोसेस किया जाता है (प्लेसहोल्डर `<media:sticker>`)
    - एनिमेटेड TGS: छोड़ा जाता है
    - वीडियो WEBM: छोड़ा जाता है
    
    स्टिकर कॉन्टेक्स्ट फ़ील्ड्स:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    स्टिकर कैश फ़ाइल:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    स्टिकर्स का वर्णन एक बार (जहाँ संभव हो) किया जाता है और दोहराए जाने वाले विज़न कॉल को कम करने के लिए कैश किया जाता है।
    
    स्टिकर क्रियाएँ सक्षम करें:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    स्टिकर भेजें क्रिया:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    कैश किए गए स्टिकर्स खोजें:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">    Telegram रिएक्शन `message_reaction` अपडेट के रूप में आते हैं (संदेश पेलोड से अलग)।

    ```
    सक्षम होने पर, OpenClaw इस प्रकार के सिस्टम इवेंट्स को क्यू में डालता है:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    कॉन्फ़िग:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (डिफ़ॉल्ट: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (डिफ़ॉल्ट: `minimal`)
    
    नोट्स:
    
    - `own` का अर्थ है केवल बॉट द्वारा भेजे गए संदेशों पर उपयोगकर्ता रिएक्शन (भेजे गए संदेश कैश के माध्यम से सर्वोत्तम प्रयास)।
    - Telegram रिएक्शन अपडेट में थ्रेड ID प्रदान नहीं करता।
      - नॉन-फ़ोरम समूह समूह चैट सेशन में रूट होते हैं
      - फ़ोरम समूह समूह के सामान्य-टॉपिक सेशन (`:topic:1`) में रूट होते हैं, न कि मूल सटीक टॉपिक में
    
    पोलिंग/वेबहुक के लिए `allowed_updates` में `message_reaction` स्वतः शामिल होता है।
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` OpenClaw द्वारा इनबाउंड संदेश प्रोसेस करते समय एक स्वीकृति इमोजी भेजता है।

    ```
    रिज़ॉल्यूशन क्रम:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - एजेंट पहचान इमोजी फ़ॉलबैक (`agents.list[].identity.emoji`, अन्यथा "👀")
    
    नोट्स:
    
    - Telegram यूनिकोड इमोजी अपेक्षित करता है (उदाहरण के लिए "👀")।
    - किसी चैनल या अकाउंट के लिए रिएक्शन अक्षम करने हेतु `""` का उपयोग करें।
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">    चैनल कॉन्फ़िग राइट्स डिफ़ॉल्ट रूप से सक्षम हैं (`configWrites !== false`)।

    ```
    Telegram-ट्रिगर राइट्स में शामिल हैं:
    
    - समूह माइग्रेशन इवेंट्स (`migrate_to_chat_id`) ताकि `channels.telegram.groups` अपडेट हो सके
    - `/config set` और `/config unset` (कमांड सक्षम होना आवश्यक)
    
    अक्षम करें:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">    डिफ़ॉल्ट: लॉन्ग पोलिंग।

    ```
    वेबहुक मोड:
    
    - `channels.telegram.webhookUrl` सेट करें
    - `channels.telegram.webhookSecret` सेट करें (जब webhook URL सेट हो, तब आवश्यक)
    - वैकल्पिक `channels.telegram.webhookPath` (डिफ़ॉल्ट `/telegram-webhook`)
    - वैकल्पिक `channels.telegram.webhookHost` (डिफ़ॉल्ट `127.0.0.1`)
    
    वेबहुक मोड के लिए डिफ़ॉल्ट लोकल लिस्नर `127.0.0.1:8787` पर बाइंड होता है।
    
    यदि आपका पब्लिक एंडपॉइंट अलग है, तो सामने एक रिवर्स प्रॉक्सी लगाएँ और `webhookUrl` को पब्लिक URL पर पॉइंट करें।
    जब आपको जानबूझकर बाहरी इनग्रेस की आवश्यकता हो, तो `webhookHost` (उदाहरण `0.0.0.0`) सेट करें।
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` का डिफ़ॉल्ट 4000 है।
    - `channels.telegram.chunkMode="newline"` लंबाई के आधार पर विभाजन से पहले पैराग्राफ सीमाओं (खाली पंक्तियों) को प्राथमिकता देता है।
    - `channels.telegram.mediaMaxMb` (डिफ़ॉल्ट 5) इनबाउंड Telegram मीडिया डाउनलोड/प्रोसेसिंग आकार को सीमित करता है।
    - `channels.telegram.timeoutSeconds` Telegram API क्लाइंट टाइमआउट को ओवरराइड करता है (यदि सेट नहीं है, तो grammY डिफ़ॉल्ट लागू होता है)।
    - समूह कॉन्टेक्स्ट इतिहास `channels.telegram.historyLimit` या `messages.groupChat.historyLimit` (डिफ़ॉल्ट 50) का उपयोग करता है; `0` अक्षम करता है।
    - DM इतिहास नियंत्रण:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - आउटबाउंड Telegram API रीट्राई `channels.telegram.retry` के माध्यम से कॉन्फ़िगर किए जा सकते हैं।

    ```
    CLI सेंड टार्गेट संख्यात्मक चैट ID या यूज़रनेम हो सकता है:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - यदि `requireMention=false`, तो Telegram प्राइवेसी मोड पूर्ण दृश्यता की अनुमति देना चाहिए।
      - BotFather: `/setprivacy` -> Disable
      - फिर बॉट को समूह से हटाएँ और दोबारा जोड़ें
    - `openclaw channels status` चेतावनी देता है जब कॉन्फ़िग बिना मेंशन किए गए समूह संदेशों की अपेक्षा करता है।
    - `openclaw channels status --probe` स्पष्ट संख्यात्मक समूह IDs की जाँच कर सकता है; वाइल्डकार्ड `"*"` की सदस्यता जाँची नहीं जा सकती।
    - त्वरित सेशन परीक्षण: `/activation always`।
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - जब `channels.telegram.groups` मौजूद हो, तो समूह सूचीबद्ध होना चाहिए (या `"*"` शामिल होना चाहिए)
    - समूह में बॉट की सदस्यता सत्यापित करें
    - स्किप कारणों के लिए लॉग्स देखें: `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - अपने सेंडर आइडेंटिटी को अधिकृत करें (पेयरिंग और/या संख्यात्मक `allowFrom`)
    - समूह नीति `open` होने पर भी कमांड ऑथराइज़ेशन लागू रहता है
    - `setMyCommands failed` आमतौर पर `api.telegram.org` तक DNS/HTTPS पहुँच समस्याओं को दर्शाता है
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + कस्टम fetch/proxy AbortSignal प्रकारों के असंगत होने पर तत्काल अबॉर्ट व्यवहार ट्रिगर कर सकता है।
    - कुछ होस्ट पहले `api.telegram.org` को IPv6 पर रेज़ॉल्व करते हैं; टूटी हुई IPv6 एग्रेस के कारण Telegram API में रुक-रुक कर विफलताएँ हो सकती हैं।
    - DNS उत्तर सत्यापित करें:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Telegram टैग्स के माध्यम से वैकल्पिक threaded replies का समर्थन करता है:

## Telegram कॉन्फ़िग संदर्भ संकेत

`channels.telegram.replyToMode` द्वारा नियंत्रित:

- `first` (डिफ़ॉल्ट), `all`, `off`।

- `channels.telegram.botToken`: बॉट टोकन (BotFather)।

- `channels.telegram.tokenFile`: फ़ाइल पथ से टोकन पढ़ें।

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (डिफ़ॉल्ट: pairing)।

- `channels.telegram.allowFrom`: DM अनुमति सूची (संख्यात्मक Telegram उपयोगकर्ता IDs)। `open` requires `"*"`. `openclaw doctor --fix` पुराने `@username` प्रविष्टियों को IDs में परिवर्तित कर सकता है।

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (डिफ़ॉल्ट: allowlist)।

- `channels.telegram.groupAllowFrom`: समूह प्रेषक अनुमति सूची (संख्यात्मक Telegram उपयोगकर्ता IDs)। `openclaw doctor --fix` पुराने `@username` प्रविष्टियों को IDs में परिवर्तित कर सकता है।

- `channels.telegram.groups`: प्रति-समूह defaults + allowlist (global defaults के लिए `"*"` का उपयोग करें)।
  - `channels.telegram.groups.<id>.groupPolicy`: per-group override for groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: mention gating default.
  - `channels.telegram.groups.<id>.skills`: skill filter (omit = all skills, empty = none).
  - `channels.telegram.groups.<id>.allowFrom`: per-group sender allowlist override.
  - `channels.telegram.groups.<id>.systemPrompt`: extra system prompt for the group.
  - `channels.telegram.groups.<id>.enabled`: disable the group when `false`.
  - `channels.telegram.groups.<id>11. .topics.<threadId>.*`: per-topic overrides (same fields as group).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: per-topic override for groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (डिफ़ॉल्ट: allowlist)।

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.

- `channels.telegram.replyToMode`: `off | first | all` (डिफ़ॉल्ट: `off`)।

- `channels.telegram.textChunkLimit`: outbound chunk size (chars)।

- `channels.telegram.chunkMode`: `length` (डिफ़ॉल्ट) या लंबाई chunking से पहले blank lines (paragraph boundaries) पर विभाजित करने के लिए `newline`।

- `channels.telegram.linkPreview`: outbound संदेशों के लिए link previews toggle करें (डिफ़ॉल्ट: true)।

- `channels.telegram.streamMode`: `off | partial | block` (लाइव स्ट्रीम पूर्वावलोकन)।

- `channels.telegram.mediaMaxMb`: inbound/outbound media cap (MB)।

- `channels.telegram.retry`: outbound Telegram API calls के लिए retry policy (attempts, minDelayMs, maxDelayMs, jitter)।

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: Bot API calls के लिए proxy URL (SOCKS/HTTP)।

- `channels.telegram.webhookUrl`: webhook मोड सक्षम करें (इसके लिए `channels.telegram.webhookSecret` आवश्यक)।

- `channels.telegram.webhookSecret`: webhook secret (जब webhookUrl सेट हो तो आवश्यक)।

- `channels.telegram.webhookPath`: local webhook path (डिफ़ॉल्ट `/telegram-webhook`)।

- `channels.telegram.webhookHost`: स्थानीय webhook बाइंड होस्ट (डिफ़ॉल्ट `127.0.0.1`)।

- `channels.telegram.actions.reactions`: Telegram tool reactions gate करें।

- `channels.telegram.actions.sendMessage`: Telegram tool message sends gate करें।

- `channels.telegram.actions.deleteMessage`: Telegram tool message deletes gate करें।

- `channels.telegram.actions.sticker`: Telegram sticker actions — send और search gate करें (डिफ़ॉल्ट: false)।

- `channels.telegram.reactionNotifications`: `off | own | all` — नियंत्रित करें कि कौन-सी reactions system events ट्रिगर करें (डिफ़ॉल्ट: सेट न होने पर `own`)।

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — एजेंट की reaction क्षमता नियंत्रित करें (डिफ़ॉल्ट: सेट न होने पर `minimal`)।

- [कॉन्फ़िगरेशन संदर्भ - Telegram](/gateway/configuration-reference#telegram)

Telegram-विशिष्ट उच्च-संकेत फ़ील्ड्स:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- access control: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- streaming: `streamMode` (पूर्वावलोकन), `draftChunk`, `blockStreaming`
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reactions: `reactionNotifications`, `reactionLevel`
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## संबंधित

- `[[audio_as_voice]]` — audio को file के बजाय voice note के रूप में भेजें।
- [चैनल रूटिंग](/channels/channel-routing)
- [समस्या निवारण](/channels/troubleshooting)

