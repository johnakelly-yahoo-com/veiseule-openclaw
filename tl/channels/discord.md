---
summary: "Katayuan ng suporta ng Discord bot, mga kakayahan, at konpigurasyon"
read_when:
  - Gumagawa sa mga feature ng Discord channel
title: "Discord"
---

# Discord (Bot API)

Status: handa para sa DM at mga guild text channel sa pamamagitan ng opisyal na Discord bot gateway.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Cross-channel diagnostics at daloy ng pag-aayos.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">Mabilis na setup
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Gumawa ng application sa Discord Developer Portal, magdagdag ng bot, pagkatapos ay i-enable:
- **Message Content Intent**
- **Server Members Intent** (kinakailangan para sa mga role allowlist at role-based routing; inirerekomenda para sa name-to-ID allowlist matching)
</Card>
</CardGroup>

## Env fallback para sa default account:

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Create an application in the Discord Developer Portal, add a bot, then enable:

    ```
    - **Message Content Intent**
    - **Server Members Intent** (required for role allowlists and role-based routing; recommended for name-to-ID allowlist matching)
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Env fallback for the default account:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Imbitahan ang bot sa iyong server na may pahintulot sa mga mensahe.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Ang mga pairing code ay mag-e-expire pagkatapos ng 1 oras.
    ```

  
</Step>
</Steps>

<Note>
Ang token resolution ay nakabatay sa account. Ang mga halaga ng config token ang masusunod kaysa sa env fallback. Ang `DISCORD_BOT_TOKEN` ay ginagamit lamang para sa default na account.
</Note>

## Runtime model

- Ang Gateway ang may kontrol sa koneksyon ng Discord.
- Deterministiko ang reply routing: ang mga inbound reply mula sa Discord ay babalik din sa Discord.
- Bilang default (`session.dmScope=main`), ang mga direct chat ay nagbabahagi ng main session ng agent (`agent:main:main`).
- Ang mga Guild channel ay may hiwa-hiwalay na session key (`agent:<agentId>:discord:channel:<channelId>`).
- Ang mga Group DM ay hindi pinapansin bilang default (`channels.discord.dm.groupEnabled=false`).
- Ang mga native slash command ay tumatakbo sa hiwalay na command session (`agent:<agentId>:discord:slash:<userId>`), habang dala pa rin ang `CommandTargetSessionKey` papunta sa na-route na session ng pag-uusap.

## Access control at routing

<Tabs>
  <Tab title="DM policy">
    Kinokontrol ng `channels.discord.dmPolicy` ang DM access (legacy: `channels.discord.dm.policy`):

    ```
    - `pairing` (default)
    - `allowlist`
    - `open` (nangangailangan na isama ng `channels.discord.allowFrom` ang `"*"`; legacy: `channels.discord.dm.allowFrom`)
    - `disabled`
    
    Kung ang DM policy ay hindi open, ang mga hindi kilalang user ay iba-block (o ipo-prompt para sa pairing sa `pairing` mode).
    
    DM target format para sa delivery:
    
    - `user:<id>`
    - `<@id>` mention
    
    Ang mga numeric ID lamang ay itinuturing na ambiguous at tinatanggihan maliban kung may malinaw na user/channel target kind na ibinigay.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Ang paghawak sa Guild ay kinokontrol ng `channels.discord.groupPolicy`:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Ang secure na baseline kapag may `channels.discord` ay `allowlist`.
    
    Pag-uugali ng `allowlist`:
    
    - ang guild ay dapat tumugma sa `channels.discord.guilds` (`id` ang mas mainam, tinatanggap ang slug)
    - opsyonal na sender allowlist: `users` (mga ID o pangalan) at `roles` (mga role ID lamang); kung alinman ang naka-configure, pinapayagan ang sender kapag tumugma sa `users` O `roles`
    - kung ang isang guild ay may naka-configure na `channels`, ang mga channel na wala sa listahan ay tatanggihan
    - kung ang isang guild ay walang `channels` block, lahat ng channel sa allowlisted guild na iyon ay pinapayagan
    
    Halimbawa:
    ```

