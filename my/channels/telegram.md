---
title: "Telegram"
---

# Telegram (Bot API)

.users`allowlists နှင့်/သို့မဟုတ်`AGENTS.md`နှင့်`SOUL.md\` ထဲရှိ clear guardrails များဖြင့် bot-to-bot reply loop များကို ကာကွယ်ပါ။ အခြေအနေ: grammY ဖြင့် bot DMs + groups အတွက် production-ready ဖြစ်ပါသည်။

## Quick setup (beginner)

1. မူလအနေဖြင့် long-polling; webhook ကို ရွေးချယ်အသုံးပြုနိုင်သည်။ **@BotFather** ဖြင့် bot တစ်ခု ဖန်တီးပါ ([direct link](https://t.me/BotFather))။
2. Token ကို သတ်မှတ်ပါ။
   - Env: `TELEGRAM_BOT_TOKEN=...`
   - သို့မဟုတ် config: `channels.telegram.botToken: "..."`။
   - နှစ်ခုလုံး သတ်မှတ်ထားပါက config ကို ဦးစားပေးမည် (env fallback သည် default-account အတွက်သာ)။
3. Gateway ကို စတင်ပါ။
4. DM ဝင်ရောက်ခွင့်သည် ပုံမှန်အားဖြင့် pairing ဖြစ်ပြီး ပထမဆုံး ဆက်သွယ်သည့်အခါ pairing code ကို အတည်ပြုပါ။

အနည်းဆုံး config:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
    },
  },
}
```

## What it is

- Gateway မှ ပိုင်ဆိုင်သည့် Telegram Bot API ချန်နယ်။
- အတိအကျ သတ်မှတ်ထားသော routing: ပြန်ကြားချက်များသည် Telegram သို့သာ ပြန်သွားပြီး model သည် ချန်နယ်ကို မရွေးချယ်ပါ။
- DMs များသည် agent ၏ အဓိက session ကို မျှဝေပြီး groups များသည် သီးခြားထားရှိသည် (`agent:<agentId>:telegram:group:<chatId>`)။

## Setup (fast path)

### 1. Create a bot token (BotFather)

