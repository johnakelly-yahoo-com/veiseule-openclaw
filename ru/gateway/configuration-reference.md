---
title: "Справочник по конфигурации"
description: "Полное построчное описание полей для ~/.openclaw/openclaw.json"
---

# Справочник по конфигурации

Все доступные поля в `~/.openclaw/openclaw.json`. Обзор, ориентированный на задачи, см. в разделе [Configuration](/gateway/configuration).

Формат конфигурации — **JSON5** (разрешены комментарии и завершающие запятые). Все поля необязательны — при их отсутствии OpenClaw использует безопасные значения по умолчанию.

---

## Каналы

Каждый канал запускается автоматически при наличии соответствующего раздела конфигурации (если не указано `enabled: false`).

### Доступ к личным сообщениям и группам

Все каналы поддерживают политики для личных сообщений (DM) и групп:

| Политика DM                                 | Поведение                                                                                            |
| ------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `pairing` (по умолчанию) | Неизвестные отправители получают одноразовый код сопряжения; владелец должен подтвердить             |
| `allowlist`                                 | Только отправители из `allowFrom` (или из хранилища разрешённых после сопряжения) |
| `open`                                      | Разрешить все входящие личные сообщения (требуется `allowFrom: ["*"]`)            |
| `disabled`                                  | Игнорировать все входящие личные сообщения                                                           |

| Политика групп                                | Поведение                                                                                                    |
| --------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| `allowlist` (по умолчанию) | Только группы, соответствующие настроенному списку разрешённых                                               |
| `open`                                        | Игнорировать списки разрешённых групп (ограничение по упоминанию по‑прежнему применяется) |
| `disabled`                                    | Блокировать все сообщения в группах/комнатах                                                                 |

<Note>
`channels.defaults.groupPolicy` задаёт значение по умолчанию, если `groupPolicy` у провайдера не установлен.
Срок действия кодов сопряжения истекает через 1 час. Количество ожидающих запросов на сопряжение в DM ограничено **3 на канал**.
Для Slack/Discord действует специальный резервный механизм: если раздел их провайдера полностью отсутствует, политика групп во время выполнения может быть установлена в `open` (с предупреждением при запуске).
</Note>

### WhatsApp

WhatsApp работает через веб-канал шлюза (Baileys Web). Он запускается автоматически при наличии связанной сессии.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // синие галочки (false в режиме чата с самим собой)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Исходящие команды по умолчанию используют аккаунт `default`, если он присутствует; иначе — первый настроенный идентификатор аккаунта (в отсортированном порядке).
- Каталог авторизации устаревшего одноаккаунтного Baileys переносится с помощью `openclaw doctor` в `whatsapp/default`.
- Переопределения для отдельных аккаунтов: `channels.whatsapp.accounts.<id>`.sendReadReceipts`, `channels.whatsapp.accounts.<id>`.dmPolicy`, `channels.whatsapp.accounts.<id>.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Держите ответы краткими.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Придерживайтесь темы.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Резервное копирование Git" },
        { command: "generate", description: "Создать изображение" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Токен бота: `channels.telegram.botToken` или `channels.telegram.tokenFile`, с `TELEGRAM_BOT_TOKEN` в качестве резервного варианта для аккаунта по умолчанию.
- `configWrites: false` блокирует изменения конфигурации, инициированные из Telegram (миграции ID супергрупп, `/config set|unset`).
- Предпросмотр потоковой передачи в Telegram использует `sendMessage` + `editMessageText` (работает в личных и групповых чатах).
- Политика повторных попыток: см. [Retry policy](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
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
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Только короткие ответы.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- Токен: `channels.discord.token`, с `DISCORD_BOT_TOKEN` в качестве резервного варианта для аккаунта по умолчанию.
- Используйте `user:<id>` (DM) или `channel:<id>` (канал guild) в качестве целей доставки; одиночные числовые ID отклоняются.
- Slug'и guild — в нижнем регистре с заменой пробелов на `-`; ключи каналов используют имя в формате slug (без `#`). Предпочитайте использовать ID guild.
- Сообщения, отправленные ботом, по умолчанию игнорируются. `allowBots: true` включает их (собственные сообщения по-прежнему фильтруются).
- `maxLinesPerMessage` (по умолчанию 17) разбивает длинные сообщения, даже если они меньше 2000 символов.
- `channels.discord.ui.components.accentColor` задаёт акцентный цвет для контейнеров компонентов Discord v2.

