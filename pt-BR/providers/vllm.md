---
summary: "Executar o OpenClaw com vLLM (servidor local compatível com OpenAI)"
read_when:
  - Você quer executar o OpenClaw usando um servidor vLLM local
  - Você quer endpoints /v1 compatíveis com OpenAI usando seus próprios modelos
title: "vLLM"
---

# vLLM

vLLM pode servir modelos open-source (e alguns personalizados) por meio de uma API HTTP **compatível com OpenAI**. O OpenClaw pode se conectar ao vLLM usando a API `openai-completions`.

O OpenClaw também pode **descobrir automaticamente** os modelos disponíveis no vLLM quando você optar por usar `VLLM_API_KEY` (qualquer valor funciona se o seu servidor não exigir autenticação) e não definir explicitamente uma entrada `models.providers.vllm`.

## Início rápido

1. Inicie o vLLM com um servidor compatível com OpenAI.

Sua URL base deve expor endpoints `/v1` (por exemplo, `/v1/models`, `/v1/chat/completions`). O vLLM normalmente é executado em:

- `http://127.0.0.1:8000/v1`

2. Ative (qualquer valor funciona se nenhuma autenticação estiver configurada):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Selecione um modelo (substitua por um dos IDs de modelo do seu vLLM):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Descoberta de modelos (provedor implícito)

Quando `VLLM_API_KEY` estiver definido (ou existir um perfil de autenticação) e você **não** definir `models.providers.vllm`, o OpenClaw fará uma consulta a:

- `GET http://127.0.0.1:8000/v1/models`

…e converterá os IDs retornados em entradas de modelo.

Se você definir `models.providers.vllm` explicitamente, a descoberta automática será ignorada e você deverá definir os modelos manualmente.

## Configuração explícita (modelos manuais)

Use configuração explícita quando:

- O vLLM estiver sendo executado em um host/porta diferente.
- Você quiser fixar os valores de `contextWindow`/`maxTokens`.
- Seu servidor exigir uma API key real (ou se você quiser controlar os headers).

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

## Solução de problemas

- Verifique se o servidor está acessível:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Se as requisições falharem com erros de autenticação, defina um `VLLM_API_KEY` real que corresponda à configuração do seu servidor ou configure o provedor explicitamente em `models.providers.vllm`.
