---
summary: "Brug NVIDIAs OpenAI-kompatible API i OpenClaw"
read_when:
  - Du vil bruge NVIDIA-modeller i OpenClaw
  - Du skal have NVIDIA_API_KEY sat op
title: "NVIDIA"
---

# NVIDIA

NVIDIA leverer en OpenAI-kompatibel API på `https://integrate.api.nvidia.com/v1` til Nemotron- og NeMo-modeller. Autentificér med en API-nøgle fra [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## CLI-opsætning

Eksportér nøglen én gang, kør derefter onboarding og angiv en NVIDIA-model:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Hvis du stadig angiver `--token`, så husk at den gemmes i shell-historikken og i `ps`-output; brug helst miljøvariablen, når det er muligt.

## Konfigurationsudsnit

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

## Model-ID'er

- `nvidia/llama-3.1-nemotron-70b-instruct` (standard)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Bemærkninger

- OpenAI-kompatibelt `/v1`-endpoint; brug en API-nøgle fra NVIDIA NGC.
- Provideren aktiveres automatisk, når `NVIDIA_API_KEY` er sat; bruger statiske standardværdier (131.072-token kontekstvindue, 4.096 maks. tokens).

