---
summary: "cron.add kirishlarini qayta ishlashni mustahkamlash, sxemalarni moslashtirish va cron UI/agent vositalarini yaxshilash"
owner: "openclaw"
status: "tugallangan"
last_updated: "2026-01-05"
title: "Cron qo‘shish mustahkamlanishi"
---

# Cron qo‘shish mustahkamlanishi va sxema moslashtirilishi

## Kontekst

So‘nggi gateway loglari noto‘g‘ri parametrlar (`sessionTarget`, `wakeMode`, `payload` yo‘qligi va noto‘g‘ri `schedule`) sababli takroriy `cron.add` xatoliklarini ko‘rsatmoqda. Bu kamida bitta mijoz (ehtimol agent vositasi chaqiruv yo‘li) o‘ralgan yoki qisman aniqlangan job payloadlarini yuborayotganini ko‘rsatadi. Bundan tashqari, TypeScript’dagi cron provider enumlari, gateway sxemasi, CLI flaglari va UI forma turlari o‘rtasida nomuvofiqlik mavjud, shuningdek UI’da `cron.status` nomuvofiqligi bor (`jobCount` kutiladi, gateway esa `jobs` qaytaradi).

## Maqsadlar

- `cron.add` INVALID_REQUEST spamini umumiy o‘ram payloadlarini normallashtirish va yetishmayotgan `kind` maydonlarini aniqlash orqali to‘xtatish.
- Cron provider ro‘yxatlarini gateway sxemasi, cron turlari, CLI hujjatlari va UI formalari bo‘ylab moslashtirish.
- LLM to‘g‘ri job payloadlarini ishlab chiqishi uchun agent cron vositasi sxemasini aniq qilish.
- Control UI’da cron statusidagi job soni ko‘rinishini tuzatish.
- Normallashtirish va vosita xatti-harakatlarini qamrab oluvchi testlar qo‘shish.

## Maqsad emas

- Cron rejalashtirish semantikasi yoki job bajarilish xatti-harakatlarini o‘zgartirish.
- Yangi schedule turlarini yoki cron ifodalarini tahlil qilishni qo‘shish.
- Zarur maydon tuzatishlaridan tashqari cron uchun UI/UX’ni to‘liq qayta ishlash.

## Topilmalar (joriy bo‘shliqlar)

- Gateway’dagi `CronPayloadSchema` `signal` va `imessage` ni istisno qiladi, TS turlari esa ularni o‘z ichiga oladi.
- Control UI CronStatus `jobCount` ni kutadi, gateway esa `jobs` qaytaradi.
- Agent cron vositasi sxemasi ixtiyoriy `job` obyektlariga ruxsat beradi, bu esa noto‘g‘ri kiritmalarga imkon yaratadi.
- Gateway `cron.add` ni normallashtirishsiz qat’iy tekshiradi, shu sababli o‘ralgan payloadlar muvaffaqiyatsiz bo‘ladi.

## Nima o‘zgardi

- `cron.add` va `cron.update` endi umumiy o‘ram shakllarini normallashtiradi va yetishmayotgan `kind` maydonlarini aniqlaydi.
- Agent cron vositasi sxemasi gateway sxemasiga moslashtirildi, bu noto‘g‘ri payloadlarni kamaytiradi.
- Provider enumlari gateway, CLI, UI va macOS tanlagichi bo‘ylab moslashtirildi.
- Control UI status uchun gateway’ning `jobs` soni maydonidan foydalanadi.

## Joriy xatti-harakat

- **Normallashtirish:** o‘ralgan `data`/`job` payloadlari ochiladi; `schedule.kind` va `payload.kind` xavfsiz bo‘lganda aniqlanadi.
- **Standartlar:** `wakeMode` va `sessionTarget` yo‘q bo‘lsa, xavfsiz standartlar qo‘llaniladi.
- **Providerlar:** Discord/Slack/Signal/iMessage endi CLI/UI bo‘ylab izchil ko‘rsatiladi.

Normallashtirilgan shakl va misollar uchun [Cron jobs](/automation/cron-jobs) ga qarang.

## Tekshiruv

- Gateway loglarida `cron.add` INVALID_REQUEST xatolarining kamayishini kuzating.
- Yangilashdan so‘ng Control UI cron statusida job soni ko‘rsatilishini tasdiqlang.

## Ixtiyoriy keyingi ishlar

- Manual Control UI smoke: har bir provider uchun cron job qo‘shing va statusdagi job sonini tekshiring.

## Ochiq savollar

- `cron.add` mijozlardan aniq `state` ni qabul qilishi kerakmi (hozirda sxema bo‘yicha ruxsat etilmagan)?
- `webchat` ni aniq yetkazib berish provideri sifatida ruxsat berishimiz kerakmi (hozirda yetkazib berishni aniqlashda filtrlanadi)?

