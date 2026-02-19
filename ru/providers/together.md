---
summary: "Настройка Together AI (аутентификация + выбор модели)"
read_when:
  - Вы хотите использовать Together AI с OpenClaw
  - Вам нужен ключ API в переменной окружения или выбор аутентификации через CLI
---

# Together AI

[Together AI](https://together.ai) предоставляет доступ к ведущим open-source моделям, включая Llama, DeepSeek, Kimi и другие, через единый API.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-совместимый

## Быстрый старт

1. Установите ключ API (рекомендуется сохранить его для Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Установите модель по умолчанию:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Пример без интерактивного режима

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Это установит `together/moonshotai/Kimi-K2.5` в качестве модели по умолчанию.

## Примечание по окружению

Если Gateway запущен как демон (launchd/systemd), убедитесь, что `TOGETHER_API_KEY`
доступен этому процессу (например, в `~/.clawdbot/.env` или через
`env.shellEnv`).

## Доступные модели

Together AI предоставляет доступ ко многим популярным open-source моделям:

- **GLM 4.7 Fp8** — модель по умолчанию с контекстным окном 200K
- **Llama 3.3 70B Instruct Turbo** — быстрая и эффективная модель для выполнения инструкций
- **Llama 4 Scout** — модель с поддержкой зрения и пониманием изображений
- **Llama 4 Maverick** — расширенные возможности зрения и логического вывода
- **DeepSeek V3.1** — мощная модель для программирования и логического вывода
- **DeepSeek R1** — продвинутая модель логического вывода
- **Kimi K2 Instruct** — высокопроизводительная модель с контекстным окном 262K

Все модели поддерживают стандартные chat completions и совместимы с OpenAI API.
