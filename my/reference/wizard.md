---
title: "Onboarding Wizard အညွှန်း"
sidebarTitle: "Wizard အညွှန်း"
---

# Onboarding Wizard အညွှန်း

ဤစာတမ်းသည် `openclaw onboard` CLI wizard အတွက် ပြည့်စုံသော reference ဖြစ်သည်။
အထွေထွေမြင်ကွင်းအတွက် [Onboarding Wizard](/start/wizard) ကို ကြည့်ပါ။

## လုပ်ငန်းစဉ် အသေးစိတ် (local mode)

<Steps>
  <Step title="Existing config detection">
    - `~/.openclaw/openclaw.json` ရှိပါက **Keep / Modify / Reset** ကို ရွေးချယ်ပါ။
    - Wizard ကို ပြန်လည် run လုပ်ခြင်းသည် **Reset** ကို သင်ကိုယ်တိုင် ရွေးချယ်ခြင်း (သို့မဟုတ် `--reset` ပေးခြင်း) မရှိပါက မည်သည့်အရာကိုမျှ မဖျက်ပါ။
    - Config မမှန်ကန်ပါက သို့မဟုတ် legacy keys ပါဝင်ပါက wizard သည် ရပ်ပြီး ဆက်လက်လုပ်ဆောင်ရန် မတိုင်မီ `openclaw doctor` ကို run လုပ်ရန် မေးမြန်းပါမည်။
    - Reset သည် `trash` ကို အသုံးပြုသည် (`rm` ကို ဘယ်တော့မှ မသုံးပါ) နှင့် scope များကို ပေးထားသည်:
      - Config only
      - Config + credentials + sessions
      - Full reset (workspace ကိုပါ ဖယ်ရှားသည်)
  
