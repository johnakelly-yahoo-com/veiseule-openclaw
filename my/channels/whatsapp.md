---
summary: "WhatsApp (ဝဘ်ချန်နယ်) ပေါင်းစည်းမှု៖ လော့ဂ်အင်၊ inbox၊ ပြန်ကြားချက်များ၊ မီဒီယာနှင့် လုပ်ဆောင်မှုများ"
read_when:
  - WhatsApp/ဝဘ် ချန်နယ်၏ အပြုအမူ သို့မဟုတ် inbox လမ်းကြောင်းခွဲခြားမှုအပေါ် အလုပ်လုပ်နေချိန်
title: "WhatsApp"
---

# WhatsApp (ဝဘ် ချန်နယ်)

Status: WhatsApp Web (Baileys) မှတဆင့် production-ready ဖြစ်သည်။ Gateway သည် ချိတ်ဆက်ထားသော session(များ) ကို စီမံခန့်ခွဲသည်။

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">    မသိသေးသော ပေးပို့သူများအတွက် မူလ DM မူဝါဒမှာ pairing ဖြစ်သည်။
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">    Channel အမျိုးမျိုးအတွက် diagnostics နှင့် ပြုပြင်ရေး playbook များ။
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">    Channel config ပုံစံများနှင့် ဥပမာများ အပြည့်အစုံ။
</Card>
</CardGroup>

## အမြန် စတင်တပ်ဆင်ခြင်း

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
    အကောင့်တစ်ခု သီးသန့်အတွက်:
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
    Pairing တောင်းဆိုမှုများသည် ၁ နာရီအတွင်း သက်တမ်းကုန်ဆုံးမည်ဖြစ်သည်။ Channel တစ်ခုလျှင် pending တောင်းဆိုမှု အများဆုံး ၃ ခုသာ ခွင့်ပြုထားသည်။
    ```

  
</Step>
</Steps>

<Note>
ဖြစ်နိုင်ပါက WhatsApp ကို ဖုန်းနံပါတ် သီးခြားတစ်ခုဖြင့် လည်ပတ်ရန် OpenClaw က အကြံပြုပါသည်။ (Channel metadata နှင့် onboarding flow ကို ထို setup အတွက် အကောင်းဆုံးဖြစ်အောင် optimize လုပ်ထားပါသည်၊ သို့သော် personal-number setup များကိုလည်း ထောက်ပံ့ထားပါသည်။)
</Note>

## Deployment ပုံစံများ

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    ဤသည်မှာ အလုပ်လုပ်ဆောင်ရန်အတွက် အရှင်းလင်းဆုံး operational mode ဖြစ်သည်:


    ````
    - OpenClaw အတွက် သီးသန့် WhatsApp identity
    - DM allowlist နှင့် routing boundary များကို ပိုမိုရှင်းလင်းစေသည်
    - ကိုယ်တိုင် chat လုပ်မိခြင်းကြောင့် ရှုပ်ထွေးမှု ဖြစ်နိုင်ခြေ လျော့နည်းစေသည်
    
    Minimal policy pattern:
    
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
    Onboarding သည် personal-number mode ကို ထောက်ပံ့ပြီး self-chat အတွက် သင့်လျော်သော baseline ကို ရေးသားပေးပါသည်:


    ```
    - `dmPolicy: "allowlist"`
    - `allowFrom` တွင် သင့်ကိုယ်ပိုင် ဖုန်းနံပါတ် ပါဝင်သည်
    - `selfChatMode: true`
    
    Runtime အတွင်းတွင် self-chat ကာကွယ်ရေးများသည် ချိတ်ဆက်ထားသော self number နှင့် `allowFrom` ကို အခြေခံ၍ လုပ်ဆောင်ပါသည်။
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">
    လက်ရှိ OpenClaw channel architecture တွင် messaging platform channel သည် WhatsApp Web အခြေခံ (`Baileys`) ဖြစ်သည်။

    ```
    Built-in chat-channel registry တွင် သီးခြား Twilio WhatsApp messaging channel မရှိပါ။
    ```

  
</Accordion>
</AccordionGroup>

## Twilio ကို ဘာကြောင့် မသုံးသလဲ?