```json5
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

    ```
    Kung `DISCORD_BOT_TOKEN` lamang ang itinakda mo at hindi ka gumawa ng `channels.discord` block, ang runtime fallback ay `groupPolicy="open"` (may kasamang babala sa logs).
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Ang mga mensahe sa Guild ay nangangailangan ng mention bilang default.

    ```
    Kasama sa mention detection ang:
    
    - tahasang pag-mention sa bot
    - mga naka-configure na mention pattern (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit na pag-reply sa bot sa mga suportadong kaso
    
    Ang `requireMention` ay kino-configure bawat guild/channel (`channels.discord.guilds...`).
    
    Group DMs:
    
    - default: hindi pinapansin (`dm.groupEnabled=false`)
    - opsyonal na allowlist sa pamamagitan ng `dm.groupChannels` (mga channel ID o slug)
    ```

  
</Tab>
</Tabs>

### Role-based na pag-route ng agent

Gamitin ang `bindings[].match.roles` upang i-route ang mga miyembro ng Discord guild sa iba’t ibang agent batay sa role ID. Ang mga role-based binding ay tumatanggap lamang ng mga role ID at sinusuri pagkatapos ng peer o parent-peer binding at bago ang guild-only binding. Kung ang isang binding ay nagtakda rin ng iba pang match field (halimbawa `peer` + `guildId` + `roles`), lahat ng naka-configure na field ay dapat tumugma.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Setup ng Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    1. Discord Developer Portal -> **Applications** -> **New Application**
    2. **Bot** -> **Add Bot**
    3. Kopyahin ang bot token
    ```

  
</Accordion>

  <Accordion title="Privileged intents">
    Sa **Bot -> Privileged Gateway Intents**, i-enable ang:

    ```
    - Message Content Intent
    - Server Members Intent (inirerekomenda)
    
    Ang Presence intent ay opsyonal at kinakailangan lamang kung nais mong makatanggap ng mga update sa presence. Ang pagtatakda ng bot presence (`setPresence`) ay hindi nangangailangan ng pag-enable ng presence updates para sa mga miyembro.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">
    OAuth URL generator:

    ```
    - scopes: `bot`, `applications.commands`
    
    Karaniwang baseline na mga pahintulot:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (opsyonal)
    
    Iwasan ang `Administrator` maliban kung tahasang kinakailangan.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    I-enable ang Discord Developer Mode, pagkatapos ay kopyahin ang:

    ```
    - server ID
    - channel ID
    - user ID
    
    Mas mainam ang paggamit ng mga numeric ID sa OpenClaw config para sa mas maaasahang audit at probes.
    ```

  
</Accordion>
</AccordionGroup>

## Mga native na command at command auth

- Ang `commands.native` ay naka-default sa `"auto"` at naka-enable para sa Discord.
- Per-channel override: `channels.discord.commands.native`.
- Ang `commands.native=false` ay tahasang nag-aalis ng mga dati nang nairehistrong Discord native command.
- Ang native command auth ay gumagamit ng parehong Discord allowlists/policies tulad ng karaniwang paghawak ng mensahe.
- Maaaring makita pa rin ang mga command sa Discord UI ng mga user na hindi awtorisado; gayunpaman, ang pagpapatupad ay patuloy na nagpapatupad ng OpenClaw auth at magbabalik ng "not authorized".

Tingnan ang [Slash commands](/tools/slash-commands) para sa katalogo at pag-uugali ng command.

## Retry policy

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Sinusuportahan ng Discord ang reply tags sa output ng agent:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Kinokontrol ng `channels.discord.replyToMode`:
    
    - `off` (default)
    - `first`
    - `all`
    
    Tandaan: Ang `off` ay nagdi-disable ng implicit reply threading. Ang mga tahasang `[[reply_to_*]]` tag ay patuloy na sinusunod.
    
    Ang mga Message ID ay inilalabas sa context/history upang ma-target ng mga agent ang partikular na mensahe.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Guild history context:

    ```
    - `channels.discord.historyLimit` default `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` disables
    
    Mga kontrol sa DM history:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread behavior:
    
    - Ang mga Discord thread ay niruruta bilang mga channel session
    - maaaring gamitin ang parent thread metadata para sa parent-session linkage
    - ang thread config ay nagmamana ng parent channel config maliban kung may thread-specific entry
    
    Ang mga topic ng channel ay ini-inject bilang **untrusted** na context (hindi bilang system prompt).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Per-guild reaction notification mode:

    ```
    - `off`
    - `own` (default)
    - `all`
    - `allowlist` (gumagamit ng `guilds.<id>.users`)
    
    Ang mga reaction event ay ginagawang system event at ikinakabit sa nirutang Discord session.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    Ang `ackReaction` ay nagpapadala ng acknowledgement emoji habang pinoproseso ng OpenClaw ang isang papasok na mensahe.

    ```
    Resolution order:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - fallback na emoji ng agent identity (`agents.list[].identity.emoji`, kung wala ay "👀")
    
    Mga Tala:
    
    - Tumatanggap ang Discord ng unicode emoji o custom emoji names.
    - Gamitin ang `""` upang i-disable ang reaction para sa isang channel o account.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Ang mga config write na sinimulan ng channel ay naka-enable bilang default.

    ```
    Nakakaapekto ito sa mga daloy ng `/config set|unset` (kapag naka-enable ang mga feature ng command).
    
    I-disable:
    ```

