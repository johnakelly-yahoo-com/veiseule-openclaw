---
title: "Sous-agents"
---

# Sous-agents

Les sous-agents vous permettent d’exécuter des tâches en arrière-plan sans bloquer la conversation principale. Lorsque vous créez un sous-agent, il s’exécute dans sa propre session isolée, effectue son travail et annonce le résultat dans le chat une fois terminé.

**Cas d’utilisation :**

- Rechercher un sujet pendant que l’agent principal continue de répondre aux questions
- Exécuter plusieurs tâches longues en parallèle (scraping web, analyse de code, traitement de fichiers)
- Déléguer des tâches à des agents spécialisés dans une configuration multi-agents

## Demarrage rapide

La manière la plus simple d’utiliser des sous-agents est de demander naturellement à votre agent :

> "Créer un sous-agent pour rechercher les dernières notes de version de Node.js"

L’agent appellera l’outil `sessions_spawn` en arrière-plan. Lorsque le sous-agent a terminé, il annonce ses conclusions dans votre chat.

Vous pouvez également être explicite sur les options :

> "Créer un sous-agent pour analyser les journaux du serveur d’aujourd’hui. Utilisez gpt-5.2 et définissez un délai d’expiration de 5 minutes."

## Fonctionnement

<Steps>
  <Step title="Main agent spawns">
    L’agent principal appelle `sessions_spawn` avec une description de la tâche. L’appel est **non bloquant** — l’agent principal reçoit immédiatement `{ status: "accepted", runId, childSessionKey }`.
  </Step>
  <Step title="Sub-agent runs in the background"> 
    Une nouvelle session isolée est créée (`agent:
    :subagent:
    `) sur la voie de file d’attente dédiée `subagent`.
  <agentId>Lorsque le sous-agent a terminé, il annonce ses conclusions au chat demandeur.<uuid>L’agent principal publie un résumé en langage naturel.</Step>
  <Step title="Result is announced">
    La session du sous-agent est automatiquement archivée après 60 minutes (configurable). Les transcriptions sont conservées.
  </Step>
  <Step title="Session is archived">
    Chaque sous-agent possède **son propre** contexte et sa propre consommation de jetons. Définissez un modèle moins coûteux pour les sous-agents afin d’économiser des coûts — voir [Définir un modèle par défaut](#setting-a-default-model) ci-dessous.
  </Step>
</Steps>

<Tip>
Les sous-agents fonctionnent immédiatement sans configuration. Modèle : sélection normale du modèle de l’agent cible (sauf si `subagents.model` est défini)
</Tip>

## Configuration

Raisonnement : aucune surcharge spécifique au sous-agent (sauf si `subagents.thinking` est défini) Valeurs par défaut :

- Concurrence maximale : 8
- Archivage automatique : après 60 minutes
- Définir un modèle par défaut
- Utilisez un modèle moins coûteux pour les sous-agents afin de réduire les coûts de jetons :

### {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;model: "minimax/MiniMax-M2.1",&#xA;},&#xA;},&#xA;},&#xA;}

Use a cheaper model for sub-agents to save on token costs:

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

### Définir un niveau de réflexion par défaut

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### 3. Surcharges par agent

4. Dans une configuration multi-agents, vous pouvez définir des valeurs par défaut de sous-agents pour chaque agent :

```json5
5. {
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### Concurrence

6. Contrôlez combien de sous-agents peuvent s’exécuter en même temps :

```json5
7. {
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

8. Les sous-agents utilisent une voie de file d’attente dédiée (`subagent`) distincte de la file d’attente de l’agent principal, de sorte que les exécutions des sous-agents ne bloquent pas les réponses entrantes.

### 9. Archivage automatique

Les sessions des sous-agents sont automatiquement archivées après une période configurable :

```json5
11. {
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>12. L’archivage renomme la transcription en `*.deleted.<timestamp>` (même dossier) — les transcriptions sont conservées, non supprimées. 14. Les minuteurs d’archivage automatique sont exécutés au mieux ; les minuteurs en attente sont perdus si la passerelle redémarre.
</Note>

## L’outil `sessions_spawn`

16. Il s’agit de l’outil que l’agent appelle pour créer des sous-agents.

### Parametres

| 17. Paramètre           | Type                                              | Par défaut                                                        | Description                                                                                                                           |
| ---------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 18. `task`              | string                                            | 19. _(obligatoire)_     | 20. Ce que le sous-agent doit faire                                                                            |
| 21. `label`             | string                                            | 22. —                                      | 23. Libellé court pour l’identification                                                                        |
| 24. `agentId`           | string                                            | 25. _(agent appelant)_  | 26. Créer sous un identifiant d’agent différent (doit être autorisé)                        |
| `modèle`                                       | string                                            | 27. _(optionnel)_       | 28. Remplacer le modèle pour ce sous-agent                                                                     |
| 29. `thinking`          | string                                            | 30. _(optionnel)_       | 31. Remplacer le niveau de réflexion (`off`, `low`, `medium`, `high`, etc.) |
| 32. `runTimeoutSeconds` | 33. nombre                 | 34. `0` (aucune limite) | 35. Interrompre le sous-agent après N secondes                                                                 |
| `nettoyage`                                    | 36. `"delete"` \| `"keep"` | 37. `"keep"`                               | 38. `"delete"` archive immédiatement après l’annonce                                                           |

### 39. Ordre de résolution du modèle

40. Le modèle du sous-agent est résolu dans cet ordre (la première correspondance l’emporte) :

1. 41. Paramètre `model` explicite dans l’appel `sessions_spawn`
2. 42. Configuration par agent : `agents.list[].subagents.model`
3. 43. Valeur par défaut globale : `agents.defaults.subagents.model`
4. 44. Résolution normale du modèle de l’agent cible pour cette nouvelle session

45) Le niveau de réflexion est résolu dans cet ordre :

1. 46. Paramètre `thinking` explicite dans l’appel `sessions_spawn`
2. 47. Configuration par agent : `agents.list[].subagents.thinking`
3. 48. Valeur par défaut globale : `agents.defaults.subagents.thinking`
4. 49. Sinon, aucune surcharge de réflexion spécifique au sous-agent n’est appliquée

<Note>50. Les valeurs de modèle invalides sont ignorées silencieusement — le sous-agent s’exécute avec la prochaine valeur par défaut valide, avec un avertissement dans le résultat de l’outil.</Note>

### Cross-Agent Spawning

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>
Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.
</Tip>

## Managing Sub-Agents (`/subagents`)

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

| Commande                                   | Description                                                       |
| ------------------------------------------ | ----------------------------------------------------------------- |
| `/subagents list`                          | List all sub-agent runs (active and completed) |
| `/subagents stop <id\\|#\\|all>`         | Stop a running sub-agent                                          |
| `/subagents log <id\\|#> [limit] [tools]` | View sub-agent transcript                                         |
| `/subagents info <id\\|#>`                | Show detailed run metadata                                        |
| `/subagents send <id\\|#> <message>`      | Send a message to a running sub-agent                             |

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">
    ```
    /subagents list
    ```

    ````
    ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">
    ```
    /subagents info 1
    ```

    ````
    ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">
    ```
    /subagents log 1 10
    ```

    ````
    Shows the last 10 messages from the sub-agent's transcript. Add `tools` to include tool call messages:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Sends a message into the running sub-agent's session and waits up to 30 seconds for a reply.
    ```

  </Accordion>
</AccordionGroup>

## Announce (How Results Come Back)

When a sub-agent finishes, it goes through an **announce** step:

1. The sub-agent's final reply is captured
2. A summary message is sent to the main agent's session with the result, status, and stats
3. The main agent posts a natural-language summary to your chat

Les réponses d’annonce conservent l’acheminement par fil/sujet lorsque disponible (fils Slack, sujets Telegram, fils Matrix).

### Announce Stats

Each announce includes a stats line with:

- Runtime duration
- Consommation de tokens (entrée/sortie/total)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- Session key, session id, and transcript path

### Announce Status

The announce message includes a status derived from the runtime outcome (not from model output):

- **successful completion** (`ok`) — task completed normally
- **error** — task failed (error details in notes)
- **timeout** — task exceeded `runTimeoutSeconds`
- **unknown** — status could not be determined

<Tip>
If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted.
This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
</Tip>

## Tool Policy

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

<AccordionGroup>
  <Accordion title="Default denied tools">
    | Denied tool | Reason |
    |-------------|--------|
    | `sessions_list` | Session management — main agent orchestrates |
    | `sessions_history` | Session management — main agent orchestrates |
    | `sessions_send` | Session management — main agent orchestrates |
    | `sessions_spawn` | No nested fan-out (sub-agents cannot spawn sub-agents) |
    | `gateway` | System admin — dangerous from sub-agent |
    | `agents_list` | System admin |
    | `whatsapp_login` | Interactive setup — not a task |
    | `session_status` | Status/scheduling — main agent coordinates |
    | `cron` | Status/scheduling — main agent coordinates |
    | `memory_search` | Pass relevant info in spawn prompt instead |
    | `memory_get` | Pass relevant info in spawn prompt instead |
  </Accordion>
</AccordionGroup>

### Customizing Sub-Agent Tools

You can further restrict sub-agent tools:

```json5
{
  tools: {
    subagents: {
      tools: {
        // deny always wins over allow
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

To restrict sub-agents to **only** specific tools:

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

<Note>
Custom deny entries are **added to** the default deny list. If `allow` is set, only those tools are available (the default deny list still applies on top).
</Note>

## Authentification

L’authentification des sous-agents est résolue par **identifiant d’agent**, et non par type de session :

- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)
- The merge is additive — main profiles are always available as fallbacks

<Note>
Fully isolated auth per sub-agent is not currently supported.
</Note>

## Contexte et prompt système

Sub-agents receive a reduced system prompt compared to the main agent:

- **Included:** Tooling, Workspace, Runtime sections, plus `AGENTS.md` and `TOOLS.md`
- **Not included:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

The sub-agent also receives a task-focused system prompt that instructs it to stay focused on the assigned task, complete it, and not act as the main agent.

## Arrêter les sous-agents

| Méthode                | Effect                                                                    |
| ---------------------- | ------------------------------------------------------------------------- |
| `/stop` in the chat    | Aborts the main session **and** all active sub-agent runs spawned from it |
| `/subagents stop <id>` | Stops a specific sub-agent without affecting the main session             |
| `runTimeoutSeconds`    | Automatically aborts the sub-agent run after the specified time           |

<Note>
`runTimeoutSeconds` does **not** auto-archive the session. La session reste active jusqu’à ce que le minuteur d’archivage normal se déclenche.
</Note>

## Exemple de configuration complète

<Accordion title="Complete sub-agent configuration">
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops can spawn sub-agents under "main"
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agents can't use the browser
      },
    },
  },
}
```
</Accordion>

## Limitations

<Warning>
- **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
</Warning>

## Voir aussi

- [Session Tools](/concepts/session-tool) — details on `sessions_spawn` and other session tools
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — per-agent tool restrictions and sandboxing
- [Configuration](/gateway/configuration) — `agents.defaults.subagents` reference
- [Queue](/concepts/queue) — how the `subagent` lane works
