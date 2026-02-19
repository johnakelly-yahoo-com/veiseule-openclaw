---
summary: "So‘rovnomani gateway + CLI orqali yuborish"
read_when:
  - Adding or modifying poll support
  - Debugging poll sends from the CLI or gateway
title: "So‘rovnomalar"
---

# So‘rovnomalar

## Qo‘llab-quvvatlanadigan kanallar

- WhatsApp (web channel)
- Discord
- MS Teams (Adaptive Cards)

## CLI

```bash
# WhatsApp
openclaw message poll --target +15555550123 \
  --poll-question "Lunch today?" --poll-option "Yes" --poll-option "No" --poll-option "Maybe"
openclaw message poll --target 123456789@g.us \
  --poll-question "Meeting time?" --poll-option "10am" --poll-option "2pm" --poll-option "4pm" --poll-multi

# Discord
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Snack?" --poll-option "Pizza" --poll-option "Sushi"
openclaw message poll --channel discord --target channel:123456789 \
  --poll-question "Plan?" --poll-option "A" --poll-option "B" --poll-duration-hours 48

# MS Teams
openclaw message poll --channel msteams --target conversation:19:abc@thread.tacv2 \
  --poll-question "Lunch?" --poll-option "Pizza" --poll-option "Sushi"
```

Variantlar:

- `--channel`: `whatsapp` (standart), `discord` yoki `msteams`
- `--poll-multi`: bir nechta variantni tanlashga ruxsat berish
- `--poll-duration-hours`: Discord-only (defaults to 24 when omitted)

## Gateway RPC

Method: `poll`

Params:

- `to` (string, required)
- `question` (string, required)
- `options` (string[], required)
- `maxSelections` (son, ixtiyoriy)
- `durationHours` (son, ixtiyoriy)
- `channel` (satr, ixtiyoriy, sukut bo‘yicha: `whatsapp`)
- `idempotencyKey` (satr, majburiy)

## Kanal farqlari

- WhatsApp: 2–12 ta variant, `maxSelections` variantlar soni doirasida bo‘lishi kerak, `durationHours` e’tiborga olinmaydi.
- Discord: 2–10 ta variant, `durationHours` 1–768 soat oralig‘iga cheklanadi (sukut bo‘yicha 24). `maxSelections > 1` ko‘p tanlashni yoqadi; Discord qat’iy tanlash sonini qo‘llab-quvvatlamaydi.
- 23. MS Teams: Adaptive Card so‘rovlari (OpenClaw tomonidan boshqariladi). Mahalliy poll API yo‘q; `durationHours` e’tiborga olinmaydi.

## Agent vositasi (Message)

`message` vositasidan `poll` amalini ishlating (`to`, `pollQuestion`, `pollOption`, ixtiyoriy `pollMulti`, `pollDurationHours`, `channel`).

Eslatma: Discord’da “aniq N ta tanlash” rejimi yo‘q; `pollMulti` ko‘p tanlashga mos keladi.
Teams so‘rovlari Adaptive Card sifatida render qilinadi va ovozlarni `~/.openclaw/msteams-polls.json` fayliga yozib olish uchun gateway doim onlayn bo‘lishi kerak.

