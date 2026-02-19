---
summary: "OpenClaw onboarding variantlari va jarayonlari haqida umumiy ma’lumot"
read_when:
  - Onboarding yo‘lini tanlash
  - Yangi muhitni sozlash
title: "Onboarding haqida umumiy ma’lumot"
sidebarTitle: "Onboarding haqida umumiy ma’lumot"
---

# Onboarding haqida umumiy ma’lumot

OpenClaw Gateway qayerda ishlashiga va providerlarni qanday sozlashni afzal ko‘rishingizga qarab bir nechta onboarding yo‘llarini qo‘llab-quvvatlaydi.

## Onboarding yo‘lingizni tanlang

- macOS, Linux va Windows (WSL2 orqali) uchun **CLI wizard**.
- Apple silicon yoki Intel Mac qurilmalarida birinchi ishga tushirish uchun **macOS app**.

## CLI onboarding wizard

Terminalda wizard’ni ishga tushiring:

```bash
openclaw onboard
```

Gateway, workspace,
kanallar va skills ustidan to‘liq nazoratni xohlasangiz, CLI wizard’dan foydalaning. Hujjatlar:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS app orqali onboarding

macOS’da to‘liq yo‘naltirilgan sozlashni xohlasangiz, OpenClaw ilovasidan foydalaning. Hujjatlar:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

Agar ro‘yxatda bo‘lmagan endpoint kerak bo‘lsa, jumladan standart OpenAI yoki Anthropic API’larini taqdim etuvchi hosted providerlar bo‘lsa, CLI wizard’da **Custom Provider** ni tanlang. Sizdan quyidagilar so‘raladi:

- OpenAI-compatible, Anthropic-compatible yoki **Unknown** (avto-aniqlash) variantini tanlash.
- Base URL va API key’ni kiritish (agar provider talab qilsa).
- Model ID va ixtiyoriy alias ko‘rsatish.
- Bir nechta custom endpoint birga ishlashi uchun Endpoint ID tanlash.

Batafsil bosqichlar uchun yuqoridagi CLI onboarding hujjatlariga murojaat qiling.

