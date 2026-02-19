---
summary: "7. imsg के माध्यम से लेगेसी iMessage सपोर्ट (stdio पर JSON-RPC)। 8. नए सेटअप्स को BlueBubbles का उपयोग करना चाहिए।"
read_when:
  - iMessage समर्थन सेट करते समय
  - iMessage भेजने/प्राप्त करने में डिबगिंग
title: "iMessage"
---

# iMessage (लीगेसी: imsg)

<Warning>
नई iMessage डिप्लॉयमेंट के लिए <a href="/channels/bluebubbles">BlueBubbles</a> का उपयोग करें।

`imsg` इंटीग्रेशन लेगेसी है और भविष्य के रिलीज़ में हटाया जा सकता है। 
</Warning>

9. स्थिति: लेगेसी बाहरी CLI इंटीग्रेशन। Gateway `imsg rpc` को स्पॉन करता है और stdio पर JSON-RPC के माध्यम से संचार करता है (कोई अलग daemon/port नहीं)।

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    नए सेटअप के लिए पसंदीदा iMessage पाथ।
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DMs डिफ़ॉल्ट रूप से पेयरिंग मोड में होते हैं।
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    iMessage फ़ील्ड का पूर्ण संदर्भ।
  
</Card>
</CardGroup>

## त्वरित सेटअप

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="OpenClaw को कॉन्फ़िगर करें">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Gateway शुरू करें">

```bash
openclaw gateway
```

      {
        channels: { imessage: { configWrites: false } },
      }

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        ```
            पेयरिंग अनुरोध 1 घंटे बाद समाप्त हो जाते हैं।
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw को केवल एक stdio-संगत `cliPath` की आवश्यकता होती है, इसलिए आप `cliPath` को ऐसे wrapper script की ओर इंगित कर सकते हैं जो किसी रिमोट Mac पर SSH करके `imsg` चलाती हो।

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    अटैचमेंट सक्षम होने पर अनुशंसित कॉन्फ़िगरेशन:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // SCP अटैचमेंट फ़ेच के लिए उपयोग किया जाता है
      includeAttachments: true,
    },
  },
}
```

    ```
    यदि `remoteHost` सेट नहीं है, तो OpenClaw SSH wrapper script को पार्स करके इसे स्वतः पहचानने का प्रयास करता है।
    ```

  
</Tab>
</Tabs>

## आवश्यकताएँ और अनुमतियाँ (macOS)

- `imsg` चलाने वाले Mac पर Messages में साइन इन होना आवश्यक है।
- OpenClaw/`imsg` चलाने वाले प्रोसेस कॉन्टेक्स्ट के लिए Full Disk Access आवश्यक है (Messages DB एक्सेस)।
- Messages.app के माध्यम से संदेश भेजने के लिए Automation अनुमति आवश्यक है।

