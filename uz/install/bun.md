---
title: "Bun (Eksperimental)"
---

# Bun (eksperimental)

Maqsad: ushbu repozitoriyani **Bun** bilan ishga tushirish (ixtiyoriy, WhatsApp/Telegram uchun tavsiya etilmaydi),  
pnpm ish jarayonlaridan chetga chiqmagan holda.

⚠️ **Gateway runtime uchun tavsiya etilmaydi** (WhatsApp/Telegram xatoliklari sababli). Ishlab chiqarish muhiti uchun Node’dan foydalaning.

## Holati

- Bun — TypeScript’ni to‘g‘ridan-to‘g‘ri ishga tushirish uchun ixtiyoriy lokal runtime (`bun run …`, `bun --watch …`).
- `pnpm` buildlar uchun standart vosita bo‘lib qoladi va to‘liq qo‘llab-quvvatlanadi (ba’zi hujjatlashtirish vositalarida ham ishlatiladi).
- Bun `pnpm-lock.yaml` faylidan foydalana olmaydi va uni e’tiborsiz qoldiradi.

## O‘rnatish

Standart usul:

```sh
bun install
```

Eslatma: `bun.lock`/`bun.lockb` fayllari gitignored qilingan, shuning uchun repozitoriyada ortiqcha o‘zgarishlar bo‘lmaydi. Agar _lockfile yozilmasligini_ xohlasangiz:

```sh
bun install --no-save
```

## Yig‘ish / Sinov (Bun)

```sh
bun run build
bun run vitest run
```

## Bun lifecycle scriptlari (standart holatda bloklanadi)

Bun bog‘liqlik lifecycle scriptlarini aniq ishonch bildirilmaguncha bloklashi mumkin (`bun pm untrusted` / `bun pm trust`).  
Ushbu repo uchun odatda bloklanadigan scriptlar majburiy emas:

- `@whiskeysockets/baileys` `preinstall`: Node major versiyasi >= 20 ekanini tekshiradi (biz Node 22+ ishlatamiz).
- `protobufjs` `postinstall`: mos kelmaydigan versiya sxemalari haqida ogohlantirishlar chiqaradi (build artefaktlari yo‘q).

Agar haqiqiy runtime muammosi yuzaga kelib, ushbu scriptlar zarur bo‘lsa, ularni aniq ishonchli deb belgilang:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## Cheklovlar

- Ba’zi scriptlar hali ham pnpm’ni qat’iy belgilagan (masalan, `docs:build`, `ui:*`, `protocol:check`). Hozircha ularni pnpm orqali ishga tushiring.


