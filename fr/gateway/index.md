---
summary: "Runbook pour le service Gateway, son cycle de vie et ses opérations"
read_when:
  - Lors de l’exécution ou du débogage du processus gateway
title: "Runbook du Gateway"
---

# Runbook du service Gateway

Utilisez cette page pour le démarrage jour 1 et les opérations jour 2 du service Gateway.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Diagnostics orientés symptômes avec séquences de commandes exactes et signatures de logs.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Guide d’installation orienté tâches + référence complète de configuration.
  
</Card>
</CardGroup>

## Démarrage local en 5 minutes

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Référence saine : `Runtime: running` et `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Le rechargement de la configuration Gateway surveille le chemin actif du fichier de configuration (résolu à partir des valeurs par défaut du profil/état, ou `OPENCLAW_CONFIG_PATH` lorsqu’il est défini).
Mode par défaut : `gateway.reload.mode="hybrid"` (application à chaud des changements sûrs, redémarrage pour les changements critiques).
</Note>

## Modèle d’exécution

- Un processus toujours actif pour le routage, le plan de contrôle et les connexions aux canaux.
- Multiplexage sur un port unique.
  - Contrôle/RPC WebSocket
  - OpenResponses (HTTP) : [`/v1/responses`](/gateway/openresponses-http-api).
  - Interface de contrôle et hooks
- Mode de liaison par défaut : `loopback`.
- L’authentification Gateway est requise par défaut : définissez `gateway.auth.token` (ou `OPENCLAW_GATEWAY_TOKEN`) ou `gateway.auth.password`.

### Priorité du port et du mode de liaison

| Paramètre       | Ordre de résolution                                                                                                                   |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Port du Gateway | Priorité des ports : `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > valeur par défaut `18789`. |
| Mode de liaison | CLI/surcharge → `gateway.bind` → `loopback`                                                                                           |

### Modes de rechargement à chaud

| Désactiver avec `gateway.reload.mode="off"`. | Comportement keepalive                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------------ |
| `off`                                                        | Aucun rechargement de la configuration                             |
| `hot`                                                        | Appliquer uniquement les modifications sûres à chaud               |
| `restart`                                                    | Redémarrer si des modifications nécessitent un rechargement        |
| `hybrid` (par défaut)                     | Appliquer à chaud lorsque c’est sûr, redémarrer lorsque nécessaire |

## Ensemble de commandes opérateur

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Accès distant

Tailscale/VPN recommandé ; sinon tunnel SSH :
Solution de secours : tunnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Les clients se connectent ensuite à `ws://127.0.0.1:18789` via le tunnel.

<Warning>
Si un jeton est configuré, les clients doivent l’inclure dans `connect.params.auth.token` même via le tunnel.
</Warning>

Voir : [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Supervision et cycle de vie du service

Utilisez des exécutions supervisées pour une fiabilité proche de la production.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Pour redémarrer, utilisez `openclaw gateway restart` (ou `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
```

Les labels LaunchAgent sont `ai.openclaw.gateway` (par défaut) ou `ai.openclaw.<profile>` lors de l’exécution d’un profil nommé. `openclaw doctor` audite et corrige les dérives de configuration du service.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Activez le lingering (requis pour que le service utilisateur survive à la déconnexion/inactivité) :

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Utilisez une unité système pour les hôtes multi-utilisateurs ou toujours actifs.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Gateways multiples (même hôte)

Généralement inutile : un Gateway peut servir plusieurs canaux de messagerie et agents.
Utilisez plusieurs Gateways uniquement pour la redondance ou une isolation stricte (ex. bot de secours).

Checklist par instance :

- `gateway.port` unique
- `OPENCLAW_CONFIG_PATH` unique
- `OPENCLAW_STATE_DIR` unique
- `agents.defaults.workspace` unique

Exemple :

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Guide complet : [Multiple gateways](/gateway/multiple-gateways).

### Profil dev (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Les paramètres par défaut incluent un état/configuration isolé et le port de base du gateway `19001`.

## Protocole (vue opérateur)

- La première trame client doit être `connect`.
- Le Gateway renvoie un instantané `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, limites/policy).
- Requêtes : `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Événements courants : `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Les exécutions d’agent se déroulent en deux étapes :

1. Accusé de réception immédiat accepté (`status:"accepted"`)
2. Les réponses `agent` sont en deux étapes : d’abord un accusé `res` `{runId,status:"accepted"}`, puis un `res` final `{runId,status:"ok"|"error",summary}` après la fin de l’exécution ; la sortie streamée arrive sous forme de `event:"agent"`.

Documentation complète : [Gateway protocol](/gateway/protocol) et [Bridge protocol (legacy)](/gateway/bridge-protocol).

## Vérifications opérationnelles

### KeepAlive : true

- Ouvrez la connexion WS et envoyez `connect`.
- Attendez la réponse `hello-ok` avec l’instantané.

### Disponibilité

```bash
`openclaw gateway health|status` — demander l’état/la santé via le WS du Gateway.
```

### Récupération des écarts

Les événements ne sont pas rejoués. En cas d’écarts de séquence, actualisez l’état (`health`, `system-presence`) avant de continuer.

## Signatures d’échec courantes

| Signature                                                      | Problème probable                                                |
| -------------------------------------------------------------- | ---------------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Liaison non-loopback sans token/mot de passe                     |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflit de port                                                  |
| `Gateway start blocked: set gateway.mode=local`                | Configuration définie en mode distant                            |
| Instantané de connexion                                        | Incompatibilité d’authentification entre le client et le Gateway |

Pour un diagnostic complet étape par étape, consultez [Gateway Troubleshooting](/gateway/troubleshooting).

## Garanties de sécurité

- Les clients du protocole Gateway échouent immédiatement lorsque le Gateway est indisponible (aucun fallback implicite vers un canal direct).
- Les premières trames non conformes ou le JSON malformé sont rejetés et la socket est fermée.
- Arrêt gracieux : émettre l’événement `shutdown` avant la fermeture ; les clients doivent gérer la fermeture + reconnexion.

---

Liens connexes :

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
