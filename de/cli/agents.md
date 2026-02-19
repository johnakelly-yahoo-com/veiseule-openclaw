---
summary: "CLI-Referenz für `openclaw agents` (auflisten/hinzufügen/löschen/Identität festlegen)"
read_when:
  - Sie möchten mehrere isolierte Agenten (Arbeitsbereiche + Routing + Authentifizierung)
title: "Agenten"
---

# `openclaw agents`

Isolierte Agenten verwalten (Arbeitsbereiche + Authentifizierung + Routing).

Verwandt:

- Multi-Agent-Routing: [Multi-Agent Routing](/concepts/multi-agent)
- Agent-Arbeitsbereich: [Agent workspace](/concepts/agent-workspace)

## Beispiele

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Identitätsdateien

Jeder Agent-Arbeitsbereich kann am Wurzelverzeichnis des Arbeitsbereichs eine `IDENTITY.md` enthalten:

- Beispielpfad: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` liest aus dem Wurzelverzeichnis des Arbeitsbereichs (oder aus einer expliziten `--identity-file`)

Avatar-Pfade werden relativ zum Wurzelverzeichnis des Arbeitsbereichs aufgelöst.

## Identität festlegen

`set-identity` schreibt Felder in `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (arbeitsbereichsrelativer Pfad, http(s)-URL oder Daten-URI)

Laden aus `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Felder explizit überschreiben:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Beispielkonfiguration:

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

