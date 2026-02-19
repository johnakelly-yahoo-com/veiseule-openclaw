# AGENTS.md - ja-JP dokumentations-oversættelsesarbejdsområde

## Læs når

- Vedligeholdelse af `docs/ja-JP/**`
- Opdatering af den japanske oversættelsespipeline (ordliste/TM/prompt)
- Håndtering af feedback eller regressioner i japansk oversættelse

## Pipeline (docs-i18n)

- Kildedokumenter: `docs/**/*.md`
- Måldokumenter: `docs/ja-JP/**/*.md`
- Ordliste: `docs/.i18n/glossary.ja-JP.json`
- Oversættelseshukommelse: `docs/.i18n/ja-JP.tm.jsonl`
- Prompt-regler: `scripts/docs-i18n/prompt.go`

Almindelige kørsler:

```bash
# Bulk (doc-tilstand; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Enkelt fil
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Små rettelser (segment-tilstand; bruger TM; ingen parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Bemærk:

- Foretræk `doc`-tilstand til oversættelse af hele sider; brug `segment`-tilstand til små rettelser.
- Hvis en meget stor fil giver timeout, så lav målrettede redigeringer eller opdel siden før du kører igen.
- Efter oversættelse: stikprøvekontrol — kode spans/blocks uændrede, links/anchors uændrede, placeholders bevaret.
