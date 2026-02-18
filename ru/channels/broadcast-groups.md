---
status: experimental
title: "Группы рассылки"
---

# Группы рассылки

**Статус:** Экспериментально  
**Версия:** Добавлено в 2026.1.9

## Обзор

Группы рассылки позволяют нескольким агентам одновременно обрабатывать и отвечать на одно и то же сообщение. Это позволяет создавать специализированные команды агентов, которые работают вместе в одной группе WhatsApp или в личных сообщениях — при этом используется один номер телефона.

Текущий охват: **только WhatsApp** (веб-канал).

Группы рассылки оцениваются после списков разрешённых каналов и правил активации групп. В группах WhatsApp это означает, что рассылка происходит тогда, когда OpenClaw обычно отвечает (например, при упоминании — в зависимости от настроек группы).

## Сценарии использования

### 1. Специализированные команды агентов

Развёртывайте нескольких агентов с атомарными, сфокусированными обязанностями:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Каждый агент обрабатывает одно и то же сообщение и предоставляет свою специализированную точку зрения.

### 2. Многоязычная поддержка

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Процессы контроля качества

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Автоматизация задач

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Конфигурация

### Базовая настройка

Добавьте верхнеуровневый раздел `broadcast` (рядом с `bindings`). Ключи — это peer id WhatsApp:

- групповые чаты: JID группы (например, `120363403215116621@g.us`)
- DMs: E.164 номер телефона (например, `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Результат:** Когда OpenClaw должен ответить в этом чате, он запустит всех трёх агентов.

### Стратегия обработки

Управляйте тем, как агенты обрабатывают сообщения:

#### Параллельно (по умолчанию)

Все агенты обрабатывают одновременно:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Последовательно

Агенты обрабатывают по порядку (каждый ждёт завершения предыдущего):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Полный пример

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Как это работает

### Поток сообщений

1. **Входящее сообщение** поступает в группу WhatsApp
2. **Проверка рассылки**: система проверяет, есть ли peer ID в `broadcast`
3. **Если в списке рассылки**:
   - Все перечисленные агенты обрабатывают сообщение
   - У каждого агента свой ключ сеанса и изолированный контекст
   - Агенты обрабатывают параллельно (по умолчанию) или последовательно
4. **Если не в списке рассылки**:
   - Применяется обычная маршрутизация (первое совпадающее связывание)

Примечание: группы рассылки не обходят списки разрешённых каналов или правила активации групп (упоминания/команды и т. п.). Они лишь меняют _какие агенты запускаются_, когда сообщение допускается к обработке.

### Изоляция сеансов

Каждый агент в группе рассылки поддерживает полностью отдельные:

- **Ключи сеансов** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Историю диалога** (агент не видит сообщения других агентов)
- **Рабочее пространство** (отдельные sandbox при настройке)
- **Доступ к инструментам** (разные списки разрешений/запретов)
- **Память/контекст** (отдельные IDENTITY.md, SOUL.md и т. д.)
- **Буфер контекста группы** (последние сообщения группы, используемые для контекста) — общий для peer, поэтому все агенты рассылки видят один и тот же контекст при срабатывании

Это позволяет каждому агенту иметь:

- Разные личности
- Разный доступ к инструментам (например, только чтение vs. чтение-запись)
- Разные модели (например, opus vs. sonnet)
- Разные установленные навыки

### Пример: изолированные сеансы

В группе `120363403215116621@g.us` с агентами `["alfred", "baerbel"]`:

**Контекст Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Контекст Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Лучшие практики

### 1. Сохраняйте фокус агентов

Проектируйте каждого агента с одной, чёткой ответственностью:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Хорошо:** у каждого агента одна задача  
❌ **Плохо:** один универсальный агент «dev-helper»

### 2. Используйте описательные имена

Делайте понятным, что делает каждый агент:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Настраивайте разный доступ к инструментам

Давайте агентам только те инструменты, которые им нужны:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. Следите за производительностью

При большом числе агентов учитывайте:

- Использование `"strategy": "parallel"` (по умолчанию) для скорости
- Ограничение групп рассылки 5–10 агентами
- Применение более быстрых моделей для простых агентов

### 5. Корректно обрабатывайте сбои

Агенты падают независимо. Ошибка одного агента не блокирует других:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Совместимость

### Провайдеры

Группы рассылки в настоящее время работают с:

- ✅ WhatsApp (реализовано)
- 🚧 Telegram (планируется)
- 🚧 Discord (планируется)
- 🚧 Slack (планируется)

### Маршрутизация

Группы рассылки работают вместе с существующей маршрутизацией:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: отвечает только alfred (обычная маршрутизация)
- `GROUP_B`: отвечают agent1 И agent2 (рассылка)

**Приоритет:** `broadcast` имеет приоритет над `bindings`.

## Устранение неполадок

### Агенты не отвечают

**Проверьте:**

1. Идентификаторы агентов существуют в `agents.list`
2. Формат peer ID корректен (например, `120363403215116621@g.us`)
3. Агенты не находятся в списках запрета

**Отладка:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Отвечает только один агент

**Причина:** peer ID может находиться в `bindings`, но отсутствовать в `broadcast`.

**Исправление:** добавьте в конфиг рассылки или удалите из привязок.

### Проблемы с производительностью

**Если медленно при большом числе агентов:**

- Уменьшите число агентов на группу
- Используйте более лёгкие модели (sonnet вместо opus)
- Проверьте время запуска sandbox

## Примеры

### Пример 1: Команда код-ревью

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Пользователь отправляет:** фрагмент кода  
**Ответы:**

- code-formatter: «Исправил отступы и добавил аннотации типов»
- security-scanner: «⚠️ Уязвимость SQL-инъекции в строке 12»
- test-coverage: «Покрытие 45%, отсутствуют тесты для сценариев ошибок»
- docs-checker: «Отсутствует docstring для функции `process_data`»

### Пример 2: Многоязычная поддержка

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Справочник API

### Схема конфига

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Поля

- `strategy` (необязательно): способ обработки агентов
  - `"parallel"` (по умолчанию): все агенты обрабатывают одновременно
  - `"sequential"`: агенты обрабатывают в порядке массива
- `[peerId]`: JID группы WhatsApp, номер E.164 или другой peer ID
  - Значение: массив идентификаторов агентов, которые должны обрабатывать сообщения

## Ограничения

1. **Макс. число агентов:** жёсткого лимита нет, но 10+ агентов могут работать медленно
2. **Общий контекст:** агенты не видят ответы друг друга (по замыслу)
3. **Порядок сообщений:** при параллельной обработке ответы могут приходить в любом порядке
4. **Лимиты:** все агенты учитываются в лимитах WhatsApp

## Будущие улучшения

Запланированные функции:

- [ ] Режим общего контекста (агенты видят ответы друг друга)
- [ ] Координация агентов (агенты могут сигнализировать друг другу)
- [ ] Динамический выбор агентов (выбор агентов на основе содержимого сообщения)
- [ ] Приоритеты агентов (некоторые агенты отвечают раньше других)

## См. также

- [Конфигурация нескольких агентов](/tools/multi-agent-sandbox-tools)
- [Конфигурация маршрутизации](/channels/channel-routing)
- [Управление сеансами](/concepts/sessions)

