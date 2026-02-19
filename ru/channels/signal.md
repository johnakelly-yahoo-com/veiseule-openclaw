---
summary: "Поддержка Signal через signal-cli (JSON-RPC + SSE), настройка и модель номеров"
read_when:
  - Настройка поддержки Signal
  - Отладка отправки/получения сообщений Signal
title: "Signal"
---

# Signal (signal-cli)

Статус: внешняя интеграция через CLI. Gateway (шлюз) взаимодействует с `signal-cli` по HTTP JSON-RPC + SSE.

## Предварительные требования

- OpenClaw установлен на вашем сервере (процесс для Linux ниже протестирован на Ubuntu 24).
- `signal-cli` доступен на хосте, где запущен gateway.
- Номер телефона, способный получить одно проверочное SMS (для пути регистрации через SMS).
- Доступ к браузеру для прохождения captcha Signal (`signalcaptchas.org`) во время регистрации.

## Быстрая настройка (для начинающих)

1. Используйте **отдельный номер Signal** для бота (рекомендуется).
2. Установите `signal-cli` (требуется Java).
3. Выберите один из вариантов настройки:
   - `signal-cli link -n "OpenClaw"`
   - **Путь B (регистрация через SMS):** зарегистрируйте выделенный номер с captcha + проверкой по SMS.
4. Настройте OpenClaw и запустите шлюз.
5. Отправьте первое личное сообщение и подтвердите сопряжение (`openclaw pairing approve signal <CODE>`).

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

Справка по полям:

| Поле                                        | Описание                                                                                 |
| ------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `account`                                   | Номер телефона бота в формате E.164 (`+15551234567`)  |
| Настройка (быстрый путь) | Набирать и читать квитанции                                                              |
| `dmPolicy`                                  | Политика доступа к личным сообщениям (рекомендуется `pairing`)        |
| `allowFrom`                                 | Номера телефонов или значения `uuid:&lt;id&gt;`, которым разрешено отправлять личные сообщения |

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

## Путь настройки A: привязать существующую учётную запись Signal (QR)

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

## Путь настройки B: зарегистрировать отдельный номер бота (SMS, Linux)

Используйте этот вариант, если вам нужен отдельный номер для бота вместо привязки существующей учётной записи приложения Signal.

1. Получите номер, который может принимать SMS (или голосовую проверку для стационарных телефонов).
   - Используйте отдельный номер бота, чтобы избежать конфликтов учётной записи/сессии.
2. Установите `signal-cli` на хосте gateway:

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Если вы используете сборку JVM (`signal-cli-${VERSION}.tar.gz`), сначала установите JRE 25+.
Поддерживайте `signal-cli` в актуальном состоянии; в upstream отмечают, что старые версии могут перестать работать при изменении API серверов Signal.

3. Зарегистрируйте и подтвердите номер:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Если требуется captcha:

1. Откройте `https://signalcaptchas.org/registration/generate.html`.
2. Пройдите captcha, скопируйте целевую ссылку `signalcaptcha://...` из «Open Signal».
3. По возможности выполняйте команду с того же внешнего IP-адреса, что и в сессии браузера.
4. Сразу же снова запустите регистрацию (токены captcha быстро истекают):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Настройте OpenClaw, перезапустите gateway, проверьте канал:

```bash
# If you run the gateway as a user systemd service:
systemctl --user restart openclaw-gateway

# Then verify:
openclaw doctor
openclaw channels status --probe
```

5. Выполните привязку отправителя личных сообщений:
   - Отправьте любое сообщение на номер бота.
   - Подтвердите код на сервере: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Сохраните номер бота как контакт в телефоне, чтобы избежать отображения «Unknown contact».

Важно: регистрация учётной записи номера телефона через `signal-cli` может деавторизовать основную сессию приложения Signal для этого номера. Предпочтительно использовать отдельный номер бота или режим привязки по QR, если нужно сохранить текущую настройку приложения на телефоне.

Ссылки на upstream:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Процесс с captcha: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Процесс привязки: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

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
- Ошибки проверки конфигурации после изменений: выполните `openclaw doctor --fix`.
- Signal отсутствует в диагностике: убедитесь, что `channels.signal.enabled: true`.

Дополнительные проверки:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Для схемы диагностики см.: [/channels/troubleshooting](/channels/troubleshooting).

## Примечания по безопасности

- `signal-cli` хранит ключи учётной записи локально (обычно `~/.local/share/signal-cli/data/`).
- Сделайте резервную копию состояния аккаунта Signal перед миграцией или пересборкой сервера.
- Свяжите устройство бота и запустите демон:
- SMS-подтверждение требуется только для регистрации или восстановления, однако потеря контроля над номером/аккаунтом может усложнить повторную регистрацию.

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