**Режимы уведомлений о реакциях:** `off` (нет), `own` (сообщения бота, по умолчанию), `all` (все сообщения), `allowlist` (из `guilds.<id>`.users\` для всех сообщений).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- JSON сервисного аккаунта: встроенный (`serviceAccount`) или из файла (`serviceAccountFile`).
- Резервные переменные окружения: `GOOGLE_CHAT_SERVICE_ACCOUNT` или `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Используйте `spaces/<spaceId>` или `users/<userId|email>` в качестве целей доставки.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Только короткие ответы.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Socket mode** требует оба токена: `botToken` и `appToken` (`SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` как резерв для переменных окружения аккаунта по умолчанию).
- **HTTP mode** требует `botToken` плюс `signingSecret` (в корне или для конкретного аккаунта).
- `configWrites: false` блокирует изменения конфигурации, инициированные из Slack.
- Используйте `user:<id>` (DM) или `channel:<id>` в качестве целей доставки.

**Режимы уведомлений о реакциях:** `off`, `own` (по умолчанию), `all`, `allowlist` (из `reactionAllowlist`).

**Изоляция сессий в потоках:** `thread.historyScope` — отдельно для каждого потока (по умолчанию) или общий для всего канала. `thread.inheritParent` копирует историю родительского канала в новые потоки.

| Группа действий         | По умолчанию | Примечания                                 |
| ----------------------- | ------------ | ------------------------------------------ |
| reactions               | включено     | Реагировать + просматривать список реакций |
| сообщения               | включено     | Чтение/отправка/редактирование/удаление    |
| закреплённые сообщения  | включено     | Закрепить/открепить/список                 |
| информация об участнике | включено     | Информация об участнике                    |
| список эмодзи           | включено     | Список пользовательских эмодзи             |

### Mattermost

Mattermost поставляется как плагин: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Режимы чата: `oncall` (ответ при @-упоминании, по умолчанию), `onmessage` (каждое сообщение), `onchar` (сообщения, начинающиеся с префикса-триггера).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Режимы уведомлений о реакциях:** `off`, `own` (по умолчанию), `all`, `allowlist` (из `reactionAllowlist`).

### iMessage

OpenClaw запускает `imsg rpc` (JSON-RPC через stdio). Демон или порт не требуются.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Требуется полный доступ к диску для базы данных Messages.
- Предпочитайте цели вида `chat_id:<id>`. Используйте `imsg chats --limit 20`, чтобы получить список чатов.
- `cliPath` может указывать на SSH-обёртку; укажите `remoteHost` для получения вложений через SCP.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Мультиаккаунт (все каналы)

Запуск нескольких аккаунтов для каждого канала (каждый со своим `accountId`):

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

- `default` используется, когда `accountId` не указан (CLI + маршрутизация).
- Токены окружения применяются только к аккаунту **default**.
- Базовые настройки канала применяются ко всем аккаунтам, если не переопределены для конкретного аккаунта.
- Используйте `bindings[].match.accountId`, чтобы направить каждый аккаунт к разному агенту.

### Ограничение по упоминаниям в групповых чатах

Сообщения в группах по умолчанию требуют **обязательного упоминания** (метаданные упоминаний или шаблоны regex). Применяется к групповым чатам WhatsApp, Telegram, Discord, Google Chat и iMessage.

**Типы упоминаний:**

- **Упоминания в метаданных**: Нативные @-упоминания платформы. Игнорируется в режиме самочата WhatsApp.
- **Текстовые шаблоны**: Regex-шаблоны в `agents.list[].groupChat.mentionPatterns`. Проверяется всегда.
- Проверка упоминаний применяется только когда обнаружение возможно (нативные упоминания или хотя бы один шаблон).

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

`messages.groupChat.historyLimit` задаёт глобальное значение по умолчанию. Каналы могут переопределять с помощью `channels.<channel> .historyLimit` (или для конкретной учётной записи). Установите `0`, чтобы отключить.

#### Лимиты истории личных сообщений (DM)

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Порядок применения: переопределение для конкретного DM → значение по умолчанию провайдера → без лимита (сохраняется всё).

Поддерживаются: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Режим самочата

Добавьте свой номер в `allowFrom`, чтобы включить режим самочата (игнорирует нативные @-упоминания, отвечает только на текстовые шаблоны):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Команды (обработка команд чата)

```json5
{
  commands: {
    native: "auto", // register native commands when supported
    text: true, // parse /commands in chat messages
    bash: false, // allow ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // allow /config
    debug: false, // allow /debug
    restart: false, // allow /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Текстовые команды должны быть **отдельными** сообщениями, начинающимися с `/`.
- `native: "auto"` включает нативные команды для Discord/Telegram, оставляя их выключенными для Slack.
- Переопределение для канала: `channels.discord.commands.native` (bool или `"auto"`). `false` удаляет ранее зарегистрированные команды.
- `channels.telegram.customCommands` добавляет дополнительные пункты меню бота Telegram.
- `bash: true` включает `! <cmd>` для оболочки хоста. Требуется `tools.elevated.enabled`, а отправитель должен быть указан в `tools.elevated.allowFrom.<channel>`.
- `config: true` включает `/config` (чтение/запись `openclaw.json`).
- `channels.<provider> .configWrites` управляет изменениями конфигурации для каждого канала (по умолчанию: true).
- `allowFrom` задаётся для каждого провайдера отдельно. Если параметр установлен, он является **единственным** источником авторизации (списки разрешённых каналов/сопряжения и `useAccessGroups` игнорируются).
- `useAccessGroups: false` позволяет командам обходить политики групп доступа, когда `allowFrom` не задан.

</Accordion>

---

## Значения по умолчанию для агента

### `agents.defaults.workspace`

По умолчанию: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Необязательный корневой каталог репозитория, отображаемый в строке Runtime системного запроса. Если не задано, OpenClaw автоматически определяет его, поднимаясь вверх от рабочей директории.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Отключает автоматическое создание файлов инициализации рабочего пространства (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`).

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Максимальное количество символов в одном файле инициализации рабочего пространства до обрезки. По умолчанию: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Максимальное общее количество символов, вставляемых во все файлы инициализации рабочего пространства. По умолчанию: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Часовой пояс для контекста системного запроса (не для временных меток сообщений). Если не указано, используется часовой пояс хоста.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Формат времени в системном запросе. По умолчанию: `auto` (настройка ОС).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: формат `provider/model` (например, `anthropic/claude-opus-4-6`). Если не указать provider, OpenClaw предполагает `anthropic` (устаревшее поведение).
- `models`: настроенный каталог моделей и список разрешённых моделей для `/model`. Каждая запись может включать `alias` (сокращение) и `params` (специфичные для provider: `temperature`, `maxTokens`).
- `imageModel`: используется только если основная модель не поддерживает ввод изображений.
- `maxConcurrent`: максимальное количество параллельных запусков агента во всех сессиях (каждая сессия по‑прежнему выполняется последовательно). По умолчанию: 1.

