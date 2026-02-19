# AGENTS.md - ja-JP dokümantasyon çeviri çalışma alanı

## Ne zaman okunmalı

- `docs/ja-JP/**` bakımını yaparken
- Japonca çeviri hattını (glossary/TM/prompt) güncellerken
- Japonca çeviri geri bildirimlerini veya gerilemeleri ele alırken

## Pipeline (docs-i18n)

- Kaynak dokümanlar: `docs/**/*.md`
- Hedef dokümanlar: `docs/ja-JP/**/*.md`
- Sözlük: `docs/.i18n/glossary.ja-JP.json`
- Çeviri belleği: `docs/.i18n/ja-JP.tm.jsonl`
- Prompt kuralları: `scripts/docs-i18n/prompt.go`

Yaygın çalıştırmalar:

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

Notlar:

- Tüm sayfa çevirileri için `doc` modunu; küçük düzeltmeler için `segment` modunu tercih edin.
- Çok büyük bir dosya zaman aşımına uğrarsa, yeniden çalıştırmadan önce hedefli düzenlemeler yapın veya sayfayı bölün.
- Çeviriden sonra kontrol edin: kod içi/kod blokları değişmemiş, bağlantılar/anchor’lar değişmemiş, yer tutucular korunmuş olmalı.
