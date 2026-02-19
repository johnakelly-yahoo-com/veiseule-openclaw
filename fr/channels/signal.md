---
summary: "Prise en charge de Signal via signal-cli (JSON-RPC + SSE), configuration et modèle de numéros"
read_when:
  - Configuration de la prise en charge de Signal
  - Dépannage de l’envoi/réception Signal
title: "Signal"
---

# Signal (signal-cli)

Statut : intégration CLI externe. La Gateway communique avec `signal-cli` via HTTP JSON-RPC + SSE.

## Prérequis

- OpenClaw installé sur votre serveur (le flux Linux ci-dessous a été testé sur Ubuntu 24).
- `signal-cli` disponible sur l’hôte où la gateway s’exécute.
- Un numéro de téléphone pouvant recevoir un SMS de vérification (pour le parcours d’enregistrement par SMS).
- Accès à un navigateur pour le captcha Signal (`signalcaptchas.org`) lors de l’enregistrement.

## Configuration (chemin rapide)

1. Utilisez un **numero Signal distinct** pour le bot (recommande).
2. Installez `signal-cli` (Java requis).
3. Choisissez un parcours d’installation :
   - `signal-cli link -n "OpenClaw"`
   - **Parcours B (enregistrement SMS) :** enregistrez un numéro dédié avec captcha + vérification SMS.
4. Configurez OpenClaw et demarrez la Gateway.
5. Envoyez un premier DM et approuvez l’appairage (`openclaw pairing approve signal <CODE>`).

Configuration minimale :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Référence des champs :

| Champ       | Description                                                                                    |
| ----------- | ---------------------------------------------------------------------------------------------- |
| `account`   | Numéro de téléphone du bot au format E.164 (`+15551234567`) |
| `cliPath`   | Demarrage rapide (debutant)                                                 |
| `dmPolicy`  | Politique d’accès DM (`pairing` recommandé)                                 |
| `allowFrom` | Numéros de téléphone ou valeurs `uuid:&lt;id&gt;` autorisés à envoyer des DM                         |

## Ce que c’est

- Canal Signal via `signal-cli` (pas de libsignal embarquee).
- Routage deterministe : les reponses reviennent toujours sur Signal.
- Les Messages prives partagent la session principale de l’agent ; les groupes sont isoles (`agent:<agentId>:signal:group:<groupId>`).

## Ecritures de configuration

Par defaut, Signal est autorise a ecrire des mises a jour de configuration declenchees par `/config set|unset` (necessite `commands.config: true`).

Desactiver avec :

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Le modele de numeros (important)

- La Gateway se connecte a un **appareil Signal** (le compte `signal-cli`).
- Si vous executez le bot sur **votre compte Signal personnel**, il ignorera vos propres messages (protection contre les boucles).
- Pour « je texte le bot et il repond », utilisez un **numero de bot distinct**.

## Parcours d’installation A : lier un compte Signal existant (QR)

1. Installez `signal-cli` (Java requis).
2. Liez un compte bot :
   - `signal-cli link -n "OpenClaw"` puis scannez le QR dans Signal.
3. Configurez Signal et demarrez la Gateway.

Exemple :

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Prise en charge multi-comptes : utilisez `channels.signal.accounts` avec une configuration par compte et `name` en option. Voir [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) pour le modele partage.

## Parcours d’installation B : enregistrer un numéro de bot dédié (SMS, Linux)

Utilisez cette option si vous souhaitez un numéro de bot dédié au lieu de lier un compte Signal existant.

1. Obtenez un numéro capable de recevoir des SMS (ou une vérification vocale pour les lignes fixes).
   - Utilisez un numéro de bot dédié afin d’éviter les conflits de compte/session.
2. Installez `signal-cli` sur l’hôte du gateway :

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Si vous utilisez la version JVM (`signal-cli-${VERSION}.tar.gz`), installez d’abord JRE 25+.
Maintenez `signal-cli` à jour ; en amont, il est indiqué que les anciennes versions peuvent cesser de fonctionner lorsque les API serveur de Signal évoluent.

3. Enregistrez et vérifiez le numéro :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Si un captcha est requis :

1. Ouvrez `https://signalcaptchas.org/registration/generate.html`.
2. Complétez le captcha, puis copiez la cible du lien `signalcaptcha://...` depuis « Open Signal ».
3. Exécutez depuis la même IP externe que la session du navigateur lorsque c’est possible.
4. Relancez immédiatement l’enregistrement (les jetons captcha expirent rapidement) :

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Liez l’appareil du bot et demarrez le daemon :

