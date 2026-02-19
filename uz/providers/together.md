---
summary: "Together AI sozlamasi (auth + model tanlash)"
read_when:
  - Siz OpenClaw bilan Together AI’dan foydalanmoqchisiz
  - Sizga API key environment o‘zgaruvchisi yoki CLI orqali auth tanlovi kerak
---

# Together AI

[Together AI](https://together.ai) yagona API orqali Llama, DeepSeek, Kimi va boshqa yetakchi open-source modellariga kirishni ta’minlaydi.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-compatible

## Tezkor boshlash

1. API key’ni o‘rnating (tavsiya etiladi: uni Gateway uchun saqlang):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Sukut bo‘yicha modelni belgilang:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Non-interactive misol

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Bu `together/moonshotai/Kimi-K2.5` modelini sukut bo‘yicha model sifatida o‘rnatadi.

## Environment haqida eslatma

Agar Gateway daemon (launchd/systemd) sifatida ishlayotgan bo‘lsa, `TOGETHER_API_KEY`
ushbu jarayon uchun mavjud ekanligiga ishonch hosil qiling (masalan, `~/.clawdbot/.env` ichida yoki
`env.shellEnv` orqali).

## Mavjud modellar

Together AI ko‘plab mashhur open-source modellarga kirishni ta’minlaydi:

- **GLM 4.7 Fp8** - 200K kontekst oynasiga ega sukut bo‘yicha model
- **Llama 3.3 70B Instruct Turbo** - Tez va samarali ko‘rsatma bajarish
- **Llama 4 Scout** - Tasvirni tushunish imkoniyatiga ega vision modeli
- **Llama 4 Maverick** - Ilg‘or vision va reasoning
- **DeepSeek V3.1** - Kuchli kod yozish va reasoning modeli
- **DeepSeek R1** - Ilg‘or reasoning modeli
- **Kimi K2 Instruct** - 262K kontekst oynasiga ega yuqori samarali model

Barcha modellar standart chat completions’ni qo‘llab-quvvatlaydi va OpenAI API bilan mos keladi.
