---
summary: "Statut de prise en charge du bot Discord, capacites et configuration"
read_when:
  - Travail sur les fonctionnalites du canal Discord
title: "Discord"
---

# Discord (API Bot)

Statut : pret pour les Messages prives et les canaux texte de guildes via la passerelle officielle de bot Discord.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Les messages privés Discord utilisent par défaut le mode appairage.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Comportement des commandes natives et catalogue des commandes.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostics inter-canaux et flux de réparation.
  
</Card>
</CardGroup>

## Configuration rapide

<Steps>
  <Step title="Create a Discord bot and enable intents">Creer l’application Discord + l’utilisateur bot

    ```
    **Server Members Intent** (recommande ; requis pour certaines recherches de membres/utilisateurs et la correspondance des allowlists dans les guildes)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Fallback d’environnement pour le compte par défaut :
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Invitez le bot sur votre serveur avec les permissions requises pour lire/envoyer des messages la ou vous souhaitez l’utiliser.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
Les expéditeurs inconnus recoivent un code d’appariement (expire apres 1 heure) ; approuvez via `openclaw pairing approve discord <code>`.
```

    ```
    Les codes d’appairage expirent après 1 heure.
    ```

  
</Step>
</Steps>

<Note>
La résolution du token dépend du compte. Les valeurs de token définies dans la configuration priment sur le fallback d’environnement. `DISCORD_BOT_TOKEN` est utilisé uniquement pour le compte par défaut.
</Note>

## Modèle d’exécution

- Gateway gère la connexion Discord.
- Conserver un routage deterministe : les reponses reviennent toujours au canal d’origine.
- L’agent peut appeler `discord` avec des actions telles que :
- Les discussions directes se fondent dans la session principale de l’agent (par defaut `agent:main:main`) ; les canaux de guilde restent isoles comme `agent:<agentId>:discord:channel:<channelId>` (les noms d’affichage utilisent `discord:<guildSlug>#<channelSlug>`).
- Les Messages prives de groupe sont ignores par defaut ; activez-les via `channels.discord.dm.groupEnabled` et restreignez-les optionnellement avec `channels.discord.dm.groupChannels`.
- Les commandes natives utilisent des cles de session isolees (`agent:<agentId>:discord:slash:<userId>`) plutot que la session partagee `main`.

## Contrôle d’accès et routage

<Tabs>
  <Tab title="DM policy">Pour conserver l’ancien comportement « ouvert a tous » : definissez `channels.discord.dm.policy="open"` et `channels.discord.dm.allowFrom=["*"]`.

    ```
    Pour une allowlist stricte : definissez `channels.discord.dm.policy="allowlist"` et listez les expediteurs dans `channels.discord.dm.allowFrom`.
    ```

  
</Tab>

  <Tab title="Guild policy">Le comportement est controle par `channels.discord.replyToMode` :

    ```
    Votre allowlist de guilde/canal refuse le canal/l’utilisateur.
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Configurez OpenClaw avec `channels.discord.token` (ou `DISCORD_BOT_TOKEN` en repli).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">    
    Les messages de guilde sont, par défaut, filtrés par mention.

    ```
    Avertissement : si vous autorisez les reponses a d’autres bots (`channels.discord.allowBots=true`), evitez les boucles bot-a-bot avec des allowlists `requireMention`, `channels.discord.guilds.*.channels.<id> .users` et/ou des garde-fous clairs dans `AGENTS.md` et `SOUL.md`.
    ```

  
</Tab>
</Tabs>

### Routage d’agent basé sur les rôles

Utilisez `bindings[].match.roles` pour router les membres d’une guilde Discord vers différents agents selon l’ID de rôle. Les bindings basés sur les rôles acceptent uniquement des IDs de rôle et sont évalués après les bindings peer ou parent-peer et avant les bindings uniquement de guilde. Si un binding définit également d’autres champs de correspondance (par exemple `peer` + `guildId` + `roles`), tous les champs configurés doivent correspondre.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuration du Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">Dans **Bot** → **Privileged Gateway Intents**, activez :

    ```
    - Message Content Intent
    - Server Members Intent (recommandé)
    
    L’intent Presence est facultatif et requis uniquement si vous souhaitez recevoir des mises à jour de présence. Définir la présence du bot (`setPresence`) ne nécessite pas d’activer les mises à jour de présence des membres.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">Dans votre application : **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    Permissions de base typiques :
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (optionnel)
    
    Évitez `Administrator` sauf en cas de nécessité explicite.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">    
    Activez le mode développeur Discord, puis copiez :

    ```
    - ID du serveur
    - ID du canal
    - ID de l’utilisateur
    
    Privilégiez les IDs numériques dans la configuration OpenClaw pour des audits et des sondes fiables.
    ```

  
