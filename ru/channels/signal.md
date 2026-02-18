---
title: "Signal"
---

# Signal (signal-cli)

Статус: внешняя интеграция через CLI. Gateway (шлюз) взаимодействует с `signal-cli` по HTTP JSON-RPC + SSE.

## Быстрая настройка (для начинающих)

1. Используйте **отдельный номер Signal** для бота (рекомендуется).
2. Установите `signal-cli` (требуется Java).
3. Свяжите устройство бота и запустите демон:
   - `signal-cli link -n "OpenClaw"`
4. Настройте OpenClaw и запустите шлюз.

Минимальный конфиг:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## Что это такое

- Канал Signal через `signal-cli` (не встроенная libsignal).
- Детерминированная маршрутизация: ответы всегда возвращаются в Signal.
- Личные сообщения используют основной сеанс агента; группы изолированы (`agent:<agentId>:signal:group:<groupId>`).

## Запись конфига

По умолчанию Signal разрешено записывать обновления конфига, инициированные `/config set|unset` (требуется `commands.config: true`).

Отключение:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Модель номеров (важно)

- Шлюз подключается к **устройству Signal** (аккаунт `signal-cli`).
- Если вы запускаете бота на **вашем личном аккаунте Signal**, он будет игнорировать ваши собственные сообщения (защита от зацикливания).
- Для сценария «я пишу боту, и он отвечает» используйте **отдельный номер бота**.

## Настройка (быстрый путь)

1. Установите `signal-cli` (требуется Java).
2. Свяжите аккаунт бота:
   - `signal-cli link -n "OpenClaw"`, затем отсканируйте QR-код в Signal.
3. Настройте Signal и запустите шлюз.

Пример:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Поддержка нескольких аккаунтов: используйте `channels.signal.accounts` с конфигурацией для каждого аккаунта и необязательным `name`. [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) для общего шаблона.

## Режим внешнего демона (httpUrl)

Если вы хотите управлять `signal-cli` самостоятельно (медленные холодные старты JVM, инициализация контейнера или общие CPU), запустите демон отдельно и укажите его адрес в OpenClaw:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Это пропускает автозапуск и ожидание старта внутри OpenClaw. При медленном старте при автозапуске установите `channels.signal.startupTimeoutMs`.

## Контроль доступа (личные сообщения + группы)

DMs:

- По умолчанию: `channels.signal.dmPolicy = "pairing"`.
- Неизвестные отправители получают код сопряжения; сообщения игнорируются до одобрения (коды истекают через 1 час).
- Одобрение через:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Сопряжение — обмен токенами по умолчанию для личных сообщений Signal. Подробности: [Pairing](/channels/pairing)
- Отправители только с UUID (из `sourceUuid`) сохраняются как `uuid:<id>` в `channels.signal.allowFrom`.

Группы:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` управляет тем, кто может инициировать сообщения в группах, когда установлен `allowlist`.

## Как это работает (поведение)

- `signal-cli` работает как демон; шлюз читает события через SSE.
- Входящие сообщения нормализуются в общий конверт канала.
- Ответы всегда маршрутизируются обратно на тот же номер или в ту же группу.

## Медиа и ограничения

- Исходящий текст разбивается на части по `channels.signal.textChunkLimit` (по умолчанию 4000).
- Необязательная разбивка по новым строкам: установите `channels.signal.chunkMode="newline"`, чтобы сначала делить по пустым строкам (границы абзацев), а затем по длине.
- Поддерживаются вложения (base64 загружается из `signal-cli`).
- Лимит медиа по умолчанию: `channels.signal.mediaMaxMb` (по умолчанию 8).
- Используйте `channels.signal.ignoreAttachments`, чтобы пропустить загрузку медиа.
- Контекст истории группы использует `channels.signal.historyLimit` (или `channels.signal.accounts.*.historyLimit`), с откатом к `messages.groupChat.historyLimit`. Установите `0` для отключения (по умолчанию 50).

## Набирать и читать квитанции

- **Индикаторы набора**: OpenClaw отправляет сигналы набора через `signal-cli sendTyping` и обновляет их, пока формируется ответ.
- **Уведомления о прочтении**: когда `channels.signal.sendReadReceipts` установлено в true, OpenClaw пересылает уведомления о прочтении для разрешённых личных сообщений.
- Signal-cli не предоставляет уведомления о прочтении для групп.

## Реакции (инструмент сообщений)

- Используйте `message action=react` с `channel=signal`.
- Цели: отправитель E.164 или UUID (используйте `uuid:<id>` из вывода сопряжения; «голый» UUID тоже подходит).
- `messageId` — это временная метка Signal для сообщения, на которое вы реагируете.
- Реакции в группах требуют `targetAuthor` или `targetAuthorUuid`.

Примеры:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Конфиг:

- `channels.signal.actions.reactions`: включить/выключить действия реакций (по умолчанию true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` отключает реакции агента (инструмент сообщений `react` выдаст ошибку).
  - `minimal`/`extensive` включает реакции агента и задаёт уровень рекомендаций.
