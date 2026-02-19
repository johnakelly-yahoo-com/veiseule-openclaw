---
summary: "Legacy iMessage support via imsg (JSON-RPC over stdio). Nye opsætninger bør bruge BlueBubbles."
read_when:
  - Opsætning af iMessage-understøttelse
  - Fejlfinding af iMessage send/modtag
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
For nye iMessage-implementeringer skal du bruge <a href="/channels/bluebubbles">BlueBubbles</a>.

`imsg`-integrationen er legacy og kan blive fjernet i en fremtidig udgivelse. 
</Warning>

Status: ældre ekstern CLI integration. Gateway starter `imsg rpc` og kommunikerer via JSON-RPC over stdio (ingen separat daemon/port).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Foretrukken iMessage-sti til nye opsætninger.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage-DM'er bruger som standard parrings-tilstand.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Fuld iMessage-feltreference.
  
</Card>
</CardGroup>

## Hurtig opsætning

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="Konfigurer OpenClaw">

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
      
        <Step title="Start gateway">

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
            Parringsanmodninger udløber efter 1 time.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw kræver kun en stdio-kompatibel `cliPath`, så du kan pege `cliPath` på et wrapper-script, der SSH'er til en fjern-Mac og kører `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Anbefalet konfiguration, når vedhæftede filer er aktiveret:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // bruges til SCP-hentning af vedhæftede filer
      includeAttachments: true,
    },
  },
}
```

    ```
    Hvis `remoteHost` ikke er angivet, forsøger OpenClaw automatisk at registrere den ved at parse SSH-wrapper-scriptet.
    ```

  
</Tab>
</Tabs>

## Krav og tilladelser (macOS)

- Beskeder skal være logget ind på den Mac, der kører `imsg`.
- Fuld diskadgang er påkrævet for den proceskontekst, der kører OpenClaw/`imsg` (adgang til Messages-databasen).
- Automation-tilladelse er påkrævet for at sende beskeder via Messages.app.

<Tip>
Tilladelser gives pr. proceskontekst. Hvis gatewayen kører headless (LaunchAgent/SSH), skal du køre en engangs interaktiv kommando i samme kontekst for at udløse tilladelsesprompter:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Adgangskontrol og routing

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` styrer direkte beskeder:

    ```
    - `pairing` (standard)
    - `allowlist`
    - `open` (kræver at `allowFrom` indeholder `"*"`)
    - `disabled`
    
    Allowlist-felt: `channels.imessage.allowFrom`.
    
    Allowlist-poster kan være handles eller chatmål (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` styrer håndtering af grupper:

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
    - DMs bruger direkte routing; grupper bruger grupperouting.
    - Med standardindstillingen `session.dmScope=main` samles iMessage-DMs i agentens hovedsession.
    - Gruppesessioner er isolerede (`agent:<agentId> :imessage:group:<chat_id>`).
    - Svar routes tilbage til iMessage ved hjælp af metadata fra den oprindelige kanal/det oprindelige mål.

    ```
    Gruppe-lignende trådadfærd:
    
    Nogle iMessage-tråde med flere deltagere kan ankomme med `is_group=false`.
    Hvis den `chat_id` eksplicit er konfigureret under `channels.imessage.groups`, behandler OpenClaw den som gruppetrafik (gruppe-gating + isolation af gruppesession).
    ```

  
</Tab>
</Tabs>

## Implementeringsmønstre

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Brug et dedikeret Apple ID og en macOS-bruger, så bottrafik er isoleret fra din personlige Messages-profil.

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
    Almindelig topologi:

    ```
    - gateway kører på Linux/VM
    - iMessage + `imsg` kører på en Mac i dit tailnet
    - `cliPath`-wrapper bruger SSH til at køre `imsg`
    - `remoteHost` aktiverer SCP-hentning af vedhæftninger
    
    Eksempel:
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
    Brug SSH-nøgler, så både SSH og SCP er ikke-interaktive.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage understøtter konfiguration pr. konto under `channels.imessage.accounts`.

    ```
    Hver konto kan tilsidesætte felter som `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` og historikindstillinger.
    ```

  
</Accordion>
</AccordionGroup>

## Medier, chunking og leveringsmål

<AccordionGroup>
  <Accordion title="Attachments and media">
    - indgående indlæsning af vedhæftninger er valgfri: `channels.imessage.includeAttachments`
    - eksterne stier til vedhæftninger kan hentes via SCP, når `remoteHost` er angivet
    - udgående mediestørrelse bruger `channels.imessage.mediaMaxMb` (standard 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - tekst-chunkgrænse: `channels.imessage.textChunkLimit` (standard 4000)
    - chunk-tilstand: `channels.imessage.chunkMode`
      - `length` (standard)
      - `newline` (opdeling med afsnit først)
  
</Accordion>

  <Accordion title="Addressing formats">
    Foretrukne eksplicitte mål:

    ```
    - `chat_id:123` (anbefales for stabil routing)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle-mål understøttes også:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Sådan virker det (adfærd)

iMessage tillader som standard kanal-initierede konfigurationsskrivninger (for `/config set|unset`, når `commands.config: true`).

Deaktiver:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Fejlfinding

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Valider binæren og RPC-understøttelse:

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
    Tjek:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - pairing-godkendelser (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    Tjek:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - allowlist-adfærd for `channels.imessage.groups`
    - konfiguration af mention-mønstre (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    Tjek:

    ```
    - `channels.imessage.remoteHost`
    - SSH/SCP-nøgleautentificering fra gateway-værten
    - læseadgang til fjernstien på den Mac, der kører Messages
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    Kør igen i en interaktiv GUI-terminal i samme bruger-/sessionskontekst og godkend forespørgsler:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Bekræft, at Full Disk Access + Automation er givet til den proceskontekst, der kører OpenClaw/`imsg`.
    ```

  
</Accordion>
</AccordionGroup>

## Konfigurationsreferencepunkter

- `agents.list[].groupChat.mentionPatterns` (eller `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

