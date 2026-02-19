---
summary: "Статус поддержки Telegram-бота, возможности и конфигурация"
read_when:
  - Работа над функциями Telegram или вебхуками
title: "Telegram"
---

# Telegram (Bot API)

Статус: готово к промышленному использованию для личных сообщений бота и групп через grammY. По умолчанию используется long-polling; вебхук — опционально.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Политика DM по умолчанию для Telegram — pairing.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Диагностика и инструкции по восстановлению для разных каналов.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Полные шаблоны конфигурации каналов и примеры.
  
</Card>
</CardGroup>

## Быстрая настройка (для начинающих)

<Steps>
  <Step title="Create the bot token in BotFather">Откройте Telegram и начните чат с **@BotFather** ([прямая ссылка](https://t.me/BotFather)).

    ```
    Выполните `/newbot`, затем следуйте подсказкам (имя + имя пользователя, оканчивающееся на `bot`).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Вариант через env: `TELEGRAM_BOT_TOKEN=...` (работает для аккаунта по умолчанию).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Неизвестные отправители получают код сопряжения; сообщения игнорируются до подтверждения (коды истекают через 1 час).
    ```

  
</Step>

  <Step title="Add the bot to a group">Добавьте бота в свою группу, затем настройте `channels.telegram.groups` и `groupPolicy` в соответствии с вашей моделью доступа.
</Step>
</Steps>

<Note>
Порядок разрешения токенов учитывает аккаунт. На практике значения из конфигурации имеют приоритет над переменными окружения, а `TELEGRAM_BOT_TOKEN` применяется только к аккаунту по умолчанию.
</Note>

## Настройки на стороне Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Боты Telegram по умолчанию работают в **Privacy Mode**, который ограничивает, какие сообщения в группах они получают.

    ```
    **Примечание:** При переключении режима приватности Telegram требует удалить и заново добавить бота
    в каждую группу, чтобы изменения вступили в силу.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Статус администратора задаётся внутри группы (через интерфейс Telegram).

    ```
    `/setprivacy` — управлять тем, видит ли бот все сообщения в группах.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — разрешить/запретить добавление бота в группы.
    ```

  
</Accordion>
</AccordionGroup>

## Контроль доступа и активация

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` управляет доступом к личным сообщениям:


    ```
    - `pairing` (по умолчанию)
    - `allowlist`
    - `open` (требует, чтобы `allowFrom` включал `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` принимает числовые идентификаторы пользователей Telegram. Префиксы `telegram:` / `tg:` поддерживаются и нормализуются.
    Мастер первичной настройки принимает ввод `@username` и преобразует его в числовые ID.
    Если вы обновились и в конфигурации есть записи allowlist с `@username`, выполните `openclaw doctor --fix` для их преобразования (по возможности; требуется токен Telegram-бота).
    
    ### Как узнать свой Telegram user ID
    
    Более безопасный способ (без сторонних ботов):
    
    1. Напишите боту в личные сообщения.
    2. Выполните `openclaw logs --follow`.
    3. Найдите `from.id`.
    
    Официальный способ через Bot API:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Сторонний способ (менее приватный): `@userinfobot` или `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Два независимых механизма контроля:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Ответы в группах по умолчанию требуют упоминания (нативное @упоминание или `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    Упоминание может происходить через:
    
    - стандартное упоминание `@botusername`, или
    - шаблоны упоминаний в:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Переключатели команд на уровне сессии:
    
    - `/activation always`
    - `/activation mention`
    
    Они изменяют состояние только текущей сессии. Для постоянного эффекта используйте конфигурацию.
    
    Пример постоянной конфигурации:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Перешлите любое сообщение из группы в `@userinfobot` или `@getidsbot` в Telegram, чтобы увидеть ID чата (отрицательное число вроде `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Поведение во время выполнения

- Telegram управляется процессом gateway.
- Детерминированная маршрутизация: ответы всегда возвращаются в Telegram; модель никогда не выбирает каналы.
- Входящие сообщения нормализуются в общий конверт канала с контекстом ответа и плейсхолдерами медиа.
- Сессии групп изолируются по ID группы. Добавляет `:topic:<threadId>` к ключу сеанса группы Telegram, чтобы каждая тема была изолирована.
- В приватных чатах в некоторых пограничных случаях может присутствовать `message_thread_id`. OpenClaw сохраняет ключ сеанса личных сообщений без изменений, но всё равно использует ID треда для ответов/чернового стриминга, если он присутствует.
- Long-polling использует runner grammY с последовательностью на чат; общая конкуррентность ограничена `agents.defaults.maxConcurrent`. Общая конкурентность runner sink задаётся через `agents.defaults.maxConcurrent`.
- Telegram Bot API не поддерживает уведомления о прочтении; опции `sendReadReceipts` не существует.

## Справочник по функциям

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw может стримить частичные ответы в личных сообщениях Telegram с использованием `sendMessageDraft`.

    ```
    Требование:
    
    - `channels.telegram.streamMode` не равно `"off"` (по умолчанию: `"partial"`)
    
    Режимы:
    
    - `off`: без предпросмотра в реальном времени
    - `partial`: частые обновления предпросмотра по мере поступления текста
    - `block`: обновления предпросмотра блоками с использованием `channels.telegram.draftChunk`
    
    Значения `draftChunk` по умолчанию для `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` ограничивается значением `channels.telegram.textChunkLimit`.
    
    Работает в личных чатах и группах/темах.
    
    Для текстовых ответов OpenClaw сохраняет то же сообщение предпросмотра и выполняет финальное редактирование на месте (без второго сообщения).
    
    Для сложных ответов (например, с медиа-вложениями) OpenClaw возвращается к обычной финальной отправке и затем удаляет сообщение предпросмотра.
    
    `streamMode` независим от потоковой передачи блоками. Если для Telegram явно включена блочная потоковая передача, OpenClaw пропускает предпросмотр, чтобы избежать двойного стриминга.
    
    Поток reasoning только для Telegram:
    
    - `/reasoning stream` отправляет reasoning в предпросмотр в реальном времени во время генерации
    - финальный ответ отправляется без текста reasoning
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Исходящий текст Telegram использует `parse_mode: "HTML"` (подмножество поддерживаемых тегов Telegram).

    ```
    - Текст в стиле Markdown преобразуется в безопасный для Telegram HTML.
    - Исходный HTML модели экранируется, чтобы уменьшить ошибки парсинга в Telegram.
    - Если Telegram отклоняет разобранный HTML, OpenClaw повторяет отправку как обычный текст.
    
    Предпросмотр ссылок включён по умолчанию и может быть отключён с помощью `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Некоторые команды могут обрабатываться плагинами/навыками без регистрации в меню команд Telegram. Они всё равно работают при вводе (просто не отображаются в `/commands` / меню).

    ```
    OpenClaw регистрирует нативные команды (например, `/status`, `/reset`, `/model`) в меню бота Telegram при запуске. Вы можете добавить пользовательские команды в меню через конфиг:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Правила:
    
    - имена нормализуются (удаляется ведущий `/`, приводятся к нижнему регистру)
    - допустимый шаблон: `a-z`, `0-9`, `_`, длина `1..32`
    - пользовательские команды не могут переопределять встроенные команды
    - конфликты/дубликаты пропускаются и логируются
    
    Примечания:
    
    - пользовательские команды — это только пункты меню; они не реализуют поведение автоматически
    - команды плагинов/skills могут работать при вводе вручную, даже если не отображаются в меню Telegram
    
    Если встроенные команды отключены, они удаляются. Пользовательские/плагин-команды всё равно могут регистрироваться при соответствующей настройке.
    
    Распространённая ошибка настройки:
    
    - `setMyCommands failed` обычно означает, что исходящие DNS/HTTPS-запросы к `api.telegram.org` заблокированы.
    
    ### Команды сопряжения устройства (плагин `device-pair`)
    
    Если установлен плагин `device-pair`:
    
    1. `/pair` генерирует код настройки
    2. вставьте код в приложение iOS
    3. `/pair approve` подтверждает последний ожидающий запрос
    
    Подробнее: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Настройте область действия inline-клавиатуры:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Переопределение для конкретного аккаунта:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = сообщения из групп не принимаются вообще
      Значение по умолчанию — `groupPolicy: "allowlist"` (заблокировано, если вы не добавили `groupAllowFrom`).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Когда пользователь нажимает кнопку, данные callback отправляются агенту как сообщение в формате:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Действия инструментов Telegram включают:


    ```
    - `sendMessage` (`to`, `content`, необязательно `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Действия с сообщениями канала предоставляют удобные алиасы (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Управление доступом:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (по умолчанию: отключено)
    
    Семантика удаления реакций: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram поддерживает опциональные тредированные ответы с помощью тегов:

    ```
    - `[[reply_to_current]]` — ответ на сообщение-инициатор
    - `[[reply_to:<id>]]` — ответ на конкретное сообщение Telegram по ID
    
    `channels.telegram.replyToMode` управляет обработкой:
    
    - `off` (по умолчанию)
    - `first`
    - `all`
    
    Примечание: `off` отключает неявную привязку ответов к сообщениям. Явные теги `[[reply_to_*]]` всё равно учитываются.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Форумные супергруппы:


    ```
    **Форумные группы:** Реакции в форумных группах включают `message_thread_id` и используют ключи сеансов вида `agent:main:telegram:group:{chatId}:topic:{threadId}`.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Аудиосообщения


    ```
    Telegram различает голосовые сообщения и аудиофайлы.
    
    - по умолчанию: поведение как у аудиофайла
    - тег `[[audio_as_voice]]` в ответе агента принудительно отправляет как голосовое сообщение
    
    Пример действия сообщения:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Видеосообщения
    
    Telegram различает видеофайлы и видеозаметки.
    
    Пример действия сообщения:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Видеозаметки не поддерживают подписи; предоставленный текст сообщения отправляется отдельно.
    
    ### Стикеры
    
    Обработка входящих стикеров:
    
    - статические WEBP: загружаются и обрабатываются (плейсхолдер `<media:sticker>`)
    - анимированные TGS: пропускаются
    - видео WEBM: пропускаются
    
    Поля контекста стикера:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Файл кэша стикеров:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Стикеры описываются один раз (по возможности) и кэшируются, чтобы сократить повторные вызовы vision.
    
    Включить действия со стикерами:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Отправить стикер:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Поиск стикеров в кэше:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Получает обновление `message_reaction` от Telegram API

    ```
    Когда включено, OpenClaw добавляет в очередь системные события, например:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Конфигурация:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (по умолчанию: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (по умолчанию: `minimal`)
    
    Примечания:
    
    - `own` означает реакции пользователей только на сообщения, отправленные ботом (по возможности через кэш отправленных сообщений).
    - Telegram не предоставляет ID темы в обновлениях реакций.
      - обычные группы направляются в сессию группового чата
      - форумные группы направляются в сессию общей темы группы (`:topic:1`), а не в исходную тему
    
    `allowed_updates` для polling/webhook автоматически включает `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` отправляет эмодзи-подтверждение, пока OpenClaw обрабатывает входящее сообщение.


    ```
    Порядок разрешения:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - резервный эмодзи из идентичности агента (`agents.list[].identity.emoji`, иначе "👀")
    
    Примечания:
    
    - Telegram ожидает emoji в формате unicode (например, "👀").
    - Используйте `""`, чтобы отключить реакцию для канала или аккаунта.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Запись конфигурации канала включена по умолчанию (`configWrites !== false`).

    ```
    Группа обновляется до супергруппы, и Telegram отправляет `migrate_to_chat_id` (ID чата меняется). OpenClaw может автоматически мигрировать `channels.telegram.groups`.
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">По умолчанию: long-polling (публичный URL не требуется).

    ```
    Режим вебхука: установите `channels.telegram.webhookUrl` и `channels.telegram.webhookSecret` (опционально `channels.telegram.webhookPath`).
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Исходящий текст разбивается на блоки по `channels.telegram.textChunkLimit` (по умолчанию 4000).
    Необязательное разбиение по переводам строк: установите `channels.telegram.chunkMode="newline"`, чтобы сначала делить по пустым строкам (границы абзацев), а затем по длине.
    Загрузка/выгрузка медиа ограничена `channels.telegram.mediaMaxMb` (по умолчанию 5).
    Запросы Telegram Bot API истекают по тайм-ауту через `channels.telegram.timeoutSeconds` (по умолчанию 500 через grammY).
    Контекст истории группы использует `channels.telegram.historyLimit` (или `channels.telegram.accounts.*.historyLimit`), с fallback на `messages.groupChat.historyLimit`. Установите `0`, чтобы отключить (по умолчанию 50).
    - Управление историей DM:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - Повторные попытки исходящих запросов к Telegram API настраиваются через `channels.telegram.retry`.

    ```
    Цель отправки в CLI может быть числовым chat ID или именем пользователя:
    ```

```bash
Пример: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Устранение неполадок

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Если вы установили `channels.telegram.groups.*.requireMention=false`, **режим приватности** Bot API Telegram должен быть отключён.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - если существует `channels.telegram.groups`, группа должна быть указана в списке (или включать `"*"`)
    - проверьте, что бот состоит в группе
    - просмотрите логи: `openclaw logs --follow`, чтобы узнать причины пропуска
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - авторизуйте свою личность отправителя (сопряжение и/или числовой `allowFrom`)
    - авторизация команд по-прежнему применяется, даже если политика группы — `open`
    - `setMyCommands failed` обычно указывает на проблемы с доступностью DNS/HTTPS к `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + пользовательский fetch/proxy могут вызывать немедленное прерывание, если типы AbortSignal не совпадают.
    - Некоторые хосты сначала резолвят `api.telegram.org` в IPv6; некорректный IPv6 egress может вызывать периодические сбои Telegram API.
    - Проверьте ответы DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Дополнительная помощь: [Устранение неполадок каналов](/channels/troubleshooting).

## Справочник конфигурации (Telegram)

Основная ссылка:

- `channels.telegram.enabled`: включить/выключить запуск канала.

- `channels.telegram.botToken`: токен бота (BotFather).

- `channels.telegram.tokenFile`: читать токен из пути к файлу.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (по умолчанию: сопряжение).

- `channels.telegram.allowFrom`: allowlist личных сообщений (ids/usernames). `open` требует `"*"`. `openclaw doctor --fix` может преобразовать устаревшие записи `@username` в ID.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (по умолчанию: allowlist).

- `channels.telegram.groupAllowFrom`: allowlist отправителей в группах (ids/usernames). `openclaw doctor --fix` может преобразовать устаревшие записи `@username` в ID.

- `channels.telegram.groups`: значения по умолчанию для групп + allowlist (используйте `"*"` для глобальных значений).
  - `channels.telegram.groups.<id>.groupPolicy`: переопределение groupPolicy для группы (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: поведение требований упоминаний по умолчанию.
  - `channels.telegram.groups.<id>.skills`: фильтр skills (не указано = все skills, пусто = ни одного).
  - `channels.telegram.groups.<id>.allowFrom`: переопределение allowlist отправителей для группы.
  - `channels.telegram.groups.<id>.systemPrompt`: дополнительный системный prompt для группы.
  - `channels.telegram.groups.<id>.enabled`: отключить группу, когда `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: переопределения для тем (те же поля, что и у группы).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: переопределение groupPolicy для темы (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: переопределение требований упоминаний для темы.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (по умолчанию: allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: переопределение на уровне аккаунта.

- `channels.telegram.replyToMode`: `off | first | all` (по умолчанию: `first`).

- `channels.telegram.textChunkLimit`: размер исходящих блоков (символы).

- `channels.telegram.chunkMode`: `length` (по умолчанию) или `newline` для разбиения по пустым строкам (границы абзацев) перед разбиением по длине.

- `channels.telegram.linkPreview`: переключение предпросмотра ссылок для исходящих сообщений (по умолчанию: true).

- `channels.telegram.streamMode`: `off | partial | block` (черновой стриминг).

- `channels.telegram.mediaMaxMb`: лимит входящих/исходящих медиа (МБ).

- `channels.telegram.retry`: политика повторных попыток для исходящих вызовов Telegram API (attempts, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: переопределение Node autoSelectFamily (true=включить, false=выключить). По умолчанию отключено на Node 22, чтобы избежать тайм-аутов Happy Eyeballs.

- `channels.telegram.proxy`: URL прокси для вызовов Bot API (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: включить режим вебхука (требует `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: секрет вебхука (обязателен, когда задан webhookUrl).

- `channels.telegram.webhookPath`: локальный путь вебхука (по умолчанию `/telegram-webhook`).

- Локальный слушатель привязывается к `0.0.0.0:8787` и по умолчанию обслуживает `POST /telegram-webhook`.

- `channels.telegram.actions.reactions`: ограничить реакции инструмента Telegram.

- `channels.telegram.actions.sendMessage`: ограничить отправку сообщений инструментом Telegram.

- `channels.telegram.actions.deleteMessage`: ограничить удаление сообщений инструментом Telegram.

- `channels.telegram.actions.sticker`: ограничить действия со стикерами Telegram — отправка и поиск (по умолчанию: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — управляет тем, какие реакции вызывают системные события (по умолчанию: `own`, если не задано).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — управляет возможностями агента по реакциям (по умолчанию: `minimal`, если не задано).

- [Справочник по конфигурации - Telegram](/gateway/configuration-reference#telegram)

Специфичные для Telegram ключевые поля:

- запуск/аутентификация: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- контроль доступа: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- команды/меню: `commands.native`, `customCommands`
- потоки/ответы: `replyToMode`
- `channels.telegram.streamMode: "off" | "partial" | "block"` (по умолчанию: `partial`)
- форматирование/доставка: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- медиа/сеть: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Инструмент: `telegram` с действием `sendMessage` (`to`, `content`, необязательные `mediaUrl`, `replyToMessageId`, `messageThreadId`).
- действия/возможности: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- Уведомления о реакциях
- запись/история: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Связанные материалы

- Подробнее: [Pairing](/channels/pairing)
- [Маршрутизация каналов](/channels/channel-routing)
- Устранение неполадок настройки (команды)

