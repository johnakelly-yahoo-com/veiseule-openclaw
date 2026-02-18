---
title: "Skills gating yoki o‘rnatish xatti-harakatlarini o‘zgartirish"
---

# Skills

Skills (macOS)

## macOS ilovasi OpenClaw skilllarini gateway orqali ko‘rsatadi; u skilllarni lokal tarzda tahlil qilmaydi.

- Maʼlumot manbai
- `skills.status` (gateway) barcha skilllarni, shuningdek moslik va yetishmayotgan talablarni qaytaradi
  (shu jumladan, bundle qilingan skilllar uchun allowlist bloklari).

## Talablar har bir `SKILL.md` dagi `metadata.openclaw.requires` dan olinadi.

- O‘rnatish amallari
- `metadata.openclaw.install` o‘rnatish variantlarini belgilaydi (brew/node/go/uv).
- Ilova installerlarni gateway hostda ishga tushirish uchun `skills.install` ni chaqiradi.

Bir nechta variant berilganda, gateway faqat bitta afzal installer-ni ko‘rsatadi
(agar mavjud bo‘lsa brew, aks holda `skills.install` dagi node manager, sukut bo‘yicha npm).
-------------------------------------------------------------------------------------------------------------------------------

- Env/API kalitlariIlova kalitlarni `~/.openclaw/openclaw.json` da `skills.entries.<skillKey>` ostida saqlaydi
- .

## `skills.update` `enabled`, `apiKey` va `env` ni yangilaydi.

- Masofaviy rejim