- Переопределения для аккаунта: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Цели доставки (CLI/cron)

- Личные сообщения: `signal:+15551234567` (или обычный E.164).
- Личные сообщения по UUID: `uuid:<id>` (или «голый» UUID).
- Группы: `signal:group:<groupId>`.
- Имена пользователей: `username:<name>` (если поддерживается вашим аккаунтом Signal).

## Устранение неполадок

Сначала запустите лестницу:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Затем при необходимости подтвердите состояние сопряжения для личных сообщений:

```bash
openclaw pairing list signal
```

Типичные сбои:

- Демон доступен, но ответов нет: проверьте настройки аккаунта/демона (`httpUrl`, `account`) и режим приёма.
- Личные сообщения игнорируются: отправитель ожидает одобрения сопряжения.
- Сообщения в группах игнорируются: доставка блокируется правилами для отправителей/упоминаний в группах.

Для схемы диагностики см.: [/channels/troubleshooting](/channels/troubleshooting).

## Справочник конфигурации (Signal)

Полная конфигурация: [Configuration](/gateway/configuration)

Параметры провайдера:

- `channels.signal.enabled`: включить/выключить запуск канала.
- `channels.signal.account`: E.164 для аккаунта бота.
- `channels.signal.cliPath`: путь к `signal-cli`.
- `channels.signal.httpUrl`: полный URL демона (переопределяет хост/порт).
- `channels.signal.httpHost`, `channels.signal.httpPort`: привязка демона (по умолчанию 127.0.0.1:8080).
- `channels.signal.autoStart`: автозапуск демона (по умолчанию true, если `httpUrl` не задан).
- `channels.signal.startupTimeoutMs`: тайм-аут ожидания старта в мс (предел 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: пропуск загрузки вложений.
- `channels.signal.ignoreStories`: игнорировать истории от демона.
- `channels.signal.sendReadReceipts`: пересылать уведомления о прочтении.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (по умолчанию: сопряжение).
- `channels.signal.allowFrom`: список разрешённых для личных сообщений (E.164 или `uuid:<id>`). `open` требует `"*"`. В Signal нет имён пользователей; используйте идентификаторы телефона/UUID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (по умолчанию: список разрешённых).
- `channels.signal.groupAllowFrom`: список разрешённых отправителей для групп.
- `channels.signal.historyLimit`: максимум сообщений группы, включаемых в контекст (0 отключает).
- `channels.signal.dmHistoryLimit`: лимит истории личных сообщений в пользовательских ходах. Переопределения для пользователя: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: размер исходящих чанков (символы).
- `channels.signal.chunkMode`: `length` (по умолчанию) или `newline` для разделения по пустым строкам (границы абзацев) перед разбиением по длине.
- `channels.signal.mediaMaxMb`: лимит медиа для входящих/исходящих (МБ).

Связанные глобальные параметры:

- `agents.list[].groupChat.mentionPatterns` (Signal не поддерживает нативные упоминания).
- `messages.groupChat.mentionPatterns` (глобальный откат).
- `messages.responsePrefix`.