```bash
# Si vous exécutez le gateway en tant que service systemd utilisateur :
systemctl --user restart openclaw-gateway

# Puis vérifiez :
openclaw doctor
openclaw channels status --probe
```

5. Associez votre expéditeur DM :
   - Envoyez n’importe quel message au numéro du bot.
   - Approuvez le code sur le serveur : `openclaw pairing approve signal <PAIRING_CODE>`.
   - Enregistrez le numéro du bot comme contact sur votre téléphone pour éviter « Unknown contact ».

Important : l’enregistrement d’un compte de numéro de téléphone avec `signal-cli` peut déconnecter la session principale de l’application Signal pour ce numéro. Privilégiez un numéro de bot dédié, ou utilisez le mode de liaison par QR si vous devez conserver la configuration existante de votre application mobile.

Références en amont :

- `signal-cli` README : `https://github.com/AsamK/signal-cli`
- Flux captcha : `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Flux de liaison : `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Mode daemon externe (httpUrl)

Si vous souhaitez gerer `signal-cli` vous-meme (demarrages JVM a froid lents, initialisation de conteneur ou CPU partages), lancez le daemon separement et pointez OpenClaw dessus :

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Cela ignore l’auto-lancement et l’attente de demarrage dans OpenClaw. Pour des demarrages lents lors de l’auto-lancement, definissez `channels.signal.startupTimeoutMs`.

## Controle d’acces (Messages prives + groupes)

DMs:

- Par defaut : `channels.signal.dmPolicy = "pairing"`.
- Les expediteurs inconnus recoivent un code d’appairage ; les messages sont ignores jusqu’a approbation (les codes expirent apres 1 heure).
- Approuver via :
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- L’appairage est l’echange de jeton par defaut pour les Messages prives Signal. Details : [Appairage](/start/pairing)
- Les expediteurs uniquement UUID (depuis `sourceUuid`) sont stockes comme `uuid:<id>` dans `channels.signal.allowFrom`.

Groupes :

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` controle qui peut declencher dans les groupes lorsque `allowlist` est defini.

## Comment ça marche (comportement)

- `signal-cli` s’execute comme un daemon ; la Gateway lit les evenements via SSE.
- Les messages entrants sont normalises dans l’enveloppe de canal partagee.
- Les reponses sont toujours renvoyees vers le meme numero ou groupe.

## Medias + limites

- Le texte sortant est segmente en blocs de `channels.signal.textChunkLimit` (par defaut 4000).
- Segmentation optionnelle par sauts de ligne : definir `channels.signal.chunkMode="newline"` pour decouper sur les lignes vides (frontieres de paragraphes) avant la segmentation par longueur.
- Pieces jointes prises en charge (base64 recupere depuis `signal-cli`).
- Limite media par defaut : `channels.signal.mediaMaxMb` (par defaut 8).
- Utilisez `channels.signal.ignoreAttachments` pour ignorer le telechargement des medias.
- Le contexte d’historique de groupe utilise `channels.signal.historyLimit` (ou `channels.signal.accounts.*.historyLimit`), avec repli vers `messages.groupChat.historyLimit`. Definir `0` pour desactiver (par defaut 50).

## Indicateurs de saisie + accusés de lecture

- **Indicateurs de saisie** : OpenClaw envoie des signaux de saisie via `signal-cli sendTyping` et les rafraichit pendant l’execution d’une reponse.
- **Accuses de lecture** : lorsque `channels.signal.sendReadReceipts` est vrai, OpenClaw transmet les accuses de lecture pour les Messages prives autorises.
- Signal-cli n’expose pas les accuses de lecture pour les groupes.

## Reactions (outil message)

- Utilisez `message action=react` avec `channel=signal`.
- Cibles : expediteur E.164 ou UUID (utilisez `uuid:<id>` depuis la sortie d’appairage ; l’UUID brut fonctionne aussi).
- `messageId` est l’horodatage Signal du message auquel vous reagissez.
- Les reactions en groupe necessitent `targetAuthor` ou `targetAuthorUuid`.

Exemples :

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuration :

- `channels.signal.actions.reactions` : activer/desactiver les actions de reaction (par defaut true).
- `channels.signal.reactionLevel` : `off | ack | minimal | extensive`.
  - `off`/`ack` desactive les reactions de l’agent (l’outil message `react` renverra une erreur).
  - `minimal`/`extensive` active les reactions de l’agent et definit le niveau de guidage.
- Surcharges par compte : `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Cibles de livraison (CLI/cron)