</Step>
  <Step title="Model/Auth">
    - **Anthropic API key (recommended)**: `ANTHROPIC_API_KEY` ရှိပါက အသုံးပြုပါသည် သို့မဟုတ် key ထည့်ရန် မေးမြန်းပြီး daemon အတွက် သိမ်းဆည်းပါသည်။
    - **Anthropic OAuth (Claude Code CLI)**: macOS တွင် wizard သည် Keychain item "Claude Code-credentials" ကို စစ်ဆေးပါသည် ("Always Allow" ကို ရွေးချယ်ပါ၊ launchd စတင်ရာတွင် မတားဆီးစေရန်); Linux/Windows တွင် `~/.claude/.credentials.json` ရှိပါက ပြန်လည်အသုံးပြုပါသည်။
    - **Anthropic token (paste setup-token)**: မည်သည့်စက်တွင်မဆို `claude setup-token` ကို run လုပ်ပြီး token ကို paste လုပ်ပါ (အမည်ပေးနိုင်သည်; အလွတ် = default)။
    - **OpenAI Code (Codex) subscription (Codex CLI)**: `~/.codex/auth.json` ရှိပါက wizard သည် ပြန်လည်အသုံးပြုနိုင်ပါသည်။
    - **OpenAI Code (Codex) subscription (OAuth)**: browser flow; `code#state` ကို paste လုပ်ပါ။
      - Model ကို မသတ်မှတ်ထားပါက သို့မဟုတ် `openai/*` ဖြစ်ပါက `agents.defaults.model` ကို `openai-codex/gpt-5.2` အဖြစ် သတ်မှတ်ပါသည်။
    - **OpenAI API key**: `OPENAI_API_KEY` ရှိပါက အသုံးပြုပါသည် သို့မဟုတ် key ထည့်ရန် မေးမြန်းပြီး `~/.openclaw/.env` တွင် သိမ်းဆည်းပါသည်၊ launchd ဖတ်နိုင်ရန်။
    - **xAI (Grok) API key**: `XAI_API_KEY` ကို မေးမြန်းပြီး xAI ကို model provider အဖြစ် configure လုပ်ပါသည်။
    - **OpenCode Zen (multi-model proxy)**: `OPENCODE_API_KEY` (သို့မဟုတ် `OPENCODE_ZEN_API_KEY`, https://opencode.ai/auth တွင် ရယူနိုင်သည်) ကို မေးမြန်းပါသည်။
    - **API key**: key ကို သင့်အတွက် သိမ်းဆည်းပေးပါသည်။
    - **Vercel AI Gateway (multi-model proxy)**: `AI_GATEWAY_API_KEY` ကို မေးမြန်းပါသည်။
    - အသေးစိတ်: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: Account ID, Gateway ID နှင့် `CLOUDFLARE_AI_GATEWAY_API_KEY` ကို မေးမြန်းပါသည်။
    - အသေးစိတ်: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: config ကို အလိုအလျောက် ရေးသားပါသည်။
    - အသေးစိတ်: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: `SYNTHETIC_API_KEY` ကို မေးမြန်းပါသည်။
    - အသေးစိတ်: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: config ကို အလိုအလျောက် ရေးသားပါသည်။
    - **Kimi Coding**: config ကို အလိုအလျောက် ရေးသားပါသည်။
    - အသေးစိတ်: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: ယခုအချိန်တွင် auth ကို မ configure လုပ်ပါ။
    - တွေ့ရှိထားသော option များထဲမှ default model ကို ရွေးချယ်ပါ (သို့မဟုတ် provider/model ကို ကိုယ်တိုင် ထည့်သွင်းပါ)။
    - Wizard သည် model စစ်ဆေးမှုကို run လုပ်ပြီး configured model မသိရှိပါက သို့မဟုတ် auth မရှိပါက သတိပေးပါသည်။
    - OAuth credentials များကို `~/.openclaw/credentials/oauth.json` တွင် သိမ်းထားပြီး auth profiles များကို `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth) တွင် သိမ်းထားပါသည်။
    - အသေးစိတ်: [/concepts/oauth](/concepts/oauth)
    <Note>
    Headless/server အကြံပြုချက်: browser ပါသော စက်တစ်လုံးတွင် OAuth ကို ပြီးစီးစေပြီးနောက်
    `~/.openclaw/credentials/oauth.json` (သို့မဟုတ် `$OPENCLAW_STATE_DIR/credentials/oauth.json`) ကို
    gateway host သို့ ကူးယူပါ။
    
</Note>
  
</Step>
  <Step title="Workspace">
    - Default `~/.openclaw/workspace` (configure လုပ်နိုင်သည်)။
    - Agent bootstrap ritual အတွက် လိုအပ်သော workspace ဖိုင်များကို seed လုပ်ပါသည်။
    - Workspace အပြည့်အစုံ ဖွဲ့စည်းပုံနှင့် backup လမ်းညွှန်: [Agent workspace](/concepts/agent-workspace)
  
</Step>
  <Step title="Gateway">
    - Port, bind, auth mode, tailscale exposure။
    - Auth အကြံပြုချက်: loopback အတွက်တောင် **Token** ကို ထိန်းထားပါ၊ ဒါမှ local WS clients များသည် authenticate လုပ်ရပါမည်။
    - Local process အားလုံးကို အပြည့်အဝ ယုံကြည်မှသာ auth ကို disable လုပ်ပါ။
    - Non‑loopback bind များသည် auth လိုအပ်နေဆဲ ဖြစ်သည်။
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): optional QR login။
    - [Telegram](/channels/telegram): bot token။
    - [Discord](/channels/discord): bot token။
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience။
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL။
    - [Signal](/channels/signal): optional `signal-cli` install + account config။
    - [BlueBubbles](/channels/bluebubbles): **iMessage အတွက် အကြံပြုထားသည်**; server URL + password + webhook။
    - [iMessage](/channels/imessage): legacy `imsg` CLI path + DB access။
    - DM security: default သည် pairing ဖြစ်သည်။ ပထမဆုံး DM တွင် code တစ်ခု ပို့ပါသည်; `openclaw pairing approve <channel> <code>` ဖြင့် အတည်ပြုပါ သို့မဟုတ် allowlists ကို အသုံးပြုပါ။
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Logged-in user session လိုအပ်သည်; headless အတွက် custom LaunchDaemon ကို အသုံးပြုပါ (မပို့ပေးထားပါ)။
    - Linux (နှင့် Windows via WSL2): systemd user unit
      - Logout ပြီးနောက် Gateway ဆက်လက်လုပ်ဆောင်ရန် wizard သည် `loginctl enable-linger <user>` ဖြင့် lingering ကို enable လုပ်ရန် ကြိုးပမ်းပါသည်။
      - sudo ကို မေးမြန်းနိုင်ပါသည် (`/var/lib/systemd/linger` ကို ရေးသားသည်); ပထမဦးစွာ sudo မလိုဘဲ ကြိုးပမ်းပါသည်။
    - **Runtime ရွေးချယ်မှု:** Node (အကြံပြုထားသည်; WhatsApp/Telegram အတွက် လိုအပ်သည်)။ Bun ကို **အကြံမပြုပါ**။
  
</Step>
  <Step title="Health check">
    - Gateway ကို (လိုအပ်ပါက) စတင်ပြီး `openclaw health` ကို လုပ်ဆောင်ပါသည်။
    - အကြံပြုချက်: `openclaw status --deep` သည် status output ထဲသို့ gateway health probes များကို ထပ်ထည့်ပေးပါသည် (အသုံးပြုနိုင်သော gateway လိုအပ်သည်)။
  
</Step>
  <Step title="Skills (recommended)">
    - ရရှိနိုင်သော skills များကို ဖတ်ရှုပြီး လိုအပ်ချက်များကို စစ်ဆေးပါသည်။
    - Node manager ကို ရွေးချယ်နိုင်ပါသည်: **npm / pnpm** (bun ကို အကြံမပြုပါ)။
    - Optional dependencies များကို ထည့်သွင်းပါသည် (အချို့သည် macOS တွင် Homebrew ကို အသုံးပြုပါသည်)။
  
</Step>
  <Step title="Finish">
    - Summary နှင့် next steps များကို ပြသပါသည်၊ အပို features များအတွက် iOS/Android/macOS apps များ ပါဝင်သည်။
  
</Step>
</Steps>

<Note>
GUI မတွေ့ရှိပါက wizard သည် browser ဖွင့်မည့်အစား Control UI အတွက် SSH port-forward လမ်းညွှန်ချက်များကို ပြသပါသည်။
Control UI assets မရှိပါက wizard သည် build လုပ်ရန် ကြိုးပမ်းပါသည်; fallback အဖြစ် `pnpm ui:build` ကို အသုံးပြုပါသည် (UI deps များကို auto-install လုပ်ပါသည်)။
</Note>

## Non-interactive mode

Onboarding ကို အလိုအလျောက်လုပ်ဆောင်ရန် သို့မဟုတ် script ဖြင့် လုပ်ရန် `--non-interactive` ကို အသုံးပြုပါ။

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Machine‑readable summary ရရှိရန် `--json` ကို ထပ်ထည့်ပါ။

<Note>
`--json` သည် non-interactive mode ကို အလိုအလျောက် မဆိုလိုပါ။ Script များအတွက် `--non-interactive` (နှင့် `--workspace`) ကို အသုံးပြုပါ။
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
</AccordionGroup>

### Add agent (non-interactive)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

Gateway သည် wizard flow ကို RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`) မှတစ်ဆင့် expose လုပ်ပါသည်။
Clients (macOS app, Control UI) များသည် onboarding logic ကို ပြန်လည်မရေးသားဘဲ steps များကို render လုပ်နိုင်ပါသည်။