1. handle သည် အတိအကျ `@BotFather` ဖြစ်ကြောင်း အတည်ပြုပြီး token ကို ကူးယူပါ။ Telegram ကို ဖွင့်ပြီး **@BotFather** နှင့် စကားပြောပါ ([direct link](https://t.me/BotFather))။
2. `/newbot` ကို အလုပ်လုပ်စေပြီး အမည် + `bot` ဖြင့် အဆုံးသတ်သည့် username ကို ထည့်ပါ။
3. Token ကို ကူးယူပြီး လုံခြုံစွာ သိမ်းဆည်းပါ။

ရွေးချယ်နိုင်သော BotFather ဆက်တင်များ:

- `/setjoingroups` — ဘော့တ်ကို group များထဲ ထည့်ခွင့် ပြု/မပြု။
- `/setprivacy` — group မက်ဆေ့ချ်အားလုံးကို ဘော့တ်မြင်နိုင်မလား ထိန်းချုပ်သည်။

### 2. Configure the token (env or config)

ဥပမာ:

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

Env option: `TELEGRAM_BOT_TOKEN=...` (works for the default account).
Env နှင့် config နှစ်ခုလုံး သတ်မှတ်ထားပါက config သည် ဦးစားပေးအဖြစ် အသုံးပြုမည်။

Multi-account ပံ့ပိုးမှု: per-account token များနှင့် optional `name` ကို အသုံးပြု၍ `channels.telegram.accounts` ကို အသုံးပြုပါ။ See [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) for the shared pattern.

3. gateway ကို စတင်ပါ။ Telegram starts when a token is resolved (config first, env fallback).
4. DM access defaults to pairing. Approve the code when the bot is first contacted.
5. Groups အတွက်: ဘော့တ်ကို ထည့်ပါ၊ privacy/admin အပြုအမူကို ဆုံးဖြတ်ပါ (အောက်တွင်)၊ ထို့နောက် mention gating + allowlists ကို ထိန်းချုပ်ရန် `channels.telegram.groups` ကို သတ်မှတ်ပါ။

## Token + privacy + permissions (Telegram side)

### Token creation (BotFather)

- `/newbot` သည် ဘော့တ်ကို ဖန်တီးပြီး token ကို ပြန်ပေးသည် (လျှို့ဝှက်ထားပါ)။
- Token ပေါက်ကြားပါက @BotFather မှတဆင့် revoke/regenerate ပြုလုပ်ပြီး config ကို အပ်ဒိတ်လုပ်ပါ။

### Group message visibility (Privacy Mode)

Telegram bots default to **Privacy Mode**, which limits which group messages they receive.
If your bot must see _all_ group messages, you have two options:

- `/setprivacy` ဖြင့် privacy mode ကို ပိတ်ပါ **သို့မဟုတ်**
- ဘော့တ်ကို group **admin** အဖြစ် ထည့်ပါ (admin ဘော့တ်များသည် မက်ဆေ့ချ်အားလုံးကို လက်ခံရရှိသည်)။

**မှတ်ချက်:** Privacy mode ကို ပြောင်းလဲပြီးပါက အပြောင်းအလဲ အကျိုးသက်ရောက်စေရန် group တစ်ခုချင်းစီမှ ဘော့တ်ကို ဖယ်ရှားပြီး ပြန်ထည့်ရန် Telegram က လိုအပ်ပါသည်။

### Group permissions (admin rights)

Admin status is set inside the group (Telegram UI). Admin bots always receive all
group messages, so use admin if you need full visibility.

## How it works (behavior)

- ဝင်လာသော မက်ဆေ့ချ်များကို reply context နှင့် media placeholders ပါသော shared channel envelope အဖြစ် normalize လုပ်ပါသည်။
- Group ပြန်ကြားချက်များအတွက် ပုံမှန်အားဖြင့် mention လိုအပ်သည် (native @mention သို့မဟုတ် `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`)။
- Multi-agent override: per-agent patterns ကို `agents.list[].groupChat.mentionPatterns` တွင် သတ်မှတ်ပါ။
- ပြန်ကြားချက်များသည် အမြဲတမ်း Telegram chat တူညီရာသို့ ပြန်သွားသည်။
- Long-polling သည် grammY runner ကို per-chat sequencing ဖြင့် အသုံးပြုပြီး စုစုပေါင်း concurrency ကို `agents.defaults.maxConcurrent` ဖြင့် ကန့်သတ်ထားသည်။
- Telegram Bot API သည် read receipts ကို မပံ့ပိုးသဖြင့် `sendReadReceipts` ရွေးချယ်မှု မရှိပါ။

## Draft streaming

OpenClaw သည် Telegram DMs တွင် `sendMessageDraft` ကို အသုံးပြု၍ partial replies များကို stream လုပ်နိုင်သည်။

လိုအပ်ချက်များ:

- @BotFather တွင် ဘော့တ်အတွက် Threaded Mode ကို ဖွင့်ထားရမည် (forum topic mode)။
- Private chat threads သာလျှင် (Telegram သည် ဝင်လာသော မက်ဆေ့ချ်များတွင် `message_thread_id` ကို ထည့်ပေးသည်)။
- `channels.telegram.streamMode` ကို `"off"` အဖြစ် မသတ်မှတ်ထားရ (default: `"partial"`, `"block"` သည် chunked draft updates ကို ဖွင့်ပေးသည်)။

Draft streaming သည် DM-only ဖြစ်ပြီး Telegram သည် groups သို့မဟုတ် channels တွင် မပံ့ပိုးပါ။

## Formatting (Telegram HTML)

- ထွက်သွားသော Telegram စာသားသည် `parse_mode: "HTML"` (Telegram ထောက်ပံ့သော tag subset) ကို အသုံးပြုသည်။
- Markdown အလား input ကို **Telegram-safe HTML** (bold/italic/strike/code/links) အဖြစ် render လုပ်ပြီး block elements များကို newline/bullets ဖြင့် စာသားအဖြစ် ပြောင်းလဲသည်။
- Model များမှ raw HTML ကို Telegram parse error မဖြစ်စေရန် escape လုပ်ထားသည်။
- Telegram သည် HTML payload ကို ပယ်ချပါက OpenClaw သည် message ကို plain text အဖြစ် ပြန်လည်ပို့ပေးသည်။

## Commands (native + custom)

OpenClaw registers native commands (like `/status`, `/reset`, `/model`) with Telegram’s bot menu on startup.
You can add custom commands to the menu via config:

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

## Setup troubleshooting (commands)

- Logs တွင် `setMyCommands failed` တွေ့ရပါက `api.telegram.org` သို့ outbound HTTPS/DNS ကို ပိတ်ထားခြင်း ဖြစ်နိုင်သည်။
- `sendMessage` သို့မဟုတ် `sendChatAction` error များ တွေ့ရပါက IPv6 routing နှင့် DNS ကို စစ်ဆေးပါ။

နောက်ထပ် အကူအညီ: [Channel troubleshooting](/channels/troubleshooting)။

မှတ်ချက်များ:

- Custom commands များသည် **menu entries သာ** ဖြစ်ပြီး OpenClaw သည် အခြားနေရာတွင် handle မလုပ်ပါက အကောင်အထည်မဖော်ပါ။
- အချို့သော command များကို Telegram ၏ command menu တွင် register မလုပ်ဘဲ plugins/skills များမှ ကိုင်တွယ်နိုင်ပါသည်။ ဤ command များကို ရိုက်ထည့်ပါက အလုပ်လုပ်ပါမည် (သို့သော် `/commands` / menu တွင် မပေါ်ပါ)။
- Command အမည်များကို normalize လုပ်ပြီး (ရှေ့က `/` ဖယ်ရှား၊ အောက်စာလုံးပြောင်း) `a-z`, `0-9`, `_` (၁–၃၂ လုံး) နှင့် ကိုက်ညီရမည်။
- Custom commands **cannot override native commands**. Conflicts are ignored and logged.
- `commands.native` ကို ပိတ်ထားပါက custom commands များသာ မှတ်ပုံတင်မည် (မရှိပါက ဖယ်ရှားမည်)။

### Device pairing command များ (`device-pair` plugin)

`device-pair` plugin ကို တပ်ဆင်ထားပါက ဖုန်းအသစ်တစ်လုံးကို pairing လုပ်ရန် Telegram-first flow ကို ထည့်ပေးပါသည် —

1. `/pair` သည် setup code တစ်ခုကို ထုတ်ပေးပါသည် (copy/paste လွယ်စေရန် သီးခြား မက်ဆေ့ချ်အဖြစ် ပို့ပါသည်)။
2. iOS app ထဲတွင် setup code ကို paste လုပ်ပြီး ချိတ်ဆက်ပါ။
3. `/pair approve` သည် နောက်ဆုံး pending device request ကို အတည်ပြုပါသည်။

အသေးစိတ်များ: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios)။

## Limits

- Outbound စာသားကို `channels.telegram.textChunkLimit` (default 4000) အထိ chunk ခွဲပို့သည်။
- Newline chunking (ရွေးချယ်နိုင်): `channels.telegram.chunkMode="newline"` ကို သတ်မှတ်၍ အရှည်အလိုက် ခွဲမီ blank lines (paragraph boundaries) အလိုက် ခွဲနိုင်သည်။
- Media download/upload များကို `channels.telegram.mediaMaxMb` (default 5) ဖြင့် ကန့်သတ်ထားသည်။
- Telegram Bot API requests time out after `channels.telegram.timeoutSeconds` (default 500 via grammY). Set lower to avoid long hangs.
- Group history context uses `channels.telegram.historyLimit` (or `channels.telegram.accounts.*.historyLimit`), falling back to `messages.groupChat.historyLimit`. Set `0` to disable (default 50).
- DM history can be limited with `channels.telegram.dmHistoryLimit` (user turns). Per-user overrides: `channels.telegram.dms["<user_id>"].historyLimit`.

## Group activation modes

By default, the bot only responds to mentions in groups (`@botname` or patterns in `agents.list[].groupChat.mentionPatterns`). To change this behavior:

### Via config (recommended)

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": { requireMention: false }, // always respond in this group
      },
    },
  },
}
```

**Important:** Setting `channels.telegram.groups` creates an **allowlist** - only listed groups (or `"*"`) will be accepted.
Forum topics inherit their parent group config (allowFrom, requireMention, skills, prompts) unless you add per-topic overrides under `channels.telegram.groups.<groupId>.topics.<topicId>`.

Group အားလုံးကို always-respond ဖြင့် ခွင့်ပြုရန်-

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false }, // all groups, always respond
      },
    },
  },
}
```