<Tip>
अनुमतियाँ प्रत्येक प्रोसेस कॉन्टेक्स्ट के अनुसार प्रदान की जाती हैं। यदि gateway headless (LaunchAgent/SSH) रूप में चल रहा है, तो प्रॉम्प्ट ट्रिगर करने के लिए उसी कॉन्टेक्स्ट में एक बार इंटरैक्टिव कमांड चलाएँ:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## एक्सेस नियंत्रण और रूटिंग

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` सीधे संदेशों को नियंत्रित करता है:

    ```
    - `pairing` (डिफ़ॉल्ट)
    - `allowlist`
    - `open` (इसके लिए `allowFrom` में `"*"` शामिल होना आवश्यक है)
    - `disabled`
    
    Allowlist फ़ील्ड: `channels.imessage.allowFrom`.
    
    Allowlist प्रविष्टियाँ हैंडल या चैट टार्गेट (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`) हो सकती हैं।
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` समूह प्रबंधन को नियंत्रित करता है:

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DMs सीधे रूटिंग का उपयोग करते हैं; समूह समूह रूटिंग का उपयोग करते हैं।
    - डिफ़ॉल्ट `session.dmScope=main` के साथ, iMessage DMs एजेंट की मुख्य सेशन में समाहित हो जाते हैं।
    - समूह सेशन अलग-थलग रहते हैं (`agent:<agentId>:imessage:group:<chat_id>`).
    - उत्तर मूल चैनल/टार्गेट मेटाडेटा का उपयोग करके वापस iMessage पर रूट किए जाते हैं।

    ```
    समूह-समान थ्रेड व्यवहार:
    
    कुछ बहु-प्रतिभागी iMessage थ्रेड `is_group=false` के साथ आ सकते हैं।
    यदि वह `chat_id` स्पष्ट रूप से `channels.imessage.groups` के अंतर्गत कॉन्फ़िगर किया गया है, तो OpenClaw उसे समूह ट्रैफ़िक के रूप में मानता है (group gating + समूह सेशन पृथक्करण)।
    ```

  
</Tab>
</Tabs>

## डिप्लॉयमेंट पैटर्न

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    एक समर्पित Apple ID और macOS उपयोगकर्ता का उपयोग करें ताकि बॉट ट्रैफ़िक आपके व्यक्तिगत Messages प्रोफ़ाइल से अलग रहे।

    ```
    {
      channels: {
        imessage: {
          cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
          remoteHost: "user@gateway-host", // for SCP file transfer
          includeAttachments: true,
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    सामान्य टोपोलॉजी:

    ```
    - gateway Linux/VM पर चलता है
    - iMessage + `imsg` आपके tailnet में एक Mac पर चलता है
    - `cliPath` रैपर `imsg` चलाने के लिए SSH का उपयोग करता है
    - `remoteHost` SCP अटैचमेंट फ़ेच को सक्षम करता है
    
    उदाहरण:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    SSH keys का उपयोग करें ताकि SSH और SCP दोनों non-interactive हों।
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage, `channels.imessage.accounts` के अंतर्गत प्रति-खाता कॉन्फ़िगरेशन का समर्थन करता है।

    ```
    प्रत्येक खाता `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, और history सेटिंग्स जैसे फ़ील्ड को ओवरराइड कर सकता है।
    ```

  
</Accordion>
</AccordionGroup>

## मीडिया, चंकिंग, और डिलीवरी टार्गेट

<AccordionGroup>
  <Accordion title="Attachments and media">
    - इनबाउंड अटैचमेंट इनजेशन वैकल्पिक है: `channels.imessage.includeAttachments`
    - जब `remoteHost` सेट हो, तो रिमोट अटैचमेंट पाथ को SCP के माध्यम से फ़ेच किया जा सकता है
    - आउटबाउंड मीडिया आकार `channels.imessage.mediaMaxMb` (डिफ़ॉल्ट 16 MB) का उपयोग करता है
  
</Accordion>

  <Accordion title="Outbound chunking">
    - टेक्स्ट चंक सीमा: `channels.imessage.textChunkLimit` (डिफ़ॉल्ट 4000)
    - चंक मोड: `channels.imessage.chunkMode`
      - `length` (डिफ़ॉल्ट)
      - `newline` (पहले पैराग्राफ के आधार पर विभाजन)
  
</Accordion>

  <Accordion title="Addressing formats">
    पसंदीदा स्पष्ट टार्गेट:

    ```
    - `chat_id:123` (स्थिर रूटिंग के लिए अनुशंसित)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    हैंडल टार्गेट भी समर्थित हैं:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Config writes

iMessage डिफ़ॉल्ट रूप से चैनल-आरंभित कॉन्फ़िग राइट्स की अनुमति देता है (जब `commands.config: true` हो तो `/config set|unset` के लिए)।

अक्षम करें:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## समस्या निवारण

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    बाइनरी और RPC समर्थन को सत्यापित करें:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    {
      channels: {
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15555550123"],
          groups: {
            "42": { requireMention: false },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    जाँच करें:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - pairing अनुमोदन (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    जाँच करें:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - `channels.imessage.groups` allowlist व्यवहार
    - mention pattern कॉन्फ़िगरेशन (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    जांचें:

    ```
    - `channels.imessage.remoteHost`
    - gateway host से SSH/SCP key auth
    - Messages चलाने वाले Mac पर remote path की readability
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    उसी user/session context में एक interactive GUI terminal में दोबारा चलाएँ और prompts को approve करें:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    सुनिश्चित करें कि Full Disk Access + Automation उस process context के लिए प्रदान किए गए हैं जो OpenClaw/`imsg` चलाता है।
    ```

  
</Accordion>
</AccordionGroup>

## Configuration संदर्भ संकेत

- `agents.list[].groupChat.mentionPatterns` (या `messages.groupChat.mentionPatterns`)।
- `messages.responsePrefix`।
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
