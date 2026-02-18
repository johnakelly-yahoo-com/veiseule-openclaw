---
title: "ဟုခ်များ"
---

# ဟုခ်များ

Hooks များသည် agent command များနှင့် event များကို တုံ့ပြန်ပြီး အလိုအလျောက်လုပ်ဆောင်မှုများ ပြုလုပ်နိုင်ရန် အဆင့်မြှင့်နိုင်သော event-driven စနစ်တစ်ခုကို ပေးစွမ်းပါသည်။ Hooks များကို directory များမှ အလိုအလျောက် ရှာဖွေတွေ့ရှိပြီး OpenClaw တွင် skills များကို စီမံခန့်ခွဲသကဲ့သို့ CLI command များမှတစ်ဆင့် စီမံနိုင်ပါသည်။

## အခြေခံသဘောတရားများကို လေ့လာခြင်း

Hooks are small scripts that run when something happens. အမျိုးအစား နှစ်မျိုး ရှိပါသည်:

- **Hooks** (ဤစာမျက်နှာ): agent အဖြစ်အပျက်များဖြစ်ပေါ်လာသည့်အခါ Gateway အတွင်းတွင် run လုပ်သည်၊ ဥပမာ `/new`၊ `/reset`၊ `/stop` သို့မဟုတ် lifecycle အဖြစ်အပျက်များ။
- **Webhooks**: အပြင်ဘက် HTTP webhooks များဖြစ်ပြီး အခြားစနစ်များမှ OpenClaw တွင် အလုပ်များကို trigger လုပ်နိုင်စေပါသည်။ [Webhook Hooks](/automation/webhook) ကို ကြည့်ရှုပါ သို့မဟုတ် Gmail helper command များအတွက် `openclaw webhooks` ကို အသုံးပြုပါ။

