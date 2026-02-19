# AGENTS.md - ja-JP ڈاکس ترجمہ ورک اسپیس

## کب پڑھیں

- `docs/ja-JP/**` کی دیکھ بھال کرتے وقت
- جاپانی ترجمہ پائپ لائن (glossary/TM/prompt) کو اپ ڈیٹ کرتے وقت
- جاپانی ترجمے سے متعلق فیڈبیک یا ریگریشن کو ہینڈل کرتے وقت

## پائپ لائن (docs-i18n)

- ماخذ ڈاکس: `docs/**/*.md`
- ہدف ڈاکس: `docs/ja-JP/**/*.md`
- گلوسری: `docs/.i18n/glossary.ja-JP.json`
- ترجمہ میموری: `docs/.i18n/ja-JP.tm.jsonl`
- پرامپٹ قواعد: `scripts/docs-i18n/prompt.go`

عام رنز:

```bash
# Bulk (doc mode; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Single file
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Small patches (segment mode; uses TM; no parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

نوٹس:

- مکمل صفحے کے ترجمے کے لیے `doc` موڈ کو ترجیح دیں؛ چھوٹی درستگیوں کے لیے `segment` موڈ استعمال کریں۔
- اگر بہت بڑی فائل ٹائم آؤٹ ہو جائے تو ہدفی ترامیم کریں یا دوبارہ چلانے سے پہلے صفحہ تقسیم کر دیں۔
- ترجمے کے بعد جائزہ لیں: کوڈ اسپین/بلاکس غیر تبدیل رہیں، لنکس/اینکرز غیر تبدیل رہیں، پلیس ہولڈرز محفوظ رہیں۔
