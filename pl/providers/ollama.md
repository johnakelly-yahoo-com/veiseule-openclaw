---
summary: "Uruchom OpenClaw z Ollama (lokalne środowisko uruchomieniowe LLM)"
read_when:
  - Chcesz uruchomić OpenClaw z lokalnymi modelami przez Ollama
  - Potrzebujesz wskazówek dotyczących konfiguracji i ustawień Ollama
title: "Ollama"
---

# Ollama

Ollama to lokalne środowisko uruchomieniowe LLM, które ułatwia uruchamianie modeli open source na Twoim komputerze. OpenClaw integruje się z kompatybilnym z OpenAI API Ollama i może **automatycznie wykrywać modele obsługujące narzędzia**, gdy włączysz `OLLAMA_API_KEY` (lub profil uwierzytelniania) i nie zdefiniujesz jawnego wpisu `models.providers.ollama`.

## Szybki start

1. Zainstaluj Ollama: [https://ollama.ai](https://ollama.ai)

2. Pobierz model:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Włącz Ollama dla OpenClaw (dowolna wartość działa; Ollama nie wymaga prawdziwego klucza):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Użyj modeli Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Wykrywanie modeli (niejawny dostawca)

Gdy ustawisz `OLLAMA_API_KEY` (lub profil uwierzytelniania) i **nie** zdefiniujesz `models.providers.ollama`, OpenClaw wykrywa modele z lokalnej instancji Ollama pod adresem `http://127.0.0.1:11434`:

- Odpytuje `/api/tags` oraz `/api/show`
- Zachowuje tylko modele, które zgłaszają obsługę `tools`
- Oznacza `reasoning`, gdy model zgłasza `thinking`
- Odczytuje `contextWindow` z `model_info["<arch>.context_length"]`, gdy jest dostępne
- Ustawia `maxTokens` na 10× okno kontekstu
- Ustawia wszystkie koszty na `0`

Pozwala to uniknąć ręcznych wpisów modeli, jednocześnie utrzymując katalog zgodny z możliwościami Ollama.

Aby zobaczyć, jakie modele są dostępne:

```bash
ollama list
openclaw models list
```

Aby dodać nowy model, po prostu pobierz go za pomocą Ollama:

```bash
ollama pull mistral
```

Nowy model zostanie automatycznie wykryty i będzie dostępny do użycia.

Jeśli jawnie ustawisz `models.providers.ollama`, automatyczne wykrywanie zostanie pominięte i musisz zdefiniować modele ręcznie (zobacz poniżej).

## Konfiguracja

### Podstawowa konfiguracja (niejawne wykrywanie)

Najprostszym sposobem włączenia Ollama jest użycie zmiennej środowiskowej:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Konfiguracja jawna (modele ręczne)

Użyj konfiguracji jawnej, gdy:

- Ollama działa na innym hoście/porcie.
- Chcesz wymusić określone okna kontekstu lub listy modeli.
- Chcesz uwzględnić modele, które nie zgłaszają obsługi narzędzi.

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

Jeśli ustawiono `OLLAMA_API_KEY`, możesz pominąć `apiKey` we wpisie dostawcy, a OpenClaw uzupełni je na potrzeby sprawdzania dostępności.

### Niestandardowy adres bazowy (konfiguracja jawna)

Jeśli Ollama działa na innym hoście lub porcie (konfiguracja jawna wyłącza automatyczne wykrywanie, więc zdefiniuj modele ręcznie):

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

### Wybór modelu

Po skonfigurowaniu wszystkie Twoje modele Ollama są dostępne:

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

## Zaawansowane

### Modele rozumowania

OpenClaw oznacza modele jako zdolne do rozumowania, gdy Ollama zgłasza `thinking` w `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Koszty modeli

Ollama jest darmowa i działa lokalnie, więc wszystkie koszty modeli są ustawione na 0 USD.

### Konfiguracja strumieniowania

Integracja OpenClaw z Ollama domyślnie używa **natywnego API Ollama** (`/api/chat`), które w pełni obsługuje jednoczesne strumieniowanie i wywoływanie narzędzi. Nie jest wymagana żadna specjalna konfiguracja.

#### Starszy tryb zgodny z OpenAI

Jeśli zamiast tego musisz użyć endpointu zgodnego z OpenAI (np. za proxy obsługującym wyłącznie format OpenAI), ustaw jawnie `api: "openai-completions"`:

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

Uwaga: Endpoint zgodny z OpenAI może nie obsługiwać jednocześnie strumieniowania i wywoływania narzędzi. Jawnie ustaw `streaming: false` dla modeli Ollama (zobacz [Konfiguracja strumieniowania](#konfiguracja-strumieniowania))

### Okna kontekstu

Dla modeli wykrywanych automatycznie OpenClaw używa okna kontekstu zgłaszanego przez Ollama, gdy jest dostępne; w przeciwnym razie domyślnie używa `8192`. Możesz nadpisać `contextWindow` oraz `maxTokens` w jawnej konfiguracji dostawcy.

## Rozwiązywanie problemów

### Ollama nie jest wykrywana

Upewnij się, że Ollama jest uruchomiona oraz że ustawiono `OLLAMA_API_KEY` (lub profil uwierzytelniania) i że **nie** zdefiniowano jawnego wpisu `models.providers.ollama`:

```bash
ollama serve
```

Oraz że API jest dostępne:

```bash
curl http://localhost:11434/api/tags
```

### Brak dostępnych modeli

OpenClaw automatycznie wykrywa tylko modele, które zgłaszają obsługę narzędzi. Jeśli Twojego modelu nie ma na liście, możesz:

- Pobrać model obsługujący narzędzia albo
- Zdefiniować model jawnie w `models.providers.ollama`.

Aby dodać modele:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Odmowa połączenia

Sprawdź, czy Ollama działa na właściwym porcie:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Zobacz także

- [Model Providers](/concepts/model-providers) – Przegląd wszystkich dostawców
- [Model Selection](/concepts/models) – Jak wybierać modele
- [Configuration](/gateway/configuration) – Pełne odniesienie konfiguracji

