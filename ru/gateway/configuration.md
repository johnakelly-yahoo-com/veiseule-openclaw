---
title: "Конфигурация"
---

# Конфигурация 🔧

OpenClaw читает необязательный конфиг **JSON5** из `~/.openclaw/openclaw.json` (разрешены комментарии и висячие запятые).

Если файл отсутствует, OpenClaw использует «достаточно безопасные» значения по умолчанию (встроенный агент Pi + сессии на отправителя + рабочее пространство `~/.openclaw/workspace`). Обычно конфиг нужен, чтобы:

- ограничить, кто может вызывать бота (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` и т. д.)
- управлять allowlist’ами групп и поведением упоминаний (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- настраивать префиксы сообщений (`messages`)
- задать рабочее пространство агента (`agents.defaults.workspace` или `agents.list[].workspace`)
- настроить значения по умолчанию встроенного агента (`agents.defaults`) и поведение сессий (`session`)
- задать идентичность для каждого агента (`agents.list[].identity`)

> **Новичкам в конфигурации?** Посмотрите руководство [Configuration Examples](/gateway/configuration-examples) с полными примерами и подробными объяснениями!

## Строгая проверка конфигурации

OpenClaw принимает только конфигурации, полностью соответствующие схеме.
Неизвестные ключи, неверные типы или недопустимые значения приводят к тому, что Gateway **отказывается запускаться** из соображений безопасности.

При неудачной проверке:

- Gateway не загружается.
- Разрешены только диагностические команды (например: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Запустите `openclaw doctor`, чтобы увидеть точные проблемы.
- Запустите `openclaw doctor --fix` (или `--yes`) для применения миграций/исправлений.

Doctor никогда не записывает изменения, если вы явно не выбрали `--fix`/`--yes`.

## Схема + подсказки UI

Gateway предоставляет представление JSON Schema для конфига через `config.schema` для UI‑редакторов.
Control UI рендерит форму из этой схемы и предлагает редактор **Raw JSON** как запасной вариант.

Плагины каналов и расширения могут регистрировать схему и UI‑подсказки для своего конфига, чтобы
настройки каналов оставались schema‑driven во всех приложениях без жёстко заданных форм.

Подсказки (метки, группировка, чувствительные поля) поставляются вместе со схемой, чтобы клиенты
могли рендерить лучшие формы без жёстко закодированных знаний о конфиге.

## Применение + перезапуск (RPC)

Используйте `config.apply`, чтобы проверить и записать полный конфиг и перезапустить Gateway за один шаг.
Он записывает «сторожок» перезапуска и пингует последнюю активную сессию после возврата Gateway.

Предупреждение: `config.apply` заменяет **весь конфиг**. Если нужно изменить лишь несколько ключей,
используйте `config.patch` или `openclaw config set`. Храните резервную копию `~/.openclaw/openclaw.json`.

Params:

- `raw` (string) — полезная нагрузка JSON5 для всего конфига
- `baseHash` (необязательно) — хэш конфига из `config.get` (обязателен, если конфиг уже существует)
- `sessionKey` (необязательно) — ключ последней активной сессии для «пробуждающего» пинга
- `note` (необязательно) — примечание для сторожка перезапуска
- `restartDelayMs` (необязательно) — задержка перед перезапуском (по умолчанию 2000)

Пример (через `gateway call`):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Частичные обновления (RPC)

Используйте `config.patch`, чтобы объединить частичное обновление с существующим конфигом, не затирая
несвязанные ключи. Применяется семантика JSON merge patch:

- объекты объединяются рекурсивно
- `null` удаляет ключ
- массивы заменяются
  Как и `config.apply`, выполняется валидация, запись конфига, сохранение сторожка перезапуска и планирование
  перезапуска Gateway (с необязательным «пробуждением», если указан `sessionKey`).

Params:

- `raw` (string) — полезная нагрузка JSON5 только с изменяемыми ключами
- `baseHash` (обязательно) — хэш конфига из `config.get`
- `sessionKey` (необязательно) — ключ последней активной сессии для пинга
- `note` (необязательно) — примечание для сторожка перезапуска
- `restartDelayMs` (необязательно) — задержка перед перезапуском (по умолчанию 2000)

Пример:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## Минимальный конфиг (рекомендуемая отправная точка)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Один раз соберите образ по умолчанию:

```bash
scripts/sandbox-setup.sh
```

## Режим самочата (рекомендуется для контроля групп)

Чтобы бот не отвечал на WhatsApp @‑упоминания в группах (отвечать только на конкретные текстовые триггеры):

```json5
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

## Включения конфига (`$include`)

Разделяйте конфиг на несколько файлов с помощью директивы `$include`. Это полезно для:

- организации больших конфигов (например, определения агентов по клиентам)
- совместного использования общих настроек между окружениями
- раздельного хранения чувствительных настроек

### Базовое использование

```json5
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

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### Поведение слияния

- **Один файл**: заменяет объект, содержащий `$include`
- **Массив файлов**: глубокое слияние по порядку (поздние файлы переопределяют ранние)
- **С соседними ключами**: соседние ключи объединяются после include (переопределяют включённые значения)
- **Соседние ключи + массивы/примитивы**: не поддерживается (включаемое содержимое должно быть объектом)

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### Вложенные включения

Включаемые файлы сами могут содержать директивы `$include` (до 10 уровней):

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### Разрешение путей

- **Относительные пути**: разрешаются относительно включающего файла
- **Абсолютные пути**: используются как есть
- **Родительские каталоги**: ссылки `../` работают как ожидается

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### Обработка ошибок

- **Отсутствующий файл**: понятная ошибка с разрешённым путём
- **Ошибка парсинга**: показывает, какой включённый файл не прошёл
- **Циклические включения**: обнаруживаются и сообщаются с цепочкой include

### Пример: мультиклиентская юридическая конфигурация

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## Общие параметры

### Env vars + `.env`

OpenClaw читает переменные окружения из родительского процесса (shell, launchd/systemd, CI и т. п.).

Дополнительно загружается:

- `.env` из текущего рабочего каталога (если присутствует)
- глобальный fallback `.env` из `~/.openclaw/.env` (также известен как `$OPENCLAW_STATE_DIR/.env`)

Ни один файл `.env` не переопределяет уже существующие переменные окружения.

Также можно задать inline‑переменные окружения в конфиге. Они применяются только если
переменной нет в окружении процесса (то же правило непереопределения):

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

См. [/environment](/help/environment) для полного порядка приоритета и источников.

### `env.shellEnv` (необязательно)

Опциональное удобство: если включено и ни один из ожидаемых ключей ещё не задан, OpenClaw
запускает ваш login‑shell и импортирует только отсутствующие ожидаемые ключи (никогда не переопределяет).
Фактически это «source» вашего профиля shell.

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

Env var эквивалент:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### Подстановка env‑переменных в конфиге

Вы можете ссылаться на переменные окружения прямо в любом строковом значении конфига,
используя синтаксис `${VAR_NAME}`. Подстановка выполняется при загрузке конфига, до валидации.

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

**Правила:**

- Сопоставляются только имена env‑переменных в верхнем регистре: `[A-Z_][A-Z0-9_]*`
- Отсутствующие или пустые env‑переменные вызывают ошибку при загрузке конфига
- Экранируйте с помощью `$${VAR}`, чтобы вывести литерал `${VAR}`
- Работает с `$include` (включённые файлы тоже проходят подстановку)

**Встроенная подстановка:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### Хранилище аутентификации (OAuth + ключи API)

OpenClaw хранит **для каждого агента** профили аутентификации (OAuth + ключи API) в:

- `<agentDir>/auth-profiles.json` (по умолчанию: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

См. также: [/concepts/oauth](/concepts/oauth)

Импорт устаревшего OAuth:

- `~/.openclaw/credentials/oauth.json` (или `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

Встроенный агент Pi поддерживает runtime‑кэш по адресу:

- `<agentDir>/auth.json` (управляется автоматически; не редактируйте вручную)

Каталог устаревшего агента (до multi‑agent):

- `~/.openclaw/agent/*` (мигрируется `openclaw doctor` в `~/.openclaw/agents/<defaultAgentId>/agent/*`)

Переопределяет:

- Каталог OAuth (только импорт устаревшего): `OPENCLAW_OAUTH_DIR`
- Каталог агента (переопределение корня агента по умолчанию): `OPENCLAW_AGENT_DIR` (предпочтительно), `PI_CODING_AGENT_DIR` (устар.)

При первом использовании OpenClaw импортирует записи `oauth.json` в `auth-profiles.json`.

### `auth`

Необязательные метаданные для профилей аутентификации. **Секреты не хранятся**; сопоставляются
ID профилей с провайдером и режимом (и опциональным email) и задаётся порядок ротации провайдеров
для failover.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

Необязательная идентичность для каждого агента, используемая для значений по умолчанию и UX. Записывается помощником онбординга для macOS.

Если задано, OpenClaw выводит значения по умолчанию (только если вы не задали их явно):

- `messages.ackReaction` из `identity.emoji` **активного агента** (fallback 👀)
- `agents.list[].groupChat.mentionPatterns` из `identity.name`/`identity.emoji` агента (чтобы «@Samantha» работало в группах Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp)
- `identity.avatar` принимает путь к изображению относительно workspace или удалённый URL/data URL. Локальные файлы должны находиться внутри workspace агента.

`identity.avatar` принимает:

- путь относительно workspace (должен оставаться внутри workspace агента)
- URL `http(s)`
- URI `data:`

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

Метаданные, записываемые мастерами CLI (`onboard`, `configure`, `doctor`).

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- Файл логов по умолчанию: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- Если нужен стабильный путь, задайте `logging.file` равным `/tmp/openclaw/openclaw.log`.
- Вывод в консоль настраивается отдельно через:
  - `logging.consoleLevel` (по умолчанию `info`, повышается до `debug` при `--verbose`)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- Сводки инструментов можно редактировать, чтобы избежать утечки секретов:
  - `logging.redactSensitive` (`off` | `tools`, по умолчанию: `tools`)
  - `logging.redactPatterns` (массив regex‑строк; переопределяет значения по умолчанию)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

Управляет обработкой личных чатов WhatsApp (DMs):

- `"pairing"` (по умолчанию): неизвестные отправители получают код сопряжения; владелец должен одобрить
- `"allowlist"`: разрешать только отправителей из `channels.whatsapp.allowFrom` (или хранилища сопряжений)
- `"open"`: разрешить все входящие DM (**требуется**, чтобы `channels.whatsapp.allowFrom` включал `"*"`)
- `"disabled"`: игнорировать все входящие DM

Коды сопряжения истекают через 1 час; бот отправляет код только при создании нового запроса. Ожидающие запросы сопряжения DM по умолчанию ограничены **3 на канал**.

Одобрения сопряжения:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

Allowlist номеров E.164, которые могут запускать автоответы WhatsApp (**только DMs**).
Если пусто и `channels.whatsapp.dmPolicy="pairing"`, неизвестные отправители получат код сопряжения.
Для групп используйте `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom`.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

Определяет, помечаются ли входящие сообщения WhatsApp как прочитанные (синие галочки). По умолчанию: `true`.

Режим самочата всегда пропускает квитанции о прочтении, даже если включены.

Переопределение для аккаунта: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (multi‑account)

Запуск нескольких аккаунтов WhatsApp в одном Gateway:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

Примечания:

- Исходящие команды по умолчанию используют аккаунт `default`, если присутствует; иначе — первый настроенный id (по сортировке).
- Устаревший single‑account каталог Baileys мигрируется `openclaw doctor` в `whatsapp/default`.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

Запуск нескольких аккаунтов на канал (у каждого аккаунта свои `accountId` и необязательные `name`):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

Примечания:

- `default` используется, когда `accountId` опущен (CLI + маршрутизация).
- Токены из env применяются только к **аккаунту по умолчанию**.
- Базовые настройки канала (политика групп, gating упоминаний и т. п.) применяются ко всем аккаунтам, если не переопределены для аккаунта.
- Используйте `bindings[].match.accountId`, чтобы направлять каждый аккаунт к разным agents.defaults.

### Gating упоминаний в групповых чатах (`agents.list[].groupChat` + `messages.groupChat`)

Сообщения в группах по умолчанию **требуют упоминания** (метаданные упоминаний или regex‑шаблоны). Применяется к групповым чатам WhatsApp, Telegram, Discord, Google Chat и iMessage.

**Типы упоминаний:**

- **Упоминания метаданных**: нативные @‑упоминания платформы (например, WhatsApp tap‑to‑mention). Игнорируются в режиме самочата WhatsApp (см. `channels.whatsapp.allowFrom`).
- **Текстовые шаблоны**: regex‑шаблоны, определённые в `agents.list[].groupChat.mentionPatterns`. Проверяются всегда, независимо от режима самочата.
- Gating упоминаний применяется только когда обнаружение упоминаний возможно (нативные упоминания или хотя бы один `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` задаёт глобальное значение по умолчанию для контекста истории групп. Каналы могут переопределять через `channels.<channel>.historyLimit` (или `channels.<channel>.accounts.*.historyLimit` для multi‑account). Установите `0`, чтобы отключить оборачивание истории.

#### Лимиты истории DMs

Диалоги DMs используют историю на основе сессий, управляемую агентом. Можно ограничить число пользовательских ходов, сохраняемых на сессию DM:

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

Порядок разрешения:

1. Переопределение для конкретного DM: `channels.<provider>.dms[userId].historyLimit`
2. Значение провайдера по умолчанию: `channels.<provider>.dmHistoryLimit`
3. Без ограничений (вся история сохраняется)

Поддерживаемые провайдеры: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

Переопределение для агента (имеет приоритет при установке, даже над `[]`):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

Значения gating упоминаний по умолчанию задаются для каждого канала (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). Когда задан `*.groups`, он также действует как allowlist групп; включите `"*"`, чтобы разрешить все группы.

Чтобы отвечать **только** на конкретные текстовые триггеры (игнорируя нативные @‑упоминания):

```json5
{
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### Политика групп (по каналам)

Используйте `channels.*.groupPolicy`, чтобы управлять тем, принимаются ли сообщения из групп/комнат вообще:

```json5
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

Примечания:

- `"open"`: группы обходят allowlist’ы; gating упоминаний всё ещё применяется.
- `"disabled"`: блокировать все сообщения из групп/комнат.
- `"allowlist"`: разрешать только группы/комнаты, соответствующие allowlist’у.
- `channels.defaults.groupPolicy` задаёт значение по умолчанию, когда `groupPolicy` провайдера не задан.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams используют `groupAllowFrom` (fallback: явный `allowFrom`).
- Discord/Slack используют allowlist’ы каналов (`channels.discord.guilds.*.channels`, `channels.slack.channels`).
- Групповые DMs (Discord/Slack) всё ещё управляются через `dm.groupEnabled` + `dm.groupChannels`.
- Значение по умолчанию — `groupPolicy: "allowlist"` (если не переопределено `channels.defaults.groupPolicy`); если allowlist не настроен, сообщения групп блокируются.

### Мульти-агентская маршрутизация (`agents.list` + `bindings`)

Запускать несколько изолированных агентов (отдельное рабочее пространство, `agentDir`, сессии) внутри одного шлюза.
Входящие сообщения направляются в агент через привязки.

- gateway/configuration.md
  - `id`: стабильный идентификатор агента (требуется).
  - `default`: необязательно; когда установлено несколько побед, регистрируются первые победы и предупреждение.
    Если ничего не установлено, **первая запись** в списке является агентом по умолчанию.
  - `name`: отображаемое имя агента.
  - `workspace`: по умолчанию `~/.openclaw/workspace-<agentId>` (для `main`, вернется к `agents.defaults.workspace`).
  - `agentDir`: по умолчанию `~/.openclaw/agents/<agentId>/agent`.
  - `model`: модель по умолчанию для агента, переопределяет `agents.defaults.model` для этого агента.
    - строка формы: `"provider/model"`, переопределяет только `agents.defaults.model.primary`
    - объектная форма: `{ primary, fallbacks }` (опции, переопределяющие `agents.defaults.model.fallbacks`; `[]` отключает глобальные запасы для этого агента)
  - `identity`: имя каждого агента/тема/эмодзи (используется для шаблонов упоминаний + реакции на экран).
  - `groupChat`: упоминание в качестве агента (`mentionPatterns`).
  - `sandbox`: конфигурация per-agent sandbox (переопределяет `agents.defaults.sandbox`).
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"агент"` | `"Shared"`
    - `workspaceRoot`: пользовательский корень рабочей области песочницы
    - `docker`: per-agent docker перезаписывает (например, `image`, `network`, `env`, `setupCommand`, ограничения; игнорировать когда `scope: "shared"`)
    - `browser`: переопределения браузера для агента (игнорируются при `scope: "shared"`)
    - `prune`: песочница для каждого агента, вызвавшего перегрузку (игнорировались при `scope: "shared"`)
  - `subagents`: субагент по умолчанию.
    - `allowAgents`: список идентификаторов агентов для `sessions_spawn` этого агента (`["*"]` = разрешить любое; по умолчанию: только тот же агент)
  - `tools`: ограничения инструментальных средств для каждого агента (применяемые до политики инструментов "песочница").
    - `profile`: профиль базовых инструментов (применяется перед разрешением/отрицанием)
    - `allow`: массив допустимых названий инструментов
    - `deny`: массив запрещенных названий инструментов (запретить победы)
- `agents.defaults`: общие настройки агентов (модель, рабочая среда, песочница и т.д.).
- `bindings[]`: направляет входящие сообщения к `agentId`.
  - `match.channel` (обязательно)
  - `match.accountId` (опционально; `*` = любой аккаунт; пропущен = аккаунт по умолчанию)
  - `match.peer` (опционально; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (опционально; канал конкретный)

Порядок совпадения с определениями:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (точно, нет пира/гильдия/команда)
5. `match.accountId: "*"` (канал по ширине, нет пира/гильдия/команда)
6. агент по умолчанию (`agents.list[].default`, ещё первая запись списка, иначе `"main"`)

В каждом матче выигрывает первая совпадающая запись в «bindings».

#### Профили доступа к агентам (многоагентный)

Каждый агент может иметь свою собственную систему "песочница" + инструментальную политику. Используйте это для смешивания доступа к
уровней в одном шлюзе:

- **Полный доступ** (персональный агент)
- **Только чтение** инструментов + рабочей области
- **Нет доступа к файловой системе** (только для сообщений/сессий)

См. [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) для сравнения и
для дополнительных примеров.

Полный доступ (без песочницы):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

Инструменты только для чтения + рабочее пространство только для чтения:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

Нет доступа к файловой системе (сообщения и сессионные инструменты включены):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

Пример: две учетные записи WhatsApp → два агента:

```json5
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

### `tools.agentToAgent` (опционально)

Агент отправляет сообщения агента по запросу:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### «messages.queue»

Определяет поведение входящих сообщений при запуске агента уже активно.

```json5
{
  messages: {
    queue: {
      режим: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      : "Суммарное", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        телеграмма: "собирать",
        диспропорция: "собирать",
        изображение: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

Подавляет частые входящие сообщения от **одного и того же отправителя**, чтобы несколько сообщений подряд превращались в один ход агента. Дебаунсинг применяется в рамках канала + диалога
и использует самое последнее сообщение для трединга ответов/ID.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 отключает
      byChannel: {
        whatsapp: 5000,
        штрих: 1500,
        дискос: 1500,
      },
    },
  },
}
```

Notes:

- Пакеты Debounce **только текстовые** сообщений; медиа/прикрепленные файлы немедленно сброшены.
- Команды управления (например: `/queue`, `/new`) обойдуются, так что они останутся в автономном режиме.

### `команды` (обработка команд чата)

Управляет включением команд чата между коннекторами.

```json5
{
  commands: {
    native: "auto", // регистрировать родные команды при их поддержке (auto)
    текст: true, // разбор косой черты команд в сообщениях в чате
    bash: false, // allow ! (псевдонимы: /bash) (только для хоста; требует инструментов. levated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediate)
    : false, // разрешить /config (запись на диск)
    отлаживать: false, // разрешить /debug (переопределение только runtime-only)
    перезапустить: false, // разрешить /restart + инструмент перезапуска шлюза
    useAccessGroups: true, Принудительно использовать разрешенные группы разрешить/политики для команд
  },
}
```

Notes:

- Текстовые команды должны быть отправлены как **стандартные** сообщения и использовать ведущий `/` (нет псевдонимов обычного текста).
- `commands.text: false` отключает парсинг сообщений для команд.
- `commands.native: "auto"` (по умолчанию) включает родные команды для Discord/Telegram и оставляет Slack выключен; неподдерживаемые каналы остаются только текстовыми.
- Установите `commands.native: true|false` чтобы заставить всех или переопределить для каждого канала `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native` (bool или `auto"`). `false` очищает ранее зарегистрированные команды при запуске Discord/Telegram; Команды Slack управляются в Slack app.
- `channels.telegram.customCommands` добавляет дополнительные записи меню Telegram. Имена нормализируются; конфликты с родными командами игнорируются.
- `commands.bash: true` разрешает `! <cmd>` для запуска команд оболочки хоста (алиас `/bash <cmd>` также работает). Требуется `tools.elevated.enabled` и разрешает отправителя в `tools.elevated.allowFrom.<channel>`.
- `commands.bashForegroundMs` управляет, сколько ожиданий bash перед фоном. Пока bash работает, новая `! Запросы `&lt;cmd&gt;\` отклоняются (по одному за раз).
- `commands.config: true` поддерживает `/config` (reads/writes `openclaw.json`).
- `каналы.<provider>.configWrites` переключает мутации, инициированные этим каналом (по умолчанию: true). Это применимо к `/config set|unset` плюс специфические автомиграции провайдера (изменения ID супергруппы Telegram, изменения ID канала Slack ).
- `commands.debug: true` активирует `/debug` (только для runtime-overrides).
- `commands.restart: true` включает `/restart` и действие перезапуска инструмента шлюза.
- `commands.useAccessGroups: false` позволяет командам обойти access-group allowlists/policies.
- Slash‑команды и директивы обрабатываются только для **авторизованных отправителей**. Авторизация определяется на основе allowlist/спаривания каналов плюс `commands.useAccessGroups`.

### `web` (время работы веб-канала WhatsApp)

WhatsApp работает через веб-канал шлюза (Baileys Web). Она запускается автоматически при наличии связанного сеанса.
Установите `web.enabled: false`, чтобы отключить его по умолчанию.

```json5
{
  web: {
    включен: true,
    heartbeatSeconds: 60,
    переподключение: {
      initialMs: 2000,
      maxMs: 120000,
      фактор: 1. ,
      jitter: 0. ,
      maxAttemps: 0,
    },
  },
}
```

### `channels.telegram` (транспорт бота)

OpenClaw запускает Telegram только при наличии секции конфигурации «channels.telegram». Токен бота разрешен из `channels.telegram.botToken` (или `channels.telegram.tokenFile`), а `TELEGRAM_BOT_TOKEN` - из резервной копии по умолчанию.
Установите `channels.telegram.enabled: false`, чтобы отключить автоматический запуск.
Поддержка нескольких аккаунтов живет в `channels.telegram.accounts` (см. раздел мультиаккаунтов выше). Маркеры Env применяются только к учетной записи по умолчанию.
Установите `channels.telegram.configWrites: false` чтобы блокировать записи настроек, созданных по инициативе Telegram, (включая перенос ID супергруппы и `/config set|unset`).

```json5
{
  каналов: {
    телеграмма: {
      включен: true,
      botToken: "your-bot-token",
      dmPolicy: "Объединение", // сопряжение | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // необязательно; "open" требует ["*"]
      групп: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          Системная подсказка: "Храните ответы краткими. ,
          темы: {
            "99": {
              requireMention: false,
              навыков: ["искать"],
              Системная подсказка: "Оставайтесь на теме. ,
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", описание: "Создать изображение" },
      ],
      Ограничение истории: 50, // включаем последние N групповые сообщения в виде контекста (0 отключений)
      replyToMode: "first", // выкл | первый | все
      linkPreview: true, // переключение просмотра исходящей ссылки
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; только для streamMode=block
        minChars: 200,
        maxChars: 800,
        breakпредпочтение: "paragraph ", // параграф | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // инструментальные ворота (ложные отключения)
      reactionNotifications: "own", // выкл | все
      mediaMaxMb: 5,
      retry: {
        // политика исходящих повторов
        попытки: 3,
        минуты задержки: 400,
        maxDelayMs: 30000,
        jitter: 0. ,
      },
      network: {
        // транспортное переопределение
        autoSelectFamily: false,
      },
      прокси: "socks5://localhost:9050",
      webhookUrl: "https://example. om/telegram-webhook", // Требуется webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

Проект потоковых примечаний:

- Использует Telegram `sendMessageDraft` (черновик, а не настоящее сообщение).
- Требует **приватные темы** (message_thread_id в DMs; у бота есть включенные темы).
- `/reasoning stream` поток, рассуждая в черновик, затем отправляет окончательный ответ.
  Повторная политика по умолчанию и поведение описаны в [Retry policy](/concepts/retry).

### `channels.discord` (транспорт бота)

Настройте бота Discord, установив токен бота и опционально:
Поддержка нескольких аккаунтов живет под `channels.discord.accounts` (см. раздел multi-account выше). Маркеры Env применяются только к учетной записи по умолчанию.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw запускает Discord только при наличии секции конфигурации `channels.discord`. Токен разрешен из `channels.discord.token`, с именем `DISCORD_BOT_TOKEN` как резервный вариант для учетной записи по умолчанию (если `channels.discord.enabled` не является `false`). Используйте `user:<id>(DM) или `channel:&lt;id&gt;`(канал гильдии), чтобы указать цели доставки для команд cron/CLI; пустые числовые ID являются неоднозначными и отвергнуты.
Спучки гильдии - строчные буквы, где пробелы заменены на`-`; клавиши каналов используют название слайд-канала (нет ведущего `#`). Предпочитайте идентификаторы гильдии как ключи во избежание неоднозначности переименования.
Бот авторы сообщений игнорируются по умолчанию. Включите с `channels.discord.allowBots\` (собственные сообщения все еще фильтруются для предотвращения циклов самоответа).
Режимы оповещения о реакции:

- `off`: без событий реакций.
- `own`: реакции на собственные сообщения бота (по умолчанию).
- `all`: все реакции на все сообщения.
- `allowlist`: реакции от `guilds.<id>.users` на всех сообщениях (пустой список отключает).
  Исходящий текст обрезается `channels.discord.textChunkLimit` (по умолчанию 2000). Установите `channels.discord.chunkMode="newline"`, чтобы разделить на пустые строки (границы параграфов) до длины куски. Клиенты Discord могут кликать очень высокие сообщения, поэтому `channels.discord.maxLinesPerMessage` (по умолчанию 17) делится длинными многострочными ответами даже при менее 2000 символов.
  Повторная политика по умолчанию и поведение описаны в [Retry policy](/concepts/retry).

### `channels.googlechat` (webhook)

Google Chat запускается через HTTP-вебхуки с пользовательским идентификатором (сервисная учетная запись).
Поддержка нескольких аккаунтов живет в `channels.googlechat.accounts` (см. раздел multi-account выше). Env vars применимы только к учетной записи по умолчанию.

```json5
{
  каналов: {
    googlechat: {
      включен: true,
      serviceAccountFile: "/path/to/service-account. son",
      аудитория: "app-url", // app-url | project-number
      аудитория: "https://gateway.example. om/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // optional; улучшает определение упоминания
      dm: {
        включен: true,
        Политика "пары данных", // парсинг | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // необязательно; "open" требует ["*"]
      },
      groupPolicy: "allowlist",
      группы: {
        "spaces/AAA": { allow: true, requireMention: true },
      },
      действий: { reactions: true },
      Индикатор печати: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Notes:

- Счет службы JSON может быть встроенным (`serviceAccount`) или основанным на файлах (`serviceAccountFile`).
- Env fallbackback for the default account: `GOOGLE_CHAT_SERVICE_ACCOUNT` or `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- `audienceType` + `audience` должен соответствовать настройке аутентификации webhook в чате.
- Используйте "spaces/&lt;spaceId&gt;" или "users/&lt;userId|email&gt;для задания целей доставки.

### `channels.slack` (режим сокета)

Slack запускается в режиме сокета и требует как токен бота, так и токен приложения:

```json5
{
  каналов: {
    slack: {
      enabled: true,
      botToken: "xoxb-. .",
      appToken: "xapp-... ,
      dm: {
        включен: true,
        политика: "парниковый", // сопряжение | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // необязательно; "open" требует ["*"]
        groupabled: false,
        групповые каналы: ["G123"],
      },
      каналы: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          Требования: true,
          allowBots: false,
          пользователей: ["U123"],
          навыки: ["Документы"],
          systemPrompt: "Только короткие ответы. ,
        },
      },
      Ограничение истории: 50, // включаем последние N каналы/групповые сообщения в виде контекста (0 отключений)
      allowBots: false,
      Уведомления о реакции: "собственные", // выкл | все | список
      реакций Разрешить: ["U123"],
      ответитьToMode: "выкл", // off | first | all
      thread: {
        historyScope: "thread", // поток | канал
        inheritParent: false,
      },
      действия: {
        реакции: true,
        сообщений: true,
        значений: true,
        членская информация: true,
        emojiList: true,
      },
      slashCommand: {
        включен: true,
        имя: "openclaw",
        префикс сессии: "slack:slash",
        эфемер: true,
      },
      textChunkLimit: 4000,
      chunkMode: "длина",
      mediaMaxMb: 20,
    },
  },
}
```

Поддержка нескольких аккаунтов живет в `channels.slack.accounts` (см. раздел multi-account выше). Маркеры Env применяются только к учетной записи по умолчанию.

OpenClaw запускает Slack при включенном провайдере и задаются оба токена (через конфигурацию или `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`). Используйте `user:<id>(DM) или `channel:&lt;id&gt;для задания целей доставки команд cron/CLI.
Установите `channels.slack.configWrites: false` на блокировку сценария конфигурации с Slack-запросом (включая миграцию ID канала и `/config set|unset`).

Бот авторы сообщений игнорируются по умолчанию. Включить с `channels.slack.allowBots` или `channels.slack.channels.<id>.allowBots`.

Режимы оповещения о реакции:

- `off`: без событий реакций.
- `own`: реакции на собственные сообщения бота (по умолчанию).
- `all`: все реакции на все сообщения.
- `allowlist`: реакции от `channels.slack.reactionAllowlist` на всех сообщениях (пустой список выключен).

Изоляция сессии в нитях:

- `channels.slack.thread.historyScope` контролирует является ли история ветви в каждом потоке (`thread`, по умолчанию) или поделится ею по всему каналу (`channel`).
- `channels.slack.thread.inheritParent` контролирует унаследование новых сессий потока, родительских субтитров (по умолчанию: false).

Группы действий недостатка (действия по инструментам `slack`):

| Группа действий | Значение по умолчанию | Notes                                   |
| --------------- | --------------------- | --------------------------------------- |
| reactions       | enabled               | Реакции + список реакций                |
| messages        | enabled               | Чтение/отправка/редактирование/удаление |
| pins            | enabled               | Закрепить/открепить/список              |
| memberInfo      | enabled               | Информация об участниках                |
| emojiList       | enabled               | Список кастомных emoji                  |

### `channels.mattermost` (токен бота)

Mattermost поставляется как плагин и не входит в состав основной установки.
Сначала установите `openclaw plugins install @openclaw/mattermost` (или `./extensions/mattermost` из git checkout).

Самая важная проблема требует токена бота плюс базовый URL-адрес для вашего сервера:

```json5
{
  каналов: {
    mattermost: {
      включен: true,
      botToken: "mm-token",
      baseUrl: "https://chat. xample. om",
      dmPolicy: "сопряжение",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharfixes: [">", "! ],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

OpenClaw запускает Mattermost когда учетная запись настроена (токен бота + базовый URL) и включена. Токен + базовый URL разрешается из `channels.mattermost.botToken` + `channels.mattermost.baseUrl` или `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL` для учетной записи по умолчанию (если `channels.mattermost.enabled` не является `false`).

Режим чата:

- `oncall` (по умолчанию): отвечать на сообщения канала только когда @упоминается.
- `onmessage`: отвечать на каждое сообщение в канале.
- `onchar`: отвечать, когда сообщение начинается с префикса триггера (`channels.mattermost.oncharPrefixes`, по умолчанию `[">", "!"]`).

Контроль доступа:

- По умолчанию: `channels.mattermost.dmPolicy="pairing"` (неизвестные отправители получают код сопряжения).
- Публичные личные сообщения: `channels.mattermost.dmPolicy="open"` плюс `channels.mattermost.allowFrom=["*"]`.
- Группа: `channels.mattermost.groupPolicy="allowlist"` по умолчанию (mention-gated). Используйте `channels.mattermost.groupAllowFrom` для ограничения отправителей.

Поддержка нескольких аккаунтов живет в `channels.mattermost.accounts` (см. раздел multi-account выше). Env vars применимы только к учетной записи по умолчанию.
Используйте «channel:&lt;id&gt;» или «user:&lt;id&gt;» (или «@username») при указании целей доставки; пустые id канала считаются идентификаторами.

### `channels.signal` (сигнал-cli)

Сигнальные реакции могут излучать системные события (общая утилита реакции):

```json5
{
  каналов: {
    signal: {
      reactionNotifications: "own", // выкл | все | список
      реакции Разрешить: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      Limit: 50, // включаем последние N групповых сообщений в виде контекста (0 отключений)
    },
  },
}
```

Режимы оповещения о реакции:

- `off`: без событий реакций.
- `own`: реакции на собственные сообщения бота (по умолчанию).
- `all`: все реакции на все сообщения.
- `allowlist`: реакции из `channels.signal.reactionAllowlist` на всех сообщениях (пустой список выключен).

### `channels.imessage` (imsg CLI)

OpenClaw создает `imsg rpc` (JSON-RPC над stdio). Демон или порт не требуется.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat. b",
      remoteHost: "user@gateway-host", // SCP для удаленных вложений при использовании оболочки SSH
      dmPolicy: "pairing", // сопряжение | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example. om", "chat_id:123"],
      Ограничение истории: 50, // включать последние N групповые сообщения как контекстные (0 отключений)
      includeAttachments: false,
      mediaMaxMb: 16,
      Услуги: "Авто",
      регион: "США",
    },
  },
}
```

Поддержка нескольких аккаунтов живет в `channels.imessage.accounts` (см. раздел multi-account выше).

Notes:

- Требуется полный доступ к БД сообщений.
- Первая отправка будет запрашивать разрешение на автоматизацию сообщений.
- Предпочитайте `chat_id:<id>`. Используйте `imsg chats --limit 20` для списка чатов.
- `channels.imessage.cliPath` может указывать на сценарий wrapper (например, `ssh` на Mac, который запускает `imsg rpc`); используйте ключи SSH во избежание запроса пароля.
- Для удаленных оболочек SSH установите `channels.imessage.remoteHost` чтобы получать вложения через SCP при включении `includeAttachments`.

Пример обёртки:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

Устанавливает **единую глобальную директорию**, используемую агентом для работы с файлами.

По умолчанию: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Если включена `agents.defaults.sandbox`, не-главные сессии могут переопределить это с помощью
собственных областей в разделе `agents.defaults.sandbox.workspaceRoot`.

### `agents.defaults.repoRoot`

Необязательный корень репозитория, чтобы показать в системной строке Runtime подсказки. Если не установлено, OpenClaw
пытается обнаружить каталог `.git`, выйдя вверх из рабочей области (и текущий рабочий каталог
). Этот путь должен быть использован.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Отключает автоматическое создание загрузочных файлов рабочего пространства (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

Используйте это для предварительного развертывания, в котором ваши файлы рабочей области приходят из репозитория.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Максимальное количество символов каждого загрузочного файла рабочей области вставки в системную подсказку
перед усечением. По умолчанию: `20000`.

Когда файл превышает этот лимит, OpenClaw записывает предупреждение и вводит усеченные
головы/хвост с маркером.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

Устанавливает часовой пояс пользователя для **контекста системного промпта** (не для временных меток в конвертах сообщений). Если не установлено, OpenClaw использует часовой пояс хоста во время запуска.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Контролирует **формат времени**, отображаемый в разделе Текущая дата и время.
По умолчанию: `auto` (настройки ОС).

```json5
Эквиваленты в переменных окружения:
```

### `сообщения`

Контролирует входящие/исходящие префиксы и необязательные реакции ack .
Смотрите [Messages](/concepts/messages) для очереди, сессий и потокового контекста.

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix` применяется к **всем исходящим ответам** (сводки инструментов, блок
потокового вещания, окончательные ответы) по каналам, если они еще не присутствуют.

Переопределения можно настроить для каждого канала и для каждого аккаунта:

- `channels.<channel>.responsePrefix`
- `channels.<channel>.accounts.<id>.responsePrefix`

Порядок разрешения (самый специфичный имеет приоритет):

1. `channels.<channel>.accounts.<id>.responsePrefix`
2. `channels.<channel>.responsePrefix`
3. `messages.responsePrefix`

При сбое валидации:

- `undefined` проходит на следующий уровень.
- `""` явно отключает префикс и останавливает каскад.
- `"auto"` получает `[{identity.name}]` для маршрутизированного агента.

Переопределения применяются ко всем каналам, включая расширения, и ко всем исходящим ответам.

Если `messages.responsePrefix` отключен, префикс не применяется по умолчанию. WhatsApp self-chat
Ответы являются исключением: по умолчанию они `[{identity.name}]` при установке, в противном случае
`[openclaw]`, поэтому разные разговоры остаются разборчивыми.
Установите его в `"auto"` для получения `[{identity.name}]` для маршрутизированного агента (когда установлен).

#### Переменные шаблона

Строка `responsePrefix` может включать в себя переменные шаблона, которые динамически преобразуются:

| Переменная                        | Description                 | Пример                                     |
| --------------------------------- | --------------------------- | ------------------------------------------ |
| \`{model}                         | Короткое название модели    | `claude-opus-4-6`, `gpt-4o`                |
| \`{modelFull}                     | Полный идентификатор модели | `anthropic/claude-opus-4-6`                |
| \`{provider}                      | Имя провайдера              | `антропия`, `openai`                       |
| \`{thinkingLevel}                 | Текущий уровень мышления    | `high`, `low`, `off`                       |
| \`{identity.name} | Имя сотрудника              | (так же как и "auto"\`) |

Переменные не чувствительны к регистру (`{MODEL}` = `{model}`). `{think}` — это псевдоним для `{thinkingLevel}`.
Неразрешенные переменные остаются буквальным текстом.

```json5
{
  messages: {
    responsePrefix: "[{model} | думает:{thinkingLevel}]",
  },
}
```

Пример: `[claude-opus-4-6 | think:high] Вот мой ответ...`

WhatsApp настроен на входящий префикс через `channels.whatsapp.messagePrefix` (устарело:
`messages.messagePrefix`). По умолчанию остается **неизменным**: `"[openclaw]"`, когда
`channels.whatsapp.allowFrom` пуст, иначе `""` (без префикса). При использовании
`"[openclaw]"`, OpenClaw будет использовать `[{identity.name}]`, когда маршрутизатор
агент имеет набор `identity.name`.

`ackReaction` посылает реакцию эмодзи на распознавание входящих сообщений
на каналах, поддерживающих реакции (Slack/Discord/Telegram/Google Chat). По умолчанию используется `identity.emoji` активного агента, если задано, иначе `"👀"`. Установил `""` для отключения.

`ackReactionScope` управляет при огне реакций:

- `group-mentions` (по умолчанию): только когда группа/комната требует упоминаний **и** бот был упомянут
- `group-все`: все сообщения группы/комнаты
- `direct`: только личные сообщения
- `все`: все сообщения

`removeAckAfterReply` удаляет реакцию на ack бота после отправки ответа
(Только Sack/Discord/Telegram/Google чат). По умолчанию: `false`.

#### `messages.tts`

Включить синтезатор речи для исходящих ответов. Если включено, OpenClaw генерирует аудио
с помощью ElevenLabs или OpenAI и прикрепляет его к ответам. Telegram использует Opus
голосовые заметки; другие каналы отправляют MP3 аудио.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

Notes:

- `messages.tts.auto` управляет auto-TTS (`off`, `always`, `inbound`, `tagged`).
- `/tts off|always|inbound|tagged` устанавливает режим автоматического перехода в пер-сессии (переопределяет конфигурацию).
- `messages.tts.enabled` - доктор мигрирует в `messages.tts.auto`.
- `prefsPath` хранит локальные переопределения (provider/limit/summarize).
- `maxTextLength` — жесткая колпачка для TTS; резюме усекаются по форме.
- `summaryModel` переопределяет `agents.defaults.model.primary` для авторезюме.
  - Принимает `provider/model` или алиас из `agents.defaults.models`.
- `modelOverrides` включает переопределения моделей типа `[[tts:...]]` тегов (по умолчанию).
- `/tts limit` и `/tts summary` параметры контроля за суммированием каждого пользователя.
- Значения `apiKey` относятся к `ELEVENLABS_API_KEY`/`XI_API_KEY` и `OPENAI_API_KEY`.
- `elevenlabs.baseUrl` переопределяет базовый URL API ElevenLabs.
- `elevenlabs.voiceSettings` поддерживает `stability`/`similarityBoost`/`style` (0..1),
  `useSpeakerBoost`, и `speed` (0.5..2.0).

### `говорить`

По умолчанию для режима Talk (macOS/iOS/Android). Голосовые идентификаторы возвращаются в `ELEVENLABS_VOICE_ID` или `SAG_VOICE_ID` при отключении.
`apiKey` возвращается к `ELEVENLABS_API_KEY` (или профилю оболочки шлюза), когда он отключен.
`voiceAliases` позволяет говорить директивы использовать дружественные имена (например `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Роджер: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

Управление временем работы встроенного агента (модели/мышление/подробности/тайм-ауты).
`agents.defaults.models` определяет конфигурационный каталог моделей (и выступает в качестве допустимого списка для `/model`).
`agents.defaults.model.primary` устанавливает стандартную модель; `agents.defaults.model.fallbacks` - глобальные отказы.
`agents.defaults.imageModel` является необязательным и **используется только в том случае, если в основной модели отсутствует образ ввода**.
Каждая запись `agents.defaults.models` может включать:

- `alias` (ярлык факультативной модели, например `/opus`).
- `params` (опциональные параметры API, переданные через запрос модели).

`params` также применяется к потоковым запускам (embedded agent + compaction). Поддерживаемые сегодня ключи: `temperature`, `maxTokens`. Это слияние с опциями времени вызова; значения, предоставленные звонящим, выиграли. `temperature` — это расширенный узел — оставьте поле пустым, если не знаете значения по умолчанию модели и не нужно изменить.

Пример:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5. ": {
          параметры: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Модели GLM-4.x Z.AI автоматически включают режим мышления, если вы:

- установите `--thinking off`, или
- определите `agents.defaults.models["zai/<model>"].params.thinking` себя.

OpenClaw также поставляет несколько встроенных короткошерстей алиаса. Значения по умолчанию применяются только если модель уже присутствует в `agents.defaults.models`.

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

Если вы настраиваете одно и то же имя (без учета регистра), то ваше значение выигрывает (по умолчанию никогда не переопределение).

Пример: Opus 4.6 первичная с резервным вариантом MiniMax M2.1 (hosted MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2. ": { alias: "minimax" },
      },
      модель: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2. "],
      },
    },
  },
}
```

MiniMax Автор: установите `MINIMAX_API_KEY` (ruv) или настройте `models.providers.minimax`.

#### `agents.defaults.cliBackends` (CLI fallback)

Необязательные CLI backends для резервного запуска (без вызовов инструментов). Это полезно в качестве пути резервного копирования
при сбое поставщиков API. Прохождение изображения поддерживается при настройке
`imageArg`, который принимает пути к файлам.

Notes:

- Бэкэнды CLI **text-first**; инструменты всегда отключены.
- Сессии поддерживаются при установке «sessionArg»; идентификаторы сессий сохраняются на одном бэкэнде.
- Для `claude-cli` по умолчанию встраивается. Переопределяет путь к команде, если PATH минимальный
  (launchd/systemd).

Пример:

```json5
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

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4. ": {
          алиас: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_think : false,
            },
          },
        },
      },
      модель: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "Открытый роутер/meta-llama/lama-3. -70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2. -vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2. -flash-vision:free"],
      },
      думает: "low",
      verboseПо умолчанию: "off", "off",
      по умолчанию: "в",
      таймааут секунд: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        цель: "последний",
      },
      maxConcurrent: 3,
      subagents: {
        модель: "minimax/MiniMax-M2. ",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        очистки: 1800000,
      },
      Токенов контекста: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (pruning)

