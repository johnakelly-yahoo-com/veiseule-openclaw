---
summary: "Slack-opsætning til socket- eller HTTP-webhook-tilstand"
read_when:
  - Opsætning af Slack eller fejlfinding af Slack socket-/HTTP-tilstand
title: "Slack"
---

# Slack

Status: produktionsklar til DMs + kanaler via Slack app-integrationer. Standardtilstand er Socket Mode; HTTP Events API-tilstand understøttes også.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DMs bruger som standard pairing-tilstand.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Indbygget kommandoadfærd og kommandokatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnose og reparations-playbooks på tværs af kanaler.
  
</Card>
</CardGroup>

## Hurtig opsætning

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        I Slack app-indstillinger:


        ```
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

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Miljø-tilbagefald (kun standardkonto):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="Abonnér på app-begivenheder">
          Abonnér på bot-begivenheder for:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Aktivér også App Home **Messages Tab** for DMs.
        
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

        
</Step>
      
        <Step title="Brug unikke webhook-stier til multi-konto HTTP">
          HTTP-tilstand pr. konto understøttes.
      
          Giv hver konto en særskilt `webhookPath`, så registreringer ikke kolliderer.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Tokenmodel

- `botToken` + `appToken` er påkrævet for Socket Mode.
- HTTP-tilstand kræver `botToken` + `signingSecret`.
- Konfigurationstokens tilsidesætter miljø-tilbagefald.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` miljø-tilbagefald gælder kun for standardkontoen.
- `userToken` (`xoxp-...`) er kun konfiguration (intet miljø-tilbagefald) og har som standard skrivebeskyttet adfærd (`userTokenReadOnly: true`).
- Valgfrit: tilføj `chat:write.customize`, hvis du vil have, at udgående beskeder bruger den aktive agentidentitet (tilpasset `username` og ikon). `icon_emoji` bruger syntaksen `:emoji_name:`.

<Tip>
Til handlinger/mappe-læsninger kan user token foretrækkes, når det er konfigureret. Til skrivninger foretrækkes bot token fortsat; skrivninger med user token er kun tilladt, når `userTokenReadOnly: false`, og bot token ikke er tilgængelig.
</Tip>

