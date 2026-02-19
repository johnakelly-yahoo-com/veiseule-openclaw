---
summary: "Telegram ဘော့တ် အထောက်အပံ့ အခြေအနေ၊ စွမ်းဆောင်ရည်များနှင့် ဖွဲ့စည်းပြင်ဆင်ခြင်း"
read_when:
  - Telegram အင်္ဂါရပ်များ သို့မဟုတ် webhook များအပေါ် အလုပ်လုပ်နေချိန်
title: "Telegram"
---

# Telegram (Bot API)

.users`allowlists နှင့်/သို့မဟုတ်`AGENTS.md`နှင့်`SOUL.md\` ထဲရှိ clear guardrails များဖြင့် bot-to-bot reply loop များကို ကာကွယ်ပါ။ Long polling သည် မူလ(default) mode ဖြစ်ပြီး webhook mode ကို လိုအပ်ပါကသာ အသုံးပြုနိုင်ပါသည်။

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Telegram အတွက် မူလ DM policy သည် pairing ဖြစ်ပါသည်။
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Channel များအကြား diagnostic နှင့် ပြုပြင်ရန် playbooks များ။
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Channel config ပုံစံများနှင့် ဥပမာများ အပြည့်အစုံ။
  
</Card>
</CardGroup>

## အမြန် စတင်ခြင်း

<Steps>
  <Step title="Create the bot token in BotFather">
    Telegram ကို ဖွင့်ပြီး **@BotFather** နှင့် စကားပြောပါ (handle သည် တိတိကျကျ `@BotFather` ဖြစ်ကြောင်း အတည်ပြုပါ)။

    ```
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

  
</Step>

  <Step title="Configure token and DM policy">

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

    ```
    Env fallback: `TELEGRAM_BOT_TOKEN=...` (default account အတွက်သာ)။
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
openclaw gateway
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

    ```
    Pairing codes များသည် ၁ နာရီအကြာတွင် သက်တမ်းကုန်ပါသည်။
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Bot ကို သင့် group ထဲသို့ ထည့်ပြီးနောက် `channels.telegram.groups` နှင့် `groupPolicy` ကို သင့် access model နှင့် ကိုက်ညီအောင် သတ်မှတ်ပါ။
  
</Step>
</Steps>

<Note>
Token resolution အစီအစဉ်သည် account အလိုက် ခွဲခြားထားပါသည်။ လက်တွေ့အသုံးပြုရာတွင် config တန်ဖိုးများသည် env fallback ထက် ဦးစားပေးပြီး `TELEGRAM_BOT_TOKEN` သည် default account အတွက်သာ သက်ရောက်မှုရှိသည်။
</Note>

## Telegram ဘက်ခြမ်း ဆက်တင်များ

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">
    Telegram bots များသည် မူလအနေဖြင့် **Privacy Mode** ကို အသုံးပြုထားပြီး၊ ၎င်းသည် group မက်ဆေ့ချ်များကို လက်ခံရရှိနိုင်မှုကို ကန့်သတ်ပေးသည်။

    ```
    Bot သည် group မက်ဆေ့ချ်အားလုံးကို မြင်နိုင်ရန်လိုအပ်ပါက အောက်ပါနည်းလမ်းများထဲမှ တစ်ခုကို ပြုလုပ်ပါ -
    
    - `/setprivacy` ဖြင့် privacy mode ကို ပိတ်ပါ၊ သို့မဟုတ်
    - bot ကို group admin အဖြစ် သတ်မှတ်ပါ။
    
    Privacy mode ကို ပြောင်းလဲပြီးပါက Telegram မှ ပြောင်းလဲမှုကို အသုံးချနိုင်ရန် group တစ်ခုချင်းစီတွင် bot ကို ဖယ်ရှားပြီး ပြန်လည်ထည့်သွင်းပါ။
    ```

  
</Accordion>

  <Accordion title="Group permissions">
    Admin အဆင့်သတ်မှတ်ခြင်းကို Telegram group settings တွင် ထိန်းချုပ်သည်။

    ```
    Admin bot များသည် group မက်ဆေ့ချ်အားလုံးကို လက်ခံရရှိနိုင်ပြီး၊ အမြဲတမ်း အလုပ်လုပ်နေသော group behavior များအတွက် အသုံးဝင်သည်။
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    - `/setjoingroups` ဖြင့် group ထည့်သွင်းခွင့်ကို ခွင့်ပြု/ငြင်းပယ်
    - `/setprivacy` ဖြင့် group တွင် မြင်နိုင်မှု အပြုအမူကို သတ်မှတ်
    ```

  
