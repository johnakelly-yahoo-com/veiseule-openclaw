---
summary: "Справочник CLI для `openclaw agents` (list/add/delete/set identity)"
read_when:
  - Вам нужны несколько изолированных агентов (рабочие пространства + маршрутизация + аутентификация)
title: "agents"
---

# `openclaw agents`

Управление изолированными агентами (рабочие пространства + аутентификация + маршрутизация).

Связанное:

- Маршрутизация нескольких агентов: [Multi-Agent Routing](/concepts/multi-agent)
- Рабочее пространство агента: [Agent workspace](/concepts/agent-workspace)

## Примеры

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Файлы идентичности

Каждое рабочее пространство агента может включать `IDENTITY.md` в корне рабочего пространства:

- Пример пути: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` читает из корня рабочего пространства (или из явного `--identity-file`)

Пути к аватарам разрешаются относительно корня рабочего пространства.

## Установка идентичности

`set-identity` записывает поля в `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (путь относительно рабочего пространства, URL http(s) или data URI)

Загрузка из `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Явное переопределение полей:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Пример конфига:

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