`agents.defaults.contextPruning` prunes **старые результаты инструментов** из контекста in-memory непосредственно перед отправкой запроса в LLM.
**не** изменяет историю сессий на диске (`*.jsonl` остается завершенным).

Предназначен для уменьшения использования токенов для чатных, накапливающих со временем большие инструменты выхода.

Высокий уровень:

- Никогда не касается сообщений пользователя/помощника.
- Защищает последние сообщения помощников keepLastAssistants (никаких результатов после этого не будет).
- Защищает префикс начальной загрузки (ничто не может быть предварительно обработано первым пользовательским сообщением).
- Режимы:
  - `адаптивная`: мягкие тримбатические результаты (сохранение головы/хвост) при сметном контекстном соотношении пересекает `softTrimRatio`.
    Затем твердый очищает результаты самых старых подходящих инструментов, когда расчетное соотношение контекста пересекает `hardClearRatio` **и**
    есть достаточный объем результата prunable tool-result (`minPrunableToolChars`).
  - `агрессивный`: всегда заменяет подходящий результат перед выходом на "hardClear.placeholder" (без проверки).

Soft vs hard pruning (какие изменения были отправлены LLM):

- **Soft-trim**: только для _чрезмерных результатов инструментов. Сохраняет начало + конец и вставляет `...` в середину.
  - До: `toolResult("…очень длинный вывод…")`
  - После: `toolResult("HEAD…\n...\n…TAIL\n\n[Результат в инструменте: …]")`
