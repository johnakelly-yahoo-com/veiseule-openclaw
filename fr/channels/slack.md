---
summary: "Configuration Slack pour le mode socket ou webhook HTTP"
read_when:
  - Configurer Slack ou depanner le mode socket/HTTP de Slack
title: "Slack"
---

# Slack

Statut : prêt pour la production pour les DM + canaux via les intégrations d’application Slack. Mode HTTP (API Events)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Les DM Slack sont par défaut en mode d’association.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Comportement natif des commandes et catalogue des commandes.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostics inter-canaux et playbooks de réparation.
  
</Card>
</CardGroup>

## Configuration rapide

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        Dans les paramètres de l’application Slack :

        ```
        **OAuth & Permissions** → installez l’application et copiez le **Bot User OAuth Token** (`xoxb-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Secours via variables d’environnement (compte par défaut uniquement) :
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="S’abonner aux événements de l’application">
          Abonnez les événements du bot pour :
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Activez également l’onglet **Messages Tab** de l’App Home pour les messages privés.
        
</Step>
      
        <Step title="Démarrer gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        Utilisez le mode webhook HTTP lorsque votre Gateway (passerelle) est accessible par Slack via HTTPS (typique pour des deploiements serveur). Le mode HTTP utilise l’API Events + Interactivity + Slash Commands avec une URL de requete partagee.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      Mode HTTP multi-comptes : definissez `channels.slack.accounts.<id> .mode = "http"` et fournissez un
      `webhookPath` unique par compte afin que chaque application Slack pointe vers sa propre URL.

  
</Tab>
</Tabs>

## Modèle de jetons

