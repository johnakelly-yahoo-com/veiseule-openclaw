---
title: "Плагин Zalo Personal"
---

# Zalo Personal (плагин)

Поддержка Zalo Personal для OpenClaw через плагин с использованием `zca-cli` для автоматизации обычного пользовательского аккаунта Zalo.

> **Предупреждение:** Неофициальная автоматизация может привести к приостановке или блокировке аккаунта. Используйте на свой риск.

## Именование

Идентификатор канала — `zalouser`, чтобы явно указать, что это автоматизация **личного пользовательского аккаунта Zalo** (неофициальная). Мы оставляем `zalo` зарезервированным для возможной будущей официальной интеграции с API Zalo.

## Где это работает

Этот плагин работает **внутри процесса Gateway (шлюза)**.

Если вы используете удалённый Gateway (шлюз), установите и настройте плагин на **машине, на которой запущен Gateway (шлюз)**, затем перезапустите Gateway (шлюз).

## Установка

### Вариант A: установка из npm

```bash
openclaw plugins install @openclaw/zalouser
```

После этого перезапустите Gateway (шлюз).

### Вариант B: установка из локальной папки (dev)

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

После этого перезапустите Gateway (шлюз).

## Предварительное требование: zca-cli

На машине Gateway (шлюза) должен быть установлен `zca` на `PATH`:

```bash
zca --version
```

## Конфигурация

Конфигурация канала находится в `channels.zalouser` (а не в `plugins.entries.*`):

```json5
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## Инструмент агента

Имя инструмента: `zalouser`

Действия: `send`, `image`, `link`, `friends`, `groups`, `me`, `status`

