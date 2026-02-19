---
summary: "Discord ဘော့တ်၏ ပံ့ပိုးမှုအခြေအနေ၊ စွမ်းဆောင်ရည်များနှင့် ဖွဲ့စည်းပြင်ဆင်မှု"
read_when:
  - Discord ချန်နယ် အင်္ဂါရပ်များကို လုပ်ဆောင်နေစဉ်
title: "Discord"
---

# Discord (Bot API)

အခြေအနေ: တရားဝင် Discord bot gateway ကို အသုံးပြု၍ DM နှင့် guild စာသား ချန်နယ်များအတွက် အသင့်ဖြစ်နေပါသည်။

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DMs များသည် ပုံမှန်အားဖြင့် pairing mode ကို အသုံးပြုသည်။
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Native command အပြုအမူနှင့် command catalog။
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Channel များအကြား diagnostics နှင့် ပြုပြင်ရေး လုပ်ငန်းစဉ်။
  
</Card>
</CardGroup>

## အမြန် စတင်ခြင်း

<Steps>
  <Step title="Create a Discord bot and enable intents">
    Discord Developer Portal တွင် application တစ်ခု ဖန်တီးပါ၊ bot တစ်ခု ထည့်ပြီးနောက် အောက်ပါတို့ကို ဖွင့်ပါ -

    ```
    - **မက်ဆေ့ခ်ျ အကြောင်းအရာ ရည်ရွယ်ချက် (Message Content Intent)**
    - **Server Members Intent** (role allowlists နှင့် role-based routing အတွက် လိုအပ်သည်; name-to-ID allowlist ကို ကိုက်ညီစေရန် အကြံပြုသည်)
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
    မူလ account အတွက် env fallback:
    ```

```bash
DISCORD_BOT_TOKEN=...
```

  
</Step>

  <Step title="Invite the bot and start gateway">    မက်ဆေ့ခ်ျ ပို့ရန် ခွင့်ပြုချက်များဖြင့် bot ကို သင့် server သို့ ဖိတ်ခေါ်ပါ။

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
    Pairing code များသည် ၁ နာရီအတွင်း သက်တမ်းကုန်ဆုံးပါသည်။
    ```

  
</Step>
</Steps>

<Note>
Token resolution သည် account အလိုက် ခွဲခြားစီမံပါသည်။ Config ထဲရှိ token တန်ဖိုးများသည် env fallback ထက် ဦးစားပေးပါသည်။ `DISCORD_BOT_TOKEN` ကို မူလ account အတွက်သာ အသုံးပြုပါသည်။
</Note>

## Runtime model

- Gateway သည် Discord connection ကို တာဝန်ယူ ကိုင်တွယ်ပါသည်။
- Reply routing သည် တိကျသေချာပါသည် — Discord မှ ဝင်လာသော reply များကို Discord သို့ ပြန်ပို့ပါသည်။
- ပုံမှန်အားဖြင့် (`session.dmScope=main`), direct chat များသည် agent main session (`agent:main:main`) ကို မျှဝေ အသုံးပြုပါသည်။
- Guild channel များသည် သီးခြား session key များ (`agent:<agentId>:discord:channel:<channelId>`) ဖြစ်ပါသည်။
- Group DM များကို ပုံမှန်အားဖြင့် လျစ်လျူရှုထားပါသည် (`channels.discord.dm.groupEnabled=false`)။
- Native slash command များသည် သီးခြား command session များ (`agent:<agentId>:discord:slash:<userId>`) တွင် လုပ်ဆောင်ပြီး၊ routed conversation session သို့ `CommandTargetSessionKey` ကိုလည်း သယ်ဆောင်ပါသည်။

## Access control နှင့် routing

<Tabs>
  <Tab title="DM policy">    `channels.discord.dmPolicy` သည် DM access ကို ထိန်းချုပ်ပါသည် (legacy: `channels.discord.dm.policy`):
- `pairing` (မူလတန်ဖိုး)
- `allowlist`
- `open` (`channels.discord.allowFrom` တွင် `"*"` ပါဝင်ရန် လိုအပ်သည်; legacy: `channels.discord.dm.allowFrom`)
- `disabled`

DM policy သည် open မဟုတ်ပါက မသိသော user များကို ပိတ်ဆို့မည် (သို့မဟုတ် `pairing` mode တွင် pairing ပြုလုပ်ရန် အသိပေးမည်)။

ပို့ဆောင်ရန်အတွက် DM target format:

- `user:<id>`
- `<@id>` mention

သာမန် ကိန်းဂဏန်း ID များသည် မသေချာသဖြင့် explicit user/channel target kind မဖော်ပြပါက လက်မခံပါ။

    ```
        Guild ကို ကိုင်တွယ်ခြင်းကို `channels.discord.groupPolicy` ဖြင့် ထိန်းချုပ်ပါသည်:
    ```

  
</Tab>

  <Tab title="Guild policy">- `open`
- `allowlist`
- `disabled`

`channels.discord` ရှိနေပါက လုံခြုံရေးအခြေခံတန်ဖိုးမှာ `allowlist` ဖြစ်သည်။

`allowlist` အပြုအမူ:

- guild သည် `channels.discord.guilds` နှင့် ကိုက်ညီရမည် (`id` ကို ဦးစားပေးပြီး slug ကိုလည်း လက်ခံသည်)
- ရွေးချယ်နိုင်သော sender allowlist များ: `users` (ID သို့မဟုတ် နာမည်များ) နှင့် `roles` (role ID များသာ); တစ်ခုခု သတ်မှတ်ထားပါက sender သည် `users` သို့မဟုတ် `roles` နှင့် ကိုက်ညီလျှင် ခွင့်ပြုသည်
- guild တွင် `channels` ကို သတ်မှတ်ထားပါက စာရင်းမပါသော channel များကို ပိတ်ပင်မည်
- guild တွင် `channels` block မရှိပါက allowlist လုပ်ထားသော guild အတွင်းရှိ channel အားလုံးကို ခွင့်ပြုမည်

ဥပမာ:

    ```
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

