---
summary: "Prise en charge iMessage legacy via imsg (JSON-RPC sur stdio). Les nouvelles configurations devraient utiliser BlueBubbles."
read_when:
  - Mise en place de la prise en charge iMessage
  - Débogage de l’envoi/la réception iMessage
title: "iMessage"
---

# iMessage (legacy : imsg)

<Warning>
**Recommandé :** Utilisez [BlueBubbles](/channels/bluebubbles) pour les nouvelles configurations iMessage.

Le canal `imsg` est une intégration CLI externe legacy et peut être supprimé dans une version future. 
</Warning>

Statut : intégration CLI externe legacy. La Gateway (passerelle) lance `imsg rpc` (JSON-RPC sur stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">Configurez iMessage et démarrez la passerelle.
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Les messages privés iMessage utilisent par défaut le mode d’appairage.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Référence complète des champs iMessage.
  
</Card>
</CardGroup>

## Configuration (chemin rapide)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Configurer OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Démarrer la passerelle">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Approuver le premier appairage DM (dmPolicy par défaut)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Les demandes d’appairage expirent après 1 heure.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">`channels.imessage.cliPath` peut pointer vers toute commande qui proxifie stdin/stdout (par exemple, un script wrapper qui se connecte en SSH à un autre Mac et exécute `imsg rpc`).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Configuration recommandée lorsque les pièces jointes sont activées :
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Si `remoteHost` n’est pas défini, OpenClaw tente de l’auto-détecter en analysant la commande SSH dans votre script wrapper.
    ```

  
</Tab>
</Tabs>

## Prérequis et autorisations (macOS)

- Assurez-vous que Messages est connecté sur ce Mac.
- Accès complet au disque pour OpenClaw + `imsg` (accès à la base de données Messages).
- L’autorisation d’automatisation est requise pour envoyer des messages via Messages.app.

<Tip>
macOS accorde les permissions TCC par contexte application/processus. Approuver les invites dans le même contexte que `imsg` (par exemple, Terminal/iTerm, une session LaunchAgent ou un processus lancé par SSH).

```bash
imsg chats --limit 1
# ou
imsg envoyer <handle> "test"
```

</Tip>

## Contrôle d’accès et routage

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` contrôle les messages directs :

    ```
    `channels.imessage.groupPolicy` : `open | allowlist | disabled` (par défaut : allowlist).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom` : liste d’autorisation des expéditeurs de groupe.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - Les messages privés utilisent le routage direct ; les groupes utilisent le routage de groupe.
    - Avec la valeur par défaut `session.dmScope=main`, les messages privés iMessage sont regroupés dans la session principale de l’agent.
    - Les sessions de groupe sont isolées (`agent:<agentId>Groupes :<chat_id>`).
    - Les réponses sont renvoyées vers iMessage en utilisant les métadonnées du canal/de la cible d’origine.

    ```
    Si un fil à plusieurs participants arrive avec `is_group=false`, vous pouvez quand même l’isoler en `chat_id` à l’aide de `channels.imessage.groups` (voir « Fils de type groupe » ci-dessous).
    ```

  
</Tab>
</Tabs>

## Modèles de déploiement

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Si vous voulez que le bot envoie depuis une **identité iMessage distincte** (et garder vos Messages personnels propres), utilisez un identifiant Apple dédié + un utilisateur macOS dédié.

    ```
    Faites pointer `channels.imessage.accounts.bot.cliPath` vers un wrapper SSH qui exécute `imsg` en tant qu’utilisateur bot.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Topologie courante :

    ```
    Si la Gateway (passerelle) s’exécute sur un hôte/VM Linux mais qu’iMessage doit s’exécuter sur un Mac, Tailscale est le pont le plus simple : la passerelle communique avec le Mac via le tailnet, exécute `imsg` via SSH et récupère les pièces jointes via SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Utilisez des clés SSH afin que SSH et SCP soient non interactifs.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Pour les configurations à compte unique, utilisez des options à plat (`channels.imessage.cliPath`, `channels.imessage.dbPath`) au lieu de la map `accounts`.

    ```
    Chaque compte peut remplacer des champs tels que `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` et les paramètres d’historique.
    ```

  
</Accordion>
</AccordionGroup>

## Médias, fragmentation et cibles de livraison

<AccordionGroup>
  <Accordion title="Attachments and media">Les envois de médias sont plafonnés par `channels.imessage.mediaMaxMb` (par défaut 16).
</Accordion>

  <Accordion title="Outbound chunking">Le texte sortant est découpé à `channels.imessage.textChunkLimit` (par défaut 4000).
</Accordion>

  <Accordion title="Addressing formats">
    Cibles explicites recommandées :

    ```
    handles directs : `imessage:+1555` / `sms:+1555` / `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Écritures de configuration

Par défaut, iMessage est autorisé à écrire des mises à jour de configuration déclenchées par `/config set|unset` (nécessite `commands.config: true`).

Désactiver avec :

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Dépannage

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Vérifiez le binaire et la prise en charge RPC :

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Si la sonde indique que RPC n’est pas pris en charge, mettez à jour `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Checklist:

    ```
    `channels.imessage.dmPolicy` : `pairing | allowlist | open | disabled` (par défaut : appairage).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Remarques :

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Approuver via :

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">Exécutez une commande interactive à usage unique dans un terminal GUI pour forcer l'invite de commande, puis recommencez :

```bash
`channels.imessage.cliPath` : chemin vers `imsg`.
```

    ```
    Démarrez la passerelle et approuvez toutes les invites macOS (Automatisation + Accès complet au disque).
    ```

  
</Accordion>
</AccordionGroup>

## Références de configuration

- Référence de configuration (iMessage)
- Configuration complète : [Configuration](/gateway/configuration)
- [Appairage](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