</Accordion>
</AccordionGroup>

## ဝင်ရောက်ခွင့် ထိန်းချုပ်မှုနှင့် စတင်အသုံးပြုခြင်း

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` သည် direct message ဝင်ရောက်ခွင့်ကို ထိန်းချုပ်သည် -

    ```
    - `pairing` (default)
    - `allowlist`
    - `open` (`allowFrom` တွင် `"*"` ပါဝင်ရန်လိုအပ်သည်)
    - `disabled`
    
    `channels.telegram.allowFrom` သည် numeric Telegram user ID များကို လက်ခံသည်။ `telegram:` / `tg:` prefix များကို လက်ခံပြီး စံပုံစံသို့ ပြောင်းလဲပေးသည်။
    Onboarding wizard သည် `@username` ကို ထည့်သွင်းခွင့်ပြုပြီး ၎င်းကို numeric ID သို့ ပြောင်းလဲပေးသည်။
    Upgrade ပြုလုပ်ပြီးနောက် config တွင် `@username` allowlist entries များ ပါဝင်နေပါက ၎င်းတို့ကို ဖြေရှင်းရန် `openclaw doctor --fix` ကို လုပ်ဆောင်ပါ (အကောင်းဆုံး ကြိုးစားမှုဖြင့် လုပ်ဆောင်မည်ဖြစ်ပြီး Telegram bot token လိုအပ်သည်)။
    
    ### သင်၏ Telegram user ID ကို ရှာဖွေရန်
    
    ပိုမိုလုံခြုံသောနည်းလမ်း (third-party bot မလိုအပ်ပါ) -
    
    1. သင့် bot သို့ DM ပို့ပါ။
    2. `openclaw logs --follow` ကို လုပ်ဆောင်ပါ။
    3. `from.id` ကို ဖတ်ရှုပါ။
    
    Official Bot API နည်းလမ်း -
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Third-party နည်းလမ်း (ကိုယ်ရေးကိုယ်တာ ပိုနည်းပါးသည်) - `@userinfobot` သို့မဟုတ် `@getidsbot`။
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">
    သီးခြားထိန်းချုပ်မှု နှစ်မျိုး ရှိသည် -

    ```
    1. **မည်သည့် group များကို ခွင့်ပြုမည်** (`channels.telegram.groups`)
       - `groups` config မရှိပါက - group အားလုံးကို ခွင့်ပြုမည်
       - `groups` ကို သတ်မှတ်ထားပါက - allowlist အဖြစ် လုပ်ဆောင်မည် (ID များ သို့မဟုတ် `"*"` ကို ဖော်ပြထားရမည်)
    
    2. **group အတွင်း မည်သည့် ပေးပို့သူများကို ခွင့်ပြုမည်** (`channels.telegram.groupPolicy`)
       - `open`
       - `allowlist` (default)
       - `disabled`
    
    `groupAllowFrom` ကို group sender filtering အတွက် အသုံးပြုသည်။ မသတ်မှတ်ထားပါက Telegram သည် `allowFrom` သို့ ပြန်လည်အသုံးပြုမည်။
    `groupAllowFrom` entries များသည် numeric Telegram user ID များဖြစ်ရမည်။
    
    ဥပမာ - သီးသန့် group တစ်ခုတွင် member မည်သူမဆိုကို ခွင့်ပြုရန် -
    ```

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

  
</Tab>

  <Tab title="Mention behavior">
    မူလအနေဖြင့် group reply များတွင် mention လိုအပ်သည်။

    ```
    Mention ကို အောက်ပါနည်းလမ်းများမှ ရရှိနိုင်သည် -
    
    - native `@botusername` mention၊ သို့မဟုတ်
    - အောက်ပါနေရာများတွင် သတ်မှတ်ထားသော mention patterns များ -
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Session အဆင့် command toggle များ -
    
    - `/activation always`
    - `/activation mention`
    
    ဤ command များသည် session state ကိုသာ အပ်ဒိတ်လုပ်သည်။ အမြဲတမ်းအသုံးပြုရန် config ကို အသုံးပြုပါ။
    
    Persistent config ဥပမာ -
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: false },
      },
    },
  },
}
```

    ```
    Group chat ID ကို ရယူရန် -
    
    - group မက်ဆေ့ချ်တစ်ခုကို `@userinfobot` / `@getidsbot` သို့ forward လုပ်ပါ
    - သို့မဟုတ် `openclaw logs --follow` မှ `chat.id` ကို ဖတ်ရှုပါ
    - သို့မဟုတ် Bot API `getUpdates` ကို စစ်ဆေးပါ
    ```

  
