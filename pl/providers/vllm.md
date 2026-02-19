---
summary: "Uruchom OpenClaw z vLLM (lokalny serwer kompatybilny z OpenAI)"
read_when:
  - Chcesz uruchomić OpenClaw z lokalnym serwerem vLLM
  - Chcesz używać endpointów /v1 kompatybilnych z OpenAI z własnymi modelami
title: "vLLM"
---

# vLLM

vLLM może udostępniać modele open-source (oraz niektóre niestandardowe) poprzez **API HTTP kompatybilne z OpenAI**. OpenClaw może łączyć się z vLLM przy użyciu API `openai-completions`.

OpenClaw może również **automatycznie wykrywać** dostępne modele z vLLM, jeśli wyrazisz na to zgodę poprzez `VLLM_API_KEY` (dowolna wartość działa, jeśli Twój serwer nie wymusza uwierzytelniania) i nie zdefiniujesz jawnego wpisu `models.providers.vllm`.

## Szybki start

1. Uruchom vLLM z serwerem kompatybilnym z OpenAI.

Twój bazowy URL powinien udostępniać endpointy `/v1` (np. `/v1/models`, `/v1/chat/completions`). vLLM zazwyczaj działa pod adresem:

- `http://127.0.0.1:8000/v1`

2. Włącz (dowolna wartość działa, jeśli nie skonfigurowano uwierzytelniania):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Wybierz model (zastąp jednym z identyfikatorów modelu vLLM):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Wykrywanie modeli (niejawny provider)

Gdy `VLLM_API_KEY` jest ustawione (lub istnieje profil uwierzytelniania) i **nie** zdefiniujesz `models.providers.vllm`, OpenClaw wykona zapytanie:

- `GET http://127.0.0.1:8000/v1/models`

…a następnie przekształci zwrócone identyfikatory w wpisy modeli.

Jeśli jawnie ustawisz `models.providers.vllm`, automatyczne wykrywanie zostanie pominięte i musisz ręcznie zdefiniować modele.

## Konfiguracja jawna (modele ręczne)

Użyj konfiguracji jawnej, gdy:

- vLLM działa na innym hoście/porcie.
- Chcesz przypiąć wartości `contextWindow`/`maxTokens`.
- Twój serwer wymaga prawdziwego klucza API (lub chcesz kontrolować nagłówki).

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Rozwiązywanie problemów

- Sprawdź, czy serwer jest osiągalny:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Jeśli żądania kończą się błędami uwierzytelniania, ustaw prawdziwy `VLLM_API_KEY` zgodny z konfiguracją serwera lub jawnie skonfiguruj providera w `models.providers.vllm`.
