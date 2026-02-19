---
summary: "Patakbuhin ang OpenClaw sa pamamagitan ng LiteLLM Proxy para sa pinag-isang access sa model at pagsubaybay sa gastos"
read_when:
  - Nais mong i-route ang OpenClaw sa pamamagitan ng isang LiteLLM proxy
  - Kailangan mo ng cost tracking, logging, o model routing sa pamamagitan ng LiteLLM
---

# LiteLLM

Ang [LiteLLM](https://litellm.ai) ay isang open-source na LLM gateway na nagbibigay ng iisang API para sa 100+ model providers. I-route ang OpenClaw sa pamamagitan ng LiteLLM upang makakuha ng sentralisadong cost tracking, logging, at kakayahang magpalit ng backends nang hindi binabago ang iyong OpenClaw config.

## Bakit gamitin ang LiteLLM kasama ng OpenClaw?

- **Cost tracking** — Makita nang eksakto kung magkano ang ginagastos ng OpenClaw sa lahat ng modelo
- **Model routing** — Magpalit sa pagitan ng Claude, GPT-4, Gemini, Bedrock nang walang pagbabago sa config
- **Virtual keys** — Gumawa ng mga key na may spend limits para sa OpenClaw
- **Logging** — Buong request/response logs para sa debugging
- **Fallbacks** — Awtomatikong failover kung hindi available ang iyong pangunahing provider

## Mabilisang pagsisimula

### Sa pamamagitan ng onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manu-manong setup

1. Simulan ang LiteLLM Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Ituro ang OpenClaw sa LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Iyon lang. Ang OpenClaw ay dadaan na ngayon sa LiteLLM.

## Configuration

### Mga environment variable

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Config file

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

## Virtual keys

Gumawa ng hiwalay na key para sa OpenClaw na may spend limits:

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

Gamitin ang nabuong key bilang `LITELLM_API_KEY`.

## Model routing

Maaaring i-route ng LiteLLM ang mga model request sa iba’t ibang backends. I-configure sa iyong LiteLLM `config.yaml`:

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

Patuloy na magre-request ang OpenClaw ng `claude-opus-4-6` — ang LiteLLM ang bahalang mag-handle ng routing.

## Pagtingin sa paggamit

Suriin ang dashboard o API ng LiteLLM:

```bash
# Key info
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Spend logs
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Mga Tala

- Ang LiteLLM ay tumatakbo sa `http://localhost:4000` bilang default
- Kumokonekta ang OpenClaw sa pamamagitan ng OpenAI-compatible na `/v1/chat/completions` endpoint
- Gumagana ang lahat ng OpenClaw features sa pamamagitan ng LiteLLM — walang limitasyon

## Tingnan din

- [LiteLLM Docs](https://docs.litellm.ai)
- [Model Providers](/concepts/model-providers)
