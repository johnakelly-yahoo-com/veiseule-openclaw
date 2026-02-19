---
summary: "Utiliser l’API compatible OpenAI de NVIDIA dans OpenClaw"
read_when:
  - Vous souhaitez utiliser des modèles NVIDIA dans OpenClaw
  - Vous devez configurer NVIDIA_API_KEY
title: "NVIDIA"
---

# NVIDIA

NVIDIA fournit une API compatible OpenAI à l’adresse `https://integrate.api.nvidia.com/v1` pour les modèles Nemotron et NeMo. Authentifiez-vous avec une clé API provenant de [NVIDIA NGC](https://catalog.ngc.nvidia.com/).

## Configuration CLI

Exportez la clé une fois, puis lancez l’onboarding et définissez un modèle NVIDIA :

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

Si vous passez encore `--token`, rappelez-vous qu’il sera enregistré dans l’historique du shell et visible via `ps` ; privilégiez la variable d’environnement lorsque c’est possible.

## Extrait de configuration

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

## Identifiants de modèles

- `nvidia/llama-3.1-nemotron-70b-instruct` (par défaut)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## Remarques

- Endpoint `/v1` compatible OpenAI ; utilisez une clé API provenant de NVIDIA NGC.
- Le fournisseur s’active automatiquement lorsque `NVIDIA_API_KEY` est défini ; utilise des valeurs par défaut statiques (fenêtre de contexte de 131 072 tokens, maximum de 4 096 tokens).

