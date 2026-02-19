---
summary: "CLI-referentie voor `openclaw agents` (lijst, toevoegen, verwijderen, identiteit instellen)"
read_when:
  - Je wilt meerdere geïsoleerde agents (werkruimtes + routing + authenticatie)
title: "agents"
---

# `openclaw agents`

Beheer geïsoleerde agents (werkruimtes + authenticatie + routing).

Gerelateerd:

- Multi-agentroutering: [Multi-agentroutering](/concepts/multi-agent)
- Agentwerkruimte: [Agentwerkruimte](/concepts/agent-workspace)

## Voorbeelden

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Identiteitsbestanden

Elke agentwerkruimte kan een `IDENTITY.md` bevatten in de root van de werkruimte:

- Voorbeeldpad: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` leest vanuit de root van de werkruimte (of een expliciete `--identity-file`)

Avatarpaden worden relatief ten opzichte van de root van de werkruimte opgelost.

## Identiteit instellen

`set-identity` schrijft velden naar `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (pad relatief aan de werkruimte, http(s)-URL of data-URI)

Laden vanuit `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Overschrijf velden expliciet:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Config-voorbeeld:

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