```json5
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Iruta ang Discord gateway WebSocket traffic sa pamamagitan ng HTTP(S) proxy gamit ang `channels.discord.proxy`.

```json5
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

    ```
    Per-account override:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">
    I-enable ang PluralKit resolution upang imapa ang mga proxied na mensahe sa identity ng system member:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // opsyonal; kailangan para sa mga private system
      },
    },
  },
}
```

    ```
    Mga Tala:
    
    - maaaring gumamit ang allowlists ng `pk:<memberId>`
    - ang mga display name ng member ay itinutugma ayon sa name/slug
    - ang mga lookup ay gumagamit ng orihinal na message ID at may limitasyon sa time-window
    - kung mabigo ang lookup, ang mga proxied na mensahe ay ituturing na bot messages at ida-drop maliban kung `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Ang mga presence update ay inilalapat lamang kapag nagtakda ka ng status o activity field.

    ```
    Halimbawa ng status lamang:
    ```

```json5
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

    ```
    Halimbawa ng activity (ang custom status ang default na activity type):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Halimbawa ng streaming:
    ```

```json5
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

    ```
    Activity type map:
    
    - 0: Playing
    - 1: Streaming (nangangailangan ng `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (ginagamit ang activity text bilang status state; opsyonal ang emoji)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Sinusuportahan ng Discord ang button-based exec approvals sa mga DM at maaaring opsyonal na mag-post ng mga approval prompt sa pinanggalingang channel.

    ```
    Config path:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, default: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Kapag ang `target` ay `channel` o `both`, makikita ang approval prompt sa channel. Tanging ang mga naka-configure na approver lamang ang maaaring gumamit ng mga button; ang ibang user ay makakatanggap ng ephemeral na pagtanggi. Kasama sa mga approval prompt ang teksto ng command, kaya i-enable lamang ang channel delivery sa mga pinagkakatiwalaang channel. Kung hindi makuha ang channel ID mula sa session key, babalik ang OpenClaw sa DM delivery.
    
    Kung mabigo ang approvals na may unknown approval IDs, i-verify ang listahan ng approver at ang pag-enable ng feature.
    
    Kaugnay na docs: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Mga tool at action gates

Kasama sa mga Discord message action ang messaging, channel admin, moderation, presence, at metadata actions.

Mga pangunahing halimbawa:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- presence: `setPresence`

Ang mga action gate ay nasa ilalim ng `channels.discord.actions.*`.

Default na behavior ng gate:

| Action group                                                                                                                                                             | Default  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

Ginagamit ng OpenClaw ang Discord components v2 para sa exec approvals at cross-context markers. Ang Discord message actions ay maaari ring tumanggap ng `components` para sa custom UI (advanced; nangangailangan ng Carbon component instances), habang nananatiling available ang legacy `embeds` ngunit hindi ito inirerekomenda.

- Itinatakda ng `channels.discord.ui.components.accentColor` ang accent color na ginagamit ng mga Discord component container (hex).
- Itakda kada account gamit ang `channels.discord.accounts.<id> .ui.components.accentColor`.
- Hindi pinapansin ang `embeds` kapag may components v2.

Halimbawa:

```json5
{
  channels: {
    discord: {
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
    },
  },
}
```

## Mga voice message

Ang mga Discord voice message ay nagpapakita ng waveform preview at nangangailangan ng OGG/Opus audio kasama ang metadata. Awtomatikong binubuo ng OpenClaw ang waveform, ngunit kailangan nito ang `ffmpeg` at `ffprobe` na available sa gateway host upang suriin at i-convert ang mga audio file.

Mga kinakailangan at limitasyon:

- Magbigay ng **lokal na file path** (hindi tinatanggap ang mga URL).
- Huwag isama ang text content (hindi pinapayagan ng Discord ang text + voice message sa iisang payload).
- Anumang audio format ay tinatanggap; kino-convert ng OpenClaw sa OGG/Opus kapag kinakailangan.

Halimbawa:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - i-enable ang Message Content Intent
    - i-enable ang Server Members Intent kapag umaasa ka sa user/member resolution
    - i-restart ang gateway pagkatapos baguhin ang intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - i-verify ang `groupPolicy`
    - i-verify ang guild allowlist sa ilalim ng `channels.discord.guilds`
    - kung may umiiral na guild `channels` map, tanging ang mga nakalistang channel lamang ang pinapayagan
    - i-verify ang `requireMention` behavior at mga mention pattern
    
    Mga kapaki-pakinabang na pagsusuri:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Mga karaniwang sanhi:

    ```
    - `groupPolicy="allowlist"` na walang katugmang guild/channel allowlist
    - naka-configure ang `requireMention` sa maling lugar (dapat nasa ilalim ng `channels.discord.guilds` o channel entry)
    - naka-block ang sender ng guild/channel `users` allowlist
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    Ang `channels status --probe` permission checks ay gumagana lamang para sa numeric channel IDs.

    ```
    Kung gumagamit ka ng slug keys, maaari pa ring gumana ang runtime matching, ngunit hindi ganap na mave-verify ng probe ang mga permission.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - naka-disable ang DM: `channels.discord.dm.enabled=false`
    - naka-disable ang DM policy: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - naghihintay ng pairing approval sa `pairing` mode
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Bilang default, hindi pinapansin ang mga mensaheng ginawa ng bot.

    ```
    Kung itatakda mo ang `channels.discord.allowBots=true`, gumamit ng mahigpit na mention at allowlist rules upang maiwasan ang loop behavior.
    ```

  
</Accordion>
</AccordionGroup>

## Mga sangguniang pointer sa configuration

Pangunahing sanggunian:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Mahahalagang Discord fields:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- patakaran: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- utos: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- mga aksyon: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- mga feature: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Kaligtasan at operasyon

- Tratuhin ang mga bot token bilang mga lihim (`DISCORD_BOT_TOKEN` ang mas pinipili sa mga supervised na environment).
- Magbigay ng pinakamababang kinakailangang Discord permissions.
- Kung luma na ang command deploy/state, i-restart ang gateway at suriin muli gamit ang `openclaw channels status --probe`.

## Kaugnay

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Slash commands](/tools/slash-commands)

