---
summary: "WhatsApp‑integration (webbkanal): inloggning, inkorg, svar, media och drift"
read_when:
  - Arbetar med WhatsApp/webbkanalens beteende eller inkorgsroutning
title: "WhatsApp"
---

# WhatsApp (webbkanal)

Status: produktionsklar via WhatsApp Web (Baileys). Gateway äger länkad(e) session(er).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standardpolicy för DM är parkoppling för okända avsändare.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnostik och reparationsguider över flera kanaler.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Fullständiga konfigurationsmönster och exempel för kanaler.
  
</Card>
</CardGroup>

## Snabbstart

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
    För ett specifikt konto:
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
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    ```
    Parkopplingsförfrågningar upphör efter 1 timme. Väntande förfrågningar är begränsade till 3 per kanal.
    ```

  
</Step>
</Steps>

<Note>
OpenClaw rekommenderar att köra WhatsApp på ett separat nummer när det är möjligt. (Kanalens metadata och onboarding-flöde är optimerade för den konfigurationen, men konfigurationer med personligt nummer stöds också.)
</Note>

## Distributionsmönster

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Detta är det renaste driftläget:

    ````
    - separat WhatsApp-identitet för OpenClaw
    - tydligare DM-allowlists och routningsgränser
    - lägre risk för förvirring i självchatt
    
    Minimal policy-konfiguration:
    
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
    ````

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Onboarding stöder läge med personligt nummer och skriver en självchattvänlig grundkonfiguration:

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

  <Accordion title="WhatsApp Web-only channel scope">
    Meddelandeplattformens kanal är WhatsApp Web-baserad (`Baileys`) i den nuvarande OpenClaw-kanalarkitekturen.

    ```
    Det finns ingen separat Twilio WhatsApp-meddelandekanal i det inbyggda chat-channel-registret.
    ```

  
</Accordion>
</AccordionGroup>

## Körningsmodell

- Gateway äger WhatsApp-socketen och återanslutningsloopen.
- Utgående meddelanden kräver en aktiv WhatsApp-lyssnare för målkontot.
- Status- och broadcast-chattar ignoreras (`@status`, `@broadcast`).
- Direktchattar använder DM-sessionsregler (`session.dmScope`; standard `main` slår ihop DM till agentens huvudsakliga session).
- Gruppsessioner är isolerade (`agent:<agentId>:whatsapp:group:<jid>`).

## Åtkomstkontroll och aktivering

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` styr åtkomst till direktchattar:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kräver att `allowFrom` inkluderar `"*"`)
    - `disabled`
    
    `allowFrom` accepterar nummer i E.164-format (normaliseras internt).
    
    Åsidosättning per konto: `channels.whatsapp.accounts.<id>.dmPolicy` (och `allowFrom`) har företräde framför kanalens standardvärden för det kontot.
    
    Detaljer om körningsbeteende:
    
    - parningar sparas i kanalens allow-store och slås samman med konfigurerad `allowFrom`
    - om ingen allowlist är konfigurerad tillåts det länkade egna numret som standard
    - utgående `fromMe` DM paras aldrig automatiskt
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Gruppåtkomst har två lager:

    ```
    1. **Allowlist för gruppmedlemskap** (`channels.whatsapp.groups`)
       - om `groups` utelämnas är alla grupper giltiga
       - om `groups` finns fungerar den som en grupp-allowlist (`"*"` tillåts)
    
    2. **Policy för gruppavsändare** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: avsändarens allowlist kringgås
       - `allowlist`: avsändaren måste matcha `groupAllowFrom` (eller `*`)
       - `disabled`: blockera all inkommande grupptrafik
    
    Fallback för avsändar-allowlist:
    
    - om `groupAllowFrom` inte är satt faller körningen tillbaka på `allowFrom` när det finns
    
    Obs: om inget `channels.whatsapp`-block alls finns är fallback för gruppolicy vid körning i praktiken `open`.
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Gruppsvar kräver omnämnande som standard.

    ```
    Identifiering av omnämnande inkluderar:
    
    - explicita WhatsApp-omnämnanden av botens identitet
    - konfigurerade regex-mönster för omnämnande (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit svar-till-bot-detektering (avsändaren av svaret matchar botens identitet)
    
    Aktiveringskommando på sessionsnivå:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` uppdaterar sessionens tillstånd (inte global konfiguration). Det är ägarstyrt.
    ```

  
</Tab>
</Tabs>

## Beteende för personligt nummer och självchatt

Inaktivera globalt:

- hoppa över läskvitton för självchattens turer
- ignorera auto-trigger-beteende för mention-JID som annars skulle pinga dig själv
- om `messages.responsePrefix` inte är satt får självchattsvar som standard `[{identity.name}]` eller `[openclaw]`

## Meddelandenormalisering och kontext

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Inkommande WhatsApp-meddelanden paketeras i det gemensamma inbound-kuvertet.

    ````
    Om ett citerat svar finns läggs kontext till i följande format:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Metadatafält för svar fylls också i när de är tillgängliga (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, avsändarens JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Inkommande meddelanden som endast innehåller media normaliseras med platshållare såsom:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Plats- och kontaktpayloads normaliseras till textuell kontext före routning.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    För grupper kan obearbetade meddelanden buffras och injiceras som kontext när boten slutligen triggas.

    ```
    - standardgräns: `50`
    - konfiguration: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` inaktiverar
    
    Injektionsmarkörer:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    Läskvitton är aktiverade som standard för accepterade inkommande WhatsApp-meddelanden.

    ````
    Inaktivera globalt:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Åsidosättning per konto:
    
    ```json5
    {
      channels: {
        whatsapp: {
          accounts: {
            work: {
              sendReadReceipts: false,
            },
          },
        },
      },
    }
    ```
    
    Självchattens turer hoppar över läskvitton även när de är globalt aktiverade.
    ````

  
</Accordion>
</AccordionGroup>

## Leverans, uppdelning och media

<AccordionGroup>
  <Accordion title="Text chunking">
    - standardgräns för chunk: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - läget `newline` föredrar styckegränser (tomma rader) och faller sedan tillbaka till längdsäker uppdelning
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - stöder bild-, video-, ljud- (PTT voice-note) och dokumentpayloads
    - `audio/ogg` skrivs om till `audio/ogg; codecs=opus` för kompatibilitet med voice-note
    - uppspelning av animerad GIF stöds via `gifPlayback: true` vid videosändningar
    - bildtexter tillämpas på det första mediaobjektet vid sändning av svar med flera media
    - mediakällan kan vara HTTP(S), `file://` eller lokala sökvägar
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - gräns för sparning av inkommande media: `channels.whatsapp.mediaMaxMb` (standard `50`)
    - gräns för utgående media vid autosvar: `agents.defaults.mediaMaxMb` (standard `5MB`)
    - bilder optimeras automatiskt (ändrad storlek/kvalitetssvep) för att hålla sig inom gränserna
    - vid fel vid mediasändning skickas en textvarning som fallback för första objektet istället för att svaret tyst tappas bort
