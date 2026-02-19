---
summary: "Statut du support du bot Telegram, capacites et configuration"
read_when:
  - Travail sur les fonctionnalites Telegram ou les webhooks
title: "Telegram"
---

# Telegram (Bot API)

Statut : prêt pour la production pour les Messages prives des bots + les groupes via grammY. Long-polling par défaut ; webhook optionnel.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Default DM policy for Telegram is pairing.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Diagnostics inter-canaux et playbooks de réparation.
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">Modèles et exemples complets de configuration de canal.
</Card>
</CardGroup>

## Configuration (chemin rapide)

<Steps>
  <Step title="Create the bot token in BotFather">Ouvrez Telegram et discutez avec **@BotFather** ([lien direct](https://t.me/BotFather)). Confirmez que le handle est exactement `@BotFather`.

    ```
    `/newbot` cree le bot et renvoie le token (gardez‑le secret).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Option env : `TELEGRAM_BOT_TOKEN=...` (fonctionne pour le compte par defaut).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Les codes d’appairage expirent après 1 heure.
    ```

  
</Step>

  <Step title="Add the bot to a group">Pour les groupes : ajoutez le bot, decidez du comportement de confidentialite/admin (ci‑dessous), puis definissez `channels.telegram.groups` pour controler le filtrage par mention + les listes d’autorisation.
</Step>
</Steps>

<Note>
L’ordre de résolution des tokens tient compte du compte. En pratique, les valeurs de configuration priment sur les variables d’environnement de secours, et `TELEGRAM_BOT_TOKEN` s’applique uniquement au compte par défaut.
</Note>

## Paramètres côté Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">**Remarque :** Lors d’un changement du mode confidentialite, Telegram exige de retirer puis de rajouter le bot
dans chaque groupe pour que le changement prenne effet.

    ```
    `/setprivacy` — controler si le bot voit tous les messages de groupe.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Le statut admin se definit dans le groupe (UI Telegram).

    ```
    Ajouter le bot comme **admin** du groupe (les bots admins recoivent tous les messages).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — autoriser/interdire l’ajout du bot aux groupes.
    ```

  
</Accordion>
</AccordionGroup>

## Contrôle d’accès et activation

<Tabs>
  <Tab title="DM policy">`channels.telegram.dmPolicy` contrôle l’accès aux messages directs :

    ```
    `channels.telegram.allowFrom` accepte des IDs utilisateur numeriques (recommande) ou des entrees `@username`. Ce n’est **pas** le nom d’utilisateur du bot ; utilisez l’ID de l’expediteur humain. L’assistant accepte `@username` et le resout vers l’ID numerique lorsque possible.
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Envoyez un Message prive a `@userinfobot` ou `@getidsbot` et utilisez l’ID utilisateur renvoye.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Deux controles independants :

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Les réponses en groupe nécessitent une mention par défaut.
La mention peut provenir de :

- mention native `@botusername`, ou
- modèles de mention dans :
  - `agents.list[].groupChat.mentionPatterns`
  - `messages.groupChat.mentionPatterns`

Commandes de bascule au niveau de la session :

- `/activation always`
- `/activation mention`

Ces commandes mettent à jour uniquement l’état de la session. Utilisez la configuration pour la persistance.

Exemple de configuration persistante :

    ```
      
</Tab>
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Transmettez n’importe quel message du groupe a `@userinfobot` ou `@getidsbot` sur Telegram pour voir l’ID du chat (nombre negatif comme `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Telegram est géré par le processus gateway.

- Les sessions de groupe sont isolées par ID de groupe.
- Routage deterministe : les reponses repartent vers Telegram ; le modele ne choisit jamais les canaux.
- Les messages entrants sont normalises dans l’enveloppe de canal partagee avec contexte de reponse et emplacements media.
- Le long polling utilise le runner grammY avec un séquencement par chat/par fil. Ajoute `:topic:<threadId>` a la cle de session du groupe Telegram afin que chaque sujet soit isole.
- Les chats prives peuvent inclure `message_thread_id` dans certains cas limites. OpenClaw conserve la cle de session DM inchangée, mais utilise tout de meme l’id de fil pour les reponses/le streaming de brouillons lorsqu’il est present.
- Référence des fonctionnalités Le long‑polling utilise le runner grammY avec un sequencage par chat ; la concurrence globale est plafonnee par `agents.defaults.maxConcurrent`.
- L’API Bot Telegram ne prend pas en charge les accusés de lecture ; il n’existe pas d’option `sendReadReceipts`.

## Exigence :- `channels.telegram.streamMode` n’est pas "off" (par défaut : "partial")Modes :- `off` : aucun aperçu en direct
- `partial` : mises à jour fréquentes de l’aperçu à partir du texte partiel
- `block` : mises à jour d’aperçu par blocs à l’aide de `channels.telegram.draftChunk`Valeurs par défaut de `draftChunk` pour `streamMode: "block"` :- `minChars: 200`
- `maxChars: 800`
- `breakPreference: "paragraph"``maxChars` est limité par `channels.telegram.textChunkLimit`.Fonctionne dans les chats directs et les groupes/sujets.Pour les réponses texte uniquement, OpenClaw conserve le même message d’aperçu et effectue une modification finale sur place (pas de second message).Pour les réponses complexes (par exemple des médias), OpenClaw revient à un envoi final normal puis supprime le message d’aperçu.`streamMode` est distinct du streaming par blocs. Lorsque le streaming par blocs est explicitement activé pour Telegram, OpenClaw ignore le flux d’aperçu afin d’éviter un double streaming.Flux de raisonnement spécifique à Telegram :- `/reasoning stream` envoie le raisonnement dans l’aperçu en direct pendant la génération
- la réponse finale est envoyée sans le texte de raisonnement

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw peut diffuser des reponses partielles dans les Messages prives Telegram via `sendMessageDraft`.

    ```
    - Le texte de type Markdown est rendu en HTML compatible Telegram.
    - Le HTML brut du modèle est échappé afin de réduire les erreurs d’analyse Telegram.
    - Si Telegram rejette le HTML analysé, OpenClaw réessaie en texte brut.
    
    Les aperçus de liens sont activés par défaut et peuvent être désactivés avec `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Le texte Telegram sortant utilise `parse_mode: "HTML"` (sous‑ensemble de balises pris en charge par Telegram).

    ```
    Règles :
    
    - les noms sont normalisés (suppression du `/` initial, en minuscules)
    - motif valide : `a-z`, `0-9`, `_`, longueur `1..32`
    - les commandes personnalisées ne peuvent pas remplacer les commandes natives
    - les conflits/doublons sont ignorés et consignés dans les logs
    
    Remarques :
    
    - les commandes personnalisées sont uniquement des entrées de menu ; elles n’implémentent pas automatiquement un comportement
    - les commandes de plugin/skill peuvent toujours fonctionner lorsqu’elles sont saisies, même si elles ne sont pas affichées dans le menu Telegram
    
    Si les commandes natives sont désactivées, les commandes intégrées sont supprimées. Les commandes personnalisées/de plugin peuvent toujours s’enregistrer si configurées.
    
    Échec de configuration courant :
    
    - `setMyCommands failed` signifie généralement que le DNS/HTTPS sortant vers `api.telegram.org` est bloqué.
    
    ### Commandes d’appairage d’appareil (`device-pair` plugin)
    
    Lorsque le plugin `device-pair` est installé :
    
    1. `/pair` génère un code de configuration
    2. collez le code dans l’application iOS
    3. `/pair approve` approuve la dernière demande en attente
    
    Plus de détails : [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    OpenClaw enregistre des commandes natives (comme `/status`, `/reset`, `/model`) dans le menu de bot Telegram au demarrage. Vous pouvez ajouter des commandes personnalisees au menu via la config :
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Configurez la portée du clavier inline :
    ```

  
</Accordion>

  <Accordion title="Inline buttons">Surcharge par compte :

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Les actions d’outil Telegram incluent :
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = aucun message de groupe accepte
      La valeur par defaut est `groupPolicy: "allowlist"` (bloque tant que vous n’ajoutez pas `groupAllowFrom`).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Lorsqu’un utilisateur clique sur un bouton, les donnees de rappel sont renvoyees a l’agent sous forme de message au format :
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">- `sendMessage` (`to`, `content`, `mediaUrl` optionnel, `replyToMessageId`, `messageThreadId`)
- `react` (`chatId`, `messageId`, `emoji`)
- `deleteMessage` (`chatId`, `messageId`)
- `editMessage` (`chatId`, `messageId`, `content`)

Les actions de message du canal exposent des alias ergonomiques (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).

Contrôles d’activation :

- `channels.telegram.actions.sendMessage`
- `channels.telegram.actions.editMessage`
- `channels.telegram.actions.deleteMessage`
- `channels.telegram.actions.reactions`
- `channels.telegram.actions.sticker` (par défaut : désactivé)

Sémantique de suppression de réaction : [/tools/reactions](/tools/reactions)

    ```
    - `[[reply_to_current]]` répond au message déclencheur
    - `[[reply_to:<id>]]` répond à un ID de message Telegram spécifique
    
    `channels.telegram.replyToMode` contrôle le comportement :
    
    - `off` (par défaut)
    - `first`
    - `all`
    
    Remarque : `off` désactive le fil de réponse implicite. Les balises explicites `[[reply_to_*]]` sont toujours respectées.
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram prend en charge les reponses filaires optionnelles via des balises :

    ```
    ### Messages audio
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Sujets (supergroupes forum)

    ```
    Fils de discussion en chat prive uniquement (Telegram inclut `message_thread_id` dans les messages entrants).
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">Les notes vidéo ne prennent pas en charge les légendes ; le texte du message fourni est envoyé séparément.

### Stickers

Gestion des stickers entrants :

- WEBP statique : téléchargé et traité (placeholder `<media:sticker>`)
- TGS animé : ignoré
- WEBM vidéo : ignoré

Champs de contexte du sticker :

- `Sticker.emoji`
- `Sticker.setName`
- `Sticker.fileId`
- `Sticker.fileUniqueId`
- `Sticker.cachedDescription`

Fichier de cache des stickers :

- `~/.openclaw/telegram/sticker-cache.json`

Les stickers sont décrits une seule fois (lorsque possible) et mis en cache afin de réduire les appels répétés à la vision.

Activer les actions de sticker :

    ```
    `[[audio_as_voice]]` — envoyer l’audio comme note vocale au lieu d’un fichier.
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Lorsqu’activé, OpenClaw met en file d’attente des événements système tels que :
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Configuration :
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (par défaut : `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (par défaut : `minimal`)
    
    Remarques :
    
    - `own` signifie les réactions des utilisateurs aux messages envoyés par le bot uniquement (meilleur effort via le cache des messages envoyés).
    - Telegram ne fournit pas d’ID de fil dans les mises à jour de réaction.
      - les groupes non forum sont routés vers la session de chat de groupe
      - les groupes forum sont routés vers la session du sujet général du groupe (`:topic:1`), et non vers le sujet d’origine exact
    
    `allowed_updates` pour le polling/webhook inclut automatiquement `message_reaction`.
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Envoi d’autocollants
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Cache des autocollants
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Recoit la mise a jour `message_reaction` depuis l’API Telegram

    ```
    Ordre de résolution :
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - emoji d’identité de l’agent en secours (`agents.list[].identity.emoji`, sinon "👀")
    
    Remarques :
    
    - Telegram attend des emoji unicode (par exemple "👀").
    - Utilisez `""` pour désactiver la réaction pour un canal ou un compte.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">**Fonctionnement des reactions :**
Les reactions Telegram arrivent sous forme d’**evenements `message_reaction` distincts**, et non comme des proprietes dans les charges de message. Lorsqu’un utilisateur ajoute une reaction, OpenClaw :

    ```
    Les écritures de configuration du canal sont activées par défaut (`configWrites !== false`).
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">Par défaut : long polling.

    ```
    Par defaut, Telegram est autorise a ecrire des mises a jour de configuration declenchees par des evenements de canal ou `/config set|unset`.
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">- Contrôles de l’historique des DM :
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["

    ```
    Si votre URL publique est differente, utilisez un proxy inverse et pointez `channels.telegram.webhookUrl` vers le point de terminaison public.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Le texte sortant est segmente jusqu’a `channels.telegram.textChunkLimit` (par defaut 4000).
    Segmentation optionnelle par sauts de ligne : definissez `channels.telegram.chunkMode="newline"` pour scinder sur les lignes vides (frontieres de paragraphes) avant la segmentation par longueur.
    Les telechargements/envois de medias sont plafonnes par `channels.telegram.mediaMaxMb` (par defaut 5).
    Les requetes API Bot Telegram expirent apres `channels.telegram.timeoutSeconds` (par defaut 500 via grammY).
    Le contexte d’historique de groupe utilise `channels.telegram.historyLimit` (ou `channels.telegram.accounts.*.historyLimit`), avec repli sur `messages.groupChat.historyLimit`. Definissez `0` pour desactiver (par defaut 50).
    "].historyLimit`
    - les tentatives de nouvelle requête vers l’API Telegram sortante sont configurables via `channels.telegram.retry`.<user_id>La cible d’envoi CLI peut être un ID de chat numérique ou un nom d’utilisateur :

    ```
      
</Accordion>
    ```

```bash
Exemple : `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Problemes courants

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Si vous avez defini `channels.telegram.groups.*.requireMention=false`, le **mode confidentialite** de l’API Bot Telegram doit etre desactive.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - Node 22+ + fetch/proxy personnalisé peut déclencher un arrêt immédiat si les types AbortSignal ne correspondent pas.
    - Certains hébergeurs résolvent `api.telegram.org` en IPv6 en premier ; une sortie IPv6 défaillante peut provoquer des échecs intermittents de l’API Telegram.
    - Vérifiez les réponses DNS :
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    `setMyCommands failed` dans les journaux signifie generalement que la sortie HTTPS/DNS est bloquee vers `api.telegram.org`.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    dig +short api.telegram.org A
    dig +short api.telegram.org AAAA
    ```

```bash
  
</Accordion>
```

  
</Accordion>
</AccordionGroup>

Plus d’aide : [Depannage des canaux](/channels/troubleshooting).

## Reference de configuration (Telegram)

Référence principale :

- `channels.telegram.enabled` : activer/desactiver le demarrage du canal.

- `channels.telegram.botToken` : token du bot (BotFather).

- `channels.telegram.tokenFile` : lire le token depuis un chemin de fichier.

- `channels.telegram.dmPolicy` : `pairing | allowlist | open | disabled` (par defaut : appairage).

- `channels.telegram.allowFrom` : liste d’autorisation DM (ids/noms d’utilisateur). `open` requiert `"*"`. `openclaw doctor --fix` peut résoudre les entrées héritées `@username` en IDs.

- `channels.telegram.groupPolicy` : `open | allowlist | disabled` (par defaut : liste d’autorisation).

- `channels.telegram.groupAllowFrom` : liste d’autorisation des expéditeurs de groupe (ids/noms d’utilisateur). `openclaw doctor --fix` peut résoudre les entrées héritées `@username` en IDs.

- `channels.telegram.groups` : valeurs par defaut par groupe + liste d’autorisation (utilisez `"*"` pour les valeurs globales).
  - `channels.telegram.groups.<id>.groupPolicy` : surcharge par groupe pour groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention` : filtrage par mention par defaut.
  - `channels.telegram.groups.<id>.skills` : filtre de skills (omis = tous les skills, vide = aucun).
  - `channels.telegram.groups.<id>.allowFrom` : surcharge de liste d’autorisation des expéditeurs par groupe.
  - `channels.telegram.groups.<id>.systemPrompt` : invite systeme supplementaire pour le groupe.
  - `channels.telegram.groups.<id>.enabled` : desactiver le groupe lorsque `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*` : surcharges par sujet (memes champs que le groupe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy` : surcharge par sujet pour groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention` : surcharge de filtrage par mention par sujet.

- `channels.telegram.capabilities.inlineButtons` : `off | dm | group | all | allowlist` (par defaut : liste d’autorisation).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons` : surcharge par compte.

- `channels.telegram.replyToMode` : `off | first | all` (par defaut : `first`).

- `channels.telegram.textChunkLimit` : taille de segmentation sortante (caracteres).

- `channels.telegram.chunkMode` : `length` (par defaut) ou `newline` pour scinder sur les lignes vides (frontieres de paragraphes) avant la segmentation par longueur.

- `channels.telegram.linkPreview` : activer/desactiver les apercus de lien pour les messages sortants (par defaut : true).

- `channels.telegram.streamMode` : `off | partial | block` (streaming de brouillons).

- `channels.telegram.mediaMaxMb` : plafond media entrant/sortant (Mo).

- `channels.telegram.retry` : politique de reessai pour les appels API Telegram sortants (tentatives, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily` : surcharge de Node autoSelectFamily (true=activer, false=desactiver). Desactive par defaut sur Node 22 pour eviter les delais Happy Eyeballs.

- `channels.telegram.proxy` : URL de proxy pour les appels API Bot (SOCKS/HTTP).

- `channels.telegram.webhookUrl` : activer le mode webhook (necessite `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret` : secret de webhook (requis lorsque webhookUrl est defini).

- `channels.telegram.webhookPath` : chemin local du webhook (par defaut `/telegram-webhook`).

- L’ecouteur local se lie a `0.0.0.0:8787` et sert `POST /telegram-webhook` par defaut.

- `channels.telegram.actions.reactions` : filtrer les reactions de l’outil Telegram.

- `channels.telegram.actions.sendMessage` : filtrer les envois de messages de l’outil Telegram.

- `channels.telegram.actions.deleteMessage` : filtrer les suppressions de messages de l’outil Telegram.

- `channels.telegram.actions.sticker` : filtrer les actions d’autocollants Telegram — envoi et recherche (par defaut : false).

- `channels.telegram.reactionNotifications` : `off | own | all` — controler quelles reactions declenchent des evenements systeme (par defaut : `own` lorsqu’il n’est pas defini).

- `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` — controler la capacite de reaction de l’agent (par defaut : `minimal` lorsqu’il n’est pas defini).

- Configuration complete : [Configuration](/gateway/configuration)

Champs à fort signal spécifiques à Telegram :

- démarrage/auth : `enabled`, `botToken`, `tokenFile`, `accounts.*`
- Les commandes necessitent une autorisation meme dans les groupes avec `groupPolicy: "open"`
- commande/menu : `commands.native`, `customCommands`
- fil de discussion/réponses : `replyToMode`
- Optionnel (uniquement pour `streamMode: "block"`) :
- formatage/livraison : `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- médias/réseau : `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Mode webhook : definissez `channels.telegram.webhookUrl` et `channels.telegram.webhookSecret` (optionnellement `channels.telegram.webhookPath`).
- actions/capacités : `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- Notifications de reactions
- écritures/historique : `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Connexe

- [Appairage](/channels/pairing)
- More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
- `channels.telegram.streamMode: "off" | "partial" | "block"` (par defaut : `partial`)
