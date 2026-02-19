---
title: CI-pijplijn
description: Hoe de OpenClaw CI-pijplijn werkt
---

# CI-pijplijn

De CI draait bij elke push naar `main` en bij elke pull request. Het gebruikt slimme afbakening om dure jobs over te slaan wanneer alleen documentatie of native code is gewijzigd.

## Joboverzicht

| Job               | Doel                                                                              | Wanneer het draait                     |
| ----------------- | --------------------------------------------------------------------------------- | -------------------------------------- |
| `docs-scope`      | Detecteer wijzigingen die alleen documentatie betreffen                           | Altijd                                 |
| `changed-scope`   | Detecteer welke onderdelen zijn gewijzigd (node/macos/android) | Niet-documentatie PR's                 |
| `check`           | TypeScript-types, linten, formatteren                                             | Niet-documentatie wijzigingen          |
| `check-docs`      | Markdown-lint + controle op gebroken links                                        | Documentatie gewijzigd                 |
| `code-analysis`   | Controle op LOC-drempel (1000 regels)                          | Alleen PR's                            |
| `secrets`         | Detecteer gelekte secrets                                                         | Altijd                                 |
| `build-artifacts` | Bouw dist één keer en deel met andere jobs                                        | Niet-documentatie, node-wijzigingen    |
| `release-check`   | Valideer npm pack-inhoud                                                          | Na de build                            |
| `checks`          | Node/Bun-tests + protocolcontrole                                                 | Niet-documentatie, node-wijzigingen    |
| `checks-windows`  | Windows-specifieke tests                                                          | Niet-documentatie, node-wijzigingen    |
| `macos`           | Swift lint/build/test + TS-tests                                                  | PR's met macos-wijzigingen             |
| `android`         | Gradle build + tests                                                              | Niet-documentatie, Android-wijzigingen |

## Fail-Fast-volgorde

Jobs worden zo geordend dat goedkope controles falen vóórdat dure worden uitgevoerd:

1. `docs-scope` + `code-analysis` + `check` (parallel, ~1-2 min)
2. `build-artifacts` (geblokkeerd door bovenstaande)
3. `checks`, `checks-windows`, `macos`, `android` (geblokkeerd door build)

## Runners

| Runner                          | Jobs                                             |
| ------------------------------- | ------------------------------------------------ |
| `blacksmith-4vcpu-ubuntu-2404`  | De meeste Linux-jobs                             |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                                 |
| `macos-latest`                  | `macos`, `ios`                                   |
| `ubuntu-latest`                 | Scope-detectie (lichtgewicht) |

## Lokale equivalenten

```bash
pnpm check          # types + lint + format
pnpm test           # vitest-tests
pnpm check:docs     # docs-opmaak + lint + gebroken links
pnpm release:check  # npm pack valideren
```
