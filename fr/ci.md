---
title: Pipeline CI
description: Fonctionnement du pipeline CI OpenClaw
---

# Pipeline CI

Le CI s’exécute à chaque push vers `main` et à chaque pull request. Il utilise un ciblage intelligent pour ignorer les tâches coûteuses lorsque seuls la documentation ou le code natif ont été modifiés.

## Aperçu des tâches

| Tâche             | Objectif                                                                         | Quand elle s’exécute                               |
| ----------------- | -------------------------------------------------------------------------------- | -------------------------------------------------- |
| `docs-scope`      | Détecter les modifications uniquement dans la documentation                      | Toujours                                           |
| `changed-scope`   | Détecter quelles zones ont été modifiées (node/macos/android) | PR hors documentation                              |
| `check`           | Types TypeScript, lint, formatage                                                | Modifications hors documentation                   |
| `check-docs`      | Lint Markdown + vérification des liens brisés                                    | Documentation modifiée                             |
| `code-analysis`   | Vérification du seuil de lignes de code (1000 lignes)         | PR uniquement                                      |
| `secrets`         | Détecter les secrets divulgués                                                   | Toujours                                           |
| `build-artifacts` | Construire dist une seule fois, partager avec les autres tâches                  | Modifications hors documentation, changements node |
| `release-check`   | Valider le contenu de npm pack                                                   | Après la compilation                               |
| `checks`          | Tests Node/Bun + vérification du protocole                                       | Modifications hors documentation, changements node |
| `checks-windows`  | Tests spécifiques à Windows                                                      | Modifications hors documentation, changements node |
| `macos`           | Lint/build/test Swift + tests TS                                                 | PR avec modifications macos                        |
| `android`         | Build Gradle + tests                                                             | Modifications hors documentation, Android          |

## Ordre Fail-Fast

Les jobs sont ordonnés afin que les vérifications peu coûteuses échouent avant l’exécution des plus coûteuses :

1. `docs-scope` + `code-analysis` + `check` (en parallèle, ~1–2 min)
2. `build-artifacts` (bloqué par les précédents)
3. `checks`, `checks-windows`, `macos`, `android` (bloqués par le build)

## Runners

| Runner                          | Jobs                                               |
| ------------------------------- | -------------------------------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | La plupart des jobs Linux                          |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                                   |
| `macos-latest`                  | `macos`, `ios`                                     |
| `ubuntu-latest`                 | Détection du périmètre (légère) |

## Équivalents locaux

```bash
pnpm check          # types + lint + format
pnpm test           # tests vitest
pnpm check:docs     # format + lint de la documentation + liens cassés
pnpm release:check  # valider npm pack
```

