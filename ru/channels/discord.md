---
summary: "Статус поддержки Discord-бота, возможности и конфигурация"
read_when:
  - Работа над возможностями канала Discord
title: "Discord"
---

# Discord (Bot API)

Статус: готов для личных сообщений (DM) и текстовых каналов серверов (guild) через официальный шлюз Discord для ботов.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Личные сообщения Discord по умолчанию используют режим сопряжения.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Поведение нативных команд и каталог команд.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Диагностика между каналами и процесс восстановления.
  
</Card>
</CardGroup>

## Быстрая настройка (для начинающих)

<Steps>
  <Step title="Create a Discord bot and enable intents">Создайте приложение Discord + пользователя-бота

    ```
    **Server Members Intent** (рекомендуется; требуется для некоторых поисков участников/пользователей и сопоставления списков разрешённых на серверах)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Резервное значение из env для учётной записи по умолчанию:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Пригласите бота на свой сервер с правами на сообщения (создайте приватный сервер, если вам нужны только DM).

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Коды сопряжения истекают через 1 час.
    ```

  
</Step>
</Steps>

<Note>
Разрешение токена учитывает учётную запись. Значения токена из конфигурации имеют приоритет над резервными значениями из env. `DISCORD_BOT_TOKEN` используется только для учётной записи по умолчанию.
</Note>

## Модель времени выполнения

- Gateway управляет подключением к Discord.
- Маршрутизация ответов детерминирована: входящие ответы из Discord возвращаются в Discord.
- Агент может вызывать `discord` с действиями, такими как:
- Прямые чаты сворачиваются в основной сеанс агента (по умолчанию `agent:main:main`); каналы серверов остаются изолированными как `agent:<agentId>:discord:channel:<channelId>` (отображаемые имена используют `discord:<guildSlug>#<channelSlug>`).
- Групповые DM по умолчанию игнорируются; включите через `channels.discord.dm.groupEnabled` и при необходимости ограничьте с помощью `channels.discord.dm.groupChannels`.
- Нативные команды используют изолированные ключи сеансов (`agent:<agentId>:discord:slash:<userId>`), а не общий сеанс `main`.

## Контроль доступа и маршрутизация

<Tabs>
  <Tab title="DM policy">Чтобы игнорировать все DM: установите `channels.discord.dm.enabled=false` или `channels.discord.dm.policy="disabled"`.

    ```
    Для жёсткого списка разрешённых: установите `channels.discord.dm.policy="allowlist"` и перечислите отправителей в `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Обработка Guild управляется параметром `channels.discord.groupPolicy`:

    ```
    Чтобы не разрешать **ни одного канала**, установите `channels.discord.groupPolicy: "disabled"` (или оставьте пустой список разрешённых).
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Если вы установили только `DISCORD_BOT_TOKEN` и никогда не создавали раздел `channels.discord`, во время выполнения
        значение `groupPolicy` по умолчанию устанавливается в `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Сообщения Guild по умолчанию требуют упоминания бота.

    ```
    Распознавание упоминаний включает:
    
    - явное упоминание бота
    - настроенные шаблоны упоминаний (`agents.list[].groupChat.mentionPatterns`, резервно `messages.groupChat.mentionPatterns`)
    - неявное поведение ответа боту в поддерживаемых случаях
    
    `requireMention` настраивается для каждого guild/канала (`channels.discord.guilds...`).
    
    Group DM:
    
    - по умолчанию: игнорируются (`dm.groupEnabled=false`)
    - опциональный allowlist через `dm.groupChannels` (ID каналов или slug)
    ```

  
</Tab>
</Tabs>

### Маршрутизация агентов на основе ролей

