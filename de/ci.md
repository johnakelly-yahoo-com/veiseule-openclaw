---
title: CI-Pipeline
description: Wie die OpenClaw CI-Pipeline funktioniert
---

# CI-Pipeline

Die CI läuft bei jedem Push auf `main` und bei jedem Pull Request. Sie verwendet intelligentes Scoping, um aufwendige Jobs zu überspringen, wenn nur Dokumentation oder nativer Code geändert wurden.

## Job-Übersicht

| Job               | Zweck                                                                             | Wann er ausgeführt wird                           |
| ----------------- | --------------------------------------------------------------------------------- | ------------------------------------------------- |
| `docs-scope`      | Änderungen nur an der Dokumentation erkennen                                      | Immer                                             |
| `changed-scope`   | Erkennen, welche Bereiche geändert wurden (node/macos/android) | PRs ohne Dokumentationsänderungen                 |
| `check`           | TypeScript-Typen, Linting, Formatierung                                           | Änderungen ohne Dokumentation                     |
| `check-docs`      | Markdown-Lint + Prüfung auf defekte Links                                         | Dokumentation geändert                            |
| `code-analysis`   | Überprüfung des LOC-Schwellenwerts (1000 Zeilen)               | Nur PRs                                           |
| `secrets`         | Erkennen von offengelegten Geheimnissen                                           | Immer                                             |
| `build-artifacts` | Dist einmal bauen und mit anderen Jobs teilen                                     | Ohne Dokumentation, Node-Änderungen               |
| `release-check`   | Inhalt von npm pack validieren                                                    | Nach dem Build                                    |
| `checks`          | Node/Bun-Tests + Protokollprüfung                                                 | Ohne Dokumentation, Node-Änderungen               |
| `checks-windows`  | Windows-spezifische Tests                                                         | Ohne Dokumentation, Node-Änderungen               |
| `macos`           | Swift-Lint/Build/Tests + TS-Tests                                                 | PRs mit macos-Änderungen                          |
| `android`         | Gradle-Build + Tests                                                              | Änderungen außerhalb der Doku, Android-Änderungen |

## Fail-Fast-Reihenfolge

Jobs sind so angeordnet, dass günstige Prüfungen fehlschlagen, bevor teure ausgeführt werden:

1. `docs-scope` + `code-analysis` + `check` (parallel, ~1–2 Min.)
2. `build-artifacts` (blockiert durch obige Schritte)
3. `checks`, `checks-windows`, `macos`, `android` (blockiert durch Build)

## Runner

| Jobs                                                 | `blacksmith-4vcpu-ubuntu-2404`  |
| ---------------------------------------------------- | ------------------------------- |
| Die meisten Linux-Jobs                               | `blacksmith-4vcpu-windows-2025` |
| `checks-windows`                                     | `macos-latest`                  |
| `macos`, `ios`                                       | `ubuntu-latest`                 |
| Scope-Erkennung (leichtgewichtig) | Lokale Entsprechungen           |

pnpm check          # Typen + Lint + Format
pnpm test           # vitest-Tests
pnpm check:docs     # Doku-Format + Lint + defekte Links
pnpm release:check  # npm pack validieren
---------------------------------------------------------

```bash
Npm-Spezifikationen sind nur für die Registry bestimmt (Paketname + optionale Version/Tag).
```

