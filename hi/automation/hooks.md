---
summary: "Hooks: कमांड और लाइफसाइकल इवेंट्स के लिए इवेंट-ड्रिवन स्वचालन"
read_when:
  - आप /new, /reset, /stop, और एजेंट लाइफसाइकल इवेंट्स के लिए इवेंट-ड्रिवन स्वचालन चाहते हैं
  - आप hooks को बनाना, इंस्टॉल करना, या डिबग करना चाहते हैं
title: "Hooks"
---

# Hooks

हुक्स एक विस्तारयोग्य इवेंट-ड्रिवन सिस्टम प्रदान करते हैं, जो एजेंट कमांड्स और इवेंट्स के जवाब में क्रियाओं को स्वचालित करता है। हुक्स को डायरेक्टरीज़ से अपने आप खोज लिया जाता है और इन्हें CLI कमांड्स के ज़रिए प्रबंधित किया जा सकता है, ठीक वैसे ही जैसे OpenClaw में स्किल्स काम करती हैं।

## Getting Oriented

हुक्स छोटे स्क्रिप्ट होते हैं जो किसी घटना के होने पर चलते हैं। दो प्रकार होते हैं:

- **Hooks** (यह पृष्ठ): Gateway के अंदर चलते हैं जब एजेंट इवेंट्स ट्रिगर होते हैं, जैसे `/new`, `/reset`, `/stop`, या लाइफसाइकल इवेंट्स।
- **वेबहुक्स**: बाहरी HTTP वेबहुक्स जो अन्य सिस्टम्स को OpenClaw में काम ट्रिगर करने देते हैं। [Webhook Hooks](/automation/webhook) देखें या Gmail हेल्पर कमांड्स के लिए `openclaw webhooks` का उपयोग करें।

