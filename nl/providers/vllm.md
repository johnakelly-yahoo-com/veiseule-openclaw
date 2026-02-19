---
summary: "OpenClaw uitvoeren met vLLM (OpenAI-compatibele lokale server)"
read_when:
  - Je wilt OpenClaw uitvoeren tegen een lokale vLLM-server
  - Je wilt OpenAI-compatibele `/v1`-endpoints gebruiken met je eigen modellen
title: "vLLM"
---

# vLLM

vLLM kan open-source (en sommige aangepaste) modellen aanbieden via een **OpenAI-compatibele** HTTP API. OpenClaw kan verbinding maken met vLLM via de `openai-completions` API.

OpenClaw kan ook beschikbare modellen van vLLM **automatisch detecteren** wanneer je inschakelt met `VLLM_API_KEY` (elke waarde werkt als je server geen authenticatie afdwingt) en je geen expliciete `models.providers.vllm`-vermelding definieert.

## Snelle start

1. Start vLLM met een OpenAI-compatibele server.

Je basis-URL moet `/v1`-endpoints beschikbaar stellen (bijv. `/v1/models`, `/v1/chat/completions`). vLLM draait doorgaans op:

- `http://127.0.0.1:8000/v1`

2. Schakel in (elke waarde werkt als er geen authenticatie is geconfigureerd):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Selecteer een model (vervang door een van je vLLM-model-ID’s):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Modeldetectie (impliciete provider)

Wanneer `VLLM_API_KEY` is ingesteld (of er een authenticatieprofiel bestaat) en je **geen** `models.providers.vllm` definieert, zal OpenClaw het volgende opvragen:

- `GET http://127.0.0.1:8000/v1/models`

…en de geretourneerde ID’s omzetten in modelvermeldingen.

Als je `models.providers.vllm` expliciet instelt, wordt automatische detectie overgeslagen en moet je modellen handmatig definiëren.

## Expliciete configuratie (handmatige modellen)

Gebruik expliciete configuratie wanneer:

- vLLM op een andere host/poort draait.
- Je `contextWindow`/`maxTokens`-waarden wilt vastzetten.
- Je server een echte API-sleutel vereist (of je headers wilt beheren).

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

## Probleemoplossing

- Controleer of de server bereikbaar is:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Als verzoeken mislukken met authenticatiefouten, stel dan een echte `VLLM_API_KEY` in die overeenkomt met je serverconfiguratie, of configureer de provider expliciet onder `models.providers.vllm`.

