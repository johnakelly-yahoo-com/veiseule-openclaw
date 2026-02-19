---
summary: "Настройка Hugging Face Inference (аутентификация + выбор модели)"
read_when:
  - Вы хотите использовать Hugging Face Inference с OpenClaw
  - Вам нужна переменная окружения HF token или выбор аутентификации через CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) предоставляют совместимые с OpenAI chat completions через единый router API. Вы получаете доступ ко многим моделям (DeepSeek, Llama и другим) с одним токеном. OpenClaw использует **OpenAI-совместимую конечную точку** (только chat completions); для text-to-image, embeddings или speech используйте [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) напрямую.

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN` (fine-grained токен с разрешением **Make calls to Inference Providers**)
- API: совместимый с OpenAI (`https://router.huggingface.co/v1`)
- Billing: единый HF токен; [pricing](https://huggingface.co/docs/inference-providers/pricing) соответствует тарифам провайдеров с бесплатным уровнем.

## Быстрый старт

1. Создайте fine-grained токен в [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) с разрешением **Make calls to Inference Providers**.
2. Запустите onboarding и выберите **Hugging Face** в выпадающем списке провайдеров, затем введите свой API key по запросу:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. В выпадающем списке **Default Hugging Face model** выберите нужную модель (список загружается из Inference API при наличии действительного токена; в противном случае отображается встроенный список). Ваш выбор сохраняется как модель по умолчанию.
4. Вы также можете задать или изменить модель по умолчанию позже в конфигурации:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Неинтерактивный пример

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Это установит `huggingface/deepseek-ai/DeepSeek-R1` в качестве модели по умолчанию.

## Примечание об окружении

Если Gateway запускается как демон (launchd/systemd), убедитесь, что `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN`
доступен этому процессу (например, в `~/.openclaw/.env` или через
`env.shellEnv`).

## Обнаружение моделей и выпадающий список onboarding

OpenClaw обнаруживает модели, вызывая **Inference endpoint напрямую**:

```bash
GET https://router.huggingface.co/v1/models
```

(Необязательно: отправьте `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` или `$HF_TOKEN` для получения полного списка; некоторые endpoints возвращают подмножество без авторизации.) Ответ имеет формат OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Когда вы настраиваете Hugging Face API key (через onboarding, `HUGGINGFACE_HUB_TOKEN` или `HF_TOKEN`), OpenClaw использует этот GET-запрос для обнаружения доступных моделей chat-completion. Во время **interactive onboarding**, после ввода токена вы увидите выпадающий список **Default Hugging Face model**, заполненный на основе этого списка (или из встроенного каталога, если запрос завершится ошибкой). Во время выполнения (например, при запуске Gateway), если ключ присутствует, OpenClaw снова вызывает **GET** `https://router.huggingface.co/v1/models`, чтобы обновить каталог. Список объединяется со встроенным каталогом (для метаданных, таких как размер контекстного окна и стоимость). Если запрос завершится ошибкой или ключ не задан, используется только встроенный каталог.

## Названия моделей и редактируемые параметры

- **Имя из API:** Отображаемое имя модели **загружается из GET /v1/models**, если API возвращает `name`, `title` или `display_name`; в противном случае оно формируется из id модели (например, `deepseek-ai/DeepSeek-R1` → «DeepSeek R1»).
- **Переопределение отображаемого имени:** Вы можете задать собственную метку для каждой модели в конфигурации, чтобы она отображалась в CLI и UI так, как вам нужно:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Выбор провайдера / политики:** Добавьте суффикс к **model id**, чтобы указать, как роутер выбирает backend:

  - **`:fastest`** — максимальная пропускная способность (выбирает роутер; выбор провайдера **зафиксирован** — интерактивный выбор backend недоступен).
  - **`:cheapest`** — минимальная стоимость за выходной токен (выбирает роутер; выбор провайдера **зафиксирован**).
  - **`:provider`** — принудительно указать конкретный backend (например, `:sambanova`, `:together`).

  При выборе **:cheapest** или **:fastest** (например, в выпадающем списке модели во время onboarding) провайдер фиксируется: роутер принимает решение по стоимости или скорости, и дополнительный шаг «предпочесть конкретный backend» не отображается. Вы можете добавить их как отдельные записи в `models.providers.huggingface.models` или задать `model.primary` с суффиксом. Вы также можете задать порядок по умолчанию в [Inference Provider settings](https://hf.co/settings/inference-providers) (без суффикса = используется этот порядок).

- **Объединение конфигурации:** Существующие записи в `models.providers.huggingface.models` (например, в `models.json`) сохраняются при объединении конфигурации. Таким образом, любые пользовательские `name`, `alias` или параметры модели, заданные там, будут сохранены.

## Идентификаторы моделей и примеры конфигурации

Ссылки на модели используют формат `huggingface/<org>/<model>` (идентификаторы в стиле Hub). Список ниже получен из **GET** `https://router.huggingface.co/v1/models`; ваш каталог может содержать больше моделей.

**Примеры идентификаторов (из inference endpoint):**

| Модель                                 | Ref (с префиксом `huggingface/`) |
| -------------------------------------- | --------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                         |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                          |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                 |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                  |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                               |
| GLM 4.7                | `zai-org/GLM-4.7`                                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                              |

Вы можете добавить `:fastest`, `:cheapest` или `:provider` (например, `:together`, `:sambanova`) к идентификатору модели. Задайте порядок по умолчанию в [настройках Inference Provider](https://hf.co/settings/inference-providers); см. [Inference Providers](https://huggingface.co/docs/inference-providers) и **GET** `https://router.huggingface.co/v1/models` для полного списка.

### Полные примеры конфигурации

**Основная DeepSeek R1 с резервной Qwen:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen по умолчанию, с вариантами :cheapest и :fastest:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS с псевдонимами:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Принудительно указать конкретный backend с помощью :provider:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Несколько моделей Qwen и DeepSeek с суффиксами политик:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (дешевый)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (быстрый)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```
