---
summary: "Aperçu de la configuration : tâches courantes, configuration rapide et liens vers la référence complète"
read_when:
  - Configurer OpenClaw pour la première fois
  - Rechercher des modèles de configuration courants
  - Naviguer vers des sections spécifiques de la configuration
title: "Configuration"
---

# Configuration

OpenClaw lit une configuration **JSON5** optionnelle depuis `~/.openclaw/openclaw.json` (commentaires + virgules finales autorisés).

Si le fichier est absent, OpenClaw utilise des valeurs par défaut relativement sûres (agent Pi intégré + sessions par expéditeur + espace de travail `~/.openclaw/workspace`). En général, vous n’avez besoin d’une configuration que pour :

- Connecter des canaux et contrôler qui peut envoyer des messages au bot
- Définir les modèles, outils, sandboxing ou automatisations (cron, hooks)
- Ajuster les sessions, médias, réseau ou l’interface utilisateur

Consultez la [référence complète](/gateway/configuration-reference) pour tous les champs disponibles.

<Tip>
`OPENCLAW_CONFIG_PATH` (configuration par instance)
</Tip>

## Configuration minimale (point de départ recommandé)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Configuration 🔧

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # assistant de configuration complet
openclaw configure     # assistant de configuration
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    La Gateway expose une représentation JSON Schema de la configuration via `config.schema` pour les éditeurs d’interface.
    L’interface de contrôle génère un formulaire à partir de ce schéma, avec un éditeur **Raw JSON** comme échappatoire.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (ou `OPENCLAW_CONFIG_PATH`) Le Gateway surveille le fichier et applique automatiquement les modifications (voir [rechargement à chaud](#config-hot-reload)).
  
</Tab>
</Tabs>

## Validation stricte de la configuration

<Warning>
OpenClaw n’accepte que les configurations qui correspondent entièrement au schéma. Les clés inconnues, les types mal formés ou les valeurs invalides amènent la Gateway (passerelle) à **refuser de démarrer** par mesure de sécurité. La seule exception au niveau racine est `$schema` (chaîne), afin que les éditeurs puissent associer des métadonnées JSON Schema.
</Warning>

En cas d’échec de la validation :

- La Gateway ne démarre pas.
- Seules les commandes de diagnostic sont autorisées (par exemple : `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Exécutez `openclaw doctor` pour voir les problèmes exacts.
- Exécutez `openclaw doctor --fix` (ou `--yes`) pour appliquer les migrations/réparations.

## Tâches courantes

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">Chaque canal possède sa propre section de configuration sous `channels.<provider>`. Consultez la page dédiée au canal pour les étapes de configuration :

    ```
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupeAllowDe: ["+15551234567"],
        },
        télégramme : {
          groupPolicy: "allowlist",
          groupAllowDe: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowDe: ["+15551234567"],
        },
        imitation : {
          groupPolicy: "allowlist",
          groupAllowDe: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org. om"],
        },
        discord: {
          groupPolicy: "allowlist",
          guildes: {
            GUILD_ID: {
              canaux: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          canaux: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">Définissez le modèle principal et les solutions de secours facultatives :

    ```
    {
      agents: {
        par défaut: {
          modèles: {
            "anthropic/claude-sonnet-4-5-20250929": {
              paramètres : { temperature: 0.6 },
            },
            "openai/gpt-5. ": {
              paramètres : { maxTokens: 8192 },
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">L’accès en DM est contrôlé par canal via `dmPolicy` :

    ```
    Groupes : `channels.mattermost.groupPolicy="allowlist"` par défaut (mention-gated). Utilisez `channels.mattermost.groupAllowFrom` pour restreindre les expéditeurs.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Les messages de groupe sont par défaut **require mention** (soit la mention des métadonnées ou les schémas de regex ). `agents.defaults.subagents` configure les valeurs par défaut des sous-agents :

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">Isolation de la session de discussion :

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // recommandé pour les environnements multi‑utilisateurs
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope` : `main` (partagé) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Voir [Session Management](/concepts/session) pour le périmètre, les liens d’identité et la politique d’envoi.
    - Voir la [référence complète](/gateway/configuration-reference#session) pour tous les champs.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">Exécuter les sessions d’agent dans des conteneurs Docker isolés :

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

    ```
    - `every` : durée (`30m`, `2h`). Définir `0m` pour désactiver.
    - `target` : `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Voir [Heartbeat](/gateway/heartbeat) pour le guide complet.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    Voir [Tâches Cron](/automation/cron-jobs) pour la vue d'ensemble des fonctionnalités et les exemples de CLI.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Activer un point de terminaison simple du webhook HTTP sur le serveur HTTP de la passerelle.

    ````
    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```
    
    Voir la [référence complète](/gateway/configuration-reference#hooks) pour toutes les options de mapping et l’intégration Gmail.
    ````

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Exécutez plusieurs agents isolés (espace de travail séparé, `agentDir`, sessions) à l'intérieur d'une passerelle.

    ```
    `<agentDir>/auth-profiles.json` (par défaut : `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Scindez votre configuration en plusieurs fichiers à l’aide de la directive `$include`. Ceci est utile pour :

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Rechargement chaud de la configuration)

Le Gateway surveille `~/.openclaw/openclaw.json` et applique automatiquement les modifications — aucun redémarrage manuel n’est nécessaire pour la plupart des paramètres.

### Modes de rechargement

| Modes:                       | Comportement                                                                                                                                                                |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (par défaut) | Applique instantanément à chaud les modifications sûres. Redémarre automatiquement pour les modifications critiques.                        |
| **`hot`**                                    | Applique à chaud uniquement les modifications sûres. Enregistre un avertissement lorsqu’un redémarrage est nécessaire — à vous de le gérer. |
| **`restart`**                                | Redémarre le Gateway à chaque modification de configuration, sûre ou non.                                                                                   |
| **`off`**                                    | Désactive la surveillance des fichiers. Les modifications prennent effet lors du prochain redémarrage manuel.                               |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Ce qui est appliqué à chaud vs ce qui nécessite un redémarrage

La plupart des champs sont appliqués à chaud sans interruption. En mode `hybrid`, les modifications nécessitant un redémarrage sont gérées automatiquement.

| Catégorie                                                     | Champs :                                           | Redémarrage nécessaire ? |
| ------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------ |
| \\`channels.<channel>                        | `web` (Exécution du canal web WhatsApp)         | Non                      |
| Agent & modèles                           | `agent`, `agents`, `models`, `routing`                             | Non                      |
| Automatisation                                                | `hooks`, `cron`, `agent.heartbeat`                                 | Non                      |
| messages                                                      | `messages`                                                         | Non                      |
| Outils & médias                           | `tools`, `browser`, `skills`, `audio`, `talk`                      | Non                      |
| Interface & divers                        | `ui`, `logging`, `identity`, `bindings`                            | Non                      |
| `passerelle` (mode serveur Gateway + lier) | `gateway` (port/bind/auth/control UI/tailscale) | **Oui**                  |
| Infrastructure                                                | `discovery`, `canvasHost`, `plugins`                               | **Oui**                  |

<Note>
`gateway.reload` et `gateway.remote` sont des exceptions — leur modification **ne** déclenche **pas** de redémarrage.
</Note>

## Mises à jour partielles (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">Utilisez `config.apply` pour valider + écrire la configuration complète et redémarrer la Gateway en une seule étape.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">Utilisez `config.patch` pour fusionner une mise à jour partielle dans la configuration existante sans écraser
les clés non liées. Cela applique la sémantique de _JSON merge patch_ :

    ````
    - Les objets sont fusionnés récursivement
    - `null` supprime une clé
    - Les tableaux sont remplacés
    
    Params:
    
    - `raw` (string) — JSON5 contenant uniquement les clés à modifier
    - `baseHash` (obligatoire) — hash de configuration provenant de `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — identiques à `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Substitution de variables d’environnement dans la configuration

OpenClaw lit les variables d’environnement (env vars) du processus parent ainsi que :

- `.env` depuis le répertoire de travail courant (s’il existe)
- un repli global `.env` depuis `~/.openclaw/.env` (alias `$OPENCLAW_STATE_DIR/.env`)

Aucun fichier `.env` ne remplace des variables d’environnement existantes. Vous pouvez également définir des variables d’environnement inline dans la configuration :

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">Option de confort : si activée et qu’aucune des clés attendues n’est encore définie, OpenClaw exécute votre shell de connexion et importe uniquement les clés attendues manquantes (ne remplace jamais).

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Vous pouvez référencer des variables d’environnement directement dans toute valeur de chaîne de la configuration en utilisant la syntaxe `${VAR_NAME}`.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**Règles :**

- Seuls les noms de variables en majuscules sont reconnus : `[A-Z_][A-Z0-9_]*`
- Il manque ou vide env vars lancer une erreur au chargement de la configuration
- Échappez avec `$${VAR}` pour produire un `${VAR}` littéral
- Fonctionne avec `$include` (les fichiers inclus bénéficient aussi de la substitution)
- Substitution inline : `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Voir [/environment](/help/environment) pour la priorité complète et les sources.

## Référence complète

**Nouveau dans la configuration ?** Consultez le guide [Configuration Examples](/gateway/configuration-examples) pour des exemples complets avec des explications détaillées !

---

gateway/configuration.md
