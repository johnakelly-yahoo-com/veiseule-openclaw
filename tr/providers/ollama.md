---
summary: "OpenClaw’ı Ollama ile çalıştırın (yerel LLM çalışma zamanı)"
read_when:
  - OpenClaw’ı Ollama üzerinden yerel modellerle çalıştırmak istiyorsanız
  - Ollama kurulumu ve yapılandırma rehberine ihtiyaç duyuyorsanız
title: "Ollama"
---

# Ollama

Ollama, açık kaynaklı modelleri makinenizde çalıştırmayı kolaylaştıran yerel bir LLM çalışma zamanıdır. OpenClaw, Ollama’nın OpenAI uyumlu API’siyle entegre olur ve `OLLAMA_API_KEY` (veya bir yetkilendirme profili) ile etkinleştirdiğinizde ve açık bir `models.providers.ollama` girdisi tanımlamadığınızda **araç destekli modelleri otomatik olarak keşfedebilir**.

## Hızlı Başlangıç

1. Ollama’yı yükleyin: [https://ollama.ai](https://ollama.ai)

2. Bir model çekin:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. OpenClaw için Ollama’yı etkinleştirin (herhangi bir değer yeterlidir; Ollama gerçek bir anahtar gerektirmez):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama modellerini kullanın:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Model keşfi (örtük sağlayıcı)

`OLLAMA_API_KEY` (veya bir yetkilendirme profili) ayarlandığında ve **`models.providers.ollama` tanımlanmadığında**, OpenClaw yerel Ollama örneğinden `http://127.0.0.1:11434` adresinde modelleri keşfeder:

- `/api/tags` ve `/api/show` sorgulanır
- Yalnızca `tools` yeteneğini bildiren modeller tutulur
- Model `thinking` bildirdiğinde `reasoning` olarak işaretlenir
- Mevcut olduğunda `model_info["<arch>.context_length"]` içinden `contextWindow` okunur
- `maxTokens`, bağlam penceresinin 10×’u olarak ayarlanır
- Tüm maliyetler `0` olarak ayarlanır

Bu, katalogu Ollama'nın yetenekleriyle uyumlu tutarken manuel model girişlerinden kaçınır.

Hangi modellerin mevcut olduğunu görmek için:

```bash
ollama list
openclaw models list
```

Yeni bir model eklemek için Ollama ile çekmeniz yeterlidir:

```bash
ollama pull mistral
```

Yeni model otomatik olarak keşfedilir ve kullanıma hazır olur.

`models.providers.ollama` açıkça ayarlanırsa, otomatik keşif atlanır ve modelleri manuel olarak tanımlamanız gerekir (aşağıya bakın).

## Yapılandırma

### Temel kurulum (örtük keşif)

Ollama’yı etkinleştirmenin en basit yolu ortam değişkenidir:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Açık kurulum (manuel modeller)

Aşağıdaki durumlarda açık yapılandırma kullanın:

- Ollama başka bir ana makine/port üzerinde çalışıyorsa.
- Belirli bağlam pencerelerini veya model listelerini zorlamak istiyorsanız.
- Araç desteği bildirmeyen modelleri dahil etmek istiyorsanız.

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

`OLLAMA_API_KEY` ayarlıysa, sağlayıcı girdisinde `apiKey`’yi atlayabilirsiniz ve OpenClaw bunu kullanılabilirlik denetimleri için doldurur.

### Özel temel URL (açık yapılandırma)

Ollama farklı bir ana makine veya portta çalışıyorsa (açık yapılandırma otomatik keşfi devre dışı bırakır, bu nedenle modelleri manuel olarak tanımlayın):

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

### Model seçimi

Yapılandırmadan sonra tüm Ollama modelleriniz kullanılabilir:

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

## Gelişmiş

### Akıl yürütme modelleri

Ollama, `/api/show` içinde `thinking` bildirdiğinde OpenClaw modelleri akıl yürütme yetenekli olarak işaretler:

```bash
ollama pull deepseek-r1:32b
```

### Model Maliyetleri

Ollama ücretsizdir ve yerel olarak çalışır; bu nedenle tüm model maliyetleri $0 olarak ayarlanır.

### Akış Yapılandırması

OpenClaw’un Ollama entegrasyonu varsayılan olarak **yerel Ollama API**’sini (`/api/chat`) kullanır; bu, aynı anda hem streaming hem de tool calling’i tam olarak destekler. Özel bir yapılandırma gerekmez.

#### Eski OpenAI-Uyumlu Mod

Bunun yerine OpenAI-uyumlu endpoint’i kullanmanız gerekiyorsa (örneğin yalnızca OpenAI formatını destekleyen bir proxy arkasında), `api: "openai-completions"` değerini açıkça ayarlayın:

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

Not: OpenAI-uyumlu endpoint aynı anda streaming + tool calling’i desteklemeyebilir. Ollama modelleri için `streaming: false`’ü açıkça ayarlayın (bkz. [Akış Yapılandırması](#akış-yapılandırması))

### Bağlam pencereleri

Otomatik keşfedilen modeller için OpenClaw, mevcut olduğunda Ollama’nın bildirdiği bağlam penceresini kullanır; aksi halde varsayılan olarak `8192` değerini kullanır. Açık sağlayıcı yapılandırmasında `contextWindow` ve `maxTokens` değerlerini geçersiz kılabilirsiniz.

## Sorun Giderme

### Ollama algılanmıyor

Ollama’nın çalıştığından ve `OLLAMA_API_KEY` (veya bir yetkilendirme profili) ayarladığınızdan ve **açık bir `models.providers.ollama` girdisi tanımlamadığınızdan** emin olun:

```bash
ollama serve
```

Ayrıca API’nin erişilebilir olduğundan emin olun:

```bash
curl http://localhost:11434/api/tags
```

### Hiç model yok

OpenClaw yalnızca araç desteği bildiren modelleri otomatik olarak keşfeder. Modeliniz listelenmiyorsa:

- Araç destekli bir model çekin veya
- Modeli `models.providers.ollama` içinde açıkça tanımlayın.

Model eklemek için:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Bağlantı reddedildi

Ollama’nın doğru portta çalıştığını kontrol edin:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Ayrıca Bakınız

- [Model Providers](/concepts/model-providers) - Tüm sağlayıcıların genel bakışı
- [Model Selection](/concepts/models) - Modeller nasıl seçilir
- [Configuration](/gateway/configuration) - Tam yapılandırma başvurusu
