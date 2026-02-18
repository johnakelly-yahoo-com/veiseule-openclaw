---
title: "cli/agents.md"
---

# `openclaw agents`

Zarządzaj odizolowanymi agentami (obszary robocze + uwierzytelnianie + routing).

Powiązane:

- Routing wieloagentowy: [Multi-Agent Routing](/concepts/multi-agent)
- Obszar roboczy agenta: [Agent workspace](/concepts/agent-workspace)

## Przykłady

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Pliki tożsamości

Każdy obszar roboczy agenta może zawierać `IDENTITY.md` w katalogu głównym obszaru roboczego:

- Przykładowa ścieżka: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` odczytuje z katalogu głównego obszaru roboczego (lub z jawnie wskazanego `--identity-file`)

Ścieżki awatara rozwiązują względem głównego obszaru roboczego.

## Ustaw tożsamość

`set-identity` zapisuje pola do `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (ścieżka względna względem obszaru roboczego, adres URL http(s) lub data URI)

Wczytaj z `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Jawnie nadpisz pola:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Przykładowa konfiguracja:

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