Hooks များကို plugins အတွင်းတွင်လည်း bundle လုပ်နိုင်သည်။ [Plugins](/tools/plugin#plugin-hooks) ကိုကြည့်ပါ။

အများအားဖြင့် အသုံးပြုမှုများ–

- session ကို reset လုပ်သည့်အခါ memory snapshot ကို သိမ်းဆည်းခြင်း
- troubleshooting သို့မဟုတ် compliance အတွက် command များ၏ audit trail ကို ထိန်းသိမ်းခြင်း
- session စတင်ခြင်း သို့မဟုတ် အဆုံးသတ်ခြင်းတွင် နောက်ဆက်တွဲ အလိုအလျောက်လုပ်ဆောင်မှုများကို trigger လုပ်ခြင်း
- အဖြစ်အပျက်များ ဖြစ်ပေါ်လာသည့်အခါ agent workspace ထဲသို့ ဖိုင်များရေးသားခြင်း သို့မဟုတ် အပြင်ဘက် API များကို ခေါ်ယူခြင်း

TypeScript function သေးသေးလေး တစ်ခု ရေးနိုင်ပါက hook တစ်ခုကို ရေးနိုင်ပါသည်။ Hooks များကို အလိုအလျောက် ရှာဖွေတွေ့ရှိပြီး CLI မှတစ်ဆင့် enable သို့မဟုတ် disable လုပ်နိုင်ပါသည်။

## အကျဉ်းချုပ်

Hooks စနစ်သည် အောက်ပါအရာများကို ခွင့်ပြုသည်–

- `/new` ကို ထုတ်ပြန်သည့်အခါ session context ကို memory သို့ သိမ်းဆည်းခြင်း
- auditing အတွက် command များအားလုံးကို log လုပ်ခြင်း
- agent lifecycle အဖြစ်အပျက်များပေါ်တွင် custom automation များကို trigger လုပ်ခြင်း
- core code ကို မပြုပြင်ဘဲ OpenClaw ၏ အပြုအမူကို တိုးချဲ့ခြင်း

## စတင်အသုံးပြုခြင်း

### တွဲဖက်ပါဝင်သော ဟုခ်များ

OpenClaw တွင် အလိုအလျောက် ရှာဖွေတွေ့ရှိနိုင်သော bundled hooks လေးခု ပါဝင်လာသည်–

- **💾 session-memory**: `/new` ကို ထုတ်ပြန်သည့်အခါ session context ကို agent workspace (default `~/.openclaw/workspace/memory/`) သို့ သိမ်းဆည်းသည်
- **📝 command-logger**: command အဖြစ်အပျက်များအားလုံးကို `~/.openclaw/logs/commands.log` သို့ log လုပ်သည်
- **🚀 boot-md**: gateway စတင်သည့်အခါ `BOOT.md` ကို run လုပ်သည် (internal hooks enabled လိုအပ်)
- **😈 soul-evil**: purge window အတွင်း သို့မဟုတ် ကျပန်းဖြစ်နိုင်ချေဖြင့် injected `SOUL.md` content ကို `SOUL_EVIL.md` ဖြင့် အစားထိုးသည်

ရရှိနိုင်သော hooks များကို စာရင်းပြုစုရန်–

```bash
openclaw hooks list
```

hook တစ်ခုကို enable လုပ်ရန်–

```bash
openclaw hooks enable session-memory
```

hook အခြေအနေကို စစ်ဆေးရန်–

```bash
openclaw hooks check
```

အသေးစိတ် အချက်အလက်များကို ရယူရန်–

```bash
openclaw hooks info session-memory
```

### စတင်မိတ်ဆက်ခြင်း

Onboarding (`openclaw onboard`) အတွင်း အကြံပြုထားသော hooks များကို enable လုပ်ရန် သင့်ကို မေးမြန်းပါလိမ့်မည်။ Wizard သည် သင့်လျော်သော hooks များကို အလိုအလျောက် ရှာဖွေတွေ့ရှိပြီး ရွေးချယ်ရန် ပြသပေးပါသည်။

## ဟုခ် ရှာဖွေခြင်း

Hooks များကို အောက်ပါ directory သုံးခုမှ (အလေးထားမှု အစဉ်လိုက်) အလိုအလျောက် ရှာဖွေတွေ့ရှိသည်–

1. **Workspace hooks**: `<workspace>/hooks/` (အေးဂျင့်တစ်ခုချင်းစီအလိုက်၊ အလေးထားမှုအမြင့်ဆုံး)
2. **Managed hooks**: `~/.openclaw/hooks/` (အသုံးပြုသူထည့်သွင်းထားသော၊ workspace များအကြား မျှဝေသုံးစွဲ)
3. **Bundled hooks**: `<openclaw>/dist/hooks/bundled/` (OpenClaw နှင့်အတူ ပါဝင်လာသည်)

Managed hook directories များသည် **single hook** တစ်ခု သို့မဟုတ် **hook pack** (package directory) တစ်ခု ဖြစ်နိုင်သည်။

hook တစ်ခုချင်းစီသည် အောက်ပါအရာများပါဝင်သည့် directory တစ်ခုဖြစ်သည်–

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## ဟုခ် အစုများ (npm/archives)

Hook packs များသည် `package.json` အတွင်းရှိ `openclaw.hooks` မှတစ်ဆင့် hook တစ်ခု သို့မဟုတ် တစ်ခုထက်ပို၍ export လုပ်ပေးသော standard npm packages များဖြစ်ပါသည်။ အောက်ပါအတိုင်း install လုပ်ပါ:

```bash
openclaw hooks install <path-or-spec>
```

`package.json` ဥပမာ–

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Entry တစ်ခုချင်းစီသည် `HOOK.md` နှင့် `handler.ts` (သို့မဟုတ် `index.ts`) ပါဝင်သော hook directory တစ်ခုကို ညွှန်ပြပါသည်။
Hook packs can ship dependencies; they will be installed under `~/.openclaw/hooks/<id>`.

## Hook Structure

### HOOK.md Format

`HOOK.md` ဖိုင်တွင် YAML frontmatter အဖြစ် metadata နှင့် Markdown documentation ပါဝင်သည်–

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.openclaw.ai/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### Metadata Fields

`metadata.openclaw` object သည် အောက်ပါအရာများကို ပံ့ပိုးသည်–

- **`emoji`**: CLI အတွက် ပြသမည့် emoji (ဥပမာ `"💾"`)
- **`events`**: နားထောင်မည့် အဖြစ်အပျက်များ၏ array (ဥပမာ `["command:new", "command:reset"]`)
- **`export`**: အသုံးပြုမည့် named export (default `"default"`)
- **`homepage`**: Documentation URL
- **`requires`**: ရွေးချယ်နိုင်သော လိုအပ်ချက်များ
  - **`bins`**: PATH ပေါ်တွင် လိုအပ်သော binaries (ဥပမာ `["git", "node"]`)
  - **`anyBins`**: အနည်းဆုံး binary တစ်ခု ရှိရမည်
  - **`env`**: လိုအပ်သော environment variables
  - **`config`**: လိုအပ်သော config paths (ဥပမာ `["workspace.dir"]`)
  - **`os`**: လိုအပ်သော platform များ (ဥပမာ `["darwin", "linux"]`)
- **`always`**: eligibility checks ကို ကျော်လွှားရန် (boolean)
- **`install`**: Installation methods (bundled hooks အတွက်: `[{"id":"bundled","kind":"bundled"}]`)

### Handler Implementation

`handler.ts` ဖိုင်သည် `HookHandler` function တစ်ခုကို export လုပ်သည်–

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push("✨ My hook executed!");
};