```json5
`DISCORD_BOT_TOKEN` ကိုသာ သတ်မှတ်ပြီး `channels.discord` block မဖန်တီးပါက runtime fallback သည် `groupPolicy="open"` ဖြစ်မည် (log များတွင် သတိပေးချက် ပြသမည်)။
```

    ```
        ပုံမှန်အားဖြင့် Guild မက်ဆေ့ခ်ျများသည် mention ဖြင့်သာ အလုပ်လုပ်စေပါသည်။
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">Mention detection တွင် ပါဝင်သည်:

- bot ကို တိုက်ရိုက် mention ပြုလုပ်ခြင်း
- သတ်မှတ်ထားသော mention pattern များ (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
- ပံ့ပိုးထားသော အခြေအနေများတွင် bot သို့ reply ပြန်သည့် အပြုအမူ

`requireMention` ကို guild/channel အလိုက် (`channels.discord.guilds...`) တွင် သတ်မှတ်ပါသည်။

Group DM များ:

- မူလတန်ဖိုး: လျစ်လျူရှုထားသည် (`dm.groupEnabled=false`)
- ရွေးချယ်နိုင်သော allowlist ကို `dm.groupChannels` (channel ID သို့မဟုတ် slug များ) ဖြင့် သတ်မှတ်နိုင်သည်

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

### Discord guild member များကို role ID အလိုက် မတူညီသော agent များသို့ လမ်းကြောင်းခွဲရန် `bindings[].match.roles` ကို အသုံးပြုပါ။

Role-based binding များသည် role ID များကိုသာ လက်ခံပြီး peer သို့မဟုတ် parent-peer binding များပြီးနောက်၊ guild-only binding များမတိုင်မီ စစ်ဆေးအကဲဖြတ်ပါသည်။ Binding တစ်ခုတွင် အခြား match field များ (ဥပမာ `peer` + `guildId` + `roles`) ကိုလည်း သတ်မှတ်ထားပါက သတ်မှတ်ထားသော field အားလုံး ကိုက်ညီရမည်။ {
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

```json5
Developer Portal setup
```

## 1. Discord Developer Portal -> **Applications** -> **New Application**
2. **Bot** -> **Add Bot**
3. bot token ကို ကူးယူပါ

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
        **Bot -> Privileged Gateway Intents** တွင် ဖွင့်ပါ:
    ```

  