**Встроенные сокращённые алиасы** (применяются только если модель указана в `agents.defaults.models`):

| Алиас          | Модель                          |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Настроенные вами алиасы всегда имеют приоритет над значениями по умолчанию.

Модели Z.AI GLM-4.x автоматически включают режим thinking, если вы не укажете `--thinking off` или не зададите `agents.defaults.models["zai/<model>"].params.thinking` самостоятельно.

### `agents.defaults.cliBackends`

Необязательные CLI-бэкенды для резервных запусков только с текстом (без вызовов инструментов). Полезно в качестве запасного варианта при сбоях у API-провайдеров.

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

- CLI-бэкенды ориентированы только на текст; инструменты всегда отключены.
- Сессии поддерживаются, если задан `sessionArg`.
- Передача изображений поддерживается, если `imageArg` принимает пути к файлам.

### `agents.defaults.heartbeat`

Периодические heartbeat-запуски.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: строка длительности (ms/s/m/h). По умолчанию: `30m`.
- Для конкретного агента: задайте `agents.list[].heartbeat`. Если любой агент определяет `heartbeat`, **только эти агенты** выполняют heartbeat-запуски.
- Heartbeat выполняют полноценные ходы агента — более короткие интервалы расходуют больше токенов.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` или `safeguard` (сегментированная суммаризация для длинной истории). См. [Compaction](/concepts/compaction).
- `memoryFlush`: тихий агентный ход перед авто-компакцией для сохранения долговременных данных. Пропускается, если рабочее пространство доступно только для чтения.

### `agents.defaults.contextPruning`

Удаляет **старые результаты инструментов** из контекста в памяти перед отправкой в LLM. **Не** изменяет историю сессии на диске.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` включает проходы очистки.
- `ttl` определяет, как часто очистка может выполняться снова (после последнего обращения к кэшу).
- Очистка сначала мягко сокращает слишком большие результаты инструментов, затем при необходимости полностью очищает более старые результаты.

**Soft-trim** сохраняет начало и конец, вставляя `...` в середине.

**Hard-clear** заменяет весь результат инструмента на заглушку.

Примечания:

- Блоки изображений никогда не сокращаются и не очищаются.
- Соотношения рассчитываются по количеству символов (приблизительно), а не по точному числу токенов.
- Если сообщений ассистента меньше, чем `keepLastAssistants`, очистка не выполняется.

</Accordion>

См. [Session Pruning](/concepts/session-pruning) для подробностей поведения.

