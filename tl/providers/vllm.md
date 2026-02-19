---
summary: "Patakbuhin ang OpenClaw gamit ang vLLM (OpenAI-compatible na lokal na server)"
read_when:
  - Gusto mong patakbuhin ang OpenClaw laban sa isang lokal na vLLM server
  - Gusto mo ng OpenAI-compatible na /v1 endpoints gamit ang sarili mong mga model
title: "vLLM"
---

# vLLM

Maaaring mag-serve ang vLLM ng open-source (at ilang custom) na mga model sa pamamagitan ng isang **OpenAI-compatible** na HTTP API. Maaaring kumonekta ang OpenClaw sa vLLM gamit ang `openai-completions` API.

Maaari ring **auto-discover** ng OpenClaw ang mga available na model mula sa vLLM kapag nag-opt in ka gamit ang `VLLM_API_KEY` (anumang value ay gagana kung hindi nag-eenforce ng auth ang iyong server) at hindi ka nagde-define ng tahasang `models.providers.vllm` entry.

## Mabilis na pagsisimula

1. Simulan ang vLLM gamit ang isang OpenAI-compatible na server.

Dapat ilantad ng iyong base URL ang `/v1` endpoints (hal. `/v1/models`, `/v1/chat/completions`). Karaniwang tumatakbo ang vLLM sa:

- `http://127.0.0.1:8000/v1`

2. Mag-opt in (anumang value ay gagana kung walang naka-configure na auth):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Pumili ng model (palitan ng isa sa iyong mga vLLM model ID):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Model discovery (implicit provider)

Kapag naka-set ang `VLLM_API_KEY` (o may umiiral na auth profile) at **hindi mo** dine-define ang `models.providers.vllm`, magtatanong ang OpenClaw sa:

- `GET http://127.0.0.1:8000/v1/models`

…at i-convert ang mga ibinalik na ID bilang mga entry ng model.

Kung tahasan mong itinakda ang `models.providers.vllm`, lalaktawan ang auto-discovery at kailangan mong manu-manong tukuyin ang mga model.

## Tahasan na configuration (manu-manong mga model)

Gumamit ng tahasang config kapag:

- Ang vLLM ay tumatakbo sa ibang host/port.
- Gusto mong i-pin ang mga halaga ng `contextWindow`/`maxTokens`.
- Ang iyong server ay nangangailangan ng tunay na API key (o gusto mong kontrolin ang mga header).

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

## Pag-troubleshoot

- Suriin kung naaabot ang server:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Kung nabigo ang mga request dahil sa auth errors, magtakda ng tunay na `VLLM_API_KEY` na tumutugma sa configuration ng iyong server, o tahasang i-configure ang provider sa ilalim ng `models.providers.vllm`.