Hooks को plugins के अंदर भी बंडल किया जा सकता है; देखें [Plugins](/tools/plugin#plugin-hooks)।

सामान्य उपयोग:

- सत्र रीसेट करने पर मेमोरी स्नैपशॉट सहेजना
- समस्या-निवारण या अनुपालन के लिए कमांड्स का ऑडिट ट्रेल रखना
- सत्र शुरू या समाप्त होने पर फॉलो-अप स्वचालन ट्रिगर करना
- इवेंट्स के होने पर एजेंट वर्कस्पेस में फाइलें लिखना या बाहरी APIs कॉल करना

यदि आप एक छोटा TypeScript फ़ंक्शन लिख सकते हैं, तो आप एक हुक लिख सकते हैं। हुक्स अपने आप खोजे जाते हैं, और आप उन्हें CLI के ज़रिए सक्षम या अक्षम करते हैं।

## Overview

Hooks सिस्टम आपको यह करने देता है:

- `/new` जारी होने पर सत्र संदर्भ को मेमोरी में सहेजना
- ऑडिटिंग के लिए सभी कमांड्स को लॉग करना
- एजेंट लाइफसाइकल इवेंट्स पर कस्टम स्वचालन ट्रिगर करना
- कोर कोड में संशोधन किए बिना OpenClaw के व्यवहार का विस्तार करना

## Getting Started

### Bundled Hooks

OpenClaw चार बंडल्ड hooks के साथ आता है जो स्वतः खोजे जाते हैं:

- **💾 session-memory**: जब आप `/new` जारी करते हैं, तो सत्र संदर्भ को आपके एजेंट वर्कस्पेस (डिफ़ॉल्ट `~/.openclaw/workspace/memory/`) में सहेजता है
- **📝 command-logger**: सभी कमांड इवेंट्स को `~/.openclaw/logs/commands.log` में लॉग करता है
- **🚀 boot-md**: Gateway शुरू होने पर `BOOT.md` चलाता है (आंतरिक hooks सक्षम होना आवश्यक)
- **😈 soul-evil**: purge विंडो के दौरान या यादृच्छिक संभावना से injected `SOUL.md` सामग्री को `SOUL_EVIL.md` से बदल देता है

उपलब्ध hooks की सूची देखें:

```bash
openclaw hooks list
```

किसी hook को सक्षम करें:

```bash
openclaw hooks enable session-memory
```

hook की स्थिति जाँचें:

```bash
openclaw hooks check
```

विस्तृत जानकारी प्राप्त करें:

```bash
openclaw hooks info session-memory
```

### Onboarding

ऑनबोर्डिंग (`openclaw onboard`) के दौरान, आपको अनुशंसित हुक्स सक्षम करने के लिए कहा जाएगा। विज़ार्ड अपने आप योग्य हुक्स खोजता है और चयन के लिए उन्हें प्रस्तुत करता है।

## Hook Discovery

Hooks तीन डायरेक्टरीज़ से स्वतः खोजे जाते हैं (प्राथमिकता क्रम में):

1. **Workspace hooks**: `<workspace>/hooks/` (प्रति-एजेंट, सर्वोच्च प्राथमिकता)
2. **Managed hooks**: `~/.openclaw/hooks/` (उपयोगकर्ता-इंस्टॉल्ड, वर्कस्पेसेज़ में साझा)
3. **Bundled hooks**: `<openclaw>/dist/hooks/bundled/` (OpenClaw के साथ वितरित)

Managed hook डायरेक्टरीज़ या तो एक **single hook** हो सकती हैं या एक **hook pack** (पैकेज डायरेक्टरी)।

प्रत्येक hook एक डायरेक्टरी होता है जिसमें शामिल है:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## Hook Packs (npm/archives)

हुक पैक्स मानक npm पैकेज होते हैं जो `package.json` में `openclaw.hooks` के माध्यम से एक या अधिक हुक्स एक्सपोर्ट करते हैं। इन्हें इस तरह इंस्टॉल करें:

```bash
openclaw hooks install <path-or-spec>
```

Npm स्पेसिफिकेशन केवल रजिस्ट्री-आधारित होते हैं (पैकेज नाम + वैकल्पिक वर्ज़न/टैग)। Git/URL/file स्पेसिफिकेशन अस्वीकार किए जाते हैं।

उदाहरण `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "openclaw": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

प्रत्येक एंट्री एक हुक डायरेक्टरी की ओर इशारा करती है जिसमें `HOOK.md` और `handler.ts` (या `index.ts`) होता है।
हुक पैक्स डिपेंडेंसीज़ के साथ आ सकते हैं; इन्हें `~/.openclaw/hooks/<id>` के अंतर्गत इंस्टॉल किया जाएगा।

सुरक्षा नोट: `openclaw hooks install` निर्भरताओं को `npm install --ignore-scripts` के साथ इंस्टॉल करता है
(कोई lifecycle scripts नहीं)। हुक पैक की निर्भरता ट्री को "pure JS/TS" रखें और उन पैकेजों से बचें जो `postinstall` बिल्ड पर निर्भर करते हैं।

## Hook Structure

### HOOK.md Format

`HOOK.md` फ़ाइल में YAML frontmatter में मेटाडेटा और साथ में Markdown दस्तावेज़ीकरण होता है:

```markdown
---
name: my-hook
description: "यह हुक क्या करता है इसका संक्षिप्त विवरण"
homepage: https://docs.openclaw.ai/automation/hooks#my-hook
metadata:
  { "openclaw": { "emoji": "🔗", "events": ["command:new"], "requires": { "bins": ["node"] } } }
---

# My Hook

विस्तृत दस्तावेज़ यहाँ जाएँगे...

## यह क्या करता है

- `/new` कमांड्स को सुनता है
- कोई क्रिया करता है
- परिणाम को लॉग करता है

## आवश्यकताएँ

- Node.js इंस्टॉल होना चाहिए

## कॉन्फ़िगरेशन

कोई कॉन्फ़िगरेशन आवश्यक नहीं है।
```

### Metadata Fields

`metadata.openclaw` ऑब्जेक्ट निम्न का समर्थन करता है:

- **`emoji`**: CLI के लिए डिस्प्ले इमोजी (उदा., `"💾"`)
- **`events`**: सुनने के लिए इवेंट्स की array (उदा., `["command:new", "command:reset"]`)
- **`export`**: उपयोग करने के लिए named export (डिफ़ॉल्ट `"default"`)
- **`homepage`**: दस्तावेज़ीकरण URL
- **`requires`**: वैकल्पिक आवश्यकताएँ
  - **`bins`**: PATH पर आवश्यक binaries (उदा., `["git", "node"]`)
  - **`anyBins`**: इनमें से कम से कम एक binary मौजूद होनी चाहिए
  - **`env`**: आवश्यक environment variables
  - **`config`**: आवश्यक config paths (उदा., `["workspace.dir"]`)
  - **`os`**: आवश्यक प्लेटफ़ॉर्म्स (उदा., `["darwin", "linux"]`)
- **`always`**: पात्रता जाँच को बायपास करें (boolean)
- **`install`**: इंस्टॉलेशन विधियाँ (बंडल्ड hooks के लिए: `[{"id":"bundled","kind":"bundled"}]`)

### Handler Implementation

`handler.ts` फ़ाइल एक `HookHandler` फ़ंक्शन एक्सपोर्ट करती है:

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

प्रत्येक इवेंट में शामिल होता है:

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

एजेंट कमांड्स जारी होने पर ट्रिगर होते हैं:

- **`agent:bootstrap`**: वर्कस्पेस bootstrap फाइलें inject होने से पहले (hooks `context.bootstrapFiles` को mutate कर सकते हैं)
- **`command:new`**: जब `/new` कमांड जारी की जाती है
- **`command:reset`**: जब `/reset` कमांड जारी की जाती है
- **`command:stop`**: जब `/stop` कमांड जारी की जाती है

### Agent Events

- **`agent:bootstrap`**: वर्कस्पेस bootstrap फाइलें inject होने से पहले (hooks `context.bootstrapFiles` को mutate कर सकते हैं)

### Gateway Events

Gateway के शुरू होने पर ट्रिगर होते हैं:

- **`gateway:startup`**: चैनल्स शुरू होने और hooks लोड होने के बाद

### Tool Result Hooks (Plugin API)

ये hooks इवेंट-स्ट्रीम लिसनर्स नहीं होते; ये plugins को OpenClaw द्वारा persist किए जाने से पहले टूल परिणामों को synchronously समायोजित करने देते हैं।

- **`tool_result_persist`**: टूल परिणामों को सेशन ट्रांसक्रिप्ट में लिखे जाने से पहले रूपांतरित करें। सिंक्रोनस होना चाहिए; अपडेट किया गया टूल परिणाम पेलोड लौटाएँ या जैसा है वैसा रखने के लिए `undefined` लौटाएँ। [Agent Loop](/concepts/agent-loop) देखें।

### Future Events

योजना किए गए इवेंट प्रकार:

- **`session:start`**: जब एक नया सत्र शुरू होता है
- **`session:end`**: जब एक सत्र समाप्त होता है
- **`agent:error`**: जब कोई एजेंट त्रुटि का सामना करता है
- **`message:sent`**: जब कोई संदेश भेजा जाता है
- **`message:received`**: जब कोई संदेश प्राप्त होता है

## Creating Custom Hooks

### 1. स्थान चुनें

- **Workspace hooks** (`<workspace>/hooks/`): प्रति-एजेंट, सर्वोच्च प्राथमिकता
- **Managed hooks** (`~/.openclaw/hooks/`): वर्कस्पेसेज़ में साझा

### 2. डायरेक्टरी संरचना बनाएँ

```bash
mkdir -p ~/.openclaw/hooks/my-hook
cd ~/.openclaw/hooks/my-hook
```

### 3. HOOK.md बनाएँ

```markdown
---
name: my-hook
description: "Does something useful"
metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. handler.ts बनाएँ

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

### 5. सक्षम करें और परीक्षण करें

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

Hooks के पास कस्टम विन्यास हो सकता है:

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

अतिरिक्त डायरेक्टरीज़ से hooks लोड करें:

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

**माइग्रेशन**: नए हुक्स के लिए नए डिस्कवरी-आधारित सिस्टम का उपयोग करें। लीगेसी हैंडलर्स को डायरेक्टरी-आधारित हुक्स के बाद लोड किया जाता है।

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

नोट: `module` एक workspace-सापेक्ष पथ होना चाहिए। workspace के बाहर के absolute पथ और traversal अस्वीकार किए जाते हैं।

**माइग्रेशन**: नए हुक्स के लिए नए डिस्कवरी-आधारित सिस्टम का उपयोग करें। लीगेसी हैंडलर्स को डायरेक्टरी-आधारित हुक्स के बाद लोड किया जाता है।

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

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (डिफ़ॉल्ट `~/.openclaw/workspace`)

**Events**: `command:new`

**Requirements**: `workspace.dir` कॉन्फ़िगर होना चाहिए

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (डिफ़ॉल्ट `~/.openclaw/workspace`)

**What it does**:

1. सही ट्रांसक्रिप्ट खोजने के लिए pre-reset सत्र प्रविष्टि का उपयोग करता है
2. बातचीत की अंतिम 15 पंक्तियाँ निकालता है
3. वर्णनात्मक फ़ाइलनाम slug जनरेट करने के लिए LLM का उपयोग करता है
4. दिनांकित मेमोरी फ़ाइल में सत्र मेटाडेटा सहेजता है

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
- `2026-01-16-1430.md` (यदि slug जनरेशन विफल हो जाए तो fallback टाइमस्टैम्प)

**Enable**:

```bash
openclaw hooks enable session-memory
```

### bootstrap-extra-files

`agent:bootstrap` के दौरान अतिरिक्त bootstrap फ़ाइलें (उदाहरण के लिए monorepo-स्थानीय `AGENTS.md` / `TOOLS.md`) इंजेक्ट करता है।

**Events**: `agent:bootstrap`

**Requirements**: `workspace.dir` कॉन्फ़िगर होना चाहिए

**आउटपुट**: कोई फ़ाइल नहीं लिखी जाती; bootstrap कॉन्टेक्स्ट केवल इन-मेमोरी संशोधित होता है।

**Config**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "bootstrap-extra-files": {
          "enabled": true,
          "paths": ["packages/*/AGENTS.md", "packages/*/TOOLS.md"]
        }
      }
    }
  }
}
```

**नोट्स**:

- पथ workspace के सापेक्ष resolve किए जाते हैं।
- फ़ाइलें workspace के अंदर ही रहनी चाहिए (realpath-checked)।
- केवल मान्यता प्राप्त bootstrap basenames लोड किए जाते हैं।
- Subagent allowlist संरक्षित रहती है (`AGENTS.md` और `TOOLS.md` ही)।

**Enable**:

```bash
openclaw hooks enable bootstrap-extra-files
```

### command-logger

सभी कमांड इवेंट्स को एक केंद्रीकृत ऑडिट फ़ाइल में लॉग करता है।

**Events**: `command`

**Output**: कोई फ़ाइल नहीं लिखी जाती; swapping केवल मेमोरी में होती है।

**Output**: `~/.openclaw/logs/commands.log`

**What it does**:

1. इवेंट विवरण कैप्चर करता है (command action, timestamp, session key, sender ID, source)
2. JSONL फ़ॉर्मेट में लॉग फ़ाइल में append करता है
3. बैकग्राउंड में चुपचाप चलता है

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

### boot-md

Gateway के शुरू होने पर (चैनल्स शुरू होने के बाद) `BOOT.md` चलाता है।
इसे चलाने के लिए आंतरिक हुक्स सक्षम होने चाहिए।

**Events**: `gateway:startup`

**आवश्यकताएँ**: `workspace.dir` कॉन्फ़िगर होना चाहिए

**What it does**:

1. आपके वर्कस्पेस से `BOOT.md` पढ़ता है
2. एजेंट रनर के माध्यम से निर्देश चलाता है
3. message टूल के माध्यम से अनुरोधित outbound संदेश भेजता है

**Enable**:

```bash
openclaw hooks enable boot-md
```

## Best Practices

### Keep Handlers Fast

हुक्स कमांड प्रोसेसिंग के दौरान चलते हैं। उन्हें हल्का रखें:

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

हमेशा जोखिम भरे ऑपरेशन्स को wrap करें:

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

इसके बजाय:

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

जहाँ संभव हो, मेटाडेटा में सटीक इवेंट्स निर्दिष्ट करें:

```yaml
metadata: { "openclaw": { "events": ["command:new"] } } # Specific
```

इसके बजाय:

```yaml
metadata: { "openclaw": { "events": ["command"] } } # General - more overhead
```

## Debugging

### Enable Hook Logging

Gateway स्टार्टअप पर hook लोडिंग को लॉग करता है:

```
Registered hook: session-memory -> command:new
Registered hook: bootstrap-extra-files -> agent:bootstrap
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Check Discovery