export default myHandler;
```

#### Event Context

event တစ်ခုချင်းစီတွင် အောက်ပါအရာများ ပါဝင်သည်–

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: OpenClawConfig
  }
}
```

## Event Types

### Command Events

agent commands များကို ထုတ်ပြန်သည့်အခါ trigger လုပ်သည်–

- **`command`**: command အဖြစ်အပျက်များအားလုံး (general listener)
- **`command:new`**: `/new` command ကို ထုတ်ပြန်သည့်အခါ
- **`command:reset`**: `/reset` command ကို ထုတ်ပြန်သည့်အခါ
- **`command:stop`**: `/stop` command ကို ထုတ်ပြန်သည့်အခါ

### Agent Events

- **`agent:bootstrap`**: workspace bootstrap ဖိုင်များကို inject မလုပ်မီ (hooks များသည် `context.bootstrapFiles` ကို ပြုပြင်နိုင်သည်)

### Gateway Events

gateway စတင်သည့်အခါ trigger လုပ်သည်–

- **`gateway:startup`**: channels စတင်ပြီး hooks များ load လုပ်ပြီးနောက်

### Tool Result Hooks (Plugin API)

ဤ hooks များသည် event-stream listener မဟုတ်ပါ။ OpenClaw မှ persist မလုပ်မီ tool results များကို plugin များက synchronous အနေဖြင့် ပြင်ဆင်နိုင်စေသည်။

- **`tool_result_persist`**: session transcript ထဲသို့ မရေးမီ tool result များကို ပြောင်းလဲပြုပြင်ရန်။ Synchronous ဖြစ်ရပါမည်; ပြင်ဆင်ပြီးသော tool result payload ကို return လုပ်ပါ သို့မဟုတ် မပြောင်းလဲလိုပါက `undefined` ကို return လုပ်ပါ။ [Agent Loop](/concepts/agent-loop) ကို ကြည့်ရှုပါ။

### Future Events

စီမံကိန်းထားသော event အမျိုးအစားများ–

- **`session:start`**: session အသစ် စတင်သည့်အခါ
- **`session:end`**: session အဆုံးသတ်သည့်အခါ
- **`agent:error`**: agent မှ error ကြုံတွေ့သည့်အခါ
- **`message:sent`**: message တစ်ခု ပို့လိုက်သည့်အခါ
- **`message:received`**: message တစ်ခု လက်ခံရရှိသည့်အခါ

## Creating Custom Hooks

### ၁။ တည်နေရာကို ရွေးချယ်ပါ

- **Workspace hooks** (`<workspace>/hooks/`): အေးဂျင့်တစ်ခုချင်းစီအလိုက်၊ အလေးထားမှုအမြင့်ဆုံး
- **Managed hooks** (`~/.openclaw/hooks/`): workspace များအကြား မျှဝေသုံးစွဲ

### ၂။ Directory Structure ကို ဖန်တီးပါ

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### ၃။ HOOK.md ကို ဖန်တီးပါ

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### ၄။ handler.ts ကို ဖန်တီးပါ

```typescript
import type { HookHandler } from "../../src/hooks/hooks.js";

const handler: HookHandler = async (event) => {
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  console.log("[my-hook] Running!");
  // Your logic here
};

export default handler;
```

### ၅။ Enable လုပ်ပြီး စမ်းသပ်ပါ

```bash
# Verify hook is discovered
openclaw hooks list

# Enable it
openclaw hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## Configuration

### New Config Format (Recommended)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Per-Hook Configuration

Hooks များတွင် custom configuration ရှိနိုင်သည်–

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Extra Directories

ထပ်မံသော directory များမှ hooks များကို load လုပ်ရန်–

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Legacy Config Format (Still Supported)

နောက်ပြန်လိုက်ဖက်ညီမှုအတွက် config format အဟောင်းကို ဆက်လက် ပံ့ပိုးထားသည်–

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**Migration**: hook အသစ်များအတွက် discovery-based စနစ်အသစ်ကို အသုံးပြုပါ။ Legacy handler များကို directory-based hooks များပြီးနောက် load လုပ်ပါသည်။

## CLI Commands

### List Hooks

```bash
# List all hooks
openclaw hooks list

