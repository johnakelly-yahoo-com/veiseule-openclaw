---
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
  - Z.AI (`zai/*`) faqat ikkilik fikrlashni qo‘llab-quvvatlaydi (`on`/`off`). `off` dan boshqa har qanday daraja `on` sifatida qabul qilinadi (`low` ga moslanadi).

## Aniqlash tartibi

1. Xabardagi inline direktiva (faqat o‘sha xabarga qo‘llanadi).
2. Sessiya override’i (faqat direktivadan iborat xabar yuborish orqali o‘rnatiladi).
3. Global standart (`config` ichidagi `agents.defaults.thinkingDefault`).
4. Zaxira holat: fikrlash imkoniyatiga ega modellar uchun `low`; boshqalar uchun `off`.

## Sessiya standartini o‘rnatish

- **Faqat** direktivadan iborat xabar yuboring (bo‘sh joylarga ruxsat beriladi), masalan: `/think:medium` yoki `/t high`.
- Bu joriy sessiya uchun saqlanadi (standart bo‘yicha har bir yuboruvchi uchun); `/think:off` yoki sessiya idle reset orqali bekor qilinadi.
- Tasdiqlovchi javob yuboriladi (`Thinking level set to high.` / `Thinking disabled.`). Agar daraja noto‘g‘ri bo‘lsa (masalan, `/thinking big`), buyruq rad etiladi va maslahat beriladi, sessiya holati o‘zgarmaydi.
- Joriy fikrlash darajasini ko‘rish uchun `/think` (yoki `/think:`) ni argumentsiz yuboring.

## Agent tomonidan qo‘llanishi

- **Embedded Pi**: aniqlangan daraja in-process Pi agent runtime’iga uzatiladi.

## Verbose direktivalari (/verbose yoki /v)

- Darajalar: `on` (minimal) | `full` | `off` (standart).
- Faqat direktivadan iborat xabar sessiya verbose holatini o‘zgartiradi va `Verbose logging enabled.` / `Verbose logging disabled.` javobini qaytaradi; noto‘g‘ri darajalar holatni o‘zgartirmasdan maslahat qaytaradi.
- `/verbose off` aniq sessiya override’ini saqlaydi; uni Sessions UI orqali `inherit` ni tanlab tozalash mumkin.
- Inline direktiva faqat o‘sha xabarga ta’sir qiladi; aks holda sessiya/global standartlar qo‘llanadi.
- Joriy verbose darajasini ko‘rish uchun `/verbose` (yoki `/verbose:`) ni argumentsiz yuboring.
- Verbose yoqilganda, strukturalangan tool natijalarini chiqaradigan agentlar (Pi, boshqa JSON agentlari) har bir tool chaqiruvini alohida, faqat metadata’dan iborat xabar sifatida yuboradi; mavjud bo‘lsa `<emoji> <tool-name>: <arg>` (yo‘l/buyruq) prefiksi bilan. Bu tool xulosalari har bir tool boshlanishi bilan darhol yuboriladi (alohida bubble), streaming delta sifatida emas.
- Verbose `full` bo‘lsa, tool natijalari yakunlangandan keyin ham yuboriladi (alohida bubble, xavfsiz uzunlikkacha qisqartirilgan). Agar `/verbose on|full|off` ni jarayon davomida o‘zgartirsangiz, keyingi tool xabarlari yangi sozlamaga amal qiladi.

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

- Heartbeat probe body — sozlangan heartbeat prompt’idir (standart: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Heartbeat xabaridagi inline direktivalar odatdagidek qo‘llanadi (ammo heartbeat orqali sessiya standartlarini o‘zgartirishdan saqlaning).
- Heartbeat yuborilishi standart bo‘yicha faqat yakuniy payload’ni o‘z ichiga oladi. Alohida `Reasoning:` xabarini ham yuborish uchun (mavjud bo‘lsa), `agents.defaults.heartbeat.includeReasoning: true` yoki har bir agent uchun `agents.list[].heartbeat.includeReasoning: true` ni sozlang.

## Veb chat interfeysi

- Web chat’dagi fikrlash tanlagichi sahifa yuklanganda kiruvchi sessiya store/config’da saqlangan sessiya darajasini aks ettiradi.
- Boshqa darajani tanlash faqat keyingi xabarga qo‘llanadi (`thinkingOnce`); yuborilgandan so‘ng tanlagich saqlangan sessiya darajasiga qaytadi.
- Sessiya standartini o‘zgartirish uchun avvalgidek `/think:<level>` direktivasini yuboring; keyingi yuklashdan so‘ng tanlagich buni aks ettiradi.
