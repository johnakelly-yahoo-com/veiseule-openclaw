---
summary: "Discord बॉट समर्थन स्थिति, क्षमताएँ और विन्यास"
read_when:
  - Discord चैनल सुविधाओं पर काम करते समय
title: "Discord"
---

# Discord (Bot API)

स्थिति: आधिकारिक Discord बॉट गेटवे के माध्यम से DM और guild टेक्स्ट चैनलों के लिए तैयार।

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DMs डिफ़ॉल्ट रूप से pairing मोड में होते हैं।
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    नेटिव कमांड व्यवहार और कमांड कैटलॉग।
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    क्रॉस-चैनल डायग्नोस्टिक्स और रिपेयर फ़्लो।
  
</Card>
</CardGroup>

## त्वरित सेटअप

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Discord Developer Portal में एक application बनाएं, एक bot जोड़ें, फिर सक्षम करें:


    ```
    - **Message Content Intent**
    - **Server Members Intent** (role allowlists और role-आधारित routing के लिए आवश्यक; name-to-ID allowlist matching के लिए अनुशंसित)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    डिफ़ॉल्ट खाते के लिए Env fallback:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    अपने सर्वर पर संदेश अनुमतियों के साथ बॉट को आमंत्रित करें।

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    पेयरिंग कोड 1 घंटे के बाद समाप्त हो जाते हैं।
    ```

  
</Step>
</Steps>

<Note>
टोकन रेज़ोल्यूशन अकाउंट-अवेयर है। कॉन्फ़िग में दिए गए टोकन मान, env fallback पर प्राथमिकता लेते हैं। `DISCORD_BOT_TOKEN` केवल डिफ़ॉल्ट अकाउंट के लिए उपयोग होता है।
</Note>

## रनटाइम मॉडल

- Gateway, Discord कनेक्शन का स्वामित्व रखता है।
- रिप्लाई रूटिंग निर्धारक है: Discord से आने वाले संदेशों के उत्तर Discord पर ही भेजे जाते हैं।
- डिफ़ॉल्ट रूप से (`session.dmScope=main`), डायरेक्ट चैट एजेंट के मुख्य सेशन (`agent:main:main`) को साझा करती हैं।
- Guild चैनल अलग-थलग सेशन कीज़ होते हैं (`agent:<agentId>:discord:channel:<channelId>`)।
- Group DM डिफ़ॉल्ट रूप से अनदेखे किए जाते हैं (`channels.discord.dm.groupEnabled=false`)।
- नेटिव स्लैश कमांड अलग-थलग कमांड सेशन (`agent:<agentId>:discord:slash:<userId>`) में चलते हैं, जबकि रूट की गई वार्तालाप सेशन के लिए `CommandTargetSessionKey` भी साथ ले जाते हैं।

