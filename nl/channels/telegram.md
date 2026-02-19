---
summary: "Ondersteuningsstatus, mogelijkheden en configuratie van Telegram-bots"
read_when:
  - Werken aan Telegram-functies of webhooks
title: "Telegram"
---

# Telegram (Bot API)

Status: productierijp voor bot-DM’s + groepen via grammY. Long-polling standaard; webhook optioneel.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Pairing is de standaard tokenuitwisseling voor Telegram-DM’s.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">Cross-channel diagnostiek en herstelplaybooks.
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">Volledige kanaalconfiguratiepatronen en voorbeelden.
</Card>
</CardGroup>

## Snelle installatie (beginner)

<Steps>
  <Step title="Create the bot token in BotFather">Open Telegram en chat met **@BotFather** ([directe link](https://t.me/BotFather)). Bevestig dat de handle exact `@BotFather` is.

    ```
    `/newbot` maakt de bot aan en retourneert de token (houd deze geheim).
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
    Env-optie: `TELEGRAM_BOT_TOKEN=...` (werkt voor het standaardaccount).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Onbekende afzenders ontvangen een pairingcode; berichten worden genegeerd tot goedkeuring (codes verlopen na 1 uur).
    ```

  
</Step>

  <Step title="Add the bot to a group">Voeg de bot toe aan je groep en stel vervolgens `channels.telegram.groups` en `groupPolicy` in volgens je toegangsmodel.
</Step>
</Steps>

<Note>
De tokenresolutievolgorde is accountbewust. In de praktijk hebben configuratiewaarden voorrang op env-fallback, en `TELEGRAM_BOT_TOKEN` geldt alleen voor het standaardaccount.
</Note>

## Telegram-instellingen

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Als je `channels.telegram.groups.*.requireMention=false` hebt ingesteld, moet Telegram’s Bot API **privacy mode** zijn uitgeschakeld.

    ```
    **Let op:** Wanneer je privacy mode wijzigt, vereist Telegram dat je de bot
    uit elke groep verwijdert en opnieuw toevoegt voordat de wijziging van kracht wordt.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Adminstatus wordt binnen de groep ingesteld (Telegram-UI).

    ```
    Voeg de bot toe als **admin** van de groep (admin-bots ontvangen alle berichten).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — toevoegen van de bot aan groepen toestaan/weigeren.
    ```

  
</Accordion>
</AccordionGroup>

## Toegangscontrole en activering

<Tabs>
  <Tab title="DM policy">`channels.telegram.dmPolicy` regelt toegang tot directe berichten:

    ```
    - `pairing` (standaard)
    - `allowlist`
    - `open` (vereist dat `allowFrom` "*" bevat)
    - `disabled`
    
    `channels.telegram.allowFrom` accepteert numerieke Telegram-gebruikers-ID's. Voorvoegsels `telegram:` / `tg:` worden geaccepteerd en genormaliseerd.
    De onboardingwizard accepteert invoer in de vorm van `@gebruikersnaam` en zet dit om naar numerieke ID's.
    Als je hebt geüpgraded en je configuratie bevat `@gebruikersnaam`-vermeldingen in de allowlist, voer dan `openclaw doctor --fix` uit om deze om te zetten (best-effort; vereist een Telegram-bottoken).
    
    ### Je Telegram-gebruikers-ID vinden
    
    Veiliger (geen bot van derden):
    
    1. Stuur een DM naar je bot.
    2. Voer `openclaw logs --follow` uit.
    3. Lees `from.id`.
    
    Officiële Bot API-methode:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Derdenmethode (minder privé): `@userinfobot` of `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Twee onafhankelijke controles:

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

  <Tab title="Mention behavior">Groepsantwoorden vereisen standaard een vermelding.

    ```
    De vermelding kan komen van:
    
    - een native `@botgebruikersnaam`-vermelding, of
    - vermeldingpatronen in:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Schakelaars voor opdrachten op sessieniveau:
    
    - `/activation always`
    - `/activation mention`
    
    Deze werken alleen de sessiestatus bij. Gebruik configuratie voor persistentie.
    
    Voorbeeld van persistente configuratie:
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
    Stuur een bericht uit de groep door naar `@userinfobot` of `@getidsbot` op Telegram om het chat-ID te zien (negatief nummer zoals `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Runtimegedrag

- Een Telegram Bot API-kanaal dat eigendom is van de Gateway.
- Deterministische routering: antwoorden gaan terug naar Telegram; het model kiest nooit kanalen.
- Inkomende berichten worden genormaliseerd naar de gedeelde kanaalenvelop met antwoordcontext en mediaplaatsaanduidingen.
- Het groepschat-ID verkrijgen Voegt `:topic:<threadId>` toe aan de Telegram-groepssessiesleutel zodat elk topic geïsoleerd is.
- Stuurt typindicatoren en antwoorden met `message_thread_id` zodat reacties in het topic blijven.
- Long polling gebruikt grammY runner met sequencing per chat/per thread. Long-polling gebruikt de grammY-runner met per-chat-sequencing; de totale gelijktijdigheid wordt begrensd door `agents.defaults.maxConcurrent`.
- De Telegram Bot API ondersteunt geen leesbevestigingen; er is geen `sendReadReceipts`-optie.

## Functiereferentie

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw kan gedeeltelijke antwoorden streamen in Telegram-DM’s met `sendMessageDraft`.

    ```
    Vereiste:
    
    - `channels.telegram.streamMode` is niet `"off"` (standaard: `"partial"`)
    
    Modi:
    
    - `off`: geen live preview
    - `partial`: frequente preview-updates op basis van gedeeltelijke tekst
    - `block`: preview-updates in blokken met `channels.telegram.draftChunk`
    
    `draftChunk`-standaarden voor `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` wordt begrensd door `channels.telegram.textChunkLimit`.
    
    Dit werkt in directe chats en groepen/topics.
    
    Voor alleen-tekstantwoorden behoudt OpenClaw hetzelfde previewbericht en voert het een laatste bewerking ter plaatse uit (geen tweede bericht).
    
    Voor complexe antwoorden (bijvoorbeeld media-payloads) valt OpenClaw terug op normale eindlevering en ruimt daarna het previewbericht op.
    
    `streamMode` staat los van block streaming. Wanneer block streaming expliciet is ingeschakeld voor Telegram, slaat OpenClaw de previewstream over om dubbele streaming te voorkomen.
    
    Telegram-only reasoning stream:
    
    - `/reasoning stream` stuurt redenering naar de live preview tijdens het genereren
    - het uiteindelijke antwoord wordt zonder redeneringstekst verzonden
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Uitgaande Telegram-tekst gebruikt `parse_mode: "HTML"` (Telegram’s ondersteunde subset van tags).

    ```
    - Markdown-achtige tekst wordt gerenderd naar Telegram-veilige HTML.
    - Ruwe model-HTML wordt ge-escaped om Telegram-parsefouten te verminderen.
    - Als Telegram geparste HTML weigert, probeert OpenClaw opnieuw als platte tekst.
    
    Linkvoorbeelden zijn standaard ingeschakeld en kunnen worden uitgeschakeld met `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Registratie van het Telegram-opdrachtenmenu wordt bij het opstarten afgehandeld met `setMyCommands`.

    ```
    OpenClaw registreert native opdrachten (zoals `/status`, `/reset`, `/model`) bij het botmenu van Telegram bij het opstarten. Je kunt aangepaste opdrachten aan het menu toevoegen via config:
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
    Regels:
    
    - namen worden genormaliseerd (leidende `/` verwijderen, lowercase)
    - geldig patroon: `a-z`, `0-9`, `_`, lengte `1..32`
    - aangepaste opdrachten mogen native opdrachten niet overschrijven
    - conflicten/dubbelen worden overgeslagen en gelogd
    
    Opmerkingen:
    
    - aangepaste opdrachten zijn alleen menu-items; ze implementeren niet automatisch gedrag
    - plugin-/skillopdrachten kunnen nog steeds werken wanneer ze worden getypt, zelfs als ze niet in het Telegram-menu worden getoond
    
    Als native opdrachten zijn uitgeschakeld, worden ingebouwde opdrachten verwijderd. Aangepaste/pluginopdrachten kunnen nog steeds worden geregistreerd indien geconfigureerd.
    
    Veelvoorkomende installatiefout:
    
    - `setMyCommands failed` betekent meestal dat uitgaande DNS/HTTPS naar `api.telegram.org` is geblokkeerd.
    
    ### Device pairing-opdrachten (`device-pair` plugin)
    
    Wanneer de `device-pair` plugin is geïnstalleerd:
    
    1. `/pair` genereert een installatiecode
    2. plak de code in de iOS-app
    3. `/pair approve` keurt het meest recente openstaande verzoek goed
    
    Meer details: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">Configureer inline-toetsenbordscope:

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
    Voor per-accountconfiguratie:
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
    `"disabled"` = helemaal geen groepsberichten geaccepteerd
      Standaard is `groupPolicy: "allowlist"` (geblokkeerd tenzij je `groupAllowFrom` toevoegt).
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
    Wanneer een gebruiker op een knop klikt, wordt de callbackdata teruggestuurd naar de agent als een bericht met het formaat:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">Telegram-toolacties omvatten:

    ```
    - `sendMessage` (`to`, `content`, optioneel `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Kanaalberichtacties bieden gebruiksvriendelijke aliassen (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Toegangscontroles:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (standaard: uitgeschakeld)
    
    Semantiek voor het verwijderen van reacties: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram ondersteunt optionele gethreadde antwoorden via tags:

    ```
    - `[[reply_to_current]]` antwoordt op het activerende bericht
    - `[[reply_to:<id>]]` antwoordt op een specifieke Telegram-bericht-ID
    
    `channels.telegram.replyToMode` regelt de afhandeling:
    
    - `off` (standaard)
    - `first`
    - `all`
    
    Opmerking: `off` schakelt impliciete reply-threading uit. Expliciete tags `[[reply_to_*]]` worden nog steeds gerespecteerd.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Topics (forum-supergroepen)

    ```
    **Forumgroepen:** Reacties in forumgroepen bevatten `message_thread_id` en gebruiken sessiesleutels zoals `agent:main:telegram:group:{chatId}:topic:{threadId}`.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">### Audioberichten

    ```
    `[[audio_as_voice]]` — verstuur audio als spraaknotitie in plaats van als bestand.
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
    ### Videoberichten
    
    Telegram maakt onderscheid tussen videobestanden en videoberichten (video notes).
    
    Voorbeeld van berichtactie:
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
    Videoberichten (video notes) ondersteunen geen bijschriften; opgegeven berichttekst wordt apart verzonden.
    
    ### Stickers
    
    Afhandeling van inkomende stickers:
    
    - statische WEBP: gedownload en verwerkt (placeholder `<media:sticker>`)
    - geanimeerde TGS: overgeslagen
    - video WEBM: overgeslagen
    
    Stickercontextvelden:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Stickercachebestand:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Stickers worden (indien mogelijk) één keer beschreven en gecachet om herhaalde vision-calls te verminderen.
    
    Stickeracties inschakelen:
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
    Stickers verzenden
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
    Stickercache
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

  <Accordion title="Reaction notifications">Ontvangt de `message_reaction`-update van de Telegram API

    ```
    Wanneer ingeschakeld, plaatst OpenClaw systeemevenementen in de wachtrij zoals:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Configuratie:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (standaard: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (standaard: `minimal`)
    
    Opmerkingen:
    
    - `own` betekent alleen gebruikersreacties op door de bot verzonden berichten (best-effort via verzonden-berichtcache).
    - Telegram levert geen thread-ID's in reactie-updates.
      - niet-forumgroepen worden gerouteerd naar de groepschatsessie
      - forumgroepen worden gerouteerd naar de algemene groepsonderwerpsessie (`:topic:1`), niet naar het exacte oorspronkelijke onderwerp
    
    `allowed_updates` voor polling/webhook bevat automatisch `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` stuurt een bevestigingsemoji terwijl OpenClaw een inkomend bericht verwerkt.

    ```
    Resolutievolgorde:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - fallback naar agent-identiteitsemoji (`agents.list[].identity.emoji`, anders "👀")
    
    Opmerkingen:
    
    - Telegram verwacht unicode-emoji (bijvoorbeeld "👀").
    - Gebruik `""` om de reactie voor een kanaal of account uit te schakelen.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">Kanaalconfiguratieschrijfacties zijn standaard ingeschakeld (`configWrites !== false`).

    ```
    Door Telegram geactiveerde schrijfacties omvatten:
    
    - groepsmigratiegebeurtenissen (`migrate_to_chat_id`) om `channels.telegram.groups` bij te werken
    - `/config set` en `/config unset` (vereist dat de opdracht is ingeschakeld)
    
    Uitschakelen:
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">Standaard: long polling.

    ```
    Webhookmodus: stel `channels.telegram.webhookUrl` en `channels.telegram.webhookSecret` in (optioneel `channels.telegram.webhookPath`).
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Uitgaande tekst wordt gechunked tot `channels.telegram.textChunkLimit` (standaard 4000).
    Optionele nieuwe-regel-chunking: stel `channels.telegram.chunkMode="newline"` in om te splitsen op lege regels (paragraafgrenzen) vóór lengte-chunking.
    Media-downloads/uploads zijn begrensd door `channels.telegram.mediaMaxMb` (standaard 5).
    Telegram Bot API-verzoeken time-outen na `channels.telegram.timeoutSeconds` (standaard 500 via grammY).
    Groepsgeschiedeniscontext gebruikt `channels.telegram.historyLimit` (of `channels.telegram.accounts.*.historyLimit`), met terugval naar `messages.groupChat.historyLimit`. Stel `0` in om uit te schakelen (standaard 50).
    DM-geschiedenis kan worden beperkt met `channels.telegram.dmHistoryLimit` (gebruikersbeurten). Per-gebruiker overrides: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>"].historyLimit`
    - uitgaande Telegram API-retries zijn configureerbaar via `channels.telegram.retry`.

    ```
    CLI-verzenddoel kan een numerieke chat-ID of gebruikersnaam zijn:
    ```

```bash
Voorbeeld: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Problemen oplossen

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
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

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - wanneer `channels.telegram.groups` bestaat, moet de groep vermeld zijn (of "*" bevatten)
    - controleer of de bot lid is van de groep
    - bekijk logs: `openclaw logs --follow` voor redenen van overslaan
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    `setMyCommands failed` in logs betekent meestal dat uitgaande HTTPS/DNS is geblokkeerd naar `api.telegram.org`.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + aangepaste fetch/proxy kan onmiddellijk abortgedrag veroorzaken als AbortSignal-typen niet overeenkomen.
    - Sommige hosts lossen `api.telegram.org` eerst op naar IPv6; gebrekkige IPv6-egress kan intermitterende Telegram API-fouten veroorzaken.
    - Valideer DNS-antwoorden:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Meer hulp: [Kanaalproblemen oplossen](/channels/troubleshooting).

## Telegram-configuratie referentiepunten

Primaire referentie:

- `channels.telegram.enabled`: kanaalstart in-/uitschakelen.

- `channels.telegram.botToken`: bot-token (BotFather).

- `channels.telegram.tokenFile`: lees token uit bestandspad.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (standaard: pairing).

- `channels.telegram.allowFrom`: DM-allowlist (ID’s/gebruikersnamen). `open` vereist `"*"`. `openclaw doctor --fix` kan verouderde `@username`-vermeldingen omzetten naar ID's.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (standaard: allowlist).

- `channels.telegram.groupAllowFrom`: groepsafzender-allowlist (ID’s/gebruikersnamen). `openclaw doctor --fix` kan verouderde `@username`-vermeldingen omzetten naar ID's.

- `channels.telegram.groups`: per-groep-standaarden + allowlist (gebruik `"*"` voor globale standaarden).
  - `channels.telegram.groups.<id>.groupPolicy`: per-groep-override voor groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: standaard mention-gating.
  - `channels.telegram.groups.<id>.skills`: skillfilter (weglaten = alle skills, leeg = geen).
  - `channels.telegram.groups.<id>.allowFrom`: per-groep-override voor afzender-allowlist.
  - `channels.telegram.groups.<id>.systemPrompt`: extra systeemprompt voor de groep.
  - `channels.telegram.groups.<id>.enabled`: schakel de groep uit wanneer `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: per-topic-overrides (zelfde velden als groep).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: per-topic-override voor groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: per-topic-override voor mention-gating.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (standaard: allowlist).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account-override.

- `channels.telegram.replyToMode`: `off | first | all` (standaard: `first`).

- `channels.telegram.textChunkLimit`: uitgaande chunkgrootte (tekens).

- `channels.telegram.chunkMode`: `length` (standaard) of `newline` om te splitsen op lege regels (paragraafgrenzen) vóór lengte-chunking.

- `channels.telegram.linkPreview`: schakel linkvoorbeelden in/uit voor uitgaande berichten (standaard: true).

- `channels.telegram.streamMode`: `off | partial | block` (conceptstreaming).

- `channels.telegram.mediaMaxMb`: inkomende/uitgaande medialimiet (MB).

- `channels.telegram.retry`: retrybeleid voor uitgaande Telegram API-calls (pogingen, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=inschakelen, false=uitschakelen). Standaard uitgeschakeld op Node 22 om Happy Eyeballs-time-outs te vermijden.

- `channels.telegram.proxy`: proxy-URL voor Bot API-calls (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: webhookmodus inschakelen (vereist `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook-secret (vereist wanneer webhookUrl is ingesteld).

- `channels.telegram.webhookPath`: lokaal webhookpad (standaard `/telegram-webhook`).

- De lokale listener bindt aan `0.0.0.0:8787` en serveert standaard `POST /telegram-webhook`.

- `channels.telegram.actions.reactions`: gate Telegram-toolreacties.

- `channels.telegram.actions.sendMessage`: gate Telegram-toolberichtverzendingen.

- `channels.telegram.actions.deleteMessage`: gate Telegram-toolberichtverwijderingen.

- `channels.telegram.actions.sticker`: gate Telegram-stickeracties — verzenden en zoeken (standaard: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — bepaal welke reacties systeemevents triggeren (standaard: `own` indien niet ingesteld).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — bepaal de reactiemogelijkheid van de agent (standaard: `minimal` indien niet ingesteld).

- [Configuratiereferentie - Telegram](/gateway/configuration-reference#telegram)

Telegram-specifieke high-signal-velden:

- opstart/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- Opdrachten vereisen autorisatie, zelfs in groepen met `groupPolicy: "open"`
- commando/menu: `commands.native`, `customCommands`
- threading/antwoorden: `replyToMode`
- Optioneel (alleen voor `streamMode: "block"`):
- opmaak/levering: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/netwerk: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- acties/mogelijkheden: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- `channels.telegram.reactionNotifications`: Bepaalt welke reacties meldingen triggeren
- schrijven/geschiedenis: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Gerelateerd

- Details: [Pairing](/channels/pairing)
- [Channelroutering](/channels/channel-routing)
- Installatieproblemen oplossen (opdrachten)

