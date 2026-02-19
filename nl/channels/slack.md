---
summary: "Slack-installatie voor socket- of HTTP-webhookmodus"
read_when:
  - Slack instellen of Slack socket-/HTTP-modus debuggen
title: "Slack"
---

# Slack

Status: production-ready voor DM's + kanalen via Slack-appintegraties. HTTP-modus (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DM's staan standaard in koppelmodus.
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">Native commandogedrag en commandocatalogus.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Diagnostiek over meerdere kanalen en herstelplaybooks.
</Card>
</CardGroup>

## Snelle installatie (beginner)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">In de Slack-appinstellingen:

        ```
        **Socket Mode** → schakel in. Ga daarna naar **Basic Information** → **App-Level Tokens** → **Generate Token and Scopes** met scope `connections:write`. Kopieer de **App Token** (`xapp-...`).
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
        Env fallback (alleen standaardaccount):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

      
</Step>
      
        <Step title="Abonneer je op app-events">
          Abonneer bot-events voor:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Schakel ook het App Home **Berichten-tabblad** in voor DM's.
        
</Step>
      
        <Step title="Start gateway">

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
        Gebruik HTTP-webhookmodus wanneer je Gateway via HTTPS door Slack bereikbaar is (typisch voor serverdeployments). HTTP-modus gebruikt de Events API + Interactivity + Slash Commands met één gedeelde request-URL.
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

      HTTP-modus met meerdere accounts: stel `channels.slack.accounts.<id> .mode = "http"` in en geef een unieke
      `webhookPath` per account zodat elke Slack-app naar zijn eigen URL kan wijzen.

  
</Tab>
</Tabs>

## Tokenmodel

- `botToken` + `appToken` zijn vereist voor Socket Mode.
- HTTP-modus vereist `botToken` + `signingSecret`.
- Config-tokens overschrijven de env fallback.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback is alleen van toepassing op het standaardaccount.
- Voorbeeld met userTokenReadOnly expliciet ingesteld (user-token-schrijven toestaan):
- Optioneel: voeg `chat:write.customize` toe als je wilt dat uitgaande berichten de actieve agentidentiteit gebruiken (aangepaste `username` en pictogram). `icon_emoji` gebruikt de syntaxis `:emoji_name:`.

<Tip>
Voor acties/directory-lezingen kan een user token de voorkeur hebben wanneer geconfigureerd. Zelfs met `userTokenReadOnly: false`
blijft het bot-token de voorkeur houden voor schrijven wanneer het beschikbaar is.
</Tip>

## Toegangsbeheer en routering

<Tabs>
  <Tab title="DM policy">Om iedereen toe te staan: stel `channels.slack.dm.policy="open"` en `channels.slack.dm.allowFrom=["*"]` in.

    ```
    - `pairing` (standaard)
    - `allowlist`
    - `open` (vereist dat `channels.slack.allowFrom` "*" bevat; legacy: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM-vlaggen:
    
    - `dm.enabled` (standaard true)
    - `channels.slack.allowFrom` (aanbevolen)
    - `dm.allowFrom` (legacy)
    - `dm.groupEnabled` (groeps-DM's standaard false)
    - `dm.groupChannels` (optionele MPIM-allowlist)
    
    Koppelen in DM's gebruikt `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` regelt kanaalafhandeling (`open|disabled|allowlist`).

    ```
    Verbonden maar geen kanaalantwoorden: kanaal geblokkeerd door `groupPolicy` of niet in de `channels.slack.channels`-allowlist.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">Kanaalberichten zijn standaard mention-beperkt.

    ```
    Mention-gating wordt geregeld via `channels.slack.channels` (stel `requireMention` in op `true`); `agents.list[].groupChat.mentionPatterns` (of `messages.groupChat.mentionPatterns`) tellen ook als mentions.
    ```

  
</Tab>
</Tabs>

## Commando's en slash-gedrag

- Native staat standaard uit voor Slack tenzij je `channels.slack.commands.native: true` instelt (globaal `commands.native` is `"auto"` waardoor Slack uit blijft).
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
- Als je native commands inschakelt, voeg één slash command per ingebouwde opdracht toe (dezelfde namen als `/help`).
- Als je native commands inschakelt, voeg één `slash_commands`-entry toe per commando dat je wilt blootstellen (overeenkomend met de lijst `/help`). Overschrijf met `channels.slack.commands.native`.

Standaardinstellingen voor slash-commando:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash-sessies gebruiken geïsoleerde sleutels:

- Slash commands gebruiken `agent:<agentId>:slack:slash:<userId>`-sessies (prefix configureerbaar via `channels.slack.slashCommand.sessionPrefix`).

en routeren de commando-uitvoering nog steeds naar de doelsessiesleutel van het gesprek (`CommandTargetSessionKey`).

## Threading, sessies en reply-tags

- DM's worden gerouteerd als `direct`; kanalen als `channel`; MPIM's als `group`.
- Met de standaard `session.dmScope=main` worden Slack-DM's samengevoegd tot de hoofdsessie van de agent.
- Kanalen mappen naar `agent:<agentId>:slack:channel:<channelId>`-sessies.
- Thread-antwoorden kunnen thread-sessiesuffixen (`:thread:<threadTs>`) aanmaken indien van toepassing.
- `channels.slack.thread.historyScope` is standaard `thread`; `thread.inheritParent` is standaard `false`.
- `channels.slack.thread.initialHistoryLimit` bepaalt hoeveel bestaande thread-berichten worden opgehaald wanneer een nieuwe thread-sessie start (standaard `20`; stel in op `0` om uit te schakelen).

Antwoord-threading

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
- legacy fallback voor directe chats: `channels.slack.dm.replyToMode`

Handmatige reply-tags worden ondersteund:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — antwoord op een specifiek bericht-id.

Opmerking: `replyToMode="off"` schakelt impliciete reply-threading uit. Expliciete `[[reply_to_*]]`-tags worden nog steeds gerespecteerd.

## Media, chunking en levering

<AccordionGroup>
  <Accordion title="Inbound attachments">Slack-bestandsbijlagen worden gedownload vanaf door Slack gehoste privé-URL’s (token-geauthenticeerde requestflow) en opgeslagen in de mediastore wanneer het ophalen slaagt en de groottelimieten dit toestaan.

    ```
    Media-uploads zijn beperkt door `channels.slack.mediaMaxMb` (standaard 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">- tekstblokken gebruiken `channels.slack.textChunkLimit` (standaard 4000)
- `channels.slack.chunkMode="newline"` schakelt splitsen op basis van eerst paragrafen in
- bestandsverzendingen gebruiken Slack-upload-API’s en kunnen threadantwoorden bevatten (`thread_ts`)
- de uitgaande medialimiet volgt `channels.slack.mediaMaxMb` wanneer geconfigureerd; anders gebruiken kanaalverzendingen de MIME-type-standaarden uit de mediapipeline
</Accordion>

  <Accordion title="Delivery targets">Voorkeur voor expliciete doelen:

    ```
    - `user:<id>` voor DM’s
    - `channel:<id>` voor kanalen
    
    Slack-DM’s worden geopend via Slack conversation-API’s bij het verzenden naar gebruikersdoelen.
    ```

  
</Accordion>
</AccordionGroup>

## Acties en gates

Slack-toolacties kunnen worden begrensd met `channels.slack.actions.*`:

Beschikbare actiegroepen in de huidige Slack-tooling:

| Actiegroep | Standaard    |
| ---------- | ------------ |
| messages   | ingeschakeld |
| reactions  | ingeschakeld |
| pins       | ingeschakeld |
| memberInfo | ingeschakeld |
| emojiList  | ingeschakeld |

## Gebeurtenissen en operationeel gedrag

- Berichtbewerkingen/-verwijderingen en thread-broadcasts worden omgezet in systeemgebeurtenissen.
- Reaction add/remove-gebeurtenissen worden omgezet in systeemgebeurtenissen.
- Lid geworden/verlaten, kanaal aangemaakt/hernoemd en pin toevoegen/verwijderen-gebeurtenissen worden omgezet in systeemgebeurtenissen.
- `channel_id_changed` kan kanaalconfiguratiesleutels migreren wanneer `configWrites` is ingeschakeld.
- Kanaaltopic-/doelmetadata wordt behandeld als niet-vertrouwde context en kan in de routeringscontext worden geïnjecteerd.

## `reactions:read`

`ackReaction` verstuurt een bevestigings-emoji terwijl OpenClaw een binnenkomend bericht verwerkt.

Volgorde van afhandeling:

- `of`channels.slack.channels.<name>.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- fallback naar agentidentiteit-emoji (`agents.list[].identity.emoji`, anders "👀")

Notities

- Slack verwacht shortcodes (bijvoorbeeld `"eyes"`).
- Gebruik `""` om de reactie voor een kanaal of account uit te schakelen.

## Manifest- en scopechecklist

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

  <Accordion title="Optional user-token scopes (read operations)">Als je `channels.slack.userToken` configureert, zijn typische lees-scopes:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Problemen oplossen

<AccordionGroup>
  <Accordion title="No replies in channels">Controleer, in volgorde:

    ```
    `users`: optionele per-kanaal gebruikers-allowlist.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">Controleer:

    ```
    DM’s genegeerd: afzender niet goedgekeurd wanneer `channels.slack.dm.policy="pairing"`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">Maak een Slack-app en schakel **Socket Mode** in.
</Accordion>

  <Accordion title="HTTP mode not receiving events">Valideer:

    ```
    - signing secret
    - webhookpad
    - Slack Request-URL’s (Events + Interactivity + Slash Commands)
    - uniek `webhookPath` per HTTP-account
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">Controleer of je het volgende bedoelde:

    ```
    Registratie van native commands gebruikt `commands.native` (globale standaard `"auto"` → Slack uit) en kan per werkruimte worden overschreven met `channels.slack.commands.native`. Tekstcommando’s vereisen losse `/...`-berichten en kunnen worden uitgeschakeld met `commands.text: false`. Slack slash commands worden in de Slack-app beheerd en niet automatisch verwijderd. Gebruik `commands.useAccessGroups: false` om access-group-controles voor commando’s te omzeilen.
    ```

  
</Accordion>
</AccordionGroup>

## Verwijzingen naar configuratiereferentie

Primaire referentie:

- `users:read` (gebruikersopzoeking)
  [https://docs.slack.dev/reference/methods/users.info](https://docs.slack.dev/reference/methods/users.info)

  Belangrijke Slack-velden:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM-toegang: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - Om **geen kanalen** toe te staan, stel `channels.slack.groupPolicy: "disabled"` in (of houd een lege allowlist aan).
  - threading/geschiedenis: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - levering: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ops/functies: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Gerelateerd

- [Pairing](/channels/pairing)
- `channel`: standaardkanalen (openbaar/privé)
- Voor triageflow: [/channels/troubleshooting](/channels/troubleshooting).
- [Configuration](/gateway/configuration)
- Volledige commandolijst + config: [Slash commands](/tools/slash-commands)

