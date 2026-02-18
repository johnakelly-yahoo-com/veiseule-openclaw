---
title: "Vercel AI Gateway"
summary: "Vercel AI Gateway sozlamasi (auth + model tanlash)"
read_when:
  - Siz OpenClaw bilan Vercel AI Gateway’dan foydalanmoqchisiz
  - Sizga API kaliti env o‘zgaruvchisi yoki CLI autentifikatsiya tanlovi kerak
---

# Vercel AI Gateway

[Vercel AI Gateway](https://vercel.com/ai-gateway) yuzlab modellarga bitta endpoint orqali kirish imkonini beruvchi yagona API taqdim etadi.

- Provayder: `vercel-ai-gateway`
- Autentifikatsiya: `AI_GATEWAY_API_KEY`
- API: Anthropic Messages bilan mos

## Tezkor boshlash

1. API kalitini o‘rnating (tavsiya etiladi: uni Gateway uchun saqlang):

```bash
openclaw onboard --auth-choice ai-gateway-api-key
```

2. Standart modelni belgilang:

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.6" },
    },
  },
}
```

## Interaktiv bo‘lmagan misol

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## Muhit haqida eslatma

Agar Gateway daemon sifatida ishlasa (launchd/systemd), `AI_GATEWAY_API_KEY`
o‘sha jarayon uchun mavjud ekanligiga ishonch hosil qiling (masalan, `~/.openclaw/.env` ichida yoki
`env.shellEnv` orqali).