</Accordion>
</AccordionGroup>

## Bekräftelsereaktioner

**Konfiguration:**

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

**Alternativ:**

- skickas omedelbart efter att inkommande har accepterats (före svar)
- `direct` (boolesk, standard: `true`): Skicka reaktioner i direkt-/DM‑chattar.
- `group` (sträng, standard: `"mentions"`): Beteende i gruppchattar:
- WhatsApp använder `channels.whatsapp.ackReaction` (äldre `messages.ackReaction` används inte här)

## Flerkonton och autentiseringsuppgifter

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - konto-id:n kommer från `channels.whatsapp.accounts`
    - val av standardkonto: `default` om det finns, annars första konfigurerade konto-id (sorterat)
    - konto-id:n normaliseras internt för uppslagning
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - nuvarande autentiseringssökväg: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - säkerhetskopia: `creds.json.bak`
    - äldre standardautentisering i `~/.openclaw/credentials/` känns fortfarande igen/migreras för standardkontoflöden
  
</Accordion>

  <Accordion title="Logout behavior">`openclaw channels logout --channel whatsapp [--account <id>]` rensar WhatsApp-autentiseringstillståndet för det kontot.

    ```
    I äldre autentiseringskataloger bevaras `oauth.json` medan Baileys-autentiseringsfiler tas bort.
    ```

  
</Accordion>
</AccordionGroup>

## Begränsningar

- Utgående text delas upp till `channels.whatsapp.textChunkLimit` (standard 4000).
- Valfri radbrytnings‑chunkning: sätt `channels.whatsapp.chunkMode="newline"` för att dela på tomma rader (styckegränser) före längd‑chunkning.
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Sparade inkommande media begränsas av `channels.whatsapp.mediaMaxMb` (standard 50 MB).

## Utgående sändning (text + media)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    Symptom: kanalstatus rapporteras som inte länkad.

    ````
    Åtgärd:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Symptom: länkat konto med upprepade frånkopplingar eller återanslutningsförsök.

    ````
    Åtgärd:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    Vid behov, länka om med `channels login`.
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    Utgående sändningar misslyckas omedelbart om ingen aktiv gateway-lyssnare finns för målkontot.

    ```
    Se till att gateway körs och att kontot är länkat.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Kontrollera i denna ordning:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist-poster
    - omnämnandekrav ("mention gating") (`requireMention` + omnämnandemönster)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway-körningen bör använda Node. Bun är markerat som inkompatibelt för stabil WhatsApp-/Telegram-gatewaydrift.
  
</Accordion>
</AccordionGroup>

## Referenser för konfiguration

Primär referens:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Viktiga WhatsApp-fält:

- åtkomst: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- leverans: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- flerkonton: `accounts.<id> .enabled`, `accounts.<id> .authDir`, kontospecifika åsidosättningar
- drift: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- sessionsbeteende: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id> .historyLimit`

## Relaterat

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
