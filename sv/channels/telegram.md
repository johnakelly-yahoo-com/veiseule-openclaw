---
summary: "Status för Telegram-botstöd, funktioner och konfiguration"
read_when:
  - Arbetar med Telegram-funktioner eller webhooks
title: "Telegram"
---

# Telegram (Bot API)

Status: produktionsredo för bot DMs + grupper via grammY. Long polling är standardläget; webhook-läge är valfritt.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standardpolicy för DM i Telegram är parkoppling.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalöverskridande diagnostik och åtgärdsguider.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Fullständiga konfigurationsmönster och exempel för kanaler.
  
</Card>
</CardGroup>

## Snabbstart

<Steps>
  <Step title="Create the bot token in BotFather">
    Öppna Telegram och chatta med **@BotFather** (bekräfta att användarnamnet är exakt `@BotFather`).

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
    Miljövariabel som fallback: `TELEGRAM_BOT_TOKEN=...` (endast standardkonto).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Parkopplingskoder upphör att gälla efter 1 timme.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Lägg till boten i din grupp och ställ sedan in `channels.telegram.groups` och `groupPolicy` så att de matchar din åtkomstmodell.
  
</Step>
</Steps>

<Note>
Token-upplösningsordningen är kontomedveten. I praktiken har konfigurationsvärden företräde framför miljövariabelns fallback, och `TELEGRAM_BOT_TOKEN` gäller endast för standardkontot.
</Note>

## Inställningar på Telegram-sidan

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Telegram-botar använder som standard **Privacy Mode**, vilket begränsar vilka gruppmeddelanden de tar emot.

    ```
    Om boten måste se alla gruppmeddelanden, antingen:
    
    - inaktivera privacy mode via `/setprivacy`, eller
    - gör boten till gruppadministratör.
    
    När du växlar privacy mode, ta bort + lägg till boten igen i varje grupp så att Telegram tillämpar ändringen.
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Administratörsstatus styrs i Telegram-gruppens inställningar.

    ```
    Admin-botar tar emot alla gruppmeddelanden, vilket är användbart för alltid-aktivt gruppbeteende.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` för att tillåta/neka att läggas till i grupper
    - `/setprivacy` för synlighetsbeteende i grupper
    ```

  
</Accordion>
</AccordionGroup>

