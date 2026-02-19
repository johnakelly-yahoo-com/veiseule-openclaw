---
summary: "Запуск OpenClaw с Ollama (локальная среда выполнения LLM)"
read_when:
  - Вы хотите запускать OpenClaw с локальными моделями через Ollama
  - Вам нужна информация по настройке и конфигурации Ollama
title: "Ollama"
---

# Ollama

Ollama — это локальная среда выполнения LLM, которая упрощает запуск моделей с открытым исходным кодом на вашем компьютере. OpenClaw интегрируется с OpenAI-совместимым API Ollama и может **автоматически обнаруживать модели с поддержкой инструментов**, если вы включили `OLLAMA_API_KEY` (или профиль аутентификации) и не определили явную запись `models.providers.ollama`.

## Быстрый старт

1. Установите Ollama: [https://ollama.ai](https://ollama.ai)

2. Загрузите модель:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Включите Ollama для OpenClaw (подойдёт любое значение; Ollama не требует реального ключа):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Используйте модели Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Обнаружение моделей (неявный провайдер)

Когда вы задаёте `OLLAMA_API_KEY` (или профиль аутентификации) и **не** определяете `models.providers.ollama`, OpenClaw обнаруживает модели из локального экземпляра Ollama по адресу `http://127.0.0.1:11434`:

- Выполняет запросы к `/api/tags` и `/api/show`
- Оставляет только модели, которые сообщают о наличии возможности `tools`
- Помечает `reasoning`, когда модель сообщает `thinking`
- Считывает `contextWindow` из `model_info["<arch>.context_length"]`, если доступно
- Устанавливает `maxTokens` равным 10× размеру контекстного окна
- Устанавливает все стоимости в `0`

Это позволяет избежать ручного описания моделей, сохраняя каталог согласованным с возможностями Ollama.

Чтобы посмотреть, какие модели доступны:

```bash
ollama list
openclaw models list
```

Чтобы добавить новую модель, просто загрузите её через Ollama:

```bash
ollama pull mistral
```

Новая модель будет автоматически обнаружена и станет доступной для использования.

Если вы явно зададите `models.providers.ollama`, автоматическое обнаружение будет отключено, и вам потребуется определить модели вручную (см. ниже).

## Конфигурация

### Базовая настройка (неявное обнаружение)

Самый простой способ включить Ollama — использовать переменную окружения:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Явная настройка (ручное описание моделей)

Используйте явную конфигурацию, если:

- Ollama запущена на другом хосте или порту.
- Вы хотите принудительно задать размеры контекстных окон или список моделей.
- Вы хотите включить модели, которые не сообщают о поддержке инструментов.

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Если задан `OLLAMA_API_KEY`, вы можете опустить `apiKey` в записи провайдера, и OpenClaw заполнит его для проверок доступности.

### Пользовательский базовый URL (явная конфигурация)

Если Ollama запущена на другом хосте или порту (явная конфигурация отключает автоматическое обнаружение, поэтому модели нужно определить вручную):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### Выбор модели

После настройки все ваши модели Ollama будут доступны:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Дополнительно

### Модели с поддержкой рассуждений

OpenClaw помечает модели как поддерживающие рассуждения, когда Ollama сообщает `thinking` в `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Стоимость моделей

Ollama бесплатна и работает локально, поэтому для всех моделей стоимость установлена в $0.

### Настройка потоковой передачи

Интеграция Ollama в OpenClaw по умолчанию использует **нативный API Ollama** (`/api/chat`), который полностью поддерживает одновременную потоковую передачу и вызов инструментов. Специальная настройка не требуется.

#### Устаревший OpenAI-совместимый режим

Если вам нужно использовать OpenAI-совместимый эндпоинт (например, за прокси, который поддерживает только формат OpenAI), явно укажите `api: "openai-completions"`:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Примечание: OpenAI-совместимый эндпоинт может не поддерживать одновременную потоковую передачу и вызов инструментов. Явно задайте `streaming: false` для моделей Ollama (см. [Настройка потоковой передачи](#настройка-потоковой-передачи))

### Контекстные окна

Для автоматически обнаруженных моделей OpenClaw использует размер контекстного окна, сообщаемый Ollama, если он доступен; в противном случае используется значение по умолчанию `8192`. Вы можете переопределить `contextWindow` и `maxTokens` в явной конфигурации провайдера.

## Устранение неполадок

### Ollama не обнаружена

Убедитесь, что Ollama запущена, и что вы задали `OLLAMA_API_KEY` (или профиль аутентификации), а также что вы **не** определили явную запись `models.providers.ollama`:

```bash
ollama serve
```

Также проверьте, что API доступен:

```bash
curl http://localhost:11434/api/tags
```

### Нет доступных моделей

OpenClaw автоматически обнаруживает только модели, которые сообщают о поддержке инструментов. Если вашей модели нет в списке, вы можете:

- Загрузить модель с поддержкой инструментов, или
- Определить модель явно в `models.providers.ollama`.

Чтобы добавить модели:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Соединение отклонено

Проверьте, что Ollama запущена на правильном порту:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## См. также

- [Model Providers](/concepts/model-providers) — обзор всех провайдеров
- [Model Selection](/concepts/models) — как выбирать модели
- [Configuration](/gateway/configuration) — полная справка по конфигурации

