---
summary: "Slack-konfiguration för socket- eller HTTP-webhook-läge"
read_when:
  - Konfigurera Slack eller felsöka Slack socket-/HTTP-läge
title: "Slack"
---

# Slack

Status: produktionsklar för DM + kanaler via Slack-appintegrationer. Standardläget är Socket Mode; HTTP Events API-läge stöds också.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack-DM använder parkopplingsläge som standard.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Inbyggt kommandobeteende och kommandokatalog.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostik över flera kanaler och reparationsguider.
  
</Card>
</CardGroup>

## Snabbstart

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        I Slack-appens inställningar:

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
            Miljövariabel som reserv (endast standardkonto):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="Prenumerera på apphändelser">
          Prenumerera på bothändelser för:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Aktivera även App Home **Messages Tab** för DM.
        
</Step>
      
        <Step title="Starta gateway">

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
      
        <Step title="Använd unika webhook-sökvägar för multi-account HTTP">
          HTTP-läge per konto stöds.
      
          Ge varje konto en unik `webhookPath` så att registreringar inte kolliderar.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Tokenmodell

- `botToken` + `appToken` krävs för Socket Mode.
- HTTP-läge kräver `botToken` + `signingSecret`.
- Konfigurationstokens åsidosätter miljövariabler som reserv.
- Miljövariablerna `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` gäller endast för standardkontot.
- `userToken` (`xoxp-...`) är endast konfigurationsbaserad (ingen reserv via miljövariabel) och använder som standard skrivskyddat läge (`userTokenReadOnly: true`).
- Valfritt: lägg till `chat:write.customize` om du vill att utgående meddelanden ska använda den aktiva agentidentiteten (anpassat `username` och ikon). `icon_emoji` använder syntaxen `:emoji_name:`.

<Tip>
För åtgärder/katalogläsningar kan user-token prioriteras när den är konfigurerad. För skrivningar är bot-token fortsatt prioriterad; skrivningar med user-token tillåts endast när `userTokenReadOnly: false` och bot-token saknas.
</Tip>

## Åtkomstkontroll och routing

