# AGENTS.md - espace de travail de traduction de la documentation ja-JP

## À lire lorsque

- Maintenance de `docs/ja-JP/**`
- Mise à jour du pipeline de traduction japonais (glossaire/TM/prompt)
- Gestion des retours ou régressions de traduction japonaise

## Pipeline (docs-i18n)

- Documentation source : `docs/**/*.md`
- Documentation cible : `docs/ja-JP/**/*.md`
- Glossaire : `docs/.i18n/glossary.ja-JP.json`
- Mémoire de traduction : `docs/.i18n/ja-JP.tm.jsonl`
- Règles de prompt : `scripts/docs-i18n/prompt.go`

Exécutions courantes :

```bash
# Bulk (mode doc ; parallélisation OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Fichier unique
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Petits correctifs (mode segment ; utilise la TM ; pas de parallélisation)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Remarques :

- Privilégiez le mode `doc` pour la traduction de pages complètes ; le mode `segment` pour les petites corrections.
- Si un fichier très volumineux dépasse le délai d’exécution, effectuez des modifications ciblées ou divisez la page avant de relancer.
- Après traduction, vérifiez rapidement : segments/blocs de code inchangés, liens/ancres inchangés, espaces réservés préservés.
