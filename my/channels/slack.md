---
summary: "Socket သို့မဟုတ် HTTP webhook မုဒ်အတွက် Slack တပ်ဆင်မှု"
read_when:
  - Slack ကို စတင်တပ်ဆင်ခြင်း သို့မဟုတ် Slack socket/HTTP mode ကို ပြဿနာရှာဖွေခြင်း
title: "Slack"
---

# Slack

အခြေအနေ - Slack app integration များမှတစ်ဆင့် DMs နှင့် channels အတွက် production-ready ဖြစ်သည်။ ပုံမှန် mode သည် Socket Mode ဖြစ်ပြီး HTTP Events API mode ကိုလည်း ထောက်ပံ့ပေးထားသည်။

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    Slack DMs များသည် ပုံမှန်အားဖြင့် pairing mode ဖြင့် လုပ်ဆောင်သည်။
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">    Native command အပြုအမူနှင့် command စာရင်း။
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    Channel များအကြား အခြေအနေစစ်ဆေးမှုနှင့် ပြုပြင်ရေး playbook များ။
</Card>
</CardGroup>

## အမြန် စတင်တပ်ဆင်ခြင်း

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Slack app settings တွင်:

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
            Env fallback (default account အတွက်သာ):
        ```

```bash
SLACK_APP_TOKEN=xapp-...
SLACK_BOT_TOKEN=xoxb-...
```

        
</Step>
      
        <Step title="Subscribe app events">
          အောက်ပါ bot events များကို Subscribe လုပ်ပါ:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          DMs အတွက် App Home ၏ **Messages Tab** ကိုလည်း ဖွင့်ပါ။
        
</Step>
      
        <Step title="Start gateway">

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
      
        <Step title="Use unique webhook paths for multi-account HTTP">
          Account တစ်ခုချင်းစီအလိုက် HTTP mode ကို ထောက်ပံ့ပေးထားသည်။
      
          Registration များ မထိခိုက်စေရန် account တစ်ခုချင်းစီအတွက် `webhookPath` မတူညီသည့် တန်ဖိုးကို သတ်မှတ်ပါ။
        
</Step>
      
</Steps>

  
</Tab>
</Tabs>

## Token မော်ဒယ်

- Socket Mode အတွက် `botToken` + `appToken` လိုအပ်သည်။
- HTTP mode အတွက် `botToken` + `signingSecret` လိုအပ်သည်။
- Config ထဲရှိ token များသည် env fallback ထက် အရေးယူထားသည်။
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` env fallback သည် default account အတွက်သာ သက်ရောက်သည်။
- `userToken` (`xoxp-...`) သည် config တွင်သာ သတ်မှတ်နိုင်ပြီး (env fallback မရှိ) ပုံမှန်အားဖြင့် read-only အပြုအမူ (`userTokenReadOnly: true`) ဖြင့် လုပ်ဆောင်သည်။
- ရွေးချယ်နိုင်သည် - outgoing messages များကို active agent identity (custom `username` နှင့် icon) ဖြင့် ပို့လိုပါက `chat:write.customize` ကို ထည့်ပါ။ `icon_emoji` သည် `:emoji_name:` ပုံစံ syntax ကို အသုံးပြုသည်။

<Tip>
Action များနှင့် directory ဖတ်ရှုမှုများအတွက် config လုပ်ထားပါက user token ကို ဦးစားပေးအသုံးပြုနိုင်သည်။ ရေးသားမှုများအတွက် bot token ကို ဆက်လက် ဦးစားပေးအသုံးပြုသည်။ `userTokenReadOnly: false` ဖြစ်ပြီး bot token မရရှိပါကသာ user-token ဖြင့် ရေးသားခွင့်ပြုသည်။
</Tip>

## Access control နှင့် routing

<Tabs>
  <Tab title="DM policy">    `channels.slack.dmPolicy` သည် DM access ကို ထိန်းချုပ်သည် (legacy: `channels.slack.dm.policy`):

    ```
    - `pairing` (ပုံမှန်)
    - `allowlist`
    - `open` (`channels.slack.allowFrom` ထဲတွင် `"*"` ပါဝင်ရန် လိုအပ်သည်; legacy: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM flags:
    
    - `dm.enabled` (ပုံမှန် true)
    - `channels.slack.allowFrom` (ဦးစားပေး)
    - `dm.allowFrom` (legacy)
    - `dm.groupEnabled` (group DMs ပုံမှန် false)
    - `dm.groupChannels` (ရွေးချယ်နိုင်သော MPIM allowlist)
    
    DM များတွင် Pairing ကို `openclaw pairing approve slack <code>` ဖြင့် အသုံးပြုသည်။
    ```

  