Group အားလုံးကို mention-only (default) အဖြစ် ထားရန်-

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

### Via command (session-level)

Group ထဲတွင် ပို့ပါ-

- `/activation always` - မက်ဆေ့ချ်အားလုံးကို တုံ့ပြန်
- `/activation mention` - mention လိုအပ် (default)

**Note:** Commands update session state only. For persistent behavior across restarts, use config.

### Getting the group chat ID

Group မှ မည်သည့် မက်ဆေ့ချ်မဆို `@userinfobot` သို့မဟုတ် `@getidsbot` သို့ forward လုပ်ပါက chat ID (ဥပမာ `-1001234567890` ကဲ့သို့ အနုတ်ဂဏန်း) ကို တွေ့ရပါမည်။

**Tip:** သင့် user ID ကို သိရန် ဘော့တ်ကို DM ပို့ပါ၊ pairing မက်ဆေ့ချ်တွင် ပြန်ကြားမည်၊ သို့မဟုတ် commands ဖွင့်ပြီးနောက် `/whoami` ကို အသုံးပြုပါ။

**Privacy note:** `@userinfobot` is a third-party bot. If you prefer, add the bot to the group, send a message, and use `openclaw logs --follow` to read `chat.id`, or use the Bot API `getUpdates`.

## Config writes

ပုံမှန်အားဖြင့် Telegram သည် channel events သို့မဟုတ် `/config set|unset` မှ ဖြစ်ပေါ်လာသော config updates များကို ရေးသားခွင့်ပြုထားသည်။

အောက်ပါအခါများတွင် ဖြစ်ပေါ်သည်-

- A group is upgraded to a supergroup and Telegram emits `migrate_to_chat_id` (chat ID changes). OpenClaw can migrate `channels.telegram.groups` automatically.
- Telegram chat ထဲတွင် `/config set` သို့မဟုတ် `/config unset` ကို run လုပ်သည့်အခါ (`commands.config: true` လိုအပ်သည်)။

ပိတ်ရန်-

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

## Topics (forum supergroups)

Telegram forum topics include a `message_thread_id` per message. OpenClaw:

- Topic တစ်ခုချင်းစီကို သီးခြားထားရန် Telegram group session key တွင် `:topic:<threadId>` ကို ပေါင်းထည့်သည်။
- Topic အတွင်းမှာသာ ဆက်ရှိစေရန် typing indicators နှင့် replies များကို `message_thread_id` ဖြင့် ပို့သည်။
- General topic (thread id `1`) သည် အထူးဖြစ်ပြီး message ပို့ရာတွင် `message_thread_id` ကို ချန်ထားရသည် (Telegram က ပယ်ချသည်)၊ သို့သော် typing indicators တွင် ဆက်လက် ပါဝင်သည်။
- Routing/templating အတွက် template context တွင် `MessageThreadId` + `IsForum` ကို ဖော်ပြပေးသည်။
- Topic-specific configuration is available under `channels.telegram.groups.<chatId>.topics.<threadId>` (skills, allowlists, auto-reply, system prompts, disable).
- Topic configs များသည် override မရှိပါက group settings (requireMention, allowlists, skills, prompts, enabled) ကို ဆက်ခံပါသည်။

Private chats can include `message_thread_id` in some edge cases. OpenClaw keeps the DM session key unchanged, but still uses the thread id for replies/draft streaming when it is present.

## Inline Buttons

Telegram သည် callback buttons ပါသော inline keyboards ကို ပံ့ပိုးပါသည်။

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

Per-account configuration အတွက်-

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

Scopes-

- `off` — inline buttons ပိတ်ထား
- `dm` — DMs သာ (group targets ပိတ်ထား)
- `group` — groups သာ (DM targets ပိတ်ထား)
- `all` — DMs + groups
- `allowlist` — DMs + groups၊ သို့သော် `allowFrom`/`groupAllowFrom` ဖြင့် ခွင့်ပြုထားသော senders များသာ (control commands နှင့် တူညီသော စည်းမျဉ်းများ)

Default: `allowlist`.
Legacy: `capabilities: ["inlineButtons"]` = `inlineButtons: "all"`.

### Sending buttons

Message tool ကို `buttons` parameter ဖြင့် အသုံးပြုပါ-

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

User တစ်ဦးက button ကို နှိပ်သည့်အခါ callback data ကို အောက်ပါ format ဖြင့် agent သို့ မက်ဆေ့ချ်အဖြစ် ပို့သည်-
`callback_data: value`

### Configuration options

Telegram စွမ်းဆောင်ရည်များကို အဆင့် နှစ်ဆင့်ဖြင့် ဖွဲ့စည်းပြင်ဆင်နိုင်သည် (အထက်တွင် ပြထားသော object ပုံစံ; legacy string arrays များကိုလည်း ဆက်လက် ပံ့ပိုးသည်)-

- `channels.telegram.capabilities`: Global default capability config — override မလုပ်ပါက Telegram accounts အားလုံးတွင် သက်ရောက်သည်။
- `channels.telegram.accounts.<account>.capabilities`: Per-account capabilities that override the global defaults for that specific account.

Use the global setting when all Telegram bots/accounts should behave the same. Use per-account configuration when different bots need different behaviors (for example, one account only handles DMs while another is allowed in groups).

