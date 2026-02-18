---
title: "Sous-agents"
---

# Sous-agents

Les sous-agents sont des exécutions d’agent en arrière-plan lancées à partir d’une exécution d’agent existante. Ils s’exécutent dans leur propre session (`agent:<agentId>:subagent:<uuid>`) et, une fois terminés, **annoncent** leur résultat dans le canal de chat demandeur.

## Commande slash

Utilisez `/subagents` pour inspecter ou contrôler les exécutions de sous-agents pour la **session en cours** :

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` affiche les métadonnées d’exécution (statut, horodatages, identifiant de session, chemin de transcription, nettoyage).

Objectifs principaux :

- Paralléliser les tâches de « recherche / tâche longue / outil lent » sans bloquer l’exécution principale.
- Garder les sous-agents isolés par défaut (séparation de session + sandboxing optionnel).
- Rendre la surface d’outils difficile à mal utiliser : les sous-agents ne reçoivent **pas** les outils de session par défaut.
- Prendre en charge une profondeur d’imbrication configurable pour les modèles d’orchestration.

Note sur les coûts : chaque sous-agent possède **son propre** contexte et sa propre consommation de tokens. Pour les tâches lourdes ou répétitives, définissez un modèle moins coûteux pour les sous-agents et conservez un modèle de meilleure qualité pour votre agent principal.  
Vous pouvez configurer cela via `agents.defaults.subagents.model` ou via des surcharges par agent.

## Outil

Utilisez `sessions_spawn` :

- Démarre une exécution de sous-agent (`deliver: false`, voie globale : `subagent`)
- Exécute ensuite une étape d’annonce et publie la réponse d’annonce dans le canal de chat demandeur
- Modèle par défaut : hérite de l’appelant sauf si vous définissez `agents.defaults.subagents.model` (ou `agents.list[].subagents.model`) ; un `sessions_spawn.model` explicite a toujours priorité.
- Réflexion par défaut : hérite de l’appelant sauf si vous définissez `agents.defaults.subagents.thinking` (ou `agents.list[].subagents.thinking`) ; un `sessions_spawn.thinking` explicite a toujours priorité.

Paramètres de l’outil :

- `task` (obligatoire)
- `label?` (optionnel)
- `agentId?` (optionnel ; lancer sous un autre identifiant d’agent si autorisé)
- `model?` (optionnel ; remplace le modèle du sous-agent ; les valeurs invalides sont ignorées et le sous-agent s’exécute avec le modèle par défaut avec un avertissement dans le résultat de l’outil)
- `thinking?` (optionnel ; remplace le niveau de réflexion pour l’exécution du sous-agent)
- `runTimeoutSeconds?` (par défaut `0` ; si défini, l’exécution du sous-agent est interrompue après N secondes)
- `cleanup?` (`delete|keep`, par défaut `keep`)

Liste d’autorisation :

- `agents.list[].subagents.allowAgents` : liste des identifiants d’agents pouvant être ciblés via `agentId` (`["*"]` pour autoriser tous). Par défaut : uniquement l’agent demandeur.

Découverte :

- Utilisez `agents_list` pour voir quels identifiants d’agents sont actuellement autorisés pour `sessions_spawn`.

Archivage automatique :

- Les sessions des sous-agents sont automatiquement archivées après `agents.defaults.subagents.archiveAfterMinutes` (par défaut : 60).
- L’archivage utilise `sessions.delete` et renomme la transcription en `*.deleted.<timestamp>` (dans le même dossier).
- `cleanup: "delete"` archive immédiatement après l’annonce (la transcription est conservée via renommage).
- L’archivage automatique est exécuté au mieux ; les minuteurs en attente sont perdus si la passerelle redémarre.
- `runTimeoutSeconds` n’archive **pas** automatiquement ; il arrête seulement l’exécution. La session reste jusqu’à l’archivage automatique.
- L’archivage automatique s’applique aussi bien aux sessions de profondeur 1 que de profondeur 2.

## Sous-agents imbriqués

Par défaut, les sous-agents ne peuvent pas lancer leurs propres sous-agents (`maxSpawnDepth: 1`). Vous pouvez activer un niveau d’imbrication en définissant `maxSpawnDepth: 2`, ce qui permet le **modèle d’orchestration** : principal → sous-agent orchestrateur → sous-sous-agents travailleurs.

### Comment activer

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // autoriser les sous-agents à lancer des enfants (par défaut : 1)
        maxChildrenPerAgent: 5, // nombre maximal d’enfants actifs par session d’agent (par défaut : 5)
        maxConcurrent: 8, // limite globale de concurrence sur la voie (par défaut : 8)
      },
    },
  },
}
```

