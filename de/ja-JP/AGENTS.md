# AGENTS.md - ja-JP-Dokumentationsübersetzungs-Workspace

## Lesen, wenn

- `docs/ja-JP/**` gepflegt wird
- Die japanische Übersetzungspipeline (Glossar/TM/Prompt) aktualisiert wird
- Japanisches Übersetzungsfeedback oder Regressionen bearbeitet werden

## Pipeline (docs-i18n)

- Quelldokumente: `docs/**/*.md`
- Zieldokumente: `docs/ja-JP/**/*.md`
- Glossar: `docs/.i18n/glossary.ja-JP.json`
- Translation Memory: `docs/.i18n/ja-JP.tm.jsonl`
- Prompt-Regeln: `scripts/docs-i18n/prompt.go`

Häufige Ausführungen:

```bash
# Bulk (doc-Modus; parallel möglich)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Einzelne Datei
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Kleine Patches (Segmentmodus; verwendet TM; nicht parallel)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Hinweise:

- Bevorzugen Sie den `doc`-Modus für die Übersetzung ganzer Seiten; den `segment`-Modus für kleine Korrekturen.
- Wenn eine sehr große Datei zu einem Timeout führt, nehmen Sie gezielte Änderungen vor oder teilen Sie die Seite vor dem erneuten Ausführen auf.
- Überprüfen Sie nach der Übersetzung stichprobenartig: Code-Spans/-Blöcke unverändert, Links/Anker unverändert, Platzhalter beibehalten.
