---
title: "Конфигурация"
---

# Конфигурация

OpenClaw читает необязательный <Tooltip tip="JSON5 поддерживает комментарии и висячие запятые">**JSON5**</Tooltip>-конфиг из `~/.openclaw/openclaw.json`.

Если файл отсутствует, OpenClaw использует безопасные значения по умолчанию. Частые причины добавить конфиг:

- Подключить каналы и управлять тем, кто может писать боту
- Настроить модели, инструменты, песочницу или автоматизацию (cron, hooks)
- Тонко настроить сессии, медиа, сеть или UI

См. [полный справочник](/gateway/configuration-reference) для всех доступных полей.

<Tip>
**Новичкам в конфигурации?** Начните с `openclaw onboard` для интерактивной настройки или посмотрите руководство [Configuration Examples](/gateway/configuration-examples) с готовыми конфигами для копирования.
</Tip>

## Минимальный конфиг

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Редактирование конфига

<Tabs>
  <Tab title="Интерактивный мастер">
    ```bash
    openclaw onboard       # полный мастер настройки
    openclaw configure     # мастер конфигурации
    ```
  </Tab>
  <Tab title="CLI (однострочники)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    Откройте [http://127.0.0.1:18789](http://127.0.0.1:18789) и используйте вкладку **Config**.  
    Control UI рендерит форму на основе схемы конфига и предоставляет редактор **Raw JSON** как запасной вариант.
  </Tab>
  <Tab title="Прямое редактирование">
    Отредактируйте `~/.openclaw/openclaw.json` напрямую. Gateway отслеживает файл и автоматически применяет изменения (см. [горячую перезагрузку](#config-hot-reload)).
  </Tab>
</Tabs>

## Строгая валидация

<Warning>
OpenClaw принимает только конфигурации, полностью соответствующие схеме. Неизвестные ключи, неверные типы или недопустимые значения приводят к тому, что Gateway **отказывается запускаться**. Единственное исключение на корневом уровне — `$schema` (string), чтобы редакторы могли подключать JSON Schema.
</Warning>

При ошибке валидации:

- Gateway не запускается
- Работают только диагностические команды (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Запустите `openclaw doctor`, чтобы увидеть точные проблемы
- Запустите `openclaw doctor --fix` (или `--yes`) для автоматического исправления

## Частые задачи

<AccordionGroup>
  <Accordion title="Настроить канал (WhatsApp, Telegram, Discord и др.)">
    У каждого канала свой раздел конфигурации в `channels.<provider>`. См. страницу нужного канала:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Все каналы используют одинаковый шаблон политики DM:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // только для allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Выбрать и настроить модели">
    Укажите основную модель и резервные:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` определяет каталог моделей и действует как allowlist для `/model`.
    - Ссылки на модели используют формат `provider/model` (например, `anthropic/claude-opus-4-6`).
    - См. [Models CLI](/concepts/models) для переключения моделей в чате и [Model Failover](/concepts/model-failover) для ротации авторизации и fallback-поведения.
    - Для кастомных/self-hosted провайдеров см. [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls).

  </Accordion>

  <Accordion title="Управлять тем, кто может писать боту">
    Доступ к DM настраивается отдельно для каждого канала через `dmPolicy`:

    - `"pairing"` (по умолчанию): неизвестные отправители получают одноразовый код подтверждения
    - `"allowlist"`: только отправители из `allowFrom` (или из хранилища сопряжений)
    - `"open"`: разрешить все входящие DM (требуется `allowFrom: ["*"]`)
    - `"disabled"`: игнорировать все DM

    Для групп используйте `groupPolicy` + `groupAllowFrom` или allowlist’ы конкретного канала.

    См. [полный справочник](/gateway/configuration-reference#dm-and-group-access).

  </Accordion>

  <Accordion title="Настроить gating упоминаний в группах">
    По умолчанию сообщения в группах **требуют упоминания**. Настройка на уровне агента:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Metadata mentions**: нативные @‑упоминания (WhatsApp tap-to-mention, Telegram @bot и т. д.)
    - **Text patterns**: regex-шаблоны в `mentionPatterns`
    - См. [полный справочник](/gateway/configuration-reference#group-chat-mention-gating).

  </Accordion>

  <Accordion title="Настроить сессии и сбросы">
    Сессии управляют непрерывностью диалога и изоляцией:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // рекомендуется для нескольких пользователей
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - См. [Session Management](/concepts/session) и [полный справочник](/gateway/configuration-reference#session).

  </Accordion>

  <Accordion title="Включить песочницу (sandbox)">
    Запускать сессии агента в изолированных Docker-контейнерах:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    Сначала соберите образ: `scripts/sandbox-setup.sh`

    См. [Sandboxing](/gateway/sandboxing) и [полный справочник](/gateway/configuration-reference#sandbox).

  </Accordion>

  <Accordion title="Настроить heartbeat (периодические проверки)">
    ```json5
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

    - `every`: строка длительности (`30m`, `2h`). Установите `0m`, чтобы отключить.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - См. [Heartbeat](/gateway/heartbeat).

  </Accordion>

  <Accordion title="Настроить cron-задачи">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    См. [Cron jobs](/automation/cron-jobs).

  </Accordion>

  <Accordion title="Настроить вебхуки (hooks)">
    Включить HTTP-вебхуки на Gateway:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    См. [полный справочник](/gateway/configuration-reference#hooks).

  </Accordion>

  <Accordion title="Мультиагентная маршрутизация">
    Запуск нескольких изолированных агентов с разными workspace и сессиями:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    См. [Multi-Agent](/concepts/multi-agent) и [полный справочник](/gateway/configuration-reference#multi-agent-routing).

  </Accordion>

  <Accordion title="Разделить конфиг на несколько файлов ($include)">
    Используйте `$include` для организации больших конфигов:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **Один файл**: заменяет содержащий объект
    - **Массив файлов**: глубокое слияние по порядку (поздние переопределяют ранние)
    - **Соседние ключи**: объединяются после include (переопределяют значения)
    - **Вложенные include**: поддерживаются до 10 уровней
    - **Относительные пути**: относительно включающего файла
    - **Ошибки**: понятные сообщения для отсутствующих файлов, ошибок парсинга и циклов include

  </Accordion>
</AccordionGroup>

## Горячая перезагрузка конфига

Gateway отслеживает `~/.openclaw/openclaw.json` и автоматически применяет изменения — перезапуск вручную обычно не требуется.

### Режимы перезагрузки

| Режим                  | Поведение                                                                 |
| ---------------------- | -------------------------------------------------------------------------- |
| **`hybrid`** (по умолчанию) | Безопасные изменения применяются сразу. Критические — с автоперезапуском. |
| **`hot`**              | Применяются только безопасные изменения. При необходимости перезапуска выводится предупреждение. |
| **`restart`**          | Gateway перезапускается при любом изменении конфига.                     |
| **`off`**              | Отслеживание отключено. Изменения вступают в силу после ручного рестарта. |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Что применяется «на горячую», а что требует перезапуска

Большинство полей применяются без простоя. В режиме `hybrid` изменения, требующие рестарта, выполняются автоматически.

| Категория           | Поля                                                                 | Нужен рестарт? |
| ------------------- | -------------------------------------------------------------------- | -------------- |
| Каналы              | `channels.*`, `web` (WhatsApp) — все встроенные и расширения       | Нет            |
| Агент и модели     | `agent`, `agents`, `models`, `routing`                               | Нет            |
| Автоматизация      | `hooks`, `cron`, `agent.heartbeat`                                   | Нет            |
| Сессии и сообщения | `session`, `messages`                                                | Нет            |
| Инструменты и медиа| `tools`, `browser`, `skills`, `audio`, `talk`                        | Нет            |
| UI и прочее        | `ui`, `logging`, `identity`, `bindings`                              | Нет            |
| Сервер Gateway     | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Да**         |
| Инфраструктура     | `discovery`, `canvasHost`, `plugins`                                 | **Да**         |

<Note>
`gateway.reload` и `gateway.remote` — исключения: их изменение **не** вызывает перезапуск.
</Note>

## Environment variables

OpenClaw читает переменные окружения из родительского процесса, а также:

- `.env` из текущей рабочей директории (если есть)
- `~/.openclaw/.env` (глобальный fallback)

Ни один `.env` не переопределяет уже существующие переменные окружения. Можно также задать inline‑переменные в конфиге:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Импорт env из shell (опционально)">
  Если включено и ожидаемые ключи не заданы, OpenClaw запускает login-shell и импортирует только отсутствующие переменные:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Эквивалент через env: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Подстановка env-переменных в значениях конфига">
  Ссылаться на env-переменные можно в любом строковом значении через `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Правила:

- Поддерживаются только имена в верхнем регистре: `[A-Z_][A-Z0-9_]*`
- Отсутствующие/пустые переменные вызывают ошибку загрузки
- Экранирование: `$${VAR}` для вывода литерала
- Работает внутри `$include`
- Inline-подстановка: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

См. [Environment](/help/environment) для полного порядка приоритета.

## Полный справочник

Для полного описания всех полей см. **[Configuration Reference](/gateway/configuration-reference)**.

---

_Связано: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_