### Niveaux de profondeur

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Agent principal                               | Toujours                     |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sous-agent (orchestrateur si profondeur 2 autorisée) | Seulement si `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sous-sous-agent (travailleur feuille)         | Jamais                       |

### Chaîne d’annonce

Les résultats remontent la chaîne :

1. Le travailleur de profondeur 2 termine → annonce à son parent (orchestrateur de profondeur 1)
2. L’orchestrateur de profondeur 1 reçoit l’annonce, synthétise les résultats, termine → annonce au principal
3. L’agent principal reçoit l’annonce et la transmet à l’utilisateur

Chaque niveau ne voit que les annonces de ses enfants directs.

### Politique d’outils par profondeur

- **Profondeur 1 (orchestrateur, lorsque `maxSpawnDepth >= 2`)** : reçoit `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` afin de gérer ses enfants. Les autres outils de session/système restent refusés.
- **Profondeur 1 (feuille, lorsque `maxSpawnDepth == 1`)** : aucun outil de session (comportement par défaut actuel).
- **Profondeur 2 (travailleur feuille)** : aucun outil de session — `sessions_spawn` est toujours refusé à la profondeur 2. Impossible de lancer d’autres enfants.

### Limite de création par agent

Chaque session d’agent (à n’importe quelle profondeur) peut avoir au maximum `maxChildrenPerAgent` (par défaut : 5) enfants actifs simultanément. Cela évite une explosion incontrôlée depuis un seul orchestrateur.

### Arrêt en cascade

L’arrêt d’un orchestrateur de profondeur 1 arrête automatiquement tous ses enfants de profondeur 2 :

- `/stop` dans le chat principal arrête tous les agents de profondeur 1 et se propage à leurs enfants de profondeur 2.
- `/subagents kill <id>` arrête un sous-agent spécifique et se propage à ses enfants.
- `/subagents kill all` arrête tous les sous-agents du demandeur et se propage.

## Authentification

L’authentification des sous-agents est résolue par **identifiant d’agent**, et non par type de session :

- La clé de session du sous-agent est `agent:<agentId>:subagent:<uuid>`.
- Le magasin d’authentification est chargé depuis le `agentDir` de cet agent.
- Les profils d’authentification de l’agent principal sont fusionnés en tant que **repli** ; les profils de l’agent remplacent ceux du principal en cas de conflit.

Remarque : la fusion est additive, donc les profils du principal sont toujours disponibles en repli. Une isolation complète de l’authentification par agent n’est pas encore prise en charge.

## Annonce

Les sous-agents rapportent via une étape d’annonce :

- L’étape d’annonce s’exécute dans la session du sous-agent (pas dans la session du demandeur).
- Si le sous-agent répond exactement `ANNOUNCE_SKIP`, rien n’est publié.
- Sinon, la réponse d’annonce est publiée dans le canal de chat demandeur via un appel `agent` de suivi (`deliver=true`).
- Les réponses d’annonce conservent l’acheminement par fil/sujet lorsque disponible (fils Slack, sujets Telegram, fils Matrix).
- Les messages d’annonce sont normalisés selon un modèle stable :
  - `Status:` dérivé du résultat d’exécution (`success`, `error`, `timeout` ou `unknown`).
  - `Result:` le contenu résumé de l’étape d’annonce (ou `(not available)` si absent).
  - `Notes:` détails d’erreur et autres informations utiles.
- `Status` n’est pas déduit de la sortie du modèle ; il provient des signaux d’exécution.

Les charges utiles d’annonce incluent une ligne de statistiques à la fin (même si encapsulées) :

- Durée d’exécution (ex. `runtime 5m12s`)
- Consommation de tokens (entrée/sortie/total)
- Coût estimé lorsque la tarification du modèle est configurée (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` et chemin de transcription (afin que l’agent principal puisse récupérer l’historique via `sessions_history` ou inspecter le fichier sur disque)

## Politique d’outils (outils des sous-agents)

Par défaut, les sous-agents reçoivent **tous les outils sauf les outils de session** et les outils système :

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Lorsque `maxSpawnDepth >= 2`, les sous-agents orchestrateurs de profondeur 1 reçoivent en plus `sessions_spawn`, `subagents`, `sessions_list` et `sessions_history` afin de gérer leurs enfants.

Remplacement via configuration :

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny l’emporte
        deny: ["gateway", "cron"],
        // si allow est défini, il devient une liste blanche (deny l’emporte toujours)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrence

Les sous-agents utilisent une voie de file d’attente dédiée en mémoire :

- Nom de la voie : `subagent`
- Concurrence : `agents.defaults.subagents.maxConcurrent` (par défaut `8`)

## Arrêt

- Envoyer `/stop` dans le chat demandeur interrompt la session demandeur et arrête toute exécution active de sous-agent lancée depuis celle-ci, avec propagation aux enfants imbriqués.
- `/subagents kill <id>` arrête un sous-agent spécifique et se propage à ses enfants.

## Limitations

- L’annonce des sous-agents est **au mieux**. Si la passerelle redémarre, les annonces en attente sont perdues.
- Les sous-agents partagent toujours les ressources du même processus de passerelle ; considérez `maxConcurrent` comme une soupape de sécurité.
- `sessions_spawn` est toujours non bloquant : il retourne immédiatement `{ status: "accepted", runId, childSessionKey }`.
- Le contexte du sous-agent injecte uniquement `AGENTS.md` + `TOOLS.md` (pas de `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ou `BOOTSTRAP.md`).
- La profondeur maximale d’imbrication est 5 (`maxSpawnDepth` plage : 1–5). La profondeur 2 est recommandée pour la plupart des cas d’usage.
- `maxChildrenPerAgent` limite le nombre d’enfants actifs par session (par défaut : 5, plage : 1–20).