</Tab>
</Tabs>

## Runtime အပြုအမူ

- Telegram ကို gateway process မှ ထိန်းချုပ်သည်။
- Routing သည် သတ်မှတ်ထားသည့်အတိုင်း ဖြစ်ပြီး Telegram မှ ဝင်လာသော မက်ဆေ့ချ်များကို Telegram သို့သာ ပြန်လည်ဖြေကြားမည် (model မှ channel မရွေးချယ်ပါ)။
- Inbound မက်ဆေ့ချ်များကို reply metadata နှင့် media placeholder များ ပါဝင်သည့် shared channel envelope ပုံစံသို့ စံညှိပေးသည်။
- Group session များကို group ID အလိုက် သီးခြားခွဲထားသည်။ Forum topic များတွင် topic များ သီးခြားဖြစ်စေရန် `:topic:<threadId>` ကို ပေါင်းထည့်သည်။
- DM မက်ဆေ့ချ်များတွင် `message_thread_id` ပါဝင်နိုင်ပြီး OpenClaw သည် thread-aware session key များဖြင့် route လုပ်ကာ reply များအတွက် thread ID ကို ထိန်းသိမ်းထားသည်။
- Long polling သည် grammY runner ကို အသုံးပြုပြီး chat တစ်ခုချင်း / thread တစ်ခုချင်း အလိုက် sequencing ပြုလုပ်သည်။ Runner sink ၏ စုစုပေါင်း concurrency ကို `agents.defaults.maxConcurrent` ဖြင့် သတ်မှတ်သည်။
- Telegram Bot API တွင် read-receipt ကို မပံ့ပိုးပါ (`sendReadReceipts` သည် သက်ရောက်မှုမရှိပါ)။

