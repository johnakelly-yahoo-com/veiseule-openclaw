---
summary: "Входящие вебхуки для пробуждения и изолированных запусков агента"
read_when:
  - Добавление или изменение конечных точек вебхуков
  - Подключение внешних систем к OpenClaw
title: "Вебхуки"
---

# Вебхуки

Gateway (шлюз) может предоставлять небольшую HTTP‑конечную точку вебхука для внешних триггеров.

## Включение

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

Примечания:

- `hooks.token` требуется, когда `hooks.enabled=true`.
- `hooks.path` по умолчанию — `/hooks`.

## Аутентификация

Каждый запрос должен включать токен хука. Предпочтительно использовать заголовки:

- `Authorization: Bearer <token>` (рекомендуется)
- `x-openclaw-token: <token>`
- Токены в query-строке отклоняются (`?token=...` возвращает `400`).

## Конечные точки

### `POST /hooks/wake`

Payload:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **обязательно** (string): Описание события (например, «Получено новое письмо»).
- `mode` необязательно (`now` | `next-heartbeat`): Нужно ли запускать немедленный сигнал keepalive (по умолчанию `now`) или ждать следующей периодической проверки.

Эффект:

- Помещает системное событие в очередь для **основного** сеанса
- Если `mode=now`, запускает немедленный сигнал keepalive

### `POST /hooks/agent`

Payload:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **обязательно** (string): Промпт или сообщение для обработки агентом.
- `name` необязательно (string): Человекочитаемое имя хука (например, «GitHub»), используется как префикс в сводках сеансов.
- `agentId` необязателен (string): направляет этот hook к конкретному агенту. Неизвестные ID приводят к использованию агента по умолчанию. Если параметр задан, hook выполняется с использованием workspace и конфигурации выбранного агента.
- `sessionKey` необязательно (string): Ключ для идентификации сеанса агента. По умолчанию это поле отклоняется, если не установлено `hooks.allowRequestSessionKey=true`.
- `wakeMode` необязательно (`now` | `next-heartbeat`): Нужно ли запускать немедленный сигнал keepalive (по умолчанию `now`) или ждать следующей периодической проверки.
- `deliver` необязательно (boolean): Если `true`, ответ агента будет отправлен в канал сообщений. По умолчанию — `true`. Ответы, которые являются лишь подтверждениями keepalive, автоматически пропускаются.
- `channel` необязательно (string): Канал сообщений для доставки. Один из: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (плагин), `signal`, `imessage`, `msteams`. По умолчанию — `last`.
- `to` необязательно (string): Идентификатор получателя для канала (например, номер телефона для WhatsApp/Signal, ID чата для Telegram, ID канала для Discord/Slack/Mattermost (плагин), ID беседы для MS Teams). По умолчанию используется последний получатель в основном сеансе.
- `model` необязательно (string): Переопределение модели (например, `anthropic/claude-3-5-sonnet` или алиас). Должно входить в список разрешённых моделей, если он ограничен.
- `thinking` необязательно (string): Переопределение уровня «мышления» (например, `low`, `medium`, `high`).
- `timeoutSeconds` необязательно (number): Максимальная длительность запуска агента в секундах.

Эффект:

- Запускает **изолированный** ход агента (собственный ключ сеанса)
- Всегда публикует сводку в **основной** сеанс
- Если `wakeMode=now`, запускает немедленный сигнал keepalive

## Политика session key (breaking change)

Переопределения `sessionKey` в payload `/hooks/agent` по умолчанию отключены.

- Рекомендуется: установите фиксированный `hooks.defaultSessionKey` и отключите переопределения в запросах.
- Необязательно: разрешайте переопределения в запросах только при необходимости и ограничивайте префиксы.

Рекомендуемая конфигурация:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