- **Сложно очистить**: заменяет результат всего инструмента на место заполнителя.
  - До: `toolResult("…очень длинный вывод…")`
  - После: `toolResult("[Old tool result content cleared]")`

Примечания/нынешние ограничения:

- Результаты инструментов, содержащие **блоки изображений, пропущены** (никогда не сокращается/очищается) прямо сейчас.
- Расчетное «соотношение контекста» основано на **символах** (приблизительно), а не на точном значении.
- Если в сеансе еще нет хотя бы "keepLastAssistants", то пропуск пропущен.
- В режиме `aggressive` игнорируется `hardClear.enabled` (соответствующие результаты инструментов всегда заменяются `hardClear.placeholder`).

По умолчанию (адаптивный):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } },
}
```

Для отключения:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } },
}
```

По умолчанию (когда 'режим' является 'адаптивным' или 'агрессивным'):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (адаптивный только)
- `hardClearRatio`: `0.5` (адаптивное только)
- `minPrunableToolChars`: `50000` (только адаптивный)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (только адаптивный)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

Пример (агрессивный, минимальный):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } },
}
```

Пример (адаптивная настройка):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0. ,
        hardClearRatio: 0. ,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { включен: true, placeholder: "[Old tool result content cleared]" },
        // Optional: restrict pruning to specific tools (deny wins; поддерживает "*" шаблоны)
        инструменты: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

См. [/concepts/session-pruning](/concepts/session-pruning) для деталей поведения.

#### `agents.defaults.compaction` (зарезервировать головной убор + флеш памяти)

`agents.defaults.compaction.mode` выбирает стратегию суммирования. По умолчанию `default`; установите `safeguard`, чтобы включить обобщение в чанках для очень длинных историй. См. [/concepts/compaction](/concepts/compaction).

`agents.defaults.compaction.reserveTokensFloor` создает минимальное значение `reserveTokens`
для Pi уплотнения (по умолчанию: `20000`). Установите на «0», чтобы отключить этаж.

`agents.defaults.compaction.memoryFlush` выполняет **тихий** агентный ход перед авто-сжатием, инструктируя модель сохранить долговременную память на диск (например, `memory/YYYY-MM-DD.md`). Срабатывает, когда оценка токенов сессии пересекает мягкий порог ниже лимита сжатия.

По умолчанию:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: встроенный по умолчанию файл `NO_REPLY`
- Примечание: сброс памяти пропускается, когда рабочее пространство сессии доступно только для чтения (`agents.defaults.sandbox.workspaceAccess: "ro"` или `"none"`).

Пример (настроенный):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          включен: true,
          Токены с мягким порогом: 6000,
          Системная подсказка: "Сессия близка к уплотнению. Сохранять долговечные воспоминания.",
          подсказка: "Напишите любые долговечные заметки в памяти/YYY-MM-DD. d; ответьте с NO_REPLY, если нечего хранить. ,
        },
      },
    },
  },
}
```

