---
summary: "Socket या HTTP webhook मोड के लिए Slack सेटअप"
read_when:
  - Slack सेटअप करना या Slack socket/HTTP मोड का डिबग करना
title: "Slack"
---

# Slack

स्थिति: Slack ऐप इंटीग्रेशन के माध्यम से DMs + चैनलों के लिए प्रोडक्शन-रेडी। डिफ़ॉल्ट मोड Socket Mode है; HTTP Events API मोड भी समर्थित है।

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs डिफ़ॉल्ट रूप से pairing मोड में रहते हैं।
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    नेटिव कमांड व्यवहार और कमांड कैटलॉग।
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    क्रॉस-चैनल डायग्नोस्टिक्स और रिपेयर प्लेबुक्स।
  
</Card>
</CardGroup>

## त्वरित सेटअप

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        Slack ऐप सेटिंग्स में:


        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Env फॉलबैक (केवल डिफ़ॉल्ट अकाउंट):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="ऐप इवेंट्स सब्सक्राइब करें">
          निम्न के लिए bot इवेंट्स सब्सक्राइब करें:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          DMs के लिए App Home में **Messages Tab** भी सक्षम करें।
        
</Step>
      
        <Step title="gateway शुरू करें">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Step>
      
        <Step title="मल्टी-अकाउंट HTTP के लिए यूनिक webhook पाथ्स का उपयोग करें">
          प्रति-अकाउंट HTTP मोड समर्थित है।
      
          प्रत्येक अकाउंट को अलग `webhookPath` दें ताकि रजिस्ट्रेशन टकराएँ नहीं।
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## टोकन मॉडल

- Socket Mode के लिए `botToken` + `appToken` आवश्यक हैं।
- HTTP मोड के लिए `botToken` + `signingSecret` आवश्यक हैं।
- कॉन्फ़िग टोकन, env फॉलबैक को ओवरराइड करते हैं।
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env फॉलबैक केवल डिफ़ॉल्ट अकाउंट पर लागू होता है।
- `userToken` (`xoxp-...`) केवल कॉन्फ़िग में उपलब्ध है (कोई env फॉलबैक नहीं) और डिफ़ॉल्ट रूप से केवल-पढ़ने योग्य व्यवहार (`userTokenReadOnly: true`) पर सेट रहता है।
- वैकल्पिक: यदि आप चाहते हैं कि आउटगोइंग संदेश सक्रिय एजेंट पहचान (कस्टम `username` और आइकन) का उपयोग करें, तो `chat:write.customize` जोड़ें। `icon_emoji` में `:emoji_name:` सिंटैक्स का उपयोग होता है।

<Tip>
actions/directory रीड्स के लिए, कॉन्फ़िग होने पर user token को प्राथमिकता दी जा सकती है। लिखने के लिए bot token प्राथमिक रहता है; user-token से लिखना केवल तब अनुमति है जब `userTokenReadOnly: false` हो और bot token उपलब्ध न हो।
</Tip>

