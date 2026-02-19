---
summary: "Gateway ဝန်ဆောင်မှု၊ အသက်တာလည်ပတ်မှုနှင့် လည်ပတ်ရေးဆိုင်ရာ Runbook"
read_when:
  - Gateway လုပ်ငန်းစဉ်ကို လည်ပတ်နေစဉ် သို့မဟုတ် ပြဿနာရှာဖွေနေစဉ်
title: "Gateway လုပ်ငန်းဆောင်ရွက်မှု လမ်းညွှန်"
---

# Gateway ဝန်ဆောင်မှု Runbook

နောက်ဆုံး ပြင်ဆင်သည့်ရက်စွဲ: 2025-12-09

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    လက္ခဏာအခြေပြု diagnostics ကို တိကျသော command အဆင့်လိုက်နှင့် log signature များဖြင့်။
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    အလုပ်အကိုင်အခြေပြု setup လမ်းညွှန် + အပြည့်အစုံ configuration ကိုးကားချက်။
  
</Card>
</CardGroup>

## ၅ မိနစ်အတွင်း local စတင်ခြင်း

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

ကျန်းမာသော အခြေခံအခြေအနေ: `Runtime: running` နှင့် `RPC probe: ok`။

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway config reload သည် လက်ရှိအသုံးပြုနေသော config file လမ်းကြောင်းကို (profile/state defaults မှ ဖြေရှင်းထားသော သို့မဟုတ် `OPENCLAW_CONFIG_PATH` သတ်မှတ်ထားပါက ၎င်းမှ) စောင့်ကြည့်သည်။
မူလ mode သည် `gateway.reload.mode="hybrid"` ဖြစ်သည်။
</Note>

## Runtime မော်ဒယ်

- Routing၊ control plane နှင့် channel ချိတ်ဆက်မှုများအတွက် အမြဲလုပ်ဆောင်နေသော process တစ်ခု။
- အောက်ပါအရာများအတွက် multiplexed port တစ်ခုတည်း:
  - WebSocket control/RPC
  - HTTP APIs (OpenAI-compatible, Responses, tools invoke)
  - Control UI နှင့် hooks
- မူလ bind mode: `loopback`။
- Auth ကို မူလအနေဖြင့် လိုအပ်သည် (`gateway.auth.token` / `gateway.auth.password` သို့မဟုတ် `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`)။

### Dev profile (`--dev`)

| Setting      | ဖြေရှင်းအစီအစဉ်                                               |
| ------------ | ------------------------------------------------------------- |
| Gateway port | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| Bind mode    | CLI/override → `gateway.bind` → `loopback`                    |

### Hot reload modes

| `gateway.reload.mode`             | အပြုအမူ                                                  |
| --------------------------------- | -------------------------------------------------------- |
| `off`                             | Config reload မရှိပါ                                     |
| `hot`                             | Hot-safe ပြောင်းလဲမှုများကိုသာ အသုံးချသည်                |
| `restart`                         | Reload လိုအပ်သော ပြောင်းလဲမှုများတွင် restart ပြုလုပ်သည် |
| `hybrid` (မူလ) | လုံခြုံပါက ချက်ချင်းအသုံးချပါ၊ လိုအပ်ပါက ပြန်လည်စတင်ပါ   |

