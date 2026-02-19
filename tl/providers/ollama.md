---
summary: "Patakbuhin ang OpenClaw gamit ang Ollama (lokal na LLM runtime)"
read_when:
  - Gusto mong patakbuhin ang OpenClaw gamit ang mga lokal na model sa pamamagitan ng Ollama
  - Kailangan mo ng gabay sa setup at konpigurasyon ng Ollama
title: "Ollama"
---

# Ollama

Ollama is a local LLM runtime that makes it easy to run open-source models on your machine. Ang OpenClaw ay naka-integrate sa native API ng Ollama (`/api/chat`), na sumusuporta sa streaming at tool calling, at maaaring **awtomatikong tuklasin ang mga modelong may kakayahang gumamit ng tools** kapag nag-opt in ka gamit ang `OLLAMA_API_KEY` (o isang auth profile) at hindi nagde-define ng tahasang `models.providers.ollama` entry.

## Mabilis na pagsisimula

1. I-install ang Ollama: [https://ollama.ai](https://ollama.ai)

2. Mag-pull ng model:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. I-enable ang Ollama para sa OpenClaw (kahit anong value ay gagana; hindi nangangailangan ang Ollama ng totoong key):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Gamitin ang mga Ollama model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Pagtuklas ng modelo (implicit na provider)

Kapag itinakda mo ang `OLLAMA_API_KEY` (o isang auth profile) at **hindi** nagde-define ng `models.providers.ollama`, dini-discover ng OpenClaw ang mga model mula sa lokal na Ollama instance sa `http://127.0.0.1:11434`:

- Kinukuwery ang `/api/tags` at `/api/show`
- Pinananatili lamang ang mga model na nag-uulat ng kakayahang `tools`
- Minamarkahan ang `reasoning` kapag nag-uulat ang model ng `thinking`
- Binabasa ang `contextWindow` mula sa `model_info["<arch>.context_length"]` kapag available
- Itinatakda ang `maxTokens` sa 10× ng context window
- Itinatakda ang lahat ng gastos sa `0`

Iniiwasan nito ang manu-manong paglalagay ng model habang pinananatiling naka-align ang catalog sa mga kakayahan ng Ollama.

Para makita kung anong mga model ang available:

```bash
ollama list
openclaw models list
```

Para magdagdag ng bagong model, i-pull lang ito gamit ang Ollama:

```bash
ollama pull mistral
```

Ang bagong model ay awtomatikong madi-discover at magiging handang gamitin.

Kung tahasan mong itinakda ang `models.providers.ollama`, lalaktawan ang auto-discovery at kailangan mong mag-define ng mga model nang manu-mano (tingnan sa ibaba).

## Konpigurasyon

### Pangunahing setup (implicit na pagtuklas)

Ang pinakasimpleng paraan para i-enable ang Ollama ay sa pamamagitan ng environment variable:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Tahasan na setup (manwal na mga modelo)

Gumamit ng explicit config kapag:

- Ang Ollama ay tumatakbo sa ibang host/port.
- Gusto mong pilitin ang partikular na context window o listahan ng model.
- Gusto mong isama ang mga model na hindi nag-uulat ng tool support.

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

Kung itinakda ang `OLLAMA_API_KEY`, maaari mong alisin ang `apiKey` sa provider entry at pupunan ito ng OpenClaw para sa mga availability check.

### Custom na base URL (tahasang config)

Kung tumatakbo ang Ollama sa ibang host o port (dinidisable ng explicit config ang auto-discovery, kaya manu-manong i-define ang mga model):

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

### Pagpili ng model

Kapag naka-configure na, available na ang lahat ng iyong Ollama model:

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

## Advanced

### Mga reasoning model

Minamarkahan ng OpenClaw ang mga model bilang may kakayahang mag-reason kapag nag-uulat ang Ollama ng `thinking` sa `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Mga Gastos ng Model

Libre ang Ollama at lokal na tumatakbo, kaya ang lahat ng gastos ng model ay nakatakda sa $0.

### Konpigurasyon ng Streaming

Ang integration ng OpenClaw sa Ollama ay gumagamit ng **native Ollama API** (`/api/chat`) bilang default, na ganap na sumusuporta sa streaming at tool calling nang sabay. Hindi kailangan ng espesyal na configuration.

#### Legacy OpenAI-Compatible Mode

Kung kailangan mong gamitin ang OpenAI-compatible endpoint sa halip (hal., nasa likod ng proxy na OpenAI format lang ang sinusuportahan), itakda nang tahasan ang `api: "openai-completions"`:

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

Tandaan: Maaaring hindi suportado ng OpenAI-compatible endpoint ang streaming + tool calling nang sabay. Maaaring kailanganin mong i-disable ang streaming gamit ang `params: { streaming: false }` sa model config.

### I-disable ang Streaming para sa Ibang Provider

For auto-discovered models, OpenClaw uses the context window reported by Ollama when available, otherwise it defaults to `8192`. You can override `contextWindow` and `maxTokens` in explicit provider config.

## Pag-troubleshoot

### Mga context window

Tiyaking tumatakbo ang Ollama at na itinakda mo ang `OLLAMA_API_KEY` (o isang auth profile), at **hindi** ka nag-define ng tahasang `models.providers.ollama` na entry:

```bash
ollama serve
```

At tiyaking naa-access ang API:

```bash
curl http://localhost:11434/api/tags
```

### Walang available na mga model

OpenClaw only auto-discovers models that report tool support. If your model isn't listed, either:

- Mag-pull ng model na may tool support, o
- I-define ang model nang tahasan sa `models.providers.ollama`.

Para magdagdag ng mga model:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Connection refused

Para magdagdag ng mga model:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Tingnan din

- [Model Providers](/concepts/model-providers) - Pangkalahatang-ideya ng lahat ng provider
- [Model Selection](/concepts/models) - Paano pumili ng mga model
- [Configuration](/gateway/configuration) - Buong reference ng config

