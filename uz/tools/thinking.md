---
summary: "/think + /verbose direktivalari sintaksisi va ularning model fikrlashiga ta’siri"
read_when:
  - Fikrlash yoki verbose direktiva parsingini yoki standart sozlamalarni o‘zgartirishda
title: "Fikrlash darajalari"
---

# Fikrlash darajalari (/think direktivalari)

## Nima qiladi

- Har qanday kiruvchi xabarda inline direktiva: `/t <level>`, `/think:<level>`, yoki `/thinking <level>`.
- Darajalar (aliaslar): `off | minimal | low | medium | high | xhigh` (faqat GPT-5.2 + Codex modellari)
  - minimal → “think”
  - past → “think hard”
  - o‘rta → “think harder”
  - high → “ultrathink” (maksimal budjet)
  - xhigh → “ultrathink+” (faqat GPT-5.2 + Codex modellari)
  - `x-high`, `x_high`, `extra-high`, `extra high`, va `extra_high` `xhigh` ga mos keladi.
  - `highest`, `max` `high` ga mos keladi.
- Provayder eslatmalari:
  - Z.AI (`zai/*`) only supports binary thinking (`on`/`off`). Any non-`off` level is treated as `on` (mapped to `low`).

## Aniqlash tartibi

1. Xabardagi inline direktiva (faqat o‘sha xabarga qo‘llanadi).
2. Sessiya override’i (faqat direktivadan iborat xabar yuborish orqali o‘rnatiladi).
3. Global standart (`config` ichidagi `agents.defaults.thinkingDefault`).
4. Zaxira holat: fikrlash imkoniyatiga ega modellar uchun `low`; boshqalar uchun `off`.

## Sessiya standartini o‘rnatish

- **Faqat** direktivadan iborat xabar yuboring (bo‘sh joylarga ruxsat beriladi), masalan: `/think:medium` yoki `/t high`.
- Bu joriy sessiya uchun saqlanadi (standart bo‘yicha har bir yuboruvchi uchun); `/think:off` yoki sessiya idle reset orqali bekor qilinadi.
- Confirmation reply is sent (`Thinking level set to high.` / `Thinking disabled.`). If the level is invalid (e.g. `/thinking big`), the command is rejected with a hint and the session state is left unchanged.
- Joriy fikrlash darajasini ko‘rish uchun `/think` (yoki `/think:`) ni argumentsiz yuboring.

## Agent tomonidan qo‘llanishi

- **Embedded Pi**: aniqlangan daraja in-process Pi agent runtime’iga uzatiladi.

## Verbose direktivalari (/verbose yoki /v)

- Darajalar: `on` (minimal) | `full` | `off` (standart).
- Faqat direktivadan iborat xabar sessiya verbose holatini o‘zgartiradi va `Verbose logging enabled.` / `Verbose logging disabled.` javobini qaytaradi; noto‘g‘ri darajalar holatni o‘zgartirmasdan maslahat qaytaradi.
- `/verbose off` aniq sessiya override’ini saqlaydi; uni Sessions UI orqali `inherit` ni tanlab tozalash mumkin.
- Inline direktiva faqat o‘sha xabarga ta’sir qiladi; aks holda sessiya/global standartlar qo‘llanadi.
- Joriy verbose darajasini ko‘rish uchun `/verbose` (yoki `/verbose:`) ni argumentsiz yuboring.
- When verbose is on, agents that emit structured tool results (Pi, other JSON agents) send each tool call back as its own metadata-only message, prefixed with `<emoji> <tool-name>: <arg>` when available (path/command). These tool summaries are sent as soon as each tool starts (separate bubbles), not as streaming deltas.
- When verbose is `full`, tool outputs are also forwarded after completion (separate bubble, truncated to a safe length). If you toggle `/verbose on|full|off` while a run is in-flight, subsequent tool bubbles honor the new setting.

## Fikrlash ko‘rinishi (/reasoning)

- Darajalar: `on|off|stream`.
- Faqat direktivadan iborat xabar javoblardagi fikrlash bloklari ko‘rsatilishini yoqadi yoki o‘chiradi.
- Yoqilganda, fikrlash **alohida xabar** sifatida `Reasoning:` prefiksi bilan yuboriladi.
- `stream` (faqat Telegram): javob yaratilayotgan paytda fikrlashni Telegram draft bubble ichida oqim tarzida ko‘rsatadi, so‘ng yakuniy javobni fikrlashsiz yuboradi.
- Taxallus: `/reason`.
- Joriy fikrlash darajasini ko‘rish uchun `/reasoning` (yoki `/reasoning:`) ni argumentsiz yuboring.

## Bog‘liq

- Elevated mode hujjatlari [Elevated mode](/tools/elevated) sahifasida joylashgan.

## Heartbeat’lar

- Heartbeat probe tanasi sozlangan heartbeat promptidir (standart: `Read HEARTBEAT.md if it exists (workspace context). Bunga qatʼiy amal qiling. Oldingi chatlardan eski vazifalarni taxmin qilmang yoki takrorlamang. Agar eʼtibor talab qilinadigan narsa boʻlmasa, HEARTBEAT_OK.`) deb javob bering. Heartbeat xabaridagi inline direktivalar odatdagidek qoʻllaniladi (lekin heartbeatlar orqali sessiya standartlarini oʻzgartirishdan saqlaning).
- Heartbeat yetkazilishi standart holatda faqat yakuniy payload bilan cheklanadi. Alohida `Reasoning:` xabarini ham yuborish uchun (mavjud boʻlsa), `agents.defaults.heartbeat.includeReasoning: true` yoki har bir agent uchun `agents.list[].heartbeat.includeReasoning: true` ni sozlang.

## Veb chat interfeysi

- Web chat’dagi fikrlash tanlagichi sahifa yuklanganda kiruvchi sessiya store/config’da saqlangan sessiya darajasini aks ettiradi.
- Boshqa darajani tanlash faqat keyingi xabarga qo‘llanadi (`thinkingOnce`); yuborilgandan so‘ng tanlagich saqlangan sessiya darajasiga qaytadi.
- Sessiya standartini o‘zgartirish uchun avvalgidek `/think:<level>` direktivasini yuboring; keyingi yuklashdan so‘ng tanlagich buni aks ettiradi.