</Tab>

  <Tab title="Channel policy">    `channels.slack.groupPolicy` သည် channel ကိုင်တွယ်ပုံကို ထိန်းချုပ်သည်:

    ```
    - `open`
    - `allowlist`
    - `disabled`
    
    Channel allowlist ကို `channels.slack.channels` အောက်တွင် သတ်မှတ်သည်။
    
    Runtime မှတ်ချက် - `channels.slack` ကို လုံးဝမသတ်မှတ်ထားပါက (env-only setup) နှင့် `channels.defaults.groupPolicy` ကိုလည်း မသတ်မှတ်ထားပါက runtime သည် `groupPolicy="open"` သို့ fallback လုပ်ပြီး သတိပေးချက်ကို log ထုတ်မည်ဖြစ်သည်။
    
    အမည်/ID ဖြေရှင်းခြင်း:
    
    - token access ခွင့်ပြုပါက channel allowlist နှင့် DM allowlist entries များကို startup အချိန်တွင် resolve လုပ်သည်
    - မဖြေရှင်းနိုင်သည့် entries များကို သတ်မှတ်ထားသည့်အတိုင်း ထိန်းသိမ်းထားသည်
    ```

  
</Tab>

  <Tab title="Mentions and channel users">    Channel messages များသည် ပုံမှန်အားဖြင့် mention-gated ဖြစ်သည်။

    ```
    Mention အရင်းအမြစ်များ:
    
    - တိတိကျကျ app mention (`<@botId>`)
    - mention regex patterns (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - bot ကို reply ပြန်ထားသည့် thread အပြုအမူ (implicit)
    
    Channel အလိုက် ထိန်းချုပ်မှုများ (`channels.slack.channels.<id|name>`):
    
    - `requireMention`
    - `users` (allowlist)
    - `allowBots`
    - `skills`
    - `systemPrompt`
    - `tools`, `toolsBySender`
    ```

  
</Tab>
</Tabs>

## OpenClaw config (အနည်းဆုံး)

- Native command auto-mode ကို Slack အတွက် **ပိတ်ထားသည်** (`commands.native: "auto"` သည် Slack native commands ကို မဖွင့်ပေးပါ)။
- `channels.slack.commands.native: true` (သို့မဟုတ် global `commands.native: true`) ဖြင့် native Slack command handlers ကို ဖွင့်ပါ။
- Native commands ကို ဖွင့်ထားပါက Slack တွင် ကိုက်ညီသည့် slash commands (`/<command>` အမည်များ) ကို register လုပ်ပါ။
- Native commands မဖွင့်ထားပါက `channels.slack.slashCommand` မှတစ်ဆင့် သတ်မှတ်ထားသည့် slash command တစ်ခုတည်းကို လည်ပတ်နိုင်သည်။

ပုံသေ slash command ဆက်တင်များ:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

`webhookPath` ကို ပေးပါ၊ Slack app တစ်ခုချင်းစီက ကိုယ်ပိုင် URL ကို ညွှန်းနိုင်စေရန်။ ဒီ Slack app manifest ကို အသုံးပြုပြီး app ကို မြန်မြန် ဖန်တီးပါ (အမည်/command ကို လိုအပ်သလို ပြင်ဆင်နိုင်သည်)။

- `agent:<agentId>:slack:slash:<userId>`

user token ကို ပြင်ဆင်သတ်မှတ်ရန် စီစဉ်ထားပါက user scopes များကို ထည့်သွင်းပါ။ native commands ကို ဖွင့်ထားပါက ဖော်ပြလိုသော command တစ်ခုချင်းစီအတွက် `slash_commands` entry တစ်ခုစီ ထည့်ပါ (`/help` စာရင်းနှင့် ကိုက်ညီရပါမည်)။

## Scopes (လက်ရှိ vs ရွေးချယ်စရာ)

- DM များကို `direct` အဖြစ် လမ်းကြောင်းချထားပြီး; channel များကို `channel` အဖြစ်; MPIM များကို `group` အဖြစ် သတ်မှတ်သည်။
- ပုံသေ `session.dmScope=main` ဖြင့် Slack DM များသည် agent ၏ main session သို့ ပေါင်းစည်းသွားသည်။
- Channel session များ: `agent:<agentId>:slack:channel:<channelId>`.
- လိုအပ်သည့်အခါ thread reply များသည် thread session suffix များ (`:thread:<threadTs>`) ကို ဖန်တီးနိုင်သည်။
- `channels.slack.thread.historyScope` ၏ ပုံသေတန်ဖိုးမှာ `thread` ဖြစ်ပြီး; `thread.inheritParent` ၏ ပုံသေတန်ဖိုးမှာ `false` ဖြစ်သည်။
- `channels.slack.thread.initialHistoryLimit` သည် thread session အသစ် စတင်သောအခါ ရှိပြီးသား thread message များကို ဘယ်နှစ်ခုယူမည်ကို ထိန်းချုပ်သည် (ပုံသေ `20`; ပိတ်လိုပါက `0` သတ်မှတ်ပါ)။

