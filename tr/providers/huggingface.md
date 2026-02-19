---
summary: "Hugging Face Inference kurulumu (kimlik doğrulama + model seçimi)"
read_when:
  - OpenClaw ile Hugging Face Inference kullanmak istiyorsunuz
  - HF token ortam değişkenine veya CLI kimlik doğrulama seçimine ihtiyacınız var
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers), tek bir router API üzerinden OpenAI uyumlu chat completions sunar. Tek bir token ile birçok modele (DeepSeek, Llama ve daha fazlası) erişim elde edersiniz. OpenClaw **OpenAI uyumlu endpoint’i** (yalnızca chat completions) kullanır; metinden görsele, embeddings veya konuşma için doğrudan [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) kullanın.

- Sağlayıcı: `huggingface`
- Kimlik Doğrulama: `HUGGINGFACE_HUB_TOKEN` veya `HF_TOKEN` (**Make calls to Inference Providers** iznine sahip ayrıntılı token)
- API: OpenAI uyumlu (`https://router.huggingface.co/v1`)
- Faturalama: Tek bir HF token; [fiyatlandırma](https://huggingface.co/docs/inference-providers/pricing) sağlayıcı tarifelerine göre yapılır ve ücretsiz bir katman içerir.

## Hızlı başlangıç

1. [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) adresinden **Make calls to Inference Providers** izniyle ayrıntılı bir token oluşturun.
2. Onboarding’i çalıştırın ve sağlayıcı açılır listesinden **Hugging Face** seçin, ardından istendiğinde API anahtarınızı girin:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** açılır listesinden istediğiniz modeli seçin (geçerli bir token’ınız varsa liste Inference API’den yüklenir; aksi halde yerleşik bir liste gösterilir). Seçiminiz varsayılan model olarak kaydedilir.
4. Varsayılan modeli daha sonra config içinde de ayarlayabilir veya değiştirebilirsiniz:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Etkileşimsiz örnek

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Bu işlem, varsayılan model olarak `huggingface/deepseek-ai/DeepSeek-R1` ayarlar.

## Ortam notu

Gateway bir daemon (launchd/systemd) olarak çalışıyorsa, `HUGGINGFACE_HUB_TOKEN` veya `HF_TOKEN` değişkeninin
bu süreç için erişilebilir olduğundan emin olun (örneğin `~/.openclaw/.env` içinde veya
`env.shellEnv` aracılığıyla).

## Model keşfi ve onboarding açılır listesi

OpenClaw, modelleri **Inference endpoint’ini doğrudan** çağırarak keşfeder:

```bash
GET https://router.huggingface.co/v1/models
```

(İsteğe bağlı: tam liste için `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` veya `$HF_TOKEN` gönderin; bazı endpoint’ler kimlik doğrulama olmadan alt küme döndürebilir.) Yanıt OpenAI tarzındadır `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Bir Hugging Face API anahtarı yapılandırdığınızda (onboarding sırasında, `HUGGINGFACE_HUB_TOKEN` veya `HF_TOKEN` aracılığıyla), OpenClaw mevcut chat-completion modellerini keşfetmek için bu GET isteğini kullanır. **Etkileşimli onboarding** sırasında, token’ınızı girdikten sonra, bu listeden (veya istek başarısız olursa yerleşik katalogdan) doldurulan bir **Varsayılan Hugging Face modeli** açılır menüsü görürsünüz. Çalışma zamanında (ör. Gateway başlatılırken), bir anahtar mevcutsa OpenClaw kataloğu yenilemek için tekrar **GET** `https://router.huggingface.co/v1/models` çağrısı yapar. Liste, bağlam penceresi ve maliyet gibi meta veriler için yerleşik bir katalogla birleştirilir. İstek başarısız olursa veya bir anahtar ayarlanmamışsa yalnızca yerleşik katalog kullanılır.

## Model adları ve düzenlenebilir seçenekler

- **API’den gelen ad:** API `name`, `title` veya `display_name` döndürdüğünde modelin görünen adı **GET /v1/models** üzerinden **hydrate edilir**; aksi takdirde model kimliğinden türetilir (ör. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Görünen adı geçersiz kılma:** CLI ve UI’da istediğiniz şekilde görünmesi için yapılandırmada model başına özel bir etiket ayarlayabilirsiniz:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Sağlayıcı / politika seçimi:** Yönlendiricinin arka ucu nasıl seçeceğini belirlemek için **model id** sonuna bir ek ekleyin:

  - **`:fastest`** — en yüksek aktarım hızı (yönlendirici seçer; sağlayıcı seçimi **kilitlidir** — etkileşimli arka uç seçici yoktur).
  - **`:cheapest`** — çıktı token başına en düşük maliyet (yönlendirici seçer; sağlayıcı seçimi **kilitlidir**).
  - **`:provider`** — belirli bir arka ucu zorla (ör. `:sambanova`, `:together`).

  **:cheapest** veya **:fastest** seçtiğinizde (ör. onboarding model açılır menüsünde), sağlayıcı kilitlenir: yönlendirici maliyet ya da hıza göre karar verir ve isteğe bağlı “belirli bir arka ucu tercih et” adımı gösterilmez. Bunları `models.providers.huggingface.models` içine ayrı girdiler olarak ekleyebilir veya `model.primary` değerini bu ek ile ayarlayabilirsiniz. Ayrıca varsayılan sıranızı [Inference Provider settings](https://hf.co/settings/inference-providers) bölümünde ayarlayabilirsiniz (ek yoksa = bu sıra kullanılır).

- **Yapılandırma birleştirme:** Yapılandırma birleştirildiğinde `models.providers.huggingface.models` içindeki mevcut girdiler (ör. `models.json` içinde) korunur. Dolayısıyla burada ayarladığınız özel `name`, `alias` veya model seçenekleri korunur.

## Model kimlikleri ve yapılandırma örnekleri

Model referansları `huggingface/<org>/<model>` biçimini kullanır (Hub tarzı kimlikler). Aşağıdaki liste **GET** `https://router.huggingface.co/v1/models` çıktısındandır; kataloğunuz daha fazlasını içerebilir.

**Örnek kimlikler (inference endpoint’ten):**

| Model                                  | Ref (`huggingface/` ile başlayın) |
| -------------------------------------- | ---------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                            |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                          |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                      |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                           |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                     |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                  |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                   |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                |
| GLM 4.7                | `zai-org/GLM-4.7`                                    |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                               |

Model kimliğine `:fastest`, `:cheapest` veya `:provider` (ör. `:together`, `:sambanova`) ekleyebilirsiniz. Varsayılan sıranızı [Inference Provider settings](https://hf.co/settings/inference-providers) bölümünde ayarlayın; tam liste için [Inference Providers](https://huggingface.co/docs/inference-providers) ve **GET** `https://router.huggingface.co/v1/models` adresine bakın.

### Tam yapılandırma örnekleri

**Qwen yedeğiyle birincil DeepSeek R1:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Varsayılan olarak Qwen, :cheapest ve :fastest varyantlarıyla:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**Alias’larla DeepSeek + Llama + GPT-OSS:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**:provider ile belirli bir backend’i zorlayın:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Politika son ekleriyle birden fazla Qwen ve DeepSeek modeli:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

