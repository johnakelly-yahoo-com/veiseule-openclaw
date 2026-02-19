---
title: IRC
description: Connectez OpenClaw aux canaux IRC et aux messages directs.
---

Utilisez IRC lorsque vous souhaitez utiliser OpenClaw dans des canaux classiques (`#room`) et en messages directs.
IRC est fourni sous forme de plugin d’extension, mais il se configure dans le fichier principal sous `channels.irc`.

## Démarrage rapide

1. Activez la configuration IRC dans `~/.openclaw/openclaw.json`.
2. Définissez au minimum :

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Démarrez/redémarrez la gateway :

```bash
openclaw gateway run
```

## Paramètres de sécurité par défaut

- `channels.irc.dmPolicy` est défini par défaut sur `"pairing"`.
- `channels.irc.groupPolicy` est défini par défaut sur `"allowlist"`.
- Avec `groupPolicy="allowlist"`, définissez `channels.irc.groups` pour spécifier les canaux autorisés.
- Utilisez TLS (`channels.irc.tls=true`) sauf si vous acceptez intentionnellement un transport en clair.

## Contrôle d’accès

Il existe deux « niveaux de contrôle » distincts pour les canaux IRC :

1. **Accès au canal** (`groupPolicy` + `groups`) : détermine si le bot accepte ou non les messages d’un canal.
2. **Accès de l’expéditeur** (`groupAllowFrom` / `groups["#channel"].allowFrom` par canal) : détermine qui est autorisé à déclencher le bot dans ce canal.

Clés de configuration :

- Liste blanche des DM (accès expéditeur en DM) : `channels.irc.allowFrom`
- Liste blanche des expéditeurs en groupe (accès expéditeur dans le canal) : `channels.irc.groupAllowFrom`
- Contrôles par canal (canal + expéditeur + règles de mention) : `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` autorise les canaux non configurés (**toujours soumis à l’obligation de mention par défaut**)

Les entrées de liste blanche peuvent utiliser le pseudo ou le format `nick!user@host`.

### Erreur fréquente : `allowFrom` s’applique aux DM, pas aux canaux

Si vous voyez des logs comme :

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…cela signifie que l’expéditeur n’était pas autorisé pour les messages de **groupe/canal**. Corrigez cela en :

- définissant `channels.irc.groupAllowFrom` (global pour tous les canaux), ou
- définissant des listes blanches d’expéditeurs par canal : `channels.irc.groups["#channel"].allowFrom`

Exemple (autoriser tout le monde dans `#tuirc-dev` à parler au bot) :

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Déclenchement des réponses (mentions)

Même si un canal est autorisé (via `groupPolicy` + `groups`) et que l’expéditeur est autorisé, OpenClaw applique par défaut une **obligation de mention** dans les contextes de groupe.

Cela signifie que vous pouvez voir des logs comme `drop channel … (missing-mention)` à moins que le message n’inclue un motif de mention correspondant au bot.

Pour que le bot réponde dans un canal IRC **sans nécessiter de mention**, désactivez l’obligation de mention pour ce canal :

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Ou pour autoriser **tous** les canaux IRC (sans liste blanche par canal) et répondre sans mentions :

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Note de sécurité (recommandé pour les canaux publics)

Si vous autorisez `allowFrom: ["*"]` dans un canal public, n’importe qui peut solliciter le bot.
Pour réduire les risques, limitez les outils pour ce canal.

### Mêmes outils pour tous dans le canal

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Outils différents selon l’expéditeur (le propriétaire a plus de permissions)

Utilisez `toolsBySender` pour appliquer une politique plus stricte à `"*"` et une plus souple à votre nick :

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Remarques :

- Les clés `toolsBySender` peuvent être un nick (par ex. `"eigen"`) ou un hostmask complet (`"eigen!~eigen@174.127.248.171"`) pour une correspondance d’identité plus fiable.
- La première politique d’expéditeur correspondante s’applique ; `"*"` est la valeur générique par défaut.

Pour en savoir plus sur l’accès par groupe vs le filtrage par mention (et leur interaction), voir : [/channels/groups](/channels/groups).

## NickServ

Pour vous identifier auprès de NickServ après la connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Enregistrement optionnel unique lors de la connexion :

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Désactivez `register` après l’enregistrement du nick afin d’éviter des tentatives REGISTER répétées.

## Variables d’environnement

Le compte par défaut prend en charge :

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (séparés par des virgules)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Dépannage

- Si le bot se connecte mais ne répond jamais dans les canaux, vérifiez `channels.irc.groups` **et** si le filtrage par mention bloque les messages (`missing-mention`). Si vous souhaitez qu’il réponde sans ping, définissez `requireMention:false` pour le canal.
- Si la connexion échoue, vérifiez la disponibilité du nick et le mot de passe du serveur.
- Si TLS échoue sur un réseau personnalisé, vérifiez l’hôte/le port et la configuration du certificat.