सभी खोजे गए hooks की सूची देखें:

```bash
openclaw hooks list --verbose
```

### Check Registration

आउटपुट में missing requirements देखें।

```typescript
const handler: HookHandler = async (event) => {
  console.log("[my-handler] Triggered:", event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

Hook निष्पादन देखने के लिए Gateway logs मॉनिटर करें:

```bash
openclaw hooks info my-hook
```

आउटपुट में missing requirements देखें।

## Testing

### Gateway Logs

Hook निष्पादन देखने के लिए Gateway logs मॉनिटर करें:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.openclaw/gateway.log
```

### Test Hooks Directly

अपने handlers को अलग-थलग करके परीक्षण करें:

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

- **`src/hooks/types.ts`**: Type परिभाषाएँ
- **`src/hooks/workspace.ts`**: डायरेक्टरी स्कैनिंग और लोडिंग
- **`src/hooks/frontmatter.ts`**: HOOK.md मेटाडेटा पार्सिंग
- **`src/hooks/config.ts`**: पात्रता जाँच
- **`src/hooks/hooks-status.ts`**: स्थिति रिपोर्टिंग
- **`src/hooks/loader.ts`**: डायनेमिक मॉड्यूल लोडर
- **`src/cli/hooks-cli.ts`**: CLI कमांड्स
- **`src/gateway/server-startup.ts`**: Gateway स्टार्ट पर hooks लोड करता है
- **`src/auto-reply/reply/commands-core.ts`**: कमांड इवेंट्स ट्रिगर करता है

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

