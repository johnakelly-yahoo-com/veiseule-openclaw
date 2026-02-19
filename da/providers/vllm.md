---
summary: "Kør OpenClaw med vLLM (OpenAI-kompatibel lokal server)"
read_when:
  - Du vil køre OpenClaw mod en lokal vLLM-server
  - Du ønsker OpenAI-kompatible /v1-endpoints med dine egne modeller
title: "vLLM"
---

# vLLM

vLLM kan servere open-source-modeller (og nogle brugerdefinerede) via en **OpenAI-kompatibel** HTTP API. OpenClaw kan forbinde til vLLM ved hjælp af `openai-completions` API.

OpenClaw kan også **automatisk finde** tilgængelige modeller fra vLLM, når du tilvælger det med `VLLM_API_KEY` (enhver værdi virker, hvis din server ikke håndhæver autentificering), og du ikke definerer en eksplicit `models.providers.vllm`-post.

## Hurtig start

1. Start vLLM med en OpenAI-kompatibel server.

Din base-URL skal eksponere `/v1`-endpoints (f.eks. `/v1/models`, `/v1/chat/completions`). vLLM kører typisk på:

- `http://127.0.0.1:8000/v1`

2. Tilmeld (enhver værdi virker, hvis der ikke er konfigureret autentificering):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Vælg en model (erstat med et af dine vLLM-model-ID’er):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Modelopdagelse (implicit udbyder)

Når `VLLM_API_KEY` er sat (eller der findes en auth-profil), og du **ikke** definerer `models.providers.vllm`, vil OpenClaw forespørge:

- `GET http://127.0.0.1:8000/v1/models`

…og konvertere de returnerede ID’er til modelposter.

Hvis du sætter `models.providers.vllm` eksplicit, springes automatisk opdagelse over, og du skal definere modeller manuelt.

## Eksplicit konfiguration (manuelle modeller)

Brug eksplicit konfiguration når:

- vLLM kører på en anden host/port.
- Du vil fastlåse værdier for `contextWindow`/`maxTokens`.
- Din server kræver en rigtig API-nøgle (eller du vil kontrollere headers).

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

## Fejlfinding

- Kontrollér, at serveren kan nås:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Hvis forespørgsler mislykkes med godkendelsesfejl, skal du angive en reel `VLLM_API_KEY`, der matcher din serverkonfiguration, eller konfigurere udbyderen eksplicit under `models.providers.vllm`.
