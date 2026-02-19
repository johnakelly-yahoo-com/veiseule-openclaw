# AGENTS.md - espacio de trabajo para la traducción de la documentación ja-JP

## Leer cuando

- Mantenimiento de `docs/ja-JP/**`
- Actualización del pipeline de traducción al japonés (glosario/TM/prompt)
- Gestión de comentarios o regresiones en la traducción al japonés

## Pipeline (docs-i18n)

- Documentos fuente: `docs/**/*.md`
- Documentos de destino: `docs/ja-JP/**/*.md`
- Glosario: `docs/.i18n/glossary.ja-JP.json`
- Memoria de traducción: `docs/.i18n/ja-JP.tm.jsonl`
- Reglas de prompt: `scripts/docs-i18n/prompt.go`

Ejecuciones comunes:

```bash
# Bulk (modo doc; paralelo permitido)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Archivo único
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Parches pequeños (modo segment; usa TM; sin paralelo)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Notas:

- Prefiere el modo `doc` para la traducción de páginas completas; el modo `segment` para pequeñas correcciones.
- Si un archivo muy grande agota el tiempo de ejecución, realiza ediciones específicas o divide la página antes de volver a ejecutarlo.
- Después de la traducción, haz una revisión rápida: los fragmentos/bloques de código sin cambios, los enlaces/anclas sin cambios, los marcadores de posición conservados.
