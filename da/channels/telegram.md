---
summary: "Status for Telegram-bot, funktioner og konfiguration"
read_when:
  - Arbejder med Telegram-funktioner eller webhooks
title: "Telegram"
---

# Telegram (Bot API)

Status: produktion-klar til bot DMs + grupper via grammY. Long polling er standardtilstanden; webhook-tilstand er valgfri.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standard-DM-politikken for Telegram er pairing.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostik og reparationsvejledninger på tværs af kanaler.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Fuldstændige kanal-konfigurationsmønstre og eksempler.
  
</Card>
</CardGroup>

## Hurtig opsætning

<Steps>
  <Step title="Create the bot token in BotFather">
    Åbn Telegram og chat med **@BotFather** (bekræft at brugernavnet er præcist `@BotFather`).

    ```
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
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (kun standardkonto).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Parringskoder udløber efter 1 time.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Tilføj botten til din gruppe, og indstil derefter `channels.telegram.groups` og `groupPolicy`, så de matcher din adgangsmodel.
  
</Step>
</Steps>

<Note>
Token-opløsningsrækkefølgen er kontoafhængig. I praksis har konfigurationsværdier forrang over env-fallback, og `TELEGRAM_BOT_TOKEN` gælder kun for standardkontoen.
</Note>

## Telegram-sideindstillinger

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Telegram-bots bruger som standard **Privacy Mode**, hvilket begrænser hvilke gruppemeddelelser de modtager.

    ```
    Hvis botten skal kunne se alle gruppemeddelelser, skal du enten:
    
    - deaktivere privacy mode via `/setprivacy`, eller
    - gøre botten til gruppeadministrator.
    
    Når du ændrer privacy mode, skal du fjerne + tilføje botten igen i hver gruppe, så Telegram anvender ændringen.
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Administratorstatus styres i Telegram-gruppens indstillinger.

    ```
    Admin-bots modtager alle gruppemeddelelser, hvilket er nyttigt for altid-aktive gruppefunktioner.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` for at tillade/afvise tilføjelse til grupper
    - `/setprivacy` for synlighedsadfærd i grupper
    ```

  
</Accordion>
</AccordionGroup>

## Adgangskontrol og aktivering

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` styrer adgang til direkte beskeder:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kræver at `allowFrom` indeholder `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` accepterer numeriske Telegram-bruger-ID’er. Præfikserne `telegram:` / `tg:` accepteres og normaliseres.
    Onboarding-guiden accepterer input som `@username` og konverterer det til numeriske ID’er.
    Hvis du har opgraderet, og din konfiguration indeholder `@username`-poster i allowlist, skal du køre `openclaw doctor --fix` for at konvertere dem (bedste forsøg; kræver et Telegram bot-token).
    
    ### Sådan finder du dit Telegram-bruger-ID
    
    Sikrere metode (ingen tredjepartsbot):
    
    1. Send en DM til din bot.
    2. Kør `openclaw logs --follow`.
    3. Læs `from.id`.
    
    Officiel Bot API-metode:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Tredjepartsmetode (mindre privat): `@userinfobot` eller `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    Der er to uafhængige kontroller:

    ```
    1. **Hvilke grupper er tilladt** (`channels.telegram.groups`)
       - ingen `groups`-konfiguration: alle grupper er tilladt
       - `groups` konfigureret: fungerer som allowlist (eksplicitte ID’er eller `"*"`)
    
    2. **Hvilke afsendere er tilladt i grupper** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (standard)
       - `disabled`
    
    `groupAllowFrom` bruges til filtrering af afsendere i grupper. Hvis den ikke er angivet, falder Telegram tilbage til `allowFrom`.
    `groupAllowFrom`-poster skal være numeriske Telegram-bruger-ID’er.
    
    Eksempel: tillad ethvert medlem i én specifik gruppe:
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

  <Tab title="Mention behavior">
    Gruppesvar kræver som standard en omtale.

    ```
    Omtale kan komme fra:
    
    - en indbygget `@botusername`-omtale, eller
    - omtale-mønstre i:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Sessionsniveau-kommandoer:
    
    - `/activation always`
    - `/activation mention`
    
    Disse opdaterer kun sessionstilstanden. Brug konfiguration for vedvarende indstillinger.
    
    Eksempel på vedvarende konfiguration:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    ```
    Sådan får du gruppe-chat-ID’et:
    
    - videresend en gruppemeddelelse til `@userinfobot` / `@getidsbot`
    - eller læs `chat.id` fra `openclaw logs --follow`
    - eller inspicér Bot API `getUpdates`
    ```

  
</Tab>
</Tabs>

## Runtime-adfærd

- Telegram ejes af gateway-processen.
- Routing er deterministisk: Telegram-indgående svarer tilbage til Telegram (modellen vælger ikke kanaler).
- Indgående meddelelser normaliseres til det delte kanal-envelope med svar-metadata og media-pladsholdere.
- Gruppesessioner er isoleret efter gruppe-ID. Forumemner tilføjer `:topic:<threadId>` for at holde emner adskilt.
- DM-meddelelser kan indeholde `message_thread_id`; OpenClaw ruter dem med trådbevidste sessionsnøgler og bevarer tråd-ID’et til svar.
- Long polling bruger grammY runner med sekventering pr. chat/pr. tråd. Samlet runner sink-konkurrrens styres af `agents.defaults.maxConcurrent`.
- Telegram Bot API har ikke understøttelse af læsekvitteringer (`sendReadReceipts` gælder ikke).

## Funktionsreference

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw kan streame delvise svar ved at sende en midlertidig Telegram-meddelelse og redigere den, efterhånden som tekst modtages.

    ```
    Krav:
    
    - `channels.telegram.streamMode` er ikke `"off"` (standard: `"partial"`)
    
    Tilstande:
    
    - `off`: ingen live-forhåndsvisning
    - `partial`: hyppige forhåndsvisningsopdateringer fra delvis tekst
    - `block`: opdelte forhåndsvisningsopdateringer ved brug af `channels.telegram.draftChunk`
    
    `draftChunk`-standarder for `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` begrænses af `channels.telegram.textChunkLimit`.
    
    Dette fungerer i direkte chats og grupper/emner.
    
    For tekstbaserede svar beholder OpenClaw den samme forhåndsvisningsmeddelelse og foretager en endelig redigering på stedet (ingen ekstra meddelelse).
    
    For komplekse svar (f.eks. media-payloads) falder OpenClaw tilbage til normal endelig levering og rydder derefter forhåndsvisningsmeddelelsen.
    
    `streamMode` er adskilt fra block streaming. Når block streaming eksplicit er aktiveret for Telegram, springer OpenClaw forhåndsvisningsstream over for at undgå dobbelt streaming.
    
    Telegram-only reasoning stream:
    
    - `/reasoning stream` sender ræsonnement til live-forhåndsvisningen under generering
    - det endelige svar sendes uden ræsonnementstekst
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Udgående tekst bruger Telegram `parse_mode: "HTML"`.

    ```
    - Markdown-lignende tekst gengives til Telegram-sikker HTML.
    - Rå model-HTML escapes for at reducere Telegram parse-fejl.
    - Hvis Telegram afviser fortolket HTML, forsøger OpenClaw igen som almindelig tekst.
    
    Link-forhåndsvisninger er aktiveret som standard og kan deaktiveres med `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Registrering af Telegram-kommandomenus håndteres ved opstart med `setMyCommands`.

    ```
    Standardindstillinger for native commands:
    
    - `commands.native: "auto"` aktiverer native commands for Telegram
    
    Tilføj brugerdefinerede poster i kommandomenet:
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

  <Accordion title="Inline buttons">
    Konfigurer scope for inline keyboard:

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
    Pr.-konto-override:
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
    Scopes:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (standard)
    
    Legacy `capabilities: ["inlineButtons"]` mappes til `inlineButtons: "all"`.
    
    Eksempel på message action:
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
    Callback-klik sendes videre til agenten som tekst:
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram-værktøjshandlinger inkluderer:

    ```
    - `sendMessage` (`to`, `content`, valgfri `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Kanalmeddelelseshandlinger eksponerer ergonomiske aliaser (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Adgangskontrol:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (standard: deaktiveret)
    
    Semantik for fjernelse af reaktioner: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Telegram understøtter eksplicitte svartråd-tags i genereret output:

    ```
    - `[[reply_to_current]]` svarer på den udløsende besked
    - `[[reply_to:<id>]]` svarer på et specifikt Telegram-besked-ID
    
    `channels.telegram.replyToMode` styrer håndteringen:
    
    - `off` (standard)
    - `first`
    - `all`
    
    Bemærk: `off` deaktiverer implicit svartrådning. Eksplicitte `[[reply_to_*]]`-tags respekteres stadig.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Forum-supergrupper:

    ```
    - emnesessionsnøgler tilføjer `:topic:<threadId>`
    - svar og skriveindikator målrettes emnetråden
    - konfigurationssti for emne:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Generelt emne (`threadId=1`) særtilfælde:
    
    - afsendelse af beskeder udelader `message_thread_id` (Telegram afviser `sendMessage(...thread_id=1)`)
    - skrivehandlinger inkluderer stadig `message_thread_id`
    
    Emnearv: emneposter arver gruppeindstillinger, medmindre de tilsidesættes (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Skabelonkontekst inkluderer:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM-trådadfærd:
    
    - private chats med `message_thread_id` bevarer DM-routing, men bruger trådbevidste sessionsnøgler/svarmål.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Lydmeddelelser

    ```
    Telegram skelner mellem stemmenoter og lydfiler.
    
    - standard: lydfiladfærd
    - tagget `[[audio_as_voice]]` i agentsvar for at tvinge afsendelse som stemmenote
    
    Eksempel på beskedhandling:
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
    ### Videomeddelelser
    
    Telegram skelner mellem videofiler og videonoter.
    
    Eksempel på beskedhandling:
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
    Videonoter understøtter ikke billedtekster; angivet beskedtekst sendes separat.
    
    ### Klistermærker
    
    Håndtering af indgående klistermærker:
    
    - statisk WEBP: downloades og behandles (pladsholder `<media:sticker>`)
    - animeret TGS: springes over
    - video WEBM: springes over
    
    Kontekstfelter for klistermærker:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Cachefil for klistermærker:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Klistermærker beskrives én gang (når muligt) og caches for at reducere gentagne vision-kald.
    
    Aktivér klistermærkehandlinger:
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
    Send klistermærkehandling:
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
    Søg i cachede klistermærker:
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

  <Accordion title="Reaction notifications">
    Telegram-reaktioner ankommer som `message_reaction`-opdateringer (adskilt fra beskedpayloads).

    ```
    Når aktiveret, sætter OpenClaw systemhændelser i kø som:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Konfiguration:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (standard: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (standard: `minimal`)
    
    Bemærkninger:
    
    - `own` betyder brugerreaktioner på bot-sendte beskeder kun (best-effort via cache over sendte beskeder).
    - Telegram leverer ikke tråd-ID’er i reaktionsopdateringer.
      - ikke-forumgrupper routes til gruppens chatsession
      - forumgrupper routes til gruppens generelle emnesession (`:topic:1`), ikke det præcise oprindelige emne
    
    `allowed_updates` for polling/webhook inkluderer automatisk `message_reaction`.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` sender en bekræftelses-emoji, mens OpenClaw behandler en indgående besked.

    ```
    Løsningsrækkefølge:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agentens identitets-emoji som fallback (`agents.list[].identity.emoji`, ellers "👀")
    
    Bemærkninger:
    
    - Telegram forventer unicode-emoji (for eksempel "👀").
    - Brug `""` for at deaktivere reaktionen for en kanal eller konto.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Kanal-konfigurationsskrivninger er aktiveret som standard (`configWrites !== false`).

    ```
    Telegram-udløste skrivninger inkluderer:
    
    - gruppemigreringshændelser (`migrate_to_chat_id`) for at opdatere `channels.telegram.groups`
    - `/config set` og `/config unset` (kræver at kommandoer er aktiveret)
    
    Deaktivér:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    Standard: long polling.

    ```
    Webhook-tilstand:
    
    - sæt `channels.telegram.webhookUrl`
    - sæt `channels.telegram.webhookSecret` (påkrævet når webhook-URL er sat)
    - valgfri `channels.telegram.webhookPath` (standard `/telegram-webhook`)
    - valgfri `channels.telegram.webhookHost` (standard `127.0.0.1`)
    
    Standard lokal lytter i webhook-tilstand binder til `127.0.0.1:8787`.
    
    Hvis dit offentlige endpoint er anderledes, skal du placere en reverse proxy foran og pege `webhookUrl` på den offentlige URL.
    Sæt `webhookHost` (for eksempel `0.0.0.0`), når du bevidst har brug for ekstern indgående trafik.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` standard er 4000.
    - `channels.telegram.chunkMode="newline"` foretrækker afsnitsgrænser (blanke linjer) før opdeling efter længde.
    - `channels.telegram.mediaMaxMb` (standard 5) begrænser størrelsen på download/behandling af indgående Telegram-medier.
    - `channels.telegram.timeoutSeconds` tilsidesætter Telegram API-klientens timeout (hvis ikke sat, bruges grammY-standard).
    - gruppe-konteksthistorik bruger `channels.telegram.historyLimit` eller `messages.groupChat.historyLimit` (standard 50); `0` deaktiverer.
    - DM-historikstyring:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - udgående Telegram API-genforsøg kan konfigureres via `channels.telegram.retry`.

    ```
    CLI-sendmål kan være numerisk chat-ID eller brugernavn:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## Fejlfinding

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - Hvis `requireMention=false`, skal Telegrams privacy mode tillade fuld synlighed.
      - BotFather: `/setprivacy` -> Disable
      - fjern derefter + tilføj botten igen til gruppen
    - `openclaw channels status` advarer, når konfigurationen forventer gruppemeddelelser uden omtale.
    - `openclaw channels status --probe` kan kontrollere eksplicitte numeriske gruppe-ID’er; wildcard `"*"` kan ikke medlemskabstestes.
    - hurtig sessionstest: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - når `channels.telegram.groups` findes, skal gruppen være angivet (eller inkludere `"*"`)
    - verificér botmedlemskab i gruppen
    - gennemse logs: `openclaw logs --follow` for årsager til at blive sprunget over
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - autorisér din afsenderidentitet (parring og/eller numerisk `allowFrom`)
    - kommandoautorisation gælder stadig, selv når gruppepolitikken er `open`
    - `setMyCommands failed` indikerer normalt DNS/HTTPS-tilgængelighedsproblemer til `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + custom fetch/proxy kan udløse øjeblikkelig abortadfærd, hvis AbortSignal-typer ikke matcher.
    - Nogle hosts resolver `api.telegram.org` til IPv6 først; defekt IPv6-egress kan forårsage intermitterende Telegram API-fejl.
    - Validér DNS-svar:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Telegram understøtter valgfri trådede svar via tags:

## Telegram-konfigurationsreferencepunkter

Styres af `channels.telegram.replyToMode`:

- `first` (standard), `all`, `off`.

- `channels.telegram.botToken`: bot-token (BotFather).

- `channels.telegram.tokenFile`: læs token fra filsti.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (standard: parring).

- `channels.telegram.allowFrom`: DM-allowlist (numeriske Telegram-bruger-ID’er). `open` kræver `"*"`. `openclaw doctor --fix` kan løse ældre `@username`-poster til ID’er.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (standard: tilladelsesliste).

- `channels.telegram.groupAllowFrom`: tilladelsesliste for gruppeafsendere (numeriske Telegram-bruger-ID'er). `openclaw doctor --fix` kan konvertere ældre `@username`-poster til ID'er.

- `channels.telegram.groups`: per-gruppe-standarder + tilladelsesliste (brug `"*"` for globale standarder).
  - `channels.telegram.groups.<id>.groupPolicy`: per-gruppe tilsidesættelse for groupPolicy (`åben - allowlist - disabled`).
  - `channels.telegram.groups.<id>.requireMention`: nævne gating default.
  - `channels.telegram.groups.<id>.skills`: skill-filter (udeladt = alle Skills, tom = ingen).
  - `channels.telegram.groups.<id>.allowFrom`: per-gruppe afsender tillad tilsidesættelse.
  - `channels.telegram.groups.<id>.systemPrompt`: ekstra systemprompt for gruppen.
  - `channels.telegram.groups.<id>.enabled`: Deaktivér gruppen når `false`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: Overskrivninger pr. emne (samme felter som gruppe).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: Overskrivning pr. emne for groupPolicy (`åben - allowlist - disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic nævne gating override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (standard: tilladelsesliste).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account overrid.

- `channels.telegram.replyToMode`: `off | first | all` (standard: `off`).

- `channels.telegram.textChunkLimit`: udgående chunk-størrelse (tegn).

- `channels.telegram.chunkMode`: `length` (standard) eller `newline` for at splitte på tomme linjer (afsnitsgrænser) før længde-chunking.

- `channels.telegram.linkPreview`: slå link-forhåndsvisninger til/fra for udgående beskeder (standard: true).

- `channels.telegram.streamMode`: `off | partial | block` (live stream-forhåndsvisning).

- `channels.telegram.mediaMaxMb`: grænse for indgående/udgående medier (MB).

- `channels.telegram.retry`: retry-politik for udgående Telegram API-kald (forsøg, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: tilsidesætte Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: proxy-URL for Bot API-kald (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: aktivér webhook-tilstand (kræver `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook-hemmelighed (påkrævet, når webhookUrl er sat).

- `channels.telegram.webhookPath`: lokal webhook-sti (standard `/telegram-webhook`).

- `channels.telegram.webhookHost`: lokal webhook-bindingsvært (standard `127.0.0.1`).

- `channels.telegram.actions.reactions`: gate Telegram-værktøjsreaktioner.

- `channels.telegram.actions.sendMessage`: gate Telegram-værktøjs-beskedafsendelser.

- `channels.telegram.actions.deleteMessage`: gate Telegram-værktøjs-beskedsletninger.

- `channels.telegram.actions.sticker`: gate Telegram-klistermærkehandlinger — send og søg (standard: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — styr hvilke reaktioner der udløser systemevents (standard: `own`, når ikke sat).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — styr agentens reaktionskapacitet (standard: `minimal`, når ikke sat).

- [Konfigurationsreference - Telegram](/gateway/configuration-reference#telegram)

Telegram-specifikke high-signal-felter:

- opstart/autentificering: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- adgangskontrol: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- kommando/menu: `commands.native`, `customCommands`
- tråde/svar: `replyToMode`
- streaming: `streamMode` (forhåndsvisning), `draftChunk`, `blockStreaming`
- formatering/levering: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- medier/netværk: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- handlinger/kapaciteter: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reaktioner: `reactionNotifications`, `reactionLevel`
- skrivninger/historik: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Relateret

- `[[audio_as_voice]]` — send lyd som en talebesked i stedet for en fil.
- [Kanalsrouting](/channels/channel-routing)
- [Fejlfinding](/channels/troubleshooting)