### Блочная потоковая передача

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- Для каналов, отличных от Telegram, требуется явно указать `*.blockStreaming: true`, чтобы включить блочные ответы.
- Переопределения для каналов: `channels.<channel>`.blockStreamingCoalesce`(а также варианты для отдельных аккаунтов). Для Signal/Slack/Discord/Google Chat по умолчанию`minChars: 1500\`.
- `humanDelay`: случайная пауза между блочными ответами. `natural` = 800–2500 мс. Переопределение для конкретного агента: `agents.list[].humanDelay`.

См. [Streaming](/concepts/streaming) для подробностей о поведении и разбиении на части.

### Индикаторы набора текста

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- По умолчанию: `instant` для личных чатов/упоминаний, `message` для групповых чатов без упоминаний.
- Переопределения для сессии: `session.typingMode`, `session.typingIntervalSeconds`.

См. [Typing Indicators](/concepts/typing-indicators).

### `agents.defaults.sandbox`

Необязательная **Docker-изоляция (sandboxing)** для встроенного агента. См. [Sandboxing](/gateway/sandboxing) для полного руководства.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**Доступ к рабочему пространству:**

- `none`: рабочее пространство sandbox в пределах области видимости в `~/.openclaw/sandboxes`
- `ro`: рабочее пространство sandbox в `/workspace`, рабочее пространство агента подключено только для чтения в `/agent`
- `rw`: рабочее пространство агента подключено для чтения и записи в `/workspace`

**Область видимости:**

- `session`: отдельный контейнер и рабочее пространство для каждой сессии
- `agent`: один контейнер и рабочее пространство на агента (по умолчанию)
- `shared`: общий контейнер и рабочее пространство (без изоляции между сессиями)

**`setupCommand`** выполняется один раз после создания контейнера (через `sh -lc`). Требуется исходящее сетевое соединение, доступная для записи корневая файловая система и пользователь root.

**По умолчанию для контейнеров установлено `network: "none"`** — установите `"bridge"`, если агенту нужен исходящий доступ.

**Входящие вложения** помещаются в `media/inbound/*` в активном рабочем пространстве.

**`docker.binds`** подключает дополнительные каталоги хоста; глобальные и для отдельных агентов объединяются.

**Изолированный браузер** (`sandbox.browser.enabled`): Chromium + CDP в контейнере. URL noVNC добавляется в системный prompt. Не требует `browser.enabled` в основной конфигурации.

- `allowHostControl: false` (по умолчанию) запрещает изолированным сессиям обращаться к браузеру хоста.
- `sandbox.browser.binds` подключает дополнительные каталоги хоста только в контейнер изолированного браузера. При указании (включая `[]`) заменяет `docker.binds` для контейнера браузера.

</Accordion>

Сборка образов:

```bash
scripts/sandbox-setup.sh           # основной образ песочницы
scripts/sandbox-browser-setup.sh   # дополнительный образ с браузером
```

### `agents.list` (переопределения для отдельных агентов)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // или { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: стабильный идентификатор агента (обязательно).
- `default`: если указано несколько, используется первый (записывается предупреждение). Если ни один не указан, по умолчанию используется первый элемент списка.
- `model`: строковая форма переопределяет только `primary`; объектная форма `{ primary, fallbacks }` переопределяет оба (`[]` отключает глобальные fallbacks).
- `identity.avatar`: путь относительно workspace, `http(s)` URL или `data:` URI.
- `identity` наследует значения по умолчанию: `ackReaction` из `emoji`, `mentionPatterns` из `name`/`emoji`.
- `subagents.allowAgents`: список разрешённых id агентов для `sessions_spawn` (`["*"]` = любой; по умолчанию: только тот же агент).

---

## Маршрутизация нескольких агентов

Запуск нескольких изолированных агентов внутри одного Gateway. См. [Multi-Agent](/concepts/multi-agent).

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

### Поля сопоставления binding

- `match.channel` (обязательно)
- `match.accountId` (необязательно; `*` = любой аккаунт; если опущено = аккаунт по умолчанию)
- `match.peer` (необязательно; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (необязательно; зависит от канала)

**Детерминированный порядок сопоставления:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (точное совпадение, без peer/guild/team)
5. `match.accountId: "*"` (на весь канал)
6. Агент по умолчанию

В пределах каждого уровня используется первый подходящий элемент `bindings`.

### Профили доступа для отдельных агентов

<Accordion title="Full access (no sandbox)">

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

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
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

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
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

</Accordion>

Подробности о приоритете см. в [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools).

---

## Сессия

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // устаревшее (runtime всегда использует "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: как группируются личные сообщения (DM).
  - `main`: все DM используют общую основную сессию.
  - `per-peer`: изоляция по id отправителя между каналами.
  - `per-channel-peer`: изоляция по каналу + отправителю (рекомендуется для общих входящих).
  - `per-account-channel-peer`: изоляция по аккаунту + каналу + отправителю (рекомендуется для нескольких аккаунтов).
- **`identityLinks`**: сопоставляет канонические id с peer, имеющими префикс провайдера, для совместного использования сессий между каналами.
- **`reset`**: основная политика сброса. `daily` выполняет сброс в `atHour` по местному времени; `idle` выполняет сброс после `idleMinutes`. Если настроены оба варианта, применяется тот, который срабатывает раньше.
- **`resetByType`**: переопределения по типам (`direct`, `group`, `thread`). Устаревший `dm` принимается как алиас для `direct`.
- **`mainKey`**: устаревшее поле. Во время выполнения теперь всегда используется `"main"` для основного бакета прямого чата.
- **`sendPolicy`**: сопоставление по `channel`, `chatType` (`direct|group|channel`, с устаревшим алиасом `dm`), `keyPrefix` или `rawKeyPrefix`. Первый запрет имеет приоритет.
- **`maintenance`**: `warn` предупреждает активную сессию при вытеснении; `enforce` применяет очистку и ротацию.

</Accordion>

---

## Сообщения

```json5
{
  messages: {
    responsePrefix: "🦞", // или "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 отключает
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Префикс ответа

Переопределения для канала/аккаунта: `channels.<channel> .responsePrefix`, `channels.<channel> .accounts.<id> .responsePrefix`.

Порядок разрешения (более специфичное имеет приоритет): аккаунт → канал → глобальный уровень. `""` отключает и останавливает каскадирование. `"auto"` формирует `[{identity.name}]`.

**Переменные шаблона:**

| Переменная        | Описание                    | Пример                                     |
| ----------------- | --------------------------- | ------------------------------------------ |
| `{model}`         | Краткое имя модели          | `claude-opus-4-6`                          |
| `{modelFull}`     | Полный идентификатор модели | `anthropic/claude-opus-4-6`                |
| `{provider}`      | Имя провайдера              | `anthropic`                                |
| `{thinkingLevel}` | Текущий уровень рассуждений | `high`, `low`, `off`                       |
| `{identity.name}` | Имя идентичности агента     | (то же, что и `"auto"`) |

Переменные нечувствительны к регистру. `{think}` — это псевдоним для `{thinkingLevel}`.

### Реакция подтверждения

- По умолчанию используется `identity.emoji` активного агента, иначе — "👀". Установите `""`, чтобы отключить.
- Переопределения для канала: `channels.<channel>
  .ackReaction`, `channels.<channel>
  .accounts.<id>
  .ackReaction`.
- Порядок разрешения: аккаунт → канал → `messages.ackReaction` → резервное значение identity.
- Область действия: `group-mentions` (по умолчанию), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: удаляет реакцию подтверждения после ответа (только Slack/Discord/Telegram/Google Chat).

### Входящий debounce

Объединяет быстро отправленные текстовые сообщения от одного отправителя в один ответ агента. Медиа/вложения отправляются немедленно. Управляющие команды обходят debounce.

### TTS (text-to-speech)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
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

- `auto` управляет автоматическим TTS. `/tts off|always|inbound|tagged` переопределяет настройку для текущей сессии.
- `summaryModel` переопределяет `agents.defaults.model.primary` для автосводки.
- API-ключи берутся из `ELEVENLABS_API_KEY`/`XI_API_KEY` и `OPENAI_API_KEY`, если не указаны явно.

---

## Talk

Настройки по умолчанию для режима Talk (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID берутся из `ELEVENLABS_VOICE_ID` или `SAG_VOICE_ID`, если не указаны явно.
- `apiKey` берётся из `ELEVENLABS_API_KEY`, если не указан явно.
- `voiceAliases` позволяет использовать в директивах Talk понятные имена.

---

## Инструменты

### Профили инструментов

`tools.profile` задаёт базовый список разрешённых инструментов перед применением `tools.allow`/`tools.deny`:

| Профиль     | Включает                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | Только `session_status`                                                                   |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Без ограничений (как если бы не задано)                                |

### Группы инструментов

| Группа             | Инструменты                                                                              |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash` допускается как псевдоним для `exec`)       |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Все встроенные инструменты (исключая плагины провайдеров)             |

### `tools.allow` / `tools.deny`

Глобальная политика разрешения/запрета инструментов (запрет имеет приоритет). Без учёта регистра, поддерживаются подстановочные знаки `*`. Применяется даже при отключённой песочнице Docker.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Дополнительное ограничение инструментов для конкретных провайдеров или моделей. Порядок: базовый профиль → профиль провайдера → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Управляет расширенным (host) доступом к `exec`:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- Переопределение для конкретного агента (`agents.list[].tools.elevated`) может только дополнительно ограничивать доступ.
- `/elevated on|off|ask|full` сохраняет состояние для каждой сессии; встроенные директивы применяются к одному сообщению.
- Расширенный `exec` выполняется на хосте, обходя песочницу.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Настраивает обработку входящих медиа (image/audio/video):

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Запись провайдера** (`type: "provider"` или опущено):

- `provider`: идентификатор API‑провайдера (`openai`, `anthropic`, `google`/`gemini`, `groq` и т. д.)
- `model`: переопределение идентификатора модели
- `profile` / `preferredProfile`: выбор профиля аутентификации

**Запись CLI** (`type: "cli"`):

- `command`: исполняемый файл для запуска
- `args`: аргументы с шаблонами (поддерживаются `{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` и т. д.)

**Общие поля:**

- `capabilities`: необязательный список (`image`, `audio`, `video`). По умолчанию: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: переопределения для конкретной записи.
- При сбоях используется следующая запись в списке.

Аутентификация провайдера выполняется в стандартном порядке: профили аутентификации → переменные окружения → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

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

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: модель по умолчанию для создаваемых субагентов. Если не указано, субагенты наследуют модель вызывающего агента.
- Политика инструментов для конкретного субагента: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Пользовательские провайдеры и base URL

OpenClaw использует каталог моделей pi-coding-agent. Добавляйте пользовательских провайдеров через `models.providers` в конфигурации или `~/.openclaw/agents/<agentId>/agent/models.json`.

```json5
{
  models: {
    mode: "merge", // merge (default) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- Используйте `authHeader: true` + `headers` для пользовательских требований к аутентификации.
- Переопределите корневую директорию конфигурации агента с помощью `OPENCLAW_AGENT_DIR` (или `PI_CODING_AGENT_DIR`).

### Примеры провайдеров

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Используйте `cerebras/zai-glm-4.7` для Cerebras; `zai/glm-4.7` — для прямого подключения к Z.AI.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

Установите `OPENCODE_API_KEY` (или `OPENCODE_ZEN_API_KEY`). Сокращённый вариант: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

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

Установите `ZAI_API_KEY`. `z.ai/*` и `z-ai/*` являются допустимыми псевдонимами. Сокращение: `openclaw onboard --auth-choice zai-api-key`.

- Общий endpoint: `https://api.z.ai/api/paas/v4`
- Endpoint для программирования (по умолчанию): `https://api.z.ai/api/coding/paas/v4`
- Для общего endpoint укажите собственного провайдера с переопределением base URL.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Для endpoint в Китае: `baseUrl: "https://api.moonshot.cn/v1"` или `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

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

Совместим с Anthropic, встроенный провайдер. Сокращение: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

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

Base URL не должен включать `/v1` (клиент Anthropic добавляет его автоматически). Сокращение: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Установите `MINIMAX_API_KEY`. Сокращение: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

См. [Local Models](/gateway/local-models). Кратко: запускайте MiniMax M2.1 через LM Studio Responses API на производительном оборудовании; оставьте размещённые модели объединёнными для резервного использования.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: необязательный список разрешённых только для встроенных skills (управляемые/рабочие skills не затрагиваются).
- `entries.<skillKey> .enabled: false` отключает skill, даже если он встроен/установлен.
- `entries.<skillKey> .apiKey`: удобный способ для skills, объявляющих основную переменную окружения.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- Загружаются из `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, а также из `plugins.load.paths`.
- **Изменения конфигурации требуют перезапуска gateway.**
- `allow`: необязательный список разрешённых (загружаются только указанные плагины). `deny` имеет приоритет.

См. [Plugins](/tools/plugin).

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false` отключает `act:evaluate` и `wait --fn`.
- Удалённые профили работают только в режиме подключения (start/stop/reset отключены).
- Порядок автоопределения: браузер по умолчанию, если на базе Chromium → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Сервис управления: только loopback (порт определяется из `gateway.port`, по умолчанию `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: акцентный цвет интерфейса нативного приложения (оттенок пузыря Talk Mode и т. д.).
- `assistant`: Переопределение идентификатора Control UI. Если не задано, используется идентификатор активного агента.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (запуск gateway) или `remote` (подключение к удалённому gateway). Gateway отказывается запускаться, если не установлен режим `local`.
- `port`: единый мультиплексированный порт для WS + HTTP. Приоритет: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (по умолчанию), `lan` (`0.0.0.0`), `tailnet` (только IP Tailscale) или `custom`.
- **Auth**: по умолчанию требуется. При bind, отличном от loopback, требуется общий token/password. Мастер первоначальной настройки по умолчанию генерирует token.
- `auth.mode: "trusted-proxy"`: делегировать аутентификацию identity-aware reverse proxy и доверять заголовкам идентификации от `gateway.trustedProxies` (см. [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: если `true`, заголовки идентификации Tailscale Serve считаются достаточными для аутентификации (проверяется через `tailscale whois`). По умолчанию `true`, если `tailscale.mode = "serve"`.
- `auth.rateLimit`: необязательное ограничение неудачных попыток аутентификации. Применяется для каждого IP клиента и для каждой области аутентификации (shared-secret и device-token учитываются отдельно). Заблокированные попытки возвращают `429` + `Retry-After`.
  - `auth.rateLimit.exemptLoopback` по умолчанию `true`; установите `false`, если намеренно хотите применять ограничение скорости и к localhost (для тестовых окружений или строгих прокси-развёртываний).
- `tailscale.mode`: `serve` (только tailnet, bind на loopback) или `funnel` (публичный доступ, требуется auth).
- `remote.transport`: `ssh` (по умолчанию) или `direct` (ws/wss). Для `direct` значение `remote.url` должно начинаться с `ws://` или `wss://`.
- `gateway.remote.token` используется только для удалённых вызовов CLI; не включает аутентификацию локального gateway.
- `trustedProxies`: IP-адреса reverse proxy, которые завершают TLS. Указывайте только те прокси, которые вы контролируете.
- `gateway.tools.deny`: дополнительные имена инструментов, блокируемые для HTTP `POST /tools/invoke` (расширяет список запретов по умолчанию).
- `gateway.tools.allow`: удалить имена инструментов из стандартного списка запретов HTTP.

</Accordion>

### Эндпоинты, совместимые с OpenAI

- Chat Completions: по умолчанию отключены. Включаются с помощью `gateway.http.endpoints.chatCompletions.enabled: true`.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Усиление безопасности URL-ввода для Responses:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Изоляция нескольких экземпляров

Запустите несколько gateway на одном хосте с уникальными портами и каталогами состояния:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Удобные флаги: `--dev` (использует `~/.openclaw-dev` + порт `19001`), `--profile <name>` (использует `~/.openclaw-<name>`).

См. [Multiple Gateways](/gateway/multiple-gateways).

---

## Хуки

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

Аутентификация: `Authorization: Bearer <token>` или `x-openclaw-token: <token>`.

**Эндпоинты:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey` из тела запроса принимается только когда `hooks.allowRequestSessionKey=true` (по умолчанию: `false`).
- `POST /hooks/<name>` → обрабатывается через `hooks.mappings`

<Accordion title="Mapping details">

- `match.path` соответствует подпути после `/hooks` (например, `/hooks/gmail` → `gmail`).
- `match.source` сопоставляется с полем в payload для универсальных путей.
- Шаблоны вида `{{messages[0].subject}}` читают данные из payload.
- `transform` может указывать на модуль JS/TS, возвращающий действие хука.
  - `transform.module` должен быть относительным путём и находиться в пределах `hooks.transformsDir` (абсолютные пути и выход за пределы каталога отклоняются).
- `agentId` направляет запрос конкретному агенту; неизвестные ID приводят к использованию агента по умолчанию.
- `allowedAgentIds`: ограничивает явную маршрутизацию (`*` или отсутствие значения = разрешить все, `[]` = запретить все).
- `defaultSessionKey`: необязательный фиксированный ключ сессии для запусков hook-агента без явного `sessionKey`.
- `allowRequestSessionKey`: разрешает вызывающим `/hooks/agent` задавать `sessionKey` (по умолчанию: `false`).
- `allowedSessionKeyPrefixes`: необязательный список разрешённых префиксов для явных значений `sessionKey` (запрос + mapping), например `[“hook:”]`.
- `deliver: true` отправляет финальный ответ в канал; `channel` по умолчанию — `last`.
- `model` переопределяет LLM для данного запуска хука (должна быть разрешена, если задан каталог моделей).

</Accordion>

### Интеграция Gmail

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Gateway автоматически запускает `gog gmail watch serve` при загрузке, если настроено. Установите `OPENCLAW_SKIP_GMAIL_WATCHER=1`, чтобы отключить.
- Не запускайте отдельный `gog gmail watch serve` одновременно с Gateway.

---

## Хост Canvas

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // или OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Обслуживает редактируемые агентом HTML/CSS/JS и A2UI по HTTP через порт Gateway:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Только локально: оставьте `gateway.bind: "loopback"` (по умолчанию).
- При привязке не к loopback: маршруты canvas требуют аутентификации Gateway (token/password/trusted-proxy), как и другие HTTP-интерфейсы Gateway.
- Node WebViews обычно не отправляют заголовки аутентификации; после сопряжения и подключения узла Gateway разрешает fallback для частного IP, чтобы узел мог загружать canvas/A2UI без утечки секретов в URL.
- Внедряет клиент live-reload в обслуживаемый HTML.
- Автоматически создаёт стартовый `index.html`, если каталог пуст.
- Также обслуживает A2UI по пути `/__openclaw__/a2ui/`.
- Изменения требуют перезапуска gateway.
- Отключите live reload для больших каталогов или при ошибках `EMFILE`.

---

## Обнаружение

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (по умолчанию): исключает `cliPath` + `sshPort` из TXT-записей.
- `full`: включает `cliPath` + `sshPort`.
- Имя хоста по умолчанию — `openclaw`. Переопределяется с помощью `OPENCLAW_MDNS_HOSTNAME`.

### Wide-area (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

Создаёт зону unicast DNS-SD в каталоге `~/.openclaw/dns/`. Для обнаружения между разными сетями используйте DNS-сервер (рекомендуется CoreDNS) + split DNS в Tailscale.

Настройка: `openclaw dns setup --apply`.

---

## Переменные окружения

### `env` (встроенные переменные окружения)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Встроенные переменные окружения применяются только если соответствующий ключ отсутствует в окружении процесса.
- Файлы `.env`: `.env` в текущем каталоге (CWD) + `~/.openclaw/.env` (ни один из них не переопределяет существующие переменные).
- `shellEnv`: импортирует отсутствующие ожидаемые ключи из профиля вашей login-оболочки.
- См. [Environment](/help/environment) для полной информации о приоритете.

### Подстановка переменных окружения

Ссылайтесь на переменные окружения в любой строке конфигурации с помощью `${VAR_NAME}`:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Сопоставляются только имена в верхнем регистре: `[A-Z_][A-Z0-9_]*`.
- Отсутствующие или пустые переменные вызывают ошибку при загрузке конфигурации.
- Экранируйте с помощью `$${VAR}`, чтобы получить литерал `${VAR}`.
- Работает с `$include`.

---

## Хранилище аутентификации

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

- Профили аутентификации для каждого агента хранятся в `<agentDir>/auth-profiles.json`.
- Устаревшие данные OAuth импортируются из `~/.openclaw/credentials/oauth.json`.
- См. [OAuth](/concepts/oauth).

---

## Логирование

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- Файл лога по умолчанию: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Укажите `logging.file`, чтобы задать постоянный путь.
- `consoleLevel` повышается до `debug` при использовании `--verbose`.

---

## Мастер

Метаданные, записываемые CLI-мастерами (`onboard`, `configure`, `doctor`):

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

---

## Идентичность

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

Написано помощником по онбордингу macOS. Выводит значения по умолчанию:

- `messages.ackReaction` из `identity.emoji` (если не задано, используется 👀)
- `mentionPatterns` из `identity.name`/`identity.emoji`
- `avatar` принимает: относительный путь к рабочей области, URL `http(s)` или URI `data:`

---

## Bridge (устаревший, удалён)

Текущие сборки больше не включают TCP‑bridge. Узлы подключаются через WebSocket Gateway. Ключи `bridge.*` больше не являются частью схемы конфигурации (валидация не проходит, пока они не удалены; `openclaw doctor --fix` может удалить неизвестные ключи).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // строка длительности или false
  },
}
```

- `sessionRetention`: как долго хранить завершённые cron‑сессии перед их удалением. По умолчанию: `24h`.

См. [Cron Jobs](/automation/cron-jobs).

---

## Переменные шаблона модели медиа

Плейсхолдеры шаблона, разворачиваемые в `tools.media.*.models[].args`:

| Переменная         | Описание                                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------------------- |
| `{{Body}}`         | Полное тело входящего сообщения                                                                               |
| `{{RawBody}}`      | Исходное тело (без истории/обёрток отправителя)                                            |
| `{{BodyStripped}}` | Тело без упоминаний группы                                                                                    |
| `{{From}}`         | Идентификатор отправителя                                                                                     |
| `{{To}}`           | Идентификатор получателя                                                                                      |
| `{{MessageSid}}`   | ID сообщения канала                                                                                           |
| `{{SessionId}}`    | UUID текущей сессии                                                                                           |
| `{{IsNewSession}}` | `"true"` при создании новой сессии                                                                            |
| `{{MediaUrl}}`     | Псевдо-URL входящего медиа                                                                                    |
| `{{MediaPath}}`    | Локальный путь к медиа                                                                                        |
| `{{MediaType}}`    | Тип медиа (изображение/аудио/документ/…)                                                   |
| `{{Transcript}}`   | Расшифровка аудио                                                                                             |
| `{{Prompt}}`       | Разрешённый медиапромпт для записей CLI                                                                       |
| `{{MaxChars}}`     | Разрешённое максимальное количество символов вывода для записей CLI                                           |
| `{{ChatType}}`     | `"direct"` или `"group"`                                                                                      |
| `{{GroupSubject}}` | Тема группы (по возможности)                                                               |
| `{{GroupMembers}}` | Предварительный список участников группы (по возможности)                                  |
| `{{SenderName}}`   | Отображаемое имя отправителя (по возможности)                                              |
| `{{SenderE164}}`   | Номер телефона отправителя (по возможности)                                                |
| `{{Provider}}`     | Подсказка провайдера (whatsapp, telegram, discord и т. д.) |

---

## Конфигурация поддерживает (`$include`)

Разделение конфигурации на несколько файлов:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**Поведение объединения:**

- Один файл: заменяет содержащий объект.
- Массив файлов: глубокое объединение по порядку (последующие переопределяют предыдущие).
- Соседние ключи: объединяются после includes (переопределяют включённые значения).
- Вложенные includes: до 10 уровней глубины.
- Пути: относительные (к включающему файлу), абсолютные или ссылки на родительскую директорию `../`.
- Ошибки: понятные сообщения об отсутствующих файлах, ошибках разбора и циклических includes.

---

_См. также: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
