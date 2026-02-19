---
summary: "Setup ng Slack para sa socket o HTTP webhook mode"
read_when:
  - Pagse-set up ng Slack o pag-debug ng Slack socket/HTTP mode
title: "Slack"
---

# Slack

Status: handa na para sa production para sa DMs + channels sa pamamagitan ng Slack app integrations. Ang default na mode ay Socket Mode; sinusuportahan din ang HTTP Events API mode.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Ang Slack DMs ay naka-default sa pairing mode.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native na pag-uugali ng command at katalogo ng mga command.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Mga diagnostic at repair playbook na cross-channel.
  
</Card>
</CardGroup>

## Mabilis na setup

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">
        Sa mga setting ng Slack app:

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "socket",
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Env fallback (default account lamang):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...

```

        
</Step>
      
        <Step title="Mag-subscribe sa mga app event">
          Mag-subscribe sa mga bot event para sa:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          I-enable rin ang App Home **Messages Tab** para sa DMs.
        
</Step>
      
        <Step title="Simulan ang gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        {
          channels: {
            slack: {
              enabled: true,
              appToken: "xapp-...",
              botToken: "xoxb-...",
            },
          },
        }
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

        
</Step>
      
        <Step title="Gumamit ng natatanging webhook path para sa multi-account HTTP">
          Sinusuportahan ang per-account HTTP mode.
      
          Bigyan ang bawat account ng natatanging `webhookPath` upang hindi magbanggaan ang mga rehistrasyon.
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Modelo ng token

- Kinakailangan ang `botToken` + `appToken` para sa Socket Mode.
- Ang HTTP mode ay nangangailangan ng `botToken` + `signingSecret`.
- Ang mga config token ay may mas mataas na priyoridad kaysa sa env fallback.
- Ang `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback ay nalalapat lamang sa default na account.
- Ang `userToken` (`xoxp-...`) ay config-only (walang env fallback) at naka-default sa read-only na pag-uugali (`userTokenReadOnly: true`).
- Opsyonal: idagdag ang `chat:write.customize` kung nais mong gamitin ng mga papalabas na mensahe ang aktibong agent identity (custom na `username` at icon). Ang `icon_emoji` ay gumagamit ng `:emoji_name:` na syntax.

<Tip>
Para sa mga aksyon/pagbasa ng directory, maaaring mas piliin ang user token kapag naka-configure. Para sa mga write, nananatiling mas pinipili ang bot token; pinapayagan lamang ang user-token writes kapag `userTokenReadOnly: false` at hindi available ang bot token.
</Tip>

## Access control at routing

<Tabs>
  <Tab title="DM policy">
    Kinokontrol ng `channels.slack.dmPolicy` ang DM access (legacy: `channels.slack.dm.policy`):
- `pairing` (default)
- `allowlist`
- `open` (nangangailangan na ang `channels.slack.allowFrom` ay may kasamang `"*"`; legacy: `channels.slack.dm.allowFrom`)
- `disabled`

Mga DM flag:

- `dm.enabled` (default true)
- `channels.slack.allowFrom` (mas pinipili)
- `dm.allowFrom` (legacy)
- `dm.groupEnabled` (ang group DMs ay default na false)
- `dm.groupChannels` (opsyonal na MPIM allowlist)

Ang pairing sa DMs ay gumagamit ng `openclaw pairing approve slack <code>`.

    ```
    
        Kinokontrol ng `channels.slack.groupPolicy` ang paghawak ng channel:
    - `open`
    - `allowlist`
    - `disabled`
    
    Ang allowlist ng channel ay nasa ilalim ng `channels.slack.channels`.
    
    Tala sa runtime: kung ang `channels.slack` ay ganap na nawawala (env-only setup) at ang `channels.defaults.groupPolicy` ay hindi nakatakda, ang runtime ay babalik sa `groupPolicy="open"` at magla-log ng babala.
    
    Name/ID resolution:
    
    - ang mga entry sa channel allowlist at DM allowlist ay nire-resolve sa startup kapag pinapayagan ng token access
    - ang mga hindi ma-resolve na entry ay pinananatili ayon sa pagka-configure
    ```

  
