---
title: "Avtomatlashtirishni nosozliklardan chiqarish"
---

# Avtomatlashtirishni nosozliklardan chiqarish

Rejalashtiruvchi va yetkazib berish muammolari uchun ushbu sahifadan foydalaning (`cron` + `heartbeat`).

## Buyruqlar zinapoyasi

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

So‘ng avtomatlashtirish tekshiruvlarini ishga tushiring:

```bash
openclaw cron status
openclaw cron list
openclaw system heartbeat last
```

## Cron ishga tushmayapti

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

Yaxshi chiqish quyidagicha bo‘ladi:

- `cron status` yoqilganini va kelajakdagi `nextWakeAtMs` ni ko‘rsatadi.
- Vazifa yoqilgan va haqiqiy jadval/vaqt mintaqasiga ega.
- `cron runs` `ok` yoki aniq o‘tkazib yuborish sababini ko‘rsatadi.

Keng tarqalgan belgilar:

- `cron: scheduler disabled; jobs will not run automatically` → cron konfiguratsiya/muhitda o‘chirilgan.
- `cron: timer tick failed` → rejalashtiruvchi tigi ishdan chiqqan; atrofdagi stack/log kontekstini tekshiring.
- Run chiqishida `reason: not-due` → qo‘lda ishga tushirish `--force` siz chaqirilgan va vazifa hali vaqti kelmagan.

## Cron ishga tushdi, ammo yetkazib berish yo‘q

```bash
openclaw cron runs --id <jobId> --limit 20
openclaw cron list
openclaw channels status --probe
openclaw logs --follow
```

Yaxshi chiqish quyidagicha bo‘ladi:

- Run holati `ok`.
- Izolyatsiyalangan vazifalar uchun yetkazib berish rejimi/maqsadi sozlangan.
- Kanal probe hisobotida maqsad kanal ulanganligi ko‘rsatiladi.

Keng tarqalgan belgilar:

- Run muvaffaqiyatli, ammo yetkazib berish rejimi `none` → tashqi xabar kutilmaydi.
- Yetkazib berish maqsadi yo‘q/yaroqsiz (`channel`/`to`) → run ichki muvaffaqiyatli bo‘lishi mumkin, ammo tashqi yuborish o‘tkazib yuboriladi.
- Kanal autentifikatsiya xatolari (`unauthorized`, `missing_scope`, `Forbidden`) → yetkazib berish kanal hisob ma’lumotlari/ruxsatlari sababli bloklangan.

## Heartbeat bostirilgan yoki o‘tkazib yuborilgan

```bash
openclaw system heartbeat last
openclaw logs --follow
openclaw config get agents.defaults.heartbeat
openclaw channels status --probe
```

Yaxshi chiqish quyidagicha bo‘ladi:

- Heartbeat nol bo‘lmagan interval bilan yoqilgan.
- So‘nggi heartbeat natijasi `ran` (yoki o‘tkazib yuborish sababi tushunarli).

1. Umumiy imzolar:

- 2. `heartbeat skipped` va `reason=quiet-hours` → `activeHours` tashqarisida.
- 3. `requests-in-flight` → asosiy yo‘lak band; heartbeat kechiktiriladi.
- 4. `empty-heartbeat-file` → `HEARTBEAT.md` mavjud, lekin bajariladigan mazmun yo‘q.
- 5. `alerts-disabled` → ko‘rinish sozlamalari tashqi heartbeat xabarlarini bostiradi.

## 6. Vaqt mintaqasi va activeHours bilan bog‘liq tuzoqlar

```bash
7. openclaw config get agents.defaults.heartbeat.activeHours
openclaw config get agents.defaults.heartbeat.activeHours.timezone
openclaw config get agents.defaults.userTimezone || echo "agents.defaults.userTimezone not set"
openclaw cron list
openclaw logs --follow
```

8. Tezkor qoidalar:

- 9. `Config path not found: agents.defaults.userTimezone` kalit o‘rnatilmaganini anglatadi; heartbeat xost vaqt mintaqasiga (yoki o‘rnatilgan bo‘lsa `activeHours.timezone`) qaytadi.
- 10. `--tz`siz cron shlyuz xostining vaqt mintaqasidan foydalanadi.
- 11. Heartbeat `activeHours` sozlangan vaqt mintaqasi yechimidan foydalanadi (`user`, `local` yoki aniq IANA tz).
- 12. Vaqt mintaqasisiz ISO vaqt belgilari cron `at` jadvali uchun UTC sifatida qabul qilinadi.

24. Keng tarqalgan imzolar:

- 14. Xost vaqt mintaqasi o‘zgargandan so‘ng ishlar noto‘g‘ri devor-soat vaqtida ishga tushadi.
- 15. `activeHours.timezone` noto‘g‘ri bo‘lgani uchun heartbeat kunduzgi vaqtingizda doim o‘tkazib yuboriladi.

16. Bog‘liq:

- 17. [/automation/cron-jobs](/automation/cron-jobs)
- 18. [/gateway/heartbeat](/gateway/heartbeat)
- 19. [/automation/cron-vs-heartbeat](/automation/cron-vs-heartbeat)
- 20. [/concepts/timezone](/concepts/timezone)
