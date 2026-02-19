---
summary: "Verwenden Sie NVIDIAs OpenAI-kompatible API in OpenClaw"
read_when:
  - Sie möchten NVIDIA-Modelle in OpenClaw verwenden
  - Sie benötigen eine eingerichtete NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA stellt unter `https://integrate.api.nvidia.com/v1` eine OpenAI-kompatible API für Nemotron- und NeMo-Modelle bereit. Authentifizieren Sie sich mit einem API-Schlüssel von [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## CLI-Einrichtung

Exportieren Sie den Schlüssel einmal und führen Sie dann das Onboarding aus und legen Sie ein NVIDIA-Modell fest:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Wenn Sie weiterhin `--token` übergeben, denken Sie daran, dass es im Shell-Verlauf und in der `ps`-Ausgabe erscheint; verwenden Sie nach Möglichkeit die Umgebungsvariable.

## Konfigurationsbeispiel

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

## Modell-IDs

- `nvidia/llama-3.1-nemotron-70b-instruct` (Standard)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Hinweise

- OpenAI-kompatibler `/v1`-Endpoint; verwenden Sie einen API-Schlüssel von NVIDIA NGC.
- Der Provider wird automatisch aktiviert, wenn `NVIDIA_API_KEY` gesetzt ist; verwendet statische Standardwerte (131.072-Token-Kontextfenster, 4.096 maximale Tokens).

