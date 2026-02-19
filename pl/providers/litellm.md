---
summary: "Uruchom OpenClaw przez LiteLLM Proxy, aby uzyskać ujednolicony dostęp do modeli i śledzenie kosztów"
read_when:
  - Chcesz kierować ruch OpenClaw przez proxy LiteLLM
  - Potrzebujesz śledzenia kosztów, logowania lub routingu modeli przez LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) to brama LLM typu open-source, która zapewnia ujednolicone API do ponad 100 dostawców modeli. Kieruj OpenClaw przez LiteLLM, aby uzyskać scentralizowane śledzenie kosztów, logowanie oraz możliwość zmiany backendu bez modyfikowania konfiguracji OpenClaw.

## Dlaczego warto używać LiteLLM z OpenClaw?

- **Śledzenie kosztów** — Zobacz dokładnie, ile OpenClaw wydaje na wszystkie modele
- **Routing modeli** — Przełączaj się między Claude, GPT-4, Gemini, Bedrock bez zmian w konfiguracji
- **Klucze wirtualne** — Twórz klucze z limitami wydatków dla OpenClaw
- **Logowanie** — Pełne logi żądań i odpowiedzi do debugowania
- **Mechanizmy zapasowe (fallback)** — Automatyczne przełączanie awaryjne, jeśli główny dostawca jest niedostępny

## Szybki start

### Przez onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Konfiguracja ręczna

1. Uruchom LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Skieruj OpenClaw do LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

To wszystko. OpenClaw kieruje teraz ruch przez LiteLLM.

## Konfiguracja

### Zmienne środowiskowe

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Plik konfiguracyjny

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Klucze wirtualne

Utwórz dedykowany klucz dla OpenClaw z limitami wydatków:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Użyj wygenerowanego klucza jako `LITELLM_API_KEY`.

## Routing modeli

LiteLLM może kierować żądania modeli do różnych backendów. Skonfiguruj w pliku `config.yaml` LiteLLM:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw nadal wysyła żądania do `claude-opus-4-6` — LiteLLM obsługuje routowanie.

## Wyświetlanie użycia

Sprawdź panel LiteLLM lub API:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Uwagi

- LiteLLM domyślnie działa pod adresem `http://localhost:4000`
- OpenClaw łączy się przez zgodny z OpenAI endpoint `/v1/chat/completions`
- Wszystkie funkcje OpenClaw działają przez LiteLLM — bez ograniczeń

## Zobacz także

- [LiteLLM Docs](https://docs.litellm.ai)
- [Dostawcy modeli](/concepts/model-providers)

