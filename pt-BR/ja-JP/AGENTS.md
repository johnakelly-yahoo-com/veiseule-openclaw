# AGENTS.md - workspace de tradução da documentação ja-JP

## Leia quando

- Mantendo `docs/ja-JP/**`
- Atualizando o pipeline de tradução para o japonês (glossário/TM/prompt)
- Lidando com feedback ou regressões na tradução para o japonês

## Pipeline (docs-i18n)

- Documentação de origem: `docs/**/*.md`
- Documentação de destino: `docs/ja-JP/**/*.md`
- Glossário: `docs/.i18n/glossary.ja-JP.json`
- Memória de tradução: `docs/.i18n/ja-JP.tm.jsonl`
- Regras de prompt: `scripts/docs-i18n/prompt.go`

Execuções comuns:

```bash
# Em lote (modo doc; paralelo OK)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Arquivo único
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Pequenos ajustes (modo segment; usa TM; sem paralelo)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Observações:

- Prefira o modo `doc` para tradução de páginas completas; use o modo `segment` para pequenas correções.
- Se um arquivo muito grande expirar por tempo limite, faça edições direcionadas ou divida a página antes de executar novamente.
- Após a tradução, faça uma verificação rápida: trechos/blocos de código inalterados, links/âncoras inalterados, placeholders preservados.
