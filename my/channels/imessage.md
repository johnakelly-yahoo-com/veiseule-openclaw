---
summary: "Legacy iMessage support via imsg (JSON-RPC over stdio). New setups should use BlueBubbles."
read_when:
  - iMessage ထောက်ပံ့မှု တပ်ဆင်ခြင်း
  - iMessage ပို့ခြင်း/လက်ခံခြင်း ကို Debug လုပ်ခြင်း
title: "iMessage"
---

# iMessage (အဟောင်း: imsg)

<Warning>
အသစ်သော iMessage deployment များအတွက် <a href="/channels/bluebubbles">BlueBubbles</a> ကို အသုံးပြုပါ။

`imsg` integration သည် legacy ဖြစ်ပြီး နောင်လာမည့် release တစ်ခုတွင် ဖယ်ရှားခံရနိုင်သည်။ 
</Warning>

Status: legacy external CLI integration. Gateway သည် `imsg rpc` ကို spawn လုပ်ပြီး stdio ပေါ်တွင် JSON-RPC ဖြင့် ဆက်သွယ်သည် (သီးခြား daemon/port မလိုအပ်ပါ).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    အသစ်သော setup များအတွက် အကြံပြုထားသော iMessage လမ်းကြောင်း။
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DMs များသည် ပုံမှန်အားဖြင့် pairing mode ဖြင့် လုပ်ဆောင်သည်။
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    iMessage field အပြည့်အစုံ ရည်ညွှန်းချက်။
  
</Card>
</CardGroup>

## အမြန် စတင်သတ်မှတ်ခြင်း

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
brew install steipete/tap/imsg
imsg rpc --help
```

        
</Step>
      
        <Step title="Configure OpenClaw">

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
            Pairing request များသည် ၁ နာရီအတွင်း သက်တမ်းကုန်ဆုံးမည်။
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">
    OpenClaw သည် stdio-compatible `cliPath` တစ်ခုသာ လိုအပ်သောကြောင့် `cliPath` ကို remote Mac တစ်ခုသို့ SSH ချိတ်ဆက်ပြီး `imsg` ကို run လုပ်ပေးသော wrapper script တစ်ခုသို့ ညွှန်ပြနိုင်သည်။

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    attachment များကို ဖွင့်ထားပါက အကြံပြုထားသော config:
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "user@gateway-host", // used for SCP attachment fetches
      includeAttachments: true,
    },
  },
}
```

    ```
    `remoteHost` ကို သတ်မှတ်မထားပါက OpenClaw သည် SSH wrapper script ကို parse လုပ်၍ အလိုအလျောက် စစ်ဆေးဖော်ထုတ်ရန် ကြိုးစားသည်။
    ```

  
</Tab>
</Tabs>

## လိုအပ်ချက်များနှင့် ခွင့်ပြုချက်များ (macOS)

- `imsg` ကို လုပ်ဆောင်နေသော Mac တွင် Messages အကောင့်ဝင်ထားရပါမည်။
- OpenClaw/`imsg` ကို လုပ်ဆောင်နေသော process context အတွက် Full Disk Access လိုအပ်ပါသည် (Messages DB ကို အသုံးပြုရန်)။
- Messages.app မှတဆင့် မက်ဆေ့ချ်များ ပို့ရန် Automation ခွင့်ပြုချက် လိုအပ်ပါသည်။

<Tip>
ခွင့်ပြုချက်များကို process context တစ်ခုချင်းအလိုက် ခွင့်ပြုရပါသည်။ gateway ကို headless (LaunchAgent/SSH) ဖြင့် လုပ်ဆောင်ပါက prompt များ ပေါ်လာစေရန် တူညီသော context တွင် တစ်ကြိမ်သာ interactive command ကို လုပ်ဆောင်ပါ။:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## ဝင်ရောက်ခွင့် ထိန်းချုပ်မှုနှင့် လမ်းကြောင်းသတ်မှတ်ခြင်း

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` သည် direct messages များကို ထိန်းချုပ်သည်:

    ```
    - `pairing` (မူလသတ်မှတ်ချက်)
    - `allowlist`
    - `open` (`allowFrom` တွင် `"*"` ထည့်သွင်းထားရန် လိုအပ်သည်)
    - `disabled`
    
    Allowlist field: `channels.imessage.allowFrom`.
    
    Allowlist entries များသည် handle များ သို့မဟုတ် chat target များ (`chat_id:*`, `chat_guid:*`, `chat_identifier:*`) ဖြစ်နိုင်သည်။
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` သည် group ကို ကိုင်တွယ်ပုံကို ထိန်းချုပ်သည်:

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
    - DMs များသည် direct routing ကို အသုံးပြုသည်၊ group များသည် group routing ကို အသုံးပြုသည်။
    - မူလသတ်မှတ်ချက် `session.dmScope=main` ဖြစ်ပါက iMessage DMs များကို agent ၏ main session ထဲတွင် ပေါင်းစည်းထားသည်။
    - Group session များသည် သီးခြားထားရှိသည် (`agent:<agentId> :imessage:group:<chat_id>`)။
    - ပြန်စာများကို မူလ channel/target metadata အပေါ် အခြေခံ၍ iMessage သို့ ပြန်လည် လမ်းကြောင်းသတ်မှတ်ပို့သည်။

    ```
    Group အလားအလာရှိသော thread အပြုအမူ:
    
    အချို့သော ပါဝင်သူများစွာပါဝင်သည့် iMessage thread များသည် `is_group=false` ဖြင့် ရောက်ရှိနိုင်သည်။
    ထို `chat_id` ကို `channels.imessage.groups` အောက်တွင် အတိအလင်း သတ်မှတ်ထားပါက OpenClaw သည် ၎င်းကို group traffic အဖြစ် သတ်မှတ်ကိုင်တွယ်သည် (group gating + group session isolation)။
    ```

  
</Tab>
</Tabs>

## Deployment ပုံစံများ

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">
    Bot traffic ကို သင်၏ ကိုယ်ပိုင် Messages profile မှ ခွဲထုတ်ထားရန် သီးသန့် Apple ID နှင့် macOS user ကို အသုံးပြုပါ။

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
    ပုံမှန် topology:

    ```
    - gateway ကို Linux/VM ပေါ်တွင် လုပ်ဆောင်သည်
    - iMessage + `imsg` ကို သင့်၏ tailnet အတွင်းရှိ Mac ပေါ်တွင် လုပ်ဆောင်သည်
    - `cliPath` wrapper သည် SSH ကို အသုံးပြု၍ `imsg` ကို လုပ်ဆောင်သည်
    - `remoteHost` သည် SCP ဖြင့် attachment များကို ရယူနိုင်စေသည်
    
    Example:
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
    SSH နှင့် SCP နှစ်ခုစလုံးကို non-interactive ဖြစ်စေရန် SSH keys များကို အသုံးပြုပါ။
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">
    iMessage သည် `channels.imessage.accounts` အောက်တွင် account တစ်ခုချင်းအလိုက် config ကို ထောက်ပံ့သည်။

    ```
    Account တစ်ခုချင်းစီသည် `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` နှင့် history settings စသည့် fields များကို override လုပ်နိုင်သည်။
    ```

  