</Accordion>

  <Accordion title="Privileged intents">- Message Content Intent
- Server Members Intent (အကြံပြုသည်)

Presence intent သည် ရွေးချယ်နိုင်သောအရာဖြစ်ပြီး presence update များကို လက်ခံလိုပါကသာ လိုအပ်သည်။ bot presence (`setPresence`) ကို သတ်မှတ်ရန် member များအတွက် presence update ကို ဖွင့်ရန် မလိုအပ်ပါ။

    ```
        OAuth URL generator:
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">- scopes: `bot`, `applications.commands`

ပုံမှန် အခြေခံ permission များ:

- View Channels
- Send Messages
- Read Message History
- Embed Links
- Attach Files
- Add Reactions (ရွေးချယ်နိုင်သည်)

အထူးလိုအပ်ချက် မရှိပါက `Administrator` ကို မသုံးပါနှင့်။

    ```
        Discord Developer Mode ကို ဖွင့်ပြီးနောက်၊ အောက်ပါတို့ကို ကူးယူပါ:
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Enable Discord Developer Mode, then copy:

    ```
    - server ID
    - channel ID
    - user ID
    
    ယုံကြည်စိတ်ချရသော audit နှင့် probe များအတွက် OpenClaw config တွင် numeric ID များကို အသုံးပြုရန် အကြံပြုပါသည်။
    ```

  
</Accordion>
</AccordionGroup>

## Native commands နှင့် command auth

- `commands.native` ၏ မူလတန်ဖိုးမှာ `"auto"` ဖြစ်ပြီး Discord အတွက် ဖွင့်ထားပါသည်။
- Channel တစ်ခုချင်းအလိုက် override ပြုလုပ်ရန်: `channels.discord.commands.native`.
- `commands.native=false` သည် ယခင်က register လုပ်ထားသော Discord native commands များကို တိတိကျကျ ဖယ်ရှားပေးပါသည်။
- Native command auth သည် ပုံမှန် message handling နှင့် အတူတူသော Discord allowlists/policies များကို အသုံးပြုပါသည်။
- ခွင့်မပြုထားသော အသုံးပြုသူများအတွက်လည်း Commands များကို Discord UI တွင် မြင်နိုင်နိုင်သော်လည်း၊ execute လုပ်သည့်အခါ OpenClaw auth ကို စစ်ဆေးပြီး "not authorized" ဟု ပြန်လည်ပေးပို့ပါမည်။

Command စာရင်းနှင့် လုပ်ဆောင်ပုံများအတွက် [Slash commands](/tools/slash-commands) ကို ကြည့်ပါ။

## Retry policy

