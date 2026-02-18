---
title: "Configuration"
---

# Configuration 🔧

OpenClaw lit une configuration optionnelle <Tooltip tip="JSON5 prend en charge les commentaires et les virgules finales">**JSON5**</Tooltip> depuis `~/.openclaw/openclaw.json`.

Si le fichier est absent, OpenClaw utilise des valeurs par défaut sûres. Raisons courantes d’ajouter une configuration :

- Restreindre qui peut envoyer des messages au bot
- Définir les modèles, outils, sandboxing ou automatisations (cron, hooks)
- Ajuster les sessions, médias, réseau ou UI

Voir la [référence complète](/gateway/configuration-reference) pour tous les champs disponibles.

<Tip>
**Nouveau dans la configuration ?** Commencez avec `openclaw onboard` pour une configuration interactive, ou consultez le guide [Configuration Examples](/gateway/configuration-examples) pour des exemples complets prêts à copier-coller.
</Tip>

## Configuration minimale

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Modification de la configuration

<Tabs>
  <Tab title="Assistant interactif">
    ```bash
    openclaw onboard       # assistant de configuration complet
    openclaw configure     # assistant de configuration
    ```
  </Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Interface de contrôle">
    Ouvrez [http://127.0.0.1:18789](http://127.0.0.1:18789) et utilisez l’onglet **Config**.  
    L’interface génère un formulaire à partir du schéma de configuration, avec un éditeur **Raw JSON** comme échappatoire.
  </Tab>
  <Tab title="Édition directe">
    Modifiez directement `~/.openclaw/openclaw.json`. La Gateway surveille le fichier et applique automatiquement les changements (voir [rechargement à chaud](#config-hot-reload)).
  </Tab>
</Tabs>

## Validation stricte

<Warning>
OpenClaw n’accepte que les configurations correspondant entièrement au schéma. Les clés inconnues, types invalides ou valeurs incorrectes entraînent un **refus de démarrage** de la Gateway. La seule exception à la racine est `$schema` (string), pour permettre aux éditeurs d’attacher des métadonnées JSON Schema.
</Warning>

En cas d’échec de validation :

- La Gateway ne démarre pas
- Seules les commandes de diagnostic fonctionnent (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Exécutez `openclaw doctor` pour voir les erreurs exactes
- Exécutez `openclaw doctor --fix` (ou `--yes`) pour appliquer les réparations

## Tâches courantes

<AccordionGroup>
  <Accordion title="Configurer un canal (WhatsApp, Telegram, Discord, etc.)">
    Chaque canal possède sa section sous `channels.<provider>`. Consultez la page dédiée du canal pour les étapes détaillées :

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    Tous les canaux partagent le même modèle de politique DM :

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // uniquement pour allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="Choisir et configurer les modèles">
    Définissez le modèle principal et des replis optionnels :

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` définit le catalogue de modèles et sert de liste d’autorisation pour `/model`.
    - Les références de modèles utilisent le format `provider/model` (ex. `anthropic/claude-opus-4-6`).
    - Voir [Models CLI](/concepts/models) pour changer de modèle en chat et [Model Failover](/concepts/model-failover) pour la rotation d’authentification et les replis.
    - Pour les fournisseurs personnalisés/self-hosted, voir [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls).

  </Accordion>

  <Accordion title="Contrôler qui peut envoyer des messages au bot">
    L’accès DM est contrôlé par canal via `dmPolicy` :

    - `"pairing"` (par défaut) : les expéditeurs inconnus reçoivent un code d’appairage unique
    - `"allowlist"` : seuls les expéditeurs dans `allowFrom`
    - `"open"` : autorise tous les DMs entrants (nécessite `allowFrom: ["*"]`)
    - `"disabled"` : ignore tous les DMs

    Pour les groupes, utilisez `groupPolicy` + `groupAllowFrom`.

    Voir la [référence complète](/gateway/configuration-reference#dm-and-group-access) pour les détails par canal.

  </Accordion>

  <Accordion title="Configurer le filtrage par mention en groupe">
    Les messages de groupe nécessitent par défaut une **mention**. Configurez les motifs par agent :

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **Mentions natives** : @-mentions propres à la plateforme
    - **Motifs texte** : regex définies dans `mentionPatterns`
    - Voir la [référence complète](/gateway/configuration-reference#group-chat-mention-gating).

  </Accordion>

  <Accordion title="Configurer les sessions et réinitialisations">
    Les sessions contrôlent la continuité des conversations :

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Voir [Session Management](/concepts/session) et la [référence complète](/gateway/configuration-reference#session).

  </Accordion>

  <Accordion title="Activer le sandboxing">
    Exécutez les sessions agent dans des conteneurs Docker isolés :

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",
            scope: "agent",
          },
        },
      },
    }
    ```

    Construisez l’image une fois : `scripts/sandbox-setup.sh`

    Voir [Sandboxing](/gateway/sandboxing).

  </Accordion>

  <Accordion title="Configurer le heartbeat (vérifications périodiques)">
    ```json5
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

    - `every` : durée (`30m`, `2h`). `0m` désactive.
    - `target` : `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Voir [Heartbeat](/gateway/heartbeat).

  </Accordion>

  <Accordion title="Configurer les tâches cron">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    Voir [Cron jobs](/automation/cron-jobs).

  </Accordion>

  <Accordion title="Configurer les webhooks (hooks)">
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

    Voir la [référence complète](/gateway/configuration-reference#hooks).

  </Accordion>

  <Accordion title="Configurer le routage multi-agents">
    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    Voir [Multi-Agent](/concepts/multi-agent).

  </Accordion>

  <Accordion title="Diviser la configuration en plusieurs fichiers ($include)">
    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **Fichier unique** : remplace l’objet
    - **Tableau de fichiers** : fusion profonde dans l’ordre
    - **Clés sœurs** : fusionnées après les inclusions
    - **Inclusions imbriquées** : jusqu’à 10 niveaux
    - **Chemins relatifs** : résolus depuis le fichier incluant
    - **Erreurs** : messages clairs pour fichiers manquants, erreurs de parsing ou boucles

  </Accordion>
</AccordionGroup>

## Rechargement à chaud de la configuration

La Gateway surveille `~/.openclaw/openclaw.json` et applique automatiquement les changements.

### Modes de rechargement

| Mode                   | Comportement                                                                 |
| ---------------------- | ---------------------------------------------------------------------------- |
| **`hybrid`** (défaut)  | Applique à chaud les changements sûrs. Redémarre automatiquement si nécessaire. |
| **`hot`**              | Applique uniquement les changements sûrs. Avertit si redémarrage requis.   |
| **`restart`**          | Redémarre la Gateway pour tout changement.                                  |
| **`off`**              | Désactive la surveillance.                                                   |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

## Variables d’environnement

OpenClaw lit les variables d’environnement du processus parent ainsi que :

- `.env` dans le répertoire courant
- `~/.openclaw/.env`

Aucun fichier `.env` ne remplace une variable déjà définie.

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Import depuis le shell (optionnel)">
  Si activé et que des clés attendues sont absentes, OpenClaw exécute votre shell de connexion et importe uniquement les clés manquantes :

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Équivalent variable d’environnement : `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="Substitution de variables d’environnement">
  Référencez des variables avec `${VAR_NAME}` :

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Règles :

- Noms en majuscules uniquement : `[A-Z_][A-Z0-9_]*`
- Variable manquante → erreur au chargement
- Échapper avec `$${VAR}`
- Fonctionne dans les fichiers `$include`
- Substitution inline : `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Voir [Environment](/help/environment).

## Référence complète

Pour la référence détaillée champ par champ, voir **[Configuration Reference](/gateway/configuration-reference)**.

---

_Related: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