## Access control (DMs + groups)

### DM access

- မူလ: `channels.telegram.dmPolicy = "pairing"`။ မသိသော ပို့သူများသည် pairing code ကို လက်ခံရရှိပြီး အတည်ပြုမပြုလုပ်မချင်း မက်ဆေ့ချ်များကို လျစ်လျူရှုမည် (code များသည် ၁ နာရီအတွင်း သက်တမ်းကုန်ဆုံးသည်)။
- အတည်ပြုရန်-
  - `openclaw pairing list telegram`
  - `openclaw pairing approve telegram <CODE>`
- Pairing is the default token exchange used for Telegram DMs. Details: [Pairing](/channels/pairing)
- `channels.telegram.allowFrom` accepts numeric user IDs (recommended) or `@username` entries. It is **not** the bot username; use the human sender’s ID. The wizard accepts `@username` and resolves it to the numeric ID when possible.

#### Finding your Telegram user ID

ပိုမိုလုံခြုံ (third-party bot မလိုအပ်)-

1. Gateway ကို စတင်ပြီး သင့် bot ကို DM ပို့ပါ။
2. `openclaw logs --follow` ကို run လုပ်ပြီး `from.id` ကို ရှာပါ။

အခြားနည်း (official Bot API)-

1. Bot ကို DM ပို့ပါ။
2. Bot token ဖြင့် updates ကို fetch လုပ်ပြီး `message.from.id` ကို ဖတ်ပါ-

   ```bash
   curl "https://api.telegram.org/bot<bot_token>/getUpdates"
   ```

Third-party (privacy နည်း)-

- `@userinfobot` သို့မဟုတ် `@getidsbot` ကို DM ပို့ပြီး ပြန်လာသော user id ကို အသုံးပြုပါ။

### Group access

ထိန်းချုပ်မှု နှစ်ခု သီးခြားရှိသည်-

**1. Which groups are allowed** (group allowlist via `channels.telegram.groups`):

- `groups` config မရှိ = groups အားလုံး ခွင့်ပြု
- `groups` config ရှိ = စာရင်းထဲရှိ groups သို့မဟုတ် `"*"` သာ ခွင့်ပြု
- ဥပမာ: `"groups": { "-1001234567890": {}, "*": {} }` သည် groups အားလုံး ခွင့်ပြု

1. \*\*2. 2. **ဘယ်သူတွေက ပို့ခွင့်ရှိသလဲ** (`channels.telegram.groupPolicy` မှတဆင့် ပို့သူ စစ်ထုတ်ခြင်း):

- `"open"` = ခွင့်ပြုထားသော groups ထဲရှိ senders အားလုံး
- `"allowlist"` = `channels.telegram.groupAllowFrom` ထဲရှိ senders များသာ
- `"disabled"` = group မက်ဆေ့ချ်အားလုံး ပိတ်ထား
  Default သည် `groupPolicy: "allowlist"` ( `groupAllowFrom` မထည့်မချင်း ပိတ်ထားသည်)။

အသုံးပြုသူအများစုအတွက် အကြံပြုချက်: `groupPolicy: "allowlist"` + `groupAllowFrom` + `channels.telegram.groups` တွင် သီးခြား groups များကို စာရင်းထည့်ခြင်း

Group တစ်ခုအတွင်း **မည်သည့် အဖွဲ့ဝင်မဆို** စကားပြောနိုင်ရန် (control commands များကို ခွင့်ပြုထားသော senders များသာ အသုံးပြုနိုင်စေရန် ထိန်းထားပြီး) per-group override ကို သတ်မှတ်ပါ-

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

## Long-polling vs webhook

- Default: long-polling (public URL မလိုအပ်)။
- Webhook mode: `channels.telegram.webhookUrl` နှင့် `channels.telegram.webhookSecret` (ရွေးချယ်နိုင်သော `channels.telegram.webhookPath`) ကို သတ်မှတ်ပါ။
  - Local listener သည် `0.0.0.0:8787` တွင် bind လုပ်ပြီး ပုံမှန်အားဖြင့် `POST /telegram-webhook` ကို serve လုပ်သည်။
  - Public URL က မတူပါက reverse proxy ကို အသုံးပြုပြီး `channels.telegram.webhookUrl` ကို public endpoint သို့ ညွှန်ပါ။

## Reply threading

Telegram သည် tags ဖြင့် threaded replies ကို ရွေးချယ်အသုံးပြုနိုင်သည်-

- `[[reply_to_current]]` -- trigger ဖြစ်သည့် မက်ဆေ့ချ်ကို ပြန်ကြား
- `[[reply_to:<id>]]` -- သတ်မှတ်ထားသော message id ကို ပြန်ကြား

`channels.telegram.replyToMode` ဖြင့် ထိန်းချုပ်သည်-

- `first` (default), `all`, `off`။

## Audio messages (voice vs file)

3. Telegram သည် **အသံမှတ်စုများ** (ဝိုင်းပုံ bubble) နှင့် **အသံဖိုင်များ** (metadata ကတ်) ကို ခွဲခြားသတ်မှတ်ထားသည်။
4. OpenClaw သည် နောက်ပြန်လိုက်ဖက်မှုအတွက် မူလအားဖြင့် audio files ကို အသုံးပြုသည်။

Agent reply များကို voice note bubble အဖြစ် ပို့လိုပါက reply အတွင်း မည်သည့်နေရာမဆို အောက်ပါ tag ကို ထည့်ပါ-

- `[[audio_as_voice]]` — file အစား voice note အဖြစ် audio ကို ပို့သည်။

5. ပို့ဆောင်ပေးသော စာသားထဲမှ tag ကို ဖယ်ရှားထားသည်။ 6. အခြား channel များသည် ဤ tag ကို လျစ်လျူရှုသည်။

Message tool ဖြင့် ပို့ရာတွင် voice-compatible audio `media` URL နှင့်အတူ `asVoice: true` ကို သတ်မှတ်ပါ
(media ရှိပါက `message` သည် ရွေးချယ်နိုင်သည်)-

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

