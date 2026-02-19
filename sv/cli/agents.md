---
summary: "CLI-referens för `openclaw agents` (lista/lägg till/ta bort/ange identitet)"
read_when:
  - Du vill ha flera isolerade agenter (arbetsytor + routning + autentisering)
title: "agenter"
---

# `openclaw agents`

Hantera isolerade agenter (arbetsytor + autentisering + routning).

Relaterat:

- Routning med flera agenter: [Multi-Agent Routing](/concepts/multi-agent)
- Agentarbetsyta: [Agent workspace](/concepts/agent-workspace)

## Exempel

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Identitetsfiler

Varje agentarbetsyta kan innehålla en `IDENTITY.md` i arbetsytans rot:

- Exempelsökväg: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` läser från arbetsytans rot (eller en explicit `--identity-file`)

Avatar-sökvägar löses relativt till arbetsytans rot.

## Ange identitet

`set-identity` skriver fält till `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (arbetsyterelativ sökväg, http(s)-URL eller data-URI)

Läs in från `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Åsidosätt fält explicit:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Konfigexempel:

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

