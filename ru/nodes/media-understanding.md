---
title: "Понимание медиа"
---

# Понимание медиа (входящее) — 2026-01-17

OpenClaw может **резюмировать входящие медиа** (изображения/аудио/видео) до запуска конвейера ответа. Он автоматически определяет доступность локальных инструментов или ключей провайдеров и может быть отключён или настроен. Если понимание выключено, модели по‑прежнему получают исходные файлы/URL как обычно.

## Цели

- Необязательно: предварительно преобразовывать входящие медиа в короткий текст для более быстрой маршрутизации и лучшего парсинга команд.
- Всегда сохранять доставку исходных медиа в модель.
- Поддержка **API провайдеров** и **резервных вариантов CLI**.
- Разрешить несколько моделей с упорядоченным резервированием (ошибка/размер/тайм‑аут).

## Поведение на высоком уровне

1. Сбор входящих вложений (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Для каждой включённой возможности (image/audio/video) выбор вложений по политике (по умолчанию: **первое**).
3. Выбор первой подходящей записи модели (размер + возможность + аутентификация).
4. Если модель не срабатывает или медиа слишком большое, **переход к следующей записи**.
5. При успехе:
   - `Body` становится блоком `[Image]`, `[Audio]` или `[Video]`.
   - Аудио устанавливает `{{Transcript}}`; парсинг команд использует текст подписи при наличии,
     иначе — транскрипт.
   - Подписи сохраняются как `User text:` внутри блока.

Если понимание не удалось или отключено, **поток ответа продолжается** с исходным телом + вложениями.

## Обзор конфигурации

`tools.media` поддерживает **общие модели** плюс переопределения для каждой возможности:

- `tools.media.models`: список общих моделей (используйте `capabilities` для гейтинга).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - значения по умолчанию (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - переопределения провайдеров (`baseUrl`, `headers`, `providerOptions`)
  - параметры Deepgram для аудио через `tools.media.audio.providerOptions.deepgram`
  - необязательный **список `models` для каждой возможности** (предпочтителен перед общими моделями)
  - политика `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (необязательный гейтинг по channel/chatType/session key)
- `tools.media.concurrency`: максимальное число одновременных запусков возможностей (по умолчанию **2**).

```json5
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### Записи моделей

Каждая запись `models[]` может быть **провайдерской** или **CLI**:

```json5
{
  type: "provider", // default if omitted
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // optional, used for multi‑modal entries
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

Шаблоны CLI также могут использовать:

- `{{MediaDir}}` (каталог, содержащий медиафайл)
- `{{OutputDir}}` (временный каталог, созданный для этого запуска)
- `{{OutputBase}}` (базовый путь временного файла, без расширения)

## Значения по умолчанию и ограничения

Рекомендуемые значения по умолчанию:

- `maxChars`: **500** для изображений/видео (коротко, удобно для команд)
- `maxChars`: **unset** для аудио (полный транскрипт, если не задан лимит)
- `maxBytes`:
  - image: **10MB**
  - audio: **20MB**
  - video: **50MB**

Правила:

- Если медиа превышает `maxBytes`, эта модель пропускается и **пробуется следующая**.
- Если модель возвращает больше, чем `maxChars`, вывод обрезается.
- `prompt` по умолчанию — простое «Describe the {media}.» плюс руководство `maxChars` (только image/video).
- Если `<capability>.enabled: true`, но модели не настроены, OpenClaw пытается использовать
  **активную модель ответа**, если её провайдер поддерживает возможность.

### Автоопределение понимания медиа (по умолчанию)

Если `tools.media.<capability>.enabled` **не** установлено в `false` и вы не
настроили модели, OpenClaw выполняет автоопределение в следующем порядке и **останавливается на первом
рабочем варианте**:

1. **Локальные CLI** (только аудио; при наличии установки)
   - `sherpa-onnx-offline` (требуется `SHERPA_ONNX_MODEL_DIR` с encoder/decoder/joiner/tokens)
   - `whisper-cli` (`whisper-cpp`; использует `WHISPER_CPP_MODEL` или встроенную tiny‑модель)
   - `whisper` (Python CLI; автоматически загружает модели)
2. **Gemini CLI** (`gemini`) с использованием `read_many_files`
3. **Ключи провайдеров**
   - Аудио: OpenAI → Groq → Deepgram → Google
   - Изображения: OpenAI → Anthropic → Google → MiniMax
   - Видео: Google

Чтобы отключить автоопределение, установите:

```json5
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

Примечание: обнаружение бинарников выполняется в режиме best‑effort для macOS/Linux/Windows; убедитесь, что CLI находится в `PATH` (мы расширяем `~`), либо задайте явную CLI‑модель с полным путём к команде.

## Возможности (необязательно)

Если вы задаёте `capabilities`, запись запускается только для этих типов медиа. Для общих
списков OpenClaw может вывести значения по умолчанию:

- `openai`, `anthropic`, `minimax`: **image**
- `google` (Gemini API): **image + audio + video**
- `groq`: **audio**
- `deepgram`: **audio**

Для записей CLI **явно задайте `capabilities`**, чтобы избежать неожиданных совпадений.
Если вы опускаете `capabilities`, запись подходит для списка, в котором она указана.

## Матрица поддержки провайдеров (интеграции OpenClaw)

| Возможность | Интеграция провайдера                              | Примечания                                                                              |
| ----------- | -------------------------------------------------- | --------------------------------------------------------------------------------------- |
| Изображение       | OpenAI / Anthropic / Google / другие через `pi-ai` | Подходит любая image‑совместимая модель из реестра.                     |
| Audio       | OpenAI, Groq, Deepgram, Google                     | Транскрипция у провайдера (Whisper/Deepgram/Gemini). |
| Video       | Google (Gemini API)             | Понимание видео у провайдера.                                           |

## Рекомендуемые провайдеры

**Image**

- Предпочитайте активную модель, если она поддерживает изображения.
- Хорошие значения по умолчанию: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Audio**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` или `deepgram/nova-3`.
- Резервный вариант CLI: `whisper-cli` (whisper-cpp) или `whisper`.
- Настройка Deepgram: [Deepgram (транскрипция аудио)](/providers/deepgram).

**Video**

- `google/gemini-3-flash-preview` (быстро), `google/gemini-3-pro-preview` (богаче).
- Резервный вариант CLI: CLI `gemini` (поддерживает `read_file` для видео/аудио).

## Политика вложений

Управление `attachments` с возможностью установки вложений:

- `mode`: `first` (по умолчанию) или `all`
- `maxAttachments`: ограничение количества обрабатываемых (по умолчанию **1**)
- `prefer`: `first`, `last`, `path`, `url`

Когда `mode: "all"`, результаты помечаются как `[Image 1/2]`, `[Audio 2/2]` и т. д.

## Примеры конфигурации

### 1. Список общих моделей + переопределения

```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2. Только Audio + Video (image выключено)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3. Необязательное понимание изображений

```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4. Мультимодальная одиночная запись (явные возможности)

```json5
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## Вывод статуса

Когда выполняется понимание медиа, `/status` включает короткую строку‑сводку:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Она показывает результаты по каждой возможности и выбранного провайдера/модель, когда применимо.

## Примечания

- Понимание — **best‑effort**. Ошибки не блокируют ответы.
- Вложения по‑прежнему передаются моделям, даже когда понимание отключено.
- Используйте `scope`, чтобы ограничить, где запускается понимание (например, только личные сообщения).

## Связанная документация

- [Конфигурация](/gateway/configuration)
- [Поддержка изображений и медиа](/nodes/images)


