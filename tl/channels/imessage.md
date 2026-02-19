---
summary: "46. Legacy na suporta sa iMessage sa pamamagitan ng imsg (JSON-RPC sa stdio). 47. Ang mga bagong setup ay dapat gumamit ng BlueBubbles."
read_when:
  - Pagse-setup ng suporta sa iMessage
  - Pag-debug ng pagpapadala/pagtanggap ng iMessage
title: "iMessage"
---

# iMessage (legacy: imsg)

<Warning>
Para sa mga bagong iMessage deployment, gamitin ang <a href="/channels/bluebubbles">BlueBubbles</a>.

Ang `imsg` integration ay legacy at maaaring alisin sa susunod na release. 
</Warning>

48. Status: legacy external CLI integration. Ang Gateway ay naglulunsad ng `imsg rpc` at nakikipag-ugnayan sa pamamagitan ng JSON-RPC sa stdio (walang hiwalay na daemon/port).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Inirerekomendang iMessage path para sa mga bagong setup.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Ang mga iMessage DM ay default sa pairing mode.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Buong reference ng mga field ng iMessage.
  
</Card>
</CardGroup>

## Mabilisang setup

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="I-configure ang OpenClaw">

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
      
        <Step title="Simulan ang gateway">

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
            Ang mga pairing request ay nag-e-expire pagkalipas ng 1 oras.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    Ang OpenClaw ay nangangailangan lamang ng stdio-compatible na `cliPath`, kaya maaari mong ituro ang `cliPath` sa isang wrapper script na nag-SSH sa isang remote Mac at nagpapatakbo ng `imsg`.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Inirerekomendang config kapag naka-enable ang attachments:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // ginagamit para sa SCP attachment fetches
      includeAttachments: true,
    },
  },
}
```

    ```
    Kung hindi naka-set ang `remoteHost`, susubukan ng OpenClaw na awtomatikong tukuyin ito sa pamamagitan ng pag-parse ng SSH wrapper script.
    ```

  
</Tab>
</Tabs>

## Mga kinakailangan at pahintulot (macOS)

- Kailangang naka-sign in ang Messages sa Mac na nagpapatakbo ng `imsg`.
- Kinakailangan ang Full Disk Access para sa process context na nagpapatakbo ng OpenClaw/`imsg` (access sa Messages DB).
- Kinakailangan ang pahintulot sa Automation upang makapagpadala ng mga mensahe sa pamamagitan ng Messages.app.

<Tip>
Ang mga pahintulot ay ibinibigay kada process context. Kung tumatakbo ang gateway nang headless (LaunchAgent/SSH), magpatakbo ng isang beses na interactive na command sa parehong context upang ma-trigger ang mga prompt:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Access control at routing

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` kinokontrol ang mga direktang mensahe:

    ```
    - `pairing` (default)
    - `allowlist`
    - `open` (kinakailangang kasama sa `allowFrom` ang `"*"`)
    - `disabled`
    
    Allowlist field: `channels.imessage.allowFrom`.
    
    Maaaring mga handle o chat target ang mga entry sa allowlist (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` ang kumokontrol sa paghawak ng mga group:

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
    - Gumagamit ng direct routing ang mga DM; gumagamit ng group routing ang mga group.
    - Sa default na `session.dmScope=main`, ang iMessage DMs ay pinagsasama sa main session ng agent.
    - Ang mga group session ay naka-isolate (`agent:<agentId>:imessage:group:<chat_id>`).
    - Ang mga reply ay niruruta pabalik sa iMessage gamit ang metadata ng pinanggalingang channel/target.

    ```
    Pag-uugali ng group-ish thread:
    
    May ilang multi-participant na iMessage thread na maaaring dumating na may `is_group=false`.
    Kung ang `chat_id` na iyon ay tahasang naka-configure sa ilalim ng `channels.imessage.groups`, ituturing ito ng OpenClaw bilang group traffic (group gating + group session isolation).
    ```

  
</Tab>
</Tabs>

## Mga pattern ng deployment

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Gumamit ng hiwalay na Apple ID at macOS user upang maihiwalay ang trapiko ng bot mula sa iyong personal na Messages profile.

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
    Karaniwang topology:

    ```
    - tumatakbo ang gateway sa Linux/VM
    - tumatakbo ang iMessage + `imsg` sa isang Mac sa iyong tailnet
    - ang `cliPath` wrapper ay gumagamit ng SSH upang patakbuhin ang `imsg`
    - pinapagana ng `remoteHost` ang mga SCP attachment fetch
    
    Halimbawa:
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
    Gamitin ang SSH keys upang maging hindi nangangailangan ng interaksyon ang parehong SSH at SCP.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    Sinusuportahan ng iMessage ang per-account na config sa ilalim ng `channels.imessage.accounts`.

    ```
    Maaaring mag-override ang bawat account ng mga field tulad ng `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, at mga setting ng history.
    ```

  
</Accordion>
</AccordionGroup>

## Media, chunking, at mga delivery target

<AccordionGroup>
  <Accordion title="Attachments and media">
    - opsyonal ang inbound attachment ingestion: `channels.imessage.includeAttachments`
    - maaaring kunin ang mga remote attachment path sa pamamagitan ng SCP kapag naka-set ang `remoteHost`
    - ginagamit ng outbound media size ang `channels.imessage.mediaMaxMb` (default 16 MB)
  
</Accordion>

  <Accordion title="Outbound chunking">
    - limit ng text chunk: `channels.imessage.textChunkLimit` (default 4000)
    - chunk mode: `channels.imessage.chunkMode`
      - `length` (default)
      - `newline` (unang hinahati ayon sa talata)
  
</Accordion>

  <Accordion title="Addressing formats">
    Mga mas mainam na tahasang target:

    ```
    - `chat_id:123` (inirerekomenda para sa stable na routing)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Sinusuportahan din ang mga handle target:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Config writes

Pinapayagan ng iMessage ang channel-initiated na pagsusulat ng config bilang default (para sa `/config set|unset` kapag `commands.config: true`).

I-disable:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## Pag-troubleshoot

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    I-validate ang binary at suporta ng RPC:

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
    Suriin:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - mga pairing approval (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    Suriin:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - allowlist behavior ng `channels.imessage.groups`
    - configuration ng mention pattern (`agents.list[].groupChat.mentionPatterns`)
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">    Suriin:

    ```
    - `channels.imessage.remoteHost`
    - SSH/SCP key auth mula sa gateway host
    - pagiging readable ng remote path sa Mac na nagpapatakbo ng Messages
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">    Patakbuhin muli sa isang interactive GUI terminal sa parehong user/session context at aprubahan ang mga prompt:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Tiyaking ang Full Disk Access + Automation ay naibigay para sa process context na nagpapatakbo ng OpenClaw/`imsg`.
    ```

  
</Accordion>
</AccordionGroup>

## Mga sanggunian sa configuration

- `agents.list[].groupChat.mentionPatterns` (o `messages.groupChat.mentionPatterns`).
- `messages.responsePrefix`.
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