Блокировка потокового вещания:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (по умолчанию выключено).

- Переопределение канала: `*.blockStreaming` (и варианты для каждого аккаунта) для принудительного блокирования потокового вещания.
  Каналы не Telegram требуют явного `*.blockStreaming: true` для включения ответов на блоки.

- `agents.defaults.blockStreamingBreak`: `"text_end"` или `"message_end"` (по умолчанию: text_end).

- `agents.defaults.blockStreamingChunk`: мягкий кусок для потоковых блоков. По умолчанию
  800–1200 символов, предпочитает разрывы параграфа (`\n\n`), затем новые строки, а затем предложения.
  Пример:

  ```json5
  {
    agents: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: объединить потоковые блоки перед отправкой.
  По умолчанию `{ idleMs: 1000 }` и наследует `minChars` из `blockStreamingChunk`
  с `maxChars` ограничен в ограничение текста канала. Сигнал/Slack/Discord/Google чат по умолчанию
  на `minChars: 1500` без переопределения.
  Канал перегружен: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `, `channels.googlechat.blockStreamingCoalesce\`
  (и варианты-аккаунты).

- `agents.defaults.humanDelay`: случайная пауза между **ответами блока** после первого.
  Режимы: `off` (по умолчанию), `natural` (800–2500ms), `custom` (используйте `minMs`/`maxMs`).
  Per-agent override: `agents.list[].humanDelay`.
  Пример:

  ```json5
  {
    agents: { humanDelay: { mode: "natural" } },
  }
  ```

  См. [/concepts/streaming](/concepts/streaming) для информации о поведении + чанках.

Набираемые показатели:

- `agents.defaults.typingMode`: `"никогда" | "мгновенный" | "думающий" | "message"`. По умолчанию
  `instant` для прямых чатов/упоминаний и `message` для неуказанных групповых чатов.
- `session.typingMode`: переопределение сеанса для режима.
- `agents.defaults.typingIntervalSeconds`: как часто обновляется печатный сигнал (по умолчанию: 6s).
- `session.typingIntervalSeconds`: переопределение каждого сеанса для интервала обновления.
  См. [/concepts/typing-indicators](/concepts/typing-indicators) для деталей поведения.

Первичное название `agents.defaults.model.primary` следует задать как `provider/model` (например, `anthropic/claude-opus-4-6`).
Псевдонимы получены от `agents.defaults.models.*.alias` (например `Opus`).
Если вы опустите провайдера, OpenClaw в настоящее время предполагает, что `anthropic` является временным
устаревшим вариантом.
Модели Z.AI доступны как `zai/<model>(например `zai/glm-4.7`) и требуют
`ZAI_API_KEY`(или старое`Z_AI_API_KEY\`) в окружении.

`agents.defaults.heartbeat` настраивает периодические настройки heartbeat запуска:

- `every`: длительная строка (`ms`, `s`, `m`, `h`); единицы измерения по умолчанию. По умолчанию:
  `30m`. Установите `0m` для отключения.
- `model`: факультативная модель переопределения пробегов heartbeat (`provider/model`).
- `includeReasoning`: когда `true`, heartbeats также доставят отдельное сообщение `Reasoning:`, когда оно будет доступно (та же фигура, что и `/reasoning on`). По умолчанию: `false`.
- `session`: факультативный ключ сессии для контроля, на какой сессии работает сердечный бот. По умолчанию: `main`.
- `to`: необязательный получатель переопределен (идентификатор отдельного канала, например E.164 для WhatsApp, идентификатор чата для Telegram).
- `target`: факультативный канал доставки (`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). По умолчанию: `last`.
- `prompt`: опционально переопределить для тела heartbeat (по умолчанию: `Read HEARTBEAT.md, если он существует (область рабочей области). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Переопределение отправляется дословно; если вы все еще хотите прочитать файл строкой `Read HEARTBEAT.md`.
- `ackMaxChars`: максимум символов, разрешенных после `HEARTBEAT_OK` перед доставкой (по умолчанию: 300).

Heartbeat для каждого агента:

- Установите `agents.list[].heartbeat`, чтобы включить или переопределить настройки heartbeat для конкретного агента.
- Если какая-либо запись агента определяет `heartbeat`, **только эти агенты** выполняют heartbeat; значения по умолчанию становятся общей базой для этих агентов.

Heartbeat выполняют полноценные ходы агента. Более короткие интервалы расходуют больше токенов; учитывайте `every`, держите `HEARTBEAT.md` минимальным и/или выбирайте более дешёвую `model`.

`tools.exec` настраивает фоновый exec по умолчанию:

- `backgroundMs`: время перед авто-фоном (мс, по умолчанию 10000)
- `timeoutSec`: автоматическое завершение после этого времени выполнения (секунды, по умолчанию 1800)
- `cleanupMs`: как долго хранить завершенные сессии в памяти (мс, по умолчанию 1800000)
- `notifyOnExit`: добавить в очередь системное событие + запрашивать heartbeat при выходах на фоне exec (по умолчанию true)
- `applyPatch.enabled`: включить экспериментальный `apply_patch` (только OpenAI/OpenAI Codex; false)
- `applyPatch.allowModels`: опционально допустимый список идентификаторов модели (например, `gpt-5.2` или `openai/gpt-5.2`)
  Примечание: `apply yPatch` находится только в `tools.exec`.

`tools.web` настраивает веб-поиск + инструменты выборки:

- `tools.web.search.enabled` (по умолчанию: true когда ключ присутствует)
- `tools.web.search.apiKey` (рекомендуется: установить через `openclaw configure --section web`, или использовать `BRAVE_API_KEY` ruv var)
- `tools.web.search.maxResults` (1–10, по умолчанию 5)
- `tools.web.search.timeoutSeconds` (по умолчанию 30)
- `tools.web.search.cacheTtlMinutes` (по умолчанию 15)
- `tools.web.fetch.enabled` (по умолчанию)
- `tools.web.fetch.maxChars` (по умолчанию 50000)
- `tools.web.fetch.maxCharsCap` (по умолчанию 50000; заклинания maxChars из config/tool calls)
- `tools.web.fetch.timeoutSeconds` (по умолчанию 30)
- `tools.web.fetch.cacheTtlMinutes` (по умолчанию 15)
- `tools.web.fetch.userAgent` (необязательное переопределение)
- `tools.web.fetch.readability` (по умолчанию true; отключите, чтобы использовать только базовую HTML-очистку)
- `tools.web.fetch.firecrawl.enabled` (по умолчанию, если установлен ключ API)
- `tools.web.fetch.firecrawl.apiKey` (опционально; по умолчанию `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (по умолчанию [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (по умолчанию true)
- `tools.web.fetch.firecrawl.maxAgeMs` (необязательно)
- `tools.web.fetch.firecrawl.timeoutSeconds` (необязательно)

`tools.media` настраивает входное понимание медиа (image/audio/video):

- `tools.media.models`: общий список моделей (функционал; используется после списков per-cap).
- `tools.media.concurrency`: максимальный параллельный запуск способностей (по умолчанию 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `включено`: отключение переключателя (по умолчанию, когда модели настроены).
  - `prompt`: необязательное переопределение подсказки (image/video добавьте подсказку `maxChars` автоматически).
  - `maxChars`: макс. символы вывода (по умолчанию 500 для изображения/видео; unset for audio).
  - `maxBytes`: максимальный размер мультимедиа для отправки (по умолчанию: изображение 10MB, аудио 20MB, видео 50MB).
  - `timeoutSeconds`: запрос таймаут (по умолчанию: изображение 60s, аудио 60s, видео 120).
  - `language`: опциональная аудиоподсказка.
  - `приложения`: политика вложений (`режим`, `maxAttachments`, `prefer`).
  - `scope`: ворота (первый матч выиграет) с `match.channel`, `match.chatType`, или `match.keyPrefix`.
  - `models`: упорядоченный список элементов модели; неудачи или чрезмерные медиа-файлы возвращаются к следующей записи.
- Каждая запись `models[]`:
  - Провайдер (`type: "provider"` или опущено):
    - `provider`: ID API провайдера (`openai`, `anthropic`, `google`/`gemini`, `groq` и т.д.).
    - `model`: идентификатор модели (необходим для изображения; по умолчанию `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo` для аудио провайдеров и `gemini-3-flash-preview` для видео).
    - `profile` / `preferredProfile`: выбор профиля авторизации.
  - Запись CLI (`type: "cli"`):
    - `command`: исполняемый для запуска.
    - `args`: шаблонные аргументы (поддерживает `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, и т.д.).
  - `capabilities`: необязательный список (`image`, `audio`, `video`), чтобы открыть общую запись. По умолчанию при отсутствии `openai`/`anthropic`/`minimax` → изображение, `google` → image+audio+video, `groq` → аудио.
  - `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language` можно переопределить для каждой записи.

Если ни одна модель не настроена (или `включена: false`), понимание пропущено; модель все еще получает оригинальные вложения.

Аутентификация провайдера соответствует стандартному порядку (профили auth, env vars `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY`, или `models.providers.*.apiKey`).

Пример:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        макс. байт: 20971520,
        область: {
          по умолчанию: "deny",
          правил: [{ Action: "allow", match: { chatType: "direct" } }],
        },
        модели [
          { провайдер: "openai", модель: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      видео: {
        включено: true,
        maxBytes: 52428800,
        модели: [{ provider: "google", модель: "gemini-3-flash-превью" }],
      },
    },
  },
}
```

`agents.defaults.subagents` настраивает настройки по умолчанию:

- `модель`: стандартная модель для субагентов (string или `{ primary, fallbacks }`). Если субагенты унаследовали модель клиента, если только не переопределен по каждому агенту или по звоноку.
- `maxConcurrent`: максимальный одновременный запуск подагента (по умолчанию 1)
- `archiveAfterMinutes`: автоматическое архивирование сессий субагента после N минут (по умолчанию 60; установите `0` чтобы выключить)
- Политика инструментов подагентства: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (запретить выигрыш)

`tools.profile` устанавливает **базовый инструмент allowlist** перед `tools.allow`/`tools.deny`:

- `minimal`: только `session_status`
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: без ограничений (то же, что и unset)

Переопределение для агента: `agents.list[].tools.profile`.

Пример (по умолчанию только сообщения, дополнительно разрешить инструменты Slack + Discord):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

Пример (профиль для кодинга, но запрет exec/process везде):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

`tools.byProvider` позволяет **дополнительно ограничивать** инструменты для конкретных провайдеров (или один `provider/model`).
Переопределение для агента: `agents.list[].tools.byProvider`.

Заказ: базовый профиль → профиль провайдера → разрешить/запретить политики.
Ключи провайдера принимают `provider` (например, `google-antigravity`) или `provider/model`
(например, `openai/gpt-5.2`).

Пример (сохранить глобальный профиль кодинга, но минимальные инструменты для Google Antigravity):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

Пример (допустимый список/конкретный модель):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

`tools.allow` / `tools.deny` настраивает глобальный инструмент allow/deny policy (запретить победы).
Соответствие не зависит от регистра и поддерживает подстановочные знаки `*` (`"*"` означает все инструменты).
Это применимо, даже если песочница Docker **выключен**.

Пример (отключить браузер/холст везде):

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

Инструменты (короткие) работают в **глобальных** и **отдельных агентов** политиках:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: все встроенные инструменты OpenClaw (исключая плагины провайдеров)

`tools.elevated` управление повышен (host) доступ к exec серверу:

- `включено`: разрешить повышенный режим (по умолчанию)
- `allowFrom`: для каждого канала допустимые списки (пусто = отключено)
  - `whatsapp`: Е.164 цифры
  - `telegram`: идентификаторы чата или имена пользователей
  - `discord`: идентификаторы пользователя или имена пользователей (относятся к `channels.discord.dm.allowFrom`, если не указано)
  - `сигнал`: E.164 цифры
  - `imessage`: идентификаторы обработки/чата
  - `webchat`: идентификаторы сессии или имена пользователей

Пример:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        разногласия: ["steipete", "1234567890123"],
      },
    },
  },
}
```

Переопределение агента (дальнейшее ограничение):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        инструменты: {
          повышено: { enabled: false },
        },
      },
    ],
  },
}
```

Notes:

- `tools.elevated` является глобальной базой. `agents.list[].tools.elevated` может ограничивать только дальнейшее ограничение (оба должно допускать).
- `/elevated в|off|ask|full` хранит состояние ключа сессии; к одному сообщению применяются встроенные директивы.
- Повышенный уровень `exec` запускается на хосте и обходит песочницу.
- Политика инструментов все еще применяется; если "exec" отрицается, то использование повышенного эффекта невозможно.

`agents.defaults.maxConcurrent` устанавливает максимальное количество встроенного агента запускает
параллельно выполняться между сессиями. Каждая сессия по-прежнему сериализована (один раз запустить
за один ключ одновременно). По умолчанию: 1.

### `agents.defaults.sandbox`

Дополнительный **Docker sandboxing** для встроенного агента. Предназначено для неосновных сессий, чтобы они не могли получить доступ к вашей хост-системе.

Подробности: [Sandboxing](/gateway/sandboxing)

По умолчанию (если включено):

- scope: `"agent"` (один контейнер + рабочее пространство для каждого агента)
- Образ на основе букворма Debian
- агент: доступ к рабочей области: `workspaceAccess: "none"` (по умолчанию)
  - `"none"`: используйте рабочую область для области песочницы с параметром `~/.openclaw/sandboxes`
- `"ro"`: сохранить рабочую область песочницы в `/workspace` и смонтировать рабочую область агента read-only в папке `/agent` (отключает `write`/`edit`/`apply_patch`)
  - `"rw"`: монтирование рабочей области агента read/write at `/workspace`
- автоочистка: простой > 24 ч ИЛИ возраст > 7 дн.
- политика инструментов: разрешить только `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` (запретить победы)
  - configure via `tools.sandbox.tools`, override per-agent via `agents.list[].tools.sandbox.tools`
  - короткие названия групп поддерживаются в политике "песочница": `group:runtime`, `group:fs`, `group:sessions`, `group:memory` (см. [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands))
- необязательный браузер в песочнице (Chromium + CDP, noVNC наблюдатель)
- упрощение knobs: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

Предупреждение: `scope: "shared"` означает общий контейнер и общую рабочую область. Отсутствует межсессионная изоляция. Используйте `scope: "session"` для межсессионной изоляции.

Legacy: `perSession` все еще поддерживается (`true` → `scope: "session"`,
`false` → `scope: "shared"`).

`setupCommand` запускается **один раз** после создания контейнера (внутри контейнера через `sh -lc`).
Для установки пакетов убедитесь, что сетевые egress, корневой файл, доступный для записи, и пользователь root.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // выкл | неглавная | вся область
        : "агент", // сессия | агент | общий (агент по умолчанию)
        workspaceAccess: "none", // ни один | ro | rw
        workspaceRoot: "~/. penclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          Префикс контейнера: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          тмп: ["/tmp", "/var/tmp", "/run"],
          сеть: "нет",
          пользователь: "1000:1000",
          capDrop: ["ВНИМЬ"],
          env: { LANG: "C. TF-8" },
          команда setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Переопределение агента (многоагент): агенты. ist[].sandbox.docker.
          pidsLimit: 256,
          память: "1g",
          swap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp. son",
          apparmorProfile: "openclaw-sandbox",
          днс: ["1. .1.1", "8.8.8. "],
          extraHosts: ["internal.service:10.0. "],
          binds: ["/var/run/docker.sock:/var/run/docker. ock", "/home/user/source:/source:rw"],
        },
        browser: {
          включено: false,
          изображение: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          головоломка: false,
          включить NoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10. .0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0. 2"],
          allowedControlPorts: [18791],
          автозапуск: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 отключает простаивание
          maxAgeDays: 7, // 0 отключает выбрасывание max-age
        },
      },
    },
  },
  инструментов: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read", "read",
          "писать",
          "редактировать",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        отрицает: ["браузер", "холст", "узлы", "cron", "discord", "шлюз"],
      },
    },
  },
}
```

Постройте изображение песочницы по умолчанию:

```bash
scripts/sandbox-setup.sh
```

Примечание: песочница по умолчанию содержит `network: "none"`; установите `agents.defaults.sandbox.docker.network`
на `"bridge"` (или вашу пользовательскую сеть), если агенту нужен исходящий доступ.

Примечание: входящие вложения помещаются в активную рабочую среду по адресу `media/inbound/*`. При помощи \`workspaceAccess: "rw" файлы записываются в рабочую среду агента.

Примечание: `docker.binds` монтирует дополнительные директории хостов; глобальные и персонализированные binds объединены.

Постройте необязательное изображение браузера с:

```bash
scripts/sandbox-browser-setup.sh
```

Когда `agents.defaults.sandbox.browser.enabled=true`, инструмент браузера использует приложение
Chromium (CDP). Если noVNC включен (по умолчанию при headless=false),
noVNC URL вводится в системную подсказку, чтобы агент мог ссылаться на него.
Это не требует `browser.enabled` в основной конфигурации; контроль песочницы
URL инъекция в каждую сессию.

`agents.defaults.sandbox.browser.allowHostControl` (по умолчанию: false) позволяет
сэндоксированных сеансов явно ориентироваться на **хост** сервер управления браузером
с помощью инструмента браузера (`target: "host"`). Отключите это, если вы хотите жесткую изоляцию песочницы
.

Разрешенные списки для дистанционного управления:

- `allowedControlUrls`: точные контрольные URL, разрешенные для `target: "custom"`.
- `allowedControlHosts`: разрешенные имена хостов (только имя хоста, нет порта).
- `allowedControlPorts`: разрешенные порты (по умолчанию: http=80, https=443).
  По умолчанию: все разрешенные списки удалены (без ограничений). `allowHostControl` по умолчанию является false.

### `Модели` (пользовательские провайдеры + базовые URL)

OpenClaw использует каталог моделей **код-агента**. Вы можете добавить пользовательских провайдеров
(LiteLLM, локальные OpenAI-совместимые серверы, Anthropic прокси и т.д.) путём записи в `~/.openclaw/agents/<agentId>/agent/models.json` или определения той же схемы внутри конфигурации OpenClaw в разделе `models.providers`.
Обзор постороннего провайдера + примеры: [/concepts/model-providers](/concepts/model-providers).

При использовании `models.providers` OpenClaw записывает/объединяет `models.json` в
`~/.openclaw/agents/<agentId>/agent/` при запуске:

- поведение по умолчанию: **слияние** (сохранение существующих провайдеров, переопределение имени)
- установите `models.mode: "replace"` чтобы перезаписать содержимое файла

Выберите модель через `agents.defaults.model.primary` (провайдер/модель).

```json5
{
  agents: {
    defaults: {
      model: { primary: "custom-proxy/llama-3. -8b" },
      модели: {
        "custom-proxy/llama-3. -8b": {},
      },
    },
  }, Модели
  {
    режим: "Объединение",
    провайдеров: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions",
        модели: [
          {
            id: "llama-3. -8b",
            название: "Лама 3. 8B",
            причина: false,
            ввод: ["текст"],
            стоимость: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

### OpenCode Zen (мультимодельный прокси)

OpenCode Zen - это многообразный шлюз с конечными точками для моделирования. OpenClaw использует встроенного провайдера `opencode` из pi-ai; установите `OPENCODE_API_KEY` (или `OPENCODE_ZEN_API_KEY`) из [https://opencode.ai/auth](https://opencode.ai/auth).

Notes:

- Модель использует `opencode/<modelId>` (пример: `opencode/claude-opus-4-6`).
- Если вы включите allowlist через `agents.defaults.models`, добавьте каждую модель, которую вы планируете использовать.
- Shortcut: `openclaw onboard --auth-choice opencode-zen`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      модели: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

### Z.AI (GLM-4.7) — поддержка псевдонимов провайдеров

Модели Z.AI доступны со встроенного поставщика `zai`. Установите `ZAI_API_KEY` в окружении и указывайте модель в формате провайдер/модель.

Ярлык: `openclaw на борту --auth-choice zai-api-key`.

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

Notes:

- `z.ai/*` и `z-ai/*` допустимы псевдонимы и нормализуйте до `zai/*`.
- Если `ZAI_API_KEY` отсутствует, запросы к `zai/*` не будут удовлетворены при ошибке авторизации во время выполнения.
- Пример ошибки: `No API key found for provider "zai".`
- Общая конечная точка Z.AI API это `https://api.z.ai/api/paas/v4`. GLM coding
  requests use the dedicated Coding endpoint `https://api.z.ai/api/coding/paas/v4`.
  Встроенный `zai` провайдер использует конечную точку кодирования. Если вам нужна общая конечная точка
  определите пользовательского провайдера в `models.providers` с базовым URL-адресом
  (см. раздел пользовательских провайдеров выше).
- Используйте поддельный плейсхолдер в документах/конфигурациях; никогда не фиксируйте реальные API ключи.

### Moonshot AI (Kimi)

Использовать конечную точку OpenAI, совместимую с Лунным шотом:

```json5
{
  env: { MOONSHOT_API_KEY: "sk-... },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2. " },
      модели: { "moonshot/kimi-k2. ": { alias: "Kimi K2. " } },
    },
  },
  модели: {
    режим: "Объединение",
    провайдеры: {
      moonshot: {
        baseUrl: "https://api. oonshot. i/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        модели: [
          {
            id: "kimi-k2. ",
            название: "Кими К2. ",
            причина: false,
            ввод: ["текст"],
            стоимость: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Notes:

- Установите `MOONSHOT_API_KEY` в окружении или используйте `openclaw на борту --auth-choice moonshot-api-key`.
- Модель ref: `moonshot/kimi-k2.5`.
- Что касается конечной точки Китая, тоже:
  - Запустите `openclaw на доске --auth-choice moonshot-api-key-cn` (мастер установит `https://api.moonshot.cn/v1`), или
  - Вручную установите `baseUrl: "https://api.moonshot.cn/v1"` в `models.providers.moonshot`.

### Kimi Coding

Используйте конечную точку кодирования ИИ Луны в режиме Кими (антропический, встроенный провайдер):

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Notes:

- Установите `KIMI_API_KEY` в окружении или используйте `openclaw на борту --auth-choice kimi-code-api-key`.
- Модель ref: `kimi-coding/k2p5`.

### Синтез (Антропическая совместимость)

Использовать антропическую конечную точку:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Notes:

- Установите `SYNTHETIC_API_KEY` или используйте `openclaw на борту --auth-choice synthetic-api-key`.
- Модель ref: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
- Базовый URL-адрес не должен быть `/v1`, потому что Антропический клиент добавляет его.

### Локальные модели (LM Studio) — рекомендуемая установка

См. [/gateway/local-models](/gateway/local-models) для текущего локального руководства. TL;DR: запустить MiniMax M2.1 с помощью LM Studio Responses API на серьёзном оборудовании; хранить размещенные модели объединены для резерва.

### MiniMax M2.1

Использовать MiniMax M2.1 напрямую без LM Studio:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2. " },
    модели: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2. ": { alias: "Minimax" },
    },
  }, Модели
  {
    режим: "Объединение",
    провайдеры: {
      minimax: {
        baseUrl: "https://api. inimax. o/антроп",
        apiKey: "${MINIMAX_API_KEY}",
        api: "антропные-сообщения",
        модели: [
          {
            id: "MiniMax-M2. ",
            название: "MiniMax M2. ",
            причина: false,
            ввод: ["текст"],
            // Цена: обновление в моделях. сын, если вам нужно точное отслеживание затрат.
            Стоимость: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Notes:

- Установите переменную окружения `MINIMAX_API_KEY` или используйте `openclaw onboard --auth-choice minimax-api`.
- Доступная модель: `MiniMax-M2.1` (по умолчанию).
- Обновите цены в `models.json`, если вам нужна точная оценка затрат.

### Церебра (GLM 4,6/ 4,7)

Использовать Cerebras через их конечную точку, совместимую с OpenAI:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-... },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4. ",
        fallbacks: ["цереброварка/заи-блем-4. "],
      },
      models: {
        "cerebras/zai-glm-4. ": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4. (Cerebras)" },
      },
    },
  },
  модели: {
    режим: "объединение",
    провайдеров: {
      cerebras: {
        baseUrl: "https://api. erebras. i/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4. ", название: "GLM 4. (Cerebras)" },
          { id: "zai-glm-4.6", имя: "GLM 4. (Cerebras)" },
        ],
      },
    },
  },
}
```

Notes:

- Используйте `cerebras/zai-glm-4.7` для Cerebras; используйте `zai/glm-4.7` для прямой Z.AI.
- Установите `CEREBRAS_API_KEY` в окружении или конфигурации.

Notes:

- Поддерживаемые API: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- Используйте `authHeader: true` + `headers` для нужд авторизации.
- Переопределите корень конфигурации агента используя `OPENCLAW_AGENT_DIR` (или `PI_CODING_AGENT_DIR`)
  если вы хотите `models.json` в другом месте (по умолчанию: `~/.openclaw/agents/main/agent`).

### `session`

Контролирует границы сеанса, политику сброса, сброс триггеров и место записи хранилища сессий.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

Поля:

- `mainKey`: ключ прямого bucket (по умолчанию: `"main"`). Полезно, когда вы хотите «переименовать» первичный поток DM без изменения `agentId`.
  - Заметка песочницы: `agents.defaults.sandbox.mode: "non-main"` использует этот ключ для обнаружения основной сессии. Любой ключ сессии, не соответствующий `mainKey` (группы/каналы), является в песочнице.
- `dmScope`: как группы ТМ (по умолчанию: `"основная"`).
  - `главный`: все ЛС имеют основную сессию на предмет преемственности.
  - `per peer`: изолировать ЛС идентификатором отправителя по каналам.
  - `per-channel-peer`: изолировать ЛС на канал + отправителя (рекомендуется для многопользовательских почтовых ящиков).
  - `per-account-channel-peer`: изолировать ЛС на аккаунт + канал + отправителя (рекомендуется для нескольких почтовых ящиков).
  - Безопасный режим DM (рекомендуется): установите `session.dmScope: "per-channel-peer"`, когда несколько человек могут DM бота (разделяемые входящие, многопользовательские разрешают списки или \`dmPolicy: "open").
- `identityLinks`: сопоставить канонические идентификаторы с префиксами провайдера, так что тот же пользователь разделяет сеанс DM по каналам при использовании `per-peer`, `per-channel-peer`, или `per-account-channel-peer`.
  - Пример: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
- `reset`: основная политика сброса. По умолчанию ежедневно сбрасывается в 4:00 местное время на хосте шлюза.
  - `mode`: `daily` или `idle` (по умолчанию: `daily` когда `reset` присутствует).
  - `atHour`: локальный час (0-23) для ежедневной границы сброса.
  - `idleMinutes`: скользящее простое окно в минутах. Если настроены и ежедневный сброс, и бездействие, срабатывает то, что истекает раньше.
- `resetByType`: переопределения для каждой сессии для `direct`, `group` и `thread`. Устаревший ключ `dm` принимается как алиас для `direct`.
  - Если вы установите только старый `session.idleMinutes` без каких-либо `reset`/`resetByType`, OpenClaw остается в режиме ожидания для обратной совместимости.
- `heartbeatIdleMinutes`: опциональное простое переопределение для проверки heartbeat (ежедневный сброс по-прежнему применяется когда включено).
- `agentToAgent.maxPingPongTurns`: max reply-back поворачивает между запросом/целью (0–5, по умолчанию 5).
- `sendPolicy.default`: `allow` или `deny` fallback, если правило не совпадает.
- `sendPolicy.rules[]`: совпадение по `channel`, `chatType` (`direct|group|room`) или `keyPrefix` (например, `cron:`). Первое отрицание побед, в противном случае допускается.

### `skills` (настройки навыков)

Управляет встроенным allowlist, предпочтениями установки, дополнительными папками навыков и переопределениями для отдельных навыков. Применяется к **встроенным** навыкам и `~/.openclaw/skills` (навыки рабочего пространства всё равно имеют приоритет при конфликте имён).

Поля:

- `allowBundled`: необязательный список разрешённых только для **bundled** skills. При наборе только те умения, которые сочетаются с
  имеют право (навыки управляемого/рабочей области не затрагиваются).
- `load.extraDirs`: дополнительные каталоги Skills для сканирования (наименьший приоритет).
- `install.preferBrew`: предпочитать установщики brew при наличии (по умолчанию: true).
- `install.nodeManager`: настройки установки узла (`npm` | `pnpm` | `yarn`, по умолчанию: npm).
- `entries.<skillKey>`: конфигурация для каждого навыка.

Поля навыков:

- `enabled`: установите `false`, чтобы отключить Skill, даже если он встроенный/установлен.
- `env`: переменные окружения, внедряемые для запуска агента (только если ещё не заданы).
- `apiKey`: дополнительное удобство для навыков, которые объявляют первичный env вар (например, `nano-banana-pro` → `GEMINI_API_KEY`).

Пример:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Проекты/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    записи: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      мешок: { enabled: false },
    },
  },
}
```

### `plugins` (расширения)

Обнаружение плагинов управления, разрешить/запретить и для каждого плагина. Плагины загружаются
из `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, плюс любые записи
`plugins.load.paths`. **Изменения конфигурации требуют перезапуска gateway.**
См. [/plugin](/tools/plugin) для полного использования.

Поля:

- `enabled`: мастер-переключатель для загрузки плагина (по умолчанию: true).
- `allow`: необязательный список идентификаторов плагинов; если установлено, только перечисленные плагины.
- `deny`: необязательный список идентификаторов плагинов (запретить победы).
- `load.paths`: дополнительные файлы плагинов или директории для загрузки (абсолютный или `~`).
- `записей.<pluginId>`: для плагинов переопределён.
  - `включена`: установите `false` чтобы отключить эту функцию.
  - `config`: специфичный для плагина объект конфигурации (проверяется плагином при наличии).

Пример:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    загрузка: {
      пути: ["~/Projects/oss/voice-call-extension"],
    },
    записи: {
      "voice-call": {
        включен: true,
        config: {
          провайдера: "twilio",
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw может запускать **выделенный, изолированный** Chrome/Brave/Edge/Chromium экземпляр для openclaw и открыть небольшую службу управления петлями.
Профили могут указывать на **удаленный** браузер на основе Chromium через `profiles.<name>.cdpUrl`. Удаленные профили
только для прикрепленных файлов (начать/остановить/сброс отключен).

`browser.cdpUrl` остается для старых конфигураций одного профиля и в качестве базовой
схемы/хоста для профилей, которые только устанавливают `cdpPort`.

Параметры:

- включено: `true`
- evaluateEnabled: `true` (установите `false`, чтобы отключить `act:evaluate` и `wait --fn`)
- контрольный сервис: только петля (порт получается от `gateway.port`, по умолчанию `18791`)
- CDP URL: `http://127.0.0.1:18792` (контрольный сервис + 1, унаследованный однопрофиль)
- цвет профиля: `#FF4500` (lobster-orange)
- Примечание: сервер управления запускается запущенным шлюзом (OpenClaw.app menubar, или `openclaw gateway`).
- Автоматическое определение порядка: браузер по умолчанию, если базируется на Chromium-браузере; в противном случае Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    включено: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127. .0. :18792", // старое однопрофильное переопределение
    defaultProfile: "chrome",
    профилей: {
      openclaw: { cdpPort: 18800, цвет: "#FF4500" },
      work: { cdpPort: 18801, цвет: "#0066CC" },
      remote: { cdpUrl: "http://10. .0.42:9222", цвет: "#00AA00" },
    },
    цвет: "#FF4500",
    // Дополнительно:
    // безголовный: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser. pp/Contents/MacOS/Brave Browser",
    // attachOnly: false, // устанавливаем True при туннелировании удаленного CDP на localhost
  },
}
```

### `ui` (внешний вид)

Необязательный цвет акцента, используемый собственными приложениями для UI chrome (например, пузырьки режима разговор).

Если выключено, клиенты возвращаются к заглушенному светло-синему свету.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Control UI assistant identity override.
    // If unset, the Control UI uses the active agent identity (config or IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, or image URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Используйте `gateway.mode` для явного объявления о том, должна ли эта машина запускать шлюз.

Переопределения:

- режим: **unset** (рассматривается как «не автозапускать»)
- bind: `loopback`
- порт: `18789` (единственный порт для WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

Базовый путь пользовательского интерфейса:

- `gateway.controlUi.basePath` устанавливает префикс URL для интерфейса управления.
- Примеры: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- По умолчанию: root (`/`) (без изменений).
- `gateway.controlUi.root` устанавливает корень файловой системы для ресурсов интерфейса управления (по умолчанию: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` позволяет авторизоваться только для token-интерфейса управления, когда
  отсутствует личность устройства (обычно по HTTP). По умолчанию: `false`. Предпочитайте HTTPS
  (сервер в масштабе часа) или `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` отключает проверку личности устройства
  Control UI (только токен/пароль). По умолчанию: `false`. Только бутылочное стекло.

Связанная документация:

- [Интерфейс управления](/web/control-ui)
- [Обзор сайта](/web)
- [Tailscale](/gateway/tailscale)
- [Удалённый доступ](/gateway/remote)

Доверенные прокси:

- `gateway.trustedProxies`: список обратных прокси IP, которые завершают TLS перед шлюзом.
- Когда соединение исходит от одного из этих IP, OpenClaw использует `x-forwarded-for` (или `x-real-ip`) для определения IP-адреса клиента для локальной проверки сопряжений и аутентичных/локальных проверок HTTP.
- Только список прокси под вашим контролем и убедитесь что они **перезаписывают** входящий `x-forwarded-for`.

Примечания:

- `openclaw gateway` отказывается запускать, если `gateway.mode` не установлен в `local` (или вы пропустили флаг переопределения).
- `gateway.port` контролирует один мультиплексированный порт, используемый для WebSocket + HTTP (интерфейс управления, хуки, A2UI).
- OpenAI Chat Completions endpoint: **по умолчанию отключено**; включите `gateway.http.endpoints.chatCompletions.enabled: true`.
- Первое: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > по умолчанию `18789`.
- По умолчанию требуется авторизация шлюза (токен/пароль или идентификатор сервера Tailscale Serve). Для нециклических соединений требуется общий токен/пароль.
- Мастер настройки генерирует токен шлюза по умолчанию (даже на цикле).
- `gateway.remote.token` **только** для удалённых CLI звонков; это не позволяет использовать имя локального шлюза. `gateway.token` игнорируется.

Auth и хвостовая шкала:

- `gateway.auth.mode` устанавливает требования рукопожатия (`token` или `password`). При выключенном ключе предполагается авторизация токена.
- `gateway.auth.token` хранит общий токен для авторизации токена (используется CLI на той же машине).
- Когда установлен `gateway.auth.mode`, используется только этот метод (плюс дополнительные заголовки Tailscal).
- Здесь можно установить `gateway.auth.password` или через `OPENCLAW_GATEWAY_PASSWORD` (рекомендуется).
- `gateway.auth.allowTailscale` позволяет запрашивать идентификационные заголовки сервера Tailscale
  (`tailscale-user-login`) для удовлетворения авторизации по прибытии запроса на loopback
  с помощью `x-forwarded-for`, `x-forwarded-proto` и `x-forwarded-host`. OpenClaw
  проверяет личность, разрешив адрес `x-forwarded-for` через
  `tailscale whois` перед принятием. Когда `true`, серверу не нужен
  токен/пароль; установите `false`, чтобы требовать явных учетных данных. По умолчанию
  `true` при `tailscale.mode = "serve"` и auth режим не `password`.
- `gateway.tailscale.mode: "serve"` использует сервер Tailscale (только tailnet, loopback bind).
- `gateway.tailscale.mode: "funnel"` публично раскрывает приборную панель; требует автора.
- `gateway.tailscale.resetOnExit` сбрасывает настройки сервера и функционала при выключении.

Настройки по умолчанию удаленного клиента (CLI):

- `gateway.remote.url` устанавливает по умолчанию URL шлюза WebSocket для CLI вызовов, когда `gateway.mode = "remote"`.
- `gateway.remote.transport` выбирает удаленный транспорт macOS (по умолчанию `ssh` `direct` для ws/wss). Когда `direct`, `gateway.remote.url` должен быть `ws://` или `wss://`. `ws://host` defaults to port `18789`.
- `gateway.remote.token` снабжает токен для удалённых вызовов (оставьте не задан для автора).
- `gateway.remote.password` передает пароль для удалённых вызовов (оставьте пустым для неавторизации).

Поведение приложения macOS:

- OpenClaw.app отслеживает `~/.openclaw/openclaw.json` и переключает режимы в реальном времени при изменении `gateway.mode` или `gateway.remote.url`.
- Если `gateway.mode` отключен, но `gateway.remote.url` установлен, macOS приложение считает его удаленным режимом.
- При смене режима подключения в macOS приложение записывает `gateway.mode` (и `gateway.remote.url` + `gateway.remote.transport` в удаленном режиме) обратно в файл конфигурации.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Пример прямого переноса (macOS app):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (настройка горячей перезагрузки)

Шлюз следит за `~/.openclaw/openclaw.json` (или `OPENCLAW_CONFIG_PATH`) и автоматически применяет изменения.

Режимы:

- `hybrid` (по умолчанию): горячие безопасные изменения; перезапустите шлюз для критических изменений.
- `hot`: применять только горячие изменения; журнал, когда требуется перезапуск.
- `restart`: перезапустите шлюз при любом изменении конфигурации.
- `off`: отключить горячую перезагрузку.

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

#### Обновлять матрицу (файлы + удары)

Просмотренные файлы:

- `~/.openclaw/openclaw.json` (или `OPENCLAW_CONFIG_PATH`)

Горячее приложение (перезапуск шлюза не полностью):

- `hooks` (webhook auth/path/mappings) + `hooks.gmail` (Gmail watcher restarted)
- `browser` (перезапуск сервера управления браузером)
- `cron` (cron сервис перезапускает + обновление в concurrence)
- `agents.defaults.heartbeat` (runner heartbeat runner)
- `web` (перезапуск веб-канала WhatsApp)
- `телеграмма, `discord`, `signal`, `imessage\` (перезапуск канала)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (динамические чтения)

Требуется перезапуск шлюза:

- `gateway` (port/bind/auth/control UI/tailscale)
- «мост» (обычный)
- `discovery`
- `canvasHost`
- `плагины`
- Любой неизвестный/неподдерживаемый путь конфигурации (по умолчанию для перезапуска в целях безопасности)

### Изоляция нескольких экземпляров

Для запуска нескольких шлюзов на одном хосте (для избыточности или спасательного бота), изолируйте каждое состояние + конфигурацию и используйте уникальные порты:

- `OPENCLAW_CONFIG_PATH` (конфигурация для каждого экземпляра)
- `OPENCLAW_STATE_DIR` (сессии/кредиты)
- `agents.defaults.workspace` (memories)
- `gateway.port` (уникальное за один экземпляр)

Флаги удобства (CLI):

- `openclaw --dev …` → использует `~/.openclaw-dev` + сдвигает порты из базы `19001`
- `openclaw --profile <name> …` → использует `~/.openclaw-<name>` (порт через config/env/flags)

См. [Gateway runbook](/gateway) описание производимых портов (gateway/browser/canvas).
Смотрите [Несколько шлюзов](/gateway/multiple-gateways) информацию об изоляции браузера/CDP портов.

Пример:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
шлюз openclaw --port 19001
```

### `hooks` (Gateway webhooks)

Включить простой HTTP-веб-хук на HTTP-сервере Шлюза.

Вложенные включения

- включено: `false`
- путь: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
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

Запросы должны включать токен хука:

- \`Авторизация: носитель <token>**или**
- `x-openclaw-token: <token>`

Конечные точки:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, для?, модели?, думаю?, timeoutSeconds? }`
- `POST /hooks/<name>` → разрешено через `hooks.mappings`

`/hooks/agent` всегда публикует резюме в основной сессии (и может при необходимости вызвать немедленное прослушивание через `wakeMode: "now"`).

Сопоставление заметок:

- `match.path` соответствует подпути после `/hooks` (например `/hooks/gmail` → `gmail`).
- `match.source` соответствует полю payload (например `{ source: "gmail" }`), так что вы можете использовать общий путь `/hooks/ingest`.
- Такие шаблоны, как `{{messages[0].subject}}`, читается из полезной нагрузки.
- `transform` может указывать на модуль JS/TS, который возвращает действие хука.
- `deliver: true` отправляет финальный ответ в канал; `channel` по умолчанию передается в `last` (возвращается в WhatsApp).
- Если раньше маршрута доставки нет, установите `channel` + `to` явно (требуется для Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS команд).
- `model` переопределяет LLM для этого хука (`provider/model` или алиаса; если установлен файл `agents.defaults.models`).

Конфигурация Gmail помощника (используется `openclaw webhooks gmail setup` / `run`):

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail. om",
      тема: "projects/<project-id>/topics/gog-gmail-watch",
      подписка: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127. .0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      Обновление EveryMinutes: 720,
      serve: { bind: "127. .0. ", port: 8788, путь: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // Необязательно: используйте более дешевую модель для обработки хука Gmail
      // Отзывами обратно к агентам. efaults.model. allbacks, затем основной, по auth/rate-limit/timeout
      модель: "openrouter/meta-llama/llama-3. -70b-instruct:free",
      // Необязательно: уровень мышления по умолчанию для Gmail хуков
      думает: "off",
    },
  },
}
```

Переопределение модели для хуков Gmail:

- `hooks.gmail.model` определяет модель для использования при обработке Gmail хука (по умолчанию - основная сессия).
- Принимает `provider/model` refs or aliases from `agents.defaults.models`.
- Возвращает `agents.defaults.model.fallbacks`, затем `agents.defaults.model.primary`, on auth/rate-limit/timeouts.
- Если установлен `agents.defaults.models`, включите в список подходящую модель хуков.
- При запуске предупреждает, нет ли настроенная модель в каталоге или списке моделей.
- `hooks.gmail.thinking` устанавливает стандартный уровень мышления для хуков Gmail и переопределен на каждый хук `think `.

Автозапуск шлюза:

- Если у вас установлен `hooks.enabled=true` и `hooks.gmail.account`, шлюз запускается
  `gog gmail watch serve` при загрузке и автоперезапуск часа.
- Установите `OPENCLAW_SKIP_GMAIL_WATCHER=1`, чтобы отключить автозапуск (для ручных запуска).
- Избегайте выполнения отдельного `gog gmail watch serve` вместе с шлюзом; он будет
  не удаётся с `listen tcp 127.0.0.1:8788: bind: уже используется`.

Примечание: когда `tailscale.mode` включен, OpenClaw по умолчанию использует `serve.path` в `/`, так что
Tailscale может правильно проксировать `/gmail-pubsub` (он разделяет префикс set-path).
Если вам нужен бэкэнд, чтобы получить префиксный путь, установите
`hooks.gmail.tailscale.target` на полный URL (и выравнивайте `serve.path`).

### `canvasHost` (LAN/tailnet Canvas file server + live reload)

Шлюз служит каталогу HTML/CSS/JS через HTTP, чтобы узлы iOS/Android могли просто `canvas.navigate` к нему.

Корень по умолчанию: `~/. penclaw/workspace/canvas`  
Порт по умолчанию: `18793` (выбран, чтобы избежать порта CDP браузера openclaw `18792`)  
Сервер слушает **шлюз** (LAN или Tailnet), так что узлы могут достичь его.

Сервер:

- обслуживает файлы в каталоге `canvasHost.root`
- вводит маленький live-reload клиент в обслуживаемый HTML
- следит за каталогом и транслирует перезагрузку через конечную точку WebSocket в `/__openclaw__/ws`
- автоматически создает стартовый файл `index.html`, когда каталог пуст (так что вы видите что-то немедленно)
- также служит A2UI в `/__openclaw__/a2ui/` и рекламируется в узлы как `canvasHostUrl`
  (всегда используется узлами для Canvas/A2UI)

Отключить прямую перезагрузку (и просмотр файлов), если каталог большой или вы нажмете `EMFILE`:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

Изменения в `canvasHost.*` требуют перезапуска шлюза (перезагрузка конфигурации перезагрузится).

Отключить:

- config: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `мост` (устаревший TCP мост, удален)

Текущие сборки больше не включают listener; ключи конфигурации `bridge.*` игнорируются.
Узлы подключаются через WebSocket. Этот раздел хранится для исторической ссылки.

Наследие поведения:

- Шлюз может выдать простой TCP мост для узлов (iOS/Android), как правило, на порту `18790`.

Параметры:

- включено: `true`
- порт: `18790`
- bind: `lan` (binds to `0.0.0.0`)

Bind modes:

- `lan`: `0.0.0.0` (доступно на любом интерфейсе, включая локальную сеть, Wi-Fi и Tailscale)
- «хвостовая сеть»: привязывается только к IP-адресу Tailscale (рекомендуется для Вены в Лондоне)
- `loopback`: `127.0.0.1` (только локально)
- `auto`: предпочтительно использовать хвост IP если присутствует, иначе `lan`

TLS:

- `bridge.tls.enabled`: включить TLS для мостовых соединений (TLS-только, когда включено).
- `bridge.tls.autoGenerate`: генерировать самоподписанный сертификат, когда cert/key нет (по умолчанию: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: пути к PEM для мостового сертификата + приватный ключ.
- `bridge.tls.caPath`: опциональный PEM CA пакет (пользовательские корни или будущие mTLS).

Когда TLS включен, шлюз рекламирует `bridgeTls=1` и `bridgeTlsSha256` в открытии TXT
записей, чтобы узлы могли прикрепить сертификат. Ручные соединения используют "trust-on-first-us", если отпечаток пальца
пока не сохранен.
Сгенерированные сертификаты требуют `openssl` на PATH; если генерация не удается, мост не запустится.

```json5
{
  моста: {
    включен: true,
    Порт: 18790,
    привязка: "хвостовая сеть",
    tls: {
      включен: true,
      // Использует ~/. penclaw/bridge/tls/bridge-{cert,key}. em when omitted.
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/. penclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (режим трансляции Bonjour / mDNS)

Управляет обнаружением mDNS трансляций (`_openclaw-gw._tcp`).

- `minimal` (по умолчанию): omit `cliPath` + `sshPort` из TXT записей
- `full`: включить `cliPath` + `sshPort` в TXT записи
- `выкл`: полностью отключить трансляции mDNS
- Имя хоста: по умолчанию `openclaw` (рекламирует `openclaw.local`). Переопределить с помощью `OPENCLAW_MDNS_HOSTNAME`.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (Wide-Area Bonjour / unicast DNS-SD)

Если эта опция включена, шлюз записывает в файл `_openclaw-gw._tcp` unicast DNS-SD zone для `_openclaw-gw._tcp` в папке `~/.openclaw/dns/` используя настроенный домен обнаружения (пример: `openclaw.internal.`).

Чтобы сделать iOS/Android обнаружить через сети (Vienna &lt;unk&gt; London), свяжите это с:

- DNS сервер на шлюзе хоста, обслуживающего выбранный домен (CoreDNS рекомендуется)
- Tailscale **split DNS** для того, чтобы клиенты могли разрешить домен через шлюз DNS сервер

Единоразовый помощник (хост шлюза):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## Переменные шаблона модели медиафайлов

Заполнители шаблонов расширены в `tools.media.*.models[].args` и `tools.media.models[].args` (и любых будущих шаблонных полей аргументов).

\| Переменная | Описание |
\| ----------------------------------------------------------------------------------------------- | ------| ------- | ---------- | ----- | ------ | ------- | ------- | ------- | --- | --- |
\| `{{Body}}` | Полное входящее тело |
\| `{{RawBody}}` | Raw входящее тело сообщения (нет оболочек истории/отправителя; лучшее для команд parsing) |
\| `{{BodyStripped}}` | Тело с упоминаниями в группах полоскано (лучшее по умолчанию для агентов) |
\| `{{From}}` | Отправитель идентификатор (E. 64 для WhatsApp; может отличаться по каналу) |
\| `{{To}}` | Идентификатор назначения |
\| `{{MessageSid}}| Идентификатор сообщения (когда доступно) | | `{{SessionId}}`| Текущая сессия UUID |
|`{{IsNewSession}}`|`"true"`при создании новой сессии |
|`{{MediaUrl}}`| Inbound media pseudo-URL (если имеется) |
|`{{MediaPath}}`| Путь к локальному носителю (если загружено) |
|`{{MediaType}}`| Тип медиа (image/audio/document/…)                                             |
|`{{Transcript}}`  | Аудиотранскрипт (когда включено)                                                 |
|`{{Prompt}}`      | Разрешённый медиапромпт для CLI-записей                                          |
|`{{MaxChars}}`    | Разрешённый максимум символов вывода для CLI-записей                            |
|`{{ChatType}}`    |`"direct"`или`"group"`                                                    |
|`{{GroupSubject}}`| Тема группы (по возможности)                                                     |
|`{{GroupMembers}}`| Предпросмотр участников группы (по возможности)                                 |
|`{{SenderName}}`  | Отображаемое имя отправителя (по возможности)                                   |
|`{{SenderE164}}`  | Телефонный номер отправителя (по возможности)                                   |
|`{{Provider}}\`     | Подсказка провайдера (whatsapp | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …)  |

## Cron (планировщик ворот)

Cron - это собственный планировщик шлюзов для пробуждения и запланированных заданий. Смотрите [Cron jobs](/automation/cron-jobs) обзор возможностей и примеры CLI.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_Далее: [Agent Runtime](/concepts/agent)_ 🦞
