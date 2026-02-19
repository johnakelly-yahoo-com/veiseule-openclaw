---
summary: "Поддержка устаревшего iMessage через imsg (JSON-RPC через stdio). Для новых установок следует использовать BlueBubbles."
read_when:
  - Настройка поддержки iMessage
  - Отладка отправки/приёма iMessage
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
**Рекомендуется:** Для новых настроек iMessage используйте [BlueBubbles](/channels/bluebubbles).

Канал `imsg` является устаревшей интеграцией через внешний CLI и может быть удалён в будущих релизах. 
</Warning>

Статус: устаревшая интеграция через внешний CLI. Gateway (шлюз) запускает `imsg rpc` (JSON-RPC через stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Предпочтительный путь iMessage для новых установок.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Личные сообщения iMessage по умолчанию работают в режиме pairing.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Полный справочник полей iMessage.
  
</Card>
</CardGroup>

## Быстрая настройка (для начинающих)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Configure OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Approve first DM pairing (default dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Запросы на pairing истекают через 1 час.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">`channels.imessage.cliPath` может указывать на любую команду, проксирующую stdin/stdout (например, скрипт-обёртку, который подключается по SSH к другому Mac и запускает `imsg rpc`).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Рекомендуемая конфигурация при включённых вложениях:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Если `remoteHost` не задан, OpenClaw пытается определить его автоматически, разбирая SSH-команду в вашем скрипте-обёртке.
    ```

  
</Tab>
</Tabs>

## Требования и разрешения (macOS)

- macOS с выполненным входом в Messages.
- Full Disk Access для OpenClaw + `imsg` (доступ к БД Messages).
- Для отправки сообщений через Messages.app требуется разрешение Automation.

<Tip>
Разрешения предоставляются отдельно для каждого контекста процесса. Если gateway запущен в headless-режиме (LaunchAgent/SSH), выполните одноразовую интерактивную команду в том же контексте, чтобы вызвать запросы на предоставление разрешений:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Контроль доступа и маршрутизация

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` управляет личными сообщениями:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (по умолчанию: allowlist).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` управляет обработкой групп:


    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - Для личных сообщений используется прямая маршрутизация; для групп — групповая маршрутизация.
    - При значении по умолчанию `session.dmScope=main` личные сообщения iMessage объединяются в основную сессию агента.
    Личные сообщения используют основной сеанс агента; группы изолированы (`agent:<agentId>:imessage:group:<chat_id>`).<agentId>Группы:<chat_id>`).
    Детерминированная маршрутизация: ответы всегда возвращаются в iMessage.

    ```
    Если поток с несколькими участниками приходит с `is_group=false`, вы всё равно можете изолировать его, настроив `chat_id` с помощью `channels.imessage.groups` (см. «Псевдогрупповые потоки» ниже).
    ```

  
</Tab>
</Tabs>

## Сценарии развертывания

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Если вы хотите, чтобы бот отправлял сообщения от **отдельной учётной записи iMessage** (и не засорял ваши личные Messages), используйте отдельный Apple ID и отдельного пользователя macOS.

    ```
    Укажите `channels.imessage.accounts.bot.cliPath` на SSH-обёртку, которая запускает `imsg` от имени пользователя бота.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Типовая топология:


    ```
    Если Gateway (шлюз) работает на Linux-хосте/ВМ, а iMessage должен работать на Mac, Tailscale — самый простой мост: шлюз общается с Mac по tailnet, запускает `imsg` по SSH и копирует вложения обратно по SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Используйте SSH-ключи, чтобы и SSH, и SCP работали без интерактивного ввода.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Канал iMessage, работающий на основе `imsg` в macOS.

    ```
    Каждая учётная запись может переопределять такие поля, как `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` и настройки истории.
    ```

  
</Accordion>
</AccordionGroup>

## Медиафайлы, разбиение на части и цели доставки

<AccordionGroup>
  <Accordion title="Attachments and media">Загрузка медиа ограничена параметром `channels.imessage.mediaMaxMb` (по умолчанию 16).
</Accordion>

  <Accordion title="Outbound chunking">Исходящий текст разбивается на чанки по `channels.imessage.textChunkLimit` (по умолчанию 4000).
</Accordion>

  <Accordion title="Addressing formats">
    Предпочтительные явные цели:


    ```
    - `chat_id:123` (рекомендуется для стабильной маршрутизации)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Также поддерживаются цели по идентификатору получателя:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Запись конфига

По умолчанию iMessage разрешено записывать обновления конфига, инициированные `/config set|unset` (требуется `commands.config: true`).

Отключение:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Устранение неполадок

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Проверьте бинарный файл и поддержку RPC:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Если проверка сообщает, что RPC не поддерживается, обновите `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Checklist:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Подтверждение через:

    ```
    {
      channels: {
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15555550123"],
          groups: {
            "42": { requireMention: false },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Примечания:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    Повторно запустите в интерактивном GUI-терминале в том же пользовательском/сеансовом контексте и подтвердите запросы:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Запустите шлюз и подтвердите все запросы macOS (Automation + Full Disk Access).
    ```

  
</Accordion>
</AccordionGroup>

## Справочные указатели по конфигурации

- Справочник конфигурации (iMessage)
- Полная конфигурация: [Конфигурация](/gateway/configuration)
- Подробности: [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

