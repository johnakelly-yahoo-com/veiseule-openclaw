---
summary: "Integrasyon ng WhatsApp (web channel): login, inbox, mga sagot, media, at ops"
read_when:
  - Kapag nagtatrabaho sa behavior ng WhatsApp/web channel o inbox routing
title: "WhatsApp"
---

# WhatsApp (web channel)

Status: production-ready sa pamamagitan ng WhatsApp Web (Baileys). Ang Gateway ang may kontrol sa naka-link na session(s).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Default na DM policy ay pairing para sa mga hindi kilalang sender.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Mga diagnostic at repair playbook para sa cross-channel.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Kumpletong mga pattern at halimbawa ng channel config.
  
</Card>
</CardGroup>

## Mabilisang setup

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
    Para sa isang partikular na account:
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
    Ang mga pairing request ay nag-e-expire pagkatapos ng 1 oras. Hanggang 3 pending request lamang ang pinapayagan bawat channel.
    ```

  
</Step>
</Steps>

<Note>
Inirerekomenda ng OpenClaw na gumamit ng hiwalay na numero para sa WhatsApp kung maaari. (Ang channel metadata at onboarding flow ay naka-optimize para sa setup na iyon, ngunit sinusuportahan din ang paggamit ng personal na numero.)
</Note>

## Mga pattern ng deployment

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Ito ang pinaka-malinis na operational mode:

    ````
    - hiwalay na WhatsApp identity para sa OpenClaw
    - mas malinaw na DM allowlists at mga hangganan ng routing
    - mas mababang posibilidad ng kalituhan sa self-chat
    
    Minimal na policy pattern:
    
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
    Sinusuportahan ng onboarding ang personal-number mode at nagsusulat ng baseline na akma sa self-chat:

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
    Ang messaging platform channel ay nakabase sa WhatsApp Web (`Baileys`) sa kasalukuyang OpenClaw channel architecture.

    ```
    Walang hiwalay na Twilio WhatsApp messaging channel sa built-in chat-channel registry.
    ```

  
</Accordion>
</AccordionGroup>

## Runtime model

- Ang Gateway ang may kontrol sa WhatsApp socket at reconnect loop.
- Ang mga outbound send ay nangangailangan ng aktibong WhatsApp listener para sa target na account.
- Hindi pinoproseso ang mga status at broadcast chat (`@status`, `@broadcast`).
- Ang mga direct chat ay gumagamit ng DM session rules (`session.dmScope`; ang default na `main` ay pinagsasama ang mga DM sa main session ng agent).
- Ang mga group session ay hiwalay (`agent:<agentId>:whatsapp:group:<jid>`).

## Access control at activation

<Tabs>
  <Tab title="DM policy">
    Kinokontrol ng `channels.whatsapp.dmPolicy` ang access sa direct chat:

    ```
    - `pairing` (default)
    - `allowlist`
    - `open` (kinakailangang isama ng `allowFrom` ang `"*"`)
    - `disabled`
    
    Tumatanggap ang `allowFrom` ng mga numerong nasa E.164 format (ina-normalize sa loob ng system).
    
    Multi-account override: Ang `channels.whatsapp.accounts.<id>.dmPolicy` (at `allowFrom`) ay may mas mataas na prioridad kaysa sa mga default sa antas ng channel para sa account na iyon.
    
    Mga detalye ng runtime behavior:
    
    - ang mga pairing ay sine-save sa channel allow-store at pinagsasama sa naka-configure na `allowFrom`
    - kung walang naka-configure na allowlist, awtomatikong pinapayagan ang naka-link na sariling numero
    - ang mga outbound `fromMe` DM ay hindi kailanman awtomatikong pini-pair
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Ang group access ay may dalawang layer:

    ```
    1. **Group membership allowlist** (`channels.whatsapp.groups`)
       - kung hindi nakasaad ang `groups`, lahat ng group ay maaaring gamitin
       - kung may `groups`, ito ay nagsisilbing group allowlist (`"*"` ay pinapayagan)
    
    2. **Group sender policy** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: nilalaktawan ang sender allowlist
       - `allowlist`: kailangang tumugma ang sender sa `groupAllowFrom` (o `*`)
       - `disabled`: harangan ang lahat ng group inbound
    
    Fallback ng sender allowlist:
    
    - kung hindi nakatakda ang `groupAllowFrom`, babalik ang runtime sa `allowFrom` kapag available
    
    Tandaan: kung walang kahit anong `channels.whatsapp` block, ang fallback ng runtime group-policy ay epektibong `open`.
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Ang mga group reply ay nangangailangan ng mention bilang default.

    ```
    Kasama sa mention detection ang:
    
    - tahasang WhatsApp mentions ng bot identity
    - mga naka-configure na mention regex pattern (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit reply-to-bot detection (ang reply sender ay tumutugma sa bot identity)
    
    Session-level activation command:
    
    - `/activation mention`
    - `/activation always`
    
    Ina-update ng `activation` ang session state (hindi ang global config). Ito ay naka-owner-gate.
    ```

  
</Tab>
</Tabs>

## Pag-uugali ng personal-number at self-chat

I-disable nang global:

- laktawan ang read receipts para sa mga self-chat turn
- huwag pansinin ang mention-JID auto-trigger behavior na maaaring mag-ping sa sarili mo
- kung hindi nakatakda ang `messages.responsePrefix`, ang mga self-chat reply ay magde-default sa `[{identity.name}]` o `[openclaw]`

## Message normalization at context

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Ang mga papasok na WhatsApp message ay binabalot sa shared inbound envelope.

    ````
    Kung may quoted reply, idinadagdag ang context sa ganitong anyo:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Pinupunan din ang mga reply metadata field kapag available (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Ang mga media-only inbound message ay ni-no-normalize gamit ang mga placeholder tulad ng:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Ang mga location at contact payload ay kino-convert sa tekstuwal na context bago i-route.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Para sa mga group, maaaring i-buffer ang mga hindi pa naprosesong mensahe at ipasok bilang context kapag na-trigger na ang bot.

    ```
    - default na limit: `50`
    - config: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` ay nagdi-disable
    
    Injection markers:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    Ang read receipts ay naka-enable bilang default para sa mga tinanggap na inbound WhatsApp message.

    ````
    Disable globally:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Override kada account:
    
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
    
    Ang self-chat ay hindi nagpapadala ng read receipts kahit naka-enable ito sa global settings.
    ````

  
</Accordion>
</AccordionGroup>

## Paghahatid, paghahati (chunking), at media

<AccordionGroup>
  <Accordion title="Text chunking">
    - default na limit ng chunk: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - mas pinapaboran ng `newline` mode ang mga hangganan ng talata (mga blangkong linya), at kung hindi posible, babalik sa length-safe na paghahati
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - sumusuporta sa image, video, audio (PTT voice-note), at document payloads
    - ang `audio/ogg` ay nire-rewrite bilang `audio/ogg; codecs=opus` para sa compatibility ng voice-note
    - sinusuportahan ang animated GIF playback sa pamamagitan ng `gifPlayback: true` kapag nagpapadala ng video
    - ang mga caption ay inilalapat sa unang media item kapag nagpapadala ng multi-media reply payloads
    - maaaring manggaling ang media source sa HTTP(S), `file://`, o mga lokal na path
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - limit ng pag-save ng inbound media: `channels.whatsapp.mediaMaxMb` (default `50`)
    - limit ng outbound media para sa auto-replies: `agents.defaults.mediaMaxMb` (default `5MB`)
    - awtomatikong ino-optimize ang mga image (resize/quality sweep) upang magkasya sa mga limit
    - kapag nabigo ang pagpapadala ng media, ang fallback sa unang item ay magpapadala ng text warning sa halip na tahimik na hindi ipadala ang response
  
</Accordion>
</AccordionGroup>

## Mga acknowledgment reaction

**Configuration:**

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

**Mga opsyon:**

- ipinapadala agad pagkatapos matanggap ang inbound (bago ang reply)
- `direct` (boolean, default: `true`): Magpadala ng reactions sa direct/DM chats.
- `group` (string, default: `"mentions"`): Behavior sa group chat:
- Ginagamit ng WhatsApp ang `channels.whatsapp.ackReaction` (hindi ginagamit dito ang legacy na `messages.ackReaction`)

## Multi-account at credentials

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - ang mga account id ay nagmumula sa `channels.whatsapp.accounts`
    - pagpili ng default account: `default` kung mayroon, kung wala ay ang unang naka-configure na account id (ayon sa pagkakasunod-sunod)
    - ang mga account id ay ini-normalize sa loob para sa lookup
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - kasalukuyang auth path: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - backup file: `creds.json.bak`
    - ang legacy default auth sa `~/.openclaw/credentials/` ay kinikilala/ini-migrate pa rin para sa default-account flows
  
</Accordion>

  <Accordion title="Logout behavior">
    `openclaw channels logout --channel whatsapp [--account <id>]` ay naglilinis ng WhatsApp auth state para sa account na iyon.

    ```
    Sa mga legacy auth directory, pinananatili ang `oauth.json` habang inaalis ang mga Baileys auth file.
    ```

  
</Accordion>
</AccordionGroup>

## Mga limitasyon

- Ang outbound text ay hinahati sa `channels.whatsapp.textChunkLimit` (default 4000).
- Opsyonal na newline chunking: itakda ang `channels.whatsapp.chunkMode="newline"` para hatiin sa mga blank line (hangganan ng talata) bago ang length chunking.
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Ang inbound media saves ay may cap na `channels.whatsapp.mediaMaxMb` (default 50 MB).

## Outbound send (text + media)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    Sintomas: ang status ng channel ay nagpapakitang hindi naka-link.
  

    ````
    Ayusin:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Sintomas: naka-link ang account ngunit may paulit-ulit na disconnect o reconnect attempts.
  

    ````
    Ayusin:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    Kung kinakailangan, mag-relink gamit ang `channels login`.
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    Ang outbound sends ay agad na mabibigo kung walang aktibong gateway listener para sa target na account.
  

    ```
    Siguraduhing tumatakbo ang gateway at naka-link ang account.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Suriin sa ganitong pagkakasunod-sunod:
  

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - mga entry ng `groups` allowlist
    - mention gating (`requireMention` + mga mention pattern)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Ang WhatsApp gateway runtime ay dapat gumamit ng Node. Ang Bun ay itinuturing na hindi compatible para sa stable na operasyon ng WhatsApp/Telegram gateway.
  
</Accordion>
</AccordionGroup>

## Mga pointer sa configuration reference

Pangunahing reference:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

Mahahalagang WhatsApp field:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>.enabled`, `accounts.<id>.authDir`, mga override sa antas ng account
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- session behavior: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>`.historyLimit\`

## Kaugnay

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
