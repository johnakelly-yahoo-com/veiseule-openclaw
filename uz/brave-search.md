---
summary: "web_search uchun Brave Search API sozlamalari"
read_when:
  - Siz web_search uchun Brave Search’dan foydalanmoqchisiz
  - Sizga BRAVE_API_KEY yoki tarif tafsilotlari kerak
title: "Brave Search"
---

# Brave Search API

OpenClaw `web_search` uchun standart provayder sifatida Brave Search’dan foydalanadi.

## API kalitini olish

1. [https://brave.com/search/api/](https://brave.com/search/api/) orqali Brave Search API hisobini yarating  
2. Boshqaruv panelida **Data for Search** rejasini tanlang va API kalitini yarating.  
3. Kalitni config’da (tavsiya etiladi) saqlang yoki Gateway muhitida `BRAVE_API_KEY` ni o‘rnating.

## Config namunasi

```json5
{
  tools: {
    web: {
      search: {
        provider: "brave",
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

## Eslatmalar

- Data for AI rejasi `web_search` bilan **mos kelmaydi**.  
- Brave bepul tarif va pullik rejalarni taklif etadi; amaldagi cheklovlar uchun Brave API portalini tekshiring.

`web_search` to‘liq sozlamalari uchun [Web tools](/tools/web) bo‘limiga qarang.