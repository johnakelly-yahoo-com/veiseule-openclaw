---
summary: "`openclaw status` uchun CLI ma’lumotnomasi (diagnostika, probalar, foydalanish holati ko‘rinishlari)"
read_when:
  - You want a quick diagnosis of channel health + recent session recipients
  - You want a pasteable “all” status for debugging
title: "status"
---

# `openclaw status`

Kanallar va sessiyalar uchun diagnostika.

```bash
openclaw status
openclaw status --all
openclaw status --deep
openclaw status --usage
```

Eslatmalar:

- Bir nechta agent sozlanganda chiqishda har bir agent uchun sessiya saqlovlari ko‘rsatiladi.
- Mavjud bo‘lsa, umumiy ko‘rinish Gateway + node host xizmati o‘rnatilishi/ishga tushish holatini o‘z ichiga oladi.
- Umumiy ko‘rinish yangilash kanali + git SHA (manbadan yig‘ilganlar uchun) ni o‘z ichiga oladi.
- `openclaw system` uchun CLI ma’lumotnomasi (tizim hodisalari, heartbeat, presence)
- Yangilanish ma’lumoti Umumiy ko‘rinishda ko‘rsatiladi; agar yangilanish mavjud bo‘lsa, holat `openclaw update` ni ishga tushirish haqida ishora chiqaradi (qarang: [Updating](/install/updating)).
