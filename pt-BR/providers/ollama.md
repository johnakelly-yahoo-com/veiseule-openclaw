---
summary: "Execute o OpenClaw com o Ollama (runtime local de LLM)"
read_when:
  - Você quer executar o OpenClaw com modelos locais via Ollama
  - Você precisa de orientação de configuração e setup do Ollama
title: "Ollama"
---

# Ollama

Ollama é um runtime local de LLM que facilita executar modelos open-source na sua máquina. O OpenClaw se integra à API compatível com OpenAI do Ollama e pode **descobrir automaticamente modelos com suporte a ferramentas** quando você opta por `OLLAMA_API_KEY` (ou um perfil de autenticação) e não define uma entrada explícita de `models.providers.ollama`.

## Início rápido

1. Instale o Ollama: [https://ollama.ai](https://ollama.ai)

2. Baixe um modelo:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Ative o Ollama para o OpenClaw (qualquer valor funciona; o Ollama não exige uma chave real):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Use modelos do Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Descoberta de modelos (provedor implícito)

Quando você define `OLLAMA_API_KEY` (ou um perfil de autenticação) e **não** define `models.providers.ollama`, o OpenClaw descobre modelos a partir da instância local do Ollama em `http://127.0.0.1:11434`:

- Consulta `/api/tags` e `/api/show`
- Mantém apenas modelos que reportam capacidade `tools`
- Marca `reasoning` quando o modelo reporta `thinking`
- Lê `contextWindow` de `model_info["<arch>.context_length"]` quando disponível
- Define `maxTokens` como 10× a janela de contexto
- Define todos os custos como `0`

Isso evita entradas manuais de modelos, mantendo o catálogo alinhado com as capacidades do Ollama.

Para ver quais modelos estão disponíveis:

```bash
ollama list
openclaw models list
```

Para adicionar um novo modelo, basta baixá-lo com o Ollama:

```bash
ollama pull mistral
```

O novo modelo será descoberto automaticamente e ficará disponível para uso.

Se você definir `models.providers.ollama` explicitamente, a descoberta automática é ignorada e você deve definir os modelos manualmente (veja abaixo).

## Configuração

### Configuração básica (descoberta implícita)

A forma mais simples de ativar o Ollama é via variável de ambiente:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Configuração explícita (modelos manuais)

Use configuração explícita quando:

- O Ollama roda em outro host/porta.
- Você quer forçar janelas de contexto ou listas de modelos específicas.
- Você quer incluir modelos que não reportam suporte a ferramentas.

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

Se `OLLAMA_API_KEY` estiver definido, você pode omitir `apiKey` na entrada do provedor e o OpenClaw irá preenchê-lo para verificações de disponibilidade.

### URL base personalizada (configuração explícita)

Se o Ollama estiver rodando em um host ou porta diferentes (a configuração explícita desativa a descoberta automática, então defina os modelos manualmente):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### Seleção de modelos

Depois de configurado, todos os seus modelos do Ollama ficam disponíveis:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## Avançado

### Modelos de raciocínio

O OpenClaw marca modelos como capazes de raciocínio quando o Ollama reporta `thinking` em `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Custos de modelos

O Ollama é gratuito e roda localmente, então todos os custos de modelos são definidos como $0.

### Configuração de streaming

A integração do Ollama no OpenClaw usa a **API nativa do Ollama** (`/api/chat`) por padrão, que oferece suporte completo a streaming e chamada de ferramentas simultaneamente. Nenhuma configuração especial é necessária.

#### Modo legado compatível com OpenAI

Se você precisar usar o endpoint compatível com OpenAI em vez disso (por exemplo, atrás de um proxy que suporte apenas o formato OpenAI), defina `api: "openai-completions"` explicitamente:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

Observação: O endpoint compatível com OpenAI pode não oferecer suporte a streaming + chamada de ferramentas simultaneamente. Pode ser necessário desativar o streaming com `params: { streaming: false }` na configuração do modelo.

### Janelas de contexto

Para modelos descobertos automaticamente, o OpenClaw usa a janela de contexto reportada pelo Ollama quando disponível; caso contrário, o padrão é `8192`. Você pode sobrescrever `contextWindow` e `maxTokens` na configuração explícita do provedor.

## Solução de problemas

### Ollama não detectado

Certifique-se de que o Ollama esteja em execução e que você tenha definido `OLLAMA_API_KEY` (ou um perfil de autenticação), e que você **não** tenha definido uma entrada explícita de `models.providers.ollama`:

```bash
ollama serve
```

E que a API esteja acessível:

```bash
curl http://localhost:11434/api/tags
```

### Nenhum modelo disponível

O OpenClaw só descobre automaticamente modelos que reportam suporte a ferramentas. Se o seu modelo não estiver listado, você pode:

- Baixar um modelo com suporte a ferramentas, ou
- Definir o modelo explicitamente em `models.providers.ollama`.

Para adicionar modelos:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Conexão recusada

Verifique se o Ollama está rodando na porta correta:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Veja também

- [Model Providers](/concepts/model-providers) - Visão geral de todos os provedores
- [Model Selection](/concepts/models) - Como escolher modelos
- [Configuration](/gateway/configuration) - Referência completa de configuração

