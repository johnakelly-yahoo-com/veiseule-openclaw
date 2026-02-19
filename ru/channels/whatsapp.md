---
summary: "Поддержка канала WhatsApp, контроль доступа, поведение доставки и эксплуатация"
read_when:
  - При работе с поведением канала WhatsApp/Web или маршрутизацией входящих
title: "WhatsApp"
---

# WhatsApp (веб‑канал)

Статус: только WhatsApp Web через Baileys. Сеанс(ы) принадлежат Gateway (шлюзу).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Политика DM по умолчанию — **сопряжение**, поэтому неизвестные отправители получают только код сопряжения, а их сообщение **не обрабатывается**.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Межканальная диагностика и сценарии восстановления.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Полные шаблоны и примеры конфигурации каналов.
  
</Card>
</CardGroup>

## Быстрая настройка (для начинающих)

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Для конкретной учётной записи:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Подтвердите командой: `openclaw pairing approve whatsapp <code>` (список — `openclaw pairing list whatsapp`).
```

    ```
    Коды истекают через 1 час; ожидающие запросы ограничены 3 на канал.
    ```

  
</Step>
</Steps>

<Note>
Отличный вариант, чтобы отделить личный WhatsApp — установите WhatsApp Business и зарегистрируйте там номер OpenClaw. (Метаданные канала и процесс онбординга оптимизированы для этой конфигурации, однако также поддерживаются конфигурации с личным номером.)
</Note>

## Шаблоны развертывания

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Это самый чистый операционный режим:

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Онбординг поддерживает режим личного номера и записывает базовую конфигурацию, удобную для self-chat:

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    Канал платформы обмена сообщениями основан на WhatsApp Web (`Baileys`) в текущей архитектуре каналов OpenClaw.

    ```
    В встроенном реестре chat-каналов нет отдельного канала обмена сообщениями Twilio WhatsApp.
    ```

  
</Accordion>
</AccordionGroup>

## Модель выполнения

- Gateway управляет сокетом WhatsApp и циклом переподключения.
- Исходящие сообщения требуют активного слушателя WhatsApp для целевого аккаунта.
- Чаты статусов/рассылок игнорируются.
- Для личных чатов применяются правила DM-сессий (`session.dmScope`; значение по умолчанию `main` объединяет DM в основную сессию агента).
- Группы сопоставляются с сессиями `agent:<agentId>:whatsapp:group:<jid>`.

## Контроль доступа и активация

<Tabs>
  <Tab title="DM policy">**Политика DM**: `channels.whatsapp.dmPolicy` управляет доступом к личным чатам (по умолчанию: `pairing`).

    ```
    Политика групп: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (по умолчанию `allowlist`).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Доступ в группах имеет два уровня:

    ```
    `channels.whatsapp.groupAllowFrom` (allowlist отправителей группы).
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Ответы в группах по умолчанию требуют упоминания.

    ```
    Распознавание упоминаний включает:
    
    - явные упоминания бота в WhatsApp
    - настроенные regex-шаблоны упоминаний (`agents.list[].groupChat.mentionPatterns`, резерв — `messages.groupChat.mentionPatterns`)
    - неявное определение ответа боту (отправитель ответа совпадает с идентификатором бота)
    
    Команда активации на уровне сессии:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` обновляет состояние сессии (не глобальную конфигурацию). Команда доступна только владельцу.
    ```

  
</Tab>
</Tabs>

## Поведение личного номера и self-chat

Когда привязанный собственный номер также присутствует в `allowFrom`, активируются защитные механизмы WhatsApp для self-chat:

- пропуск подтверждений прочтения для сообщений в self-chat
- игнорирование авто-триггера по mention-JID, который иначе вызывал бы уведомление самому себе
- Ответы в self‑chat по умолчанию используют `[{identity.name}]` при установленном значении (иначе `[openclaw]`),  
  если `messages.responsePrefix` не задан.

