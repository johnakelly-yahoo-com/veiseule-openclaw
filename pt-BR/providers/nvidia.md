---
summary: "Use a API compatível com OpenAI da NVIDIA no OpenClaw"
read_when:
  - Você quer usar modelos da NVIDIA no OpenClaw
  - Você precisa configurar a NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

A NVIDIA fornece uma API compatível com OpenAI em `https://integrate.api.nvidia.com/v1` para modelos Nemotron e NeMo. Autentique-se com uma API key do [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Configuração da CLI

Exporte a chave uma vez, depois execute o onboarding e defina um modelo da NVIDIA:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Se você ainda passar `--token`, lembre-se de que ele ficará no histórico do shell e na saída do `ps`; prefira a variável de ambiente sempre que possível.

## Trecho de configuração

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

## IDs de modelo

- `nvidia/llama-3.1-nemotron-70b-instruct` (padrão)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Notas

- Endpoint `/v1` compatível com OpenAI; use uma chave de API do NVIDIA NGC.
- O provedor é ativado automaticamente quando `NVIDIA_API_KEY` está definida; usa padrões estáticos (janela de contexto de 131.072 tokens, máximo de 4.096 tokens).