## ဗီဒီယို မက်ဆေ့ချ်များ (video vs video note)

Telegram သည် **video notes** (စက်ဝိုင်းပုံ bubble) နှင့် **video files** (လေးထောင့်ပုံ) ကို ခွဲခြားပါသည်။
OpenClaw သည် ပုံမှန်အားဖြင့် video files ကို အသုံးပြုပါသည်။

Message tool ဖြင့် ပို့ရန်အတွက် video `media` URL နှင့်အတူ `asVideoNote: true` ကို သတ်မှတ်ပါ —

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

(မှတ်ချက်: Video notes တွင် caption မထောက်ပံ့ပါ။ Message စာသား ပေးထားပါက သီးခြား မက်ဆေ့ချ်အဖြစ် ပို့ပါမည်။)

## Stickers

OpenClaw သည် Telegram stickers များကို လက်ခံခြင်းနှင့် ပို့ခြင်းကို intelligent caching ဖြင့် ပံ့ပိုးသည်။

### Receiving stickers

User တစ်ဦးက sticker ပို့လာသောအခါ sticker အမျိုးအစားအလိုက် OpenClaw သည် အောက်ပါအတိုင်း ကိုင်တွယ်သည်-

- 7. **Static stickers (WEBP):** ဒေါင်းလုဒ်လုပ်ပြီး vision မှတဆင့် လုပ်ဆောင်သည်။ 8. စတစ်ကာသည် မက်ဆေ့ခ်ျ အကြောင်းအရာအတွင်း `<media:sticker>` placeholder အဖြစ် ပေါ်လာသည်။
- **Animated stickers (TGS):** ကျော်လွှားထားသည် (Lottie format ကို မပံ့ပိုးပါ)။
- **Video stickers (WEBM):** ကျော်လွှားထားသည် (video format ကို မပံ့ပိုးပါ)။

Sticker လက်ခံစဉ် template context field ရရှိနိုင်သည်-

- `Sticker` — object တစ်ခုပါဝင်ပြီး-
  - `emoji` — sticker နှင့် ဆက်စပ်သော emoji
  - `setName` — sticker set အမည်
  - `fileId` — Telegram file ID (sticker ကို ပြန်ပို့ရန်)
  - `fileUniqueId` — cache lookup အတွက် stable ID
  - `cachedDescription` — ရရှိနိုင်ပါက cached vision description

### Sticker cache

9. စတစ်ကာများကို AI ၏ vision စွမ်းရည်များဖြင့် လုပ်ဆောင်ပြီး ဖော်ပြချက်များ ထုတ်လုပ်သည်။ 10. တူညီသော စတစ်ကာများကို မကြာခဏ ထပ်မံပို့လေ့ရှိသောကြောင့် OpenClaw သည် မလိုအပ်သော API ခေါ်ဆိုမှုများကို ရှောင်ရှားရန် ဤဖော်ပြချက်များကို cache ထားသည်။

**အလုပ်လုပ်ပုံ:**

1. 11. **ပထမအကြိမ်တွေ့ရှိမှု:** စတစ်ကာ ပုံရိပ်ကို vision ခွဲခြမ်းစိတ်ဖြာရန် AI သို့ ပို့သည်။ 12. AI သည် ဖော်ပြချက်တစ်ခု ထုတ်လုပ်သည် (ဥပမာ — "A cartoon cat waving enthusiastically")။
2. **Cache သိမ်းဆည်းခြင်း:** Description ကို sticker ၏ file ID၊ emoji နှင့် set name နှင့်အတူ သိမ်းဆည်းသည်။
3. 13. **နောက်တစ်ကြိမ်တွေ့ရှိမှုများ:** တူညီသော စတစ်ကာကို ထပ်မံတွေ့သောအခါ cache ထားသော ဖော်ပြချက်ကို တိုက်ရိုက် အသုံးပြုသည်။ 14. ပုံရိပ်ကို AI သို့ မပို့တော့ပါ။

**Cache location:** `~/.openclaw/telegram/sticker-cache.json`

**Cache entry format:**

```json
{
  "fileId": "CAACAgIAAxkBAAI...",
  "fileUniqueId": "AgADBAADb6cxG2Y",
  "emoji": "👋",
  "setName": "CoolCats",
  "description": "A cartoon cat waving enthusiastically",
  "cachedAt": "2026-01-15T10:30:00.000Z"
}
```

**အကျိုးကျေးဇူးများ:**

- Sticker တူများအတွက် vision API ခေါ်ဆိုမှုများ ထပ်ခါတလဲလဲ မဖြစ်စေရန် API ကုန်ကျစရိတ် လျော့ချသည်
- Cached stickers များအတွက် တုံ့ပြန်ချိန် ပိုမိုမြန်ဆန်သည် (vision processing မလို)
- Cached descriptions အပေါ်အခြေခံ၍ sticker search လုပ်နိုင်စေသည်

15. စတစ်ကာများ လက်ခံရရှိလာသည်နှင့်အမျှ cache ကို အလိုအလျောက် ဖြည့်သွင်းသည်။ 16. လက်ဖြင့် cache စီမံခန့်ခွဲရန် မလိုအပ်ပါ။

### Sending stickers

17. agent သည် `sticker` နှင့် `sticker-search` actions များကို အသုံးပြုပြီး စတစ်ကာများကို ပို့ခြင်းနှင့် ရှာဖွေခြင်း ပြုလုပ်နိုင်သည်။ 18. ဤအရာများသည် မူလအားဖြင့် ပိတ်ထားပြီး config ထဲတွင် ဖွင့်ရပါမည်:

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

**Sticker ပို့ရန်:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

Parameters-

- 19. `fileId` (လိုအပ်) — စတစ်ကာ၏ Telegram file ID ဖြစ်သည်။ 20. စတစ်ကာ လက်ခံရရှိသည့်အခါ `Sticker.fileId` မှ သို့မဟုတ် `sticker-search` ရလဒ်မှ ရယူနိုင်သည်။
- `replyTo` (ရွေးချယ်နိုင်) — ပြန်ကြားမည့် message ID။
- `threadId` (ရွေးချယ်နိုင်) — forum topics အတွက် message thread ID။

