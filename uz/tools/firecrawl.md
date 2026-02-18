---
title: "Firecrawl"
---

# Firecrawl

OpenClaw `web_fetch` uchun zaxira ajratuvchi sifatida **Firecrawl** dan foydalanishi mumkin. Bu joylashtirilgan
kontent ajratish xizmati bo‘lib, botlardan himoyani chetlab o‘tish va keshlashni qo‘llab-quvvatlaydi, bu esa
JS-ga boy saytlar yoki oddiy HTTP so‘rovlarini bloklaydigan sahifalar bilan ishlashda yordam beradi.

## API kalitini oling

1. Firecrawl akkauntini yarating va API kalit hosil qiling.
2. Uni konfiguratsiyada saqlang yoki gateway muhitida `FIRECRAWL_API_KEY` ni o‘rnating.

## Firecrawl ni sozlash

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60,
        },
      },
    },
  },
}
```

Izohlar:

- API kalit mavjud bo‘lsa, `firecrawl.enabled` sukut bo‘yicha true bo‘ladi.
- `maxAgeMs` keshlangan natijalar qanchalik eski bo‘lishi mumkinligini (ms) belgilaydi. Sukut bo‘yicha 2 kun.

## Stealth / botlardan himoyani chetlab o‘tish

Firecrawl botlardan himoyani chetlab o‘tish uchun **proxy mode** parametrini taqdim etadi (`basic`, `stealth` yoki `auto`).
OpenClaw har doim Firecrawl so‘rovlari uchun `proxy: "auto"` va `storeInCache: true` dan foydalanadi.
Agar proxy ko‘rsatilmasa, Firecrawl sukut bo‘yicha `auto` ni ishlatadi. `auto` dastlabki urinish muvaffaqiyatsiz bo‘lsa, stealth proxy bilan qayta urinadi, bu esa
faqat basic scrapingga qaraganda ko‘proq kredit sarflanishiga olib kelishi mumkin.

## `web_fetch` Firecrawl dan qanday foydalanadi

`web_fetch` ajratish tartibi:

1. Readability (lokal)
2. Firecrawl (agar sozlangan bo‘lsa)
3. Oddiy HTML tozalash (oxirgi zaxira variant)

To‘liq web tool sozlamalari uchun [Web tools](/tools/web) ga qarang.