## Feature အညွှန်း

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">
    OpenClaw သည် ယာယီ Telegram မက်ဆေ့ချ်တစ်ခု ပို့ပြီး စာသားရောက်ရှိလာသလို ပြင်ဆင်ကာ partial reply များကို stream လုပ်နိုင်သည်။

    ```
    လိုအပ်ချက် -
    
    - `channels.telegram.streamMode` သည် `"off"` မဖြစ်ရပါ (default - `"partial"`)
    
    Mode များ -
    
    - `off` - live preview မရှိပါ
    - `partial` - partial စာသားများမှ မကြာခဏ preview အပ်ဒိတ်များ
    - `block` - `channels.telegram.draftChunk` ကို အသုံးပြု၍ chunk အလိုက် preview အပ်ဒိတ်များ
    
    `streamMode: "block"` အတွက် `draftChunk` default တန်ဖိုးများ -
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` ကို `channels.telegram.textChunkLimit` ဖြင့် ကန့်သတ်ထားသည်။
    
    ဤအင်္ဂါရပ်သည် direct chat များနှင့် group/topic များတွင် အလုပ်လုပ်သည်။
    
    Text-only reply များအတွက် OpenClaw သည် preview မက်ဆေ့ချ်တစ်ခုတည်းကို ဆက်လက်အသုံးပြုပြီး နောက်ဆုံးတွင် ထိုမက်ဆေ့ချ်ကိုပင် ပြင်ဆင်မည် (ဒုတိယ မက်ဆေ့ချ် မပို့ပါ)။
    
    Complex reply များ (ဥပမာ media payload များ) အတွက် OpenClaw သည် ပုံမှန် final delivery သို့ ပြန်လည်သုံးပြီး ထို့နောက် preview မက်ဆေ့ချ်ကို ဖယ်ရှားမည်။
    
    `streamMode` သည် block streaming နှင့် မတူညီပါ။ Telegram အတွက် block streaming ကို အထူးသတ်မှတ်ဖွင့်ထားပါက double-streaming မဖြစ်စေရန် OpenClaw သည် preview stream ကို ကျော်လွှားမည်။
    
    Telegram-only reasoning stream -
    
    - `/reasoning stream` သည် generate လုပ်နေစဉ် reasoning ကို live preview သို့ ပို့မည်
    - နောက်ဆုံးအဖြေကို reasoning စာသားမပါဘဲ ပို့မည်
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">
    Outbound စာသားများတွင် Telegram `parse_mode: "HTML"` ကို အသုံးပြုသည်။

    ```
    - Markdown ပုံစံ စာသားများကို Telegram-safe HTML သို့ ပြောင်းလဲသည်။
    - Model မှ ထွက်လာသော raw HTML ကို Telegram parse error လျော့နည်းစေရန် escape လုပ်သည်။
    - Telegram မှ parsed HTML ကို ငြင်းပယ်ပါက OpenClaw သည် plain text အဖြစ် ပြန်လည်ကြိုးစားမည်။
    
    Link preview များကို မူလအနေဖြင့် ဖွင့်ထားပြီး `channels.telegram.linkPreview: false` ဖြင့် ပိတ်နိုင်သည်။
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">
    Telegram command menu registration ကို startup အချိန်တွင် `setMyCommands` ဖြင့် ကိုင်တွယ်သည်။

    ```
    Native command default များ -
    
    - `commands.native: "auto"` သည် Telegram အတွက် native command များကို ဖွင့်ပေးသည်
    
    Custom command menu entries များ ထည့်ရန် -
    ```

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

    ```
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

  
</Accordion>

  <Accordion title="Inline buttons">
    Inline keyboard scope ကို သတ်မှတ်ရန် -

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

    ```
    Account တစ်ခုချင်းအလိုက် override -
    ```

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

    ```
    Scope များ -
    
    - `off`
    - `dm`
    - `group`
    - `all`
    - `allowlist` (default)
    
    Legacy `capabilities: ["inlineButtons"]` သည် `inlineButtons: "all"` သို့ map လုပ်သည်။
    
    Message action ဥပမာ -
    ```

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

    ```
    Callback click များကို agent သို့ စာသားအဖြစ် ပေးပို့သည် -
    `callback_data: <value>`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram tool actions တွင် အောက်ပါတို့ ပါဝင်သည်:

    ```
    - `sendMessage` (`to`, `content`, optional `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Channel message actions များတွင် အသုံးပြုရ လွယ်ကူသော alias များ (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`) ကို ဖော်ပြထားသည်။
    
    Gating controls:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (default: disabled)
    
    Reaction removal semantics: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">
    Telegram သည် generated output တွင် explicit reply threading tags များကို ပံ့ပိုးသည်:

    ```
    - `[[reply_to_current]]` သည် trigger ဖြစ်သည့် message ကို reply ပြန်သည်
    - `[[reply_to:<id>]]` သည် သတ်မှတ်ထားသော Telegram message ID တစ်ခုကို reply ပြန်သည်
    
    `channels.telegram.replyToMode` သည် ကိုင်တွယ်ပုံကို ထိန်းချုပ်သည်:
    
    - `off` (default)
    - `first`
    - `all`
    
    မှတ်ချက် - `off` သည် implicit reply threading ကို ပိတ်ထားသည်။ Explicit `[[reply_to_*]]` tags များကိုတော့ ဆက်လက် လိုက်နာဆောင်ရွက်မည်ဖြစ်သည်။
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">
    Forum supergroups:

    ```
    - topic session keys တွင် `:topic:<threadId>` ကို ပေါင်းထည့်သည်
    - replies နှင့် typing များကို topic thread သို့ ပို့ဆောင်သည်
    - topic config path:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    General topic (`threadId=1`) အတွက် အထူးကိစ္စ:
    
    - message ပို့ရာတွင် `message_thread_id` မပါဝင်ပါ (Telegram သည် `sendMessage(...thread_id=1)` ကို လက်မခံပါ)
    - typing actions များတွင်တော့ `message_thread_id` ပါဝင်နေဆဲဖြစ်သည်
    
    Topic inheritance: topic entries များသည် override မလုပ်ထားပါက group settings များကို အမွေဆက်ခံမည် (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`)။
    
    Template context တွင် ပါဝင်သည်:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM thread behavior:
    
    - `message_thread_id` ပါရှိသော private chats များသည် DM routing ကို ဆက်လက်ထိန်းသိမ်းထားသော်လည်း thread-aware session keys/reply targets များကို အသုံးပြုသည်။
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Audio messages

    ```
    Telegram တွင် voice notes နှင့် audio files ကို ခွဲခြားထားသည်။
    
    - default: audio file behavior
    - agent reply ထဲတွင် `[[audio_as_voice]]` tag ထည့်ပါက voice-note အဖြစ် ပို့ရန် အတင်းအကျပ် သတ်မှတ်နိုင်သည်
    
    Message action example:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Video messages
    
    Telegram တွင် video files နှင့် video notes ကို ခွဲခြားထားသည်။
    
    Message action example:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Video notes များတွင် captions မထောက်ပံ့ပါ; ပေးထားသော message စာသားကို သီးခြား ပို့မည်ဖြစ်သည်။
    
    ### Stickers
    
    Inbound sticker handling:
    
    - static WEBP: ဒေါင်းလုဒ်လုပ်ပြီး လုပ်ဆောင်သည် (placeholder `<media:sticker>`)
    - animated TGS: ကျော်လွှားထားသည်
    - video WEBM: ကျော်လွှားထားသည်
    
    Sticker context fields:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Sticker cache file:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Stickers များကို (ဖြစ်နိုင်ပါက) တစ်ကြိမ်သာ ဖော်ပြပြီး repeated vision calls များ လျှော့ချရန် cache ထားသည်။
    
    Enable sticker actions:
    ```

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

    ```
    Send sticker action:
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Search cached stickers:
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">
    Telegram reactions များသည် `message_reaction` updates အဖြစ် ရောက်ရှိလာသည် (message payload များနှင့် သီးခြားဖြစ်သည်)။

    ```
    Enable လုပ်ထားပါက OpenClaw သည် အောက်ပါကဲ့သို့သော system events များကို queue ထဲသို့ ထည့်မည်ဖြစ်သည်:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Config:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (default: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (default: `minimal`)
    
    မှတ်ချက်များ:
    
    - `own` ဆိုသည်မှာ bot ပို့ထားသော message များအပေါ် user reactions များသာကို ဆိုလိုသည် (sent-message cache ကို best-effort အဖြစ် အသုံးပြုသည်)။
    - Telegram သည် reaction updates များတွင် thread IDs မပေးပါ။
      - non-forum groups များကို group chat session သို့ ပို့ဆောင်သည်
      - forum groups များကို group general-topic session (`:topic:1`) သို့ ပို့ဆောင်ပြီး မူလ topic အတိအကျသို့ မပို့ပါ
    
    polling/webhook အတွက် `allowed_updates` တွင် `message_reaction` ကို အလိုအလျောက် ထည့်သွင်းထားသည်။
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` သည် OpenClaw မှ inbound message ကို လုပ်ဆောင်နေစဉ် acknowledgement emoji တစ်ခုကို ပို့သည်။

    ```
    Resolution order:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent identity emoji fallback (`agents.list[].identity.emoji`, မရှိပါက "👀")
    
    မှတ်ချက်များ:
    
    - Telegram သည် unicode emoji ကို မျှော်လင့်သည် (ဥပမာ "👀")။
    - channel သို့မဟုတ် account တစ်ခုအတွက် reaction ကို ပိတ်လိုပါက `""` ကို အသုံးပြုပါ။
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Channel config writes များကို default အနေဖြင့် ဖွင့်ထားသည် (`configWrites !== false`)။

    ```
    Telegram-triggered writes များတွင် ပါဝင်သည်:
    
    - `channels.telegram.groups` ကို update လုပ်ရန် group migration events (`migrate_to_chat_id`)
    - `/config set` နှင့် `/config unset` (command enablement လိုအပ်သည်)
    
    Disable:
    ```

```json5
{
  channels: {
    telegram: {
      configWrites: false,
    },
  },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    Default: long polling.

    ```
    Webhook mode:
    
    - `channels.telegram.webhookUrl` ကို သတ်မှတ်ပါ
    - `channels.telegram.webhookSecret` ကို သတ်မှတ်ပါ (webhook URL သတ်မှတ်ထားပါက မဖြစ်မနေလိုအပ်သည်)
    - optional `channels.telegram.webhookPath` (default `/telegram-webhook`)
    - optional `channels.telegram.webhookHost` (default `127.0.0.1`)
    
    Webhook mode အတွက် default local listener သည် `127.0.0.1:8787` တွင် bind လုပ်ထားသည်။
    
    သင်၏ public endpoint မတူပါက reverse proxy တစ်ခုကို ရှေ့တွင်ထားပြီး `webhookUrl` ကို public URL သို့ ညွှန်ပြပါ။
    External ingress လိုအပ်သည့် အခြေအနေတွင် `webhookHost` (ဥပမာ `0.0.0.0`) ကို သတ်မှတ်ပါ။
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    - `channels.telegram.textChunkLimit` ၏ default တန်ဖိုးမှာ 4000 ဖြစ်သည်။
    - `channels.telegram.chunkMode="newline"` သည် စာပိုဒ်ခွဲများ (blank lines) ကို ဦးစားပေးကာ အရှည်အလိုက် ခွဲမတိုင်မီ ဆောင်ရွက်သည်။
    - `channels.telegram.mediaMaxMb` (default 5) သည် inbound Telegram media ဒေါင်းလုဒ်/လုပ်ဆောင်မှု အရွယ်အစားကို ကန့်သတ်သည်။
    - `channels.telegram.timeoutSeconds` သည် Telegram API client timeout ကို override လုပ်သည် (မသတ်မှတ်ထားပါက grammY default ကို အသုံးပြုသည်)။
    - group context history သည် `channels.telegram.historyLimit` သို့မဟုတ် `messages.groupChat.historyLimit` (default 50) ကို အသုံးပြုသည်; `0` သည် ပိတ်သည်။
    - DM history controls:
      - `channels.telegram.dmHistoryLimit`
      - `channels.telegram.dms["<user_id>"].historyLimit`
    - outbound Telegram API retries များကို `channels.telegram.retry` မှတဆင့် configure လုပ်နိုင်သည်။

    ```
    CLI send target သည် numeric chat ID သို့မဟုတ် username ဖြစ်နိုင်သည်:
    ```

```bash
openclaw message send --channel telegram --target 123456789 --message "hi"
openclaw message send --channel telegram --target @name --message "hi"
```

  
</Accordion>
</AccordionGroup>

## Troubleshooting

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - `requireMention=false` ဖြစ်ပါက Telegram privacy mode တွင် full visibility ကို ခွင့်ပြုထားရမည်။
      - BotFather: `/setprivacy` -> Disable
      - ထို့နောက် bot ကို group ထဲမှ ဖယ်ရှားပြီး ပြန်လည် ထည့်သွင်းပါ
    - `openclaw channels status` သည် mention မပါသော group messages များကို config မှ မျှော်လင့်ထားပါက သတိပေးမည်ဖြစ်သည်။
    - `openclaw channels status --probe` သည် သတ်မှတ်ထားသော numeric group IDs များကို စစ်ဆေးနိုင်သည်; wildcard `"*"` ကို membership-probe မလုပ်နိုင်ပါ။
    - အမြန် session စမ်းသပ်ရန်: `/activation always`။
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - `channels.telegram.groups` ရှိပါက group ကို စာရင်းထဲတွင် ထည့်ထားရမည် (သို့မဟုတ် `"*"` ပါဝင်ရမည်)
    - group ထဲတွင် bot ပါဝင်မှုကို စစ်ဆေးပါ
    - skip ဖြစ်ရသည့် အကြောင်းရင်းများကို ကြည့်ရန် logs ကို ပြန်လည်သုံးသပ်ပါ: `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - သင့် sender identity ကို authorize လုပ်ပါ (pairing နှင့်/သို့မဟုတ် numeric `allowFrom`)
    - group policy သည် `open` ဖြစ်နေသော်လည်း command authorization သည် ဆက်လက် အကျိုးသက်ရောက်မည်ဖြစ်သည်
    - `setMyCommands failed` သည် များသောအားဖြင့် `api.telegram.org` သို့ DNS/HTTPS ချိတ်ဆက်နိုင်မှု ပြဿနာများကို ပြသသည်
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + custom fetch/proxy သည် AbortSignal types မကိုက်ညီပါက ချက်ချင်း abort ဖြစ်နိုင်သည်။
    - အချို့ host များတွင် `api.telegram.org` ကို IPv6 သို့ ပထမဦးစွာ resolve လုပ်သည်; IPv6 egress ပျက်ကွက်ပါက Telegram API failures များ အချိန်လိုက် ဖြစ်ပေါ်နိုင်သည်။
    - DNS အဖြေများကို စစ်ဆေးပါ:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Telegram သည် tags ဖြင့် threaded replies ကို ရွေးချယ်အသုံးပြုနိုင်သည်-

## Telegram config reference pointers

`channels.telegram.replyToMode` ဖြင့် ထိန်းချုပ်သည်-

- `first` (default), `all`, `off`။

- `channels.telegram.botToken`: bot token (BotFather)။

- `channels.telegram.tokenFile`: token ကို file path မှ ဖတ်ရန်။

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (default: pairing)။

- `channels.telegram.allowFrom`: DM allowlist (numeric Telegram user IDs). 41. `open` သည် `"*"` ကို လိုအပ်သည်။ `openclaw doctor --fix` သည် legacy `@username` entries များကို IDs များအဖြစ် ဖြေရှင်းပေးနိုင်သည်။

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (default: allowlist)။

- `channels.telegram.groupAllowFrom`: အုပ်စုမှ ပေးပို့သူ allowlist (ဂဏန်း Telegram user ID များ)। `openclaw doctor --fix` သည် အဟောင်း `@username` အချက်အလက်များကို ID များသို့ ပြောင်းလဲဖြေရှင်းနိုင်သည်။

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

- `channels.telegram.replyToMode`: `off | first | all` (မူလသတ်မှတ်ချက်: `off`)။

- `channels.telegram.textChunkLimit`: outbound chunk size (chars)။

- `channels.telegram.chunkMode`: `length` (default) သို့မဟုတ် blank lines (paragraph boundaries) အလိုက် ခွဲရန် `newline`။

- `channels.telegram.linkPreview`: outbound messages များအတွက် link previews ကို ဖွင့်/ပိတ် (default: true)။

- `channels.telegram.streamMode`: `off | partial | block` (live stream အစမ်းကြည့်ရှုမှု)။

- `channels.telegram.mediaMaxMb`: inbound/outbound media cap (MB)။

- `channels.telegram.retry`: outbound Telegram API calls အတွက် retry policy (attempts, minDelayMs, maxDelayMs, jitter)။

- `channels.telegram.network.autoSelectFamily`: override Node autoSelectFamily (true=enable, false=disable). Defaults to disabled on Node 22 to avoid Happy Eyeballs timeouts.

- `channels.telegram.proxy`: Bot API calls အတွက် proxy URL (SOCKS/HTTP)။

- `channels.telegram.webhookUrl`: webhook mode ကို ဖွင့် ( `channels.telegram.webhookSecret` လိုအပ်သည်)။

- `channels.telegram.webhookSecret`: webhook secret (webhookUrl သတ်မှတ်ထားပါက လိုအပ်)။

- `channels.telegram.webhookPath`: local webhook path (default `/telegram-webhook`)။

- `channels.telegram.webhookHost`: local webhook bind host (မူလသတ်မှတ်ချက် `127.0.0.1`)။

- `channels.telegram.actions.reactions`: Telegram tool reactions ကို gate လုပ်ရန်။

- `channels.telegram.actions.sendMessage`: Telegram tool message sends ကို gate လုပ်ရန်။

- `channels.telegram.actions.deleteMessage`: Telegram tool message deletes ကို gate လုပ်ရန်။

- `channels.telegram.actions.sticker`: Telegram sticker actions — send and search (default: false)။

- `channels.telegram.reactionNotifications`: `off | own | all` — system events ဖြစ်စေမည့် reactions များကို ထိန်းချုပ် (default: မသတ်မှတ်ပါက `own`)။

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — agent ၏ reaction စွမ်းဆောင်ရည်ကို ထိန်းချုပ် (default: မသတ်မှတ်ပါက `minimal`)။

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram အတွက် အရေးပါသော high-signal fields များ:

- Telegram သည် **အသံမှတ်စုများ** (ဝိုင်းပုံ bubble) နှင့် **အသံဖိုင်များ** (metadata ကတ်) ကို ခွဲခြားသတ်မှတ်ထားသည်။
- OpenClaw သည် နောက်ပြန်လိုက်ဖက်မှုအတွက် မူလအားဖြင့် audio files ကို အသုံးပြုသည်။
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- streaming: `streamMode` (အစမ်းကြည့်ရှုမှု), `draftChunk`, `blockStreaming`
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- reactions: `reactionNotifications`, `reactionLevel`
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## ဆက်စပ်အကြောင်းအရာများ

- `[[audio_as_voice]]` — file အစား voice note အဖြစ် audio ကို ပို့သည်။
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
