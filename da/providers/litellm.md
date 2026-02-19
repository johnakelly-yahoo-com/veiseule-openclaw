---
summary: "Kør OpenClaw gennem LiteLLM Proxy for samlet modeladgang og omkostningssporing"
read_when:
  - Du vil route OpenClaw gennem en LiteLLM-proxy
  - Du har brug for omkostningssporing, logning eller modelrouting gennem LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) er en open source LLM-gateway, der tilbyder en samlet API til 100+ modeludbydere. Route OpenClaw gennem LiteLLM for at få centraliseret omkostningssporing, logning og fleksibiliteten til at skifte backend uden at ændre din OpenClaw-konfiguration.

## Hvorfor bruge LiteLLM med OpenClaw?

- **Omkostningssporing** — Se præcist hvad OpenClaw bruger på tværs af alle modeller
- **Modelrouting** — Skift mellem Claude, GPT-4, Gemini, Bedrock uden konfigurationsændringer
- **Virtuelle nøgler** — Opret nøgler med forbrugsgrænser til OpenClaw
- **Logning** — Fuld log over requests/responses til fejlfinding
- **Fallbacks** — Automatisk failover, hvis din primære udbyder er nede

## Hurtig start

### Via onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manuel opsætning

1. Start LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Peg OpenClaw til LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Det var det. OpenClaw ruter nu gennem LiteLLM.

## Konfiguration

### Miljøvariabler

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Konfigurationsfil

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

## Virtuelle nøgler

Opret en dedikeret nøgle til OpenClaw med forbrugsgrænser:

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

Brug den genererede nøgle som `LITELLM_API_KEY`.

## Modelrouting

LiteLLM kan rute modelanmodninger til forskellige backends. Konfigurer i din LiteLLM `config.yaml`:

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

OpenClaw fortsætter med at anmode om `claude-opus-4-6` — LiteLLM håndterer routingen.

## Visning af forbrug

Tjek LiteLLM's dashboard eller API:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Bemærkninger

- LiteLLM kører som standard på `http://localhost:4000`
- OpenClaw forbinder via det OpenAI-kompatible `/v1/chat/completions` endpoint
- Alle OpenClaw-funktioner fungerer gennem LiteLLM — ingen begrænsninger

## Se også

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)