</Tab>

  <Tab title="Channel policy">    Ang mga mensahe sa channel ay naka-mention-gated bilang default.

    ```
    Mga pinagmumulan ng mention:
    
    - tahasang pagbanggit sa app (`<@botId>`)
    - mga pattern ng mention regex (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - implicit na pag-reply sa thread ng bot
    
    Mga kontrol bawat channel (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>

  <Tab title="Mentions and channel users">  
</Tab>

    ```
    Ang native command auto-mode ay **naka-off** para sa Slack (`commands.native: "auto"` ay hindi nagpapagana ng Slack native commands).
    ```

  
</Tab>
</Tabs>

## OpenClaw config (minimal)

- Native command auto-mode is **off** for Slack (`commands.native: "auto"` does not enable Slack native commands).
- Enable native Slack command handlers with `channels.slack.commands.native: true` (or global `commands.native: true`).
- Kapag naka-enable ang mga native command, magrehistro ng mga katugmang slash command sa Slack (`/<command>` names).
- Kung hindi naka-enable ang mga native command, maaari kang magpatakbo ng isang naka-configure na slash command sa pamamagitan ng `channels.slack.slashCommand`.

Default na mga setting ng slash command:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Gumagamit ang mga slash session ng hiwa-hiwalay na mga key:

- `agent:<agentId>:slack:slash:<userId>`

at niruruta pa rin ang pagpapatupad ng command laban sa target na conversation session (`CommandTargetSessionKey`).

## Scopes (kasalukuyan vs opsyonal)

- Ang mga DM ay niruruta bilang `direct`; ang mga channel bilang `channel`; ang mga MPIM bilang `group`.
- Kapag default ang `session.dmScope=main`, ang mga Slack DM ay pinagsasama sa main session ng agent.
- Mga channel session: `agent:<agentId>:slack:channel:<channelId>`.
- Ang mga reply sa thread ay maaaring lumikha ng mga thread session suffix (`:thread:<threadTs>`) kung naaangkop.
- Ang default ng `channels.slack.thread.historyScope` ay `thread`; ang default ng `thread.inheritParent` ay `false`.
- Kinokontrol ng `channels.slack.thread.initialHistoryLimit` kung ilang umiiral na mensahe sa thread ang kukunin kapag nagsimula ang bagong thread session (default `20`; itakda sa `0` para i-disable).

Mga kontrol sa reply threading:

- `chat:write` (magpadala/mag-update/mag-delete ng mga mensahe via `chat.postMessage`)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (magbukas ng DMs via `conversations.open` para sa user DMs)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Sinusuportahan ang mga manual reply tag:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Tandaan: ang `replyToMode="off"` ay nagdi-disable ng implicit reply threading. Ang mga tahasang `[[reply_to_*]]` na tag ay iginagalang pa rin.

## Hindi kailangan sa ngayon (ngunit posibleng sa hinaharap)

<AccordionGroup>
  <Accordion title="Inbound attachments">    Ang mga Slack file attachment ay dina-download mula sa mga pribadong URL na naka-host sa Slack (token-authenticated request flow) at isinusulat sa media store kapag matagumpay ang pagkuha at pinapayagan ng mga limitasyon sa laki.

    ```
    Ang default na inbound size cap sa runtime ay `20MB` maliban kung ma-override ng `channels.slack.mediaMaxMb`.
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">    - ang mga text chunk ay gumagamit ng `channels.slack.textChunkLimit` (default 4000)
    - ang `channels.slack.chunkMode="newline"` ay nag-e-enable ng paragraph-first splitting
    - ang pagpapadala ng file ay gumagamit ng Slack upload APIs at maaaring magsama ng mga thread reply (`thread_ts`)
    - ang outbound media cap ay sumusunod sa `channels.slack.mediaMaxMb` kapag naka-configure; kung hindi, ang pagpapadala sa channel ay gumagamit ng mga default na MIME-kind mula sa media pipeline
  