- OpenClaw ၏ အစောပိုင်း build များတွင် Twilio ၏ WhatsApp Business integration ကို ပံ့ပိုးခဲ့သည်။
- WhatsApp Business နံပါတ်များသည် ကိုယ်ပိုင် assistant အတွက် မသင့်လျော်ပါ။
- Meta သည် ၂၄ နာရီ ပြန်ကြားချိန် ကန့်သတ်ချက်ကို အတင်းအကျပ် သတ်မှတ်ထားသည် — နောက်ဆုံး ၂၄ နာရီအတွင်း မပြန်ကြားခဲ့ပါက business နံပါတ်မှ မက်ဆေ့ချ်အသစ် စတင်ပို့လို့ မရပါ။
- အသုံးပြုမှု များပြားခြင်း သို့မဟုတ် “chatty” ဖြစ်ခြင်းသည် ပြင်းထန်သော blocking ကို ဖြစ်စေတတ်သည်၊ business အကောင့်များကို ကိုယ်ပိုင် assistant မက်ဆေ့ချ်များ အများကြီး ပို့ရန် မရည်ရွယ်ထားသောကြောင့် ဖြစ်သည်။
- အကျိုးရလဒ်အဖြစ် ပို့ဆောင်မှု မယုံကြည်ရပြီး မကြာခဏ ပိတ်ပင်ခံရသဖြင့် ပံ့ပိုးမှုကို ဖယ်ရှားခဲ့သည်။

