---
title: "Telegram"
---

# Telegram (Bot API)

Statut : prêt pour la production pour les Messages prives des bots + les groupes via grammY. Long-polling par défaut ; webhook optionnel.

## Demarrage rapide (debutant)

1. Creez un bot avec **@BotFather** ([lien direct](https://t.me/BotFather)). Confirmez que le handle est exactement `@BotFather`, puis copiez le token.
2. Definissez le token :
   - Env : `TELEGRAM_BOT_TOKEN=...`
   - Ou config : `channels.telegram.botToken: "..."`.
   - Si les deux sont definis, la config est prioritaire (le repli env est uniquement pour le compte par defaut).
3. Demarrez la Gateway (passerelle).
4. L'accès DM est appairage par défaut; approuve le code d'appairage au premier contact.

Configuration minimale :

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## De quoi s’agit‑il ?

- Un canal Telegram Bot API detenu par la Gateway (passerelle).
- Routage deterministe : les reponses repartent vers Telegram ; le modele ne choisit jamais les canaux.
- Les Messages prives partagent la session principale de l’agent ; les groupes restent isoles (`agent:<agentId>:telegram:group:<chatId>`).

## Configuration (chemin rapide)

### 1. Creer un token de bot (BotFather)

1. Ouvrez Telegram et discutez avec **@BotFather** ([lien direct](https://t.me/BotFather)). Confirmez que le handle est exactement `@BotFather`.
2. Executez `/newbot`, puis suivez les invites (nom + nom d’utilisateur se terminant par `bot`).
3. Copiez le token et conservez‑le en lieu sûr.

Parametres BotFather optionnels :

- `/setjoingroups` — autoriser/interdire l’ajout du bot aux groupes.
- `/setprivacy` — controler si le bot voit tous les messages de groupe.

### 2. Configurer le token (env ou config)

Exemple :

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

Option env : `TELEGRAM_BOT_TOKEN=...` (fonctionne pour le compte par defaut).
Si env et config sont definis, la config est prioritaire.

Support multi‑comptes : utilisez `channels.telegram.accounts` avec des tokens par compte et `name` optionnel. Voir [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modele partage.

3. Demarrez la Gateway (passerelle). Telegram demarre lorsqu’un token est resolu (config en premier, repli env).
4. L’acces en Message prive est par defaut en appairage. Approuvez le code au premier contact avec le bot.
5. Pour les groupes : ajoutez le bot, decidez du comportement de confidentialite/admin (ci‑dessous), puis definissez `channels.telegram.groups` pour controler le filtrage par mention + les listes d’autorisation.

## Token + confidentialite + permissions (cote Telegram)

### Creation du token (BotFather)

- `/newbot` cree le bot et renvoie le token (gardez‑le secret).
- En cas de fuite du token, revoquez/regenez‑le via @BotFather et mettez a jour votre configuration.

### Visibilite des messages de groupe (Mode Confidentialite)

Les bots Telegram sont par defaut en **Mode Confidentialite**, ce qui limite les messages de groupe qu’ils recoivent.
Si votre bot doit voir _tous_ les messages de groupe, deux options :

- Desactiver le mode confidentialite avec `/setprivacy` **ou**
- Ajouter le bot comme **admin** du groupe (les bots admins recoivent tous les messages).

**Remarque :** Lors d’un changement du mode confidentialite, Telegram exige de retirer puis de rajouter le bot
dans chaque groupe pour que le changement prenne effet.

### Permissions de groupe (droits admin)

Le statut admin se definit dans le groupe (UI Telegram). Les bots admins recoivent toujours tous
les messages de groupe ; utilisez admin si vous avez besoin d’une visibilite complete.

## Comment ça marche (comportement)

- Les messages entrants sont normalises dans l’enveloppe de canal partagee avec contexte de reponse et emplacements media.
- Les reponses en groupe exigent une mention par defaut (mention @ native ou `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).
- Surcharge multi‑agents : definissez des motifs par agent sur `agents.list[].groupChat.mentionPatterns`.
- Les reponses sont toujours routees vers le meme chat Telegram.
- Le long‑polling utilise le runner grammY avec un sequencage par chat ; la concurrence globale est plafonnee par `agents.defaults.maxConcurrent`.
- L’API Bot Telegram ne prend pas en charge les accusés de lecture ; il n’existe pas d’option `sendReadReceipts`.

## Streaming de brouillons

OpenClaw peut diffuser des reponses partielles dans les Messages prives Telegram via `sendMessageDraft`.

Exigences :

- Mode fil de discussion active pour le bot dans @BotFather (mode sujet de forum).
- Fils de discussion en chat prive uniquement (Telegram inclut `message_thread_id` dans les messages entrants).
- `channels.telegram.streamMode` non defini sur `"off"` (par defaut : `"partial"`, `"block"` active les mises a jour de brouillon par blocs).

Le streaming de brouillons est reserve aux Messages prives ; Telegram ne le prend pas en charge dans les groupes ou les canaux.

## Mise en forme (HTML Telegram)

- Le texte Telegram sortant utilise `parse_mode: "HTML"` (sous‑ensemble de balises pris en charge par Telegram).
- Les entrees de type Markdown sont rendues en **HTML compatible Telegram** (gras/italique/barre/code/liens) ; les blocs sont aplatis en texte avec retours a la ligne/puces.
- Le HTML brut provenant des modeles est echappe pour eviter les erreurs d’analyse Telegram.
- Si Telegram rejette la charge HTML, OpenClaw renvoie le meme message en texte brut.

## Commandes (natives + personnalisees)

OpenClaw enregistre des commandes natives (comme `/status`, `/reset`, `/model`) dans le menu de bot Telegram au demarrage.
Vous pouvez ajouter des commandes personnalisees au menu via la config :

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

## Configuration des dépannages (commandes)

- `setMyCommands failed` dans les journaux signifie generalement que la sortie HTTPS/DNS est bloquee vers `api.telegram.org`.
- Si vous voyez des echecs `sendMessage` ou `sendChatAction`, verifiez le routage IPv6 et le DNS.

Plus d’aide : [Depannage des canaux](/channels/troubleshooting).

Notes :

- Les commandes personnalisees sont **uniquement des entrees de menu** ; OpenClaw ne les implemente pas sauf si vous les gerez ailleurs.
- Some commands can be handled by plugins/skills without being registered in Telegram’s command menu. These still work when typed (they just won't show up in `/commands` / the menu).
- Les noms de commande sont normalises (le `/` initial est supprime, mis en minuscules) et doivent correspondre a `a-z`, `0-9`, `_` (1–32 caracteres).
- Les commandes personnalisees **ne peuvent pas remplacer les commandes natives**. Les conflits sont ignores et journalises.
- Si `commands.native` est desactive, seules les commandes personnalisees sont enregistrees (ou effacees s’il n’y en a aucune).

### Device pairing commands (`device-pair` plugin)

If the `device-pair` plugin is installed, it adds a Telegram-first flow for pairing a new phone:

1. `/pair` generates a setup code (sent as a separate message for easy copy/paste).
2. Paste the setup code in the iOS app to connect.
3. `/pair approve` approves the latest pending device request.

More details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).

## Limites

- Le texte sortant est segmente jusqu’a `channels.telegram.textChunkLimit` (par defaut 4000).
- Segmentation optionnelle par sauts de ligne : definissez `channels.telegram.chunkMode="newline"` pour scinder sur les lignes vides (frontieres de paragraphes) avant la segmentation par longueur.
- Les telechargements/envois de medias sont plafonnes par `channels.telegram.mediaMaxMb` (par defaut 5).
- Les requetes API Bot Telegram expirent apres `channels.telegram.timeoutSeconds` (par defaut 500 via grammY). Reduisez pour eviter de longs blocages.
- Le contexte d’historique de groupe utilise `channels.telegram.historyLimit` (ou `channels.telegram.accounts.*.historyLimit`), avec repli sur `messages.groupChat.historyLimit`. Definissez `0` pour desactiver (par defaut 50).
- L’historique des Messages prives peut etre limite avec `channels.telegram.dmHistoryLimit` (tours utilisateur). Surcharges par utilisateur : `channels.telegram.dms["<user_id>"].historyLimit`.

## Modes d’activation des groupes

Par defaut, le bot ne repond qu’aux mentions dans les groupes (`@botname` ou motifs dans `agents.list[].groupChat.mentionPatterns`). Pour modifier ce comportement :

### Via la config (recommande)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**Important :** Definir `channels.telegram.groups` cree une **liste d’autorisation** : seuls les groupes listes (ou `"*"`) seront acceptes.
Les sujets de forum heritent de la config du groupe parent (allowFrom, requireMention, skills, prompts), sauf si vous ajoutez des surcharges par sujet sous `channels.telegram.groups.<groupId>.topics.<topicId>`.

Pour autoriser tous les groupes avec reponse permanente :

```json5
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

Pour conserver les mentions uniquement pour tous les groupes (comportement par defaut) :

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

### Via commande (niveau session)

Envoyez dans le groupe :

- `/activation always` – repondre a tous les messages
- `/activation mention` – exiger les mentions (par defaut)

**Remarque :** Les commandes mettent a jour uniquement l’etat de la session. Pour un comportement persistant apres redemarrage, utilisez la config.

### Obtenir l’ID du chat de groupe

Transmettez n’importe quel message du groupe a `@userinfobot` ou `@getidsbot` sur Telegram pour voir l’ID du chat (nombre negatif comme `-1001234567890`).

**Astuce :** Pour votre propre ID utilisateur, envoyez un Message prive au bot et il repondra avec votre ID utilisateur (message d’appairage), ou utilisez `/whoami` une fois les commandes activees.

**Note de confidentialite :** `@userinfobot` est un bot tiers. Si vous preferez, ajoutez le bot au groupe, envoyez un message et utilisez `openclaw logs --follow` pour lire `chat.id`, ou utilisez l’API Bot `getUpdates`.

## Ecritures de configuration

Par defaut, Telegram est autorise a ecrire des mises a jour de configuration declenchees par des evenements de canal ou `/config set|unset`.

Cela se produit lorsque :

- Un groupe est mis a niveau en supergroupe et Telegram emet `migrate_to_chat_id` (l’ID de chat change). OpenClaw peut migrer `channels.telegram.groups` automatiquement.
- Vous executez `/config set` ou `/config unset` dans un chat Telegram (necessite `commands.config: true`).

Desactiver avec :

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Sujets (supergroupes forum)

Les sujets de forum Telegram incluent un `message_thread_id` par message. OpenClaw :

- Ajoute `:topic:<threadId>` a la cle de session du groupe Telegram afin que chaque sujet soit isole.
- Envoie des indicateurs de saisie et des reponses avec `message_thread_id` pour que les reponses restent dans le sujet.
- Le sujet general (id de fil `1`) est special : les envois de message omettent `message_thread_id` (rejete par Telegram), mais les indicateurs de saisie l’incluent toujours.
- Expose `MessageThreadId` + `IsForum` dans le contexte de modele pour le routage/la mise en forme.
- Une configuration specifique au sujet est disponible sous `channels.telegram.groups.<chatId>.topics.<threadId>` (skills, listes d’autorisation, reponse automatique, invites systeme, desactivation).
- Les configurations de sujet heritent des parametres du groupe (requireMention, listes d’autorisation, skills, invites, active) sauf surcharge par sujet.

Les chats prives peuvent inclure `message_thread_id` dans certains cas limites. OpenClaw conserve la cle de session DM inchangée, mais utilise tout de meme l’id de fil pour les reponses/le streaming de brouillons lorsqu’il est present.

## Boutons en ligne

Telegram prend en charge les claviers en ligne avec boutons de rappel.

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

Pour une configuration par compte :

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

Portées :

- `off` — boutons en ligne desactives
- `dm` — Messages prives uniquement (cibles de groupe bloquees)
- `group` — groupes uniquement (cibles de Message prive bloquees)
- `all` — Messages prives + groupes
- `allowlist` — Messages prives + groupes, mais uniquement les expéditeurs autorises par `allowFrom`/`groupAllowFrom` (memes regles que les commandes de controle)

Par defaut : `allowlist`.
Legacy : `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Envoi de boutons

Utilisez l’outil message avec le parametre `buttons` :

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

Lorsqu’un utilisateur clique sur un bouton, les donnees de rappel sont renvoyees a l’agent sous forme de message au format :
`callback_data: value`

### Options de configuration

Les capacites Telegram peuvent etre configurees a deux niveaux (forme objet ci‑dessus ; les tableaux de chaines legacy sont toujours pris en charge) :

- `channels.telegram.capabilities` : configuration globale par defaut appliquee a tous les comptes Telegram sauf surcharge.
- `channels.telegram.accounts.<account>.capabilities` : capacites par compte qui remplacent les valeurs globales pour ce compte specifique.

Utilisez le parametre global lorsque tous les bots/comptes Telegram doivent se comporter de la meme facon. Utilisez la configuration par compte lorsque des bots differents necessitent des comportements differents (par exemple, un compte ne gere que les Messages prives tandis qu’un autre est autorise dans les groupes).

## Controle d’acces (Messages prives + groupes)

### Accès DM

- Par defaut : `channels.telegram.dmPolicy = "pairing"`. Les expéditeurs inconnus recoivent un code d’appairage ; les messages sont ignores jusqu’a approbation (les codes expirent apres 1 heure).
- Approuver via :
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- L’appairage est l’echange de token par defaut pour les Messages prives Telegram. Details : [Appairage](/start/pairing)
- `channels.telegram.allowFrom` accepte des IDs utilisateur numeriques (recommande) ou des entrees `@username`. Ce n’est **pas** le nom d’utilisateur du bot ; utilisez l’ID de l’expediteur humain. L’assistant accepte `@username` et le resout vers l’ID numerique lorsque possible.

#### Trouver votre ID utilisateur Telegram

Plus sûr (sans bot tiers) :

1. Demarrez la Gateway (passerelle) et envoyez un Message prive a votre bot.
2. Executez `openclaw logs --follow` et recherchez `from.id`.

Alternative (API Bot officielle) :

1. Envoyez un Message prive a votre bot.
2. Recuperez les mises a jour avec le token du bot et lisez `message.from.id` :

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Tiers (moins prive) :

- Envoyez un Message prive a `@userinfobot` ou `@getidsbot` et utilisez l’ID utilisateur renvoye.

### Acces aux groupes

Deux controles independants :

**1. Quels groupes sont autorises** (liste d’autorisation de groupe via `channels.telegram.groups`) :

- Pas de config `groups` = tous les groupes autorises
- Avec config `groups` = seuls les groupes listes ou `"*"` sont autorises
- Exemple : `"groups": { "-1001234567890": {}, "*": {} }` autorise tous les groupes

**2. Quels expéditeurs sont autorises** (filtrage des expéditeurs via `channels.telegram.groupPolicy`) :

- `"open"` = tous les expéditeurs des groupes autorises peuvent envoyer des messages
- `"allowlist"` = seuls les expéditeurs dans `channels.telegram.groupAllowFrom` peuvent envoyer des messages
- `"disabled"` = aucun message de groupe accepte
  La valeur par defaut est `groupPolicy: "allowlist"` (bloque tant que vous n’ajoutez pas `groupAllowFrom`).

La plupart des utilisateurs veulent : `groupPolicy: "allowlist"` + `groupAllowFrom` + des groupes specifiques listes dans `channels.telegram.groups`

Pour autoriser **tout membre du groupe** a parler dans un groupe specifique (tout en conservant les commandes de controle reservees aux expéditeurs autorises), definissez une surcharge par groupe :

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

## Long‑polling vs webhook

- Par defaut : long‑polling (aucune URL publique requise).
- Mode webhook : definissez `channels.telegram.webhookUrl` et `channels.telegram.webhookSecret` (optionnellement `channels.telegram.webhookPath`).
  - L’ecouteur local se lie a `0.0.0.0:8787` et sert `POST /telegram-webhook` par defaut.
  - Si votre URL publique est differente, utilisez un proxy inverse et pointez `channels.telegram.webhookUrl` vers le point de terminaison public.

## Fil de reponse

Telegram prend en charge les reponses filaires optionnelles via des balises :

- `[[reply_to_current]]` — repondre au message declencheur.
- `[[reply_to:<id>]]` — repondre a un ID de message specifique.

Controle par `channels.telegram.replyToMode` :

- `first` (par defaut), `all`, `off`.

## Messages audio (voix vs fichier)

Telegram distingue les **notes vocales** (bulle ronde) des **fichiers audio** (carte avec metadonnees).
OpenClaw utilise par defaut les fichiers audio pour compatibilite ascendante.

Pour forcer une bulle de note vocale dans les reponses de l’agent, incluez cette balise n’importe ou dans la reponse :

- `[[audio_as_voice]]` — envoyer l’audio comme note vocale au lieu d’un fichier.

La balise est supprimee du texte livre. Les autres canaux ignorent cette balise.

Pour les envois via l’outil message, definissez `asVoice: true` avec une URL `media` audio compatible voix
(`message` est optionnel lorsque le media est present) :

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## Video messages (video vs video note)

Telegram distinguishes **video notes** (round bubble) from **video files** (rectangular).
OpenClaw defaults to video files.

For message tool sends, set `asVideoNote: true` with a video `media` URL:

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(Remarque : les notes vidéo ne prennent pas en charge les sous-titres. Si vous fournissez un texte de message, il sera envoyé comme un message distinct.)

## Autocollants (stickers)

OpenClaw prend en charge la reception et l’envoi d’autocollants Telegram avec une mise en cache intelligente.

### Reception d’autocollants

Lorsqu’un utilisateur envoie un autocollant, OpenClaw le gere selon le type :

- **Autocollants statiques (WEBP) :** telecharges et traites via la vision. L’autocollant apparait comme un emplacement `<media:sticker>` dans le contenu du message.
- **Autocollants animes (TGS) :** ignores (format Lottie non pris en charge pour le traitement).
- **Autocollants video (WEBM) :** ignores (format video non pris en charge pour le traitement).

Champ de contexte de modele disponible lors de la reception d’autocollants :

- `Sticker` — objet avec :
  - `emoji` — emoji associe a l’autocollant
  - `setName` — nom du pack d’autocollants
  - `fileId` — ID de fichier Telegram (renvoyer le meme autocollant)
  - `fileUniqueId` — ID stable pour la recherche en cache
  - `cachedDescription` — description de vision mise en cache lorsque disponible

### Cache des autocollants

Les autocollants sont traites via les capacites de vision de l’IA pour generer des descriptions. Comme les memes autocollants sont souvent envoyes a repetiton, OpenClaw met ces descriptions en cache pour eviter des appels API redondants.

**Fonctionnement :**

1. **Premiere rencontre :** L’image de l’autocollant est envoyee a l’IA pour analyse de vision. L’IA genere une description (par exemple, « Un chat de dessin anime qui salue avec enthousiasme »).
2. **Stockage en cache :** La description est enregistree avec l’ID de fichier de l’autocollant, l’emoji et le nom du pack.
3. **Rencontres suivantes :** Lorsque le meme autocollant est vu a nouveau, la description en cache est utilisee directement. L’image n’est pas envoyee a l’IA.

**Emplacement du cache :** `~/.openclaw/telegram/sticker-cache.json`

**Format d’entree du cache :**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**Avantages :**

- Reduit les couts API en evitant des appels de vision repetes pour le meme autocollant
- Temps de reponse plus rapides pour les autocollants en cache (pas de delai de traitement vision)
- Permet la fonctionnalite de recherche d’autocollants basee sur les descriptions en cache

Le cache est alimente automatiquement a mesure que les autocollants sont recus. Aucune gestion manuelle du cache n’est necessaire.

### Envoi d’autocollants

L’agent peut envoyer et rechercher des autocollants via les actions `sticker` et `sticker-search`. Celles‑ci sont desactivees par defaut et doivent etre activees dans la config :

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

**Envoyer un autocollant :**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parametres :

- `fileId` (requis) — l’ID de fichier Telegram de l’autocollant. Obtenez‑le depuis `Sticker.fileId` lors de la reception d’un autocollant, ou depuis un resultat `sticker-search`.
- `replyTo` (optionnel) — ID du message auquel repondre.
- `threadId` (optionnel) — ID de fil de message pour les sujets de forum.

**Rechercher des autocollants :**

L’agent peut rechercher des autocollants mis en cache par description, emoji ou nom de pack :

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Renvoie les autocollants correspondants depuis le cache :

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

La recherche utilise un appariement approximatif sur le texte de description, les caracteres emoji et les noms de pack.

**Exemple avec fil de discussion :**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Streaming (brouillons)

Telegram peut diffuser des **bulles de brouillon** pendant que l’agent genere une reponse.
OpenClaw utilise l’API Bot `sendMessageDraft` (pas de vrais messages), puis envoie la
reponse finale comme message normal.

Exigences (API Bot Telegram 9.3+) :

- **Chats prives avec sujets actives** (mode sujet de forum pour le bot).
- Les messages entrants doivent inclure `message_thread_id` (fil de sujet prive).
- Le streaming est ignore pour les groupes/supergroupes/canaux.

Config :

- `channels.telegram.streamMode: "off" | "partial" | "block"` (par defaut : `partial`)
  - `partial` : mettre a jour la bulle de brouillon avec le dernier texte de streaming.
  - `block` : mettre a jour la bulle de brouillon par blocs plus larges (segmente).
  - `off` : desactiver le streaming de brouillons.
- Optionnel (uniquement pour `streamMode: "block"`) :
  - `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference? }`
    - valeurs par defaut : `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (plafonne a `channels.telegram.textChunkLimit`).

Remarque : le streaming de brouillons est distinct du **streaming par blocs** (messages de canal).
Le streaming par blocs est desactive par defaut et necessite `channels.telegram.blockStreaming: true`
si vous souhaitez des messages Telegram anticipes au lieu de mises a jour de brouillon.

Flux de raisonnement (Telegram uniquement) :

- `/reasoning stream` diffuse le raisonnement dans la bulle de brouillon pendant la generation
  de la reponse, puis envoie la reponse finale sans raisonnement.
- Si `channels.telegram.streamMode` est `off`, le flux de raisonnement est desactive.
  Plus de contexte : [Streaming + segmentation](/concepts/streaming).

## Politique de reessai

Les appels sortants a l’API Telegram reessaient en cas d’erreurs reseau transitoires/429 avec backoff exponentiel et jitter. Configurez via `channels.telegram.retry`. Voir [Politique de reessai](/concepts/retry).

## Outil agent (messages + reactions)

- Outil : `telegram` avec action `sendMessage` (`to`, `content`, `mediaUrl` optionnel, `replyToMessageId`, `messageThreadId`).
- Outil : `telegram` avec action `react` (`chatId`, `messageId`, `emoji`).
- Outil : `telegram` avec action `deleteMessage` (`chatId`, `messageId`).
- Semantique de suppression des reactions : voir [/tools/reactions](/tools/reactions).
- Filtrage des outils : `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (par defaut : active), et `channels.telegram.actions.sticker` (par defaut : desactive).

## Notifications de reactions

**Fonctionnement des reactions :**
Les reactions Telegram arrivent sous forme d’**evenements `message_reaction` distincts**, et non comme des proprietes dans les charges de message. Lorsqu’un utilisateur ajoute une reaction, OpenClaw :

1. Recoit la mise a jour `message_reaction` depuis l’API Telegram
2. La convertit en **evenement systeme** au format : `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. Met en file l’evenement systeme en utilisant la **meme cle de session** que les messages normaux
4. A l’arrivee du message suivant dans cette conversation, les evenements systeme sont vidanges et prependus au contexte de l’agent

L’agent voit les reactions comme des **notifications systeme** dans l’historique de conversation, et non comme des metadonnees de message.

**Configuration :**

- `channels.telegram.reactionNotifications` : controle quelles reactions declenchent des notifications
  - `"off"` — ignorer toutes les reactions
  - `"own"` — notifier lorsque les utilisateurs reagissent aux messages du bot (meilleur effort ; en memoire) (par defaut)
  - `"all"` — notifier pour toutes les reactions

- `channels.telegram.reactionLevel` : controle la capacite de reaction de l’agent
  - `"off"` — l’agent ne peut pas reagir aux messages
  - `"ack"` — le bot envoie des reactions d’accuse de reception (👀 pendant le traitement) (par defaut)
  - `"minimal"` — l’agent peut reagir avec parcimonie (ligne directrice : 1 pour 5–10 echanges)
  - `"extensive"` — l’agent peut reagir librement lorsque approprie

**Groupes forum :** Les reactions dans les groupes forum incluent `message_thread_id` et utilisent des cles de session comme `agent:main:telegram:group:{chatId}:topic:{threadId}`. Cela garantit que les reactions et les messages du meme sujet restent ensemble.

**Exemple de config :**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**Exigences :**

- Les bots Telegram doivent explicitement demander `message_reaction` dans `allowed_updates` (configure automatiquement par OpenClaw)
- En mode webhook, les reactions sont incluses dans le webhook `allowed_updates`
- En mode polling, les reactions sont incluses dans les `getUpdates` `allowed_updates`

## Cibles de livraison (CLI/cron)

- Utilisez un ID de chat (`123456789`) ou un nom d’utilisateur (`@name`) comme cible.
- Exemple : `openclaw message send --channel telegram --target 123456789 --message "hi"`.

## Problemes courants

**Le bot ne repond pas aux messages sans mention dans un groupe :**

- Si vous avez defini `channels.telegram.groups.*.requireMention=false`, le **mode confidentialite** de l’API Bot Telegram doit etre desactive.
  - BotFather : `/setprivacy` → **Desactiver** (puis retirer + rajouter le bot au groupe)
- `openclaw channels status` affiche un avertissement lorsque la config attend des messages de groupe sans mention.
- `openclaw channels status --probe` peut verifier en plus l’appartenance pour des IDs de groupe numeriques explicites (il ne peut pas auditer les regles generiques `"*"`).
- Test rapide : `/activation always` (session uniquement ; utilisez la config pour la persistance)

**Le bot ne voit aucun message de groupe :**

- Si `channels.telegram.groups` est defini, le groupe doit etre liste ou utiliser `"*"`
- Verifiez les parametres de confidentialite dans @BotFather → « Group Privacy » doit etre **OFF**
- Verifiez que le bot est bien membre (pas seulement admin sans acces lecture)
- Verifiez les journaux de la Gateway (passerelle) : `openclaw logs --follow` (recherchez « skipping group message »)

**Le bot repond aux mentions mais pas a `/activation always` :**

- La commande `/activation` met a jour l’etat de la session mais ne persiste pas dans la config
- Pour un comportement persistant, ajoutez le groupe a `channels.telegram.groups` avec `requireMention: false`

**Les commandes comme `/status` ne fonctionnent pas :**

- Assurez‑vous que votre ID utilisateur Telegram est autorise (via appairage ou `channels.telegram.allowFrom`)
- Les commandes necessitent une autorisation meme dans les groupes avec `groupPolicy: "open"`

**Le long‑polling s’arrete immediatement sur Node 22+ (souvent avec proxies/fetch personnalise) :**

- Node 22+ est plus strict concernant les instances `AbortSignal` ; des signaux etrangers peuvent interrompre les appels `fetch` immediatement.
- Mettez a niveau vers une version d’OpenClaw qui normalise les signaux d’annulation, ou executez la Gateway (passerelle) sur Node 20 jusqu’a la mise a niveau.

**Le bot demarre puis cesse silencieusement de repondre (ou journalise `HttpError: Network request ... failed`) :**

- Certains hebergeurs resolvent `api.telegram.org` en IPv6 en premier. Si votre serveur n’a pas de sortie IPv6 fonctionnelle, grammY peut rester bloque sur des requetes IPv6 uniquement.
- Corrigez en activant la sortie IPv6 **ou** en forcant la resolution IPv4 pour `api.telegram.org` (par exemple, ajoutez une entree `/etc/hosts` utilisant l’enregistrement A IPv4, ou preferez IPv4 dans la pile DNS de votre OS), puis redemarrez la Gateway (passerelle).
- Verification rapide : `dig +short api.telegram.org A` et `dig +short api.telegram.org AAAA` pour confirmer ce que renvoie le DNS.

## Reference de configuration (Telegram)

Configuration complete : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.telegram.enabled` : activer/desactiver le demarrage du canal.
- `channels.telegram.botToken` : token du bot (BotFather).
- `channels.telegram.tokenFile` : lire le token depuis un chemin de fichier.
- `channels.telegram.dmPolicy` : `pairing | allowlist | open | disabled` (par defaut : appairage).
- `channels.telegram.allowFrom` : liste d’autorisation DM (ids/noms d’utilisateur). `open` requiert `"*"`.
- `channels.telegram.groupPolicy` : `open | allowlist | disabled` (par defaut : liste d’autorisation).
- `channels.telegram.groupAllowFrom` : liste d’autorisation des expéditeurs de groupe (ids/noms d’utilisateur).
- `channels.telegram.groups` : valeurs par defaut par groupe + liste d’autorisation (utilisez `"*"` pour les valeurs globales).
  - `channels.telegram.groups.<id>.groupPolicy` : surcharge par groupe pour groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention` : filtrage par mention par defaut.
  - `channels.telegram.groups.<id>.skills` : filtre de skills (omis = tous les skills, vide = aucun).
  - `channels.telegram.groups.<id>.allowFrom` : surcharge de liste d’autorisation des expéditeurs par groupe.
  - `channels.telegram.groups.<id>.systemPrompt` : invite systeme supplementaire pour le groupe.
  - `channels.telegram.groups.<id>.enabled` : desactiver le groupe lorsque `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*` : surcharges par sujet (memes champs que le groupe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy` : surcharge par sujet pour groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention` : surcharge de filtrage par mention par sujet.
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
- `channels.telegram.actions.reactions` : filtrer les reactions de l’outil Telegram.
- `channels.telegram.actions.sendMessage` : filtrer les envois de messages de l’outil Telegram.
- `channels.telegram.actions.deleteMessage` : filtrer les suppressions de messages de l’outil Telegram.
- `channels.telegram.actions.sticker` : filtrer les actions d’autocollants Telegram — envoi et recherche (par defaut : false).
- `channels.telegram.reactionNotifications` : `off | own | all` — controler quelles reactions declenchent des evenements systeme (par defaut : `own` lorsqu’il n’est pas defini).
- `channels.telegram.reactionLevel` : `off | ack | minimal | extensive` — controler la capacite de reaction de l’agent (par defaut : `minimal` lorsqu’il n’est pas defini).

Options globales associees :

- `agents.list[].groupChat.mentionPatterns` (motifs de filtrage par mention).
- `messages.groupChat.mentionPatterns` (repli global).
- `commands.native` (par defaut `"auto"` → actif pour Telegram/Discord, inactif pour Slack), `commands.text`, `commands.useAccessGroups` (comportement des commandes). Surcharger avec `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`.

