---
summary: "Использование OpenAI-совместимого API NVIDIA в OpenClaw"
read_when:
  - Вы хотите использовать модели NVIDIA в OpenClaw
  - Необходимо настроить NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA предоставляет OpenAI-совместимый API по адресу `https://integrate.api.nvidia.com/v1` для моделей Nemotron и NeMo. Аутентификация выполняется с помощью API-ключа из [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Настройка CLI

Экспортируйте ключ один раз, затем выполните onboarding и установите модель NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Если вы всё же передаёте `--token`, помните, что он сохраняется в истории shell и выводе `ps`; по возможности используйте переменную окружения.

## Фрагмент конфигурации

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Идентификаторы моделей

- `nvidia/llama-3.1-nemotron-70b-instruct` (по умолчанию)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Примечания

- OpenAI-совместимый эндпоинт `/v1`; используйте API-ключ из NVIDIA NGC.
- Провайдер автоматически включается при установке `NVIDIA_API_KEY`; используются статические значения по умолчанию (контекстное окно 131 072 токена, максимум 4 096 токенов).
