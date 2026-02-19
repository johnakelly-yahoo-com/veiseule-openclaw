---
summary: "Together AI kurulumu (kimlik doğrulama + model seçimi)"
read_when:
  - OpenClaw ile Together AI kullanmak istiyorsunuz
  - API anahtarı env değişkenine veya CLI kimlik doğrulama seçimine ihtiyacınız var
---

# Together AI

[Together AI](https://together.ai), Llama, DeepSeek, Kimi ve daha fazlası dahil olmak üzere önde gelen açık kaynak modellerine birleşik bir API üzerinden erişim sağlar.

- Sağlayıcı: `together`
- Kimlik Doğrulama: `TOGETHER_API_KEY`
- API: OpenAI-uyumlu

## Hızlı başlangıç

1. API anahtarını ayarlayın (önerilen: Gateway için saklayın):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Varsayılan bir model ayarlayın:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Etkileşimsiz örnek

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Bu, `together/moonshotai/Kimi-K2.5` modelini varsayılan model olarak ayarlayacaktır.

## Ortam notu

Gateway bir daemon (launchd/systemd) olarak çalışıyorsa, `TOGETHER_API_KEY` değişkeninin
bu süreç tarafından erişilebilir olduğundan emin olun (örneğin `~/.clawdbot/.env` içinde veya
`env.shellEnv` aracılığıyla).

## Mevcut modeller

Together AI, birçok popüler açık kaynak modele erişim sağlar:

- **GLM 4.7 Fp8** - 200K bağlam penceresine sahip varsayılan model
- **Llama 3.3 70B Instruct Turbo** - Hızlı ve verimli talimat takibi
- **Llama 4 Scout** - Görsel anlama yeteneğine sahip vision modeli
- **Llama 4 Maverick** - Gelişmiş görsel analiz ve akıl yürütme
- **DeepSeek V3.1** - Güçlü kodlama ve akıl yürütme modeli
- **DeepSeek R1** - Gelişmiş akıl yürütme modeli
- **Kimi K2 Instruct** - 262K bağlam penceresine sahip yüksek performanslı model

Tüm modeller standart sohbet tamamlama işlemlerini destekler ve OpenAI API ile uyumludur.
