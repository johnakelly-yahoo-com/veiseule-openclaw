---
title: "WhatsApp"
---

# WhatsApp (canal web)

Statut : WhatsApp Web via Baileys uniquement. La Gateway (passerelle) possède la/les session(s).

## Démarrage rapide (débutant)

1. Utilisez un **numéro de téléphone distinct** si possible (recommandé).
2. Configurez WhatsApp dans `~/.openclaw/openclaw.json`.
3. Exécutez `openclaw channels login` pour scanner le QR code (Appareils liés).
4. Démarrez la Gateway (passerelle).

Configuration minimale :

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## Objectifs

- Plusieurs comptes WhatsApp (multi‑compte) dans un seul processus de Gateway.
- Routage déterministe : les réponses reviennent vers WhatsApp, sans routage par modèle.
- Le modèle reçoit suffisamment de contexte pour comprendre les réponses citées.

## Écritures de configuration

Par défaut, WhatsApp est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Désactiver avec :

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Architecture (qui possède quoi)

- **Gateway** possède le socket Baileys et la boucle de boîte de réception.
- **CLI / application macOS** communiquent avec la Gateway ; pas d’utilisation directe de Baileys.
- Un **listener actif** est requis pour les envois sortants ; sinon l’envoi échoue immédiatement.

## Obtenir un numéro de téléphone (deux modes)

WhatsApp exige un véritable numéro mobile pour la vérification. Les numéros VoIP et virtuels sont généralement bloqués. Il existe deux façons prises en charge d’exécuter OpenClaw sur WhatsApp :

### Numéro dédié (recommandé)

Utilisez un **numéro distinct** pour OpenClaw. Meilleure UX, routage propre, pas de bizarreries d’auto‑discussion. Configuration idéale : **ancien/téléphone Android de secours + eSIM**. Laissez‑le connecté au Wi‑Fi et à l’alimentation, puis liez‑le via QR.

**WhatsApp Business :** Vous pouvez utiliser WhatsApp Business sur le même appareil avec un autre numéro. Idéal pour garder votre WhatsApp personnel séparé — installez WhatsApp Business et enregistrez-y le numéro OpenClaw.

**Exemple de configuration (numéro dédié, liste d’autorisation mono‑utilisateur) :**

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

**Mode d’appairage (optionnel) :**
Si vous souhaitez l’appairage plutôt qu’une liste d’autorisation, définissez `channels.whatsapp.dmPolicy` sur `pairing`. Les expéditeurs inconnus reçoivent un code d’appairage ; approuvez avec :
`openclaw pairing approve whatsapp <code>`

### Numéro personnel (solution de repli)

Solution rapide : exécutez OpenClaw sur **votre propre numéro**. Envoyez‑vous des messages (WhatsApp « Message yourself ») pour tester afin d’éviter de spammer vos contacts. Attendez‑vous à lire des codes de vérification sur votre téléphone principal pendant la configuration et les essais. **Le mode auto‑discussion doit être activé.**
Lorsque l’assistant vous demande votre numéro WhatsApp personnel, saisissez le téléphone depuis lequel vous enverrez des messages (le propriétaire/expéditeur), pas le numéro de l’assistant.

**Exemple de configuration (numéro personnel, auto‑discussion) :**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Les réponses en auto‑discussion utilisent par défaut `[{identity.name}]` lorsque défini (sinon `[openclaw]`)
si `messages.responsePrefix` n’est pas défini. Définissez‑le explicitement pour personnaliser ou désactiver
le préfixe (utilisez `""` pour le supprimer).

### Conseils pour l’obtention du numéro