<Tabs>
  <Tab title="DM policy">
    `channels.slack.dmPolicy` styr DM-åtkomst (äldre: `channels.slack.dm.policy`):

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kräver att `channels.slack.allowFrom` innehåller `"*"`; äldre: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM-flaggor:
    
    - `dm.enabled` (standard true)
    - `channels.slack.allowFrom` (rekommenderas)
    - `dm.allowFrom` (äldre)
    - `dm.groupEnabled` (grupp-DM är som standard false)
    - `dm.groupChannels` (valfri MPIM-tillåtslista)
    
    Parkoppling i DM använder `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">
    `channels.slack.groupPolicy` styr kanalhantering:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Kanalens tillåtslista finns under `channels.slack.channels`.
    
    Körningsanteckning: om `channels.slack` helt saknas (endast miljövariabler) och `channels.defaults.groupPolicy` inte är satt, faller körningen tillbaka till `groupPolicy="open"` och loggar en varning.
    
    Namn-/ID-upplösning:
    
    - poster i kanalens och DM:ens tillåtslistor löses vid uppstart när token-åtkomst tillåter
    - olösta poster behålls som konfigurerade
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Kanalmeddelanden kräver omnämnande som standard.

    ```
    Källor för omnämnande:
    
    - explicit app-omnämnande (`<@botId>`)
    - regex-mönster för omnämnande (`agents.list[].groupChat.mentionPatterns`, reserv `messages.groupChat.mentionPatterns`)
    - implicit svar-i-tråd-beteende till bot
    
    Kontroller per kanal (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (tillåtslista)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## OpenClaw-konfig (minimal)

- Inbyggt kommando auto-läge är **av** för Slack (`commands.native: "auto"` aktiverar inte Slack-inbyggda kommandon).
- Aktivera inbyggda Slack-kommandohanterare med `channels.slack.commands.native: true` (eller globalt `commands.native: true`).
- När inbyggda kommandon är aktiverade, registrera motsvarande slash-kommandon i Slack (`/<command>`-namn).
- Om inbyggda kommandon inte är aktiverade kan du köra ett enskilt konfigurerat slash-kommando via `channels.slack.slashCommand`.

Standardinställningar för slash-kommando:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Snedstreckssessioner använder isolerade nycklar:

- `agent:<agentId>:slack:slash:<userId>`

och dirigerar fortfarande kommandokörning mot målkonversationssessionen (`CommandTargetSessionKey`).

## Scopes (aktuella vs valfria)

- DMs dirigeras som `direct`; kanaler som `channel`; MPIMs som `group`.
- Med standardinställningen `session.dmScope=main` slås Slack-DMs ihop till agentens huvudsession.
- Kanalsessioner: `agent:<agentId>:slack:channel:<channelId>`.
- Trådsvar kan skapa trådsessionssuffix (`:thread:<threadTs>`) när det är tillämpligt.
- `channels.slack.thread.historyScope` är som standard `thread`; `thread.inheritParent` är som standard `false`.
- `channels.slack.thread.initialHistoryLimit` styr hur många befintliga trådmeddelanden som hämtas när en ny trådsession startar (standard `20`; sätt `0` för att inaktivera).

Inställningar för svarstrådning:

- `chat:write` (skicka/uppdatera/ta bort meddelanden via `chat.postMessage`)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (öppna DM via `conversations.open` för användar-DM)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Manuella svarstaggar stöds:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Obs: `replyToMode="off"` inaktiverar implicit svarstrådning. Explicita `[[reply_to_*]]`-taggar respekteras fortfarande.

## Inte behövda idag (men sannolikt i framtiden)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack-filbilagor laddas ner från Slack-hostade privata URL:er (tokenautentiserat begärdeflöde) och skrivs till medielagret när hämtningen lyckas och storleksgränser tillåter.


    ```
    Gränsen för inkommande storlek vid körning är som standard `20MB` om den inte åsidosätts av `channels.slack.mediaMaxMb`.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - textblock använder `channels.slack.textChunkLimit` (standard 4000)
    - `channels.slack.chunkMode="newline"` aktiverar styckevis uppdelning först
    - filsändningar använder Slack upload APIs och kan inkludera trådsvar (`thread_ts`)
    - utgående mediagräns följer `channels.slack.mediaMaxMb` när den är konfigurerad; annars använder kanalsändningar MIME-typsstandarder från mediapipelinen
  
</Accordion>

  <Accordion title="Delivery targets">
    Föredragna explicita mål:

    ```
    - `user:<id>` för DMs
    - `channel:<id>` för kanaler
    
    Slack-DMs öppnas via Slack conversation APIs när meddelanden skickas till användarmål.
    ```

  
</Accordion>
</AccordionGroup>

## Begränsningar

Slack-åtgärder styrs av `channels.slack.actions.*`.

Tillgängliga åtgärdsgrupper i nuvarande Slack-verktyg:

| Grupp      | Standard |
| ---------- | -------- |
| messages   | enabled  |
| reactions  | enabled  |
| pins       | enabled  |
| memberInfo | enabled  |
| emojiList  | enabled  |

## Händelser och operativt beteende

- Redigeringar/raderingar av meddelanden och trådsändningar mappas till systemhändelser.
- Händelser för att lägga till/ta bort reaktioner mappas till systemhändelser.
- Händelser för medlem som går med/lämnar, kanal som skapas/byter namn samt nålning som läggs till/tas bort mappas till systemhändelser.
- `channel_id_changed` kan migrera kanalens konfigurationsnycklar när `configWrites` är aktiverat.
- Kanalens ämnes-/syftesmetadata behandlas som opålitlig kontext och kan injiceras i routningskontexten.

## Trådning per chatttyp

Du kan konfigurera olika trådningsbeteenden per chatttyp genom att sätta `channels.slack.replyToModeByChatType`:

Upplösningsordning:

- `channels.slack.accounts.<accountId>`.ackReaction
- `channels.slack.ackReaction`
- `messages.ackReaction`
- reserv-emoji för agentidentitet (`agents.list[].identity.emoji`, annars "👀")

Obs:

- Slack förväntar sig shortcodes (till exempel `"eyes"`).
- Använd `""` för att inaktivera reaktionen för en kanal eller ett konto.

## Manifest och omfattningschecklista

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
    Om du konfigurerar `channels.slack.userToken` är typiska läsbehörigheter:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (om du är beroende av Slack-sökning för läsning)
    ```

  
</Accordion>
</AccordionGroup>

## Felsökning

<AccordionGroup>
  <Accordion title="No replies in channels">
    Kontrollera, i ordning:

    ```
    - `groupPolicy`
    - kanalens allowlist (`channels.slack.channels`)
    - `requireMention`
    - per-kanal `users` allowlist
    
    Användbara kommandon:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Kontrollera:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (eller äldre `channels.slack.dm.policy`)
    - parkopplingsgodkännanden / allowlist-poster
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Verifiera bot- och app-token samt att Socket Mode är aktiverat i Slack-appens inställningar.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Verifiera:

    ```
    - signing secret
    - webhook-sökväg
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - unik `webhookPath` per HTTP-konto
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Verifiera om du avsåg:

    ```
    - inbyggt kommandoläge (`channels.slack.commands.native: true`) med matchande slash commands registrerade i Slack
    - eller läge med ett enda slash-kommando (`channels.slack.slashCommand.enabled: true`)
    
    Kontrollera även `commands.useAccessGroups` och kanal-/användar-allowlists.
    ```

  
</Accordion>
</AccordionGroup>

## Verktygsåtgärder

Slack-verktygsåtgärder kan spärras med `channels.slack.actions.*`:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Viktiga Slack-fält:

  - läge/autentisering: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM-åtkomst: `dm.enabled`, `dmPolicy`, `allowFrom` (äldre: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - kanalåtkomst: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - trådar/historik: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - leverans: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - drift/funktioner: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Säkerhetsnoteringar

- Skrivningar använder som standard bot-token så att tillståndsändrande åtgärder hålls inom
  appens botbehörigheter och identitet.
- [Channel routing](/channels/channel-routing)
- Om du aktiverar skrivningar med användartoken, säkerställ att användartoken inkluderar de skriv-
  scopes du förväntar dig (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) annars kommer dessa operationer att misslyckas.
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)

