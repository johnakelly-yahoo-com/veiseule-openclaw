---
summary: "Używanie zgodnego z OpenAI API NVIDIA w OpenClaw"
read_when:
  - Chcesz używać modeli NVIDIA w OpenClaw
  - Musisz skonfigurować NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA udostępnia zgodne z OpenAI API pod adresem `https://integrate.api.nvidia.com/v1` dla modeli Nemotron i NeMo. Uwierzytelnij się za pomocą klucza API z [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Konfiguracja CLI

Wyeksportuj klucz jednorazowo, następnie uruchom onboarding i ustaw model NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Jeśli nadal przekazujesz `--token`, pamiętaj, że trafi on do historii powłoki i wyjścia `ps`; w miarę możliwości preferuj zmienną środowiskową.

## Fragment konfiguracji

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

## Identyfikatory modeli

- `nvidia/llama-3.1-nemotron-70b-instruct` (domyślny)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Uwagi

- Zgodny z OpenAI endpoint `/v1`; użyj klucza API z NVIDIA NGC.
- Dostawca włącza się automatycznie po ustawieniu `NVIDIA_API_KEY`; używa statycznych domyślnych wartości (okno kontekstu 131 072 tokenów, maks. 4 096 tokenów).