- **eSIM locale** auprès de l’opérateur mobile de votre pays (le plus fiable)
  - Autriche : [hot.at](https://www.hot.at)
  - Royaume‑Uni : [giffgaff](https://www.giffgaff.com) — SIM gratuite, sans engagement
- **SIM prépayée** — bon marché, doit simplement recevoir un SMS de vérification

**À éviter :** TextNow, Google Voice, la plupart des services de « SMS gratuits » — WhatsApp les bloque agressivement.

**Astuce :** Le numéro n’a besoin de recevoir qu’un seul SMS de vérification. Ensuite, les sessions WhatsApp Web persistent via `creds.json`.

## Pourquoi pas Twilio ?

- Les premières versions d’OpenClaw prenaient en charge l’intégration WhatsApp Business de Twilio.
- Les numéros WhatsApp Business conviennent mal à un assistant personnel.
- Meta impose une fenêtre de réponse de 24 heures ; sans réponse dans les 24 dernières heures, le numéro business ne peut pas initier de nouveaux messages.
- Les usages à fort volume ou « bavards » déclenchent des blocages agressifs, car les comptes business ne sont pas conçus pour envoyer des dizaines de messages d’assistant personnel.
- Résultat : livraison peu fiable et blocages fréquents, donc le support a été retiré.

## Connexion + identifiants

- Commande de connexion : `openclaw channels login` (QR via Appareils liés).
- Connexion multi‑comptes : `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Compte par défaut (lorsque `--account` est omis) : `default` s’il est présent, sinon le premier identifiant de compte configuré (trié).
- Identifiants stockés dans `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Copie de sauvegarde dans `creds.json.bak` (restaurée en cas de corruption).
- Compatibilité héritée : les anciennes installations stockaient les fichiers Baileys directement dans `~/.openclaw/credentials/`.
- Déconnexion : `openclaw channels logout` (ou `--account <id>`) supprime l’état d’authentification WhatsApp (mais conserve le `oauth.json` partagé).
- Socket déconnecté => erreur demandant une re‑liaison.

## Flux entrant (Message privé + groupe)

- Les événements WhatsApp proviennent de `messages.upsert` (Baileys).
- Les listeners de boîte de réception sont détachés à l’arrêt pour éviter l’accumulation de gestionnaires d’événements lors des tests/redémarrages.
- Les discussions de statut/diffusion sont ignorées.
- Les discussions directes utilisent E.164 ; les groupes utilisent un JID de groupe.
- **Politique de messages privés** : `channels.whatsapp.dmPolicy` contrôle l’accès aux discussions directes (par défaut : `pairing`).
  - Appairage : les expéditeurs inconnus reçoivent un code d’appairage (approbation via `openclaw pairing approve whatsapp <code>` ; les codes expirent après 1 heure).
  - Ouvert : nécessite que `channels.whatsapp.allowFrom` inclue `"*"`.
  - Votre numéro WhatsApp lié est implicitement approuvé, donc les messages à soi‑même ignorent les vérifications `channels.whatsapp.dmPolicy` et `channels.whatsapp.allowFrom`.

### Mode numéro personnel (solution de repli)

Si vous exécutez OpenClaw sur **votre numéro WhatsApp personnel**, activez `channels.whatsapp.selfChatMode` (voir l’exemple ci‑dessus).

Comportement :

- Les messages privés sortants ne déclenchent jamais de réponses d’appairage (évite de spammer les contacts).
- Les expéditeurs inconnus entrants suivent toujours `channels.whatsapp.dmPolicy`.
- Le mode auto‑discussion (allowFrom inclut votre numéro) évite les accusés de lecture automatiques et ignore les JID de mention.
- Reçus de lecture envoyés pour les MP non auto-chat.

## Accusés de lecture

Par défaut, la Gateway marque les messages WhatsApp entrants comme lus (coches bleues) une fois acceptés.

Désactiver globalement :

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Désactiver par compte :

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

Remarques :

- Le mode auto‑discussion ignore toujours les accusés de lecture.

## FAQ WhatsApp : envoi de messages + appairage

**OpenClaw enverra‑t‑il des messages à des contacts au hasard lorsque je lie WhatsApp ?**  
Non. La politique de messages privés par défaut est **l’appairage**, donc les expéditeurs inconnus ne reçoivent qu’un code d’appairage et leur message n’est **pas traité**. OpenClaw ne répond qu’aux discussions qu’il reçoit, ou aux envois que vous déclenchez explicitement (agent/CLI).

**Comment fonctionne l’appairage sur WhatsApp ?**  
L’appairage est une porte d’accès DM pour les expéditeurs inconnus :

- Le premier message privé d’un nouvel expéditeur renvoie un code court (le message n’est pas traité).
- Approuver avec : `openclaw pairing approve whatsapp <code>` (liste avec `openclaw pairing list whatsapp`).
- Les codes expirent après 1 heure ; les demandes en attente sont plafonnées à 3 par canal.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Les réponses proviennent toujours du **même compte WhatsApp**, et les discussions directes se regroupent dans la session principale de chaque agent ; utilisez donc **un agent par personne**. Le contrôle d’accès DM (`dmPolicy`/`allowFrom`) est global par compte WhatsApp. Voir [Multi‑Agent Routing](/concepts/multi-agent).

**Pourquoi l’assistant me demande‑t‑il mon numéro de téléphone ?**  
L’assistant l’utilise pour définir votre **liste d’autorisation/propriétaire** afin que vos propres messages privés soient autorisés. Il n’est pas utilisé pour l’envoi automatique. Si vous exécutez sur votre numéro WhatsApp personnel, utilisez ce même numéro et activez `channels.whatsapp.selfChatMode`.

## Normalisation des messages (ce que voit le modèle)

- `Body` est le corps du message courant avec enveloppe.

- Le contexte de réponse citée est **toujours ajouté** :

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Les métadonnées de réponse sont également définies :
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = corps cité ou espace réservé de média
  - `ReplyToSender` = E.164 lorsque connu

- Les messages entrants contenant uniquement des médias utilisent des espaces réservés :
  - `<media:image|video|audio|document|sticker>`

## Groupes

- Les groupes correspondent à des sessions `agent:<agentId>:whatsapp:group:<jid>`.
- Politique de groupe : `channels.whatsapp.groupPolicy = open|disabled|allowlist` (par défaut `allowlist`).
- Modes d’activation :
  - `mention` (par défaut) : nécessite une @mention ou une correspondance regex.
  - `always` : déclenche toujours.
- `/activation mention|always` est réservé au propriétaire et doit être envoyé comme message autonome.
- Propriétaire = `channels.whatsapp.allowFrom` (ou l’E.164 de soi‑même s’il n’est pas défini).
- **Injection d’historique** (en attente uniquement) :
  - Les messages récents _non traités_ (50 par défaut) sont insérés sous :
    `[Chat messages since your last reply - for context]` (les messages déjà présents dans la session ne sont pas réinjectés)
  - Message courant sous :
    `[Current message - respond to this]`
  - Suffixe d’expéditeur ajouté : `[from: Name (+E164)]`
- Les métadonnées de groupe sont mises en cache 5 min (sujet + participants).

## Livraison des réponses (threading)

- WhatsApp Web envoie des messages standard (pas de threading de réponse citée dans la Gateway actuelle).
- Les balises de réponse sont ignorées sur ce canal.

## Réactions d’accusé de réception (auto‑réaction à la réception)

WhatsApp peut envoyer automatiquement des réactions emoji aux messages entrants immédiatement à la réception, avant que le bot ne génère une réponse. Cela fournit un retour instantané indiquant que le message a été reçu.

**Configuration :**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Options :**

- `emoji` (chaîne) : emoji à utiliser pour l’accusé (p. ex., « 👀 », « ✅ », « 📨 »). Vide ou omis = fonctionnalité désactivée.
- `direct` (booléen, par défaut : `true`) : envoyer des réactions dans les discussions directes/DM.
- `group` (chaîne, par défaut : `"mentions"`) : comportement en groupe :
  - `"always"` : réagir à tous les messages de groupe (même sans @mention)
  - `"mentions"` : réagir uniquement lorsque le bot est @mentionné
  - `"never"` : ne jamais réagir en groupe

**Surcharge par compte :**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Notes de comportement :**

- Les réactions sont envoyées **immédiatement** à la réception du message, avant les indicateurs de saisie ou les réponses du bot.
- Dans les groupes avec `requireMention: false` (activation : toujours), `group: "mentions"` réagit à tous les messages (pas seulement aux @mentions).
- « Fire‑and‑forget » : les échecs de réaction sont journalisés mais n’empêchent pas le bot de répondre.
- Le JID du participant est automatiquement inclus pour les réactions de groupe.
- WhatsApp ignore `messages.ackReaction` ; utilisez `channels.whatsapp.ackReaction` à la place.

## Outil agent (réactions)

- Outil : `whatsapp` avec l’action `react` (`chatJid`, `messageId`, `emoji`, `remove` optionnel).
- Optionnel : `participant` (expéditeur de groupe), `fromMe` (réagir à votre propre message), `accountId` (multi‑compte).
- Sémantique de suppression des réactions : voir [/tools/reactions](/tools/reactions).
- Garde‑fou de l’outil : `channels.whatsapp.actions.reactions` (par défaut : activé).

## Limites

- Le texte sortant est découpé à `channels.whatsapp.textChunkLimit` (4000 par défaut).
- Découpage optionnel par saut de ligne : définissez `channels.whatsapp.chunkMode="newline"` pour découper sur les lignes vides (limites de paragraphes) avant le découpage par longueur.
- Les sauvegardes de médias entrants sont plafonnées par `channels.whatsapp.mediaMaxMb` (50 Mo par défaut).
- Les éléments de médias sortants sont plafonnés par `agents.defaults.mediaMaxMb` (5 Mo par défaut).

## Envoi sortant (texte + médias)

- Utilise le listener web actif ; erreur si la Gateway n’est pas en cours d’exécution.
- Découpage du texte : 4 k max par message (configurable via `channels.whatsapp.textChunkLimit`, `channels.whatsapp.chunkMode` optionnel).
- Médias :
  - Image/vidéo/audio/document pris en charge.
  - Audio envoyé en PTT ; `audio/ogg` => `audio/ogg; codecs=opus`.
  - Légende uniquement sur le premier élément média.
  - La récupération des médias prend en charge HTTP(S) et les chemins locaux.
  - GIF animés : WhatsApp attend un MP4 avec `gifPlayback: true` pour une boucle en ligne.
    - CLI : `openclaw message send --media <mp4> --gif-playback`
    - Gateway : les paramètres `send` incluent `gifPlayback: true`

## Notes vocales (audio PTT)

WhatsApp envoie l’audio sous forme de **notes vocales** (bulle PTT).

- Meilleurs résultats : OGG/Opus. OpenClaw réécrit `audio/ogg` en `audio/ogg; codecs=opus`.
- `[[audio_as_voice]]` est ignoré pour WhatsApp (l’audio est déjà envoyé comme note vocale).

## Limites médias + optimisation

- Plafond sortant par défaut : 5 Mo (par élément média).
- Surcharge : `agents.defaults.mediaMaxMb`.
- Les images sont automatiquement optimisées en JPEG sous le plafond (redimensionnement + balayage de qualité).
- Médias trop volumineux => erreur ; la réponse média bascule vers un avertissement texte.

## Heartbeats

- **Heartbeat de la Gateway** journalise l’état de santé de la connexion (`web.heartbeatSeconds`, 60 s par défaut).
- **Heartbeat d’agent** configurable par agent (`agents.list[].heartbeat`) ou globalement
  via `agents.defaults.heartbeat` (repli lorsqu’aucune entrée par agent n’est définie).
  - Utilise l’invite de heartbeat configurée (par défaut : `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + le comportement d’ignorance `HEARTBEAT_OK`.
  - La livraison utilise par défaut le dernier canal utilisé (ou la cible configurée).

## Comportement de reconnexion

- Politique de backoff : `web.reconnect` :
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Si maxAttempts est atteint, la surveillance web s’arrête (mode dégradé).
- Déconnecté => arrêt et re‑liaison requise.

## Carte rapide de configuration

- `channels.whatsapp.dmPolicy` (politique DM : appairage/liste d’autorisation/ouvert/désactivé).
- `channels.whatsapp.selfChatMode` (configuration « même téléphone » ; le bot utilise votre numéro WhatsApp personnel).
- `channels.whatsapp.allowFrom` (liste d’autorisation DM). WhatsApp utilise des numéros E.164 (pas de noms d’utilisateur).
- `channels.whatsapp.mediaMaxMb` (plafond de sauvegarde des médias entrants).
- `channels.whatsapp.ackReaction` (auto‑réaction à la réception des messages : `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (paramètres par compte + `authDir` optionnel).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (plafond de médias entrants par compte).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (surcharge de réaction d’accusé par compte).
- `channels.whatsapp.groupAllowFrom` (liste d’autorisation des expéditeurs de groupe).
- `channels.whatsapp.groupPolicy` (politique de groupe).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (contexte d’historique de groupe ; `0` désactive).
- `channels.whatsapp.dmHistoryLimit` (limite d’historique DM en tours utilisateur). Surcharges par utilisateur : `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (liste d’autorisation de groupe + valeurs par défaut de filtrage par mention ; utilisez `"*"` pour autoriser tout).
- `channels.whatsapp.actions.reactions` (verrouiller les réactions d’outil WhatsApp).
- `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`).
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (préfixe entrant ; par compte : `channels.whatsapp.accounts.<accountId>.messagePrefix` ; obsolète : `messages.messagePrefix`)
- `messages.responsePrefix` (préfixe sortant)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (surcharge optionnelle)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (surcharges par agent)
- `session.*` (portée, inactivité, stockage, mainKey)
- `web.enabled` (désactiver le démarrage du canal lorsque false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Journaux + dépannage

- Sous‑systèmes : `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Fichier journal : `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurable).
- Guide de dépannage : [Gateway troubleshooting](/gateway/troubleshooting).

## Dépannage (rapide)

**Non lié / connexion QR requise**

- Symptôme : `channels status` affiche `linked: false` ou avertit « Not linked ».
- Correctif : exécutez `openclaw channels login` sur l’hôte de la Gateway et scannez le QR (WhatsApp → Paramètres → Appareils liés).

**Lié mais déconnecté / boucle de reconnexion**

- Symptôme : `channels status` affiche `running, disconnected` ou avertit « Linked but disconnected ».
- Correctif : `openclaw doctor` (ou redémarrez la Gateway). Si le problème persiste, reliez via `channels login` et inspectez `openclaw logs --follow`.

**Runtime Bun**

- Bun n’est **pas recommandé**. WhatsApp (Baileys) et Telegram sont peu fiables avec Bun.
  Exécutez la Gateway avec **Node**. (Voir la note de runtime dans Premiers pas.)