**Sticker ရှာဖွေရန်:**

Agent သည် cached stickers များကို description၊ emoji သို့မဟုတ် set name အလိုက် ရှာဖွေနိုင်သည်-

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

Cache မှ ကိုက်ညီသော stickers များကို ပြန်ပေးသည်-

```json5
{
  ok: true,
  count: 2,
  stickers: [
    {
      fileId: "CAACAgIAAxkBAAI...",
      emoji: "👋",
      description: "A cartoon cat waving enthusiastically",
      setName: "CoolCats",
    },
  ],
}
```

ရှာဖွေမှုသည် description စာသား၊ emoji အက္ခရာများနှင့် set name များအပေါ် fuzzy matching ကို အသုံးပြုသည်။

**Threading ပါသော ဥပမာ:**

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "-1001234567890",
  fileId: "CAACAgIAAxkBAAI...",
  replyTo: 42,
  threadId: 123,
}
```

## Streaming (drafts)

21. agent က တုံ့ပြန်မှု ထုတ်လုပ်နေစဉ် Telegram သည် **draft bubbles** ကို stream လုပ်ပေးနိုင်သည်။
22. OpenClaw သည် Bot API `sendMessageDraft` ကို (အမှန်တကယ် မက်ဆေ့ချ် မဟုတ်ပါ) အသုံးပြုပြီး ထို့နောက် နောက်ဆုံး တုံ့ပြန်ချက်ကို ပုံမှန် မက်ဆေ့ချ်အဖြစ် ပို့သည်။

လိုအပ်ချက်များ (Telegram Bot API 9.3+)-

- **Topics ဖွင့်ထားသော private chats** (bot အတွက် forum topic mode)။
- ဝင်လာသော မက်ဆေ့ချ်များတွင် `message_thread_id` ပါဝင်ရမည် (private topic thread)။
- Groups/supergroups/channels များတွင် streaming ကို လျစ်လျူရှုသည်။

Config-

- `channels.telegram.streamMode: "off" | "partial" | "block"` (default: `partial`)
  - `partial`: streaming စာသား အသစ်ဆုံးဖြင့် draft bubble ကို update လုပ်သည်။
  - `block`: block ကြီးများ (chunked) ဖြင့် update လုပ်သည်။
  - `off`: draft streaming ပိတ်ထားသည်။
- ရွေးချယ်နိုင် ( `streamMode: "block"` အတွက်သာ)-
  - 23. `channels.telegram.draftChunk: { minChars?, maxChars?, breakPreference?` 24. }\`
    - defaults: `minChars: 200`, `maxChars: 800`, `breakPreference: "paragraph"` (`channels.telegram.textChunkLimit` အထိ ကန့်သတ်)။

25. မှတ်ချက်: draft streaming သည် **block streaming** (channel messages) နှင့် သီးခြားဖြစ်သည်။
26. Block streaming သည် မူလအားဖြင့် ပိတ်ထားပြီး draft updates အစား Telegram မက်ဆေ့ချ်များကို စောစီးစွာ လိုချင်ပါက `channels.telegram.blockStreaming: true` ကို လိုအပ်သည်။

Reasoning stream (Telegram only)-

- `/reasoning stream` သည် reply ထုတ်လုပ်နေစဉ် reasoning ကို draft bubble ထဲ stream လုပ်ပြီး နောက်ဆုံးတွင် reasoning မပါသော final answer ကို ပို့သည်။
- 27. `channels.telegram.streamMode` သည် `off` ဖြစ်ပါက reasoning stream ကို ပိတ်ထားသည်။
  28. ပိုမိုသော အကြောင်းအရာ: [Streaming + chunking](/concepts/streaming).

## Retry policy

29. အပြင်ထွက် Telegram API ခေါ်ဆိုမှုများသည် ယာယီ network/429 အမှားများတွင် exponential backoff နှင့် jitter ဖြင့် retry လုပ်သည်။ 30. `channels.telegram.retry` ဖြင့် ချိန်ညှိနိုင်သည်။ 31. [Retry policy](/concepts/retry) ကို ကြည့်ပါ။

## Agent tool (messages + reactions)

- Tool: `telegram` with `sendMessage` action (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)။
- Tool: `telegram` with `react` action (`chatId`, `messageId`, `emoji`)။
- Tool: `telegram` with `deleteMessage` action (`chatId`, `messageId`)။
- Reaction removal semantics: [/tools/reactions](/tools/reactions) ကို ကြည့်ပါ။
- Tool gating: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (default: enabled) နှင့် `channels.telegram.actions.sticker` (default: disabled)။

## Reaction notifications

32. **Reactions အလုပ်လုပ်ပုံ:**
    Telegram reactions များသည် မက်ဆေ့ချ် payload အတွင်းရှိ property မဟုတ်ဘဲ **သီးခြား `message_reaction` event များ** အဖြစ် ရောက်ရှိလာသည်။ 33. အသုံးပြုသူတစ်ဦး reaction ထည့်သည့်အခါ OpenClaw သည်:

1. Telegram API မှ `message_reaction` update ကို လက်ခံရရှိသည်
2. ၎င်းကို အောက်ပါ format ဖြင့် **system event** အဖြစ် ပြောင်းလဲသည်- `"Telegram reaction added: {emoji} by {user} on msg {id}"`
3. ပုံမှန် မက်ဆေ့ချ်များနှင့် **session key တူညီစွာ** system event ကို queue ထဲ ထည့်သည်
4. ထို conversation ထဲသို့ နောက်ထပ် မက်ဆေ့ချ် ဝင်လာသည့်အခါ system events များကို drain လုပ်ပြီး agent context ရှေ့တွင် ပေါင်းထည့်သည်

Agent သည် reactions များကို message metadata အဖြစ် မမြင်ဘဲ conversation history ထဲရှိ **system notifications** အဖြစ်သာ မြင်သည်။

**Configuration:**

- `channels.telegram.reactionNotifications`: မည်သည့် reactions များကို notification အဖြစ် ထုတ်လုပ်မည်ကို ထိန်းချုပ်သည်
  - `"off"` — reactions အားလုံးကို လျစ်လျူရှု
  - `"own"` — bot မက်ဆေ့ချ်များကို users react လုပ်သည့်အခါ အသိပေး (best-effort; in-memory) (default)
  - `"all"` — reactions အားလုံးကို အသိပေး

- `channels.telegram.reactionLevel`: Agent ၏ reaction စွမ်းဆောင်ရည်ကို ထိန်းချုပ်သည်
  - `"off"` — agent သည် reactions မလုပ်နိုင်
  - `"ack"` — bot သည် acknowledgment reactions ပို့သည် (processing အတွင်း 👀) (default)
  - `"minimal"` — agent သည် တစ်ခါတစ်ရံ reaction လုပ်နိုင် (လမ်းညွှန်: ၅–၁၀ exchanges လျှင် ၁ ကြိမ်)
  - `"extensive"` — သင့်လျော်သည့်အခါ reaction ကို မကြာခဏ လုပ်နိုင်

34. **Forum groups:** forum groups အတွင်းရှိ reactions များတွင် `message_thread_id` ပါဝင်ပြီး `agent:main:telegram:group:{chatId}:topic:{threadId}` ကဲ့သို့သော session keys ကို အသုံးပြုသည်။ 35. ဤအရာသည် တူညီသော topic အတွင်းရှိ reactions နှင့် messages များကို အတူတကွ ထိန်းသိမ်းထားနိုင်စေသည်။

**ဥပမာ config:**

```json5
{
  channels: {
    telegram: {
      reactionNotifications: "all", // See all reactions
      reactionLevel: "minimal", // Agent can react sparingly
    },
  },
}
```

**လိုအပ်ချက်များ:**

- Telegram bots များသည် `allowed_updates` တွင် `message_reaction` ကို အထူးတောင်းဆိုရမည် (OpenClaw မှ အလိုအလျောက် သတ်မှတ်ပေးသည်)
- Webhook mode တွင် reactions များကို webhook `allowed_updates` ထဲတွင် ပါဝင်သည်
- Polling mode တွင် reactions များကို `getUpdates` `allowed_updates` ထဲတွင် ပါဝင်သည်

## Delivery targets (CLI/cron)

- Target အဖြစ် chat id (`123456789`) သို့မဟုတ် username (`@name`) ကို အသုံးပြုနိုင်သည်။
- ဥပမာ: `openclaw message send --channel telegram --target 123456789 --message "hi"`။

## Troubleshooting

**Group ထဲတွင် mention မပါသော မက်ဆေ့ချ်များကို ဘော့တ် မတုံ့ပြန်ပါက:**

- `channels.telegram.groups.*.requireMention=false` ကို သတ်မှတ်ထားပါက Telegram Bot API **privacy mode** ကို ပိတ်ထားရမည်။
  - BotFather: `/setprivacy` → **Disable** (ပြီးနောက် group မှ ဘော့တ်ကို ဖယ်ရှားပြီး ပြန်ထည့်ပါ)
- `openclaw channels status` သည် config တွင် mention မပါသော group messages များကို မျှော်မှန်းထားသည့်အခါ warning ပြသည်။
- `openclaw channels status --probe` သည် သတ်မှတ်ထားသော numeric group IDs များအတွက် membership ကို ထပ်မံ စစ်ဆေးနိုင်သည် (wildcard `"*"` စည်းမျဉ်းများကို audit မလုပ်နိုင်ပါ)။
- Quick test: `/activation always` (session-only; persistent အတွက် config ကို အသုံးပြုပါ)

**Group မက်ဆေ့ချ်များကို လုံးဝ မမြင်ပါက:**

- `channels.telegram.groups` ကို သတ်မှတ်ထားပါက group ကို စာရင်းထဲ ထည့်ထားရမည် သို့မဟုတ် `"*"` ကို အသုံးပြုရမည်
- @BotFather → "Group Privacy" တွင် Privacy Settings ကို **OFF** ဖြစ်ကြောင်း စစ်ဆေးပါ
- ဘော့တ်သည် အဖွဲ့ဝင်အဖြစ် အမှန်တကယ် ပါဝင်နေကြောင်း စစ်ဆေးပါ (admin ဖြစ်ပေမယ့် read access မရှိခြင်း မဖြစ်ရ)
- Gateway logs ကို စစ်ဆေးပါ: `openclaw logs --follow` ("skipping group message" ကို ရှာပါ)

**Mentions ကိုသာ တုံ့ပြန်ပြီး `/activation always` မတုံ့ပြန်ပါက:**

- `/activation` command သည် session state ကိုသာ ပြောင်းလဲပြီး config တွင် မသိမ်းဆည်းပါ
- အမြဲတမ်း အကျိုးသက်ရောက်စေရန် `channels.telegram.groups` တွင် `requireMention: false` ဖြင့် group ကို ထည့်ပါ

**`/status` ကဲ့သို့သော commands မအလုပ်လုပ်ပါက:**

- သင့် Telegram user ID သည် pairing သို့မဟုတ် `channels.telegram.allowFrom` ဖြင့် ခွင့်ပြုထားကြောင်း သေချာပါစေ
- `groupPolicy: "open"` ပါသော groups တွင်တောင် commands များသည် authorization လိုအပ်သည်

**Node 22+ တွင် long-polling ချက်ချင်း ရပ်သွားပါက (proxies/custom fetch များတွင် မကြာခဏ ဖြစ်တတ်):**

- Node 22+ သည် `AbortSignal` instances များကို ပိုမို တင်းကြပ်စွာ ကိုင်တွယ်ပြီး foreign signals များကြောင့် `fetch` calls များကို ချက်ချင်း abort လုပ်နိုင်သည်။
- Abort signals ကို normalize လုပ်ထားသော OpenClaw build သို့ upgrade လုပ်ပါ၊ သို့မဟုတ် upgrade မလုပ်နိုင်သေးပါက Node 20 တွင် gateway ကို chạy ပါ။

36. **Bot စတင်ပြီးနောက် တိတ်တဆိတ် မတုံ့ပြန်တော့ခြင်း (သို့မဟုတ် `HttpError: Network request ...` ကို log တွင် တွေ့ရခြင်း):** 37.

- 38. အချို့ host များသည် `api.telegram.org` ကို ပထမဦးဆုံး IPv6 သို့ resolve လုပ်သည်။ 39. သင့် server တွင် IPv6 egress အလုပ်မလုပ်ပါက grammY သည် IPv6-only requests များတွင် ချိတ်မိနေတတ်သည်။
- ဖြေရှင်းရန် IPv6 egress ကို ဖွင့်ပါ **သို့မဟုတ်** `api.telegram.org` အတွက် IPv4 resolution ကို အတင်းသုံးပါ (ဥပမာ IPv4 A record ကို အသုံးပြုသော `/etc/hosts` entry ထည့်ခြင်း သို့မဟုတ် OS DNS stack တွင် IPv4 ကို ဦးစားပေးခြင်း)၊ ထို့နောက် gateway ကို ပြန်စတင်ပါ။
- Quick check: DNS return ကို အတည်ပြုရန် `dig +short api.telegram.org A` နှင့် `dig +short api.telegram.org AAAA` ကို အသုံးပြုပါ။

## Configuration reference (Telegram)

အပြည့်အစုံ ဖွဲ့စည်းပြင်ဆင်မှု: [Configuration](/gateway/configuration)

Provider options-

- `channels.telegram.enabled`: channel startup ကို ဖွင့်/ပိတ်။
- `channels.telegram.botToken`: bot token (BotFather)။
- `channels.telegram.tokenFile`: token ကို file path မှ ဖတ်ရန်။
- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing)။
- 40. `channels.telegram.allowFrom`: DM allowlist (ids/usernames)။ 41. `open` သည် `"*"` ကို လိုအပ်သည်။
- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (default: allowlist)။
- `channels.telegram.groupAllowFrom`: group sender allowlist (ids/usernames)။
- `channels.telegram.groups`: per-group defaults + allowlist (global defaults အတွက် `"*"` ကို အသုံးပြုပါ)။
  - 44. `channels.telegram.groups.<id>`43. `.groupPolicy`: groupPolicy (`open | allowlist | disabled`) အတွက် group တစ်ခုချင်းစီအလိုက် override။
  - 42. `channels.telegram.groups.<id>`45. `.requireMention`: mention gating မူလသတ်မှတ်ချက်။
  - 44. `channels.telegram.groups.<id>`47. `.skills`: skill filter (မထည့်ပါက = skill အားလုံး, အလွတ် = မရှိ)။
  - 46. `channels.telegram.groups.<id>`49. `.allowFrom`: group တစ်ခုချင်းစီအတွက် sender allowlist override။
  - 48. `channels.telegram.groups.<id>`.systemPrompt\`: extra system prompt for the group.
  - 50. `channels.telegram.groups.<id>`.enabled`: disable the group when `false\`.
  - `channels.telegram.groups.<id>.topics.<threadId>.*`: per-topic overrides (same fields as group).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: per-topic override for groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.topics.<threadId>.requireMention`: per-topic mention gating override.
- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (default: allowlist)။
- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: per-account override.
- `channels.telegram.replyToMode`: `off | first | all` (default: `first`)။
- `channels.telegram.textChunkLimit`: outbound chunk size (chars)။
- `channels.telegram.chunkMode`: `length` (default) သို့မဟုတ် blank lines (paragraph boundaries) အလိုက် ခွဲရန် `newline`။
- `channels.telegram.linkPreview`: outbound messages များအတွက် link previews ကို ဖွင့်/ပိတ် (default: true)။
- `channels.telegram.streamMode`: `off | partial | block` (draft streaming)။
- `channels.telegram.mediaMaxMb`: inbound/outbound media cap (MB)။
- `channels.telegram.retry`: outbound Telegram API calls အတွက် retry policy (attempts, minDelayMs, maxDelayMs, jitter)။
- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.
- `channels.telegram.proxy`: Bot API calls အတွက် proxy URL (SOCKS/HTTP)။
- `channels.telegram.webhookUrl`: webhook mode ကို ဖွင့် ( `channels.telegram.webhookSecret` လိုအပ်သည်)။
- `channels.telegram.webhookSecret`: webhook secret (webhookUrl သတ်မှတ်ထားပါက လိုအပ်)။
- `channels.telegram.webhookPath`: local webhook path (default `/telegram-webhook`)။
- `channels.telegram.actions.reactions`: Telegram tool reactions ကို gate လုပ်ရန်။
- `channels.telegram.actions.sendMessage`: Telegram tool message sends ကို gate လုပ်ရန်။
- `channels.telegram.actions.deleteMessage`: Telegram tool message deletes ကို gate လုပ်ရန်။
- `channels.telegram.actions.sticker`: Telegram sticker actions — send and search (default: false)။
- `channels.telegram.reactionNotifications`: `off | own | all` — system events ဖြစ်စေမည့် reactions များကို ထိန်းချုပ် (default: မသတ်မှတ်ပါက `own`)။
- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — agent ၏ reaction စွမ်းဆောင်ရည်ကို ထိန်းချုပ် (default: မသတ်မှတ်ပါက `minimal`)။

Related global options-

- `agents.list[].groupChat.mentionPatterns` (mention gating patterns)။
- `messages.groupChat.mentionPatterns` (global fallback)။
- `commands.native` (defaults to `"auto"` → on for Telegram/Discord, off for Slack), `commands.text`, `commands.useAccessGroups` (command behavior). Override with `channels.telegram.commands.native`.
- `messages.responsePrefix`, `messages.ackReaction`, `messages.ackReactionScope`, `messages.removeAckAfterReply`။