## एक्सेस कंट्रोल और रूटिंग

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` DM एक्सेस को नियंत्रित करता है (लेगेसी: `channels.discord.dm.policy`):

    ```
    - `pairing` (डिफ़ॉल्ट)
    - `allowlist`
    - `open` (इसके लिए `channels.discord.allowFrom` में `"*"` शामिल होना चाहिए; लेगेसी: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    यदि DM पॉलिसी open नहीं है, तो अज्ञात उपयोगकर्ताओं को ब्लॉक कर दिया जाता है (या `pairing` मोड में पेयरिंग के लिए प्रॉम्प्ट किया जाता है)।
    
    डिलीवरी के लिए DM लक्ष्य प्रारूप:
    
    - `user:<id>`
    - `<@id>` मेंशन
    
    साधारण संख्यात्मक IDs अस्पष्ट मानी जाती हैं और अस्वीकार कर दी जाती हैं, जब तक कि स्पष्ट user/channel target kind प्रदान न किया गया हो।
    ```

  
</Tab>

  <Tab title="Guild policy">
    Guild हैंडलिंग को `channels.discord.groupPolicy` द्वारा नियंत्रित किया जाता है:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    जब `channels.discord` मौजूद हो, तो सुरक्षित बेसलाइन `allowlist` होती है।
    
    `allowlist` व्यवहार:
    
    - guild का मिलान `channels.discord.guilds` से होना चाहिए (`id` वरीयता, slug स्वीकार्य)
    - वैकल्पिक प्रेषक allowlists: `users` (IDs या नाम) और `roles` (केवल role IDs); यदि इनमें से कोई भी कॉन्फ़िगर है, तो प्रेषक `users` या `roles` में से किसी से मेल खाने पर अनुमति प्राप्त करता है
    - यदि किसी guild में `channels` कॉन्फ़िगर हैं, तो सूचीबद्ध न किए गए चैनल अस्वीकृत होंगे
    - यदि किसी guild में `channels` ब्लॉक नहीं है, तो उस allowlisted guild के सभी चैनल अनुमत होंगे
    
    उदाहरण:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    यदि आप केवल `DISCORD_BOT_TOKEN` सेट करते हैं और `channels.discord` ब्लॉक नहीं बनाते हैं, तो रनटाइम fallback `groupPolicy="open"` होता है (लॉग्स में एक चेतावनी के साथ)।
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Guild संदेश डिफ़ॉल्ट रूप से मेंशन-गेटेड होते हैं।

    ```
    मेंशन डिटेक्शन में शामिल है:
    
    - स्पष्ट बॉट मेंशन
    - कॉन्फ़िगर किए गए मेंशन पैटर्न (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - समर्थित मामलों में बॉट को implicit reply व्यवहार
    
    `requireMention` प्रत्येक guild/channel के लिए कॉन्फ़िगर किया जाता है (`channels.discord.guilds...`)।
    
    Group DMs:
    
    - डिफ़ॉल्ट: अनदेखा (`dm.groupEnabled=false`)
    - वैकल्पिक allowlist `dm.groupChannels` के माध्यम से (channel IDs या slugs)
    ```

  
</Tab>
</Tabs>

### भूमिका-आधारित एजेंट रूटिंग

Discord guild सदस्यों को role ID के आधार पर विभिन्न एजेंटों की ओर रूट करने के लिए `bindings[].match.roles` का उपयोग करें। भूमिका-आधारित bindings केवल role IDs स्वीकार करते हैं और peer या parent-peer bindings के बाद तथा केवल-guild bindings से पहले मूल्यांकित किए जाते हैं। यदि कोई binding अन्य match फ़ील्ड्स भी सेट करता है (उदाहरण के लिए `peer` + `guildId` + `roles`), तो सभी कॉन्फ़िगर किए गए फ़ील्ड्स का मिलान होना आवश्यक है।

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal सेटअप

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. बॉट टोकन कॉपी करें
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    **Bot -> Privileged Gateway Intents** में, सक्षम करें:

    ```
    - Message Content Intent
    - Server Members Intent (अनुशंसित)
    
    Presence intent वैकल्पिक है और केवल तभी आवश्यक है जब आप presence अपडेट प्राप्त करना चाहते हों। बॉट presence सेट करना (`setPresence`) के लिए सदस्यों के presence अपडेट सक्षम करना आवश्यक नहीं है।
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL जनरेटर:

    ```
    - scopes: `bot`, `applications.commands`
    
    सामान्य बेसलाइन अनुमतियाँ:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (वैकल्पिक)
    
    जब तक स्पष्ट रूप से आवश्यक न हो, `Administrator` से बचें।
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Discord Developer Mode सक्षम करें, फिर कॉपी करें:

    ```
    - server ID
    - channel ID
    - user ID
    
    विश्वसनीय ऑडिट और प्रोब के लिए OpenClaw कॉन्फ़िग में संख्यात्मक IDs को प्राथमिकता दें।
    ```

  
</Accordion>
</AccordionGroup>

## नेटिव कमांड्स और कमांड प्रमाणीकरण

- `commands.native` डिफ़ॉल्ट रूप से `"auto"` होता है और Discord के लिए सक्षम रहता है।
- प्रति-चैनल ओवरराइड: `channels.discord.commands.native`।
- `commands.native=false` पहले से पंजीकृत Discord नेटिव कमांड्स को स्पष्ट रूप से हटा देता है।
- नेटिव कमांड प्रमाणीकरण सामान्य संदेश हैंडलिंग की तरह ही Discord allowlists/policies का उपयोग करता है।
- कमांड्स Discord UI में उन उपयोगकर्ताओं को दिखाई दे सकते हैं जो अधिकृत नहीं हैं; लेकिन निष्पादन के समय OpenClaw प्रमाणीकरण लागू होता है और "not authorized" लौटाया जाता है।

कमांड कैटलॉग और व्यवहार के लिए [Slash commands](/tools/slash-commands) देखें।

## Retry policy

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord एजेंट आउटपुट में reply टैग्स का समर्थन करता है:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    `channels.discord.replyToMode` द्वारा नियंत्रित:
    
    - `off` (डिफ़ॉल्ट)
    - `first`
    - `all`
    
    नोट: `off` अप्रत्यक्ष reply थ्रेडिंग को अक्षम करता है। स्पष्ट `[[reply_to_*]]` टैग्स अभी भी मान्य रहते हैं।
    
    Message IDs को context/history में उपलब्ध कराया जाता है ताकि एजेंट विशेष संदेशों को लक्षित कर सकें।
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild history संदर्भ:

    ```
    - `channels.discord.historyLimit` डिफ़ॉल्ट `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` अक्षम करता है
    
    DM history नियंत्रण:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread व्यवहार:
    
    - Discord threads को channel sessions के रूप में रूट किया जाता है
    - parent thread metadata का उपयोग parent-session लिंकिंग के लिए किया जा सकता है
    - thread कॉन्फ़िग parent channel कॉन्फ़िग को इनहेरिट करता है जब तक कि thread-विशिष्ट एंट्री मौजूद न हो
    
    Channel topics को **untrusted** context के रूप में इंजेक्ट किया जाता है (system prompt के रूप में नहीं)।
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    प्रति-guild reaction सूचना मोड:

    ```
    - `off`
    - `own` (डिफ़ॉल्ट)
    - `all`
    - `allowlist` ( `guilds.<id>.users` का उपयोग करता है )
    
    Reaction events को system events में बदला जाता है और रूट किए गए Discord session से जोड़ा जाता है।
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` OpenClaw द्वारा इनबाउंड संदेश प्रोसेस करते समय एक acknowledgment emoji भेजता है।

    ```
    Resolution क्रम:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, अन्यथा "👀")
    
    नोट्स:
    
    - Discord unicode emoji या custom emoji नाम स्वीकार करता है।
    - किसी चैनल या अकाउंट के लिए reaction अक्षम करने हेतु `""` का उपयोग करें।
    ```

  
</Accordion>

  <Accordion title="Config writes">
    चैनल-प्रारंभित कॉन्फ़िग राइट्स डिफ़ॉल्ट रूप से सक्षम हैं।

    ```
    यह `/config set|unset` फ्लो को प्रभावित करता है (जब कमांड फीचर्स सक्षम हों)।
    
    अक्षम करें:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    `channels.discord.proxy` के साथ Discord gateway WebSocket ट्रैफ़िक को HTTP(S) प्रॉक्सी के माध्यम से रूट करें।

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    ```
    प्रति-अकाउंट ओवरराइड:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    प्रॉक्सी किए गए संदेशों को सिस्टम सदस्य पहचान से मैप करने के लिए PluralKit resolution सक्षम करें:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // वैकल्पिक; निजी सिस्टम्स के लिए आवश्यक
      },
    },
  },
}
```

    ```
    नोट्स:
    
    - allowlists `pk:<memberId>` का उपयोग कर सकती हैं
    - सदस्य display names का मिलान नाम/slug से किया जाता है
    - lookups मूल message ID का उपयोग करते हैं और समय-सीमा से प्रतिबंधित होते हैं
    - यदि lookup विफल होता है, तो प्रॉक्सी किए गए संदेशों को bot संदेश माना जाता है और हटा दिया जाता है जब तक कि `allowBots=true` न हो
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence अपडेट केवल तब लागू होते हैं जब आप status या activity फ़ील्ड सेट करते हैं।

    ```
    केवल status उदाहरण:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    Activity उदाहरण (custom status डिफ़ॉल्ट activity type है):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Streaming उदाहरण:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    Activity type मैप:
    
    - 0: Playing
    - 1: Streaming (`activityUrl` आवश्यक)
    - 2: Listening
    - 3: Watching
    - 4: Custom (activity टेक्स्ट को status state के रूप में उपयोग करता है; emoji वैकल्पिक है)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord DMs में बटन-आधारित exec approvals का समर्थन करता है और वैकल्पिक रूप से मूल चैनल में approval prompts पोस्ट कर सकता है।

    ```
    कॉन्फ़िग पथ:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, डिफ़ॉल्ट: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    जब `target` `channel` या `both` होता है, तो approval prompt चैनल में दिखाई देता है। केवल कॉन्फ़िगर किए गए approvers ही बटनों का उपयोग कर सकते हैं; अन्य उपयोगकर्ताओं को एक ephemeral denial प्राप्त होता है। Approval prompts में कमांड टेक्स्ट शामिल होता है, इसलिए चैनल डिलीवरी केवल विश्वसनीय चैनलों में ही सक्षम करें। यदि session key से चैनल ID प्राप्त नहीं की जा सकती, तो OpenClaw DM डिलीवरी पर वापस लौटता है।
    
    यदि approvals अज्ञात approval IDs के साथ विफल होते हैं, तो approver सूची और फीचर enablement सत्यापित करें।
    
    संबंधित दस्तावेज़: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## टूल्स और एक्शन गेट्स

