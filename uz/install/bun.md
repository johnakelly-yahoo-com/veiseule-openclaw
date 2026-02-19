---
summary: "Bun ish jarayoni (eksperimental): o‘rnatish va pnpm bilan bog‘liq nozik jihatlar"
read_when:
  - Siz eng tezkor lokal dev jarayonini xohlaysiz (bun + watch)
  - Siz Bun o‘rnatish/patch/lifecycle script muammolariga duch keldingiz
title: "Bun (Eksperimental)"
---

# Bun (eksperimental)

Maqsad: ushbu repozitoriyani **Bun** bilan ishga tushirish (ixtiyoriy, WhatsApp/Telegram uchun tavsiya etilmaydi),  
pnpm ish jarayonlaridan chetga chiqmagan holda.

⚠️ **Not recommended for Gateway runtime** (WhatsApp/Telegram bugs). Use Node for production.

## Holati

- Bun — TypeScript’ni to‘g‘ridan-to‘g‘ri ishga tushirish uchun ixtiyoriy lokal runtime (`bun run …`, `bun --watch …`).
- `pnpm` buildlar uchun standart vosita bo‘lib qoladi va to‘liq qo‘llab-quvvatlanadi (ba’zi hujjatlashtirish vositalarida ham ishlatiladi).
- Bun `pnpm-lock.yaml` faylidan foydalana olmaydi va uni e’tiborsiz qoldiradi.

## O‘rnatish

Standart usul:

```sh
bun install
```

Note: `bun.lock`/`bun.lockb` are gitignored, so there’s no repo churn either way. If you want _no lockfile writes_:

```sh
bun install --no-save
```

## Yig‘ish / Sinov (Bun)

```sh
bun run build
bun run vitest run
```

## Bun lifecycle scriptlari (standart holatda bloklanadi)

Bun may block dependency lifecycle scripts unless explicitly trusted (`bun pm untrusted` / `bun pm trust`).
For this repo, the commonly blocked scripts are not required:

- `@whiskeysockets/baileys` `preinstall`: Node major versiyasi >= 20 ekanini tekshiradi (biz Node 22+ ishlatamiz).
- `protobufjs` `postinstall`: mos kelmaydigan versiya sxemalari haqida ogohlantirishlar chiqaradi (build artefaktlari yo‘q).

Agar haqiqiy runtime muammosi yuzaga kelib, ushbu scriptlar zarur bo‘lsa, ularni aniq ishonchli deb belgilang:

```sh
bun pm trust @whiskeysockets/baileys protobufjs
```

## Cheklovlar

- Some scripts still hardcode pnpm (e.g. `docs:build`, `ui:*`, `protocol:check`). Run those via pnpm for now.