## Operator command စုစည်းမှု

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw logs --follow
openclaw doctor
```

## Remote access

အကြိုက်ဆုံးနည်းလမ်း: Tailscale/VPN။
အစားထိုးနည်းလမ်း: SSH tunnel။

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Profile တစ်ခုချင်းစီအတွက် service install။

<Warning>
Gateway auth ကို စီစဉ်ထားပါက SSH tunnel မှတစ်ဆင့် ချိတ်ဆက်သော်လည်း client များသည် auth (`token`/`password`) ကို မဖြစ်မနေ ပေးပို့ရပါမည်။
</Warning>

ဥပမာ။

## ကြီးကြပ်မှုနှင့် service lifecycle

Production ကဲ့သို့ ယုံကြည်စိတ်ချရမှုအတွက် supervised run များကို အသုံးပြုပါ။

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

LaunchAgent label များမှာ `ai.openclaw.gateway` (default) သို့မဟုတ် `ai.openclaw.<profile>` (အမည်ပေးထားသော profile)。 `openclaw doctor` သည် service config drift ကို စစ်ဆေး၍ ပြုပြင်ပေးသည်။

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

Logout ပြုလုပ်ပြီးနောက် ဆက်လက်တည်ရှိစေရန် lingering ကို ဖွင့်ပါ:

```bash
sudo loginctl enable-linger <user>
```

  
</Tab>

  <Tab title="Linux (system service)">

Multi-user/အမြဲဖွင့်ထားသော host များအတွက် system unit ကို အသုံးပြုပါ။

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Host တစ်ခုတည်းပေါ်တွင် Gateway အများအပြား

စနစ်အများစုတွင် **Gateway တစ်ခုတည်း** ကိုသာ လည်ပတ်သင့်သည်။
တင်းကျပ်သော သီးခြားခွဲထားမှု/အရန်စနစ်အတွက်သာ (ဥပမာ rescue profile) များစွာ အသုံးပြုပါ။

Instance တစ်ခုစီအတွက် စစ်ဆေးစာရင်း။

- ထူးခြားသည့် `gateway.port`
- ထူးခြားသည့် `OPENCLAW_CONFIG_PATH`
- ထူးခြားသည့် `OPENCLAW_STATE_DIR`
- ထူးခြားသည့် `agents.defaults.workspace`

ဥပမာ။

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

ကြည့်ရှုရန်: [Multiple gateways](/gateway/multiple-gateways)。

### Gateway ဝန်ဆောင်မှု စီမံခန့်ခွဲမှု (CLI)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

Default များတွင် သီးခြား state/config နှင့် အခြေခံ gateway port `19001` ပါဝင်သည်။

## Protocol အမြန်ကိုးကားချက် (operator အမြင်)

- `gateway status` သည် service ၏ resolved port/config ကို အသုံးပြု၍ Gateway RPC ကို မူလအားဖြင့် probe လုပ်သည် (`--url` ဖြင့် override လုပ်နိုင်သည်)။
- `gateway status --deep` သည် system-level scans (LaunchDaemons/system units) ကို ထည့်သွင်းသည်။
- `gateway status --no-probe` သည် RPC probe ကို ကျော်လွှားသည် (network မရရှိသည့်အခါ အသုံးဝင်သည်)။
- `gateway status --json` သည် scripts အတွက် တည်ငြိမ်သည်။

Bundled mac app။

1. ချက်ချင်း လက်ခံကြောင်း ack (`status:"accepted"`)
2. သန့်ရှင်းစွာ ရပ်တန့်ရန် `openclaw gateway stop` (သို့မဟုတ် `launchctl bootout gui/$UID/bot.molt.gateway`) ကို အသုံးပြုပါ။

Protocol စာတမ်းအပြည့်အစုံကို ကြည့်ရှုရန်: [Gateway Protocol](/gateway/protocol)。

## Operational checks

### Liveness

- WS ကို ဖွင့်ပြီး `connect` ပေးပို့ပါ။
- Snapshot ပါဝင်သော `hello-ok` တုံ့ပြန်မှုကို မျှော်လင့်ပါ။

### Readiness

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### Gap ပြန်လည်ရယူခြင်း

Events are not replayed. Sequence gap ဖြစ်ပေါ်ပါက ဆက်လက်လုပ်ဆောင်မီ state (`health`, `system-presence`) ကို ပြန်လည် refresh လုပ်ပါ။

## အဖြစ်များသော ချို့ယွင်းမှု လက္ခဏာများ

| လက္ခဏာ                                                         | ဖြစ်နိုင်ချေရှိသော ပြဿနာ                                  |
| -------------------------------------------------------------- | --------------------------------------------------------- |
| `refusing to bind gateway ... `without auth\`                  | token/password မရှိဘဲ loopback မဟုတ်သော bind ပြုလုပ်ခြင်း |
| `another gateway instance is already listening` / `EADDRINUSE` | Port ပဋိပက္ခ                                              |
| `Gateway start blocked: set gateway.mode=local`                | Config ကို remote mode အဖြစ် သတ်မှတ်ထားသည်                |
| `unauthorized` during connect                                  | client နှင့် gateway အကြား auth မကိုက်ညီမှု               |

ပြည့်စုံသော diagnosis အဆင့်များအတွက် [Gateway Troubleshooting](/gateway/troubleshooting) ကို အသုံးပြုပါ။

## Windows (WSL2)

- Gateway မရရှိနိုင်သည့်အခါ Gateway protocol clients များသည် ချက်ချင်း fail ဖြစ်သည် (implicit direct-channel fallback မရှိပါ)။
- Invalid/non-connect ပထမ frame များကို ပယ်ချပြီး connection ကို ပိတ်ပါသည်။
- Graceful shutdown ပြုလုပ်သည့်အခါ socket မပိတ်မီ `shutdown` event ကို ထုတ်လွှင့်ပါသည်။

---

ဆက်စပ်အကြောင်းအရာများ:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

