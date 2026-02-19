---
summary: "Sous-agents : lancement d’exécutions d’agents isolées qui annoncent les résultats au chat demandeur"
read_when:
  - Vous souhaitez un travail en arrière-plan/parallèle via l’agent
  - Vous modifiez la politique sessions_spawn ou l’outil de sous-agent
title: "Sous-agents"
---

# Sous-agents

Les sous-agents vous permettent d’exécuter des tâches en arrière-plan sans bloquer la conversation principale. Lorsque vous créez un sous-agent, il s’exécute dans sa propre session isolée, effectue son travail et annonce le résultat dans le chat une fois terminé.

## Commande slash

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` affiche les métadonnées d’exécution (statut, horodatages, identifiant de session, chemin de transcription, nettoyage).

Objectifs principaux :

- Paralléliser le travail de type « recherche / tâche longue / outil lent » sans bloquer l’exécution principale.
- Garder les sous-agents isolés par défaut (séparation des sessions + sandboxing optionnel).
- Maintenir une surface d’outils difficile à détourner : les sous-agents n’obtiennent **pas** les outils de session par défaut.
- Prendre en charge une profondeur d’imbrication configurable pour les modèles d’orchestrateur.

Chaque sous-agent possède **son propre** contexte et sa propre consommation de jetons. Pour les tâches lourdes ou répétitives,
définissez un modèle moins coûteux pour les sous-agents et conservez votre agent principal sur un modèle de meilleure qualité.
Configuration par agent : `agents.list[].subagents.model`

## #> [limit] [tools]\`

Paramètre `thinking` explicite dans l’appel `sessions_spawn`

- Démarre une exécution de sous-agent (`deliver: false`, voie globale : `subagent`)
- Exécute ensuite une étape d’annonce et publie la réponse d’annonce sur le canal de discussion du demandeur
- Paramètre `model` explicite dans l’appel `sessions_spawn`
- Raisonnement : aucune surcharge spécifique au sous-agent (sauf si `subagents.thinking` est défini) Valeurs par défaut :

Paramètres de l’outil :

- `task`
- `label`
- Créer sous un identifiant d’agent différent (doit être autorisé)
- Les valeurs de modèle invalides sont ignorées silencieusement — le sous-agent s’exécute avec la prochaine valeur par défaut valide, avec un avertissement dans le résultat de l’outil.
</Note>
- `thinking?` (optionnel ; remplace le niveau de réflexion pour l’exécution du sous-agent)
- Automatically aborts the sub-agent run after the specified time
- `"delete"` | `"keep"`

Liste d’autorisation :

- `agents.list[].subagents.allowAgents` : liste des identifiants d’agents pouvant être ciblés via `agentId` (`["*"]` pour autoriser n’importe lequel). Par défaut : uniquement l’agent demandeur.

Découverte :

- Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.

Archivage automatique : après 60 minutes

- Les sessions des sous-agents sont automatiquement archivées après une période configurable :
- L’archivage renomme la transcription en \`\*.deleted.<timestamp> (même dossier).
- `"delete"` archive immédiatement après l’annonce
- - **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
- `runTimeoutSeconds` does **not** auto-archive the session. La session reste active jusqu’à ce que le minuteur d’archivage normal se déclenche.
- L’archivage automatique s’applique de la même manière aux sessions de profondeur 1 et de profondeur 2.

## Stop a running sub-agent

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

### Comment activer

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Niveaux de profondeur

| Profondeur | Format de la clé de session                                                                                                                                                                                                                                                                                                                                                                                                  | Rôle                                                                                | Peut créer ?                       |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------- |
| 0          | Configuration par agent : `agents.list[].subagents.thinking`                                                                                                                                                                                                                                                                                                                                                 | Agent principal                                                                     | Toujours                           |
| 1          | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | Sous-agent (orchestrateur lorsque la profondeur 2 est autorisée) | Uniquement si `maxSpawnDepth >= 2` |
| 2          | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Managing Sub-Agents (`/subagents`)                               | Jamais                             |

### Chaîne d’annonce

Les résultats remontent la chaîne :

1. Le worker de profondeur 2 termine → annonce à son parent (orchestrateur de profondeur 1)
2. L’orchestrateur de profondeur 1 reçoit l’annonce, synthétise les résultats, termine → annonce au principal
3. L’agent principal reçoit l’annonce et la transmet à l’utilisateur

Chaque niveau ne voit que les annonces de ses enfants directs.

### Tool Policy

- **Profondeur 1 (orchestrateur, lorsque `maxSpawnDepth >= 2`)** : Reçoit `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` afin de pouvoir gérer ses enfants. Les autres outils de session/système restent refusés.
- **Profondeur 1 (feuille, lorsque `maxSpawnDepth == 1`)** : Aucun outil de session (comportement par défaut actuel).
- **Profondeur 2 (worker feuille)** : Aucun outil de session — `sessions_spawn` est toujours refusé en profondeur 2. Ne peut pas créer d’enfants supplémentaires.

### Cross-Agent Spawning

Chaque session d’agent (à n’importe quelle profondeur) peut avoir au maximum `maxChildrenPerAgent` (par défaut : 5) enfants actifs en même temps. Cela empêche une prolifération incontrôlée à partir d’un seul orchestrateur.

### Arrêt en cascade

L’arrêt d’un orchestrateur de profondeur 1 arrête automatiquement tous ses enfants de profondeur 2 :

- `/stop` dans le chat principal arrête tous les agents de profondeur 1 et se propage à leurs enfants de profondeur 2.
- `/subagents stop <id>`
- `/subagents kill all` arrête tous les sous-agents du demandeur et se propage.

## Authentification

L’authentification des sous-agents est résolue par **identifiant d’agent**, et non par type de session :

- La clé de session du sous-agent est `agent:<agentId>:subagent:<uuid>`.
- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)

The merge is additive — main profiles are always available as fallbacks
Fully isolated auth per sub-agent is not currently supported.

## Announce Status

Les sous-agents font un rapport via une étape d’annonce :

- L’étape d’annonce s’exécute dans la session du sous-agent (et non dans la session du demandeur).
- If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted. This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
- Sinon, la réponse d’annonce est publiée sur le canal de chat du demandeur via un appel `agent` de suivi (`deliver=true`).
- Les réponses d’annonce conservent l’acheminement par fil/sujet lorsque disponible (fils Slack, sujets Telegram, fils Matrix).
- Les messages d’annonce sont normalisés selon un modèle stable :
  - `Status:` dérivé du résultat de l’exécution (`success`, `error`, `timeout` ou `unknown`).
  - `Result:` le contenu résumé issu de l’étape d’annonce (ou `(not available)` s’il est absent).
  - `Notes:` détails des erreurs et autres informations utiles.
- The announce message includes a status derived from the runtime outcome (not from model output):

Each announce includes a stats line with:

- Durée d’exécution (par ex., `runtime 5m12s`)
- Consommation de tokens (entrée/sortie/total)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` et chemin du transcript (afin que l’agent principal puisse récupérer l’historique via `sessions_history` ou inspecter le fichier sur le disque)