## Adgangskontrol og routing

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` styrer DM-adgang (legacy: `channels.slack.dm.policy`):


    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kræver, at `channels.slack.allowFrom` inkluderer `"*"`; legacy: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM-flag:
    
    - `dm.enabled` (standard true)
    - `channels.slack.allowFrom` (foretrukken)
    - `dm.allowFrom` (legacy)
    - `dm.groupEnabled` (gruppe-DMs er som standard false)
    - `dm.groupChannels` (valgfri MPIM-allowlist)
    
    Pairing i DMs bruger `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` styrer kanalhåndtering:


    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Kanal-allowlist findes under `channels.slack.channels`.
    
    Runtime-bemærkning: hvis `channels.slack` mangler helt (kun miljø-opsætning) og `channels.defaults.groupPolicy` ikke er sat, falder runtime tilbage til `groupPolicy="open"` og logger en advarsel.
    
    Navne-/ID-opløsning:
    
    - kanal-allowlist-poster og DM-allowlist-poster opløses ved opstart, når token-adgang tillader det
    - ikke-opløste poster bevares som konfigureret
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Kanalbeskeder er som standard mention-gated.


    ```
    Mention-kilder:
    
    - eksplicit app-mention (`<@botId>`)
    - mention regex-mønstre (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit svar-til-bot tråd-adfærd
    
    Kontroller pr. kanal (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## OpenClaw-konfiguration (minimal)

- Native command auto-tilstand er **deaktiveret** for Slack (`commands.native: "auto"` aktiverer ikke Slack native commands).
- Aktivér native Slack-kommandohåndteringer med `channels.slack.commands.native: true` (eller globalt `commands.native: true`).
- Når native commands er aktiveret, skal tilsvarende slash commands registreres i Slack (`/<command>`-navne).
- Hvis native commands ikke er aktiveret, kan du køre en enkelt konfigureret slash command via `channels.slack.slashCommand`.

Standardindstillinger for slash command:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash-sessioner bruger isolerede nøgler:

- `agent:<agentId>:slack:slash:<userId>`

og dirigerer stadig kommandoafvikling mod målkonversationssessionen (`CommandTargetSessionKey`).

## Scopes (aktuelle vs. valgfri)

- DMs dirigeres som `direct`; kanaler som `channel`; MPIMs som `group`.
- Med standardindstillingen `session.dmScope=main` samles Slack-DMs i agentens hovedsession.
- Kanalsessioner: `agent:<agentId>:slack:channel:<channelId>`.
- Trådsvar kan oprette trådsessionsuffikser (`:thread:<threadTs>`) når relevant.
- `channels.slack.thread.historyScope` har som standard værdien `thread`; `thread.inheritParent` har som standard værdien `false`.
- `channels.slack.thread.initialHistoryLimit` styrer, hvor mange eksisterende trådbeskeder der hentes, når en ny trådsession starter (standard `20`; sæt til `0` for at deaktivere).

Styring af svar i tråde:

- `chat:write` (send/opdatér/slet beskeder via `chat.postMessage`)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (åbn DMs via `conversations.open` for bruger-DMs)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Manuelle svar-tags understøttes:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Bemærk: `replyToMode="off"` deaktiverer implicit svartrådning. Eksplicitte `[[reply_to_*]]`-tags respekteres stadig.

## Ikke nødvendige i dag (men sandsynligvis fremtid)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack-filvedhæftninger downloades fra Slack-hostede private URL’er (token-autentificeret forespørgselsflow) og skrives til medielageret, når hentning lykkes og størrelsesgrænser tillader det.

    ```
    Grænsen for indgående størrelse under kørsel er som standard `20MB`, medmindre den tilsidesættes af `channels.slack.mediaMaxMb`.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - tekstblokke bruger `channels.slack.textChunkLimit` (standard 4000)
    - `channels.slack.chunkMode="newline"` aktiverer opdeling med afsnit først
    - filafsendelser bruger Slack upload-API’er og kan inkludere trådsvar (`thread_ts`)
    - udgående mediegrænse følger `channels.slack.mediaMaxMb`, når konfigureret; ellers bruger kanalafsendelser MIME-type-standarder fra mediepipelinen
  
</Accordion>

  <Accordion title="Delivery targets">
    Foretrukne eksplicitte mål:

    ```
    - `user:<id>` for DMs
    - `channel:<id>` for kanaler
    
    Slack-DMs åbnes via Slacks samtale-API’er ved afsendelse til brugermål.
    ```

  
</Accordion>
</AccordionGroup>

## Grænser

Slack-handlinger styres af `channels.slack.actions.*`.

Tilgængelige handlingsgrupper i den nuværende Slack-tooling:

| Gruppe     | Standard |
| ---------- | -------- |
| messages   | enabled  |
| reactions  | enabled  |
| pins       | enabled  |
| memberInfo | enabled  |
| emojiList  | enabled  |

## Hændelser og operationel adfærd

- Redigering/sletning af beskeder samt trådbroadcasts mappes til systemhændelser.
- Hændelser for tilføjelse/fjernelse af reaktioner mappes til systemhændelser.
- Hændelser for medlem tilslutter/forlader, kanal oprettet/omdøbt samt tilføjelse/fjernelse af fastgørelse mappes til systemhændelser.
- `channel_id_changed` kan migrere kanalens konfigurationsnøgler, når `configWrites` er aktiveret.
- Kanalens emne-/formålsmetadata behandles som ikke-betroet kontekst og kan injiceres i routingskonteksten.

## Trådning pr. chat-type

Du kan konfigurere forskellig trådningsadfærd pr. chat-type ved at sætte `channels.slack.replyToModeByChatType`:

Opløsningsrækkefølge:

- `channels.slack.accounts.<accountId>``.ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- fallback til agentidentitets-emoji (`agents.list[].identity.emoji`, ellers "👀")

Bemærkninger:

- Slack forventer shortcodes (for eksempel `"eyes"`).
- Brug `""` for at deaktivere reaktionen for en kanal eller konto.

## Manifest- og scope-tjekliste

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
        "im:history",
        "mpim:history",
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

  <Accordion title="Optional user-token scopes (read operations)">
    Hvis du konfigurerer `channels.slack.userToken`, er typiske læse-scopes:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (hvis du er afhængig af Slack-søgning)
    ```

  
</Accordion>
</AccordionGroup>

## Fejlfinding

<AccordionGroup>
  <Accordion title="No replies in channels">
    Tjek, i rækkefølge:

    ```
    - `groupPolicy`
    - kanal-allowlist (`channels.slack.channels`)
    - `requireMention`
    - pr.-kanal `users` allowlist
    
    Nyttige kommandoer:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Tjek:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (eller legacy `channels.slack.dm.policy`)
    - parringsgodkendelser / allowlist-poster
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Validér bot- og app-tokens samt at Socket Mode er aktiveret i Slack-appens indstillinger.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Validér:

    ```
    - signing secret
    - webhook-sti
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - unik `webhookPath` pr. HTTP-konto
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Bekræft, om du havde til hensigt:

    ```
    - native command-tilstand (`channels.slack.commands.native: true`) med matchende slash commands registreret i Slack
    - eller single slash command-tilstand (`channels.slack.slashCommand.enabled: true`)
    
    Tjek også `commands.useAccessGroups` og kanal-/bruger-allowlists.
    ```

  
</Accordion>
</AccordionGroup>

## Værktøjshandlinger

Slack-værktøjshandlinger kan gates med `channels.slack.actions.*`:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Slack-felter med høj relevans:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM-adgang: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - kanaladgang: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - tråde/historik: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - levering: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - drift/funktioner: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Sikkerhedsnoter

- Skrivehandlinger bruger som standard bot token, så tilstandsændrende handlinger forbliver afgrænset til
  appens bot-tilladelser og identitet.
- [Channel routing](/channels/channel-routing)
- Hvis du aktiverer skrivninger med user token, skal du sikre, at user token inkluderer de forventede skrive-scopes
  (`chat:write`, `reactions:write`, `pins:write`, `files:write`), ellers vil disse handlinger fejle.
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)
