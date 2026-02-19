---
title: CI-pipeline
description: Så fungerar OpenClaw CI-pipeline
---

# CI-pipeline

CI körs vid varje push till `main` och varje pull request. Den använder smart avgränsning för att hoppa över kostsamma jobb när endast dokumentation eller native-kod har ändrats.

## Jobböversikt

| Jobb              | Syfte                                                                     | När det körs                              |
| ----------------- | ------------------------------------------------------------------------- | ----------------------------------------- |
| `docs-scope`      | Upptäck ändringar endast i dokumentation                                  | Alltid                                    |
| `changed-scope`   | Upptäck vilka områden som ändrats (node/macos/android) | Icke-dokumentations-PR:er |
| `check`           | TypeScript-typer, lint, formatering                                       | Icke-dokumentationsändringar              |
| `check-docs`      | Markdown lint + kontroll av brutna länkar                                 | Dokumentation ändrad                      |
| `code-analysis`   | LOC-tröskelkontroll (1000 rader)                       | Endast PR:er              |
| `secrets`         | Upptäck läckta hemligheter                                                | Alltid                                    |
| `build-artifacts` | Bygg dist en gång, dela med andra jobb                                    | Icke-dokumentation, node-ändringar        |
| `release-check`   | Validera innehållet i npm pack                                            | Efter build                               |
| `checks`          | Node/Bun-tester + protokollkontroll                                       | Icke-dokumentation, node-ändringar        |
| `checks-windows`  | Windows-specifika tester                                                  | Icke-dokumentation, node-ändringar        |
| `macos`           | Swift lint/build/test + TS-tester                                         | PR:er med macos-ändringar |
| `android`         | Gradle‑build + tester                                                     | Icke-dokumentation, Android-ändringar     |

## Fail‑Fast‑ordning

Jobb är ordnade så att billiga kontroller misslyckas innan dyra körs:

1. `docs-scope` + `code-analysis` + `check` (parallellt, ~1–2 min)
2. `build-artifacts` (blockerat av ovanstående)
3. `checks`, `checks-windows`, `macos`, `android` (blockerat av build)

## Runners

| Runner                          | Jobb                                               |
| ------------------------------- | -------------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | De flesta Linux-jobb                               |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                                   |
| `macos-latest`                  | `macos`, `ios`                                     |
| `ubuntu-latest`                 | Omfångsdetektering (lättviktig) |

## Lokala motsvarigheter

```bash
pnpm check          # typer + lint + format
pnpm test           # vitest-tester
pnpm check:docs     # dokumentationsformat + lint + brutna länkar
pnpm release:check  # validera npm pack
```
