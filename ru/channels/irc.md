---
title: IRC
description: Подключите OpenClaw к каналам IRC и личным сообщениям.
---

Используйте IRC, если хотите работать с OpenClaw в классических каналах (`#room`) и личных сообщениях.
IRC поставляется как плагин-расширение, но настраивается в основном конфигурационном файле в разделе `channels.irc`.

## Быстрый старт

1. Включите конфигурацию IRC в `~/.openclaw/openclaw.json`.
2. Укажите как минимум:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Запустите/перезапустите gateway:

```bash
openclaw gateway run
```

## Настройки безопасности по умолчанию

- `channels.irc.dmPolicy` по умолчанию имеет значение `"pairing"`.
- `channels.irc.groupPolicy` по умолчанию имеет значение `"allowlist"`.
- При `groupPolicy="allowlist"` задайте `channels.irc.groups`, чтобы определить разрешённые каналы.
- Используйте TLS (`channels.irc.tls=true`), если только вы намеренно не допускаете передачу данных в открытом виде.

## Контроль доступа

Для каналов IRC существуют два отдельных «уровня» доступа:

1. **Доступ к каналу** (`groupPolicy` + `groups`): принимает ли бот сообщения из канала в принципе.
2. **Доступ отправителя** (`groupAllowFrom` / per-channel `groups["#channel"].allowFrom`): кому разрешено вызывать бота внутри этого канала.

Ключи конфигурации:

- Список разрешённых для DM (доступ отправителей в DM): `channels.irc.allowFrom`
- Список разрешённых отправителей в группах (доступ отправителей в канале): `channels.irc.groupAllowFrom`
- Настройки для конкретного канала (канал + отправитель + правила упоминаний): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` разрешает не настроенные каналы (**по умолчанию всё равно требуется упоминание**)

В записях allowlist можно использовать формы nick или `nick!user@host`.

### Распространённая ошибка: `allowFrom` предназначен для DM, а не для каналов

Если вы видите в логах что-то вроде:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…это означает, что отправителю не разрешено отправлять сообщения в **группу/канал**. Исправить это можно одним из способов:

- установить `channels.irc.groupAllowFrom` (глобально для всех каналов), или
- задать список разрешённых отправителей для конкретного канала: `channels.irc.groups["#channel"].allowFrom`

Пример (разрешить любому в `#tuirc-dev` обращаться к боту):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Срабатывание ответа (упоминания)

Даже если канал разрешён (через `groupPolicy` + `groups`) и отправитель разрешён, в групповых контекстах OpenClaw по умолчанию требует **упоминание**.

Это означает, что вы можете увидеть в логах что-то вроде `drop channel … (missing-mention)`, если сообщение не содержит шаблон упоминания, соответствующий боту.

Чтобы бот отвечал в IRC-канале **без необходимости упоминания**, отключите требование упоминания для этого канала:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Или чтобы разрешить **все** IRC-каналы (без allowlist для каждого канала) и при этом отвечать без упоминаний:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Примечание по безопасности (рекомендуется для публичных каналов)

Если вы разрешаете `allowFrom: ["*"]` в публичном канале, любой сможет отправлять запросы боту.
Чтобы снизить риски, ограничьте инструменты для этого канала.

### Одинаковые инструменты для всех в канале

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Разные инструменты для каждого отправителя (владелец получает больше возможностей)

Используйте `toolsBySender`, чтобы применить более строгую политику к `"*"` и более мягкую — к вашему нику:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Примечания:

- Ключи `toolsBySender` могут быть ником (например, `"eigen"`) или полной hostmask (`"eigen!~eigen@174.127.248.171"`) для более надёжного сопоставления личности.
- Применяется первая подходящая политика отправителя; `"*"` — это универсальная маска по умолчанию.

Подробнее о доступе групп и ограничении по упоминаниям (и о том, как они взаимодействуют) см.: [/channels/groups](/channels/groups).

## NickServ

Чтобы выполнить идентификацию через NickServ после подключения:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Необязательная однократная регистрация при подключении:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Отключите `register` после регистрации ника, чтобы избежать повторных попыток REGISTER.

## Переменные окружения

Учетная запись по умолчанию поддерживает:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (через запятую)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Устранение неполадок

- Если бот подключается, но никогда не отвечает в каналах, проверьте `channels.irc.groups` **и** не блокирует ли сообщения ограничение по упоминаниям (`missing-mention`). Если вы хотите, чтобы он отвечал без пингов, установите `requireMention:false` для канала.
- Если вход не выполняется, проверьте доступность ника и пароль сервера.
- Если TLS не работает в пользовательской сети, проверьте host/port и настройку сертификата.
