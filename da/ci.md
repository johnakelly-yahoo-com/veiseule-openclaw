---
title: CI-pipeline
description: Sådan fungerer OpenClaw CI-pipelinen
---

# CI-pipeline

CI kører ved hvert push til `main` og ved hver pull request. Den bruger smart afgrænsning til at springe ressourcekrævende jobs over, når kun docs eller native-kode er ændret.

## Joboversigt

| Job               | Formål                                                                         | Hvornår det kører                     |
| ----------------- | ------------------------------------------------------------------------------ | ------------------------------------- |
| `docs-scope`      | Registrer ændringer kun i docs                                                 | Altid                                 |
| `changed-scope`   | Registrer hvilke områder der er ændret (node/macos/android) | Ikke-docs PR'er                       |
| `check`           | TypeScript-typer, lint, formatering                                            | Ikke-docs ændringer                   |
| `check-docs`      | Markdown-lint + kontrol af døde links                                          | Docs ændret                           |
| `code-analysis`   | Kontrol af LOC-grænse (1000 linjer)                         | Kun PR'er                             |
| `secrets`         | Registrer lækkede secrets                                                      | Altid                                 |
| `build-artifacts` | Byg dist én gang, del med andre jobs                                           | Ikke-docs, node-ændringer             |
| `release-check`   | Valider indholdet af npm pack                                                  | Efter build                           |
| `checks`          | Node/Bun-tests + protokolkontrol                                               | Ikke-docs, node-ændringer             |
| `checks-windows`  | Windows-specifikke tests                                                       | Ikke-docs, node-ændringer             |
| `macos`           | Swift lint/build/test + TS-tests                                               | PR'er med macos-ændringer             |
| `android`         | Gradle build + tests                                                           | Ikke-dokumentation, Android-ændringer |

## Fail-Fast-rækkefølge

Jobs er arrangeret, så billige checks fejler før de dyre køres:

1. `docs-scope` + `code-analysis` + `check` (parallelt, ~1-2 min)
2. `build-artifacts` (blokeret af ovenstående)
3. `checks`, `checks-windows`, `macos`, `android` (blokeret af build)

## Runners

| Runner                          | Jobs                                          |
| ------------------------------- | --------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | De fleste Linux-jobs                          |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                              |
| `macos-latest`                  | `macos`, `ios`                                |
| `ubuntu-latest`                 | Scope-detektion (letvægts) |

## Lokale ækvivalenter

```bash
pnpm check          # typer + lint + format
pnpm test           # vitest-tests
pnpm check:docs     # docs-format + lint + brudte links
pnpm release:check  # valider npm pack
```