Reply threading ထိန်းချုပ်မှုများ:

- `chat:write` (`chat.postMessage` ဖြင့် မက်ဆေ့ချ် ပို့/ပြင်/ဖျက်)
  [https://docs.slack.dev/reference/methods/chat.postMessage](https://docs.slack.dev/reference/methods/chat.postMessage)
- `im:write` (user DM များအတွက် `conversations.open` ဖြင့် DM ဖွင့်ခြင်း)
  [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
- `channels:history`, `groups:history`, `im:history`, `mpim:history`
  [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)

Manual reply tag များကို ပံ့ပိုးထားသည်:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

မှတ်ချက် - `replyToMode="off"` သတ်မှတ်ပါက အလိုအလျောက် reply threading ကို ပိတ်မည်ဖြစ်သည်။ သို့သော် Explicit `[[reply_to_*]]` tag များကို ဆက်လက် လက်ခံအသုံးပြုမည်ဖြစ်သည်။

## ယနေ့မလိုအပ်သေးသော်လည်း (နောင်တွင် ဖြစ်နိုင်)

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack ဖိုင်တွဲချိတ်ဆက်မှုများကို Slack တွင် host လုပ်ထားသော private URL များမှ (token-authenticated request flow ဖြင့်) ဒေါင်းလုဒ်လုပ်ပြီး fetch အောင်မြင်ပြီး အရွယ်အစားကန့်သတ်ချက်အတွင်းရှိပါက media store ထဲသို့ ရေးသွင်းသိမ်းဆည်းသည်။

    ```
    Runtime inbound အရွယ်အစားကန့်သတ်ချက်၏ ပုံသေတန်ဖိုးမှာ `20MB` ဖြစ်ပြီး `channels.slack.mediaMaxMb` ဖြင့် ပြောင်းလဲသတ်မှတ်နိုင်သည်။
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - text chunk များသည် `channels.slack.textChunkLimit` (ပုံသေ 4000) ကို အသုံးပြုသည်
    - `channels.slack.chunkMode="newline"` သတ်မှတ်ပါက paragraph ကို ဦးစားပေး ခွဲခြမ်းခြင်းကို ဖွင့်နိုင်သည်
    - ဖိုင်ပို့ခြင်းများတွင် Slack upload API များကို အသုံးပြုပြီး thread reply (`thread_ts`) ကိုလည်း ထည့်သွင်းနိုင်သည်
    - outbound media ကန့်သတ်ချက်သည် သတ်မှတ်ထားပါက `channels.slack.mediaMaxMb` ကို လိုက်နာမည်; မဟုတ်ပါက channel ပို့ခြင်းများသည် media pipeline မှ MIME-kind ပုံသေတန်ဖိုးများကို အသုံးပြုမည်
  
</Accordion>

  <Accordion title="Delivery targets">
    ပိုမိုသင့်လျော်သော explicit target များ:

    ```
    - DM များအတွက် `user:<id>`
    - channel များအတွက် `channel:<id>`
    
    Slack DM များကို user target များသို့ ပို့ရာတွင် Slack conversation API များဖြင့် ဖွင့်လှစ်သည်။
    ```

  
</Accordion>
</AccordionGroup>

## ကန့်သတ်ချက်များ

Slack action များကို `channels.slack.actions.*` ဖြင့် ထိန်းချုပ်သည်။

လက်ရှိ Slack tooling တွင် ရရှိနိုင်သော action group များ:

| အုပ်စု     | Default |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## Event များနှင့် လုပ်ဆောင်မှုဆိုင်ရာ အပြုအမူများ

- Message ပြင်ဆင်မှု/ဖျက်မှု/thread broadcast များကို system event များအဖြစ် map လုပ်ထားသည်။
- Reaction ထည့်ခြင်း/ဖယ်ရှားခြင်း event များကို system event များအဖြစ် map လုပ်ထားသည်။
- Member ဝင်/ထွက်ခြင်း၊ channel ဖန်တီးခြင်း/အမည်ပြောင်းခြင်း နှင့် pin ထည့်ခြင်း/ဖယ်ရှားခြင်း event များကို system event များအဖြစ် map လုပ်ထားသည်။
- `channel_id_changed` သည် `configWrites` ဖွင့်ထားပါက channel config key များကို ပြောင်းရွှေ့နိုင်သည်။
- Channel topic/purpose metadata ကို ယုံကြည်စိတ်ချရခြင်းမရှိသော context အဖြစ် သတ်မှတ်ပြီး routing context ထဲသို့ ထည့်သွင်းနိုင်သည်။

## Chat အမျိုးအစားအလိုက် threading

`channels.slack.replyToModeByChatType` ကို သတ်မှတ်ခြင်းဖြင့် chat အမျိုးအစားတစ်ခုချင်းစီအလိုက် threading behavior ကို သတ်မှတ်နိုင်ပါသည်။

ဖြေရှင်းမှု အစဉ်လိုက်:

- `channels.slack.accounts.<accountId>`.ackReaction
- `channels.slack.ackReaction`
- `messages.ackReaction`
- agent identity emoji fallback (`agents.list[].identity.emoji`, မရှိပါက "👀")

မှတ်ချက်များ:

- Slack သည် shortcodes များ (ဥပမာ "eyes") ကို မျှော်လင့်ပါသည်။
- channel သို့မဟုတ် account အတွက် reaction ကို ပိတ်ရန် `""` ကို အသုံးပြုပါ။

## Manifest နှင့် scope စစ်ဆေးရန်စာရင်း

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "OpenClaw အတွက် Slack connector"
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
        "description": "OpenClaw သို့ မက်ဆေ့ချ် ပို့ရန်",
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
    `channels.slack.userToken` ကို configure လုပ်ထားပါက ပုံမှန် read scopes များမှာ -

    ```
    - `channels:history`, `groups:history`, `im:history`, `mpim:history`
    - `channels:read`, `groups:read`, `im:read`, `mpim:read`
    - `users:read`
    - `reactions:read`
    - `pins:read`
    - `emoji:read`
    - `search:read` (Slack search read ကို အသုံးပြုထားပါက)
    ```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="No replies in channels">
    အောက်ပါအတိုင်း အစဉ်လိုက် စစ်ဆေးပါ -

    ```
    - `groupPolicy`
    - channel allowlist (`channels.slack.channels`)
    - `requireMention`
    - channel အလိုက် `users` allowlist
    
    အသုံးဝင်သော command များ -
    ```

```bash
openclaw channels status --probe
openclaw logs --follow
openclaw doctor
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    စစ်ဆေးပါ -

    ```
    - `channels.slack.dm.enabled`
    - `channels.slack.dmPolicy` (သို့မဟုတ် legacy `channels.slack.dm.policy`)
    - pairing approvals / allowlist entries
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Slack app settings ထဲတွင် bot + app tokens နှင့် Socket Mode ဖွင့်ထားမှုကို အတည်ပြုပါ။
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    အတည်ပြုပါ -

    ```
    - signing secret
    - webhook path
    - Slack Request URLs (Events + Interactivity + Slash Commands)
    - HTTP account တစ်ခုစီအတွက် သီးခြား `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    သင် ရည်ရွယ်ထားသည်မှာ အောက်ပါတို့အနက် ဘယ်ဟာလဲ စစ်ဆေးပါ -

    ```
    - Slack တွင် ကိုက်ညီသော slash commands များ register လုပ်ထားပြီး native command mode (`channels.slack.commands.native: true`) ကို အသုံးပြုခြင်း
    - သို့မဟုတ် single slash command mode (`channels.slack.slashCommand.enabled: true`)
    
    `commands.useAccessGroups` နှင့် channel/user allowlists များကိုလည်း စစ်ဆေးပါ။
    ```

  
</Accordion>
</AccordionGroup>

## Tool actions

Slack tool actions များကို `channels.slack.actions.*` ဖြင့် gating လုပ်နိုင်ပါသည်။

- [Configuration reference - Slack](/gateway/configuration-reference#slack)

  အရေးကြီးသော Slack fields များ -

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM access: `dm.enabled`, `dmPolicy`, `allowFrom` (legacy: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - channel access: `groupPolicy`, `channels.*`, `channels.*.users`, `channels.*.requireMention`
  - threading/history: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ops/features: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## လုံခြုံရေး မှတ်ချက်များ

- ရေးသားရေးရာ လုပ်ဆောင်ချက်များကို ပုံမှန်အားဖြင့် bot token ဖြင့်သာ လုပ်ဆောင်ပြီး state ပြောင်းလဲမှုများကို app ၏ bot permission နှင့် identity အတွင်း ထိန်းသိမ်းထားပါသည်။
- [Channel routing](/channels/channel-routing)
- user-token writes ကို ဖွင့်ပါက user token တွင် မျှော်လင့်ထားသော write scopes (`chat:write`, `reactions:write`, `pins:write`,
  `files:write`) ပါဝင်ကြောင်း သေချာစေရန်၊ မပါဝင်ပါက အဆိုပါ လုပ်ဆောင်ချက်များ မအောင်မြင်ပါ။
- [Configuration](/gateway/configuration)
- [Slash commands](/tools/slash-commands)

