---
summary: "`openclaw hooks` (एजेंट हुक्स) के लिए CLI संदर्भ"
read_when:
  - आप एजेंट हुक्स का प्रबंधन करना चाहते हैं
  - आप हुक्स को इंस्टॉल या अपडेट करना चाहते हैं
title: "hooks"
---

# `openclaw hooks`

एजेंट हुक्स का प्रबंधन करें (घटना-आधारित स्वचालन, जैसे `/new`, `/reset`, और Gateway स्टार्टअप के लिए कमांड्स)।

संबंधित:

- हुक्स: [हुक्स](/automation/hooks)
- प्लगइन हुक्स: [प्लगइन्स](/tools/plugin#plugin-hooks)

## सभी हुक्स की सूची

```bash
openclaw hooks list
```

वर्कस्पेस, प्रबंधित, और बंडल्ड निर्देशिकाओं से खोजे गए सभी हुक्स की सूची दिखाएँ।

**विकल्प:**

- `--eligible`: केवल योग्य हुक्स दिखाएँ (आवश्यकताएँ पूरी)
- `--json`: JSON के रूप में आउटपुट
- `-v, --verbose`: गायब आवश्यकताओं सहित विस्तृत जानकारी दिखाएँ

**उदाहरण आउटपुट:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**उदाहरण (वर्बोज़):**

```bash
openclaw hooks list --verbose
```

अयोग्य हुक्स के लिए गायब आवश्यकताएँ दिखाता है।

**उदाहरण (JSON):**

```bash
openclaw hooks list --json
```

प्रोग्रामेटिक उपयोग के लिए संरचित JSON लौटाता है।

## हुक जानकारी प्राप्त करें

```bash
openclaw hooks info <name>
```

किसी विशिष्ट हुक के बारे में विस्तृत जानकारी दिखाएँ।

**आर्ग्युमेंट्स:**

- `<name>`: हुक नाम (उदा., `session-memory`)

**विकल्प:**

- `--json`: JSON के रूप में आउटपुट

**उदाहरण:**

```bash
openclaw hooks info session-memory
```

**आउटपुट:**

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

## हुक्स की पात्रता जाँचें

```bash
openclaw hooks check
```

हुक पात्रता स्थिति का सार दिखाएँ (कितने तैयार हैं बनाम कितने नहीं)।

**विकल्प:**

- `--json`: JSON के रूप में आउटपुट

**उदाहरण आउटपुट:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## किसी हुक को सक्षम करें

```bash
openclaw hooks enable <name>
```

अपने विन्यास (`~/.openclaw/config.json`) में जोड़कर किसी विशिष्ट हुक को सक्षम करें।

**Note:** Hooks managed by plugins show `plugin:<id>` in `openclaw hooks list` and
can’t be enabled/disabled here. Enable/disable the plugin instead.

**आर्ग्युमेंट्स:**

- `<name>`: हुक नाम (उदा., `session-memory`)

**उदाहरण:**

```bash
openclaw hooks enable session-memory
```

**आउटपुट:**

```
✓ Enabled hook: 💾 session-memory
```

**यह क्या करता है:**

- जाँचता है कि हुक मौजूद है और योग्य है
- Updates `hooks.internal.entries.<name>.enabled = true` in your config
- विन्यास को डिस्क पर सहेजता है

**सक्षम करने के बाद:**

- हुक्स को पुनः लोड करने के लिए Gateway को पुनः प्रारंभ करें (macOS पर मेनू बार ऐप रीस्टार्ट करें, या dev में अपने Gateway प्रोसेस को रीस्टार्ट करें)।

## किसी हुक को अक्षम करें

```bash
openclaw hooks disable <name>
```

अपने विन्यास को अपडेट करके किसी विशिष्ट हुक को अक्षम करें।

**आर्ग्युमेंट्स:**

- `<name>`: हुक नाम (उदा., `command-logger`)

**उदाहरण:**

```bash
openclaw hooks disable command-logger
```

**आउटपुट:**

```
⏸ Disabled hook: 📝 command-logger
```

**अक्षम करने के बाद:**

- हुक्स को पुनः लोड करने के लिए Gateway को पुनः प्रारंभ करें

## हुक्स इंस्टॉल करें

```bash
openclaw hooks install <path-or-spec>
```

स्थानीय फ़ोल्डर/आर्काइव या npm से एक हुक पैक इंस्टॉल करें।

Npm specs केवल **registry-only** हैं (package name + वैकल्पिक version/tag)। Git/URL/file
specs अस्वीकृत हैं। सुरक्षा के लिए dependency installs `--ignore-scripts` के साथ चलाए जाते हैं।

**यह क्या करता है:**

- हुक पैक को `~/.openclaw/hooks/<id>` में कॉपी करता है
- इंस्टॉल किए गए हुक्स को `hooks.internal.entries.*` में सक्षम करता है
- इंस्टॉल को `hooks.internal.installs` के अंतर्गत रिकॉर्ड करता है

**विकल्प:**

- `-l, --link`: कॉपी करने के बजाय किसी स्थानीय डायरेक्टरी को लिंक करें (इसे `hooks.internal.load.extraDirs` में जोड़ता है)

**उदाहरण:**

**उदाहरण:**

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

## हुक्स अपडेट करें

```bash
openclaw hooks update <id>
openclaw hooks update --all
```

**विकल्प:**

**विकल्प:**

- `--all`: सभी ट्रैक किए गए हुक पैक्स अपडेट करें
- `--dry-run`: लिखे बिना दिखाएँ कि क्या बदलेगा

## बंडल्ड हुक्स

### session-memory

**सक्षम करें:**

**सक्षम करें:**

```bash
openclaw hooks enable session-memory
```

**देखें:** [session-memory प्रलेखन](/automation/hooks#session-memory)

**देखें:** [session-memory प्रलेखन](/automation/hooks#session-memory)

### bootstrap-extra-files

**सक्षम करें:**

**सक्षम करें:**

```bash
openclaw hooks bootstrap-extra-files को सक्षम करते हैं
```

**लॉग्स देखें:**

### command-logger

**देखें:** [command-logger प्रलेखन](/automation/hooks#command-logger)

**सक्षम करें:**

```bash
openclaw hooks enable command-logger
```

**सक्षम करें:**

**लॉग्स देखें:**

```bash
# Recent commands
tail -n 20 ~/.openclaw/logs/commands.log

# Pretty-print
cat ~/.openclaw/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.openclaw/logs/commands.log | jq .
```

**देखें:** [command-logger प्रलेखन](/automation/hooks#command-logger)

### boot-md

**इवेंट्स**: `gateway:startup`

**सक्षम करें**:

**सक्षम करें**:

```bash
openclaw hooks enable boot-md
```

**देखें:** [boot-md प्रलेखन](/automation/hooks#boot-md)

