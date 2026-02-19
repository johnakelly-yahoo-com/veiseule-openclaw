---
summary: "Rozumienie przychodzących obrazów/dźwięku/wideo (opcjonalne) z użyciem dostawcy oraz mechanizmów awaryjnych CLI"
read_when:
  - Projektowanie lub refaktoryzacja rozumienia mediów
  - Strojenie przetwarzania wstępnego przychodzących nagrań audio/wideo/obrazów
title: "Rozumienie mediów"
---

# Rozumienie mediów (przychodzących) — 2026-01-17

OpenClaw może **podsumowywać przychodzące media** (obraz/dźwięk/wideo) zanim uruchomi się potok odpowiedzi. System automatycznie wykrywa dostępność narzędzi lokalnych lub kluczy dostawców i może zostać wyłączony albo dostosowany. Jeśli rozumienie jest wyłączone, modele nadal otrzymują oryginalne pliki/adresy URL jak zwykle.

## Cele

- Opcjonalnie: wstępne „strawienie” przychodzących mediów do krótkiego tekstu dla szybszego routingu i lepszego parsowania poleceń.
- Zachowanie oryginalnego dostarczania mediów do modelu (zawsze).
- Obsługa **API dostawców** oraz **mechanizmów awaryjnych CLI**.
- Umożliwienie wielu modeli z uporządkowanym fallbackiem (błąd/rozmiar/limit czasu).

## Zachowanie wysokiego poziomu

1. Zbieranie przychodzących załączników (`MediaPaths`, `MediaUrls`, `MediaTypes`).
2. Dla każdej włączonej możliwości (obraz/dźwięk/wideo) wybór załączników zgodnie z polityką (domyślnie: **pierwszy**).
3. Wybór pierwszego kwalifikującego się wpisu modelu (rozmiar + możliwości + uwierzytelnienie).
4. Jeśli model zawiedzie lub media są zbyt duże, **następuje przejście do następnego wpisu**.
5. Po sukcesie:
   - `Body` staje się blokiem `[Image]`, `[Audio]` lub `[Video]`.
   - Audio ustawia `{{Transcript}}`; parsowanie poleceń używa tekstu podpisu, jeśli jest obecny,
     w przeciwnym razie transkrypcji.
   - Podpisy są zachowywane jako `User text:` wewnątrz bloku.

Jeśli rozumienie się nie powiedzie lub jest wyłączone, **przepływ odpowiedzi jest kontynuowany** z oryginalnym treścią + załącznikami.

## Przegląd konfiguracji

`tools.media` obsługuje **modele współdzielone** oraz nadpisania per‑możliwość:

- `tools.media.models`: lista modeli współdzielonych (użyj `capabilities` do ograniczeń).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - domyślne (`prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`)
  - nadpisania dostawcy (`baseUrl`, `headers`, `providerOptions`)
  - opcje audio Deepgram przez `tools.media.audio.providerOptions.deepgram`
  - opcjonalna **lista `models` per‑możliwość** (preferowana przed modelami współdzielonymi)
  - polityka `attachments` (`mode`, `maxAttachments`, `prefer`)
  - `scope` (opcjonalne ograniczanie według kanału/chatType/klucza sesji)
- `tools.media.concurrency`: maksymalna liczba równoległych uruchomień możliwości (domyślnie **2**).

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

### Wpisy modeli

Każdy wpis `models[]` może być **dostawcą** lub **CLI**:

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

Szablony CLI mogą także używać:

- `{{MediaDir}}` (katalog zawierający plik multimedialny)
- `{{OutputDir}}` (katalog tymczasowy utworzony na to uruchomienie)
- `{{OutputBase}}` (bazowa ścieżka pliku tymczasowego, bez rozszerzenia)

## Ustawienia domyślne i limity

Zalecane ustawienia domyślne:

- `maxChars`: **500** dla obrazu/wideo (krótkie, przyjazne dla poleceń)
- `maxChars`: **nieustawione** dla audio (pełna transkrypcja, chyba że ustawisz limit)
- `maxBytes`:
  - obraz: **10MB**
  - audio: **20MB**
  - wideo: **50MB**

Zasady:

- Jeśli media przekraczają `maxBytes`, ten model jest pomijany i **próbowany jest następny model**.
- Jeśli model zwróci więcej niż `maxChars`, wynik jest przycinany.
- `prompt` domyślnie używa prostego „Describe the {media}.” plus wskazówek `maxChars` (tylko obraz/wideo).
- Jeśli `<capability>.enabled: true`, ale nie skonfigurowano modeli, OpenClaw próbuje użyć
  **aktywnego modelu odpowiedzi**, gdy jego dostawca obsługuje daną możliwość.

### Automatyczne wykrywanie rozumienia mediów (domyślnie)

