---
title: "Skriptlar"
---

# Skriptlar

`scripts/` katalogida lokal ish jarayonlari va ops vazifalari uchun yordamchi skriptlar mavjud.
Use these when a task is clearly tied to a script; otherwise prefer the CLI.

## Konvensiyalar

- Skriptlar **ixtiyoriy** hisoblanadi, agar ular hujjatlarda yoki reliz tekshiruv ro‘yxatlarida keltirilmagan bo‘lsa.
- Mavjud bo‘lganda CLI interfeyslarini afzal ko‘ring (masalan: autentifikatsiya monitoringi uchun `openclaw models status --check` ishlatiladi).
- Skriptlar muayyan xostga moslangan deb hisoblang; ularni yangi mashinada ishga tushirishdan oldin o‘qib chiqing.

## Autentifikatsiya monitoringi skriptlari

Autentifikatsiya monitoringi skriptlari bu yerda hujjatlashtirilgan:
[/automation/auth-monitoring](/automation/auth-monitoring)

## Skript qo‘shishda

- Skriptlarni aniq maqsadga yo‘naltirilgan va hujjatlashtirilgan holda saqlang.
- Tegishli hujjatga qisqa yozuv qo‘shing (yoki mavjud bo‘lmasa, yangisini yarating).


