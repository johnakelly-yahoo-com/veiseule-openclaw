---
summary: "macOS UI avtomatlashtirish uchun PeekabooBridge integratsiyasi"
read_when:
  - OpenClaw.app’da PeekabooBridge’ni joylashtirish
  - Peekaboo’ni Swift Package Manager orqali integratsiya qilish
  - PeekabooBridge protokoli/yo‘llarini o‘zgartirish
title: "Peekaboo Bridge"
---

# Peekaboo Bridge (macOS UI avtomatlashtirish)

OpenClaw can host **PeekabooBridge** as a local, permission‑aware UI automation
broker. This lets the `peekaboo` CLI drive UI automation while reusing the
macOS app’s TCC permissions.

## Bu nima (va nima emas)

- **Host**: OpenClaw.app PeekabooBridge hosti sifatida ishlashi mumkin.
- **Client**: `peekaboo` CLI’dan foydalaning (alohida `openclaw ui ...` interfeysi yo‘q).
- **UI**: vizual overlay’lar Peekaboo.app’da qoladi; OpenClaw esa yengil broker hosti vazifasini bajaradi.

## Bridge’ni yoqish

macOS ilovasida:

- Sozlamalar → **Peekaboo Bridge’ni yoqish**

When enabled, OpenClaw starts a local UNIX socket server. If disabled, the host
is stopped and `peekaboo` will fall back to other available hosts.

## Client aniqlash tartibi

Peekaboo client’lari odatda hostlarni quyidagi tartibda sinab ko‘radi:

1. Peekaboo.app (to‘liq UX)
2. Claude.app (agar o‘rnatilgan bo‘lsa)
3. OpenClaw.app (yengil broker)

Use `peekaboo bridge status --verbose` to see which host is active and which
socket path is in use. You can override with:

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

Snapshotlar xotirada saqlanadi va qisqa muddatdan so‘ng avtomatik ravishda muddati tugaydi.
Agar uzoqroq saqlash kerak bo‘lsa, mijoz tomondan qayta snapshot oling.

## Nosozliklarni bartaraf etish

- Agar `peekaboo` “bridge client is not authorized” deb xabar bersa, client
  to‘g‘ri imzolanganini tekshiring yoki host’ni faqat **debug** rejimida
  `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` bilan ishga tushiring.
- Agar hech qanday host topilmasa, host ilovalardan birini (Peekaboo.app yoki OpenClaw.app)
  oching va ruxsatlar berilganini tasdiqlang.