</Accordion>

  <Accordion title="Delivery targets">    Mga mas pinipiling tahasang target:

    ```
    - `user:<id>` para sa mga DM
    - `channel:<id>` para sa mga channel
    
    Ang mga Slack DM ay binubuksan sa pamamagitan ng Slack conversation APIs kapag nagpapadala sa mga user target.
    ```

  
</Accordion>
</AccordionGroup>

## Limits

Ang mga Slack action ay kinokontrol ng `channels.slack.actions.*`.

Mga available na action group sa kasalukuyang Slack tooling:

| Grupo      | Default |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## Mga event at operasyonal na pag-uugali

- Ang mga pag-edit/pagbura ng mensahe at thread broadcast ay mina-map bilang mga system event.
- Ang mga reaction add/remove event ay mina-map bilang mga system event.
- Ang mga event ng pagpasok/pag-alis ng miyembro, paggawa/pagpapalit ng pangalan ng channel, at pagdaragdag/pag-alis ng pin ay mina-map bilang mga system event.
- Maaaring ilipat ng `channel_id_changed` ang mga key ng config ng channel kapag naka-enable ang `configWrites`.
- Ang metadata ng channel topic/purpose ay itinuturing na hindi pinagkakatiwalaang context at maaaring i-inject sa routing context.

## Per-chat-type threading

Maaari kang mag-configure ng magkakaibang threading behavior kada uri ng chat sa pamamagitan ng pagtatakda ng `channels.slack.replyToModeByChatType`:

Pagkakasunod-sunod ng resolusyon:

- `channels.slack.accounts.<accountId> .ackReaction`
- `channels.slack.ackReaction`
- `messages.ackReaction`
- fallback na emoji ng pagkakakilanlan ng agent (`agents.list[].identity.emoji`, kung hindi ay "👀")

Mga Tala:

- Inaasa ng Slack ang mga shortcode (halimbawa, `"eyes"`).
- Gamitin ang `""` upang i-disable ang reaction para sa isang channel o account.

## Checklist ng Manifest at scope

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "im:history",
        "mpim:history",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Kung iko-configure mo ang `channels.slack.userToken`, ang karaniwang read scopes ay:

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (kung umaasa ka sa Slack search reads)
    ```

  
</Accordion>
</AccordionGroup>

## Pag-troubleshoot

<AccordionGroup>
  <Accordion title="No replies in channels">
    Suriin, ayon sa pagkakasunod-sunod:

    ```
    - `groupPolicy`
    - allowlist ng channel (`channels.slack.channels`)
    - `requireMention`
    - per-channel na `users` allowlist
    
    Mga kapaki-pakinabang na command:
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Suriin:

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (o legacy `channels.slack.dm.policy`)
    - mga pag-apruba sa pairing / mga entry sa allowlist
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    I-validate ang bot + app tokens at ang pag-enable ng Socket Mode sa Slack app settings.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    I-validate:

    ```
    - signing secret
    - webhook path
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - natatanging `webhookPath` para sa bawat HTTP account
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Tiyakin kung alin ang iyong nilalayon:

    ```
    - native command mode (`channels.slack.commands.native: true`) na may tumutugmang slash commands na naka-register sa Slack
    - o single slash command mode (`channels.slack.slashCommand.enabled: true`)
    
    Suriin din ang `commands.useAccessGroups` at mga allowlist ng channel/user.
    ```

  
</Accordion>
</AccordionGroup>

## Tool actions

Maaaring i-gate ang mga Slack tool actions gamit ang `channels.slack.actions.*`:

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  Mahahalagang Slack fields:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM access: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - channel access: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - threading/history: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ops/features: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Mga tala sa seguridad

- Ang writes ay nagde-default sa bot token para manatiling naka-scope ang mga aksyong nagbabago ng state sa
  mga permiso at identidad ng bot ng app.
- [Channel routing](/channels/channel-routing)
- Kung i-enable mo ang user-token writes, siguraduhing kasama sa user token ang mga write
  scopes na inaasahan mo (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) kung hindi ay mabibigo ang mga operasyong iyon.
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)