## Customizing Sub-Agent Tools

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

-   
</Accordion>
- `runTimeoutSeconds`
- \#> <message>\`
- L’outil `sessions_spawn`

Lorsque `maxSpawnDepth >= 2`, les sous-agents orchestrateurs de profondeur 1 reçoivent en plus `sessions_spawn`, `subagents`, `sessions_list` et `sessions_history` afin de pouvoir gérer leurs enfants.

Remplacement via la configuration :

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

## Concurrence

Les sous-agents utilisent une file dédiée en mémoire :

- Nom de la file : `subagent`
- 7. {
     agents: {
     defaults: {
     subagents: {
     maxConcurrent: 4, // default: 8
     },
     },
     },
     }

## Arrêt

- L’envoi de `/stop` dans le chat du demandeur interrompt la session du demandeur et arrête toutes les exécutions actives de sous-agents lancées depuis celle-ci, avec propagation aux enfants imbriqués.
- Aborts the main session **and** all active sub-agent runs spawned from it

## Limitations

- L’annonce du sous-agent est en **best-effort**. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- L’appel est **non bloquant** — l’agent principal reçoit immédiatement `{ status: "accepted", runId, childSessionKey }`.
- Le contexte du sous-agent injecte uniquement `AGENTS.md` + `TOOLS.md` (pas de `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ni `BOOTSTRAP.md`).
- La profondeur maximale d’imbrication est de 5 (plage `maxSpawnDepth` : 1–5). Une profondeur de 2 est recommandée pour la plupart des cas d’usage.
- `maxChildrenPerAgent` limite le nombre d’enfants actifs par session (par défaut : 5, plage : 1–20).

