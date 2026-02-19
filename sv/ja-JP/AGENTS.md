# AGENTS.md - ja-JP dokumentationsöversättningsarbetsyta

## Läs när

- Underhåller `docs/ja-JP/**`
- Uppdaterar det japanska översättningsflödet (ordlista/TM/prompt)
- Hanterar feedback eller regressioner i japanska översättningar

## Pipeline (docs-i18n)

- Källdokument: `docs/**/*.md`
- Måldokument: `docs/ja-JP/**/*.md`
- Ordlista: `docs/.i18n/glossary.ja-JP.json`
- Översättningsminne: `docs/.i18n/ja-JP.tm.jsonl`
- Promptregler: `scripts/docs-i18n/prompt.go`

Vanliga körningar:

```bash
# Bulk (doc-läge; parallellt OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Enskild fil
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Små patchar (segment-läge; använder TM; ingen parallellkörning)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Obs:

- Föredra `doc`-läge för översättning av hela sidor; använd `segment`-läge för mindre korrigeringar.
- Om en mycket stor fil får timeout, gör riktade ändringar eller dela upp sidan innan du kör igen.
- Efter översättning, gör en snabbkontroll: kodspann/kodblock oförändrade, länkar/ankare oförändrade, platshållare bevarade.
