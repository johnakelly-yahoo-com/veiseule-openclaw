---
summary: "Kör OpenClaw genom LiteLLM Proxy för enhetlig modellåtkomst och kostnadsspårning"
read_when:
  - Du vill dirigera OpenClaw genom en LiteLLM-proxy
  - Du behöver kostnadsspårning, loggning eller modellroutning via LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) är en LLM-gateway med öppen källkod som erbjuder ett enhetligt API till 100+ modellleverantörer. Dirigera OpenClaw genom LiteLLM för att få centraliserad kostnadsspårning, loggning och flexibiliteten att byta backend utan att ändra din OpenClaw-konfiguration.

## Varför använda LiteLLM med OpenClaw?

- **Kostnadsspårning** — Se exakt vad OpenClaw spenderar på alla modeller
- **Modellroutning** — Växla mellan Claude, GPT-4, Gemini, Bedrock utan konfigurationsändringar
- **Virtuella nycklar** — Skapa nycklar med utgiftsgränser för OpenClaw
- **Loggning** — Fullständiga request/response-loggar för felsökning
- **Fallbacks** — Automatisk failover om din primära leverantör ligger nere

## Snabbstart

### Via onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manuell installation

1. Starta LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Peka OpenClaw till LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Det är allt. OpenClaw dirigerar nu via LiteLLM.

## Konfiguration

### Miljövariabler

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

## Virtuella nycklar

Skapa en dedikerad nyckel för OpenClaw med utgiftsgränser:

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

Använd den genererade nyckeln som `LITELLM_API_KEY`.

## Modellroutning

LiteLLM kan dirigera modellförfrågningar till olika backends. Konfigurera i din LiteLLM `config.yaml`:

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

OpenClaw fortsätter att begära `claude-opus-4-6` — LiteLLM hanterar routningen.

## Visa användning

Kontrollera LiteLLM:s dashboard eller API:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Anteckningar

- LiteLLM körs på `http://localhost:4000` som standard
- OpenClaw ansluter via den OpenAI-kompatibla endpointen `/v1/chat/completions`
- Alla OpenClaw-funktioner fungerar via LiteLLM — inga begränsningar

## Se även

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)
