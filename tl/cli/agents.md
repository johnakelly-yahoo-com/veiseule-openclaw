---
title: "agents"
---

# `openclaw agents`

Pamahalaan ang mga hiwalay na agent (mga workspace + auth + routing).

Kaugnay:

- Pag-route ng multi-agent: [Pag-route ng Multi-Agent](/concepts/multi-agent)
- Workspace ng agent: [Workspace ng Agent](/concepts/agent-workspace)

## Mga halimbawa

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Mga identity file

Bawat agent workspace ay maaaring maglaman ng isang `IDENTITY.md` sa root ng workspace:

- Halimbawang path: `~/.openclaw/workspace/IDENTITY.md`
- Nagbabasa ang `set-identity --from-identity` mula sa root ng workspace (o isang tahasang `--identity-file`)

Ang mga path ng avatar ay nireresolba kaugnay ng root ng workspace.

## Itakda ang identity

Isinusulat ng `set-identity` ang mga field papunta sa `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (path na relative sa workspace, http(s) URL, o data URI)

Mag-load mula sa `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

I-override ang mga field nang tahasan:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Halimbawang config:

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

