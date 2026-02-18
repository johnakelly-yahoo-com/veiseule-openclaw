---
title: "Discord"
---

# Discord (Bot API)

स्थिति: आधिकारिक Discord gateway के माध्यम से DMs और guild चैनलों के लिए तैयार।

<CardGroup cols={3}>
  <Card title="पेयरिंग" icon="link" href="/channels/pairing">
    Discord DMs डिफ़ॉल्ट रूप से pairing मोड में होते हैं।
  </Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native command व्यवहार और command catalog।
  </Card>
  <Card title="चैनल समस्या‑निवारण" icon="wrench" href="/channels/troubleshooting">
    क्रॉस‑चैनल डायग्नोस्टिक्स और रिपेयर फ्लो।
  </Card>
</CardGroup>

## त्वरित सेटअप

<Steps>
  <Step title="Discord बॉट बनाएँ और intents सक्षम करें">
    Discord Developer Portal में एक application बनाएँ, एक bot जोड़ें, फिर सक्षम करें:

    - **Message Content Intent**
    - **Server Members Intent** (role allowlists और role‑based routing के लिए आवश्यक; name‑to‑ID allowlist matching के लिए अनुशंसित)

  </Step>

  <Step title="टोकन कॉन्फ़िगर करें">

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

    Default account के लिए env fallback:

```bash
DISCORD_BOT_TOKEN=...
```

  </Step>

  <Step title="बॉट को invite करें और gateway शुरू करें">
    संदेश अनुमतियों के साथ बॉट को अपने सर्वर में invite करें।

```bash
openclaw gateway
```

  </Step>

  <Step title="पहला DM pairing अनुमोदित करें">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    Pairing codes 1 घंटे बाद समाप्त हो जाते हैं।

  </Step>
</Steps>

<Note>
Token resolution account‑aware है। Config token values env fallback पर प्राथमिकता लेते हैं। `DISCORD_BOT_TOKEN` केवल default account के लिए उपयोग होता है।
</Note>

## Runtime model

- Gateway Discord connection का स्वामी होता है।
- Reply routing निर्धारक है: Discord inbound का उत्तर वापस Discord पर ही जाता है।
- डिफ़ॉल्ट रूप से (`session.dmScope=main`), direct chats agent main session (`agent:main:main`) साझा करते हैं।
- Guild चैनल अलग session keys उपयोग करते हैं (`agent:<agentId>:discord:channel:<channelId>`).
- Group DMs डिफ़ॉल्ट रूप से अनदेखी की जाती हैं (`channels.discord.dm.groupEnabled=false`)।
- Native slash commands अलग command sessions में चलते हैं (`agent:<agentId>:discord:slash:<userId>`), लेकिन routed conversation session के लिए `CommandTargetSessionKey` साथ ले जाते हैं।

## Access control और routing

<Tabs>
  <Tab title="DM policy">
    `channels.discord.dmPolicy` DM access नियंत्रित करता है (legacy: `channels.discord.dm.policy`):

    - `pairing` (default)
    - `allowlist`
    - `open` (आवश्यक है कि `channels.discord.allowFrom` में `"*"` शामिल हो; legacy: `channels.discord.dm.allowFrom`)
    - `disabled`

    यदि DM policy open नहीं है, तो unknown users ब्लॉक कर दिए जाते हैं (या `pairing` मोड में pairing के लिए प्रॉम्प्ट किए जाते हैं)।

    Delivery के लिए DM target format:

    - `user:<id>`
    - `<@id>` mention

    केवल संख्यात्मक IDs अस्पष्ट माने जाते हैं और अस्वीकृत होते हैं जब तक स्पष्ट user/channel target kind प्रदान न किया जाए।

  </Tab>

  <Tab title="Guild policy">
    Guild handling `channels.discord.groupPolicy` द्वारा नियंत्रित होता है:

    - `open`
    - `allowlist`
    - `disabled`

    जब `channels.discord` मौजूद हो, तो सुरक्षित baseline `allowlist` है।

    `allowlist` व्यवहार:

    - guild को `channels.discord.guilds` से मेल खाना चाहिए (`id` प्राथमिकता, slug स्वीकार्य)
    - वैकल्पिक sender allowlists: `users` (IDs या names) और `roles` (केवल role IDs); यदि इनमें से कोई भी कॉन्फ़िगर है, तो sender तब अनुमति पाता है जब वह `users` OR `roles` से मेल खाए
    - यदि guild में `channels` कॉन्फ़िगर है, तो सूचीबद्ध न किए गए चैनल अस्वीकृत होते हैं
    - यदि guild में `channels` ब्लॉक नहीं है, तो उस allowlisted guild के सभी चैनल अनुमत होते हैं

    उदाहरण:

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

    यदि आप केवल `DISCORD_BOT_TOKEN` सेट करते हैं और `channels.discord` ब्लॉक नहीं बनाते, तो runtime fallback `groupPolicy="open"` होता है (logs में warning के साथ)।

  </Tab>

  <Tab title="Mentions और group DMs">
    Guild संदेश डिफ़ॉल्ट रूप से mention‑gated होते हैं।

    Mention detection में शामिल है:

    - explicit bot mention
    - configured mention patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - समर्थित मामलों में implicit reply‑to‑bot व्यवहार

    `requireMention` प्रति guild/channel (`channels.discord.guilds...`) कॉन्फ़िगर किया जाता है।

    Group DMs:

    - डिफ़ॉल्ट: अनदेखी (`dm.groupEnabled=false`)
    - वैकल्पिक allowlist via `dm.groupChannels` (channel IDs या slugs)

  </Tab>
</Tabs>

### Role-based agent routing

Discord guild members को role ID के आधार पर अलग‑अलग agents पर route करने के लिए `bindings[].match.roles` उपयोग करें। Role‑based bindings केवल role IDs स्वीकार करते हैं और peer या parent‑peer bindings के बाद तथा guild‑only bindings से पहले evaluate होते हैं। यदि किसी binding में अन्य match fields भी सेट हैं (उदाहरण के लिए `peer` + `guildId` + `roles`), तो सभी कॉन्फ़िगर किए गए fields को match होना चाहिए।

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