<AccordionGroup>
  <Accordion title="Reply tags and native replies">    Discord သည် agent output ထဲတွင် reply tags များကို ထောက်ပံ့ပါသည်:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    `channels.discord.replyToMode` ဖြင့် ထိန်းချုပ်ပါသည်:
    
    - `off` (မူလတန်ဖိုး)
    - `first`
    - `all`
    
    မှတ်ချက်: `off` သည် အလိုအလျောက် reply threading ကို ပိတ်ထားပါသည်။ သို့သော် `[[reply_to_*]]` tags များကို တိတိကျကျ အသုံးပြုထားပါက ဆက်လက် လေးစားလုပ်ဆောင်ပါသည်။
    
    Agent များသည် သီးသန့် message များကို ရည်ညွှန်းနိုင်ရန် Message ID များကို context/history ထဲတွင် ဖော်ပြပေးထားပါသည်။
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">    Guild history context:

    ```
    - `channels.discord.historyLimit` မူလတန်ဖိုး `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` သည် ပိတ်ထားခြင်းဖြစ်သည်
    
    DM history ထိန်းချုပ်မှုများ:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Thread အပြုအမူ:
    
    - Discord threads များကို channel sessions အဖြစ် route လုပ်ပါသည်
    - parent thread metadata ကို parent-session linkage အတွက် အသုံးပြုနိုင်ပါသည်
    - thread-specific entry မရှိပါက thread config သည် parent channel config ကို အမွေဆက်ခံပါသည်
    
    Channel topics များကို **untrusted** context အဖြစ် (system prompt အဖြစ်မဟုတ်ဘဲ) ထည့်သွင်းပါသည်။
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">    Per-guild reaction notification mode:

    ```
    - `off`
    - `own` (မူလတန်ဖိုး)
    - `all`
    - `allowlist` (`guilds.<id>.users` ကို အသုံးပြုသည်)
    
    Reaction events များကို system events အဖြစ် ပြောင်းလဲပြီး route လုပ်ထားသော Discord session သို့ ချိတ်ဆက်ပါသည်။
    ```

  
</Accordion>

  <Accordion title="Ack reactions">`ackReaction` သည် OpenClaw မှ inbound message ကို လုပ်ဆောင်နေစဉ် acknowledgement emoji တစ်ခု ပို့ပေးပါသည်။

    ```
    Resolution order:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, မရှိပါက "👀")
    
    မှတ်ချက်များ:
    
    - Discord သည် unicode emoji သို့မဟုတ် custom emoji name များကို လက်ခံပါသည်။
    - Channel သို့မဟုတ် account အတွက် reaction ကို ပိတ်လိုပါက `""` ကို အသုံးပြုပါ။
    ```

  
</Accordion>

  <Accordion title="Config writes">    Channel မှ စတင်သော config write များကို မူလအနေဖြင့် ဖွင့်ထားပါသည်။

    ```
    ဤအရာသည် `/config set|unset` flow များ (command features များ ဖွင့်ထားသည့်အခါ) ကို သက်ရောက်စေပါသည်။
    
    Disable:
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

  <Accordion title="Gateway proxy">    Discord gateway WebSocket traffic ကို HTTP(S) proxy မှတဆင့် route လုပ်ရန် `channels.discord.proxy` ကို အသုံးပြုပါ။

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

  <Accordion title="PluralKit support">    Proxied messages များကို system member identity နှင့် map လုပ်ရန် PluralKit resolution ကို ဖွင့်ပါ:

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; private systems များအတွက် လိုအပ်သည်
      },
    },
  },
}
```

    ```
    မှတ်ချက်များ:
    
    - allowlists တွင် `pk:<memberId>` ကို အသုံးပြုနိုင်သည်
    - member display names များကို name/slug ဖြင့် ကိုက်ညီမှု စစ်ဆေးပါသည်
    - lookups များသည် မူရင်း message ID ကို အသုံးပြုပြီး အချိန်ကာလကန့်သတ်ထားပါသည်
    - lookup မအောင်မြင်ပါက proxied messages များကို bot messages အဖြစ် သတ်မှတ်ပြီး `allowBots=true` မထားလျှင် ဖယ်ရှားပါသည်
    ```

  
</Accordion>

  <Accordion title="Presence configuration">    Status သို့မဟုတ် activity field တစ်ခုခုကို သတ်မှတ်ထားသောအခါမှသာ Presence updates ကို အသုံးချပါသည်။

    ```
    Status only ဥပမာ:
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
    Activity ဥပမာ (custom status သည် မူလ activity type ဖြစ်သည်):
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
    Streaming ဥပမာ:
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
    - 1: Streaming (`activityUrl` လိုအပ်သည်)
    - 2: Listening
    - 3: Watching
    - 4: Custom (activity text ကို status state အဖြစ် အသုံးပြုသည်; emoji ကို မဖြစ်မနေ မလိုအပ်ပါ)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    Discord သည် DMs အတွင်း button-based exec approvals များကို ထောက်ပံ့ပြီး မူရင်း channel တွင် approval prompt များကို ရွေးချယ်နိုင်စွာ တင်ပို့နိုင်ပါသည်။

    ```
    Config path:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, မူလတန်ဖိုး: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    `target` ကို `channel` သို့မဟုတ် `both` ဟု သတ်မှတ်ထားပါက approval prompt ကို channel တွင် မြင်နိုင်ပါသည်။ သတ်မှတ်ထားသော approvers များသာ button များကို အသုံးပြုနိုင်ပြီး အခြားအသုံးပြုသူများသည် ephemeral denial ကို လက်ခံရရှိပါမည်။ Approval prompt များတွင် command text ပါဝင်သောကြောင့် ယုံကြည်ရသော channel များတွင်သာ channel delivery ကို ဖွင့်ပါ။ Session key မှ channel ID ကို မထုတ်ယူနိုင်ပါက OpenClaw သည် DM delivery သို့ ပြန်လည် fallback လုပ်ပါသည်။
    
    Approval များသည် unknown approval IDs ဖြင့် မအောင်မြင်ပါက approver list နှင့် feature enablement ကို စစ်ဆေးပါ။
    
    ဆက်စပ်စာရွက်စာတမ်း: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Tools နှင့် action gates

Discord message actions တွင် messaging, channel admin, moderation, presence နှင့် metadata actions များ ပါဝင်ပါသည်။

အခြေခံ ဥပမာများ:

- messaging: `sendMessage`, `readMessages`, `editMessage`, `deleteMessage`, `threadReply`
- reactions: `react`, `reactions`, `emojiList`
- moderation: `timeout`, `kick`, `ban`
- presence: `setPresence`

`channels.discord.actions.*` အောက်တွင် Action gates များရှိသည်။

Default gate လုပ်ဆောင်ပုံ:

| Action group                                                                                                                                                             | Default  |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- |
| reactions, messages, threads, pins, polls, search, memberInfo, roleInfo, channelInfo, channels, voiceStatus, events, stickers, emojiUploads, stickerUploads, permissions | enabled  |
| roles                                                                                                                                                                    | disabled |
| moderation                                                                                                                                                               | disabled |
| presence                                                                                                                                                                 | disabled |

## Components v2 UI

OpenClaw သည် exec approvals နှင့် cross-context markers များအတွက် Discord components v2 ကို အသုံးပြုပါသည်။ Discord message actions များသည် custom UI အတွက် `components` ကိုလည်း လက်ခံနိုင်သည် (အဆင့်မြင့်; Carbon component instances လိုအပ်သည်)၊ legacy `embeds` များကိုလည်း အသုံးပြုနိုင်သေးသော်လည်း အကြံပြုထားခြင်းမရှိပါ။

- `channels.discord.ui.components.accentColor` သည် Discord component containers များတွင် အသုံးပြုမည့် accent color (hex) ကို သတ်မှတ်ပေးသည်။
- အကောင့်တစ်ခုချင်းစီအလိုက် `channels.discord.accounts.<id>` တွင် သတ်မှတ်နိုင်သည်။.ui.components.accentColor\`.
- `components v2` ရှိပါက `embeds` များကို လျစ်လျူရှုမည်ဖြစ်သည်။

ဥပမာ:

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

## Voice messages

Discord voice messages များတွင် waveform preview ပြသပြီး OGG/Opus audio နှင့် metadata လိုအပ်သည်။ OpenClaw သည် waveform ကို အလိုအလျောက် ဖန်တီးပေးသည်၊ သို့သော် audio ဖိုင်များကို စစ်ဆေးခြင်းနှင့် ပြောင်းလဲခြင်း ပြုလုပ်ရန် gateway host တွင် `ffmpeg` နှင့် `ffprobe` ရရှိနိုင်ရမည်။

လိုအပ်ချက်များနှင့် ကန့်သတ်ချက်များ:

- **local file path** ကို ပေးရမည် (URL များကို လက်မခံပါ)။
- စာသား content မထည့်ပါနှင့် (Discord သည် payload တစ်ခုတည်းအတွင်း text + voice message ကို ခွင့်မပြုပါ)။
- မည်သည့် audio format မဆို လက်ခံနိုင်သည်; လိုအပ်ပါက OpenClaw သည် OGG/Opus သို့ ပြောင်းလဲပေးမည်။

ဥပမာ:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Troubleshooting

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - Message Content Intent ကို ဖွင့်ပါ
    - user/member resolution ကို အသုံးပြုမည်ဆိုပါက Server Members Intent ကို ဖွင့်ပါ
    - intents များ ပြောင်းလဲပြီးနောက် gateway ကို restart ပြုလုပ်ပါ
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    - `groupPolicy` ကို စစ်ဆေးပါ
    - `channels.discord.guilds` အောက်ရှိ guild allowlist ကို စစ်ဆေးပါ
    - guild တွင် `channels` map ရှိပါက စာရင်းပြုစုထားသော channels များသာ ခွင့်ပြုသည်
    - `requireMention` လုပ်ဆောင်ပုံနှင့် mention patterns များကို စစ်ဆေးပါ
    
    Useful checks:
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    ပုံမှန် ဖြစ်ပေါ်ရသော အကြောင်းရင်းများ:

    ```
    - ကိုက်ညီသော guild/channel allowlist မရှိဘဲ `groupPolicy="allowlist"` ကို အသုံးပြုခြင်း
    - `requireMention` ကို မှားယွင်းသော နေရာတွင် သတ်မှတ်ထားခြင်း (`channels.discord.guilds` သို့မဟုတ် channel entry အောက်တွင်သာ ထားရမည်)
    - ပို့သူကို guild/channel `users` allowlist မှ ပိတ်ဆို့ထားခြင်း
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">
    `channels status --probe` permission စစ်ဆေးမှုများသည် numeric channel ID များအတွက်သာ အလုပ်လုပ်သည်။

    ```
    slug keys များကို အသုံးပြုပါက runtime matching သည် အလုပ်လုပ်နိုင်သော်လည်း probe သည် permissions ကို အပြည့်အဝ မစစ်ဆေးနိုင်ပါ။
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    - DM ပိတ်ထားသည်: `channels.discord.dm.enabled=false`
    - DM policy ပိတ်ထားသည်: `channels.discord.dmPolicy="disabled"` (legacy: `channels.discord.dm.policy`)
    - `pairing` mode တွင် approval ကို စောင့်ဆိုင်းနေခြင်း
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    ပုံမှန်အားဖြင့် bot မှ ပို့ထားသော မက်ဆေ့ချ်များကို လျစ်လျူရှုသည်။

    ```
    `channels.discord.allowBots=true` ဟု သတ်မှတ်ပါက loop behavior မဖြစ်စေရန် တင်းကျပ်သော mention နှင့် allowlist စည်းမျဉ်းများကို အသုံးပြုပါ။
    ```

  
</Accordion>
</AccordionGroup>

## Configuration reference အညွှန်းများ

အဓိက reference:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

အချက်အလက်အရေးကြီးသော Discord fields:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- command: `commands.native`, `commands.useAccessGroups`, `configWrites`
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## လုံခြုံရေးနှင့် လုပ်ဆောင်ရေးဆိုင်ရာ

- bot token များကို လျှို့ဝှက်အချက်အလက်အဖြစ် ကိုင်တွယ်ပါ (`DISCORD_BOT_TOKEN` ကို supervised environments များတွင် အသုံးပြုရန် အကြံပြုသည်).
- အနည်းဆုံးလိုအပ်သည့် (least-privilege) Discord permissions များသာ ခွင့်ပြုပါ.
- command deploy/state သည် အဟောင်းဖြစ်နေပါက gateway ကို restart လုပ်ပြီး `openclaw channels status --probe` ဖြင့် ပြန်လည်စစ်ဆေးပါ.

## ဆက်စပ်အကြောင်းအရာများ

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- [Slash commands](/tools/slash-commands)

