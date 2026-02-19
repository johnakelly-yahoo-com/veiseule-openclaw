---
summary: "Legacy iMessage-ondersteuning via imsg (JSON-RPC over stdio). Nieuwe installaties moeten BlueBubbles gebruiken."
read_when:
  - iMessage-ondersteuning instellen
  - Problemen oplossen bij iMessage verzenden/ontvangen
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
**Aanbevolen:** Gebruik [BlueBubbles](/channels/bluebubbles) voor nieuwe iMessage-installaties.

Het kanaal `imsg` is een legacy externe-CLI-integratie en kan in een toekomstige release worden verwijderd. 
</Warning>

Status: legacy externe CLI-integratie. De Gateway start `imsg rpc` (JSON-RPC over stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">Configureer iMessage en start de Gateway.
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">Koppeling is de standaard tokenuitwisseling voor iMessage-DM's.
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Volledige iMessage-veldreferentie.
  
</Card>
</CardGroup>

## Snelle installatie (beginner)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="OpenClaw configureren">

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
      
        <Step title="Gateway starten">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Eerste DM-koppeling goedkeuren (standaard dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Koppelingsverzoeken verlopen na 1 uur.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">Als je iMessage op een andere Mac wilt, stel `channels.imessage.cliPath` in op een wrapper die `imsg` uitvoert op de externe macOS-host via SSH. OpenClaw heeft alleen stdio nodig.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Aanbevolen configuratie wanneer bijlagen zijn ingeschakeld:
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
    Als `remoteHost` niet is ingesteld, probeert OpenClaw dit automatisch te detecteren door het SSH-commando in je wrapper-script te parseren.
    ```

  
</Tab>
</Tabs>

## Vereisten en machtigingen (macOS)

- `channels.imessage.cliPath`: pad naar `imsg`.
- Volledige schijftoegang voor OpenClaw + `imsg` (toegang tot de Berichten-database).
- Automatiseringsmachtiging is vereist om berichten te verzenden via Messages.app.

<Tip>
Machtigingen worden per procescontext verleend. Als de gateway headless draait (LaunchAgent/SSH), voer dan een eenmalige interactieve opdracht uit in diezelfde context om de prompts te activeren:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Toegangsbeheer en routering

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` beheert directe berichten:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (standaard: allowlist).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: groepsafzender-toegestane lijst.

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
    - DM's gebruiken directe routering; groepen gebruiken groepsroutering.
    - Met de standaardinstelling `session.dmScope=main` worden iMessage-DM's samengevoegd in de hoofdsessie van de agent.
    DM's delen de hoofdsessie van de agent; groepen zijn geïsoleerd (`agent:<agentId>:imessage:group:<chat_id>`).<agentId>Groepen:<chat_id>`).
    - Antwoorden worden teruggerouteerd naar iMessage met behulp van metadata van het oorspronkelijke kanaal/doel.

    ```
    Als een thread met meerdere deelnemers binnenkomt met `is_group=false`, kun je deze alsnog isoleren door `chat_id` te gebruiken met `channels.imessage.groups` (zie “Group-ish threads” hieronder).
    ```

  
</Tab>
</Tabs>

## Implementatiepatronen

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Open Berichten in die macOS-gebruiker en meld je aan bij iMessage met de bot-Apple ID.

    ```
    Laat `channels.imessage.accounts.bot.cliPath` verwijzen naar een SSH-wrapper die `imsg` uitvoert als de bot-gebruiker.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Veelvoorkomende topologie:


    ```
    Als de Gateway op een Linux-host/VM draait maar iMessage op een Mac moet draaien, is Tailscale de eenvoudigste brug: de Gateway praat met de Mac via het tailnet, voert `imsg` uit via SSH en haalt bijlagen terug via SCP.
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
    Gebruik SSH-sleutels zodat zowel SSH als SCP niet-interactief zijn.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">iMessage-kanaal ondersteund door `imsg` op macOS.

    ```
    Elk account kan velden overschrijven zoals `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` en geschiedenisinstellingen.
    ```

  
</Accordion>
</AccordionGroup>

## Media, chunking en afleverdoelen

<AccordionGroup>
  <Accordion title="Attachments and media">Media-uploads zijn beperkt door `channels.imessage.mediaMaxMb` (standaard 16).
</Accordion>

  <Accordion title="Outbound chunking">Optionele newline-chunking: stel `channels.imessage.chunkMode="newline"` in om te splitsen op lege regels (paragraafgrenzen) vóór lengte-chunking.
</Accordion>

  <Accordion title="Addressing formats">
    Voorkeursdoelen expliciet opgeven:


    ```
    - `chat_id:123` (aanbevolen voor stabiele routering)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle-doelen worden ook ondersteund:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Minimale config:

Standaard mag iMessage config-updates wegschrijven die worden getriggerd door `/config set|unset` (vereist `commands.config: true`).

Uitschakelen met:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Probleemoplossing

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Valideer de binary en RPC-ondersteuning:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Als de probe meldt dat RPC niet wordt ondersteund, werk dan `imsg` bij.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Checklist:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (standaard: pairing).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Goedkeuren via:

    ```
    `channels.imessage.groupAllowFrom` bepaalt wie in groepen kan triggeren wanneer `allowlist` is ingesteld.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Notities:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">Voer eenmalig een interactieve opdracht uit in een GUI-terminal om de prompt af te dwingen en probeer het daarna opnieuw:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Bevestig dat Full Disk Access + Automation zijn verleend voor de procescontext die OpenClaw/`imsg` uitvoert.
    ```

  
</Accordion>
</AccordionGroup>

## Config-wegschrijvingen

- Configuratiereferentie (iMessage)
- Volledige configuratie: [Configuratie](/gateway/configuration)
- Details: [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
