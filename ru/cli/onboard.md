---
summary: "Справка CLI для `openclaw onboard` (интерактивный мастер онбординга)"
read_when:
  - Вам нужна пошаговая настройка Gateway (шлюз), рабочего пространства, аутентификации, каналов и Skills
title: "onboard"
---

# `openclaw onboard`

Интерактивный мастер онбординга (локальная или удалённая настройка Gateway (шлюз)).

## Связанные руководства

- Центр онбординга CLI: [Onboarding Wizard (CLI)](/start/wizard)
- Обзор онбординга: [Onboarding Overview](/start/onboarding-overview)
- Автоматизация CLI: [CLI Automation](/start/wizard-cli-automation)
- Справочник онбординга CLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Онбординг в macOS: [Onboarding (macOS App)](/start/onboarding)

## Примеры

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Пользовательский провайдер (неинтерактивный режим):

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` является необязательным в неинтерактивном режиме. Если параметр не указан, при онбординге проверяется `CUSTOM_API_KEY`.

Варианты endpoint Z.AI для неинтерактивного режима:

Примечание: `--auth-choice zai-api-key` теперь автоматически определяет лучший endpoint Z.AI для вашего ключа (предпочитает общий API с `zai/glm-5`).
Если вам нужны именно endpoint плана GLM Coding, выберите `zai-coding-global` или `zai-coding-cn`.

```bash
# Выбор endpoint без запроса
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Другие варианты endpoint Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Примечания к потоку:

- `quickstart`: минимальные запросы, автоматически генерирует токен шлюза.
- `manual`: полный набор запросов для порта/привязки/аутентификации (алиас `advanced`).
- Самый быстрый первый чат: `openclaw dashboard` (Control UI, без настройки каналов).
- Пользовательский провайдер: подключение к любому endpoint, совместимому с OpenAI или Anthropic,
  включая размещённых провайдеров, не указанных в списке. Используйте Unknown для автоопределения.

## Часто используемые последующие команды

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` не подразумевает неинтерактивный режим. Для скриптов используйте `--non-interactive`.
</Note>