Discord संदेश एक्शंस में messaging, channel admin, moderation, presence और metadata एक्शंस शामिल हैं।

मुख्य उदाहरण:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- presence: `setPresence`

Action gates `channels.discord.actions.*` के अंतर्गत होते हैं।

डिफ़ॉल्ट गेट व्यवहार:

| Action group                                                                                                                                                             | Default  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

OpenClaw exec approvals और cross-context markers के लिए Discord components v2 का उपयोग करता है। Discord message actions कस्टम UI के लिए `components` भी स्वीकार कर सकते हैं (उन्नत; Carbon component instances आवश्यक), जबकि legacy `embeds` अभी भी उपलब्ध हैं लेकिन अनुशंसित नहीं हैं।

- `channels.discord.ui.components.accentColor` Discord component containers द्वारा उपयोग किए जाने वाले accent color (hex) को सेट करता है।
- प्रत्येक account के लिए `channels.discord.accounts.<id>.ui.components.accentColor` सेट करें।
- जब components v2 मौजूद हों तो `embeds` को अनदेखा कर दिया जाता है।

उदाहरण:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Voice messages

Discord voice messages में waveform preview दिखाई देता है और OGG/Opus ऑडियो के साथ metadata आवश्यक होता है। OpenClaw waveform स्वतः उत्पन्न करता है, लेकिन ऑडियो फ़ाइलों की जाँच और रूपांतरण के लिए gateway host पर `ffmpeg` और `ffprobe` उपलब्ध होने चाहिए।

