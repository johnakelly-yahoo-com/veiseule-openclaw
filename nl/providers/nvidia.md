---
summary: "Gebruik de OpenAI-compatibele API van NVIDIA in OpenClaw"
read_when:
  - Je wilt NVIDIA-modellen gebruiken in OpenClaw
  - Je moet NVIDIA_API_KEY instellen
title: "NVIDIA"
---

# NVIDIA

NVIDIA biedt een OpenAI-compatibele API op `https://integrate.api.nvidia.com/v1` voor Nemotron- en NeMo-modellen. Authenticeer met een API-sleutel van [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## CLI-installatie

Exporteer de sleutel eenmalig, voer daarna onboarding uit en stel een NVIDIA-model in:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Als je nog steeds `--token` doorgeeft, onthoud dan dat dit in de shellgeschiedenis en de `ps`-uitvoer terechtkomt; geef waar mogelijk de voorkeur aan de omgevingsvariabele.

## Configuratiefragment

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## Model-ID's

- `nvidia/llama-3.1-nemotron-70b-instruct` (standaard)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Opmerkingen

- OpenAI-compatibel `/v1`-endpoint; gebruik een API-sleutel van NVIDIA NGC.
- De provider wordt automatisch ingeschakeld wanneer `NVIDIA_API_KEY` is ingesteld; gebruikt statische standaardwaarden (131.072-token contextvenster, 4.096 max tokens).
