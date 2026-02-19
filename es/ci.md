---
title: Pipeline de CI
description: Cómo funciona el pipeline de CI de OpenClaw
---

# Pipeline de CI

La CI se ejecuta en cada push a `main` y en cada pull request. Utiliza un alcance inteligente para omitir trabajos costosos cuando solo cambian la documentación o el código nativo.

## Resumen de trabajos

| Trabajo           | Propósito                                                            | Cuándo se ejecuta                                     |
| ----------------- | -------------------------------------------------------------------- | ----------------------------------------------------- |
| `docs-scope`      | Detectar cambios solo en la documentación                            | Siempre                                               |
| `changed-scope`   | Detectar qué áreas cambiaron (node/macos/android) | PRs que no son de documentación                       |
| `check`           | Tipos de TypeScript, lint, formato                                   | Cambios que no son de documentación                   |
| `check-docs`      | Lint de Markdown + verificación de enlaces rotos                     | Cambios en la documentación                           |
| `code-analysis`   | Verificación del umbral de LOC (1000 líneas)      | Solo PRs                                              |
| `secrets`         | Detectar secretos filtrados                                          | Siempre                                               |
| `build-artifacts` | Construir dist una vez y compartirlo con otros trabajos              | Cambios que no son de documentación, cambios en node  |
| `release-check`   | Validar el contenido de npm pack                                     | Después de la compilación                             |
| `checks`          | Pruebas de Node/Bun + verificación de protocolo                      | Cambios que no son de documentación, cambios en node  |
| `checks-windows`  | Pruebas específicas de Windows                                       | Cambios que no son de documentación, cambios en node  |
| `macos`           | Lint/compilación/pruebas de Swift + pruebas de TS                    | PRs con cambios en macos                              |
| `android`         | Compilación de Gradle + pruebas                                      | Cambios no relacionados con la documentación, android |

## Orden Fail-Fast

Los trabajos se ordenan para que las comprobaciones económicas fallen antes de ejecutar las más costosas:

1. `docs-scope` + `code-analysis` + `check` (paralelo, ~1-2 min)
2. `build-artifacts` (bloqueado por los anteriores)
3. `checks`, `checks-windows`, `macos`, `android` (bloqueados por la compilación)

## Runners

| Runner                          | Trabajos                                         |
| ------------------------------- | ------------------------------------------------ |
| `blacksmith-4vcpu-ubuntu-2404`  | La mayoría de los trabajos en Linux              |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                                 |
| `macos-latest`                  | `macos`, `ios`                                   |
| `ubuntu-latest`                 | Detección de alcance (ligera) |

## Equivalentes locales

```bash
pnpm check          # tipos + lint + formato
pnpm test           # pruebas vitest
pnpm check:docs     # formato + lint + enlaces rotos en la documentación
pnpm release:check  # validar npm pack
```
