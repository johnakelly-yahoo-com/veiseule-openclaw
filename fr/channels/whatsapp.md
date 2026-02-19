---
summary: "Prise en charge du canal WhatsApp, contrôles d’accès, comportement de livraison et opérations"
read_when:
  - Travail sur le comportement du canal WhatsApp/web ou le routage de la boîte de réception
title: "WhatsApp"
---

# WhatsApp (canal web)

Statut : WhatsApp Web via Baileys uniquement. La Gateway (passerelle) possède la/les session(s).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    La politique DM par défaut est l’appairage pour les expéditeurs inconnus.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostics inter-canaux et guides de résolution.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Modèles et exemples complets de configuration des canaux.
  
</Card>
</CardGroup>

## Carte rapide de configuration

<Steps>
  <Step title="Configure WhatsApp access policy">

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

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Pour un compte spécifique :
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Approuver avec : `openclaw pairing approve whatsapp <code>` (liste avec `openclaw pairing list whatsapp`).
```

    ```
    Les codes expirent après 1 heure ; les demandes en attente sont plafonnées à 3 par canal.
    ```

  
</Step>
</Steps>

<Note>
Idéal pour garder votre WhatsApp personnel séparé — installez WhatsApp Business et enregistrez-y le numéro OpenClaw. 
(Les métadonnées du canal et le flux d’onboarding sont optimisés pour cette configuration, mais les configurations avec numéro personnel sont également prises en charge.)
</Note>

## Modèles de déploiement

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Il s’agit du mode opérationnel le plus propre :


    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    L’onboarding prend en charge le mode numéro personnel et écrit une configuration de base adaptée à l’auto-chat :


    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">Le canal de la plateforme de messagerie est basé sur WhatsApp Web (`Baileys`) dans l’architecture actuelle des canaux OpenClaw.

    ```
    Il n’existe pas de canal de messagerie WhatsApp Twilio distinct dans le registre intégré des canaux de chat.
    ```

  
</Accordion>
</AccordionGroup>

## Modèle d’exécution

- Gateway gère le socket WhatsApp et la boucle de reconnexion.
- Les envois sortants nécessitent un listener WhatsApp actif pour le compte cible.
- Les discussions de statut et de diffusion sont ignorées (`@status`, `@broadcast`).
- Les discussions directes utilisent les règles de session DM (`session.dmScope` ; la valeur par défaut `main` regroupe les DMs dans la session principale de l’agent).
- Les groupes correspondent à des sessions `agent:<agentId>:whatsapp:group:<jid>`.

## Contrôle d’accès et activation

<Tabs>
  <Tab title="DM policy">**Politique de messages privés** : `channels.whatsapp.dmPolicy` contrôle l’accès aux discussions directes (par défaut : `pairing`).

    ```
    `channels.whatsapp.dmPolicy` (politique DM : appairage/liste d’autorisation/ouvert/désactivé).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    L’accès aux groupes comporte deux niveaux :

    ```
    `channels.whatsapp.groups` (liste d’autorisation de groupe + valeurs par défaut de filtrage par mention ; utilisez `"*"` pour autoriser tout).
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Les réponses dans les groupes nécessitent une mention par défaut.

La détection des mentions inclut :

- mentions WhatsApp explicites de l’identité du bot
- modèles regex de mention configurés (`agents.list[].groupChat.mentionPatterns`, repli vers `messages.groupChat.mentionPatterns`)
- détection implicite des réponses au bot (l’expéditeur de la réponse correspond à l’identité du bot)

Commande d’activation au niveau de la session :

- `/activation mention`
- `/activation always`

`activation` met à jour l’état de la session (pas la configuration globale). Elle est réservée au propriétaire.

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## Lorsque le numéro personnel lié est également présent dans `allowFrom`, les protections WhatsApp pour les discussions avec soi-même s’activent :

ignorer les accusés de lecture pour les messages envoyés à soi-même

- ignorer le déclenchement automatique par mention-JID qui, autrement, vous notifierait vous-même
- ```
  Les messages WhatsApp entrants sont encapsulés dans l’enveloppe d’entrée partagée.
  ```
- Les réponses en auto‑discussion utilisent par défaut `[{identity.name}]` lorsque défini (sinon `[openclaw]`)
  si `messages.responsePrefix` n’est pas défini.

## Normalisation des messages (ce que voit le modèle)

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">Si une réponse citée existe, le contexte est ajouté sous la forme suivante :

```text
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

Les champs de métadonnées de réponse sont également renseignés lorsque disponibles (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, JID/E.164 de l’expéditeur).

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Les charges utiles de localisation et de contact sont normalisées en contexte textuel avant le routage.
    ```

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Les messages entrants contenant uniquement des médias utilisent des espaces réservés :

    ```
    
        Pour les groupes, les messages non traités peuvent être mis en mémoire tampon et injectés comme contexte lorsque le bot est finalement déclenché.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">  