Jeśli `tools.media.<capability>.enabled` **nie** jest ustawione na `false` i nie skonfigurowano
modeli, OpenClaw automatycznie wykrywa w tej kolejności i **zatrzymuje się na pierwszej
działającej opcji**:

1. **Lokalne CLI** (tylko audio; jeśli zainstalowane)
   - `sherpa-onnx-offline` (wymaga `SHERPA_ONNX_MODEL_DIR` z enkoderem/dekoderem/łącznikiem/tokenami)
   - `whisper-cli` (`whisper-cpp`; używa `WHISPER_CPP_MODEL` lub dołączonego małego modelu)
   - `whisper` (CLI w Pythonie; automatycznie pobiera modele)
2. **Gemini CLI** (`gemini`) z użyciem `read_many_files`
3. **Klucze dostawców**
   - Dźwięk: OpenAI → Groq → Deepgram → Google
   - Obraz: OpenAI → Anthropic → Google → MiniMax
   - Wideo: Google

Aby wyłączyć automatyczne wykrywanie, ustaw:

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

Uwaga: wykrywanie binariów jest realizowane w trybie best‑effort na macOS/Linux/Windows; upewnij się, że CLI znajduje się w `PATH` (rozwijamy `~`), lub ustaw jawny model CLI z pełną ścieżką do polecenia.

## Możliwości (opcjonalnie)

Jeśli ustawisz `capabilities`, wpis uruchamia się tylko dla tych typów mediów. Dla list
współdzielonych OpenClaw może wywnioskować wartości domyślne:

- `openai`, `anthropic`, `minimax`: **obraz**
- `google` (Gemini API): **obraz + audio + wideo**
- `groq`: **audio**
- `deepgram`: **audio**

Dla wpisów CLI **ustaw `capabilities` jawnie**, aby uniknąć nieoczekiwanych dopasowań.
Jeśli pominiesz `capabilities`, wpis kwalifikuje się do listy, w której się znajduje.

## Macierz obsługi dostawców (integracje OpenClaw)

| Możliwość | Integracja dostawcy                              | Uwagi                                                                               |
| --------- | ------------------------------------------------ | ----------------------------------------------------------------------------------- |
| Obraz     | OpenAI / Anthropic / Google / inne przez `pi-ai` | Każdy model obsługujący obrazy w rejestrze działa.                  |
| Dźwięk    | OpenAI, Groq, Deepgram, Google                   | Transkrypcja dostawcy (Whisper/Deepgram/Gemini). |
| Wideo     | Google (Gemini API)           | Rozumienie wideo przez dostawcę.                                    |

## Zalecani dostawcy

**Obraz**

- Preferuj aktywny model, jeśli obsługuje obrazy.
- Dobre ustawienia domyślne: `openai/gpt-5.2`, `anthropic/claude-opus-4-6`, `google/gemini-3-pro-preview`.

**Dźwięk**

- `openai/gpt-4o-mini-transcribe`, `groq/whisper-large-v3-turbo` lub `deepgram/nova-3`.
- Fallback CLI: `whisper-cli` (whisper-cpp) lub `whisper`.
- Konfiguracja Deepgram: [Deepgram (transkrypcja audio)](/providers/deepgram).

**Wideo**

- `google/gemini-3-flash-preview` (szybkie), `google/gemini-3-pro-preview` (bogatsze).
- Fallback CLI: CLI `gemini` (obsługuje `read_file` dla wideo/audio).

## Polityka załączników

`attachments` per‑możliwość kontroluje, które załączniki są przetwarzane:

- `mode`: `first` (domyślnie) lub `all`
- `maxAttachments`: limit liczby przetwarzanych (domyślnie **1**)
- `prefer`: `first`, `last`, `path`, `url`

Gdy `mode: "all"`, wyniki są oznaczane jako `[Image 1/2]`, `[Audio 2/2]` itd.

## Przykłady konfiguracji

### 1. Lista modeli współdzielonych + nadpisania

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

### 2. Tylko audio + wideo (obraz wyłączony)

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

### 3. Opcjonalne rozumienie obrazu

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

### 4. Pojedynczy wpis multimodalny (jawne możliwości)

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

## Wyjście statusu

Gdy działa rozumienie mediów, `/status` zawiera krótką linię podsumowania:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

Pokazuje to wyniki per‑możliwość oraz wybranego dostawcę/model, gdy ma to zastosowanie.

## Uwagi

- Rozumienie działa w trybie **best‑effort**. Błędy nie blokują odpowiedzi.
- Załączniki są nadal przekazywane do modeli nawet wtedy, gdy rozumienie jest wyłączone.
- Użyj `scope`, aby ograniczyć miejsca, w których uruchamia się rozumienie (np. tylko DM‑y).

## Powiązana dokumentacja

- [Konfiguracja](/gateway/configuration)
- [Obsługa obrazów i mediów](/nodes/images)