1. Binaries (PATH जाँचें)

   ```bash
   ls -la ~/.openclaw/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. HOOK.md फ़ॉर्मेट सत्यापित करें:

   ```bash
   cat ~/.openclaw/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. Config मान

   ```bash
   openclaw hooks list
   ```

### Hook Not Eligible

आवश्यकताएँ जाँचें:

```bash
openclaw hooks info my-hook
```

TypeScript/import त्रुटियों की जाँच करें:

- Binaries (PATH जाँचें)
- Environment variables
- Config मान
- OS संगतता

### Hook Not Executing

1. सुनिश्चित करें कि hook सक्षम है:

   ```bash
   openclaw hooks list
   # Should show ✓ next to enabled hooks
   ```

2. hooks के पुनः लोड होने के लिए अपना Gateway प्रोसेस पुनः आरंभ करें।

3. त्रुटियों के लिए Gateway logs जाँचें:

   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### Handler Errors

TypeScript/import त्रुटियों की जाँच करें:

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

1. hook डायरेक्टरी बनाएँ:

   ```bash
   mkdir -p ~/.openclaw/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.openclaw/hooks/my-hook/handler.ts
   ```

2. HOOK.md बनाएँ:

   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: { "openclaw": { "emoji": "🎯", "events": ["command:new"] } }
   ---

   # My Hook

   Does something useful.
   ```

3. config अपडेट करें:

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

4. सत्यापित करें और अपना Gateway प्रोसेस पुनः आरंभ करें:

   ```bash
   openclaw hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Benefits of migration**:

- स्वतः discovery
- CLI प्रबंधन
- पात्रता जाँच
- बेहतर दस्तावेज़ीकरण
- सुसंगत संरचना

## See Also

- [CLI Reference: hooks](/cli/hooks)
- [Bundled Hooks README](https://github.com/openclaw/openclaw/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [Configuration](/gateway/configuration#hooks)