</Accordion>

    ```
    Les messages récents _non traités_ (50 par défaut) sont insérés sous :
    `[Chat messages since your last reply - for context]` (les messages déjà présents dans la session ne sont pas réinjectés)
    ```

  
</Accordion>

  <Accordion title="Read receipts">Par défaut, la Gateway marque les messages WhatsApp entrants comme lus (coches bleues) une fois acceptés.

    ```
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

  
</Accordion>
</AccordionGroup>

```
- prend en charge les charges utiles image, vidéo, audio (note vocale PTT) et document
- `audio/ogg` est réécrit en `audio/ogg; codecs=opus` pour la compatibilité avec les notes vocales
- la lecture des GIF animés est prise en charge via `gifPlayback: true` lors de l’envoi de vidéos
- les légendes sont appliquées au premier élément média lors de l’envoi de réponses multi-médias
- la source des médias peut être HTTP(S), `file://` ou des chemins locaux
```
---

<AccordionGroup>
  <Accordion title="Text chunking">Découpage optionnel par saut de ligne : définissez `channels.whatsapp.chunkMode="newline"` pour découper sur les lignes vides (limites de paragraphes) avant le découpage par longueur.
</Accordion>

  <Accordion title="Outbound media behavior">
    - limite d’enregistrement des médias entrants : `channels.whatsapp.mediaMaxMb` (par défaut `50`)
    - limite des médias sortants pour les réponses automatiques : `agents.defaults.mediaMaxMb` (par défaut `5MB`)
    - les images sont automatiquement optimisées (redimensionnement/ajustement de qualité) pour respecter les limites
    - en cas d’échec d’envoi de média, un mécanisme de secours envoie un avertissement textuel au lieu d’abandonner silencieusement la réponse
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">Réactions d’accusé de réception
</Accordion>
</AccordionGroup>

## envoyées immédiatement après l’acceptation d’un message entrant (pré-réponse)

`channels.whatsapp.ackReaction` (auto‑réaction à la réception des messages : `{emoji, direct, group}`).

```json5
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

Remarques :

- les échecs sont consignés dans les logs mais ne bloquent pas l’envoi normal de la réponse
- le mode groupe `mentions` réagit lors des messages déclenchés par mention ; l’activation de groupe `always` agit comme un contournement de cette vérification
- Multi-compte et identifiants
- WhatsApp ignore `messages.ackReaction` ; utilisez `channels.whatsapp.ackReaction` à la place.

## ]\` efface l’état d’authentification WhatsApp pour ce compte.

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (paramètres par compte + `authDir` optionnel).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Configurez WhatsApp dans `~/.openclaw/openclaw.json`.<accountId>Identifiants stockés dans `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
</Accordion>

  <Accordion title="Logout behavior">Connexion multi‑comptes : `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>Dans les répertoires d’authentification legacy, `oauth.json` est conservé tandis que les fichiers d’authentification Baileys sont supprimés.

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## La prise en charge des outils de l’agent inclut l’action de réaction WhatsApp (`react`).

- Contrôles des actions :
- Dépannage
  - `channels.whatsapp.actions.reactions` (verrouiller les réactions d’outil WhatsApp).
  - `channels.whatsapp.groupPolicy` (politique de groupe).
- {
  channels: { whatsapp: { configWrites: false } },
  }

## ```
Symptôme : compte lié avec déconnexions répétées ou tentatives de reconnexion.
```

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Symptôme : `channels status` affiche `linked: false` ou avertit « Not linked ».

    ```
    Déconnexion : `openclaw channels logout` (ou `--account <id>`) supprime l’état d’authentification WhatsApp (mais conserve le `oauth.json` partagé).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Les envois sortants échouent immédiatement lorsqu’aucun listener gateway actif n’existe pour le compte cible.

    ```
    Correctif : `openclaw doctor` (ou redémarrez la Gateway). Si le problème persiste, reliez via `channels login` et inspectez `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">Assurez-vous que le gateway est en cours d’exécution et que le compte est lié.

    ```
    
        Vérifiez dans cet ordre :
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">Le runtime du gateway WhatsApp doit utiliser Node.

    ```
    Politique de groupe : `channels.whatsapp.groupPolicy = open|disabled|allowlist` (par défaut `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Pointeurs de référence de configuration WhatsApp (Baileys) et Telegram sont peu fiables avec Bun. Exécutez la Gateway avec **Node**.
  
</Accordion>
</AccordionGroup>

## Référence principale :

[Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

- Champs WhatsApp à fort impact :

access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- livraison : `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-compte : `accounts.<id>``.enabled`, `accounts.<id>``.authDir`, remplacements au niveau du compte
- opérations : `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- comportement de session : `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Associé

- [Appairage](/channels/pairing)
- [Routage des canaux](/channels/channel-routing)
- Guide de dépannage : [Gateway troubleshooting](/gateway/troubleshooting).

