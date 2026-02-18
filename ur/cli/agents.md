---
title: "ایجنٹس"
---

# `openclaw agents`

علیحدہ ایجنٹس (ورک اسپیس + تصدیق + روٹنگ) کا نظم کریں۔

متعلقہ:

- ملٹی ایجنٹ روٹنگ: [Multi-Agent Routing](/concepts/multi-agent)
- ایجنٹ ورک اسپیس: [Agent workspace](/concepts/agent-workspace)

## مثالیں

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## شناختی فائلیں

ہر ایجنٹ ورک اسپیس میں ورک اسپیس روٹ پر ایک `IDENTITY.md` شامل ہو سکتا ہے:

- مثال راستہ: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` ورک اسپیس روٹ سے پڑھتا ہے (یا کسی صریح `--identity-file` سے)

اوتار کے راستے ورک اسپیس روٹ کے نسبت سے حل ہوتے ہیں۔

## شناخت سیٹ کریں

`set-identity`، `agents.list[].identity` میں فیلڈز لکھتا ہے:

- `name`
- `theme`
- `emoji`
- `avatar` (ورک اسپیس کے نسبت راستہ، http(s) URL، یا data URI)

`IDENTITY.md` سے لوڈ کریں:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

فیلڈز کو صریح طور پر اوور رائیڈ کریں:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

کنفیگ نمونہ:

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