# Show only eligible hooks
openclaw hooks list --eligible

# Verbose output (show missing requirements)
openclaw hooks list --verbose

# JSON output
openclaw hooks list --json
```

### Hook Information

```bash
# Show detailed info about a hook
openclaw hooks info session-memory

# JSON output
openclaw hooks info session-memory --json
```

### Check Eligibility

```bash
# Show eligibility summary
openclaw hooks check

# JSON output
openclaw hooks check --json
```

### Enable/Disable

```bash
# Enable a hook
openclaw hooks enable session-memory

# Disable a hook
openclaw hooks disable command-logger
```

## Bundled hook reference

### session-memory

`/new` ကို ထုတ်ပြန်သည့်အခါ session context ကို memory သို့ သိမ်းဆည်းသည်။

**Events**: `command:new`

**Requirements**: `workspace.dir` ကို configuration ပြုလုပ်ထားရမည်

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (default `~/.openclaw/workspace`)

**What it does**:

1. pre-reset session entry ကို အသုံးပြုပြီး မှန်ကန်သော transcript ကို ရှာဖွေသည်
2. စကားဝိုင်း၏ နောက်ဆုံး စာကြောင်း ၁၅ ကြောင်းကို ထုတ်ယူသည်
3. LLM ကို အသုံးပြုပြီး ဖော်ပြချက်ပါသော filename slug ကို ဖန်တီးသည်
4. ရက်စွဲပါ memory ဖိုင်တစ်ခုသို့ session metadata ကို သိမ်းဆည်းသည်

**Example output**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Filename examples**:

- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (slug ဖန်တီးမှု မအောင်မြင်ပါက fallback timestamp)

**Enable**:

```bash
openclaw hooks enable session-memory
```

### command-logger

command အဖြစ်အပျက်များအားလုံးကို အလယ်ဗဟို audit ဖိုင်တစ်ခုသို့ log လုပ်သည်။

**Events**: `command`

**Requirements**: မရှိပါ

**Output**: `~/.openclaw/logs/commands.log`

**What it does**:

1. event အသေးစိတ်များ (command action, timestamp, session key, sender ID, source) ကို ဖမ်းယူသည်
2. JSONL format ဖြင့် log ဖိုင်သို့ ထည့်ပေါင်းရေးသားသည်
3. နောက်ခံတွင် တိတ်တဆိတ် run လုပ်သည်

**Example log entries**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**View logs**:

```bash
# View recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print with jq
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**Enable**:

```bash
openclaw hooks enable command-logger
```

### soul-evil

purge window အတွင်း သို့မဟုတ် ကျပန်းဖြစ်နိုင်ချေဖြင့် injected `SOUL.md` content ကို `SOUL_EVIL.md` ဖြင့် အစားထိုးသည်။

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil Hook](/hooks/soul-evil)

**Output**: ဖိုင်များ မရေးသားပါ; in-memory အတွင်းသာ အစားထိုးလုပ်ဆောင်သည်။

**Enable**:

```bash
openclaw hooks enable soul-evil
```

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

### boot-md

Gateway စတင်ချိန်တွင် (channels များ စတင်ပြီးနောက်) `BOOT.md` ကို လုပ်ဆောင်ပါသည်။
ဤအရာ လည်ပတ်စေရန် internal hooks များကို enable လုပ်ထားရပါမည်။

**Events**: `gateway:startup`

**Requirements**: `workspace.dir` ကို configuration ပြုလုပ်ထားရမည်

**What it does**:

1. workspace ထဲမှ `BOOT.md` ကို ဖတ်ယူသည်
2. agent runner မှတဆင့် အညွှန်းချက်များကို run လုပ်သည်
3. message tool ကို အသုံးပြုပြီး လိုအပ်သော outbound messages များကို ပို့သည်

**Enable**:

```bash
openclaw hooks enable boot-md
```

## Best Practices

### Keep Handlers Fast

Hooks များသည် command processing အတွင်း လည်ပတ်ပါသည်။ ပေါ့ပါးစွာ ထားရှိပါ:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Handle Errors Gracefully

