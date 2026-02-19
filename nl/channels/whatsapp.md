---
summary: "WhatsApp-kanaalondersteuning, toegangscontroles, leveringsgedrag en operaties"
read_when:
  - Werken aan gedrag van het WhatsApp/webkanaal of inboxroutering
title: "WhatsApp"
---

# WhatsApp (webkanaal)

Status: Alleen WhatsApp Web via Baileys. De Gateway beheert de sessie(s).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Het standaard DM-beleid is **koppelen**, dus onbekende afzenders krijgen alleen een koppelcode en hun bericht wordt **niet verwerkt**.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Cross-channel diagnostiek en herstelplaybooks.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Volledige kanaalconfiguratiepatronen en voorbeelden.
  
</Card>
</CardGroup>

## Snelle installatie (beginner)

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
    Per account uitschakelen:
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
Goedkeuren met: `openclaw pairing approve whatsapp <code>` (lijst met `openclaw pairing list whatsapp`).
```

    ```
    Codes verlopen na 1 uur; openstaande verzoeken zijn beperkt tot 3 per kanaal.
    ```

  
</Step>
</Steps>

<Note>
Handig om je persoonlijke WhatsApp gescheiden te houden — installeer WhatsApp Business en registreer daar het OpenClaw-nummer. (De kanaalmetadata en onboardingflow zijn geoptimaliseerd voor die setup, maar configuraties met een persoonlijk nummer worden ook ondersteund.)
</Note>

## Implementatiepatronen

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Dit is de meest overzichtelijke operationele modus:


    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Onboarding ondersteunt de modus met persoonlijk nummer en schrijft een zelfchat-vriendelijke basisconfiguratie:


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

  <Accordion title="WhatsApp Web-only channel scope">Het messagingplatform-kanaal is WhatsApp Web-gebaseerd (`Baileys`) in de huidige OpenClaw-kanaalarchitectuur.

    ```
    Er is geen apart Twilio WhatsApp-messagingkanaal in het ingebouwde chat-channel-register.
    ```

  
</Accordion>
</AccordionGroup>

## Runtime-model

- Gateway beheert de WhatsApp-socket en de reconnect-loop.
- Voor uitgaande verzendingen is een actieve WhatsApp-listener voor het doelaccount vereist.
- Status-/broadcastchats worden genegeerd.
- Directe chats gebruiken DM-sessieregels (`session.dmScope`; standaard `main` voegt DM's samen in de hoofdsessie van de agent).
- Groepen mappen naar `agent:<agentId>:whatsapp:group:<jid>`-sessies.

## Toegangscontrole en activatie

<Tabs>
  <Tab title="DM policy">**DM-beleid**: `channels.whatsapp.dmPolicy` bepaalt toegang tot directe chats (standaard: `pairing`).

    ```
    Groepsbeleid: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (standaard `allowlist`).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Groepstoegang heeft twee lagen:

    ```
    `channels.whatsapp.groupAllowFrom` (toegestane lijst voor groepsafzenders).
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Groepsantwoorden vereisen standaard een vermelding.

    ```
    Vermeldingsdetectie omvat:
    
    - expliciete WhatsApp-vermeldingen van de bot-identiteit
    - geconfigureerde mention-regexpatronen (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - impliciete reply-naar-bot-detectie (reply-afzender komt overeen met de bot-identiteit)
    
    Activatiecommando op sessieniveau:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` werkt de sessiestatus bij (geen globale configuratie). Het is beperkt tot de eigenaar.
    ```

  
</Tab>
</Tabs>

## Gedrag van persoonlijk nummer en zelfchat

Wanneer het gekoppelde eigen nummer ook aanwezig is in `allowFrom`, worden WhatsApp-zelfchatbeveiligingen geactiveerd:

- leesbevestigingen overslaan voor zelfchat-beurten
- mention-JID auto-triggergedrag negeren dat je anders jezelf zou laten pingen
- Zelf-chatantwoorden gebruiken standaard `[{identity.name}]` wanneer ingesteld (anders `[openclaw]`)
  als `messages.responsePrefix` niet is ingesteld.

## Berichtnormalisatie en context

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Binnenkomende WhatsApp-berichten worden verpakt in de gedeelde inbound-envelop.

    ````
    Als er een geciteerde reply bestaat, wordt context in deze vorm toegevoegd:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Reply-metadata-velden worden ook ingevuld wanneer beschikbaar (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, afzender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Inkomende berichten met alleen media gebruiken placeholders:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Locatie- en contactpayloads worden genormaliseerd naar tekstuele context vóór routering.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Voor groepen kunnen onverwerkte berichten worden gebufferd en als context worden ingevoegd wanneer de bot uiteindelijk wordt getriggerd.

    ```
    Recente _niet-verwerkte_ berichten (standaard 50) ingevoegd onder:
    `[Chat messages since your last reply - for context]` (berichten die al in de sessie staan worden niet opnieuw geïnjecteerd)
    ```

  
</Accordion>

  <Accordion title="Read receipts">Standaard markeert de gateway inkomende WhatsApp-berichten als gelezen (blauwe vinkjes) zodra ze zijn geaccepteerd.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Levering, chunking en media

<AccordionGroup>
  <Accordion title="Text chunking">Optioneel splitsen op nieuwe regels: stel `channels.whatsapp.chunkMode="newline"` in om op lege regels (paragraafgrenzen) te splitsen vóór lengte-opknippen.
</Accordion>

  <Accordion title="Outbound media behavior">
    - ondersteunt image-, video-, audio- (PTT-spraakbericht) en documentpayloads
    - `audio/ogg` wordt herschreven naar `audio/ogg; codecs=opus` voor compatibiliteit met spraakberichten
    - geanimeerde GIF-weergave wordt ondersteund via `gifPlayback: true` bij het verzenden van video
    - bijschriften worden toegepast op het eerste media-item bij het verzenden van multi-media reply-payloads
    - mediabron kan HTTP(S), `file://` of lokale paden zijn
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - limiet voor opslaan van inkomende media: `channels.whatsapp.mediaMaxMb` (standaard `50`)
    - limiet voor uitgaande media voor automatische antwoorden: `agents.defaults.mediaMaxMb` (standaard `5MB`)
    - afbeeldingen worden automatisch geoptimaliseerd (resize/kwaliteitsaanpassing) om binnen limieten te passen
    - bij mislukken van mediaverzending stuurt een fallback van het eerste item een tekstwaarschuwing in plaats van het antwoord stilzwijgend te laten vallen
  
</Accordion>
</AccordionGroup>

## Bevestigingsreacties

`channels.whatsapp.ackReaction` (auto-reactie bij ontvangst van berichten: `{emoji, direct, group}`).

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

Notities:

- direct verzonden nadat inbound is geaccepteerd (vóór antwoord)
- fouten worden gelogd maar blokkeren de normale antwoordlevering niet
- groepsmodus `mentions` reageert op door vermelding getriggerde beurten; groepsactivatie `always` fungeert als bypass voor deze controle
- WhatsApp negeert `messages.ackReaction`; gebruik `channels.whatsapp.ackReaction` in plaats daarvan.

## Multi-account en inloggegevens

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (per-accountinstellingen + optioneel `authDir`).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Referenties opgeslagen in `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.<accountId>/creds.json`
    - back-upbestand: `creds.json.bak`
    - legacy standaardauthenticatie in `~/.openclaw/credentials/` wordt nog steeds herkend/gemigreerd voor default-accountflows
  
</Accordion>

  <Accordion title="Logout behavior">Multi-account inloggen: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>Uitloggen: `openclaw channels logout` (of `--account <id>`) verwijdert de WhatsApp-authenticatiestatus (maar behoudt gedeelde `oauth.json`).

    ```
    In legacy-authenticatiemap­pen wordt `oauth.json` behouden terwijl Baileys-authbestanden worden verwijderd.
    ```

  
</Accordion>
</AccordionGroup>

## Tools, acties en configuratieschrijfacties

- Tool: `whatsapp` met actie `react` (`chatJid`, `messageId`, `emoji`, optioneel `remove`).
- Actiepoorten:
  - `channels.whatsapp.actions.reactions` (gate WhatsApp-toolreacties).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Problemen oplossen (snel)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Symptoom: `channels status` toont `linked: false` of waarschuwt “Not linked”.

    ```
    {
      channels: { whatsapp: { sendReadReceipts: false } },
    }
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Symptoom: gekoppeld account met herhaalde verbrekingen of herverbindingspogingen.

    ```
    Oplossing: `openclaw doctor` (of herstart de gateway). Als het aanhoudt, koppel opnieuw via `channels login` en inspecteer `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">Uitgaande verzendingen mislukken direct wanneer er geen actieve gateway-listener bestaat voor het doelaccount.

    ```
    Zorg ervoor dat de gateway draait en het account is gekoppeld.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Controleer in deze volgorde:

    ```
    `channels.whatsapp.groups` (groeps-toegestane lijst + mention-gating-standaarden; gebruik `"*"` om alles toe te staan)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp-gatewayruntime moet Node gebruiken. Bun is gemarkeerd als incompatibel voor stabiele WhatsApp-/Telegram-gatewaywerking.
  
</Accordion>
</AccordionGroup>

## Config-wegschrijvingen

Primaire referentie:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Belangrijke WhatsApp-velden:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- levering: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>``.enabled`, `accounts.<id>``.authDir`, overschrijvingen op accountniveau
- bewerkingen: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- sessiegedrag: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Gerelateerd

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- Probleemoplossingsgids: [Gateway troubleshooting](/gateway/troubleshooting).
