# AGENTS.md - ja-JP hujjatlar tarjima ish maydoni

## Qachon o‘qish kerak

- `docs/ja-JP/**` ni qo‘llab-quvvatlashda
- Yaponcha tarjima pipeline’ini (glossary/TM/prompt) yangilashda
- Yaponcha tarjima bo‘yicha fikr-mulohazalar yoki regressiyalarni ko‘rib chiqishda

## Pipeline (docs-i18n)

- Manba hujjatlar: `docs/**/*.md`
- Maqsad hujjatlar: `docs/ja-JP/**/*.md`
- Glossary: `docs/.i18n/glossary.ja-JP.json`
- Tarjima xotirasi: `docs/.i18n/ja-JP.tm.jsonl`
- Prompt qoidalari: `scripts/docs-i18n/prompt.go`

Odatdagi ishga tushirishlar:

```bash
# Bulk (doc rejimi; parallel mumkin)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Bitta fayl
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Kichik patchlar (segment rejimi; TM dan foydalanadi; parallelsiz)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Eslatmalar:

- Butun sahifani tarjima qilish uchun `doc` rejimini afzal ko‘ring; kichik tuzatishlar uchun `segment` rejimi.
- Agar juda katta fayl taym-autga uchrasa, qayta ishga tushirishdan oldin maqsadli tahrirlar qiling yoki sahifani bo‘lib chiqing.
- Tarjimadan so‘ng tekshirib chiqing: kod satrlari/bloklari o‘zgarmagan, havolalar/anchorlar o‘zgarmagan, placeholderlar saqlangan.