आवश्यकताएँ और सीमाएँ:

- **स्थानीय फ़ाइल पथ** प्रदान करें (URLs अस्वीकार किए जाते हैं)।
- टेक्स्ट सामग्री शामिल न करें (Discord एक ही payload में टेक्स्ट + voice message की अनुमति नहीं देता)।
- कोई भी ऑडियो फ़ॉर्मेट स्वीकार्य है; आवश्यकता होने पर OpenClaw इसे OGG/Opus में बदल देता है।

उदाहरण:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## समस्या-निवारण

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - Message Content Intent सक्षम करें
    - जब आप user/member resolution पर निर्भर हों तो Server Members Intent सक्षम करें
    - intents बदलने के बाद gateway पुनः प्रारंभ करें
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - `groupPolicy` सत्यापित करें
    - `channels.discord.guilds` के अंतर्गत guild allowlist सत्यापित करें
    - यदि guild `channels` map मौजूद है, तो केवल सूचीबद्ध channels की अनुमति है
    - `requireMention` व्यवहार और mention patterns सत्यापित करें
    
    Useful checks:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    सामान्य कारण:

    ```
    - मिलती-जुलती guild/channel allowlist के बिना `groupPolicy="allowlist"`
    - `requireMention` गलत स्थान पर कॉन्फ़िगर किया गया है (यह `channels.discord.guilds` या channel entry के अंतर्गत होना चाहिए)
    - sender guild/channel `users` allowlist द्वारा ब्लॉक किया गया है
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` permission checks केवल numeric channel IDs के लिए काम करते हैं।

    ```
    यदि आप slug keys का उपयोग करते हैं, तो runtime matching फिर भी काम कर सकता है, लेकिन probe पूरी तरह से permissions सत्यापित नहीं कर सकता।
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM अक्षम: `channels.discord.dm.enabled=false`
    - DM policy अक्षम: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - `pairing` mode में pairing approval की प्रतीक्षा में
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    डिफ़ॉल्ट रूप से bot द्वारा लिखे गए संदेशों को अनदेखा किया जाता है।

    ```
    यदि आप `channels.discord.allowBots=true` सेट करते हैं, तो loop व्यवहार से बचने के लिए सख्त mention और allowlist नियमों का उपयोग करें।
    ```

  
</Accordion>
</AccordionGroup>

## Configuration reference pointers

प्राथमिक संदर्भ:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

उच्च-प्राथमिकता वाले Discord फ़ील्ड्स:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## सुरक्षा और संचालन

- बॉट टोकन को सीक्रेट की तरह संभालें (सुपरवाइज़्ड वातावरण में `DISCORD_BOT_TOKEN` को प्राथमिकता दें)।
- न्यूनतम-आवश्यक Discord अनुमतियाँ प्रदान करें।
- यदि command deploy/state पुराना है, तो gateway को पुनः शुरू करें और `openclaw channels status --probe` के साथ दोबारा जाँच करें।

## संबंधित

- [पेयरिंग](/channels/pairing)
- [चैनल रूटिंग](/channels/channel-routing)
- [समस्या निवारण](/channels/troubleshooting)
- [स्लैश कमांड्स](/tools/slash-commands)

