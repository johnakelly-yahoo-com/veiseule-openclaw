---
title: "Peekaboo Bridge"
---

# Peekaboo Bridge (macOS UI avtomatlashtirish)

OpenClaw **PeekabooBridge**’ni mahalliy, ruxsatlarga moslashuvchan UI avtomatlashtirish
brokeri sifatida joylashtirishi mumkin. Bu `peekaboo` CLI’ga macOS ilovasining
TCC ruxsatlaridan qayta foydalanib, UI avtomatlashtirishni boshqarish imkonini beradi.

## Bu nima (va nima emas)

- **Host**: OpenClaw.app PeekabooBridge hosti sifatida ishlashi mumkin.
- **Client**: `peekaboo` CLI’dan foydalaning (alohida `openclaw ui ...` interfeysi yo‘q).
- **UI**: vizual overlay’lar Peekaboo.app’da qoladi; OpenClaw esa yengil broker hosti vazifasini bajaradi.

## Bridge’ni yoqish

macOS ilovasida:

- Sozlamalar → **Peekaboo Bridge’ni yoqish**

Yoqilganda, OpenClaw mahalliy UNIX socket serverini ishga tushiradi. O‘chirilganda, host
to‘xtatiladi va `peekaboo` boshqa mavjud hostlarga o‘tadi.

## Client aniqlash tartibi

Peekaboo client’lari odatda hostlarni quyidagi tartibda sinab ko‘radi:

1. Peekaboo.app (to‘liq UX)
2. Claude.app (agar o‘rnatilgan bo‘lsa)
3. OpenClaw.app (yengil broker)

Qaysi host faol ekanini va qaysi socket yo‘li ishlatilayotganini ko‘rish uchun `peekaboo bridge status --verbose` buyrug‘idan foydalaning. Quyidagicha majburan belgilashingiz mumkin:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## Xavfsizlik va ruxsatlar

- Bridge **chaqiruvchi kod imzolarini** tekshiradi; TeamID’lar uchun allowlist
  qo‘llaniladi (Peekaboo host TeamID + OpenClaw ilova TeamID).
- So‘rovlar ~10 soniyadan so‘ng vaqt bo‘yicha bekor qilinadi.
- Zarur ruxsatlar mavjud bo‘lmasa, bridge System Settings’ni ishga tushirish o‘rniga
  aniq xato xabarini qaytaradi.

## Snapshot xatti-harakati (avtomatlashtirish)

Snapshot’lar xotirada saqlanadi va qisqa muddatdan so‘ng avtomatik ravishda o‘chadi.
Agar uzoqroq saqlash kerak bo‘lsa, client’dan qayta suratga oling.

## Nosozliklarni bartaraf etish

- Agar `peekaboo` “bridge client is not authorized” deb xabar bersa, client
  to‘g‘ri imzolanganini tekshiring yoki host’ni faqat **debug** rejimida
  `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` bilan ishga tushiring.
- Agar hech qanday host topilmasa, host ilovalardan birini (Peekaboo.app yoki OpenClaw.app)
  oching va ruxsatlar berilganini tasdiqlang.