## Login + credentials

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` သည် direct chat ဝင်ရောက်ခွင့်ကို ထိန်းချုပ်ပါသည်:

    ```
    - `pairing` (မူလတန်ဖိုး)
    - `allowlist`
    - `open` (`allowFrom` တွင် `"*"` ပါဝင်ရမည်)
    - `disabled`
    
    `allowFrom` သည် E.164 ပုံစံ ဖုန်းနံပါတ်များကို လက်ခံသည် (အတွင်းပိုင်းတွင် normalize လုပ်ပါသည်)။
    
    Multi-account override: `channels.whatsapp.accounts.<id>.dmPolicy` (နှင့် `allowFrom`) သည် ထို account အတွက် channel-level default ထက် ဦးစားပေးပါသည်။
    
    Runtime behavior အသေးစိတ်:
    
    - pairings များကို channel allow-store တွင် သိမ်းဆည်းပြီး configure လုပ်ထားသော `allowFrom` နှင့် ပေါင်းစည်းပါသည်
    - allowlist မသတ်မှတ်ထားပါက ချိတ်ဆက်ထားသော self number ကို မူလအားဖြင့် ခွင့်ပြုပါသည်
    - outbound `fromMe` DM များကို auto-pair မလုပ်ပါ
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Group ဝင်ရောက်ခွင့်တွင် layer နှစ်ခုရှိသည်:

    ```
    1. **Group membership allowlist** (`channels.whatsapp.groups`)
       - `groups` မပါရှိပါက group အားလုံး အကျုံးဝင်သည်
       - `groups` ပါရှိပါက group allowlist အဖြစ် လုပ်ဆောင်သည် (`"*"` ခွင့်ပြု)
    
    2. **Group sender policy** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: sender allowlist ကို ကျော်လွှားသည်
       - `allowlist`: sender သည် `groupAllowFrom` (သို့မဟုတ် `*`) နှင့် ကိုက်ညီရမည်
       - `disabled`: group မှ inbound အားလုံးကို ပိတ်ပင်သည်
    
    Sender allowlist fallback:
    
    - `groupAllowFrom` မသတ်မှတ်ထားပါက runtime သည် ရနိုင်ပါက `allowFrom` သို့ fallback လုပ်ပါသည်
    
    မှတ်ချက် - `channels.whatsapp` block လုံးဝ မရှိပါက runtime group-policy fallback သည် အကျိုးသက်ရောက်မှုအရ `open` ဖြစ်သည်။
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    မူလအားဖြင့် group reply များတွင် mention လိုအပ်သည်။

    ```
    Mention detection တွင် ပါဝင်သည်:
    
    - bot identity ကို explicit WhatsApp mention ပြုလုပ်ခြင်း
    - configure လုပ်ထားသော mention regex pattern များ (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - bot ကို reply ပြန်ခြင်းအားဖြင့် ဖြစ်ပေါ်သော implicit detection (reply sender သည် bot identity နှင့် ကိုက်ညီသည်)
    
    Session-level activation command:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` သည် session state ကို update လုပ်သည် (global config မဟုတ်ပါ)။ ၎င်းကို owner သာ အသုံးပြုနိုင်သည်။
    ```

  
</Tab>
</Tabs>

## Personal-number နှင့် self-chat အပြုအမူ

ချိတ်ဆက်ထားသော self number ကို `allowFrom` တွင်ပါ ထည့်ထားပါက WhatsApp self-chat ကာကွယ်ရေးများ အလုပ်လုပ်ပါသည်:

- self-chat အလှည့်များအတွက် read receipt မပို့ပါ
- ကိုယ်တိုင်ကို mention လုပ်သည့် auto-trigger အပြုအမူကို လျစ်လျူရှုပါသည်
- `messages.responsePrefix` မသတ်မှတ်ထားပါက self-chat reply များသည် မူလအားဖြင့် `[{identity.name}]` သို့မဟုတ် `[openclaw]` ကို အသုံးပြုပါသည်

## Message normalization နှင့် context

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    ဝင်ရောက်လာသော WhatsApp message များကို shared inbound envelope အတွင်း ထည့်သွင်းထားပါသည်။

    ````
    quoted reply ရှိပါက context ကို အောက်ပါပုံစံဖြင့် ပေါင်းထည့်ပါသည်:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Reply metadata field များကိုလည်း ရရှိနိုင်ပါက ဖြည့်သွင်းပါသည် (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164)。
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">
    Media သာ ပါဝင်သော inbound message များကို အောက်ပါ placeholder များဖြင့် normalize လုပ်ပါသည်:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Location နှင့် contact payload များကို routing မလုပ်မီ စာသား context အဖြစ် normalize လုပ်ပါသည်။
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Group များအတွက် bot ကို trigger မလုပ်ရသေးသော message များကို buffer လုပ်ပြီး နောက်ဆုံး trigger ဖြစ်သည့်အခါ context အဖြစ် ထည့်သွင်းနိုင်ပါသည်။

    ```
    - မူလ limit: `50`
    - config: `channels.whatsapp.historyLimit`
    - fallback: `messages.groupChat.historyLimit`
    - `0` သည် ပိတ်သိမ်းခြင်း ဖြစ်သည်
    
    Injection marker များ:
    
    - `[Chat messages since your last reply - for context]`
    - `[Current message - respond to this]`
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    လက်ခံထားသော inbound WhatsApp message များအတွက် မူလအားဖြင့် read receipt ကို ဖွင့်ထားပါသည်။

    ````
    
    Global အနေဖြင့် ပိတ်ရန်:
    
    ```json5
    {
      channels: {
        whatsapp: {
          sendReadReceipts: false,
        },
      },
    }
    ```
    
    Per-account override:
    
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
    
    Global အနေဖြင့် ဖွင့်ထားသော်လည်း self-chat အလှည့်များတွင် read receipt မပို့ပါ။
    ````

  
</Accordion>
</AccordionGroup>

## Reply delivery (threading)

<AccordionGroup>
  <Accordion title="Text chunking">
    - မူလ chunk limit: `channels.whatsapp.textChunkLimit = 4000`
    - `channels.whatsapp.chunkMode = "length" | "newline"`
    - `newline` mode သည် paragraph boundary (blank line) များကို ဦးစားပေးပြီး၊ မရရှိပါက length-safe chunking သို့ ပြန်လည်အသုံးပြုပါသည်
  
</Accordion>

  <Accordion title="Outbound media behavior">
    - image, video, audio (PTT voice-note) နှင့် document payload များကို ထောက်ပံ့သည်
    - `audio/ogg` ကို voice-note compatibility အတွက် `audio/ogg; codecs=opus` အဖြစ် ပြန်ရေးသားသည်
    - video ပို့ရာတွင် `gifPlayback: true` ဖြင့် animated GIF playback ကို ထောက်ပံ့သည်
    - multi-media reply payload ပို့ရာတွင် caption ကို ပထမ media item တွင် ထည့်သွင်းသည်
    - media source သည် HTTP(S), `file://`, သို့မဟုတ် local path ဖြစ်နိုင်သည်
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - inbound media သိမ်းဆည်းနိုင်သော အများဆုံး: `channels.whatsapp.mediaMaxMb` (မူလ `50`)
    - auto-reply အတွက် outbound media အများဆုံး: `agents.defaults.mediaMaxMb` (မူလ `5MB`)
    - image များကို limit နှင့် ကိုက်ညီရန် auto-optimize (resize/quality sweep) လုပ်ပါသည်
    - media ပို့ရာတွင် ပျက်ကွက်ပါက ပထမ item fallback အဖြစ် စာသားသတိပေးချက် ပို့ပြီး တုံ့ပြန်မှုကို မပျောက်ကွယ်စေပါ
  
</Accordion>
</AccordionGroup>

## Acknowledgment reaction များ

WhatsApp သည် inbound လက်ခံသည့်အချိန်တွင် `channels.whatsapp.ackReaction` မှတဆင့် ချက်ချင်း ack reaction ပေးနိုင်ပါသည်။

```json5
{
  channels: {
    whatsapp: {
      ackReaction: {
        emoji: "👀",
        direct: true,
        group: "mentions", // always | mentions | never
      },
    },
  },
}
```

Behavior မှတ်ချက်များ:

- inbound ကို လက်ခံပြီး ချက်ချင်း (reply မပေးမီ) ပို့သည်
- failure ဖြစ်ပါက log မှတ်တမ်းတင်သော်လည်း ပုံမှန် reply ပို့ခြင်းကို မတားဆီးပါ
- group mode `mentions` သည် mention ဖြင့် trigger လုပ်ထားသော turn များတွင်သာ တုံ့ပြန်သည်။ group activation `always` သည် ဤစစ်ဆေးမှုကို ကျော်လွှားရန် အသုံးပြုနိုင်သည်။
- WhatsApp သည် `channels.whatsapp.ackReaction` ကို အသုံးပြုသည် (`messages.ackReaction` legacy ကို ဤနေရာတွင် မအသုံးပြုပါ)

## Multi-account နှင့် credentials

<AccordionGroup>
  <Accordion title="Account selection and defaults">
    - account id များကို `channels.whatsapp.accounts` မှ ရယူသည်
    - default account ရွေးချယ်မှု: `default` ရှိပါက ၎င်းကို အသုံးပြုမည်၊ မရှိပါက (sorted လုပ်ထားသော) ပထမဆုံး configured account id ကို အသုံးပြုမည်
    - lookup အတွက် account id များကို အတွင်းပိုင်းတွင် normalize လုပ်ထားသည်
  
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - လက်ရှိ auth path: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - backup ဖိုင်: `creds.json.bak`
    - `~/.openclaw/credentials/` ထဲရှိ legacy default auth ကိုလည်း default-account flow များအတွက် အသိအမှတ်ပြု/ပြောင်းရွှေ့ပေးထားသည်
  
</Accordion>

  <Accordion title="Logout behavior">
    `openclaw channels logout --channel whatsapp [--account <id>]` သည် ထို account အတွက် WhatsApp auth state ကို ရှင်းလင်းပစ်သည်။

    ```
    Legacy auth directories များတွင် `oauth.json` ကို ထိန်းသိမ်းထားပြီး Baileys auth ဖိုင်များကို ဖယ်ရှားသည်။
    ```

  
</Accordion>
</AccordionGroup>

## Tools, actions နှင့် config writes

- Agent tool support တွင် WhatsApp reaction action (`react`) ပါဝင်သည်။
- Action gates:
  - `channels.whatsapp.actions.reactions`
  - `channels.whatsapp.actions.polls`
- Channel မှ စတင်သော config writes ကို မူလအခြေအနေအနေဖြင့် ဖွင့်ထားသည် (`channels.whatsapp.configWrites=false` ဖြင့် ပိတ်နိုင်သည်)။

## ပြဿနာဖြေရှင်းခြင်း

<AccordionGroup>
  <Accordion title="Not linked (QR required)">
    လက္ခဏာ: channel status တွင် not linked ဟု ပြသနေသည်။

    ````
    ဖြေရှင်းရန်:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    လက္ခဏာ: linked account ဖြစ်သော်လည်း အကြိမ်ကြိမ် disconnect သို့မဟုတ် reconnect ကြိုးစားမှုများ ဖြစ်ပေါ်နေသည်။

    ````
    ဖြေရှင်းရန်:
    
    ```bash
    openclaw doctor
    openclaw logs --follow
    ```
    
    လိုအပ်ပါက `channels login` ဖြင့် ပြန်လည် link လုပ်ပါ။
    ````

  
</Accordion>

  <Accordion title="No active listener when sending">
    ပစ်မှတ် account အတွက် active gateway listener မရှိပါက outbound sends များသည် ချက်ချင်း fail ဖြစ်မည်။

    ```
    gateway ကို လည်ပတ်နေကြောင်းနှင့် account ကို link လုပ်ထားကြောင်း သေချာစစ်ဆေးပါ။
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    အောက်ပါအစဉ်အတိုင်း စစ်ဆေးပါ:

    ```
    - `groupPolicy`
    - `groupAllowFrom` / `allowFrom`
    - `groups` allowlist entries
    - mention gating (`requireMention` + mention patterns)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway runtime သည် Node ကို အသုံးပြုသင့်သည်။ Bun ကို တည်ငြိမ်သော WhatsApp/Telegram gateway လည်ပတ်မှုအတွက် မကိုက်ညီကြောင်း သတ်မှတ်ထားသည်။
  
</Accordion>
</AccordionGroup>

## Troubleshooting (အမြန်)

**Not linked / QR login လိုအပ်**

- လက္ခဏာ: `channels status` တွင် `linked: false` ပြသခြင်း သို့မဟုတ် “Not linked” သတိပေးချက်။

**Linked ဖြစ်သော်လည်း ချိတ်ဆက်မရ / reconnect loop**

- လက္ခဏာ: `channels status` တွင် `running, disconnected` ပြသခြင်း သို့မဟုတ် “Linked but disconnected” သတိပေးချက်။
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- multi-account: `accounts.<id>.enabled`, `accounts.<id>.authDir`, account-level overrides
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- session behavior: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>.historyLimit`

## ဆက်စပ်သောအကြောင်းအရာများ

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

