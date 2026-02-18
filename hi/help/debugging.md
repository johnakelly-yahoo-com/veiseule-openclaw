---
title: "डिबगिंग"
---

# डिबगिंग

यह पृष्ठ स्ट्रीमिंग आउटपुट के लिए डिबगिंग सहायक टूल्स को कवर करता है, विशेषकर तब जब
कोई प्रदाता सामान्य टेक्स्ट में तर्क को मिला देता है।

## रनटाइम डिबग ओवरराइड्स

**केवल-रनटाइम** कॉन्फ़िग ओवरराइड (मेमोरी में, डिस्क पर नहीं) सेट करने के लिए चैट में `/debug` का उपयोग करें।
`/debug` is disabled by default; enable with `commands.debug: true`.
This is handy when you need to toggle obscure settings without editing `openclaw.json`.

उदाहरण:

```
/debug show
/debug set messages.responsePrefix="[openclaw]"
/debug unset messages.responsePrefix
/debug reset
```

**`/debug reset`** सभी ओवरराइड्स साफ़ करता है और ऑन-डिस्क कॉन्फ़िग पर लौटता है।

## Gateway वॉच मोड

तेज़ दोहराव के लिए, Gateway को फ़ाइल वॉचर के तहत चलाएँ:

```bash
pnpm gateway:watch --force
```

यह इसके बराबर मैप होता है:

```bash
tsx watch src/entry.ts gateway --force
```

**`gateway:watch`** के बाद कोई भी Gateway CLI फ़्लैग जोड़ें और वे
हर रीस्टार्ट पर पास-थ्रू हो जाएँगे।

## डेव प्रोफ़ाइल + डेव Gateway (--dev)

स्टेट को अलग रखने और एक सुरक्षित, अस्थायी सेटअप शुरू करने के लिए dev प्रोफ़ाइल का उपयोग करें।
debugging. There are **two** `--dev` flags:

- **ग्लोबल `--dev` (प्रोफ़ाइल):** स्थिति को **`~/.openclaw-dev`** के अंतर्गत अलग करता है और
  Gateway पोर्ट को डिफ़ॉल्ट रूप से **`19001`** पर सेट करता है (व्युत्पन्न पोर्ट्स उसी के साथ शिफ्ट होते हैं)।
- **`gateway --dev`: Gateway को डिफ़ॉल्ट कॉन्फ़िग + वर्कस्पेस स्वतः बनाने के लिए बताता है**
  जब वे अनुपस्थित हों (और BOOTSTRAP.md को स्किप करता है)।

अनुशंसित फ़्लो (डेव प्रोफ़ाइल + डेव बूटस्ट्रैप):

```bash
pnpm gateway:dev
OPENCLAW_PROFILE=dev openclaw tui
```

यदि आपके पास अभी ग्लोबल इंस्टॉल नहीं है, तो CLI को **`pnpm openclaw ...`** के माध्यम से चलाएँ।

यह क्या करता है:

1. **प्रोफ़ाइल आइसोलेशन** (ग्लोबल **`--dev`**)
   - **`OPENCLAW_PROFILE=dev`**
   - **`OPENCLAW_STATE_DIR=~/.openclaw-dev`**
   - **`OPENCLAW_CONFIG_PATH=~/.openclaw-dev/openclaw.json`**
   - **`OPENCLAW_GATEWAY_PORT=19001`** (ब्राउज़र/कैनवास उसी अनुसार शिफ्ट होते हैं)

2. **डेव बूटस्ट्रैप** (**`gateway --dev`**)
   - अनुपस्थित होने पर न्यूनतम कॉन्फ़िग लिखता है (**`gateway.mode=local`**, loopback पर bind)।
   - **`agent.workspace`** को डेव वर्कस्पेस पर सेट करता है।
   - **`agent.skipBootstrap=true`** सेट करता है (BOO TSTRAP.md नहीं)।
   - वर्कस्पेस फ़ाइलों को, यदि अनुपस्थित हों, सीड करता है:
     **`AGENTS.md`**, **`SOUL.md`**, **`TOOLS.md`**, **`IDENTITY.md`**, **`USER.md`**, **`HEARTBEAT.md`**।
   - डिफ़ॉल्ट पहचान: **C3‑PO** (प्रोटोकॉल ड्रॉइड)।
   - डेव मोड में चैनल प्रदाताओं को स्किप करता है (**`OPENCLAW_SKIP_CHANNELS=1`**)।

रीसेट फ़्लो (ताज़ा शुरुआत):

```bash
pnpm gateway:dev:reset
```

नोट: `--dev` एक **ग्लोबल** प्रोफ़ाइल फ़्लैग है और कुछ रनर्स द्वारा अनदेखा कर दिया जाता है।
If you need to spell it out, use the env var form:

```bash
OPENCLAW_PROFILE=dev openclaw gateway --dev --reset
```

**`--reset`** कॉन्फ़िग, क्रेडेंशियल्स, सत्रों और डेव वर्कस्पेस को साफ़ करता है (उपयोग करते हुए
**`trash`**, **`rm`** नहीं), फिर डिफ़ॉल्ट डेव सेटअप को पुनः बनाता है।

सुझाव: यदि कोई नॉन‑डेव Gateway पहले से चल रहा हो (launchd/systemd), तो पहले उसे रोकें:

```bash
openclaw gateway stop
```

## रॉ स्ट्रीम लॉगिंग (OpenClaw)

OpenClaw किसी भी फ़िल्टरिंग/फ़ॉर्मेटिंग से पहले **रॉ असिस्टेंट स्ट्रीम** को लॉग कर सकता है।
This is the best way to see whether reasoning is arriving as plain text deltas
(or as separate thinking blocks).

CLI के माध्यम से सक्षम करें:

```bash
pnpm gateway:watch --force --raw-stream
```

वैकल्पिक पाथ ओवरराइड:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.openclaw/logs/raw-stream.jsonl
```

समतुल्य env vars:

```bash
OPENCLAW_RAW_STREAM=1
OPENCLAW_RAW_STREAM_PATH=~/.openclaw/logs/raw-stream.jsonl
```

डिफ़ॉल्ट फ़ाइल:

`~/.openclaw/logs/raw-stream.jsonl`

## रॉ चंक लॉगिंग (pi-mono)

ब्लॉक्स में पार्स होने से पहले **कच्चे OpenAI‑compat चंक्स** कैप्चर करने के लिए,
pi-mono एक अलग लॉगर एक्सपोज़ करता है:

```bash
PI_RAW_STREAM=1
```

वैकल्पिक पाथ:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

डिफ़ॉल्ट फ़ाइल:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> टिप्पणी: यह केवल उन प्रक्रियाओं द्वारा उत्सर्जित होता है जो pi-mono के
> **`openai-completions`** प्रदाता का उपयोग करती हैं।

## सुरक्षा नोट्स

- रॉ स्ट्रीम लॉग्स में पूर्ण प्रॉम्प्ट्स, टूल आउटपुट और उपयोगकर्ता डेटा शामिल हो सकता है।
- लॉग्स को स्थानीय रखें और डिबगिंग के बाद उन्हें हटा दें।
- यदि आप लॉग्स साझा करते हैं, तो पहले सीक्रेट्स और PII को स्क्रब करें।
