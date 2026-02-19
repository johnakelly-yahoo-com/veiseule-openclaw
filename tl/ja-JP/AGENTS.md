# AGENTS.md - workspace ng pagsasalin ng docs sa ja-JP

## Basahin Kapag

- Pinananatili ang `docs/ja-JP/**`
- Ina-update ang Japanese translation pipeline (glossary/TM/prompt)
- Humahawak ng feedback o regression sa Japanese translation

## Pipeline (docs-i18n)

- Mga source na docs: `docs/**/*.md`
- Mga target na docs: `docs/ja-JP/**/*.md`
- Glossary: `docs/.i18n/glossary.ja-JP.json`
- Translation memory: `docs/.i18n/ja-JP.tm.jsonl`
- Mga patakaran sa prompt: `scripts/docs-i18n/prompt.go`

Mga karaniwang pagpapatakbo:

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

Mga Tala:

- Mas piliin ang `doc` mode para sa buong pahinang pagsasalin; `segment` mode para sa maliliit na pag-aayos.
- Kung mag-timeout ang napakalaking file, gumawa ng target na mga edit o hatiin ang pahina bago muling patakbuhin.
- Pagkatapos ng pagsasalin, magsagawa ng spot-check: tiyaking hindi nabago ang code spans/blocks, hindi nabago ang links/anchors, at napanatili ang mga placeholder.