## टोकन उपयोग

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` DM एक्सेस को नियंत्रित करता है (लेगेसी: `channels.slack.dm.policy`):

    ```
    - `pairing` (डिफ़ॉल्ट)
    - `allowlist`
    - `open` (इसके लिए `channels.slack.allowFrom` में `"*"` शामिल होना चाहिए; लेगेसी: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM फ्लैग्स:
    
    - `dm.enabled` (डिफ़ॉल्ट true)
    - `channels.slack.allowFrom` (प्राथमिक)
    - `dm.allowFrom` (लेगेसी)
    - `dm.groupEnabled` (ग्रुप DMs डिफ़ॉल्ट false)
    - `dm.groupChannels` (वैकल्पिक MPIM allowlist)
    
    DMs में pairing के लिए `openclaw pairing approve slack <code>` का उपयोग करें।
    ```

  
</Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` चैनल हैंडलिंग को नियंत्रित करता है:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    चैनल allowlist `channels.slack.channels` के अंतर्गत स्थित है।
    
    रनटाइम नोट: यदि `channels.slack` पूरी तरह अनुपस्थित है (केवल env सेटअप) और `channels.defaults.groupPolicy` सेट नहीं है, तो रनटाइम `groupPolicy="open"` पर फॉलबैक करता है और एक चेतावनी लॉग करता है।
    
    नाम/ID रेज़ोल्यूशन:
    
    - चैनल allowlist एंट्रीज़ और DM allowlist एंट्रीज़ स्टार्टअप पर रेज़ॉल्व की जाती हैं जब टोकन एक्सेस अनुमति देता है
    - अनरिज़ॉल्व्ड एंट्रीज़ को कॉन्फ़िग के अनुसार ही रखा जाता है
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    चैनल संदेश डिफ़ॉल्ट रूप से mention-गेटेड होते हैं।

    ```
    Mention स्रोत:
    
    - स्पष्ट ऐप mention (`<@botId>`)
    - mention regex पैटर्न्स (`agents.list[].groupChat.mentionPatterns`, फॉलबैक `messages.groupChat.mentionPatterns`)
    - बॉट-थ्रेड के लिए implicit reply व्यवहार
    
    प्रति-चैनल नियंत्रण (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## कमांड्स और slash व्यवहार

- Slack के लिए नेटिव कमांड auto-mode **बंद** है (`commands.native: "auto"` Slack नेटिव कमांड्स को सक्षम नहीं करता)।
- नेटिव Slack कमांड हैंडलर्स को `channels.slack.commands.native: true` (या ग्लोबल `commands.native: true`) से सक्षम करें।
- जब नेटिव कमांड्स सक्षम हों, तो Slack में संबंधित slash कमांड्स (`/<command>` नाम) रजिस्टर करें।
- यदि नेटिव कमांड्स सक्षम नहीं हैं, तो आप `channels.slack.slashCommand` के माध्यम से एकल कॉन्फ़िगर किया गया slash कमांड चला सकते हैं।

डिफ़ॉल्ट slash कमांड सेटिंग्स:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash सत्र अलग-अलग कुंजियों का उपयोग करते हैं:

- `agent:<agentId>:slack:slash:<userId>`

और फिर भी कमांड निष्पादन को लक्ष्य वार्तालाप सत्र (`CommandTargetSessionKey`) के विरुद्ध रूट करते हैं।

## थ्रेडिंग, सत्र और उत्तर टैग

- DMs को `direct` के रूप में रूट किया जाता है; चैनल्स को `channel`; MPIMs को `group`।
- डिफ़ॉल्ट `session.dmScope=main` के साथ, Slack DMs एजेंट के मुख्य सत्र में समाहित हो जाते हैं।
- चैनल सत्र: `agent:<agentId>:slack:channel:<channelId>`।
- थ्रेड उत्तर लागू होने पर थ्रेड सत्र प्रत्यय (`:thread:<threadTs>`) बना सकते हैं।
- `channels.slack.thread.historyScope` का डिफ़ॉल्ट `thread` है; `thread.inheritParent` का डिफ़ॉल्ट `false` है।
- `channels.slack.thread.initialHistoryLimit` यह नियंत्रित करता है कि नया थ्रेड सत्र शुरू होने पर मौजूदा थ्रेड संदेशों में से कितने प्राप्त किए जाएँ (डिफ़ॉल्ट `20`; अक्षम करने के लिए `0` सेट करें)।

उत्तर थ्रेडिंग नियंत्रण:

- `channels.slack.replyToMode`: `off|first|all` (डिफ़ॉल्ट `off`)
- `channels.slack.replyToModeByChatType`: प्रत्येक `direct|group|channel` के लिए
- डायरेक्ट चैट्स के लिए पुराना फॉलबैक: `channels.slack.dm.replyToMode`

यदि आप `channels.slack.userToken` कॉन्फ़िगर करते हैं, तो इन्हें **User Token Scopes** के अंतर्गत जोड़ें।

- `channels:history`, `groups:history`, `im:history`, `mpim:history`
- `channels:read`, `groups:read`, `im:read`, `mpim:read`

नोट: `replyToMode="off"` अप्रत्यक्ष उत्तर थ्रेडिंग को अक्षम करता है। स्पष्ट `[[reply_to_*]]` टैग अभी भी मान्य रहेंगे।

## मीडिया, चंकिंग और डिलीवरी

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack फ़ाइल अटैचमेंट्स को Slack-होस्टेड निजी URLs (टोकन-प्रमाणित अनुरोध प्रवाह) से डाउनलोड किया जाता है और यदि प्राप्ति सफल हो तथा आकार सीमाएँ अनुमति दें तो मीडिया स्टोर में लिखा जाता है।

    ```
    रनटाइम इनबाउंड आकार सीमा डिफ़ॉल्ट रूप से `20MB` है, जब तक कि `channels.slack.mediaMaxMb` द्वारा ओवरराइड न किया जाए।
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - टेक्स्ट चंक्स `channels.slack.textChunkLimit` (डिफ़ॉल्ट 4000) का उपयोग करते हैं
    - `channels.slack.chunkMode="newline"` पैराग्राफ-प्रथम विभाजन सक्षम करता है
    - फ़ाइल भेजने में Slack upload APIs का उपयोग होता है और इसमें थ्रेड उत्तर (`thread_ts`) शामिल हो सकते हैं
    - आउटबाउंड मीडिया सीमा कॉन्फ़िगर होने पर `channels.slack.mediaMaxMb` का पालन करती है; अन्यथा चैनल भेजने में मीडिया पाइपलाइन से MIME-काइंड डिफ़ॉल्ट्स उपयोग होते हैं
  
</Accordion>

  <Accordion title="Delivery targets">
    वांछित स्पष्ट लक्ष्य:

    ```
    - DMs के लिए `user:<id>`
    - चैनल्स के लिए `channel:<id>`
    
    Slack DMs, user targets पर भेजते समय Slack conversation APIs के माध्यम से खोले जाते हैं।
    ```

  
</Accordion>
</AccordionGroup>

## क्रियाएँ और गेट्स

Slack क्रियाएँ `channels.slack.actions.*` द्वारा नियंत्रित होती हैं।

वर्तमान Slack टूलिंग में उपलब्ध क्रिया समूह:

| Mode       | Default |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## इवेंट्स और परिचालन व्यवहार

- संदेश संपादन/हटाना/थ्रेड प्रसारण को सिस्टम इवेंट्स में मैप किया जाता है।
- रिएक्शन जोड़ने/हटाने की घटनाओं को सिस्टम इवेंट्स में मैप किया जाता है।
- सदस्य जुड़ना/छोड़ना, चैनल बनना/नाम बदलना, और पिन जोड़ना/हटाना जैसी घटनाओं को सिस्टम इवेंट्स में मैप किया जाता है।
- `channel_id_changed`, जब `configWrites` सक्षम हो, तो चैनल कॉन्फ़िग कुंजियों को माइग्रेट कर सकता है।
- चैनल का विषय/उद्देश्य मेटाडेटा अविश्वसनीय संदर्भ के रूप में माना जाता है और इसे रूटिंग संदर्भ में इंजेक्ट किया जा सकता है।

## Ack रिएक्शन

`ackReaction` इनबाउंड संदेश को प्रोसेस करते समय OpenClaw द्वारा एक स्वीकृति इमोजी भेजता है।

समर्थित चैट प्रकार:

- `channels.slack.accounts.<accountId>`.ackReaction\`
- `group`: group DMs / MPIMs (Slack `mpim`)
- `channel`: मानक चैनल (public/private)
- एजेंट पहचान इमोजी फॉलबैक (`agents.list[].identity.emoji`, अन्यथा "👀")

प्राथमिकता क्रम:

- Slack शॉर्टकोड्स की अपेक्षा करता है (उदाहरण के लिए `"eyes"`)।
- `replyToMode`

## मैनिफेस्ट और स्कोप चेकलिस्ट

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "OpenClaw के लिए Slack कनेक्टर"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "OpenClaw को एक संदेश भेजें",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    यदि आप `channels.slack.userToken` कॉन्फ़िगर करते हैं, तो सामान्य read स्कोप होते हैं:

    ```
    {
      channels: {
        slack: {
          replyToMode: "off",
          replyToModeByChatType: { group: "first" },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## समस्या-निवारण

<AccordionGroup>
  <Accordion title="No replies in channels">
    क्रम में जाँचें:

    ```
    - `groupPolicy`
    - चैनल allowlist (`channels.slack.channels`)
    - `requireMention`
    - प्रति-चैनल `users` allowlist
    
    उपयोगी कमांड:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    जाँचें:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (या पुराना `channels.slack.dm.policy`)
    - pairing अनुमोदन / allowlist प्रविष्टियाँ
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Slack ऐप सेटिंग्स में bot + app टोकन और Socket Mode सक्षम है या नहीं, इसकी पुष्टि करें.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    सत्यापित करें:

    ```
    - signing secret
    - webhook path
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - प्रत्येक HTTP अकाउंट के लिए अद्वितीय `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    सत्यापित करें कि आपका आशय था:

    ```
    - native command mode (`channels.slack.commands.native: true`) और Slack में पंजीकृत मिलते-जुलते slash commands
    - या single slash command mode (`channels.slack.slashCommand.enabled: true`)
    
    साथ ही `commands.useAccessGroups` और channel/user allowlists की जाँच करें.
    ```

  
</Accordion>
</AccordionGroup>

## कॉन्फ़िगरेशन संदर्भ संकेत

मुख्य संदर्भ:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  उच्च-प्रभाव वाले Slack फ़ील्ड:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM एक्सेस: `dm.enabled`, `dmPolicy`, `allowFrom` (पुराना: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - चैनल एक्सेस: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - थ्रेडिंग/इतिहास: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - डिलीवरी: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ऑप्स/फ़ीचर्स: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## संबंधित

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)