</Accordion>
</AccordionGroup>

## မီဒီယာ၊ အပိုင်းခွဲခြင်း (chunking) နှင့် ပို့ဆောင်ရေး target များ

<AccordionGroup>
  <Accordion title="Attachments and media">
    - ဝင်လာသော attachment များကို လက်ခံခြင်းသည် optional ဖြစ်သည်: `channels.imessage.includeAttachments`
    - `remoteHost` သတ်မှတ်ထားပါက remote attachment path များကို SCP ဖြင့် ရယူနိုင်သည်
    - ထွက်ပေါ်မည့် မီဒီယာ အရွယ်အစားကို `channels.imessage.mediaMaxMb` (မူလ 16 MB) ဖြင့် သတ်မှတ်သည်
  
</Accordion>

  <Accordion title="Outbound chunking">
    - စာသား chunk ကန့်သတ်ချက်: `channels.imessage.textChunkLimit` (မူလ 4000)
    - chunk mode: `channels.imessage.chunkMode`
      - `length` (မူလ)
      - `newline` (paragraph ကို ဦးစားပေး၍ ခွဲခြားခြင်း)
  
</Accordion>

  <Accordion title="Addressing formats">
    ဦးစားပေး အသုံးပြုသင့်သော explicit target များ:

    ```
    - `chat_id:123` (တည်ငြိမ်သော routing အတွက် အကြံပြုသည်)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle target များကိုလည်း ထောက်ပံ့သည်:
    
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

iMessage သည် မူလအားဖြင့် channel မှ စတင်သော config write များကို ခွင့်ပြုထားသည် (`commands.config: true` ဖြစ်သောအခါ `/config set|unset` အတွက်)။

ပိတ်ရန်:

```json5
{
  channels: {
    imessage: {
      configWrites: false,
    },
  },
}
```

## ပြဿနာဖြေရှင်းခြင်း

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Binary နှင့် RPC ထောက်ပံ့မှုကို စစ်ဆေးပါ:

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
    စစ်ဆေးရန်:

    ```
    - `channels.imessage.dmPolicy`
    - `channels.imessage.allowFrom`
    - pairing အတည်ပြုချက်များ (`openclaw pairing list imessage`)
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">
    စစ်ဆေးပါ:

    ```
    - `channels.imessage.groupPolicy`
    - `channels.imessage.groupAllowFrom`
    - `channels.imessage.groups` allowlist လုပ်ဆောင်ပုံ
    - mention pattern configuration (`agents.list[].groupChat.mentionPatterns`) ကို ဖော်ပြပါ
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">
    စစ်ဆေးပါ:

    ```
    - `channels.imessage.remoteHost`
    - gateway host မှ SSH/SCP key auth
    - Messages လည်ပတ်နေသော Mac ပေါ်ရှိ remote path ကို ဖတ်ရှုနိုင်မှု
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    တူညီသော user/session context အတွင်း interactive GUI terminal ဖြင့် ပြန်လည်လုပ်ဆောင်ပြီး prompt များကို အတည်ပြုပါ:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    OpenClaw/`imsg` ကို လည်ပတ်စေသော process context အတွက် Full Disk Access + Automation ခွင့်ပြုချက်များ ပေးထားကြောင်း အတည်ပြုပါ။
    ```

  
</Accordion>
</AccordionGroup>

## Configuration reference အညွှန်းများ

- `agents.list[].groupChat.mentionPatterns` (သို့မဟုတ် `messages.groupChat.mentionPatterns`)။
- `messages.responsePrefix`။
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