- `botToken` + `appToken` sont requis pour le mode Socket.
- Le mode HTTP nécessite `botToken` + `signingSecret`.
- Les jetons configurés remplacent le secours via variables d’environnement.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` via variables d’environnement s’appliquent uniquement au compte par défaut.
- Definir `userTokenReadOnly: false` permet d’utiliser le token utilisateur pour les
  operations d’ecriture lorsqu’aucun token de bot n’est disponible, ce qui signifie que les actions s’executent avec l’acces de l’utilisateur installateur.
- Optionnel : ajoutez `chat:write.customize` si vous souhaitez que les messages sortants utilisent l’identité active de l’agent (avec `username` personnalisé et icône). `icon_emoji` utilise la syntaxe `:emoji_name:`.

<Tip>
Pour les actions/lectures d’annuaire, un user token peut être privilégié lorsqu’il est configuré. Meme avec `userTokenReadOnly: false`, le token du bot reste
prefere pour les ecritures lorsqu’il est disponible.
</Tip>

## Contrôle d’accès et routage

<Tabs>
  <Tab title="DM policy">DMs ignorés: l'expéditeur n'est pas approuvé lorsque `channels.slack.dm.policy="appairage"`.

    ```
    Pour autoriser tout le monde : definissez `channels.slack.dm.policy="open"` et `channels.slack.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` controle la gestion des canaux (`open|disabled|allowlist`).

    ```
    Pour n’autoriser **aucun canal**, definissez `channels.slack.groupPolicy: "disabled"` (ou conservez une liste d’autorisation vide).
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Les messages de canal sont filtrés par mention par défaut.

    ```
    Le controle des mentions est gere via `channels.slack.channels` (definissez `requireMention` sur `true`) ; `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`) comptent egalement comme mentions.
    ```

  
</Tab>
</Tabs>

## Commandes et comportement des slash commands

- Par defaut, le mode natif est desactive pour Slack, sauf si vous definissez `channels.slack.commands.native: true` (la valeur globale `commands.native` est `"auto"`, ce qui laisse Slack desactive).
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Lorsque les commandes natives sont activées, enregistrez les slash commands correspondantes dans Slack (noms `/<command>`).
- Si les commandes natives ne sont pas activées, vous pouvez exécuter une seule slash command configurée via `channels.slack.slashCommand`.

Paramètres par défaut des slash commands :

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Les sessions slash utilisent des clés isolées :

- Les commandes slash utilisent des sessions `agent:<agentId>:slack:slash:<userId>` (prefixe configurable via `channels.slack.slashCommand.sessionPrefix`).

et continuent de router l’exécution des commandes vers la session de conversation cible (`CommandTargetSessionKey`).

## Threads, sessions et balises de réponse

- Les messages privés sont routés comme `direct` ; les canaux comme `channel` ; les MPIM comme `group`.
- Avec la valeur par défaut `session.dmScope=main`, les messages privés Slack sont regroupés dans la session principale de l’agent.
- Les canaux correspondent a des sessions `agent:<agentId>:slack:channel:<channelId>`.
- Les réponses dans un thread peuvent créer des suffixes de session de thread (`:thread:<threadTs>`) lorsque applicable.
- `channels.slack.thread.historyScope` est défini par défaut sur `thread` ; `thread.inheritParent` est défini par défaut sur `false`.
- `channels.slack.thread.initialHistoryLimit` contrôle le nombre de messages existants du thread récupérés au démarrage d’une nouvelle session de thread (par défaut `20` ; définissez `0` pour désactiver).

Fil de reponse

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- Secours hérité pour les discussions directes : `channels.slack.dm.replyToMode`

Les balises de réponse manuelles sont prises en charge :

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Remarque : `replyToMode="off"` désactive le threading implicite des réponses. Les balises explicites `[[reply_to_*]]` sont toujours respectées.

## Médias, segmentation et distribution

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Les pièces jointes Slack sont téléchargées depuis des URLs privées hébergées par Slack (flux de requêtes authentifié par jeton) et enregistrées dans le stockage média lorsque la récupération réussit et que les limites de taille le permettent.

    ```
    Les televersements de medias sont limites par `channels.slack.mediaMaxMb` (par defaut 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">    - les segments de texte utilisent `channels.slack.textChunkLimit` (4000 par défaut)
    - `channels.slack.chunkMode="newline"` active le découpage prioritaire par paragraphes
    - les envois de fichiers utilisent les API d’upload Slack et peuvent inclure des réponses en fil (`thread_ts`)
    - la limite des médias sortants suit `channels.slack.mediaMaxMb` lorsqu’elle est configurée ; sinon, les envois du canal utilisent les valeurs par défaut par type MIME du pipeline média
</Accordion>

  <Accordion title="Delivery targets">    Cibles explicites recommandées :

    ```
    - `user:<id>` pour les messages privés (DM)
    - `channel:<id>` pour les canaux
    
    Les DM Slack sont ouverts via les API de conversation Slack lors de l’envoi vers des cibles utilisateur.
    ```

  
</Accordion>
</AccordionGroup>

## Actions et contrôles

Les actions d’outils Slack peuvent etre controlees via `channels.slack.actions.*` :

Groupes d’actions disponibles dans l’outillage Slack actuel :

| Groupe d’actions | Par défaut |
| ---------------- | ---------- |
| messages         | active     |
| reactions        | active     |
| pins             | active     |
| memberInfo       | active     |
| emojiList        | active     |

## Événements et comportement opérationnel

- Les modifications/suppressions de messages et les diffusions en fil sont mappées vers des événements système.
- Les événements d’ajout/suppression de réaction sont mappés vers des événements système.
- Les événements d’arrivée/départ de membres, de création/renommage de canal et d’ajout/suppression d’épinglage sont mappés vers des événements système.
- `channel_id_changed` peut migrer les clés de configuration du canal lorsque `configWrites` est activé.
- Les métadonnées de sujet/description du canal sont traitées comme un contexte non fiable et peuvent être injectées dans le contexte de routage.

## React + lister les reactions

`ackReaction` envoie un emoji d’accusé de réception pendant que OpenClaw traite un message entrant.

Ordre de résolution :

- `ou`channels.slack.channels.<name>.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- emoji d’identité de l’agent par défaut (`agents.list[].identity.emoji`, sinon "👀")

Notes

- Slack attend des shortcodes (par exemple `"eyes"`).
- Utilisez `""` pour désactiver la réaction pour un canal ou un compte.

## Checklist du manifeste et des permissions (scopes)

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">    Si vous configurez `channels.slack.userToken`, les permissions de lecture typiques sont :

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    https://docs.slack.dev/reference/methods/conversations.history
    ```

  
</Accordion>
</AccordionGroup>

## Problemes courants

<AccordionGroup>
  <Accordion title="No replies in channels">    Vérifiez, dans l’ordre :

    ```
    `users` : liste d’autorisation utilisateur optionnelle par canal.
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">    Vérifiez :

    ```
    `allowlist` exige que les canaux soient listes dans `channels.slack.channels`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Creez une application Slack et activez le **Mode socket**.
</Accordion>

  <Accordion title="HTTP mode not receiving events">    Validez :

    ```
    - signing secret
    - chemin du webhook
    - URLs de requête Slack (Events + Interactivity + Slash Commands)
    - `webhookPath` unique par compte HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">Vérifiez si vous aviez l’intention d’utiliser :

    ```
    - le mode de commande natif (`channels.slack.commands.native: true`) avec les slash commands correspondantes enregistrées dans Slack
    - ou le mode à commande slash unique (`channels.slack.slashCommand.enabled: true`)
    
    Vérifiez également `commands.useAccessGroups` et les listes d’autorisation des canaux/utilisateurs.
    ```

  
</Accordion>
</AccordionGroup>

## Références de configuration

Priorite :

- `users:read` (recherche d’utilisateurs)
  https://docs.slack.dev/reference/methods/users.info

  Champs Slack à fort impact :

  - mode/auth : `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - Accès DM : `dm.enabled`, `dmPolicy`, `allowFrom` (hérités : `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow` : autoriser/refuser le canal lorsque `groupPolicy="allowlist"`.
  - Repli sur `messages.groupChat.historyLimit`.
  - distribution : `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ops/fonctionnalités : `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Associé

- [Pairing](/channels/pairing)
- `channel` : canaux standards (publics/prives)
- [Dépannage](/channels/troubleshooting)
- [Configuration](/gateway/configuration)
- Liste complete des commandes + configuration : [Slash commands](/tools/slash-commands)