## Нормализация сообщений и контекст

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Входящие сообщения WhatsApp оборачиваются в общий inbound-конверт.

    ````
    Если существует цитируемый ответ, контекст добавляется в следующем виде:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Метаданные ответа также заполняются при наличии (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Входящие сообщения только с медиа используют плейсхолдеры:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Данные о местоположении и контактах нормализуются в текстовый контекст перед маршрутизацией.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Для групп необработанные сообщения могут буферизоваться и добавляться в контекст, когда бот в итоге активируется.

    ```
    Недавние _необработанные_ сообщения (по умолчанию 50) вставляются под:
    `[Chat messages since your last reply - for context]` (сообщения, уже находящиеся в сессии, не переинжектируются)
    ```

  
</Accordion>

  <Accordion title="Read receipts">По умолчанию gateway помечает входящие сообщения WhatsApp как прочитанные (синие галочки) после принятия.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Доставка, разбиение и медиа

<AccordionGroup>
  <Accordion title="Text chunking">Необязательное разбиение по переводам строк: установите `channels.whatsapp.chunkMode="newline"`, чтобы сначала делить по пустым строкам (границы абзацев), затем по длине.
</Accordion>

  <Accordion title="Outbound media behavior">
    - поддерживаются payload’ы изображений, видео, аудио (PTT voice-note) и документов
    - `audio/ogg` преобразуется в `audio/ogg; codecs=opus` для совместимости с voice-note
    - поддерживается воспроизведение анимированных GIF через `gifPlayback: true` при отправке видео
    - подписи применяются к первому медиафайлу при отправке ответа с несколькими медиа
    - источник медиа может быть HTTP(S), `file://` или локальные пути
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - лимит сохранения входящих медиа: `channels.whatsapp.mediaMaxMb` (по умолчанию `50`)
    - лимит исходящих медиа для авто-ответов: `agents.defaults.mediaMaxMb` (по умолчанию `5MB`)
    - изображения автоматически оптимизируются (изменение размера/подбор качества) для соответствия лимитам
    - при ошибке отправки медиа вместо тихого пропуска ответа отправляется текстовое предупреждение для первого элемента
  
</Accordion>
</AccordionGroup>

## Реакции-подтверждения

`channels.whatsapp.ackReaction` (авто‑реакция при получении: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

Примечания:

- отправляются сразу после принятия входящего сообщения (до ответа)
- ошибки логируются, но не блокируют обычную доставку ответа
- в групповом режиме `mentions` реакция отправляется только при срабатывании по упоминанию; активация группы `always` действует как обход этой проверки
- WhatsApp игнорирует `messages.ackReaction`; используйте `channels.whatsapp.ackReaction`.

## Несколько аккаунтов и учетные данные

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (настройки для аккаунта + необязательный `authDir`).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - текущий путь аутентификации: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - резервный файл: `creds.json.bak`
    - устаревшая аутентификация по умолчанию в `~/.openclaw/credentials/` по-прежнему распознается/мигрируется для сценариев с аккаунтом по умолчанию
  
</Accordion>

  <Accordion title="Logout behavior">Вход для нескольких аккаунтов: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>Выход: `openclaw channels logout` (или `--account <id>`) удаляет состояние аутентификации WhatsApp (но сохраняет общий `oauth.json`).

    ```
    В устаревших каталогах аутентификации `oauth.json` сохраняется, а файлы аутентификации Baileys удаляются.
    ```

  
</Accordion>
</AccordionGroup>

## Инструменты, действия и запись конфигурации

- Инструмент: `whatsapp` с действием `react` (`chatJid`, `messageId`, `emoji`, необязательный `remove`).
- Ограничения действий:
  - `channels.whatsapp.actions.reactions` (ограничение реакций инструмента WhatsApp).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Устранение неполадок (кратко)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Симптом: `channels status` показывает `linked: false` или предупреждает «Not linked».

    ```
    {
      channels: { whatsapp: { sendReadReceipts: false } },
    }
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">    Симптом: связанный аккаунт с повторяющимися отключениями или попытками переподключения.

    ```
    Исправление: `openclaw doctor` (или перезапустите gateway). Если проблема сохраняется, перепривяжите через `channels login` и проверьте `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">    Исходящие отправки завершаются с ошибкой, если для целевого аккаунта нет активного listener’а gateway.

    ```
    Убедитесь, что gateway запущен и аккаунт связан.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">    Проверьте в следующем порядке:

    ```
    `channels.whatsapp.groups` (allowlist групп + умолчания для gating по упоминаниям; используйте `"*"`, чтобы разрешить всё)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Среда выполнения WhatsApp gateway должна использовать Node. Bun помечен как несовместимый для стабильной работы WhatsApp/Telegram gateway.
  
</Accordion>
</AccordionGroup>

## Ссылки на справочник по конфигурации

Основной справочник:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Ключевые поля WhatsApp:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>.enabled`, `accounts.<id>.authDir`, переопределения на уровне аккаунта
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- поведение сессии: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>`messages.groupChat.historyLimit\`

## Связанные материалы

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

