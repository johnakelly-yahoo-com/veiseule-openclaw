---
title: "Firecrawl"
---

# Firecrawl

OpenClaw **Firecrawl** کو `web_fetch` کے لیے بطور fallback extractor استعمال کر سکتا ہے۔ یہ ایک ہوسٹڈ
content extraction service that supports bot circumvention and caching, which helps
with JS-heavy sites or pages that block plain HTTP fetches.

## API کلید حاصل کریں

1. Firecrawl اکاؤنٹ بنائیں اور ایک API کلید تیار کریں۔
2. اسے کنفیگ میں محفوظ کریں یا گیٹ وے ماحول میں `FIRECRAWL_API_KEY` سیٹ کریں۔

## Firecrawl کنفیگر کریں

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

نوٹس:

- جب API کلید موجود ہو تو `firecrawl.enabled` بطورِ طے شدہ true ہوتا ہے۔
- `maxAgeMs` کنٹرول کرتا ہے کہ cached نتائج کتنے پرانے ہو سکتے ہیں (ms)۔ ڈیفالٹ 2 دن ہے۔

## اسٹیلتھ / بوٹ سے بچاؤ

Firecrawl ایک **proxy mode** پیرامیٹر فراہم کرتا ہے جو بوٹس سے بچاؤ کے لیے ہے (`basic`, `stealth`, یا `auto`)۔
OpenClaw always uses `proxy: "auto"` plus `storeInCache: true` for Firecrawl requests.
If proxy is omitted, Firecrawl defaults to `auto`. `auto` retries with stealth proxies if a basic attempt fails, which may use more credits
than basic-only scraping.

## `web_fetch` Firecrawl کیسے استعمال کرتا ہے

`web_fetch` استخراج کی ترتیب:

1. Readability (لوکل)
2. Firecrawl (اگر کنفیگر کیا گیا ہو)
3. بنیادی HTML صفائی (آخری فال بیک)

مکمل ویب ٹول سیٹ اپ کے لیے [Web tools](/tools/web) دیکھیں۔
