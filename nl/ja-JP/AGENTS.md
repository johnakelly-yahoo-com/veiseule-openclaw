# AGENTS.md - ja-JP documentatievertalingswerkruimte

## Lezen wanneer

- Onderhoud van `docs/ja-JP/**`
- Het bijwerken van de Japanse vertaalpipeline (glossary/TM/prompt)
- Afhandelen van feedback of regressies in de Japanse vertaling

## Pipeline (docs-i18n)

- Brondocumentatie: `docs/**/*.md`
- Doeldocumentatie: `docs/ja-JP/**/*.md`
- Woordenlijst: `docs/.i18n/glossary.ja-JP.json`
- Vertaalgeheugen: `docs/.i18n/ja-JP.tm.jsonl`
- Promptregels: `scripts/docs-i18n/prompt.go`

Veelvoorkomende uitvoeringen:

```bash
# Bulk (doc-modus; parallel OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Enkel bestand
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Kleine patches (segment-modus; gebruikt TM; geen parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Opmerkingen:

- Gebruik bij voorkeur de `doc`-modus voor vertaling van volledige pagina’s; de `segment`-modus voor kleine correcties.
- Als een zeer groot bestand een time-out veroorzaakt, voer gerichte bewerkingen uit of splits de pagina voordat je opnieuw uitvoert.
- Controleer na vertaling steekproefsgewijs: codefragmenten/blokken ongewijzigd, links/anchors ongewijzigd, placeholders behouden.