- Messages prives : `signal:+15551234567` (ou E.164 simple).
- Messages prives UUID : `uuid:<id>` (ou UUID brut).
- Groupes : `signal:group:<groupId>`.
- Noms d’utilisateur : `username:<name>` (si pris en charge par votre compte Signal).

## Problemes courants

Exécutez d'abord cette échelle :

```bash
openclaw models auth paste-token --provider anthropic
openclaw models status
```

Ensuite, confirmez l'état d'appairage du DM si nécessaire:

```bash
openclaw pairing list signal
```

Échecs communs :

- Le démon est joignable mais pas de réponses : vérifiez les paramètres du compte/démon (`httpUrl`, `account`) et le mode réception.
- DMs ignorés: l'expéditeur est en attente d'approbation du jumelage.
- Les messages de groupe ont été ignorés : envoi de blocs de barrière d'expéditeur/mention de groupe.
- Erreurs de validation de configuration après modification : exécutez `openclaw doctor --fix`.
- Signal absent des diagnostics : vérifiez que `channels.signal.enabled: true`.

Vérifications supplémentaires :

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

channels/signal.md

## Notes de sécurité

- `signal-cli` stocke les clés de compte localement (généralement `~/.local/share/signal-cli/data/`).
- Sauvegardez l’état du compte Signal avant une migration ou une reconstruction du serveur.
- openclaw models auth paste-token --provider anthropic
  openclaw models status
- La vérification par SMS n’est requise que pour l’enregistrement ou les procédures de récupération, mais la perte de contrôle du numéro/compte peut compliquer un nouvel enregistrement.

## Reference de configuration (Signal)

Configuration complete : [Configuration](/gateway/configuration)

Options du fournisseur :

- `channels.signal.enabled` : activer/desactiver le demarrage du canal.
- `channels.signal.account` : E.164 pour le compte bot.
- `channels.signal.cliPath` : chemin vers `signal-cli`.
- `channels.signal.httpUrl` : URL complete du daemon (remplace hote/port).
- `channels.signal.httpHost`, `channels.signal.httpPort` : liaison du daemon (par defaut 127.0.0.1:8080).
- `channels.signal.autoStart` : auto-lancement du daemon (par defaut true si `httpUrl` n’est pas defini).
- `channels.signal.startupTimeoutMs` : delai d’attente au demarrage en ms (plafond 120000).
- `channels.signal.receiveMode` : `on-start | manual`.
- `channels.signal.ignoreAttachments` : ignorer le telechargement des pieces jointes.
- `channels.signal.ignoreStories` : ignorer les stories du daemon.
- `channels.signal.sendReadReceipts` : transmettre les accuses de lecture.
- `channels.signal.dmPolicy` : `pairing | allowlist | open | disabled` (par defaut : appairage).
- `channels.signal.allowFrom` : liste blanche des Messages prives (E.164 ou `uuid:<id>`). `open` necessite `"*"`. Signal n’a pas de noms d’utilisateur ; utilisez des identifiants telephone/UUID.
- `channels.signal.groupPolicy` : `open | allowlist | disabled` (par defaut : liste blanche).
- `channels.signal.groupAllowFrom` : liste blanche des expediteurs de groupe.
- `channels.signal.historyLimit` : nombre maximal de messages de groupe a inclure comme contexte (0 desactive).
- `channels.signal.dmHistoryLimit` : limite d’historique des Messages prives en tours utilisateur. Surcharges par utilisateur : `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit` : taille de segmentation sortante (caracteres).
- `channels.signal.chunkMode` : `length` (par defaut) ou `newline` pour decouper sur les lignes vides (frontieres de paragraphes) avant la segmentation par longueur.
- `channels.signal.mediaMaxMb` : limite media entrante/sortante (Mo).

Options globales associees :

- `agents.list[].groupChat.mentionPatterns` (Signal ne prend pas en charge les mentions natives).
- `messages.groupChat.mentionPatterns` (repli global).
- `messages.responsePrefix`.

