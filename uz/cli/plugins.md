---
summary: "`openclaw plugins` uchun CLI maʼlumotnomasi (roʻyxatlash, oʻrnatish, yoqish/oʻchirish, doctor)"
read_when:
  - Siz in-process Gateway plaginlarini oʻrnatmoqchi yoki boshqarmoqchisiz
  - Siz plagin yuklanishidagi nosozliklarni tuzatmoqchisiz
title: "plaginlar"
---

# `openclaw plugins`

Gateway plaginlari/kengaytmalarini (jarayon ichida yuklangan) boshqarish.

Bogʻliq:

- Plagin tizimi: [Plugins](/tools/plugin)
- Plagin manifesti + sxema: [Plugin manifest](/plugins/manifest)
- Xavfsizlikni mustahkamlash: [Security](/gateway/security)

## Buyruqlar

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Birga keladigan plaginlar OpenClaw bilan yetkaziladi, ammo dastlab o‘chirilgan bo‘ladi. Ularni faollashtirish uchun `plugins enable` dan foydalaning.

Barcha plaginlar ichki JSON Sxemaga ega (`configSchema`, bo‘sh bo‘lsa ham) `openclaw.plugin.json` faylini taqdim etishi shart. Yetishmaydigan/yaroqsiz manifestlar yoki sxemalar plaginning yuklanishiga to‘sqinlik qiladi va konfiguratsiya tekshiruvini muvaffaqiyatsiz qiladi.

### O‘rnatish

```bash
openclaw plugins install <path-or-spec>
```

Xavfsizlik eslatmasi: plaginlarni o‘rnatishni kodni ishga tushirishdek qabul qiling. Mahkamlangan (pinned) versiyalarni afzal ko‘ring.

Npm spetsifikatsiyalari faqat **registry-only** (paket nomi + ixtiyoriy versiya/tag). Git/URL/file
spetsifikatsiyalari rad etiladi. Bog‘liqliklarni o‘rnatish xavfsizlik uchun `--ignore-scripts` bilan bajariladi.

Mahalliy katalogni nusxalamaslik uchun `--link` dan foydalaning (`plugins.load.paths` ga qo‘shadi):

Mahalliy katalogni nusxalamaslik uchun `--link` dan foydalaning (`plugins.load.paths` ga qo‘shadi):

```bash
openclaw plugins install -l ./my-plugin
```

### O‘chirish

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` plugin yozuvlarini `plugins.entries`, `plugins.installs`,
plugin allowlist va mavjud bo‘lsa bog‘langan `plugins.load.paths` yozuvlaridan olib tashlaydi.
Faol memory pluginlari uchun memory slot `memory-core` ga qayta o‘rnatiladi.

Standart bo‘yicha, uninstall faol state dir extensions ildizi ostidagi
plugin o‘rnatish katalogini ham o‘chiradi (`$OPENCLAW_STATE_DIR/extensions/<id>`). Diskdagi fayllarni saqlab qolish uchun
`--keep-files` dan foydalaning.

`--keep-config` eskirgan alias sifatida `--keep-files` uchun qo‘llab-quvvatlanadi.

### Yangilash

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Yangilanishlar faqat npm’dan o‘rnatilgan ( `plugins.installs` da kuzatiladigan) plaginlarga qo‘llanadi.
