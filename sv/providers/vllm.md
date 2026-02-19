---
summary: "Kör OpenClaw med vLLM (OpenAI-kompatibel lokal server)"
read_when:
  - Du vill köra OpenClaw mot en lokal vLLM-server
  - Du vill ha OpenAI-kompatibla /v1-endpoints med dina egna modeller
title: "vLLM"
---

# vLLM

vLLM kan köra open-source-modeller (och vissa anpassade) via ett **OpenAI-kompatibelt** HTTP API. OpenClaw kan ansluta till vLLM med API:et `openai-completions`.

OpenClaw kan också **automatiskt upptäcka** tillgängliga modeller från vLLM när du väljer att aktivera det med `VLLM_API_KEY` (vilket värde som helst fungerar om din server inte kräver autentisering) och du inte definierar en explicit `models.providers.vllm`-post.

## Snabbstart

1. Starta vLLM med en OpenAI-kompatibel server.

Din base URL ska exponera `/v1`-endpoints (t.ex. `/v1/models`, `/v1/chat/completions`). vLLM körs vanligtvis på:

- `http://127.0.0.1:8000/v1`

2. Aktivera (vilket värde som helst fungerar om ingen autentisering är konfigurerad):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Välj en modell (ersätt med ett av dina vLLM-modell-ID:n):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Modellupptäckt (implicit provider)

När `VLLM_API_KEY` är satt (eller en autentiseringsprofil finns) och du **inte** definierar `models.providers.vllm`, kommer OpenClaw att fråga:

- `GET http://127.0.0.1:8000/v1/models`

…och omvandla de returnerade ID:na till modellposter.

Om du sätter `models.providers.vllm` explicit hoppas automatisk upptäckt över och du måste definiera modeller manuellt.

## Explicit konfiguration (manuella modeller)

Använd explicit konfiguration när:

- vLLM körs på en annan host/port.
- Du vill låsa `contextWindow`/`maxTokens`-värden.
- Din server kräver en riktig API-nyckel (eller om du vill kontrollera headers).

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

## Felsökning

- Kontrollera att servern är nåbar:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Om förfrågningar misslyckas med autentiseringsfel, ange en giltig `VLLM_API_KEY` som matchar din serverkonfiguration, eller konfigurera leverantören uttryckligen under `models.providers.vllm`.

