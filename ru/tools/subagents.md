---
summary: "Субагенты: запуск изолированных агентных прогонов, которые объявляют результаты обратно в чат инициатора"
read_when:
  - Вам нужна фоновая/параллельная работа через агента
  - Вы изменяете sessions_spawn или политику инструментов субагентов
title: "Субагенты"
---

# Субагенты

Sub-agents — это фоновые запуски агента, инициированные из существующего запуска агента. Они выполняются в собственной сессии (`agent:<agentId>:subagent:<uuid>`) и по завершении **объявляют** свой результат обратно в чат-канал запрашивающей стороны.

## Slash command

Используйте слеш-команду `/subagents` для просмотра и управления запусками субагентов в текущей сессии:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` показывает метаданные запуска (статус, временные метки, id сессии, путь к транскрипту, очистка).

Основные цели:

- Распараллеливание задач типа «исследование / длительная задача / медленный инструмент» без блокировки основного запуска.
- Сохранять изоляцию sub-agents по умолчанию (разделение сессий + опциональная песочница).
- Сделать поверхность инструментов устойчивой к неправильному использованию: sub-agents по умолчанию **не** получают инструменты сессии.
- Поддержка настраиваемой глубины вложенности для оркестрационных сценариев.

Примечание о стоимости: каждый sub-agent имеет **собственный** контекст и потребление токенов. Для ресурсоёмких или повторяющихся
задач установите более дешёвую модель для sub-agents, а для основного агента оставьте модель более высокого качества.
In a multi-agent setup, you can set sub-agent defaults per agent:

## #> [limit] [tools]\`

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- Затем выполняется шаг announce, и ответ announce публикуется в чат-канал запрашивающей стороны
- Модель по умолчанию: наследуется от вызывающего агента, если не задано `agents.defaults.subagents.model` (или `agents.list[].subagents.model` для конкретного агента); явное значение `sessions_spawn.model` всё равно имеет приоритет.
- Per-agent config: `agents.list[].subagents.thinking`

Параметры инструмента:

- `task`
- _(optional)_
- Spawn under a different agent id (must be allowed)
- Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
- `thinking?` (необязательно; переопределяет уровень thinking для запуска sub-agent)
- Abort the sub-agent after N seconds
- `"delete"` \\

Allowlist:

- Per-agent config: `agents.list[].subagents.model` По умолчанию: только запрашивающий агент.

Обнаружение:

- Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted.` (та же папка).
- `"delete"` archives immediately after announce
- Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- Автоархивация одинаково применяется к сессиям глубины 1 и глубины 2.

## Stopping Sub-Agents

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

### Как включить

```json5
{
  agents: {
  defaults: {
  subagents: {
  model: "minimax/MiniMax-M2.1",
  },
  },
  },
  }
```

### Уровни глубины

| Глубина | Формат ключа сессии                                                                                                                                                                                                                                                                                                                                                                                                          | Роль                                                                 | Может создавать?                 |
| ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------- | -------------------------------- |
| 0       | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | Per-Agent Overrides                                                  | Всегда                           |
| 1       | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | Суб-агент (оркестратор, если разрешена глубина 2) | Только если `maxSpawnDepth >= 2` |
| 2       | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Managing Sub-Agents (`/subagents`)                | Никогда                          |

### Цепочка announce

Результаты передаются обратно вверх по цепочке:

1. Исполнитель глубины 2 завершает работу → отправляет announce своему родителю (оркестратору глубины 1)
2. Оркестратор глубины 1 получает announce, обобщает результаты, завершает работу → отправляет announce в main
3. Основной агент получает announce и передаёт результат пользователю

Каждый уровень видит только announce от своих непосредственных потомков.

### Tool Policy

- **Глубина 1 (оркестратор, когда `maxSpawnDepth >= 2`)**: Получает `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`, чтобы управлять своими дочерними агентами. Остальные инструменты сессии/системы остаются недоступными.
- **Глубина 1 (лист, когда `maxSpawnDepth == 1`)**: Нет инструментов сессии (поведение по умолчанию).
- **Глубина 2 (исполнитель)**: Нет инструментов сессии — `sessions_spawn` всегда запрещён на глубине 2. Не может создавать дальнейших дочерних агентов.

### Cross-Agent Spawning

Каждая сессия агента (на любой глубине) может иметь не более `maxChildrenPerAgent` (по умолчанию: 5) активных дочерних агентов одновременно. Это предотвращает неконтролируемое разрастание от одного оркестратора.

### Каскадная остановка

Остановка оркестратора глубины 1 автоматически останавливает всех его дочерних агентов глубины 2:

- `/stop` в основном чате останавливает всех агентов глубины 1 и каскадно их дочерних агентов глубины 2.
- Aborts the main session **and** all active sub-agent runs spawned from it
- `/subagents kill all` останавливает всех суб-агентов для инициатора запроса и выполняет каскадную остановку.

## Аутентификация

Аутентификация субагента определяется **id агента**, а не типом сессии:

- Ключ сессии суб-агента: `agent:<agentId>:subagent:<uuid>`.
- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)

The merge is additive — main profiles are always available as fallbacks
Fully isolated auth per sub-agent is not currently supported.

## Announce Status

Суб-агенты отчитываются обратно через шаг announce:

- Шаг announce выполняется внутри сессии суб-агента (а не в сессии инициатора запроса).
- If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted. This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
- В противном случае ответ announce публикуется в чат инициатора запроса через последующий вызов `agent` (`deliver=true`).
- Ответы объявления сохраняют маршрутизацию тредов/тем, где это доступно (треды Slack, темы Telegram, треды Matrix).
- Сообщения announce приводятся к стабильному шаблону:
  - `Status:` определяется по результату выполнения (`success`, `error`, `timeout` или `unknown`).
  - `Result:` краткое содержимое из шага announce (или `(not available)`, если отсутствует).
  - `Примечания:` детали ошибки и другой полезный контекст.
- The announce message includes a status derived from the runtime outcome (not from model output):

Each announce includes a stats line with:

- Время выполнения (например, `runtime 5m12s`)
- Расход токенов (вход/выход/всего)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` и путь к транскрипту (чтобы основной агент мог получить историю через `sessions_history` или просмотреть файл на диске)

## Customizing Sub-Agent Tools

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

Когда `maxSpawnDepth >= 2`, суб-агенты оркестратора уровня глубины 1 дополнительно получают `sessions_spawn`, `subagents`, `sessions_list` и `sessions_history`, чтобы управлять своими дочерними агентами.

Переопределение через конфигурацию:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

## Конвертация

Суб-агенты используют выделенную очередь внутри процесса:

- Имя очереди: `subagent`
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## Остановка

- Отправка `/stop` в чате инициатора прерывает сессию инициатора и останавливает все активные запуски суб-агентов, созданные из неё, с каскадной остановкой вложенных дочерних агентов.
- `/subagents stop <id>`

## Ограничения

- Объявление суб-агента выполняется в режиме **best-effort**. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- Вызов **неблокирующий** — основной агент немедленно получает `{ status: "accepted", runId, childSessionKey }`.
- Контекст суб-агента включает только `AGENTS.md` + `TOOLS.md` (без `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` или `BOOTSTRAP.md`).
- Максимальная глубина вложенности — 5 (диапазон `maxSpawnDepth`: 1–5). Для большинства случаев рекомендуется глубина 2.
- `maxChildrenPerAgent` ограничивает количество активных дочерних агентов на сессию (по умолчанию: 5, диапазон: 1–20).
