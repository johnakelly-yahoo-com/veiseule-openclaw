---
title: Pipeline de CI
description: Como funciona o pipeline de CI do OpenClaw
---

# Pipeline de CI

O CI é executado a cada push para `main` e em cada pull request. Ele usa escopo inteligente para pular jobs custosos quando apenas documentação ou código nativo foram alterados.

## Visão geral dos jobs

| Job               | Objetivo                                                                     | Quando é executado                                              |
| ----------------- | ---------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `docs-scope`      | Detectar alterações apenas na documentação                                   | Sempre                                                          |
| `changed-scope`   | Detectar quais áreas foram alteradas (node/macos/android) | PRs que não são apenas de documentação                          |
| `check`           | Tipos TypeScript, lint, formatação                                           | Alterações que não são apenas de documentação                   |
| `check-docs`      | Lint de Markdown + verificação de links quebrados                            | Documentação alterada                                           |
| `code-analysis`   | Verificação de limite de LOC (1000 linhas)                | Apenas PRs                                                      |
| `secrets`         | Detectar segredos expostos                                                   | Sempre                                                          |
| `build-artifacts` | Gerar a pasta dist uma vez e compartilhar com outros jobs                    | Alterações que não são apenas de documentação, mudanças em node |
| `release-check`   | Validar o conteúdo do npm pack                                               | Após o build                                                    |
| `checks`          | Testes Node/Bun + verificação de protocolo                                   | Alterações que não são apenas de documentação, mudanças em node |
| `checks-windows`  | Testes específicos do Windows                                                | Alterações que não são apenas de documentação, mudanças em node |
| `macos`           | Lint/build/test em Swift + testes TS                                         | PRs com alterações em macos                                     |
| `android`         | Build do Gradle + testes                                                     | Alterações não relacionadas à documentação, android             |

## Ordem Fail-Fast

Os jobs são ordenados para que verificações baratas falhem antes da execução das mais caras:

1. `docs-scope` + `code-analysis` + `check` (paralelo, ~1-2 min)
2. `build-artifacts` (bloqueado pelos anteriores)
3. `checks`, `checks-windows`, `macos`, `android` (bloqueados pelo build)

## Runners

| Runner                          | Jobs                                         |
| ------------------------------- | -------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | Maioria dos jobs Linux                       |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                             |
| `macos-latest`                  | `macos`, `ios`                               |
| `ubuntu-latest`                 | Detecção de escopo (leve) |

## Equivalentes locais

```bash
pnpm check          # tipos + lint + formatação
pnpm test           # testes vitest
pnpm check:docs     # formatação + lint da documentação + links quebrados
pnpm release:check  # validar npm pack
```