Конфигурация для совместимости (устаревшее поведение):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // настоятельно рекомендуется
  },
}
```

### `POST /hooks/<name>` (с маппингом)

Пользовательские имена хуков разрешаются через `hooks.mappings` (см. конфигурацию). Маппинг может
преобразовывать произвольные полезные нагрузки в действия `wake` или `agent` с опциональными шаблонами или
преобразованиями кода.

Параметры маппинга (кратко):

- `hooks.presets: ["gmail"]` включает встроенный маппинг Gmail.
- `hooks.mappings` позволяет определить `match`, `action` и шаблоны в конфиге.
- `hooks.transformsDir` + `transform.module` загружает JS/TS‑модуль для пользовательской логики.
  - `hooks.transformsDir` (если задан) должен находиться в корневом каталоге transforms в директории конфигурации OpenClaw (обычно `~/.openclaw/hooks/transforms`).
  - `transform.module` должен разрешаться в пределах фактической директории transforms (пути с обходом/выходом за пределы отклоняются).
- Используйте `match.source`, чтобы сохранить универсальную конечную точку приёма (маршрутизация по полезной нагрузке).
- TS‑преобразования требуют TS‑загрузчик (например, `bun` или `tsx`) либо заранее скомпилированный `.js` во время выполнения.
- Установите `deliver: true` + `channel`/`to` в маппингах, чтобы направлять ответы на чат‑поверхность
  (`channel` по умолчанию — `last` и с откатом на WhatsApp).
- `agentId` направляет хук к конкретному агенту; неизвестные идентификаторы приводят к использованию агента по умолчанию.
- `hooks.allowedAgentIds` ограничивает явную маршрутизацию по `agentId`. Опустите это поле (или укажите `*`), чтобы разрешить любого агента. Установите `[]`, чтобы запретить явную маршрутизацию по `agentId`.
- `hooks.defaultSessionKey` задаёт сессию по умолчанию для запусков агента через хук, если явный ключ не указан.
- `hooks.allowRequestSessionKey` определяет, могут ли payload’ы `/hooks/agent` задавать `sessionKey` (по умолчанию: `false`).
- `hooks.allowedSessionKeyPrefixes` при необходимости ограничивает явные значения `sessionKey` из payload’ов запросов и сопоставлений.
- `allowUnsafeExternalContent: true` отключает внешний контур безопасности контента для этого хука
  (опасно; только для доверенных внутренних источников).
- `openclaw webhooks gmail setup` записывает конфиг `hooks.gmail` для `openclaw webhooks gmail run`.
  Полный поток наблюдения Gmail см. в [Gmail Pub/Sub](/automation/gmail-pubsub).

## Ответы

- `200` для `/hooks/wake`
- `202` для `/hooks/agent` (асинхронный запуск начат)
- `401` при ошибке аутентификации
- `429` после повторных неудачных попыток аутентификации от одного клиента (проверьте `Retry-After`)
- `400` при недопустимой полезной нагрузке
- `413` при слишком больших полезных нагрузках

## Примеры

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### Использование другой модели

Добавьте `model` в полезную нагрузку агента (или маппинг), чтобы переопределить модель для этого запуска:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

Если вы принудительно применяете `agents.defaults.models`, убедитесь, что модель переопределения включена в этот список.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## Безопасность

- Держите конечные точки хуков за loopback, tailnet или доверенным обратным прокси.
- Используйте отдельный токен хука; не переиспользуйте токены аутентификации Gateway (шлюза).
- Повторные неудачные попытки аутентификации ограничиваются по частоте для каждого адреса клиента, чтобы замедлить brute-force атаки.
- Если вы используете маршрутизацию с несколькими агентами, установите `hooks.allowedAgentIds`, чтобы ограничить явный выбор `agentId`.
- Сохраняйте `hooks.allowRequestSessionKey=false`, если вам не требуется выбор сессии вызывающей стороной.
- Если вы включаете `sessionKey` в запросах, ограничьте `hooks.allowedSessionKeyPrefixes` (например, `["hook:"]`).
- Избегайте включения чувствительных необработанных полезных нагрузок в логи вебхуков.
- Полезные нагрузки хуков по умолчанию считаются недоверенными и оборачиваются границами безопасности.
  Если необходимо отключить это для конкретного хука, установите `allowUnsafeExternalContent: true`
  в маппинге этого хука (опасно).
