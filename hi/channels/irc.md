---
title: IRC
description: OpenClaw को IRC चैनलों और डायरेक्ट मैसेज से कनेक्ट करें।
---

जब आप OpenClaw को पारंपरिक चैनलों (`#room`) और डायरेक्ट मैसेज में उपयोग करना चाहते हैं, तब IRC का उपयोग करें।
IRC एक extension plugin के रूप में आता है, लेकिन इसे मुख्य config में `channels.irc` के अंतर्गत कॉन्फ़िगर किया जाता है।

## त्वरित प्रारंभ

1. `~/.openclaw/openclaw.json` में IRC config सक्षम करें।
2. कम से कम यह सेट करें:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. gateway शुरू/पुनः शुरू करें:

```bash
openclaw gateway run
```

## सुरक्षा डिफ़ॉल्ट

- `channels.irc.dmPolicy` का डिफ़ॉल्ट मान `"pairing"` है।
- `channels.irc.groupPolicy` का डिफ़ॉल्ट मान `"allowlist"` है।
- यदि `groupPolicy="allowlist"` है, तो अनुमत चैनलों को परिभाषित करने के लिए `channels.irc.groups` सेट करें।
- जब तक आप जानबूझकर plaintext ट्रांसपोर्ट स्वीकार नहीं कर रहे हैं, तब तक TLS (`channels.irc.tls=true`) का उपयोग करें।

## एक्सेस नियंत्रण

IRC चैनलों के लिए दो अलग-अलग “गेट” होते हैं:

1. **Channel access** (`groupPolicy` + `groups`): क्या बॉट किसी चैनल से संदेश स्वीकार करता है या नहीं।
2. **Sender access** (`groupAllowFrom` / प्रति-चैनल `groups["#channel"].allowFrom`): उस चैनल के भीतर बॉट को ट्रिगर करने की अनुमति किसे है।

Config keys:

- DM allowlist (DM प्रेषक एक्सेस): `channels.irc.allowFrom`
- Group sender allowlist (चैनल प्रेषक एक्सेस): `channels.irc.groupAllowFrom`
- Per-channel नियंत्रण (चैनल + प्रेषक + mention नियम): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` बिना कॉन्फ़िगर किए गए चैनलों की अनुमति देता है (**डिफ़ॉल्ट रूप से अभी भी mention-gated रहता है**)

Allowlist प्रविष्टियाँ nick या `nick!user@host` प्रारूप का उपयोग कर सकती हैं।

### सामान्य गलती: `allowFrom` DM के लिए है, चैनलों के लिए नहीं

यदि आपको इस तरह के लॉग दिखाई दें:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…इसका अर्थ है कि **group/channel** संदेशों के लिए प्रेषक को अनुमति नहीं थी। इसे ठीक करने के लिए या तो:

- `channels.irc.groupAllowFrom` सेट करें (सभी चैनलों के लिए वैश्विक रूप से), या
- प्रति-चैनल प्रेषक allowlist सेट करें: `channels.irc.groups["#channel"].allowFrom`

उदाहरण (`#tuirc-dev` में किसी को भी बॉट से बात करने की अनुमति देने के लिए):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## उत्तर ट्रिगर करना (mentions)

भले ही किसी चैनल को अनुमति हो ( `groupPolicy` + `groups` के माध्यम से ) और प्रेषक को भी अनुमति हो, OpenClaw समूह संदर्भों में डिफ़ॉल्ट रूप से **mention-gating** सक्षम रखता है।

इसका अर्थ है कि आपको `drop channel …` जैसे लॉग दिखाई दे सकते हैं `(missing-mention)` जब तक संदेश में ऐसा mention पैटर्न शामिल न हो जो बॉट से मेल खाता हो।

IRC चैनल में बॉट को **बिना mention की आवश्यकता के** उत्तर देने के लिए, उस चैनल के लिए mention gating अक्षम करें:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

या सभी IRC चैनलों को **अनुमति देने के लिए** (प्रति-चैनल allowlist के बिना) और फिर भी mentions के बिना उत्तर देने के लिए:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## सुरक्षा नोट (सार्वजनिक चैनलों के लिए अनुशंसित)

यदि आप किसी सार्वजनिक चैनल में `allowFrom: ["*"]` की अनुमति देते हैं, तो कोई भी बॉट को prompt कर सकता है।
जोखिम कम करने के लिए, उस चैनल के लिए tools को प्रतिबंधित करें।

### चैनल में सभी के लिए समान tools

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### प्रत्येक प्रेषक के लिए अलग-अलग tools (owner को अधिक अधिकार मिलते हैं)

`toolsBySender` का उपयोग करें ताकि `"*"` पर सख्त नीति लागू हो और आपके nick पर थोड़ी ढीली नीति लागू हो:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

नोट्स:

- `toolsBySender` की keys एक nick (जैसे `"eigen"`) या अधिक सशक्त पहचान मिलान के लिए पूर्ण hostmask (`"eigen!~eigen@174.127.248.171"`) हो सकती हैं।
- पहली मेल खाने वाली sender नीति लागू होती है; `"*"` wildcard fallback है।

group access बनाम mention-gating (और वे कैसे परस्पर क्रिया करते हैं) के बारे में अधिक जानकारी के लिए देखें: [/channels/groups](/channels/groups)।

## NickServ

कनेक्ट होने के बाद NickServ के साथ पहचान स्थापित करने के लिए:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

कनेक्ट होने पर वैकल्पिक एक-बार पंजीकरण:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

nick पंजीकृत हो जाने के बाद `register` को अक्षम कर दें ताकि बार-बार REGISTER प्रयासों से बचा जा सके।

## Environment variables

डिफ़ॉल्ट खाता निम्न का समर्थन करता है:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (comma-separated)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## समस्या निवारण

- यदि बॉट कनेक्ट हो जाता है लेकिन चैनलों में कभी उत्तर नहीं देता, तो `channels.irc.groups` **और** यह जाँचें कि कहीं mention-gating संदेशों को (`missing-mention`) के कारण रोक तो नहीं रहा है। यदि आप चाहते हैं कि यह pings के बिना उत्तर दे, तो उस चैनल के लिए `requireMention:false` सेट करें।
- यदि लॉगिन विफल हो, तो nick की उपलब्धता और server password की जाँच करें।
- यदि किसी कस्टम नेटवर्क पर TLS विफल हो, तो host/port और certificate सेटअप की जाँच करें।

