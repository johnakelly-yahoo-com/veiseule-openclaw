---
summary: "Äldre iMessage stöd via imsg (JSON-RPC över stdio). Nya inställningar bör använda BlueBubbles."
read_when:
  - Konfigurering av iMessage-stöd
  - Felsökning av iMessage sändning/mottagning
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
För nya iMessage-distributioner, använd <a href="/channels/bluebubbles">BlueBubbles</a>.

`imsg`-integrationen är föråldrad och kan tas bort i en framtida version. 
</Warning>

Status: äldre extern CLI-integration. Gateway startar `imsg rpc` och kommunicerar via JSON-RPC över stdio (ingen separat daemon/port).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Föredragen iMessage-väg för nya installationer.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage-DM använder parningsläge som standard.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Fullständig fältreferens för iMessage.
  
</Card>
</CardGroup>

## Snabbstart

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="Konfigurera OpenClaw">

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
      
        <Step title="Starta gateway">

```bash
openclaw gateway
```

      {
        channels: { imessage: { configWrites: false } },
      }

```bash
openclaw pairing list imessage
openclaw pairing approve imessage <CODE>
```

        ```
            Parningsförfrågningar upphör att gälla efter 1 timme.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw kräver endast en stdio-kompatibel `cliPath`, så du kan peka `cliPath` på ett wrapper-skript som SSH:ar till en fjärr-Mac och kör `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Rekommenderad konfiguration när bilagor är aktiverade:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // används för SCP-hämtning av bilagor
      includeAttachments: true,
    },
  },
}
```

    ```
    Om `remoteHost` inte är angiven försöker OpenClaw att automatiskt identifiera den genom att tolka SSH-wrapper-skriptet.
    ```

  
</Tab>
</Tabs>

## Krav och behörigheter (macOS)

- Messages måste vara inloggat på den Mac som kör `imsg`.
- Full skivåtkomst krävs för processkontexten som kör OpenClaw/`imsg` (åtkomst till Messages-databasen).
- Automationsbehörighet krävs för att skicka meddelanden via Messages.app.

<Tip>
Behörigheter beviljas per processkontext. Om gateway körs headless (LaunchAgent/SSH), kör ett engångskommando interaktivt i samma kontext för att utlösa behörighetsdialoger:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Åtkomstkontroll och routning

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` styr direktmeddelanden:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kräver att `allowFrom` inkluderar `"*"`)
    - `disabled`
    
    Allowlist-fält: `channels.imessage.allowFrom`.
    
    Poster i allowlist kan vara handles eller chattmål (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` styr hantering av grupper:

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
    - DM använder direkt routning; grupper använder grupproutning.
    - Med standardinställningen `session.dmScope=main` slås iMessage-DM ihop till agentens huvudsession.
    - Gruppsessioner är isolerade (`agent:<agentId> :imessage:group:<chat_id>`).
    - Svar routas tillbaka till iMessage med metadata för ursprunglig kanal/mål.

    ```
    Grupp-liknande trådbeteende:
    
    Vissa iMessage-trådar med flera deltagare kan komma in med `is_group=false`.
    Om det `chat_id` uttryckligen är konfigurerat under `channels.imessage.groups` behandlar OpenClaw det som grupptrafik (grupp-gating + isolering av gruppsession).
    ```

  
</Tab>
</Tabs>

## Distributionsmönster

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Använd ett dedikerat Apple-ID och en separat macOS-användare så att bottrafik är isolerad från din personliga Messages-profil.

    ```
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

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Vanlig topologi:

    ```
    - gateway körs på Linux/VM
    - iMessage + `imsg` körs på en Mac i ditt tailnet
    - `cliPath`-wrapper använder SSH för att köra `imsg`
    - `remoteHost` möjliggör hämtning av bilagor via SCP
    
    Exempel:
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
    Använd SSH-nycklar så att både SSH och SCP är icke-interaktiva.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage stöder kontospecifik konfiguration under `channels.imessage.accounts`.

    ```
    Varje konto kan åsidosätta fält som `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` och historikinställningar.
    ```

  
</Accordion>
</AccordionGroup>

## Media, chunkning och leveransmål

<AccordionGroup>
  <Accordion title="Attachments and media">
    - inläsning av inkommande bilagor är valfri: `channels.imessage.includeAttachments`
    - fjärrsökvägar för bilagor kan hämtas via SCP när `remoteHost` är satt
    - maximal storlek för utgående media styrs av `channels.imessage.mediaMaxMb` (standard 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - gräns för textchunk: `channels.imessage.textChunkLimit` (standard 4000)
    - chunk-läge: `channels.imessage.chunkMode`
      - `length` (standard)
      - `newline` (delning med stycken först)
  
</Accordion>

  <Accordion title="Addressing formats">Föredragna explicita mål:

    ```
    - `chat_id:123` (rekommenderas för stabil routning)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle-mål stöds också:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Hur det fungerar (beteende)

iMessage tillåter som standard konfigurationsändringar initierade från kanalen (för `/config set|unset` när `commands.config: true`).

Inaktivera:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Felsökning

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Verifiera binären och RPC-stöd:

```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    {
      channels: {
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15555550123"],
          groups: {
            "42": { requireMention: false },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">
    Kontrollera:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - parkopplingsgodkännanden (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    Kontrollera:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - allowlist-beteende för `channels.imessage.groups`
    - konfiguration av omnämnandemönster (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    Kontrollera:

    ```
    - `channels.imessage.remoteHost`
    - SSH/SCP key auth from the gateway host
    - remote path readability on the Mac running Messages
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">Kör igen i en interaktiv GUI-terminal i samma användar-/sessionskontext och godkänn uppmaningarna:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Bekräfta att Full Disk Access + Automation är beviljade för den processkontext som kör OpenClaw/`imsg`.
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferenser

- `agents.list[].groupChat.mentionPatterns` (eller `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

