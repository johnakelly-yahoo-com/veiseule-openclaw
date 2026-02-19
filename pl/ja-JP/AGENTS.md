# AGENTS.md - obszar roboczy tłumaczenia dokumentacji ja-JP

## Czytaj, gdy

- Utrzymujesz `docs/ja-JP/**`
- Aktualizujesz pipeline tłumaczeń japońskich (glossary/TM/prompt)
- Obsługujesz uwagi lub regresje dotyczące tłumaczeń na japoński

## Pipeline (docs-i18n)

- Dokumenty źródłowe: `docs/**/*.md`
- Dokumenty docelowe: `docs/ja-JP/**/*.md`
- Glossary: `docs/.i18n/glossary.ja-JP.json`
- Pamięć tłumaczeń: `docs/.i18n/ja-JP.tm.jsonl`
- Zasady promptu: `scripts/docs-i18n/prompt.go`

Typowe uruchomienia:

```bash
# Masowe (tryb doc; równoległość OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Pojedynczy plik
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Małe poprawki (tryb segment; używa TM; bez równoległości)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Uwagi:

- Preferuj tryb `doc` do tłumaczenia całych stron; tryb `segment` do drobnych poprawek.
- Jeśli bardzo duży plik powoduje przekroczenie limitu czasu, wykonaj ukierunkowane edycje lub podziel stronę przed ponownym uruchomieniem.
- Po tłumaczeniu sprawdź wyrywkowo: czy zakresy/bloki kodu są niezmienione, linki/anchory niezmienione, a placeholdery zachowane.
