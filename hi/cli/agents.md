---
summary: "CLI के लिए संदर्भ: `openclaw agents` (सूची/जोड़ें/हटाएँ/पहचान सेट करें)"
read_when:
  - आपको कई अलग-थलग एजेंट्स (वर्कस्पेस + रूटिंग + प्रमाणीकरण) चाहिए
title: "एजेंट्स"
---

# `openclaw agents`

अलग-थलग एजेंट्स (वर्कस्पेस + प्रमाणीकरण + रूटिंग) का प्रबंधन करें।

संबंधित:

- मल्टी-एजेंट रूटिंग: [मल्टी-एजेंट रूटिंग](/concepts/multi-agent)
- एजेंट वर्कस्पेस: [एजेंट वर्कस्पेस](/concepts/agent-workspace)

## उदाहरण

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## पहचान फ़ाइलें

प्रत्येक एजेंट वर्कस्पेस में वर्कस्पेस रूट पर एक `IDENTITY.md` शामिल हो सकता है:

- उदाहरण पथ: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` वर्कस्पेस रूट से पढ़ता है (या किसी स्पष्ट `--identity-file` से)

अवतार पथ वर्कस्पेस रूट के सापेक्ष हल होते हैं।

## पहचान सेट करें

`set-identity` `agents.list[].identity` में फ़ील्ड्स लिखता है:

- `name`
- `theme`
- `emoji`
- `avatar` (वर्कस्पेस-सापेक्ष पथ, http(s) URL, या data URI)

`IDENTITY.md` से लोड करें:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

फ़ील्ड्स को स्पष्ट रूप से ओवरराइड करें:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

विन्यास नमूना:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```

