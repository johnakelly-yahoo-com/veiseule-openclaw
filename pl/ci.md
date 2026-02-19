---
title: Potok CI
description: Jak działa potok CI OpenClaw
---

# Potok CI

CI uruchamia się przy każdym pushu do `main` oraz przy każdym pull requeście. Wykorzystuje inteligentne zakresowanie, aby pomijać kosztowne zadania, gdy zmieniono tylko dokumentację lub kod natywny.

## Przegląd zadań

| Zadanie           | Cel                                                                              | Kiedy się uruchamia                             |
| ----------------- | -------------------------------------------------------------------------------- | ----------------------------------------------- |
| `docs-scope`      | Wykrywanie zmian wyłącznie w dokumentacji                                        | Zawsze                                          |
| `changed-scope`   | Wykrywanie, które obszary uległy zmianie (node/macos/android) | PR-y niedotyczące dokumentacji                  |
| `check`           | Typy TypeScript, lintowanie, formatowanie                                        | Zmiany niedotyczące dokumentacji                |
| `check-docs`      | Lintowanie Markdown + sprawdzanie niedziałających linków                         | Zmiany w dokumentacji                           |
| `code-analysis`   | Sprawdzenie progu LOC (1000 linii)                            | Tylko PR-y                                      |
| `secrets`         | Wykrywanie wycieków sekretów                                                     | Zawsze                                          |
| `build-artifacts` | Jednorazowe budowanie dist i udostępnianie innym zadaniom                        | Zmiany niedotyczące dokumentacji, zmiany w node |
| `release-check`   | Walidacja zawartości npm pack                                                    | Po zbudowaniu                                   |
| `checks`          | Testy Node/Bun + sprawdzenie protokołu                                           | Zmiany niedotyczące dokumentacji, zmiany w node |
| `checks-windows`  | Testy specyficzne dla Windows                                                    | Zmiany niedotyczące dokumentacji, zmiany w node |
| `macos`           | Lintowanie/budowanie/testy Swift + testy TS                                      | PR-y ze zmianami w macos                        |
| `android`         | Kompilacja Gradle + testy                                                        | Zmiany niedotyczące dokumentacji, Android       |

## Kolejność Fail-Fast

Zadania są uporządkowane tak, aby tańsze sprawdzenia kończyły się błędem przed uruchomieniem droższych:

1. `docs-scope` + `code-analysis` + `check` (równolegle, ~1–2 min)
2. `build-artifacts` (zablokowane przez powyższe)
3. `checks`, `checks-windows`, `macos`, `android` (zablokowane przez build)

## Runnery

| Runner                          | Zadania                                        |
| ------------------------------- | ---------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | Większość zadań Linux                          |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                               |
| `macos-latest`                  | `macos`, `ios`                                 |
| `ubuntu-latest`                 | Wykrywanie zakresu (lekkie) |

## Lokalne odpowiedniki

```bash
pnpm check          # typy + lint + format
pnpm test           # testy vitest
pnpm check:docs     # formatowanie dokumentacji + lint + niedziałające linki
pnpm release:check  # walidacja npm pack
```