</Accordion>
</AccordionGroup>

## Commandes natives et autorisation des commandes

- Commandes natives optionnelles : `commands.native` vaut par defaut `"auto"` (active pour Discord/Telegram, desactive pour Slack).
- `channels.discord.execApprovals.enabled: true` dans votre config.
- Remplacez avec `channels.discord.commands.native: true|false|"auto"` ; `false` efface les commandes precedemment enregistrees.
- Les commandes natives respectent les memes allowlists que les Messages prives/messages de guilde (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, regles par canal).
- Les slash commands peuvent rester visibles dans l’UI Discord pour des utilisateurs non autorises ; OpenClaw applique les allowlists a l’execution et repond « not authorized ».

Voir [Slash commands](/tools/slash-commands) pour le catalogue des commandes et leur fonctionnement.

## Détails des fonctionnalités

<AccordionGroup>
  <Accordion title="Reply tags and native replies">    
    Discord prend en charge les balises de réponse dans la sortie de l’agent :

    ```
    `[[reply_to:<id>]]` — repondre a un id de message specifique depuis le contexte/historique. Les id de messages courants sont ajoutes aux prompts comme `[message_id: …]` ; les entrees d’historique incluent deja des id.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">    
    Contexte d’historique de guilde :

    ```
    - `channels.discord.historyLimit` par défaut `20`
    - fallback : `messages.groupChat.historyLimit`
    - `0` désactive
    
    Contrôles d’historique des DM :
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Comportement des threads :
    
    - Les threads Discord sont routés comme des sessions de canal
    - les métadonnées du thread parent peuvent être utilisées pour lier à la session parente
    - la configuration du thread hérite de celle du canal parent sauf si une entrée spécifique au thread existe
    
    Les sujets de canal sont injectés comme contexte **non fiable** (et non comme prompt système).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications` :

    ```
    `guilds.<id> .reactionNotifications` : mode d’evenement du systeme de reactions (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">    
    `ackReaction` envoie un emoji d’accusé de réception pendant que OpenClaw traite un message entrant.

    ```
    Ordre de résolution :
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - emoji d’identité de l’agent par défaut (`agents.list[].identity.emoji`, sinon "👀")
    
    Remarques :
    
    - Discord accepte les emojis unicode ou les noms d’emojis personnalisés.
    - Utilisez `""` pour désactiver la réaction pour un canal ou un compte.
    ```

  
</Accordion>

  <Accordion title="Config writes">    
    Les écritures de configuration initiées depuis le canal sont activées par défaut.

    ```
    Cela affecte les flux `/config set|unset` (lorsque les fonctionnalités de commande sont activées).
    
    Désactiver :
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">    
    Faites passer le trafic WebSocket du gateway Discord via un proxy HTTP(S) avec `channels.discord.proxy`.

```json5
Ou config : `channels.discord.token: "..."`.
```

    ```
    Surcharge par compte :
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">`pluralkit` : resoudre les messages proxifies par PluralKit afin que les membres du systeme apparaissent comme expediteurs distincts.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Remarques :
    
    - les listes d’autorisation peuvent utiliser `pk:<memberId>`
    - les noms d’affichage des membres sont mis en correspondance par nom/slug
    - les recherches utilisent l’ID de message d’origine et sont limitées à une fenêtre temporelle
    - si la recherche échoue, les messages relayés sont traités comme des messages de bot et ignorés sauf si `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">    
    Les mises à jour de présence sont appliquées uniquement lorsque vous définissez un champ de statut ou d’activité.

    ```
    Exemple avec statut uniquement :
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Exemple d’activité (le statut personnalisé est le type d’activité par défaut) :
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Exemple de streaming :
    ```

```json5
Lancez la Gateway (passerelle) ; elle demarre automatiquement le canal Discord lorsqu’un jeton est disponible (config en priorite, env en repli) et que `channels.discord.enabled` n’est pas `false`.
```

    ```
    Correspondance des types d’activité :
    
    - 0 : Playing
    - 1 : Streaming (nécessite `activityUrl`)
    - 2 : Listening
    - 3 : Watching
    - 4 : Custom (utilise le texte d’activité comme état du statut ; emoji facultatif)
    - 5 : Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    
    Discord prend en charge les approbations d’exécution via boutons dans les DM et peut éventuellement publier les demandes d’approbation dans le canal d’origine.

    ```
    Chemin de configuration :
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, par défaut : `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Lorsque `target` est défini sur `channel` ou `both`, la demande d’approbation est visible dans le canal. Seuls les approbateurs configurés peuvent utiliser les boutons ; les autres utilisateurs reçoivent un refus éphémère. Les demandes d’approbation incluent le texte de la commande ; activez donc l’envoi dans le canal uniquement pour des canaux de confiance. Si l’ID du canal ne peut pas être dérivé de la clé de session, OpenClaw revient à un envoi en DM.
    
    Si les approbations échouent avec des IDs d’approbation inconnus, vérifiez la liste des approbateurs et l’activation de la fonctionnalité.
    
    Documentation associée : [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Actions d’outils

Les actions de message Discord incluent la messagerie, l’administration des canaux, la modération, la présence et les actions liées aux métadonnées.

Exemples principaux :

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- React + lister reactions + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Les passerelles d’action se trouvent sous `channels.discord.actions.*`.

Action par défaut de l'outil

| Groupe d’actions                                                                                              | Par défaut |
| ------------------------------------------------------------------------------------------------------------- | ---------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | active     |
| rôles                                                                                                         | desactive  |
| modération                                                                                                    | desactive  |
| présence                                                                                                      | desactive  |

## Interface Components v2

OpenClaw utilise les composants Discord v2 pour les approbations d’exécution et les marqueurs inter-contexte. Les actions de message Discord peuvent également accepter `components` pour une interface personnalisée (avancé ; nécessite des instances de composant Carbon), tandis que les `embeds` hérités restent disponibles mais ne sont pas recommandés.

- `channels.discord.ui.components.accentColor` définit la couleur d’accent utilisée par les conteneurs de composants Discord (hexadécimal).
- À définir par compte avec `channels.discord.accounts.<id> .ui.components.accentColor`.
- Les `embeds` sont ignorés lorsque les composants v2 sont présents.

Exemple :

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## messages

Les messages vocaux Discord affichent un aperçu de la forme d’onde et nécessitent un audio OGG/Opus ainsi que des métadonnées. OpenClaw génère automatiquement la forme d’onde, mais nécessite que `ffmpeg` et `ffprobe` soient disponibles sur l’hôte gateway pour inspecter et convertir les fichiers audio.

Capacites et limites

- Fournissez un **chemin de fichier local** (les URL sont refusées).
- Omettez le contenu texte (Discord n’autorise pas texte + message vocal dans la même charge utile).
- Tous les formats audio sont acceptés ; OpenClaw convertit en OGG/Opus si nécessaire.

Exemple :

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Problemes courants

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - activer Message Content Intent
    - activer Server Members Intent lorsque vous dépendez de la résolution utilisateur/membre
    - redémarrer gateway après modification des intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy` : controle la gestion des canaux de guilde (`open|disabled|allowlist`) ; `allowlist` requiert des allowlists de canaux.
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Causes courantes :

    ```
    Pour n’autoriser **aucun canal**, definissez `channels.discord.groupPolicy: "disabled"` (ou conservez une allowlist vide).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Audits de permissions** (`channels status --probe`) ne verifient que les ID numeriques de canal.

    ```
    Si vous utilisez des clés slug, la correspondance à l’exécution peut toujours fonctionner, mais probe ne peut pas vérifier entièrement les autorisations.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **Les Messages prives ne fonctionnent pas** : `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, ou vous n’avez pas encore ete approuve (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Par défaut, les messages rédigés par des bots sont ignorés.

    ```
    Si vous définissez `channels.discord.allowBots=true`, utilisez des règles strictes de mention et d’allowlist pour éviter les comportements en boucle.
    ```

  
</Accordion>
</AccordionGroup>

## Points de référence de configuration

Référence principale :

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Champs Discord à fort impact :

- `presence` (statut/activite du bot, defaut `false`)
- `guilds.<id> .channels.<channel> .allow` : autoriser/refuser le canal lorsque `groupPolicy="allowlist"`.
- Utilisez `commands.useAccessGroups: false` pour contourner les verifications de groupes d’acces pour les commandes.
- `dmHistoryLimit` : limite d’historique des Messages prives en tours utilisateur. Surcharges par utilisateur : `dms["<user_id>"].historyLimit`.
- delivery : `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- média/retry : `mediaMaxMb`, `retry`
- actions : `actions.*`
- presence : `activity`, `status`, `activityType`, `activityUrl`
- UI : `ui.components.accentColor`
- fonctionnalités : `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Securite et exploitation

- Traitez les tokens de bot comme des secrets (`DISCORD_BOT_TOKEN` préféré dans les environnements supervisés).
- Discord bloque les « privileged intents » a moins de les activer explicitement.
- Si le déploiement/état des commandes est obsolète, redémarrez gateway et vérifiez à nouveau avec `openclaw channels status --probe`.

## Associé

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Dépannage](/channels/troubleshooting)
- Liste complete des commandes + config : [Slash commands](/tools/slash-commands)
