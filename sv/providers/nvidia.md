---
summary: "Använd NVIDIAs OpenAI-kompatibla API i OpenClaw"
read_when:
  - Du vill använda NVIDIA-modeller i OpenClaw
  - Du behöver ha NVIDIA_API_KEY konfigurerad
title: "NVIDIA"
---

# NVIDIA

NVIDIA tillhandahåller ett OpenAI-kompatibelt API på `https://integrate.api.nvidia.com/v1` för Nemotron- och NeMo-modeller. Autentisera med en API-nyckel från [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## CLI-installation

Exportera nyckeln en gång och kör sedan onboarding och ange en NVIDIA-modell:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Om du fortfarande skickar med `--token`, tänk på att det hamnar i shell-historiken och i `ps`-utdata; använd helst miljövariabeln när det är möjligt.

## Konfigurationsutdrag

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

## Modell-ID:n

- `nvidia/llama-3.1-nemotron-70b-instruct` (standard)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Anteckningar

- OpenAI-kompatibel `/v1`-endpoint; använd en API-nyckel från NVIDIA NGC.
- Providern aktiveras automatiskt när `NVIDIA_API_KEY` är satt; använder statiska standardvärden (kontextfönster på 131 072 token, 4 096 max token).
