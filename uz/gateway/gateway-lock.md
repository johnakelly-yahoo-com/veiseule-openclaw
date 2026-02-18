---
summary: "WebSocket tinglovchi bind orqali Gateway singleton himoyasi"
read_when:
  - Gateway jarayonini ishga tushirayotganda yoki nosozliklarni tahlil qilayotganda
  - Yagona instansiya majburiyligini tekshirayotganda
title: "Gateway qulfi"
---

# Gateway qulfi

Oxirgi yangilanish: 2025-12-11

## Nima uchun

- Bir xil xostda har bir asosiy port uchun faqat bitta gateway instansiyasi ishlashini ta’minlash; qo‘shimcha gateway’lar alohida profil va noyob portlardan foydalanishi kerak.
- Avariya yoki SIGKILL holatlarida eskirgan lock fayllar qolib ketmasligini ta’minlash.
- Boshqaruv porti allaqachon band bo‘lsa, aniq xato bilan tezda to‘xtash.

## Mexanizm

- Gateway ishga tushishi bilan darhol WebSocket tinglovchini (standart `ws://127.0.0.1:18789`) eksklyuziv TCP tinglovchi orqali bind qiladi.
- Agar bind `EADDRINUSE` bilan muvaffaqiyatsiz tugasa, ishga tushirish `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` xatosini chiqaradi.
- Har qanday jarayon yakunlanganda, jumladan avariya va SIGKILL holatlarida ham, OT tinglovchini avtomatik ravishda bo‘shatadi — alohida lock fayl yoki tozalash bosqichi talab qilinmaydi.
- O‘chirish vaqtida gateway portni tezda bo‘shatish uchun WebSocket server va uning ostidagi HTTP serverni yopadi.

## Xatolik yuzasi

- Agar portni boshqa jarayon egallagan bo‘lsa, ishga tushirish `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` xatosini chiqaradi.
- Boshqa bind xatolari `GatewayLockError("failed to bind gateway socket on ws://127.0.0.1:<port>: …")` sifatida ko‘rinadi.

## Operatsion eslatmalar

- Agar portni _boshqa_ jarayon egallagan bo‘lsa, xato bir xil bo‘ladi; portni bo‘shating yoki `openclaw gateway --port <port>` orqali boshqasini tanlang.
- macOS ilovasi gateway’ni ishga tushirishdan oldin hali ham o‘zining yengil PID himoyasini saqlab qoladi; ammo ish vaqtidagi qulf WebSocket bind orqali ta’minlanadi.