## Åtkomstkontroll och aktivering

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` styr åtkomst till direktmeddelanden:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kräver att `allowFrom` inkluderar `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` accepterar numeriska Telegram-användar-ID:n. Prefixen `telegram:` / `tg:` accepteras och normaliseras.
    Onboarding-guiden accepterar `@username`-inmatning och löser det till numeriska ID:n.
    Om du har uppgraderat och din konfiguration innehåller `@username`-poster i allowlist, kör `openclaw doctor --fix` för att lösa dem (best effort; kräver en Telegram-bottoken).
    
    ### Hitta ditt Telegram-användar-ID
    
    Säkrare (ingen tredjepartsbot):
    
    1. Skicka DM till din bot.
    2. Kör `openclaw logs --follow`.
    3. Läs `from.id`.
    
    Officiell Bot API-metod:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Tredjepartsmetod (mindre privat): `@userinfobot` eller `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    Det finns två oberoende kontroller:

    ```
    1. **Vilka grupper som är tillåtna** (`channels.telegram.groups`)
       - ingen `groups`-konfiguration: alla grupper tillåtna
       - `groups` konfigurerad: fungerar som allowlist (explicita ID:n eller `"*"`)
    
    2. **Vilka avsändare som är tillåtna i grupper** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (standard)
       - `disabled`
    
    `groupAllowFrom` används för filtrering av gruppavsändare. Om det inte är satt faller Telegram tillbaka på `allowFrom`.
    `groupAllowFrom`-poster måste vara numeriska Telegram-användar-ID:n.
    
    Exempel: tillåt valfri medlem i en specifik grupp:
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
    Gruppsvar kräver omnämnande som standard.

    ```
    Omnämnande kan komma från:
    
    - inbyggt `@botusername`-omnämnande, eller
    - omnämnandemönster i:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Sessionsnivåns kommandoväxlingar:
    
    - `/activation always`
    - `/activation mention`
    
    Dessa uppdaterar endast sessionstillståndet. Använd konfiguration för beständighet.
    
    Exempel på beständig konfiguration:
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
    Hämta gruppens chat-ID:
    
    - vidarebefordra ett gruppmeddelande till `@userinfobot` / `@getidsbot`
    - eller läs `chat.id` från `openclaw logs --follow`
    - eller inspektera Bot API `getUpdates`
    ```

  
</Tab>
</Tabs>

## Körningsbeteende

- Telegram ägs av gateway-processen.
- Routning är deterministisk: Telegram inkommande svarar tillbaka till Telegram (modellen väljer inte kanaler).
- Inkommande meddelanden normaliseras till det delade kanal-kuvertet med svarsmetadata och media-platshållare.
- Gruppsessioner isoleras per grupp-ID. Forumämnen lägger till `:topic:<threadId>` för att hålla ämnen isolerade.
- DM-meddelanden kan innehålla `message_thread_id`; OpenClaw routar dem med trådmedvetna sessionsnycklar och bevarar tråd-ID för svar.
- Long polling använder grammY runner med sekvensering per chatt/per tråd. Övergripande samtidighet för runner sink använder `agents.defaults.maxConcurrent`.
- Telegram Bot API har inget stöd för läskvitton (`sendReadReceipts` gäller inte).

## Funktionsreferens

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw kan strömma partiella svar genom att skicka ett tillfälligt Telegram-meddelande och redigera det när text anländer.

    ```
    Krav:
    
    - `channels.telegram.streamMode` är inte `"off"` (standard: `"partial"`)
    
    Lägen:
    
    - `off`: ingen liveförhandsvisning
    - `partial`: frekventa förhandsvisningsuppdateringar från partiell text
    - `block`: uppdelade förhandsvisningsuppdateringar med `channels.telegram.draftChunk`
    
    `draftChunk`-standardvärden för `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` begränsas av `channels.telegram.textChunkLimit`.
    
    Detta fungerar i direktchattar och grupper/ämnen.
    
    För textbaserade svar behåller OpenClaw samma förhandsvisningsmeddelande och gör en slutlig redigering på plats (inget andra meddelande).
    
    För komplexa svar (till exempel mediepayloads) faller OpenClaw tillbaka till normal slutleverans och städar sedan upp förhandsvisningsmeddelandet.
    
    `streamMode` är separat från blockströmning. När blockströmning uttryckligen är aktiverad för Telegram hoppar OpenClaw över förhandsvisningsströmmen för att undvika dubbelströmning.
    
    Telegram-endast reasoning-ström:
    
    - `/reasoning stream` skickar reasoning till liveförhandsvisningen under generering
    - slutligt svar skickas utan reasoning-text
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Utgående text använder Telegram `parse_mode: "HTML"`.

    ```
    - Markdown-liknande text renderas till Telegram-säker HTML.
    - Rå modell-HTML escapes för att minska Telegrams tolkningsfel.
    - Om Telegram avvisar tolkad HTML försöker OpenClaw igen som vanlig text.
    
    Länkförhandsvisningar är aktiverade som standard och kan inaktiveras med `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Registrering av Telegram-kommandomeny hanteras vid uppstart med `setMyCommands`.

    ```
    Standard för inbyggda kommandon:
    
    - `commands.native: "auto"` aktiverar inbyggda kommandon för Telegram
    
    Lägg till egna poster i kommandomenyn:
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
    Konfigurera omfång för inline-tangentbord:

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
    Per-konto-överskrivning:
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
    Omfång:
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (standard)
    
    Äldre `capabilities: ["inlineButtons"]` mappas till `inlineButtons: "all"`.
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
    Exempel på meddelandeåtgärd:
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">Callback-klick skickas vidare till agenten som text:
`callback_data: <value>`

    ```
    
        Telegram-verktygsåtgärder inkluderar:
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Telegram stöder explicita svarstråd-taggar i genererad output:

    ```
    - `[[reply_to_current]]` svarar på det utlösande meddelandet
    - `[[reply_to:<id>]]` svarar på ett specifikt Telegram-meddelande-ID
    
    `channels.telegram.replyToMode` styr hanteringen:
    
    - `off` (standard)
    - `first`
    - `all`
    
    Obs: `off` inaktiverar implicit svarstrådning. Explicita `[[reply_to_*]]`-taggar respekteras fortfarande.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Forum-supergrupper:

    ```
    - ämnessessionsnycklar lägger till `:topic:<threadId>`
    - svar och skrivindikator riktas till ämnestråden
    - konfigurationssökväg för ämne:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Allmänt ämne (`threadId=1`) specialfall:
    
    - meddelanden som skickas utelämnar `message_thread_id` (Telegram avvisar `sendMessage(...thread_id=1)`)
    - skrivåtgärder inkluderar fortfarande `message_thread_id`
    
    Ämnesarv: ämnesposter ärver gruppinställningar om de inte åsidosätts (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Mallkontext inkluderar:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM-trådbeteende:
    
    - privata chattar med `message_thread_id` behåller DM-routing men använder trådmedvetna sessionsnycklar/svarsmål.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Ljudmeddelanden

    ```
    Telegram skiljer mellan röstmeddelanden och ljudfiler.
    
    - standard: beteende för ljudfil
    - taggen `[[audio_as_voice]]` i agentens svar för att tvinga sändning som röstmeddelande
    
    Exempel på meddelandeåtgärd:
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
    ### Videomeddelanden
    
    Telegram skiljer mellan videofiler och videomeddelanden.
    
    Exempel på meddelandeåtgärd:
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
    Videomeddelanden stöder inte bildtexter; angiven meddelandetext skickas separat.
    
    ### Klistermärken
    
    Hantering av inkommande klistermärken:
    
    - statisk WEBP: laddas ner och bearbetas (platshållare `<media:sticker>`)
    - animerad TGS: hoppas över
    - video WEBM: hoppas över
    
    Kontextfält för klistermärken:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Cachefil för klistermärken:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Klistermärken beskrivs en gång (när möjligt) och cachas för att minska upprepade vision-anrop.
    
    Aktivera klistermärkesåtgärder:
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
    Skicka klistermärkesåtgärd:
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
    Sök i cachade klistermärken:
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
    Telegram-reaktioner anländer som `message_reaction`-uppdateringar (separata från meddelandepayloads).

    ```
    När aktiverat köar OpenClaw systemhändelser som:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Konfiguration:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (standard: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (standard: `minimal`)
    
    Obs:
    
    - `own` betyder användarreaktioner på bot-skickade meddelanden endast (bästa möjliga via cache för skickade meddelanden).
    - Telegram tillhandahåller inte tråd-ID:n i reaktionsuppdateringar.
      - icke-forumgrupper routas till gruppchattens session
      - forumgrupper routas till gruppens allmänna ämnessession (`:topic:1`), inte det exakta ursprungsämnet
    
    `allowed_updates` för polling/webhook inkluderar `message_reaction` automatiskt.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` skickar en bekräftelse-emoji medan OpenClaw bearbetar ett inkommande meddelande.

    ```
    Upplösningsordning:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - reserv-emoji från agentidentitet (`agents.list[].identity.emoji`, annars "👀")
    
    Obs:
    
    - Telegram förväntar sig unicode-emoji (till exempel "👀").
    - Använd `""` för att inaktivera reaktionen för en kanal eller ett konto.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Kanalens konfigurationsskrivningar är aktiverade som standard (`configWrites !== false`).

    ```
    Telegram-utlösta skrivningar inkluderar:
    
    - gruppmigreringshändelser (`migrate_to_chat_id`) för att uppdatera `channels.telegram.groups`
    - `/config set` och `/config unset` (kräver att kommandon är aktiverade)
    
    Inaktivera:
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
    Webhook-läge:
    
    - ange `channels.telegram.webhookUrl`
    - ange `channels.telegram.webhookSecret` (krävs när webhook-URL är angiven)
    - valfritt `channels.telegram.webhookPath` (standard `/telegram-webhook`)
    - valfritt `channels.telegram.webhookHost` (standard `127.0.0.1`)
    
    Standardlokal lyssnare för webhook-läge binder till `127.0.0.1:8787`.
    
    Om din publika endpoint skiljer sig, placera en reverse proxy framför och peka `webhookUrl` mot den publika URL:en.
    Ange `webhookHost` (till exempel `0.0.0.0`) när du avsiktligt behöver extern ingress.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` har standardvärdet 4000.
    - `channels.telegram.chunkMode="newline"` föredrar styckegränser (tomma rader) före uppdelning efter längd.
    - `channels.telegram.mediaMaxMb` (standard 5) begränsar nedladdnings-/bearbetningsstorlek för inkommande Telegram-media.
    - `channels.telegram.timeoutSeconds` åsidosätter timeout för Telegram API-klienten (om inte angivet används grammY-standard).
    - gruppkontextens historik använder `channels.telegram.historyLimit` eller `messages.groupChat.historyLimit` (standard 50); `0` inaktiverar.
    - DM-historikkontroller:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - utgående Telegram API-omförsök kan konfigureras via `channels.telegram.retry`.

    ```
    CLI-sändningsmål kan vara numeriskt chatt-ID eller användarnamn:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## Felsökning

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - Om `requireMention=false` måste Telegrams sekretessläge tillåta full synlighet.
      - BotFather: `/setprivacy` -> Disable
      - ta sedan bort + lägg till boten i gruppen igen
    - `openclaw channels status` varnar när konfigurationen förväntar sig gruppmeddelanden utan omnämnande.
    - `openclaw channels status --probe` kan kontrollera explicita numeriska grupp-ID:n; jokertecknet `"*"` kan inte medlemskapskontrolleras.
    - snabb sessionstest: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - när `channels.telegram.groups` finns måste gruppen vara listad (eller inkludera `"*"`)
    - verifiera botens medlemskap i gruppen
    - granska loggar: `openclaw logs --follow` för orsaker till att något hoppas över
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - auktorisera din avsändaridentitet (parkoppling och/eller numerisk `allowFrom`)
    - kommandoauktorisering gäller fortfarande även när gruppolicyn är `open`
    - `setMyCommands failed` indikerar vanligtvis DNS-/HTTPS-åtkomstproblem till `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + anpassad fetch/proxy kan utlösa omedelbart avbrottsbeteende om AbortSignal-typer inte matchar.
    - Vissa värdar slår upp `api.telegram.org` till IPv6 först; trasig IPv6-egress kan orsaka intermittenta Telegram API-fel.
    - Validera DNS-svar:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Telegram stöder valfri trådad svarning via taggar:

## Telegram konfigurationsreferenspunkter

Styrs av `channels.telegram.replyToMode`:

- `first` (standard), `all`, `off`.

- `channels.telegram.botToken`: bot‑token (BotFather).

- `channels.telegram.tokenFile`: läs token från filsökväg.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (standard: parkoppling).

- `channels.telegram.allowFrom`: DM-tillåtslista (numeriska Telegram-användar-ID:n). `open` kräver `"*"`. `openclaw doctor --fix` kan lösa äldre `@username`-poster till ID:n.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (standard: tillåtelselista).

- `channels.telegram.groupAllowFrom`: tillåtslista för gruppavsändare (numeriska Telegram-användar-ID:n). `openclaw doctor --fix` kan lösa äldre `@username`-poster till ID:n.

- `channels.telegram.groups`: per‑grupp‑standarder + tillåtelselista (använd `"*"` för globala standarder).
  - `channels.telegram.groups.<id>.groupPolicy`: åsidosätt per grupp för groupPolicy (`open <unk> allowlist <unk> disabled`).
  - `channels.telegram.groups.<id>.requireMention`: nämna gating default.
  - `channels.telegram.groups.<id>.skills`: färdighetsfilter (utelämna = alla Skills, tom = inga).
  - `channels.telegram.groups.<id>.allowFrom`: Avsändare per grupp tillåten lista åsidosätt.
  - `channels.telegram.groups.<id>.systemPrompt`: extra systemprompt för gruppen.
  - `channels.telegram.groups.<id>.enabled`: inaktivera gruppen när `false`.
  - `channels.telegram.groups.<id>.trådar.<threadId>.*`: åsidosättningar per ämne (samma fält som grupp).
  - `channels.telegram.groups.<id>.trådar.<threadId>.groupPolicy`: åsidosätt per ämne för groupPolicy (`open <unk> allowlist <unk> disabled`).
  - `channels.telegram.groups.<id>.trådar.<threadId>.requireMention`: per ämne nämner gating override.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (standard: tillåtelselista).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: åsidosätter per konto.

- `channels.telegram.replyToMode`: `off | first | all` (standard: `off`).

- `channels.telegram.textChunkLimit`: utgående chunk‑storlek (tecken).

- `channels.telegram.chunkMode`: `length` (standard) eller `newline` för att dela på tomrader (styckegränser) före längd‑chunkning.

- `channels.telegram.linkPreview`: växla länkförhandsvisningar för utgående meddelanden (standard: true).

- `channels.telegram.streamMode`: `off | partial | block` (förhandsvisning av live‑ström).

- `channels.telegram.mediaMaxMb`: gräns för inkommande/utgående media (MB).

- `channels.telegram.retry`: policy för omförsök för utgående Telegram API‑anrop (försök, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: åsidosätta Noden autoSelectFamily (true=enable, false=disable). Standard är inaktiverat på Node 22 för att undvika tidsgräns för Happy Eyeball.

- `channels.telegram.proxy`: proxy‑URL för Bot API‑anrop (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: aktivera webhook‑läge (kräver `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: webhook‑hemlighet (krävs när webhookUrl är satt).

- `channels.telegram.webhookPath`: lokal webhook‑sökväg (standard `/telegram-webhook`).

- `channels.telegram.webhookHost`: lokal bind‑värd för webhook (standard `127.0.0.1`).

- `channels.telegram.actions.reactions`: gate Telegram‑verktygsreaktioner.

- `channels.telegram.actions.sendMessage`: gate Telegram‑verktygets meddelandesändningar.

- `channels.telegram.actions.deleteMessage`: gate Telegram‑verktygets borttagning av meddelanden.

- `channels.telegram.actions.sticker`: gate Telegram‑klistermärkesåtgärder — skicka och sök (standard: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — styr vilka reaktioner som triggar systemhändelser (standard: `own` när ej satt).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — styr agentens reaktionsförmåga (standard: `minimal` när ej satt).

- [Konfigurationsreferens - Telegram](/gateway/configuration-reference#telegram)

Telegram‑specifika fält med hög signal:

- uppstart/autentisering: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- åtkomstkontroll: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- kommandon/meny: `commands.native`, `customCommands`
- trådning/svar: `replyToMode`
- strömning: `streamMode` (förhandsvisning), `draftChunk`, `blockStreaming`
- formatering/leverans: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/nätverk: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- åtgärder/funktioner: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reaktioner: `reactionNotifications`, `reactionLevel`
- skrivningar/historik: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Relaterat

- `[[audio_as_voice]]` — skicka ljud som röstanteckning i stället för fil.
- [Kanalroutning](/channels/channel-routing)
- [Felsökning](/channels/troubleshooting)
