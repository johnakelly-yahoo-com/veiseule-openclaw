---
title: "agenter"
---

# `openclaw agents`

Administrér isolerede agenter (workspaces + auth + routing).

Relateret:

- Routing med flere agenter: [Multi-Agent Routing](/concepts/multi-agent)
- Agent-workspace: [Agent workspace](/concepts/agent-workspace)

## Eksempler

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Identitetsfiler

Hvert agent-workspace kan indeholde en `IDENTITY.md` i workspace-roden:

- Eksempelsti: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` læser fra workspace-roden (eller en eksplicit `--identity-file`)

Avatarstier opløses relativt til workspace-roden.

## Angiv identitet

`set-identity` skriver felter ind i `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (workspace-relativ sti, http(s)-URL eller data-URI)

Indlæs fra `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Tilsidesæt felter eksplicit:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Konfigurationseksempel:

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