Используйте `bindings[].match.roles`, чтобы направлять участников Discord guild к разным агентам по ID роли. Привязки на основе ролей принимают только ID ролей и вычисляются после привязок peer или parent-peer и перед привязками только для guild. Если привязка также задаёт другие поля match (например, `peer` + `guildId` + `roles`), должны совпадать все настроенные поля.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Настройка Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">В **Bot** → **Privileged Gateway Intents** включите:

    ```
    - Message Content Intent
    - Server Members Intent (рекомендуется)
    
    Intent присутствия (Presence intent) является необязательным и требуется только если вы хотите получать обновления присутствия. Установка присутствия бота (`setPresence`) не требует включения обновлений присутствия для участников.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">В приложении: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    Типовой базовый набор разрешений:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (опционально)
    
    Избегайте `Administrator`, если это не требуется явно.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Включите Discord Developer Mode, затем скопируйте:

    ```
    - ID сервера
    - ID канала
    - ID пользователя
    
    Предпочитайте числовые ID в конфигурации OpenClaw для надёжных аудитов и проверок.
    ```

  
</Accordion>
</AccordionGroup>

## Встроенные команды и авторизация команд

- Необязательные нативные команды: `commands.native` по умолчанию равен `"auto"` (включено для Discord/Telegram, выключено для Slack).
- Поведение управляется через `channels.discord.replyToMode`:
- Переопределяется через `channels.discord.commands.native: true|false|"auto"`; `false` очищает ранее зарегистрированные команды.
- Авторизация встроенных команд использует те же allowlist/политики Discord, что и обычная обработка сообщений.
- Slash-команды могут быть видны в UI Discord пользователям вне списков разрешённых; OpenClaw проверяет доступ при выполнении и отвечает «not authorized».

См. [Slash commands](/tools/slash-commands) для каталога команд и описания поведения.

## Подробности функций

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord поддерживает теги ответа в выводе агента:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Управляется параметром `channels.discord.replyToMode`:
    
    - `off` (по умолчанию)
    - `first`
    - `all`
    
    Примечание: `off` отключает неявную ветвизацию ответов. Явные теги `[[reply_to_*]]` по-прежнему учитываются.
    
    ID сообщений передаются в контекст/историю, чтобы агенты могли нацеливаться на конкретные сообщения.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Контекст истории Guild:

    ```
    - `channels.discord.historyLimit` по умолчанию `20`
    - резервно: `messages.groupChat.historyLimit`
    - `0` отключает
    
    Настройки истории DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Поведение тредов:
    
    - треды Discord маршрутизируются как сессии канала
    - метаданные родительского треда могут использоваться для привязки к родительской сессии
    - конфигурация треда наследует конфигурацию родительского канала, если не задана отдельная запись для треда
    
    Темы каналов добавляются как **недоверенный** контекст (не как системный prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: режим системных событий реакций (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` отправляет emoji-подтверждение, пока OpenClaw обрабатывает входящее сообщение.

    ```
    Порядок разрешения:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - резервно emoji из идентичности агента (`agents.list[].identity.emoji`, иначе "👀")
    
    Примечания:
    
    - Discord принимает unicode emoji или имена пользовательских emoji.
    - Используйте `""`, чтобы отключить реакцию для канала или учётной записи.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Запись конфигурации, инициированная из канала, включена по умолчанию.

    ```
    Это влияет на сценарии `/config set|unset` (когда функции команд включены).
    
    Отключить:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Маршрутизируйте WebSocket-трафик Discord gateway через HTTP(S)-прокси с помощью `channels.discord.proxy`.

```json5
Или конфиг: `channels.discord.token: "..."`.
```

    ```
    Переопределение для отдельной учётной записи:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    Включите разрешение PluralKit для сопоставления проксированных сообщений с личностью участника системы:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Примечания:
    
    - allowlist может использовать `pk:<memberId>`
    - отображаемые имена участников сопоставляются по имени/slug
    - поиск использует исходный ID сообщения и ограничен временным окном
    - если поиск не удался, проксированные сообщения рассматриваются как сообщения бота и отбрасываются, если только `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Обновления присутствия применяются только при установке поля статуса или активности.

    ```
    Пример только со статусом:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Пример активности (custom status — тип активности по умолчанию):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Пример потоковой трансляции:
    ```

```json5
Запустите Gateway (шлюз); он автоматически запускает канал Discord, когда доступен токен (сначала конфиг, затем env как запасной вариант) и `channels.discord.enabled` не равно `false`.
```

    ```
    Карта типов активности:
    
    - 0: Играет
    - 1: Ведёт трансляцию (требуется `activityUrl`)
    - 2: Слушает
    - 3: Смотрит
    - 4: Пользовательский (использует текст активности как статус; эмодзи необязателен)
    - 5: Соревнуется
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord поддерживает подтверждение выполнения на основе кнопок в личных сообщениях и при необходимости может публиковать запросы на подтверждение в исходном канале.

    ```
    **Подтверждения выполнения (exec approvals) в Discord**: Discord поддерживает **UI с кнопками** для подтверждений в DM (Allow once / Always allow / Deny). `/approve <id> ...` используется только для пересылаемых подтверждений и не решает запросы с кнопками Discord. Если вы видите `❌ Failed to submit approval: Error: unknown approval id` или UI не появляется, проверьте:
    ```

  
</Accordion>
</AccordionGroup>

## Действия инструментов

Действия сообщений Discord включают отправку сообщений, администрирование каналов, модерацию, управление присутствием и действия с метаданными.

Основные примеры:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- Реакции + список реакций + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Гейты действий находятся в `channels.discord.actions.*`.

Поведение гейта по умолчанию:

| Группа действий                                                                                               | По умолчанию |
| ------------------------------------------------------------------------------------------------------------- | ------------ |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled      |
| роли                                                                                                          | отключено    |
| модерация                                                                                                     | отключено    |
| присутствие                                                                                                   | отключено    |

## Интерфейс Components v2

OpenClaw использует компоненты Discord v2 для подтверждений выполнения и маркеров между контекстами. Действия сообщений Discord также могут принимать `components` для пользовательского интерфейса (расширенный вариант; требует экземпляров компонентов Carbon), при этом устаревшие `embeds` остаются доступными, но не рекомендуются.

- `channels.discord.ui.components.accentColor` задаёт акцентный цвет, используемый контейнерами компонентов Discord (hex).
- Задаётся для каждой учётной записи в `channels.discord.accounts.<id> .ui.components.accentColor`.
- `embeds` игнорируются при наличии компонентов v2.

Пример:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

## messages

Голосовые сообщения Discord отображают предварительный просмотр формы волны и требуют аудио в формате OGG/Opus плюс метаданные. OpenClaw генерирует форму волны автоматически, однако на хосте gateway должны быть доступны `ffmpeg` и `ffprobe` для анализа и конвертации аудиофайлов.

Возможности и ограничения

- Укажите **локальный путь к файлу** (URL-адреса отклоняются).
- Опустите текстовое содержимое (Discord не допускает текст + голосовое сообщение в одном payload).
- Принимается любой аудиоформат; при необходимости OpenClaw конвертирует в OGG/Opus.

Пример:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Устранение неполадок

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - включите Message Content Intent
    - включите Server Members Intent, если используете разрешение пользователей/участников
    - перезапустите gateway после изменения intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `channels.discord.groupPolicy` по умолчанию равен **allowlist**; установите `"open"` или добавьте запись сервера в `channels.discord.guilds` (при необходимости перечислите каналы в `channels.discord.guilds.<id>
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Распространённые причины:

    ```
    `groupPolicy`: управление обработкой каналов серверов (`open|disabled|allowlist`); `allowlist` требует списков разрешённых каналов.
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Аудиты прав** (`channels status --probe`) проверяют только числовые id каналов.

    ```
    Если вы используете slug/имена как ключи `channels.discord.guilds.*.channels`, аудит не сможет проверить права.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DM не работают**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` или вы ещё не одобрены (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">Сообщения, созданные ботом, по умолчанию игнорируются; установите `channels.discord.allowBots=true`, чтобы разрешить их (собственные сообщения всё равно фильтруются).

    ```
    Предупреждение: если вы разрешаете ответы другим ботам (`channels.discord.allowBots=true`), предотвратите циклы «бот↔бот» с помощью списков разрешённых `requireMention`, `channels.discord.guilds.*.channels.<id>
    ```

  
</Accordion>
</AccordionGroup>

## Ссылки на справочник конфигурации

Основной справочник:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Ключевые поля Discord:

- `presence` (статус/активность бота, по умолчанию `false`)
- `guilds.<id> .channels.<channel> .allow`: разрешить/запретить канал при `groupPolicy="allowlist"`.
- Используйте `commands.useAccessGroups: false`, чтобы обойти проверки групп доступа для команд.
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Безопасность и эксплуатация

- Рассматривайте токены бота как секреты (`DISCORD_BOT_TOKEN` предпочтителен в управляемых средах).
- Предоставьте Discord минимально необходимые разрешения (least-privilege).
- Если команда deploy/state устарела, перезапустите gateway и повторно проверьте с помощью `openclaw channels status --probe`.

## Связано

- [Pairing](/channels/pairing)
- Управление каналами/категориями
- [Troubleshooting](/channels/troubleshooting)
- Полный список команд и конфиг: [Slash commands](/tools/slash-commands)

