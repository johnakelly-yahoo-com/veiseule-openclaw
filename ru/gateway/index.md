---
summary: "Рунбук для сервиса Gateway, его жизненного цикла и операций"
read_when:
  - При запуске или отладке процесса Gateway
title: "Рунбук Gateway"
---

# Рунбук сервиса Gateway

Используйте эту страницу для запуска в первый день (day-1) и эксплуатации во второй день (day-2) сервиса Gateway.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Диагностика от симптомов с точными последовательностями команд и сигнатурами логов.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Руководство по настройке, ориентированное на задачи, + полный справочник по конфигурации.
  
</Card>
</CardGroup>

## Локальный запуск за 5 минут

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Базовое состояние: `Runtime: running` и `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Перезагрузка конфигурации Gateway отслеживает путь к активному файлу конфигурации (определяется из значений профиля/состояния по умолчанию или из `OPENCLAW_CONFIG_PATH`, если задан).
Режим по умолчанию: `gateway.reload.mode="hybrid"` (безопасные изменения применяются на лету, критические — с перезапуском).
</Note>

## Модель выполнения

- Один постоянно работающий процесс для маршрутизации, control plane и подключений каналов.
- Мультиплексирование на одном порту.
  - WebSocket control/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - Control UI и хуки
- Режим привязки по умолчанию: `loopback`.
- Аутентификация Gateway по умолчанию обязательна: задайте `gateway.auth.token` (или `OPENCLAW_GATEWAY_TOKEN`) либо `gateway.auth.password`.

### Приоритет порта и привязки

| Параметр       | Порядок определения                                                                                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Порт Gateway   | Приоритет портов: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > значение по умолчанию `18789`. |
| Режим привязки | CLI/override → `gateway.bind` → `loopback`                                                                                             |

### Режимы горячей перезагрузки

| Отключается с помощью `gateway.reload.mode="off"`. | Поведение                                                         |
| ------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `off`                                                              | Без перезагрузки конфигурации                                     |
| `hot`                                                              | Применять только изменения, безопасные для hot-режима             |
| `restart`                                                          | Перезапуск при изменениях, требующих перезапуска                  |
| `hybrid` (по умолчанию)                         | Горячее применение, когда безопасно, перезапуск при необходимости |

## Набор команд оператора

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Удалённый доступ

Предпочтительно Tailscale/VPN; в противном случае — SSH-туннель:
Резервный вариант: SSH-туннель.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Затем клиенты подключаются к `ws://127.0.0.1:18789` через туннель.

<Warning>
Если настроен токен, клиенты должны включать его в `connect.params.auth.token` даже через туннель.
</Warning>

См.: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Контроль и жизненный цикл службы

Используйте supervised-запуск для надежности уровня production.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Для перезапуска используйте `openclaw gateway restart` (или `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
```

Метки LaunchAgent: `ai.openclaw.gateway` (по умолчанию) или `ai.openclaw.<profile>` при запуске именованного профиля. `openclaw doctor` проверяет и устраняет расхождения конфигурации службы.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Для сохранения работы после выхода из системы включите lingering:

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Используйте system unit для многопользовательских/постоянно работающих хостов.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Несколько Gateway (на одном хосте)

Обычно не требуется: один Gateway может обслуживать несколько каналов сообщений и агентов.
Используйте несколько Gateway только для резервирования или строгой изоляции (например, rescue bot).

Checklist per instance:

- уникальный `gateway.port`
- уникальный `OPENCLAW_CONFIG_PATH`
- уникальный `OPENCLAW_STATE_DIR`
- уникальный `agents.defaults.workspace`

Пример:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

См. [Multiple gateways](/gateway/multiple-gateways).

### Профиль Dev (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

По умолчанию включают изолированное состояние/конфигурацию и базовый порт gateway `19001`.

## Протокол (взгляд оператора)

- Клиенты должны переподключиться.
- Gateway возвращает снимок `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy).
- Запросы: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Типичные события: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Запуски агента двухэтапные:

1. Немедленное подтверждение принятия (`status:"accepted"`)
2. Ответы `agent` двухэтапные: сначала `res` ack `{runId,status:"accepted"}`, затем финальный `res` `{runId,status:"ok"|"error",summary}` после завершения выполнения; потоковый вывод приходит как `event:"agent"`.

Полная документация: [Протокол Gateway](/gateway/protocol) и [Протокол Bridge (устаревший)](/gateway/bridge-protocol).

## Операционные проверки

### Проверка доступности (liveness)

- Откройте WS и отправьте `connect`.
- Ожидайте ответ `hello-ok` со снимком состояния.

### Готовность (readiness)

```bash
`openclaw gateway health|status` — запросить health/status через WS Gateway.
```

### Восстановление после пропуска последовательности

События не воспроизводятся повторно. При пропуске последовательности обновите состояние (`health`, `system-presence`) перед продолжением.

## Типичные сигнатуры сбоев

| Сигнатура                                                      | Вероятная проблема                                     |
| -------------------------------------------------------------- | ------------------------------------------------------ |
| `refusing to bind gateway ... without auth`                    | Привязка не к loopback без токена/пароля               |
| `another gateway instance is already listening` / `EADDRINUSE` | Конфликт портов                                        |
| `Gateway start blocked: set gateway.mode=local`                | Конфигурация установлена в удалённый режим             |
| `unauthorized` при подключении                                 | Несоответствие аутентификации между клиентом и Gateway |

Для полной диагностики используйте [Gateway Troubleshooting](/gateway/troubleshooting).

## Гарантии безопасности

- Нет резервного перехода к прямым подключениям Baileys; если Gateway недоступен, отправки немедленно завершаются ошибкой.
- Некорректные первые фреймы подключения или повреждённый JSON отклоняются, и сокет закрывается.
- Корректное завершение: перед закрытием отправляется событие `shutdown`; клиенты должны обрабатывать закрытие и переподключение.

---

Связано:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

