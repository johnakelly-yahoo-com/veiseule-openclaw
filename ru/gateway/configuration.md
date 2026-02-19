---
summary: "Обзор конфигурации: типовые задачи, быстрая настройка и ссылки на полную справку"
read_when:
  - Первая настройка OpenClaw
  - Поиск распространённых шаблонов конфигурации
  - Переход к конкретным разделам конфигурации
title: "Конфигурация"
---

# Конфигурация 🔧

OpenClaw читает необязательный конфиг **JSON5** из `~/.openclaw/openclaw.json` (разрешены комментарии и висячие запятые).

Если файл отсутствует, OpenClaw использует «достаточно безопасные» значения по умолчанию (встроенный агент Pi + сессии на отправителя + рабочее пространство `~/.openclaw/workspace`). Обычно конфиг нужен, чтобы:

- Подключение каналов и управление тем, кто может отправлять сообщения боту
- Настройка моделей, инструментов, песочницы или автоматизации (cron, hooks)
- Настройка сессий, медиа, сети или UI

См. [полную справку](/gateway/configuration-reference) для всех доступных полей.

<Tip>
`OPENCLAW_CONFIG_PATH` (конфигурация для каждого экземпляра)
</Tip>

## Минимальный конфиг (рекомендуемая отправная точка)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Редактирование конфигурации

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # full setup wizard
openclaw configure     # config wizard
```
</Tab>
  <Tab title="CLI (one-liners)">    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
</Tab>
  <Tab title="Control UI">
    Gateway предоставляет представление JSON Schema для конфига через `config.schema` для UI‑редакторов.
    Control UI рендерит форму из этой схемы и предлагает редактор **Raw JSON** как запасной вариант.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (или `OPENCLAW_CONFIG_PATH`) Gateway отслеживает файл и применяет изменения автоматически (см. [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Строгая валидация

<Warning>
OpenClaw принимает только конфигурации, полностью соответствующие схеме. Неизвестные ключи, неверные типы или недопустимые значения приводят к тому, что Gateway **отказывается запускаться** из соображений безопасности. Единственное исключение на корневом уровне — `$schema` (string), чтобы редакторы могли подключать метаданные JSON Schema.
</Warning>

При неудачной проверке:

- Gateway не загружается.
- Разрешены только диагностические команды (например: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Запустите `openclaw doctor`, чтобы увидеть точные проблемы.
- Запустите `openclaw doctor --fix` (или `--yes`) для применения миграций/исправлений.

## Распространённые задачи

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">    Each channel has its own config section under `channels.<provider>`. См. отдельную страницу канала для шагов по настройке:

    ```
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">Укажите основную модель и при необходимости резервные:

    ```
    {
      agents: {
        defaults: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              команда: "my-cli",
              аргов: ["--json"],
              Выход: "json",
              Образец модели: "--model",
              Сессия: "--session",
              сессионный режим: "существует",
              системная подсказка: "--system",
              systemPromptкогда: "first",
              imageArg: "--image",
              imageMode: "repeat",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">Доступ к DM настраивается для каждого канала через `dmPolicy`:

    ```
    Группа: `channels.mattermost.groupPolicy="allowlist"` по умолчанию (mention-gated). Используйте `channels.mattermost.groupAllowFrom` для ограничения отправителей.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Сообщения в группах по умолчанию **требуют упоминания** (метаданные упоминаний или regex‑шаблоны). `agents.defaults.subagents` настраивает настройки по умолчанию:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">Изоляция сессии в нитях:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // рекомендуется для многопользовательской среды
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (общая) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - См. [Session Management](/concepts/session) для информации об областях действия, связях идентичностей и политике отправки.
    - См. [full reference](/gateway/configuration-reference#session) для полного списка полей.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    Запускать сессии агента в изолированных Docker-контейнерах:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    ```
    - `every`: строка длительности (`30m`, `2h`). Установите `0m`, чтобы отключить.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - См. [Heartbeat](/gateway/heartbeat) для полного руководства.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Смотрите [Cron jobs](/automation/cron-jobs) обзор возможностей и примеры CLI.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">    Включить HTTP webhook-эндпоинты в Gateway:

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        путь: "/hooks",
        пресет: ["gmail"],
        transformsDir: "~/. пенклоу/печеньки",
        сопоставления: [
          {
            match: { path: "gmail" },
            Действие: "Агент",
            Режим пробуждения: "now",
            название: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            шаблона сообщений: "От: {{messages[0].from}}\nТема: {{messages[0].subject}}\n{{messages[0].snippet}}",
            доставка: true,
            канал: "последний",
            модель: "openai/gpt-5. -mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Запускать несколько изолированных агентов (отдельное рабочее пространство, `agentDir`, сессии) внутри одного шлюза.

    ```
    {
      agents: {
        list: [
          { id: "home", по умолчанию: true, workspace: "~/. penclaw/workspace-home" },
          { id: "work", workspace: "~/. penclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", ID аккаунта: "biz" } },
      ],
      каналов: {
        whatsapp: {
          accounts: {
            personal: {},
            байт: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Разделяйте конфиг на несколько файлов с помощью директивы `$include`. Это полезно для:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (настройка горячей перезагрузки)

Gateway отслеживает `~/.openclaw/openclaw.json` и применяет изменения автоматически — для большинства настроек ручной перезапуск не требуется.

### Режимы перезагрузки

| Режимы:                        | Поведение                                                                                                                                                                        |
| ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (по умолчанию) | Мгновенно применяет безопасные изменения. Автоматически перезапускается для критических изменений.                                               |
| **Правила:**                   | Применяет только безопасные изменения без перезапуска. Выводит предупреждение в журнале, когда требуется перезапуск — вы выполняете его вручную. |
| **`restart`**                                  | `restart`: перезапустите шлюз при любом изменении конфигурации.                                                                                  |
| \\`{provider}                                 | Отключает отслеживание файла. Изменения вступают в силу при следующем ручном перезапуске.                                                        |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Что применяется «на лету», а что требует перезапуска

Большинство полей применяются «на лету» без простоя. В режиме `hybrid` изменения, требующие перезапуска, обрабатываются автоматически.

| Категория                            | Поля                                                                                    | Требуется перезапуск?                       |
| ------------------------------------ | --------------------------------------------------------------------------------------- | ------------------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — все встроенные и расширяемые каналы | Нет                                         |
| Агент и модели                       | `agent`, `agents`, `models`, `routing`                                                  | Нет                                         |
| Автоматизация                        | `hooks`, `cron`, `agent.heartbeat`                                                      | Нет                                         |
| messages                             | `messages.inbound`                                                                      | Нет                                         |
| Инструменты и медиа                  | `tools`, `browser`, `skills`, `audio`, `talk`                                           | Нет                                         |
| UI и прочее                          | `ui`, `logging`, `identity`, `bindings`                                                 | Нет                                         |
| Сервер Gateway                       | `gateway` (port/bind/auth/control UI/tailscale)                      | **Встроенная подстановка:** |
| Инфраструктура                       | `discovery`, `canvasHost`, `plugins`                                                    | **Типы упоминаний:**        |

<Note>
`gateway.reload` и `gateway.remote` — исключения: их изменение **не** приводит к перезапуску.
</Note>

## Частичные обновления (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Используйте `config.apply`, чтобы проверить и записать полный конфиг и перезапустить Gateway за один шаг.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Используйте `config.patch`, чтобы объединить частичное обновление с существующим конфигом, не затирая
несвязанные ключи. Применяется семантика JSON merge patch:

    ````
    - Объекты объединяются рекурсивно
    - `null` удаляет ключ
    - Массивы заменяются
    
    Параметры:
    
    - `raw` (string) — JSON5 только с изменяемыми ключами
    - `baseHash` (обязательно) — хэш конфигурации из `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — те же, что и в `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Эквиваленты в переменных окружения:

OpenClaw считывает переменные окружения из родительского процесса, а также:

- `.env` из текущего рабочего каталога (если присутствует)
- глобальный fallback `.env` из `~/.openclaw/.env` (также известен как `$OPENCLAW_STATE_DIR/.env`)

Ни один файл `.env` не переопределяет уже существующие переменные окружения. Также можно задать inline‑переменные окружения в конфиге.

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">Опциональное удобство: если включено и ни один из ожидаемых ключей ещё не задан, OpenClaw
запускает ваш login‑shell и импортирует только отсутствующие ожидаемые ключи (никогда не переопределяет).

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Вы можете ссылаться на переменные окружения прямо в любом строковом значении конфига,
используя синтаксис `${VAR_NAME}`.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

Порядок разрешения:

- Сопоставляются только имена env‑переменных в верхнем регистре: `[A-Z_][A-Z0-9_]*`
- Отсутствующие/пустые переменные вызывают ошибку при загрузке
- Экранируйте с помощью `$${VAR}`, чтобы вывести литерал `${VAR}`
- Работает с `$include` (включённые файлы тоже проходят подстановку)
- Встроенная подстановка: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

См. [/environment](/help/environment) для полного порядка приоритета и источников.

## Полная справка

**Новичкам в конфигурации?** Посмотрите руководство [Configuration Examples](/gateway/configuration-examples) с полными примерами и подробными объяснениями!

---

gateway/configuration.md

