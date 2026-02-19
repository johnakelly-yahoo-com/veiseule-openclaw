---
title: CI Pipeline
description: Paano gumagana ang OpenClaw CI pipeline
---

# CI Pipeline

Ang CI ay tumatakbo sa bawat push sa `main` at sa bawat pull request. Gumagamit ito ng matalinong scoping upang laktawan ang mga mabibigat na job kapag docs o native code lang ang nabago.

## Pangkalahatang-ideya ng Job

| Job               | Layunin                                                                          | Kailan ito tumatakbo                         |
| ----------------- | -------------------------------------------------------------------------------- | -------------------------------------------- |
| `docs-scope`      | Tukuyin kung docs-only ang mga pagbabago                                         | Palagi                                       |
| `changed-scope`   | Tukuyin kung aling mga bahagi ang nabago (node/macos/android) | Mga PR na hindi docs                         |
| `check`           | TypeScript types, lint, format                                                   | Mga pagbabagong hindi docs                   |
| `check-docs`      | Markdown lint + pagsusuri ng sirang link                                         | Nagbago ang docs                             |
| `code-analysis`   | Pagsusuri ng LOC threshold (1000 linya)                       | Mga PR lamang                                |
| `secrets`         | Tukuyin ang mga na-leak na lihim                                                 | Palagi                                       |
| `build-artifacts` | Buuin ang dist nang isang beses, ibahagi sa ibang mga job                        | Hindi docs, mga pagbabago sa node            |
| `release-check`   | I-validate ang nilalaman ng npm pack                                             | Pagkatapos ng build                          |
| `checks`          | Mga test ng Node/Bun + pagsusuri ng protocol                                     | Hindi docs, mga pagbabago sa node            |
| `checks-windows`  | Mga test na partikular sa Windows                                                | Hindi docs, mga pagbabago sa node            |
| `macos`           | Swift lint/build/test + mga TS test                                              | Mga PR na may pagbabago sa macos             |
| `android`         | Gradle build + mga test                                                          | Mga pagbabagong hindi dokumentasyon, android |

## Fail-Fast na Pagkakasunod-sunod

Inaayos ang pagkakasunod-sunod ng mga job upang mabigo muna ang murang mga pagsusuri bago patakbuhin ang mga mas mahal:

1. `docs-scope` + `code-analysis` + `check` (parallel, ~1-2 min)
2. `build-artifacts` (nakaharang hanggang matapos ang nasa itaas)
3. `checks`, `checks-windows`, `macos`, `android` (nakaharang sa build)

## Mga Runner

| Runner                          | Mga Job                                        |
| ------------------------------- | ---------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | Karamihan sa mga Linux job                     |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                               |
| `macos-latest`                  | `macos`, `ios`                                 |
| `ubuntu-latest`                 | Pagtukoy ng saklaw (magaan) |

## Mga Lokal na Katumbas

```bash
pnpm check          # types + lint + format
pnpm test           # mga vitest test
pnpm check:docs     # format ng docs + lint + sirang mga link
pnpm release:check  # i-validate ang npm pack
```

