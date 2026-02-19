---
summary: "Voer OpenClaw uit via LiteLLM Proxy voor uniforme modeltoegang en kostenregistratie"
read_when:
  - Je wilt OpenClaw routeren via een LiteLLM-proxy
  - Je hebt kostenregistratie, logging of modelroutering via LiteLLM nodig
---

# LiteLLM

[LiteLLM](https://litellm.ai) is een open-source LLM-gateway die een uniforme API biedt voor 100+ modelproviders. Routeer OpenClaw via LiteLLM om gecentraliseerde kostenregistratie, logging en de flexibiliteit om van backend te wisselen te krijgen zonder je OpenClaw-configuratie te wijzigen.

## Waarom LiteLLM gebruiken met OpenClaw?

- **Kostenregistratie** — Zie precies wat OpenClaw uitgeeft over alle modellen heen
- **Modelroutering** — Wissel tussen Claude, GPT-4, Gemini, Bedrock zonder configuratiewijzigingen
- **Virtuele sleutels** — Maak sleutels met bestedingslimieten voor OpenClaw
- **Logging** — Volledige request/response-logs voor debugging
- **Fallbacks** — Automatische failover als je primaire provider niet beschikbaar is

## Snel starten

### Via onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Handmatige installatie

1. Start de LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Wijs OpenClaw naar LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Dat is alles. OpenClaw routeert nu via LiteLLM.

## Configuratie

### Omgevingsvariabelen

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Configuratiebestand

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

## Virtuele sleutels

Maak een aparte sleutel voor OpenClaw met bestedingslimieten:

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

Gebruik de gegenereerde sleutel als `LITELLM_API_KEY`.

## Modelroutering

LiteLLM kan modelverzoeken naar verschillende backends routeren. Configureer dit in je LiteLLM `config.yaml`:

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

OpenClaw blijft `claude-opus-4-6` aanvragen — LiteLLM verzorgt de routering.

## Gebruik bekijken

Controleer het LiteLLM-dashboard of de API:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Opmerkingen

- LiteLLM draait standaard op `http://localhost:4000`
- OpenClaw maakt verbinding via het OpenAI-compatibele `/v1/chat/completions`-endpoint
- Alle OpenClaw-functies werken via LiteLLM — geen beperkingen

## Zie ook

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)
