---
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

**नोट:** प्लगइन्स द्वारा प्रबंधित हुक्स `openclaw hooks list` में `plugin:<id>` के रूप में दिखते हैं और
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
- आपकी config में `hooks.internal.entries.<name>.enabled = true` को अपडेट करता है
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

**यह क्या करता है:**

- हुक पैक को `~/.openclaw/hooks/<id>` में कॉपी करता है
- इंस्टॉल किए गए हुक्स को `hooks.internal.entries.*` में सक्षम करता है
- इंस्टॉल को `hooks.internal.installs` के अंतर्गत रिकॉर्ड करता है

**विकल्प:**

- `-l, --link`: कॉपी करने के बजाय किसी स्थानीय डायरेक्टरी को लिंक करें (इसे `hooks.internal.load.extraDirs` में जोड़ता है)

**समर्थित आर्काइव्स:** `.zip`, `.tgz`, `.tar.gz`, `.tar`

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

इंस्टॉल किए गए हुक पैक्स को अपडेट करें (केवल npm इंस्टॉल्स)।

**विकल्प:**

- `--all`: सभी ट्रैक किए गए हुक पैक्स अपडेट करें
- `--dry-run`: लिखे बिना दिखाएँ कि क्या बदलेगा

## बंडल्ड हुक्स

### session-memory

जब आप `/new` जारी करते हैं, तब सत्र संदर्भ को मेमोरी में सहेजता है।

**सक्षम करें:**

```bash
openclaw hooks enable session-memory
```

**आउटपुट:** `~/.openclaw/workspace/memory/YYYY-MM-DD-slug.md`

**देखें:** [session-memory प्रलेखन](/automation/hooks#session-memory)

### command-logger

सभी कमांड इवेंट्स को एक केंद्रीकृत ऑडिट फ़ाइल में लॉग करता है।

**सक्षम करें:**

```bash
openclaw hooks enable command-logger
```

**आउटपुट:** `~/.openclaw/logs/commands.log`

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

### soul-evil

पर्ज विंडो के दौरान या यादृच्छिक संभावना से इंजेक्ट किए गए `SOUL.md` कंटेंट को `SOUL_EVIL.md` से बदल देता है।

**सक्षम करें:**

```bash
openclaw hooks enable soul-evil
```

**देखें:** [SOUL Evil Hook](/hooks/soul-evil)

### boot-md

Gateway के शुरू होने पर (चैनल्स शुरू होने के बाद) `BOOT.md` चलाता है।

**इवेंट्स**: `gateway:startup`

**सक्षम करें**:

```bash
openclaw hooks enable boot-md
```

**देखें:** [boot-md प्रलेखन](/automation/hooks#boot-md)
