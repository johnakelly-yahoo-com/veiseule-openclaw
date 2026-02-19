---
summary: "WhatsApp (webkanal) integration: login, indbakke, svar, medier og drift"
read_when:
  - Arbejde med WhatsApp/webkanal-adfærd eller routing i indbakken
title: "WhatsApp"
---

# WhatsApp (webkanal)

Status: produktionsklar via WhatsApp Web (Baileys). Gateway ejer tilknyttede session(er).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standard DM-politik er parring for ukendte afsendere.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Tværkanal-diagnostik og reparationsvejledninger.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Fuldstændige kanal-konfigurationsmønstre og eksempler.
  
</Card>
</CardGroup>

## Hurtig opsætning

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
    For en specifik konto:
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
    Parringsanmodninger udløber efter 1 time. Afventende anmodninger er begrænset til 3 pr. kanal.
    ```

  
</Step>
</Steps>

<Note>
OpenClaw anbefaler at køre WhatsApp på et separat nummer, når det er muligt. (Kanalens metadata og onboarding-flow er optimeret til den opsætning, men opsætninger med personligt nummer understøttes også.)
</Note>

## Implementeringsmønstre

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Dette er den reneste driftsmodel:

    ````
    - separat WhatsApp-identitet til OpenClaw
    - tydeligere DM-allowlists og routing-grænser
    - lavere risiko for forvirring ved selv-chat
    
    Minimal politikmønster:
    
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
    Onboarding understøtter tilstand med personligt nummer og skriver en selv-chat-venlig baseline:

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
    Messaging-platformkanalen er WhatsApp Web-baseret (`Baileys`) i den nuværende OpenClaw-kanalarkitektur.

    ```
    Der findes ikke en separat Twilio WhatsApp-beskedkanal i det indbyggede chat-kanalregister.
    ```

  
</Accordion>
</AccordionGroup>

## Runtime-model

- Gateway ejer WhatsApp-socketen og reconnect-loopet.
- Udgående afsendelser kræver en aktiv WhatsApp-lytter for målkontoen.
- Status- og broadcast-chats ignoreres (`@status`, `@broadcast`).
- Direkte chats bruger DM-sessionsregler (`session.dmScope`; standard `main` samler DMs i agentens hovedsession).
- Gruppesessioner er isolerede (`agent:<agentId>:whatsapp:group:<jid>`).

## Adgangskontrol og aktivering

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` styrer adgang til direkte chats:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kræver at `allowFrom` inkluderer `"*"`)
    - `disabled`
    
    `allowFrom` accepterer numre i E.164-format (normaliseres internt).
    
    Multi-konto-override: `channels.whatsapp.accounts.<id>.dmPolicy` (og `allowFrom`) har forrang over kanalniveauets standardindstillinger for den pågældende konto.
    
    Detaljer om runtime-adfærd:
    
    - pairings gemmes i kanalens allow-store og flettes med konfigureret `allowFrom`
    - hvis ingen allowlist er konfigureret, tillades det tilknyttede eget nummer som standard
    - udgående `fromMe` DMs parres aldrig automatisk
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Gruppeadgang har to lag:

    ```
    1. **Allowlist for gruppemedlemskab** (`channels.whatsapp.groups`)
       - hvis `groups` udelades, er alle grupper kvalificerede
       - hvis `groups` er angivet, fungerer den som en gruppe-allowlist (`"*"` tilladt)
    
    2. **Afsenderpolitik for grupper** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: afsender-allowlist omgås
       - `allowlist`: afsender skal matche `groupAllowFrom` (eller `*`)
       - `disabled`: bloker al indgående gruppetrafik
    
    Fallback for afsender-allowlist:
    
    - hvis `groupAllowFrom` ikke er angivet, falder runtime tilbage til `allowFrom`, når den er tilgængelig
    
    Bemærk: hvis der slet ikke findes en `channels.whatsapp`-blok, er runtime fallback for gruppepolitik reelt `open`.
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Gruppesvar kræver som standard en mention.

    ```
    Mention-detektion omfatter:
    
    - eksplicitte WhatsApp-mentions af bot-identiteten
    - konfigurerede mention-regexmønstre (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit svar-til-bot-detektion (svarafsender matcher bot-identiteten)
    
    Aktiveringskommando på sessionsniveau:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` opdaterer sessionstilstand (ikke global konfiguration). Den er owner-gated.
    ```

  
</Tab>
</Tabs>

## Adfærd for personligt nummer og selv-chat

Deaktivér globalt:

- spring læsekvitteringer over for selv-chat-beskeder
- ignorér auto-trigger-adfærd for mention-JID, som ellers ville pinge dig selv
- hvis `messages.responsePrefix` ikke er angivet, bruger selv-chat-svar som standard `[{identity.name}]` eller `[openclaw]`

## Beskednormalisering og kontekst

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Indgående WhatsApp-beskeder pakkes ind i den delte inbound envelope.

    ````
    Hvis der findes et citeret svar, tilføjes kontekst i denne form:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Metadatafelter for svar udfyldes også, når de er tilgængelige (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, afsender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Indgående beskeder kun med medie normaliseres med pladsholdere såsom:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Lokations- og kontaktpayloads normaliseres til tekstlig kontekst før routing.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    For grupper kan ubehandlede beskeder bufferes og injiceres som kontekst, når botten endelig trigges.

    ```
    - standardgrænse: `50`
    - konfiguration: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` deaktiverer
    
    Injektionsmarkører:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    Læsekvitteringer er som standard aktiveret for accepterede indgående WhatsApp-beskeder.

    ````
    Deaktiver globalt:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Override pr. konto:
    
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
    
    Selv-chat springer læsekvitteringer over, selv når de er globalt aktiveret.
    ````

  
</Accordion>
</AccordionGroup>

## Levering, chunking og medier

<AccordionGroup>
  <Accordion title="Text chunking">
    - standard chunk-grænse: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - `newline`-tilstand foretrækker afsnitsgrænser (blanke linjer) og falder derefter tilbage til længdesikker chunking
</Accordion>

  <Accordion title="Outbound media behavior">- understøtter image-, video-, audio- (PTT voice-note) og document-payloads
- `audio/ogg` omskrives til `audio/ogg; codecs=opus` for voice-note-kompatibilitet
- afspilning af animerede GIF’er understøttes via `gifPlayback: true` ved afsendelse af video
- billedtekster anvendes på det første medieelement ved afsendelse af multi-media reply-payloads
- mediekilden kan være HTTP(S), `file://` eller lokale stier
</Accordion>

  <Accordion title="Media size limits and fallback behavior">- grænse for lagring af indgående medier: `channels.whatsapp.mediaMaxMb` (standard `50`)
- grænse for udgående medier til autosvar: `agents.defaults.mediaMaxMb` (standard `5MB`)
- billeder optimeres automatisk (resize/quality sweep) for at overholde grænserne
- ved fejl i afsendelse af medier sendes en tekstadvarsel som fallback for første element i stedet for at svaret droppes lydløst
</Accordion>
</AccordionGroup>

## Bekræftelsesreaktioner

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

**Valgmuligheder:**

- sendes straks efter indgående besked er accepteret (før svar)
- `direct` (boolean, standard: `true`): Send reaktioner i direkte/DM-chats.
- `group` (string, standard: `"mentions"`): Gruppechat-adfærd:
- WhatsApp bruger `channels.whatsapp.ackReaction` (legacy `messages.ackReaction` bruges ikke her)

## Multi-account og legitimationsoplysninger

<AccordionGroup>
  <Accordion title="Account selection and defaults">- konto-id’er kommer fra `channels.whatsapp.accounts`
- valg af standardkonto: `default` hvis til stede, ellers første konfigurerede konto-id (sorteret)
- konto-id’er normaliseres internt til opslag
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">- nuværende auth-sti: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- backup-fil: `creds.json.bak`
- legacy standard-auth i `~/.openclaw/credentials/` genkendes/migreres stadig for default-account-flows
</Accordion>

  <Accordion title="Logout behavior">`openclaw channels logout --channel whatsapp [--account <id>]` rydder WhatsApp-auth-tilstanden for den konto.

    ```
    I legacy auth-mapper bevares `oauth.json`, mens Baileys-auth-filer fjernes.
    ```

  
</Accordion>
</AccordionGroup>

## Grænser

- Udgående tekst opdeles i bidder på `channels.whatsapp.textChunkLimit` (standard 4000).
- Valgfri linjeskifts-opdeling: sæt `channels.whatsapp.chunkMode="newline"` for at splitte på tomme linjer (afsnitsgrænser) før længdeopdeling.
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Indgående medie-gemninger er begrænset af `channels.whatsapp.mediaMaxMb` (standard 50 MB).

## Udgående afsendelse (tekst + medier)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Symptom: kanalstatus rapporterer ikke linked.

    ````
    Løsning:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">Symptom: linked konto med gentagne afbrydelser eller genforbindelsesforsøg.

    ````
    Løsning:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    Om nødvendigt, link igen med `channels login`.
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">Udgående afsendelser fejler straks, når der ikke findes en aktiv gateway listener for mål-kontoen.

    ```
    Sørg for, at gateway kører, og at kontoen er linked.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">Tjek i denne rækkefølge:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist-poster
    - mention-gating (`requireMention` + mention-mønstre)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway runtime skal bruge Node. Bun er markeret som inkompatibel for stabil WhatsApp/Telegram gateway-drift.
  
</Accordion>
</AccordionGroup>

## Konfigurationsreference-pegepinde

Primær reference:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

WhatsApp-felter med høj signalværdi:

- adgang: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- levering: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>.enabled`, `accounts.<id>.authDir`, konto-niveau overrides
- drift: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- session-adfærd: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## Relateret

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Fejlfinding](/channels/troubleshooting)
