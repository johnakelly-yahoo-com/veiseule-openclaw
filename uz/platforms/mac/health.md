---
title: "Sog‘liq tekshiruvlari"
---

# macOS’da sog‘liq tekshiruvlari

Menyu paneli ilovasi orqali ulangan kanal sog‘lom ekanligini qanday ko‘rish mumkin.

## Menyu paneli

- Holat nuqtasi endi Baileys sog‘lig‘ini aks ettiradi:
  - Yashil: ulangan + socket yaqinda ochilgan.
  - To‘q sariq: ulanmoqda/qayta urinmoqda.
  - Qizil: tizimdan chiqqan yoki tekshiruv muvaffaqiyatsiz tugagan.
- Ikkinchi qatorda "linked · auth 12m" yozuvi yoki xatolik sababi ko‘rsatiladi.
- "Run Health Check" menu item triggers an on-demand probe.

## Settings

- 1. Umumiy (General) yorlig‘ida quyidagilarni ko‘rsatadigan Health kartasi paydo bo‘ladi: bog‘langan autentifikatsiya yoshi, session-store yo‘li/soni, oxirgi tekshiruv vaqti, oxirgi xato/holat kodi hamda Run Health Check / Reveal Logs tugmalari.
- 2. UI darhol yuklanishi uchun keshlangan snapshotdan foydalanadi va oflayn bo‘lganda muloyim tarzda fallback qiladi.
- 3. **Channels yorlig‘i** WhatsApp/Telegram uchun kanal holati va boshqaruvlarini ko‘rsatadi (login QR, logout, probe, oxirgi uzilish/xato).

## 4. Probe qanday ishlaydi

- 5. Ilova har ~60 soniyada va talab bo‘yicha `ShellExecutor` orqali `openclaw health --json` ni ishga tushiradi. 6. Probe credentiallarni yuklaydi va xabar yubormasdan holatni hisobot qiladi.
- 7. Miltillashni oldini olish uchun oxirgi yaxshi snapshot va oxirgi xatoni alohida keshlang; har birining vaqt tamg‘asini ko‘rsating.

## 8. Ikki yo‘l qolsa

- 9. Siz hali ham [Gateway health](/gateway/health) dagi CLI oqimidan (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) foydalanishingiz va `web-heartbeat` / `web-reconnect` uchun `/tmp/openclaw/openclaw-*.log` ni tail qilishingiz mumkin.


