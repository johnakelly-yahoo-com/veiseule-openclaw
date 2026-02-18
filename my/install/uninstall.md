---
summary: "OpenClaw ကို အပြည့်အဝ ဖယ်ရှားခြင်း (CLI၊ service၊ state၊ workspace)"
read_when:
  - စက်တစ်လုံးမှ OpenClaw ကို ဖယ်ရှားလိုသောအခါ
  - uninstall ပြီးနောက် Gateway service ဆက်လက် လည်ပတ်နေသေးသောအခါ
title: "ဖယ်ရှားခြင်း"
---

# ဖယ်ရှားခြင်း

လမ်းကြောင်း နှစ်မျိုးရှိသည် —

- **လွယ်ကူသော လမ်းကြောင်း** — `openclaw` ကို ဆက်လက် ထည့်သွင်းထားဆဲ ဖြစ်ပါက။
- **service ကို လက်ဖြင့် ဖယ်ရှားခြင်း** — CLI မရှိတော့သော်လည်း service ဆက်လက် လည်ပတ်နေပါက။

## လွယ်ကူသော လမ်းကြောင်း (CLI ဆက်လက် ထည့်သွင်းထားဆဲ)

အကြံပြုချက် — ပါရှိပြီးသား uninstaller ကို အသုံးပြုပါ —

```bash
openclaw uninstall
```

အပြန်အလှန်မလိုသော (automation / npx):

```bash
openclaw uninstall --all --yes --non-interactive
npx -y openclaw uninstall --all --yes --non-interactive
```

လက်ဖြင့် အဆင့်လိုက် ဆောင်ရွက်ခြင်း (ရလဒ်တူညီ):

1. Gateway service ကို ရပ်တန့်ပါ —

```bash
openclaw gateway stop
```

2. Gateway service ကို ဖယ်ရှားပါ (launchd/systemd/schtasks) —

```bash
openclaw gateway uninstall
```

3. state + config ကို ဖျက်ပါ —

```bash
rm -rf "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
```

`OPENCLAW_CONFIG_PATH` ကို state dir အပြင်ဘက်ရှိ custom location သို့ သတ်မှတ်ထားပါက အဲဒီဖိုင်ကိုပါ ဖျက်ပါ။

4. workspace ကို ဖျက်ပါ (ရွေးချယ်နိုင်သည်၊ agent ဖိုင်များကို ဖယ်ရှားပါမည်) —

```bash
rm -rf ~/.openclaw/workspace
```

5. CLI install ကို ဖယ်ရှားပါ (သင် အသုံးပြုခဲ့သည့် နည်းလမ်းကို ရွေးချယ်ပါ) —

```bash
npm rm -g openclaw
pnpm remove -g openclaw
bun remove -g openclaw
```

6. macOS app ကို ထည့်သွင်းထားပါက —

```bash
rm -rf /Applications/OpenClaw.app
```

မှတ်ချက်များ —

- profiles (`--profile` / `OPENCLAW_PROFILE`) ကို အသုံးပြုထားပါက state dir တစ်ခုချင်းစီအတွက် အဆင့် 3 ကို ထပ်လုပ်ပါ (မူလတန်ဖိုးများမှာ `~/.openclaw-<profile>`) ဖြစ်သည်။
- remote mode တွင် state dir သည် **Gateway ဟို့စ်** ပေါ်တွင် ရှိနေသဖြင့် အဆင့် 1-4 ကို အဲဒီနေရာတွင်ပါ ဆောင်ရွက်ပါ။

## service ကို လက်ဖြင့် ဖယ်ရှားခြင်း (CLI မရှိ)

Gateway service ဆက်လက် လည်ပတ်နေသော်လည်း `openclaw` မရှိပါက ဒီနည်းကို အသုံးပြုပါ။

### macOS (launchd)

မူလ label သည် `bot.molt.gateway` ဖြစ်သည် (သို့မဟုတ် `bot.molt.<profile>`; အဟောင်း `com.openclaw.*` သည် ဆက်လက်ရှိနိုင်သည်):

```bash
launchctl bootout gui/$UID/bot.molt.gateway
rm -f ~/Library/LaunchAgents/bot.molt.gateway.plist
```

profile ကို အသုံးပြုထားပါက label နှင့် plist အမည်ကို `bot.molt.<profile>` သို့ အစားထိုးပါ။ ရှိနေပါက အဟောင်း `com.openclaw.*` plists များကို ဖယ်ရှားပါ။

### Linux (systemd user unit)

မူလ unit အမည်သည် `openclaw-gateway.service` (သို့မဟုတ် `openclaw-gateway-<profile>.service`) —

```bash
systemctl --user disable --now openclaw-gateway.service
rm -f ~/.config/systemd/user/openclaw-gateway.service
systemctl --user daemon-reload
```

### Windows (Scheduled Task)

မူလ task အမည်မှာ `OpenClaw Gateway` ဖြစ်သည် (သို့မဟုတ် `OpenClaw Gateway (<profile>)`)။
The task script lives under your state dir.

```powershell
schtasks /Delete /F /TN "OpenClaw Gateway"
Remove-Item -Force "$env:USERPROFILE\.openclaw\gateway.cmd"
```

profile ကို အသုံးပြုထားပါက ကိုက်ညီသော task အမည်နှင့် `~\.openclaw-<profile>\gateway.cmd` ကို ဖျက်ပါ။

## ပုံမှန် install နှင့် source checkout

### ပုံမှန် install (install.sh / npm / pnpm / bun)

`https://openclaw.ai/install.sh` သို့မဟုတ် `install.ps1` ကို အသုံးပြုခဲ့ပါက CLI ကို `npm install -g openclaw@latest` ဖြင့် ထည့်သွင်းထားသည်။
Remove it with `npm rm -g openclaw` (or `pnpm remove -g` / `bun remove -g` if you installed that way).

### Source checkout (git clone)

repo checkout (`git clone` + `openclaw ...` / `bun run openclaw ...`) မှ လည်ပတ်နေပါက —

1. repo ကို မဖျက်မီ **Gateway service ကို အရင် ဖယ်ရှားပါ** (အပေါ်ပါ လွယ်ကူသော လမ်းကြောင်း သို့မဟုတ် service ကို လက်ဖြင့် ဖယ်ရှားခြင်း ကို အသုံးပြုပါ)။
2. repo directory ကို ဖျက်ပါ။
3. အပေါ်တွင် ဖော်ပြထားသည့်အတိုင်း state + workspace ကို ဖယ်ရှားပါ။
