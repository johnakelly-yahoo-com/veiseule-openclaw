---
title: "लॉगिंग"
---

# लॉगिंग

उपयोगकर्ता-उन्मुख अवलोकन (CLI + Control UI + विन्यास) के लिए, देखें [/logging](/logging)।

OpenClaw में दो लॉग “सतहें” हैं:

- **कंसोल आउटपुट** (जो आप टर्मिनल / Debug UI में देखते हैं)।
- **फ़ाइल लॉग** (JSON लाइन्स), जिन्हें Gateway लॉगर द्वारा लिखा जाता है।

## फ़ाइल-आधारित लॉगर

- डिफ़ॉल्ट रोलिंग लॉग फ़ाइल `/tmp/openclaw/` के अंतर्गत होती है (प्रति दिन एक फ़ाइल): `openclaw-YYYY-MM-DD.log`
  - तारीख Gateway होस्ट के स्थानीय टाइमज़ोन का उपयोग करती है।
- लॉग फ़ाइल पथ और स्तर `~/.openclaw/openclaw.json` के माध्यम से विन्यस्त किए जा सकते हैं:
  - `logging.file`
  - `logging.level`

फ़ाइल फ़ॉर्मैट प्रति पंक्ति एक JSON ऑब्जेक्ट होता है।

37. Control UI का Logs टैब gateway के माध्यम से इस फ़ाइल को tail करता है (`logs.tail`)।
38. CLI भी वही कर सकता है:

```bash
openclaw logs --follow
```

**Verbose बनाम लॉग स्तर**

- **फ़ाइल लॉग** केवल `logging.level` द्वारा नियंत्रित होते हैं।
- `--verbose` केवल **कंसोल verbosity** (और WS लॉग शैली) को प्रभावित करता है; यह फ़ाइल लॉग स्तर **नहीं** बढ़ाता।
- verbose-केवल विवरण फ़ाइल लॉग में कैप्चर करने के लिए, `logging.level` को `debug` या `trace` पर सेट करें।

## कंसोल कैप्चर

CLI `console.log/info/warn/error/debug/trace` को कैप्चर करता है और उन्हें फ़ाइल लॉग में लिखता है,
जबकि stdout/stderr पर प्रिंट करना जारी रखता है।

आप कंसोल verbosity को स्वतंत्र रूप से ट्यून कर सकते हैं:

- `logging.consoleLevel` (डिफ़ॉल्ट `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## टूल सारांश रिडैक्शन

39. Verbose tool summaries (जैसे `🛠️ Exec: ...`) console stream पर आने से पहले संवेदनशील tokens को mask कर सकते हैं। 40. यह **केवल tools** के लिए है और फ़ाइल logs को नहीं बदलता।

- `logging.redactSensitive`: `off` | `tools` (डिफ़ॉल्ट: `tools`)
- `logging.redactPatterns`: regex स्ट्रिंग्स की array (डिफ़ॉल्ट्स को ओवरराइड करती है)
  - raw regex स्ट्रिंग्स का उपयोग करें (auto `gi`), या यदि कस्टम फ़्लैग्स चाहिए हों तो `/pattern/flags`।
  - मैच होने पर पहले 6 + आख़िरी 4 अक्षर रखकर मास्क किया जाता है (लंबाई >= 18), अन्यथा `***`।
  - डिफ़ॉल्ट्स सामान्य key असाइनमेंट्स, CLI फ़्लैग्स, JSON फ़ील्ड्स, bearer हेडर्स, PEM ब्लॉक्स, और लोकप्रिय टोकन प्रीफ़िक्स को कवर करते हैं।

## Gateway WebSocket लॉग

Gateway WebSocket प्रोटोकॉल लॉग को दो मोड में प्रिंट करता है:

- **Normal मोड (बिना `--verbose`)**: केवल “दिलचस्प” RPC परिणाम प्रिंट होते हैं:
  - त्रुटियाँ (`ok=false`)
  - धीमी कॉल्स (डिफ़ॉल्ट थ्रेशहोल्ड: `>= 50ms`)
  - parse त्रुटियाँ
- **Verbose मोड (`--verbose`)**: सभी WS अनुरोध/प्रतिक्रिया ट्रैफ़िक प्रिंट करता है।

### WS लॉग शैली

`openclaw gateway` प्रति-Gateway शैली स्विच का समर्थन करता है:

- `--ws-log auto` (डिफ़ॉल्ट): normal मोड अनुकूलित रहता है; verbose मोड कॉम्पैक्ट आउटपुट उपयोग करता है
- `--ws-log compact`: verbose होने पर कॉम्पैक्ट आउटपुट (जोड़ीदार अनुरोध/प्रतिक्रिया)
- `--ws-log full`: verbose होने पर पूर्ण प्रति-फ़्रेम आउटपुट
- `--compact`: `--ws-log compact` के लिए उपनाम

उदाहरण:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## कंसोल फ़ॉर्मैटिंग (सब-सिस्टम लॉगिंग)

41. console formatter **TTY-aware** है और एकसमान, prefixed lines प्रिंट करता है।
42. Subsystem loggers आउटपुट को समूहबद्ध और स्कैन करने योग्य रखते हैं।

व्यवहार:

- 43. **Subsystem prefixes** हर लाइन पर (जैसे `[gateway]`, `[canvas]`, `[tailscale]`)
- **सब-सिस्टम रंग** (प्रति सब-सिस्टम स्थिर) साथ में स्तर रंग
- **जब आउटपुट TTY हो या परिवेश एक रिच टर्मिनल जैसा दिखे तब रंग** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), `NO_COLOR` का सम्मान करता है
- 44. **Shortened subsystem prefixes**: शुरुआती `gateway/` + `channels/` हटा देता है, आख़िरी 2 segments रखता है (जैसे `whatsapp/outbound`)
- **सब-सिस्टम द्वारा सब-लॉगर्स** (auto प्रीफ़िक्स + संरचित फ़ील्ड `{ subsystem }`)
- **QR/UX आउटपुट के लिए `logRaw()`** (कोई प्रीफ़िक्स नहीं, कोई फ़ॉर्मैटिंग नहीं)
- 45. **Console styles** (जैसे `pretty | compact | json`)
- **कंसोल लॉग स्तर** फ़ाइल लॉग स्तर से अलग (जब `logging.level` को `debug`/`trace` पर सेट किया जाता है, फ़ाइल पूरी जानकारी रखती है)
- **WhatsApp संदेश बॉडीज़** `debug` पर लॉग की जाती हैं (उन्हें देखने के लिए `--verbose` का उपयोग करें)

यह मौजूदा फ़ाइल लॉग्स को स्थिर रखते हुए इंटरैक्टिव आउटपुट को स्कैन करने योग्य बनाता है।
