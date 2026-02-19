---
summary: "`openclaw hooks` (agent hooks) အတွက် CLI ကိုးကားချက်"
read_when:
  - agent hooks များကို စီမံခန့်ခွဲလိုသောအခါ
  - hooks များကို ထည့်သွင်းခြင်း သို့မဟုတ် အပ်ဒိတ်လုပ်လိုသောအခါ
title: "hooks"
---

# `openclaw hooks`

agent hooks များကို စီမံခန့်ခွဲရန် ( `/new`, `/reset` ကဲ့သို့သော အမိန့်များနှင့် Gateway စတင်ချိန်အတွက် ဖြစ်ရပ်အခြေပြု အလိုအလျောက်လုပ်ဆောင်မှုများ)။

ဆက်စပ်အကြောင်းအရာများ:

- Hooks: [Hooks](/automation/hooks)
- Plugin hooks: [Plugins](/tools/plugin#plugin-hooks)

## Hooks အားလုံးကို စာရင်းပြရန်

```bash
openclaw hooks list
```

workspace၊ managed နှင့် bundled ဒိုင်ရက်ထရီများမှ တွေ့ရှိထားသော hooks အားလုံးကို စာရင်းပြပါသည်။

**ရွေးချယ်စရာများ:**

- `--eligible`: အသုံးပြုနိုင်သော hooks များသာ ပြရန် (လိုအပ်ချက်များ ပြည့်မီပြီး)
- `--json`: JSON အဖြစ် ထုတ်ပြရန်
- `-v, --verbose`: မပြည့်မီသော လိုအပ်ချက်များအပါအဝင် အသေးစိတ်အချက်အလက်များ ပြရန်

**ဥပမာ ထွက်ရှိမှု:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**ဥပမာ (အသေးစိတ်):**

```bash
openclaw hooks list --verbose
```

အသုံးမပြုနိုင်သော hooks များအတွက် မရှိသေးသော လိုအပ်ချက်များကို ပြပါသည်။

**ဥပမာ (JSON):**

```bash
openclaw hooks list --json
```

ပရိုဂရမ်မှ အသုံးပြုရန် အတွက် ဖွဲ့စည်းထားသော JSON ကို ပြန်ပေးပါသည်။

## Hook အချက်အလက်ရယူရန်

```bash
openclaw hooks info <name>
```

သတ်မှတ်ထားသော hook တစ်ခုအတွက် အသေးစိတ် အချက်အလက်များကို ပြပါသည်။

**အငြင်းပွားချက်များ:**

- `<name>`: Hook အမည် (ဥပမာ၊ `session-memory`)

**ရွေးချယ်စရာများ:**

- `--json`: JSON အဖြစ် ထုတ်ပြရန်

**ဥပမာ:**

```bash
openclaw hooks info session-memory
```

**ထွက်ရှိမှု:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: openclaw-bundled
  Path: /path/to/openclaw/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/openclaw/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.openclaw.ai/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## Hooks အသုံးပြုနိုင်မှုကို စစ်ဆေးရန်

```bash
openclaw hooks check
```

hook အသုံးပြုနိုင်မှု အခြေအနေကို အကျဉ်းချုပ် ပြပါသည် (အဆင်သင့် ဖြစ်နေသည့် အရေအတွက်နှင့် မဖြစ်သေးသည့် အရေအတွက်)။

**ရွေးချယ်စရာများ:**

- `--json`: JSON အဖြစ် ထုတ်ပြရန်

**ဥပမာ ထွက်ရှိမှု:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## Hook တစ်ခုကို ဖွင့်ရန်

```bash
openclaw hooks enable <name>
```

သင့် config (`~/.openclaw/config.json`) ထဲသို့ ထည့်သွင်းခြင်းဖြင့် သတ်မှတ်ထားသော hook ကို ဖွင့်ပါသည်။

**မှတ်ချက်:** plugin များဖြင့် စီမံခန့်ခွဲထားသော hooks များသည် `openclaw hooks list` တွင် `plugin:<id>` ဟုပေါ်လာပြီး
ဒီနေရာမှ enable/disable မလုပ်နိုင်ပါ။ plugin ကို enable/disable လုပ်ပါ။

**အငြင်းပွားချက်များ:**

- `<name>`: Hook အမည် (ဥပမာ၊ `session-memory`)

**ဥပမာ:**

```bash
openclaw hooks enable session-memory
```

**ထွက်ရှိမှု:**

```
✓ Enabled hook: 💾 session-memory
```

**လုပ်ဆောင်ပုံ:**

- hook ရှိမရှိနှင့် အသုံးပြုနိုင်မှုကို စစ်ဆေးသည်
- သင့် config ထဲရှိ `hooks.internal.entries.<name>.enabled = true` ကို update လုပ်ပါ
- config ကို disk သို့ သိမ်းဆည်းသည်

**ဖွင့်ပြီးနောက်:**

- hooks များ ပြန်လည် load ဖြစ်ရန် Gateway ကို ပြန်စတင်ပါ (macOS တွင် menu bar app ကို ပြန်စတင်ခြင်း၊ သို့မဟုတ် dev အတွက် သင့် Gateway process ကို ပြန်စတင်ပါ)။

## Hook တစ်ခုကို ပိတ်ရန်

```bash
openclaw hooks disable <name>
```

သင့် config ကို အပ်ဒိတ်လုပ်ခြင်းဖြင့် သတ်မှတ်ထားသော hook ကို ပိတ်ပါသည်။

**အငြင်းပွားချက်များ:**

- `<name>`: Hook အမည် (ဥပမာ၊ `command-logger`)

**ဥပမာ:**

```bash
openclaw hooks disable command-logger
```

**ထွက်ရှိမှု:**

```
⏸ Disabled hook: 📝 command-logger
```

**ပိတ်ပြီးနောက်:**

- hooks များ ပြန်လည် load ဖြစ်ရန် Gateway ကို ပြန်စတင်ပါ

## Hooks ထည့်သွင်းရန်

```bash
openclaw hooks install <path-or-spec>
```

local ဖိုလ်ဒါ/အာခိုင်ဗ် သို့မဟုတ် npm မှ hook pack တစ်ခုကို ထည့်သွင်းပါသည်။

Npm specs များသည် **registry-only** (package name + optional version/tag) ဖြစ်ရမည်။ Git/URL/file
specs များကို လက်မခံပါ။ လုံခြုံရေးအတွက် dependency install များကို `--ignore-scripts` ဖြင့် လုပ်ဆောင်သည်။

**လုပ်ဆောင်ပုံ:**

- hook pack ကို `~/.openclaw/hooks/<id>` ထဲသို့ ကူးယူသည်
- ထည့်သွင်းထားသော hooks များကို `hooks.internal.entries.*` ထဲတွင် ဖွင့်ပေးသည်
- ထည့်သွင်းမှုကို `hooks.internal.installs` အောက်တွင် မှတ်တမ်းတင်သည်

**ရွေးချယ်စရာများ:**

- `-l, --link`: ကူးယူမည့်အစား local ဒိုင်ရက်ထရီကို ချိတ်ဆက်ပါ ( `hooks.internal.load.extraDirs` ထဲသို့ ထည့်ပေါင်းသည်)

**ဥပမာများ:**

**ဥပမာများ:**

```bash
# Local directory
openclaw hooks install ./my-hook-pack

# Local archive
openclaw hooks install ./my-hook-pack.zip

# NPM package
openclaw hooks install @openclaw/my-hook-pack

# Link a local directory without copying
openclaw hooks install -l ./my-hook-pack
```

## Hooks အပ်ဒိတ်လုပ်ရန်

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

**ရွေးချယ်စရာများ:**

**ရွေးချယ်စရာများ:**

- `--all`: ခြေရာခံထားသော hook packs အားလုံးကို အပ်ဒိတ်လုပ်ရန်
- `--dry-run`: ရေးသားမပြုလုပ်ဘဲ ဘာတွေ ပြောင်းလဲမည်ကို ပြရန်

## Bundled Hooks

### session-memory

**ဖွင့်ရန်:**

**ဖွင့်ရန်:**

```bash
openclaw hooks enable session-memory
```

**ကြည့်ရန်:** [session-memory documentation](/automation/hooks#session-memory)

**ကြည့်ရန်:** [session-memory documentation](/automation/hooks#session-memory)

### bootstrap-extra-files

**ဖွင့်ရန်:**

**ဖွင့်ရန်:**

```bash
openclaw hooks enable bootstrap-extra-files
```

**မှတ်တမ်းများ ကြည့်ရန်:**

### command-logger

**ကြည့်ရန်:** [command-logger documentation](/automation/hooks#command-logger)

**ဖွင့်ရန်:**

```bash
openclaw hooks enable command-logger
```

**ဖွင့်ရန်:**

**မှတ်တမ်းများ ကြည့်ရန်:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**ကြည့်ရန်:** [command-logger documentation](/automation/hooks#command-logger)

### boot-md

**ဖြစ်ရပ်များ**: `gateway:startup`

**ဖွင့်ရန်**:

**ဖွင့်ရန်**:

```bash
openclaw hooks enable boot-md
```

**ကြည့်ရန်:** [boot-md documentation](/automation/hooks#boot-md)