## Signal setup (signal-cli)

Wizard သည် GitHub releases မှ `signal-cli` ကို ထည့်သွင်းနိုင်ပါသည်။

- သင့် platform နှင့် ကိုက်ညီသော release asset ကို ဒေါင်းလုဒ်လုပ်ပါသည်။
- `~/.openclaw/tools/signal-cli/<version>/` အောက်တွင် သိမ်းဆည်းပါသည်။
- သင့် config ထဲသို့ `channels.signal.cliPath` ကို ရေးသွင်းပါသည်။

မှတ်ချက်များ:

- JVM builds များအတွက် **Java 21** လိုအပ်ပါသည်။
- Native builds များကို ရရှိနိုင်ပါက အသုံးပြုပါသည်။
- Windows သည် WSL2 ကို အသုံးပြုပါသည်; signal-cli install သည် WSL အတွင်း Linux flow အတိုင်း ဆက်လက်လုပ်ဆောင်ပါသည်။

## Wizard က ရေးသွင်းသော အရာများ

`~/.openclaw/openclaw.json` ထဲရှိ ပုံမှန် field များ:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (Minimax ကို ရွေးချယ်ပါက)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Prompt များအတွင်း ရွေးချယ်ပါက Channel allowlists (Slack/Discord/Matrix/Microsoft Teams) ကို ထည့်သွင်းရေးသားပါသည် (အမည်များကို ဖြစ်နိုင်သမျှ ID များသို့ ပြောင်းလဲပါသည်)။
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` သည် `agents.list[]` နှင့် optional `bindings` ကို ရေးသွင်းပါသည်။

WhatsApp credentials များကို `~/.openclaw/credentials/whatsapp/<accountId>/` အောက်တွင် သိမ်းဆည်းပါသည်။
Sessions များကို `~/.openclaw/agents/<agentId>/sessions/` အောက်တွင် သိမ်းဆည်းပါသည်။

အချို့ channels များကို plugins အဖြစ် ပေးပို့ပါသည်။ Onboarding အတွင်း တစ်ခုကို ရွေးချယ်ပါက configure မလုပ်မီ (npm သို့မဟုတ် local path ဖြင့်) install လုပ်ရန် wizard က မေးမြန်းပါမည်။

## Related docs

- Wizard overview: [Onboarding Wizard](/start/wizard)
- macOS app onboarding: [Onboarding](/start/onboarding)
- Config reference: [Gateway configuration](/gateway/configuration)
- Providers: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)