အန္တရာယ်ရှိသော လုပ်ဆောင်ချက်များကို အမြဲ wrap လုပ်ပါ–

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error("[my-handler] Failed:", err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### Filter Events Early

event မသက်ဆိုင်ပါက အစောဆုံး return လုပ်ပါ–

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== "command" || event.action !== "new") {
    return;
  }

  // Your logic here
};
```

### Use Specific Event Keys

metadata တွင် ဖြစ်နိုင်သမျှ တိကျသော event keys များကို သတ်မှတ်ပါ–

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

အောက်ပါအစား–

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Debugging

### Enable Hook Logging

gateway သည် စတင်ချိန်တွင် hook loading ကို log လုပ်သည်–

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Check Discovery

ရှာဖွေတွေ့ရှိထားသော hooks အားလုံးကို စာရင်းပြုစုပါ–

```bash
openclaw hooks list --verbose
```

### Check Registration

handler အတွင်း၊ ခေါ်ယူခံရသည့်အခါ log ထုတ်ပါ–

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

hook တစ်ခု မသင့်လျော်ရခြင်း၏ အကြောင်းရင်းကို စစ်ဆေးပါ–

```bash
openclaw hooks info my-hook
```

output ထဲတွင် မဖြည့်စွက်ရသေးသော လိုအပ်ချက်များကို ကြည့်ရှုပါ။

## Testing

### Gateway Logs

hook execution ကို ကြည့်ရှုရန် gateway logs များကို စောင့်ကြည့်ပါ–

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Test Hooks Directly

handlers များကို သီးခြားစမ်းသပ်ပါ–

```typescript
import { test } from "vitest";
import { createHookEvent } from "./src/hooks/hooks.js";
import myHandler from "./hooks/my-hook/handler.js";

test("my handler works", async () => {
  const event = createHookEvent("command", "new", "test-session", {
    foo: "bar",
  });

  await myHandler(event);

  // Assert side effects
});
```

## Architecture

### Core Components

- **`src/hooks/types.ts`**: Type definitions
- **`src/hooks/workspace.ts`**: Directory scanning နှင့် loading
- **`src/hooks/frontmatter.ts`**: HOOK.md metadata parsing
- **`src/hooks/config.ts`**: Eligibility checking
- **`src/hooks/hooks-status.ts`**: Status reporting
- **`src/hooks/loader.ts`**: Dynamic module loader
- **`src/cli/hooks-cli.ts`**: CLI commands
- **`src/gateway/server-startup.ts`**: gateway စတင်ချိန်တွင် hooks များကို load လုပ်သည်
- **`src/auto-reply/reply/commands-core.ts`**: command events များကို trigger လုပ်သည်

### Discovery Flow

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### Event Flow

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## Troubleshooting

### Hook Not Discovered

1. directory structure ကို စစ်ဆေးပါ–

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. HOOK.md format ကို အတည်ပြုပါ–

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. ရှာဖွေတွေ့ရှိထားသော hooks အားလုံးကို စာရင်းပြုစုပါ–

   ```bash
   openclaw hooks list
   ```

### Hook Not Eligible

လိုအပ်ချက်များကို စစ်ဆေးပါ–

```bash
openclaw hooks info my-hook
```

မရှိနေသော အရာများကို ရှာဖွေပါ–

- Binaries (PATH ကို စစ်ဆေးပါ)
- Environment variables
- Config values
- OS လိုက်ဖက်ညီမှု

### Hook Not Executing

1. hook ကို enable လုပ်ထားကြောင်း အတည်ပြုပါ–

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. hooks များကို reload လုပ်ရန် gateway process ကို restart လုပ်ပါ။

3. error များအတွက် gateway logs ကို စစ်ဆေးပါ–

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler Errors

TypeScript/import error များကို စစ်ဆေးပါ–

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## Migration Guide

### From Legacy Config to Discovery

**Before**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**After**:

1. hook directory တစ်ခု ဖန်တီးပါ–

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md ကို ဖန်တီးပါ–

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. config ကို update လုပ်ပါ–

   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. gateway process ကို verify လုပ်ပြီး restart လုပ်ပါ–

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Migration ၏ အကျိုးကျေးဇူးများ**:

- အလိုအလျောက် ရှာဖွေတွေ့ရှိမှု
- CLI စီမံခန့်ခွဲမှု
- Eligibility checking
- ပိုမိုကောင်းမွန်သော documentation
- တစ်ပြေးညီသော structure

## See Also

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)


