---
summary: "Настройка Slack для режима Socket или HTTP webhook"
read_when:
  - Настройка Slack или отладка режимов Slack socket/HTTP
title: "Slack"
---

# Slack

Статус: готово к продакшену для личных сообщений и каналов через интеграции Slack app. HTTP mode (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Личные сообщения Slack по умолчанию работают в режиме сопряжения.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Нативное поведение команд и каталог команд.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Диагностика между каналами и сценарии восстановления.
  
</Card>
</CardGroup>

## Быстрая настройка (для начинающих)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        В настройках Slack app:

        ```
        Создайте **App Token** (`xapp-...`) и **Bot Token** (`xoxb-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Резервное использование переменных окружения (только для аккаунта по умолчанию):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Подписка на события приложения">
          Подпишитесь на события бота:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Также включите вкладку App Home **Messages Tab** для личных сообщений.
        
</Step>
      
        <Step title="Запуск шлюза">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        **Event Subscriptions** → включите события и задайте **Request URL** на путь webhook вашего шлюза (по умолчанию `/slack/events`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      HTTP mode с несколькими аккаунтами: задайте `channels.slack.accounts.<id> .mode = "http"` и укажите уникальный
      `webhookPath` для каждого аккаунта, чтобы каждое приложение Slack указывало на свой URL.

  
</Tab>
</Tabs>

## Модель токенов

- `botToken` + `appToken` обязательны для Socket Mode.
- Для HTTP-режима требуются `botToken` + `signingSecret`.
- Токены из конфигурации имеют приоритет над переменными окружения.
- Резервные переменные окружения `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` применяются только к аккаунту по умолчанию.
- User token scopes (необязательные, по умолчанию только чтение)
- Дополнительно: добавьте `chat:write.customize`, если хотите, чтобы исходящие сообщения использовали активную идентичность агента (настраиваемые `username` и иконку). `icon_emoji` использует синтаксис `:emoji_name:`.

<Tip>
Для действий/чтения каталога при наличии настройки может использоваться пользовательский токен. Даже при `userTokenReadOnly: false` bot token остаётся
предпочтительным для записи, когда он доступен.
</Tip>

## Контроль доступа и маршрутизация

<Tabs>
  <Tab title="DM policy">DMs игнорируются: отправитель не одобрен при `channels.slack.dm.policy="pairing"`.

    ```
    Slash Commands → создайте `/openclaw`, если используете `channels.slack.slashCommand`. Если вы включаете нативные команды, добавьте по одной slash-команде на каждую встроенную команду (с теми же именами, что и `/help`). По умолчанию нативные команды для Slack выключены, если вы не зададите `channels.slack.commands.native: true` (глобальное `commands.native` равно `"auto"`, что оставляет Slack выключенным).
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` управляет обработкой каналов (`open|disabled|allowlist`).

    ```
    Чтобы разрешить всем: задайте `channels.slack.dm.policy="open"` и `channels.slack.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Сообщения в каналах по умолчанию обрабатываются только при упоминании.
  

    ```
    Контроль упоминаний управляется через `channels.slack.channels` (установите `requireMention` в `true`); `agents.list[].groupChat.mentionPatterns` (или `messages.groupChat.mentionPatterns`) также считаются упоминаниями.
    ```

  
</Tab>
</Tabs>

## Команды и поведение slash-команд

- Автоматический режим нативных команд **отключён** для Slack (`commands.native: "auto"` не включает нативные команды Slack).
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Когда нативные команды включены, зарегистрируйте соответствующие slash-команды в Slack (имена `/<command>`).
- Если вы включаете нативные команды, добавьте по одному элементу `slash_commands` для каждой команды, которую хотите открыть (в соответствии со списком `/help`). Переопределяйте с помощью `channels.slack.commands.native`.

Значение по умолчанию провайдера (`off`)

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Сессии slash-команд используют изолированные ключи:

- Slash‑команды используют сессии `agent:<agentId>:slack:slash:<userId>` (префикс настраивается через `channels.slack.slashCommand.sessionPrefix`).

и при этом выполнение команды направляется в сессию целевого диалога (`CommandTargetSessionKey`).

## Потоки, сессии и теги ответов

- Тредить групповые DMs, но оставлять каналы в корне:
- При значении по умолчанию `session.dmScope=main` личные сообщения Slack объединяются в основную сессию агента.
- Каналы сопоставляются с сессиями `agent:<agentId>:slack:channel:<channelId>`.
- Ответы в тредах могут создавать суффиксы сессий тредов (`:thread:<threadTs>`), когда это применимо.
- Значение по умолчанию `channels.slack.thread.historyScope` — `thread`; `thread.inheritParent` по умолчанию — `false`.
- `channels.slack.historyLimit` (или `channels.slack.accounts.*.historyLimit`) управляет тем, сколько последних сообщений канала/группы включается в prompt.

Управление ответами в тредах:

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- устаревший резервный режим для личных чатов: `channels.slack.dm.replyToMode`

Поддерживаются ручные теги ответа:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Примечание: `replyToMode="off"` отключает неявное ветвление ответов. Явные теги `[[reply_to_*]]` по‑прежнему обрабатываются.

## Медиа, разбиение на части и доставка

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Вложения файлов Slack загружаются с приватных URL, размещённых в Slack (поток запросов с аутентификацией по токену), и сохраняются в хранилище медиа при успешной загрузке и соблюдении ограничений по размеру.


    ```
    Загрузка медиа ограничена `channels.slack.mediaMaxMb` (по умолчанию 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">- текстовые части используют `channels.slack.textChunkLimit` (по умолчанию 4000)
    - `channels.slack.chunkMode="newline"` включает разбиение сначала по абзацам
    - отправка файлов использует API загрузки Slack и может включать ответы в тредах (`thread_ts`)
    - ограничение на исходящие медиа следует `channels.slack.mediaMaxMb`, если задано; в противном случае используются значения по умолчанию для типа MIME из медиапайплайна
</Accordion>

  <Accordion title="Delivery targets">
    Предпочтительные явные цели:

    ```
    `im:write` (открытие личных сообщений через `conversations.open` для DMs пользователей)
    [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
    ```

  
</Accordion>
</AccordionGroup>

## Действия и ограничения

Действия инструментов Slack можно ограничивать через `channels.slack.actions.*`:

Доступные группы действий в текущем инструментарии Slack:

| Группа действий | По умолчанию |
| --------------- | ------------ |
| messages        | enabled      |
| reactions       | enabled      |
| pins            | enabled      |
| memberInfo      | enabled      |
| emojiList       | enabled      |

## События и операционное поведение

- Редактирование/удаление сообщений и рассылка в тредах сопоставляются с системными событиями.
- События добавления/удаления реакций сопоставляются с системными событиями.
- События входа/выхода участников, создания/переименования каналов и добавления/удаления закреплений сопоставляются с системными событиями.
- `channel_id_changed` может переносить ключи конфигурации канала, если включён `configWrites`.
- Метаданные темы/назначения канала рассматриваются как недоверенный контекст и могут быть внедрены в контекст маршрутизации.

## Реакции + список реакций

`ackReaction` отправляет эмодзи‑подтверждение, пока OpenClaw обрабатывает входящее сообщение.

Порядок разрешения:

- `или`channels.slack.channels.<name>`.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- эмодзи идентичности агента по умолчанию (`agents.list[].identity.emoji`, иначе "👀")

Примечания

- Slack ожидает короткие коды (например, `"eyes"`).
- Используйте `""`, чтобы отключить реакцию для канала или учётной записи.

## Контрольный список манифеста и областей доступа

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Если вы настраиваете `channels.slack.userToken`, типичные области доступа для чтения:


    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Устранение неполадок

<AccordionGroup>
  <Accordion title="No replies in channels">
    Проверьте по порядку:


    ```
    `users`: необязательный allowlist пользователей для канала.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Проверьте:


    ```
    `allowlist` требует, чтобы каналы были перечислены в `channels.slack.channels`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Создайте приложение Slack и включите **Socket Mode**.
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Убедитесь:


    ```
    Используйте HTTP webhook режим, когда ваш Gateway (шлюз) доступен для Slack по HTTPS (типично для серверных развёртываний). HTTP mode использует Events API + Interactivity + Slash Commands с общим URL запроса.
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Проверьте, что вы намеревались использовать:
- нативный режим команд (`channels.slack.commands.native: true`) с соответствующими slash‑командами, зарегистрированными в Slack
- или режим одной slash‑команды (`channels.slack.slashCommand.enabled: true`)

Также проверьте `commands.useAccessGroups` и списки разрешённых каналов/пользователей.

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## Основной справочник:

Ключевые поля Slack:

- `users:read` (поиск пользователей)
  [https://docs.slack.dev/reference/methods/users.info](https://docs.slack.dev/reference/methods/users.info)

  mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`

  - Доступ к DM: `dm.enabled`, `dmPolicy`, `allowFrom` (устаревшие: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - треды/история: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - `allow`: разрешить/запретить канал, когда `groupPolicy="allowlist"`.
  - доставка: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - операции/функции: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`
  - ops/features: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Связанные материалы

- [Сопряжение](/channels/pairing)
- `channel`: обычные каналы (публичные/приватные)
- схему триажа: [/channels/troubleshooting](/channels/troubleshooting).
- [Конфигурация](/gateway/configuration)
- Полный список команд и конфигурация: [Slash commands](/tools/slash-commands)
