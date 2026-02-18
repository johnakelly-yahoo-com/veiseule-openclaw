---
title: "Compactage"
---

# Fenêtre de contexte & compaction

Chaque modèle possède une **fenêtre de contexte** (nombre maximal de tokens qu’il peut voir). Les discussions de longue durée accumulent des messages et des résultats d’outils ; lorsque la fenêtre devient contrainte, OpenClaw **compacte** l’historique plus ancien pour rester dans les limites.

## Ce qu’est la compaction

La compaction **résume les conversations plus anciennes** en une entrée de synthèse compacte et conserve les messages récents intacts. Le résumé est stocké dans l’historique de la session, de sorte que les requêtes futures utilisent :

- Le résumé de compaction
- Les messages récents après le point de compaction

La compaction **persiste** dans l’historique JSONL de la session.

## Configuration

Voir [Configuration et modes de compaction](/concepts/compaction) pour les paramètres `agents.defaults.compaction`.

## Auto-compaction (activée par défaut)

Lorsqu’une session approche ou dépasse la fenêtre de contexte du modèle, OpenClaw déclenche l’auto-compaction et peut réessayer la requête initiale en utilisant le contexte compacté.

Vous verrez :

- `🧹 Auto-compaction complete` en mode verbeux
- `/status` indiquant `🧹 Compactions: <count>`

Avant la compaction, OpenClaw peut exécuter un tour de **vidage silencieux de la mémoire** afin d’enregistrer des notes durables sur le disque. Voir [Mémoire](/concepts/memory) pour les détails et la configuration.

## Compaction manuelle

Utilisez `/compact` (éventuellement avec des instructions) pour forcer un passage de compaction :

```
/compact Focus on decisions and open questions
```

## Source de la fenêtre de contexte

La fenêtre de contexte est spécifique au modèle. OpenClaw utilise la définition du modèle issue du catalogue de fournisseurs configuré pour déterminer les limites.

## Compaction vs élagage

- **Compaction** : résume et **persiste** en JSONL.
- **Élagage de session** : supprime uniquement les **résultats d’outils** anciens, **en mémoire**, par requête.

Voir [/concepts/session-pruning](/concepts/session-pruning) pour les détails sur l’élagage.

## Conseils

- Utilisez `/compact` lorsque les sessions semblent obsolètes ou que le contexte est encombré.
- Les sorties d’outils volumineuses sont déjà tronquées ; l’élagage peut réduire davantage l’accumulation des résultats d’outils.
- Si vous avez besoin d’une page blanche, `/new` ou `/reset` démarre un nouvel identifiant de session.


