---
title: "agents"
---

# `openclaw agents`

Izolyatsiyalangan agentlarni boshqarish (ishchi muhitlar + autentifikatsiya + marshrutlash).

Bog‘liq:

- Ko‘p agentli marshrutlash: [Multi-Agent Routing](/concepts/multi-agent)
- Agent ishchi muhiti: [Agent workspace](/concepts/agent-workspace)

## Misollar

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Shaxsiyat fayllari

Har bir agent ishchi muhiti ildizida `IDENTITY.md` faylini o‘z ichiga olishi mumkin:

- Namunaviy yo‘l: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` ishchi muhit ildizidan (yoki aniq ko‘rsatilgan `--identity-file`) o‘qiydi.

Avatar yo‘llari ishchi muhit ildiziga nisbatan aniqlanadi.

## Shaxsiyatni sozlash

`set-identity` maydonlarni `agents.list[].identity` ga yozadi:

- `name`
- `theme`
- `emoji`
- `avatar` (ishchi muhitga nisbiy yo‘l, http(s) URL yoki data URI)

`IDENTITY.md` dan yuklash:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Maydonlarni aniq ko‘rsatib almashtirish:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Konfiguratsiya namunasi:

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


