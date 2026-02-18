---
title: "संदर्भ"
---

# संदर्भ

34. “Context” वह **सब कुछ है जो OpenClaw किसी run के लिए model को भेजता है**। 35. यह model की **context window** (token limit) द्वारा सीमित होता है।

शुरुआती मानसिक मॉडल:

- **System prompt** (OpenClaw-निर्मित): नियम, टूल्स, Skills सूची, समय/रनटाइम, और इंजेक्ट की गई वर्कस्पेस फ़ाइलें।
- **Conversation history**: इस सत्र के लिए आपके संदेश + सहायक के संदेश।
- **Tool calls/results + attachments**: कमांड आउटपुट, फ़ाइल रीड्स, इमेज/ऑडियो, आदि।

संदर्भ “memory” के समान _नहीं_ है: memory को डिस्क पर संग्रहीत करके बाद में पुनः लोड किया जा सकता है; संदर्भ वह है जो मॉडल की वर्तमान विंडो के भीतर होता है।

## त्वरित प्रारंभ (संदर्भ निरीक्षण)

- 36. `/status` → जल्दी से “मेरी window कितनी भरी है?” view + session settings।
- `/context list` → क्या इंजेक्ट किया गया है + अनुमानित आकार (प्रति फ़ाइल + कुल)।
- `/context detail` → गहन विभाजन: प्रति-फ़ाइल, प्रति-टूल स्कीमा आकार, प्रति-Skill प्रविष्टि आकार, और system prompt का आकार।
- `/usage tokens` → सामान्य उत्तरों में प्रति-उत्तर उपयोग फ़ुटर जोड़ें।
- `/compact` → विंडो स्थान मुक्त करने के लिए पुराने इतिहास को एक कॉम्पैक्ट प्रविष्टि में संक्षेपित करें।

यह भी देखें: [Slash commands](/tools/slash-commands), [Token use & costs](/reference/token-use), [Compaction](/concepts/compaction).

## उदाहरण आउटपुट

मान मॉडल, प्रदाता, टूल नीति, और आपकी वर्कस्पेस सामग्री के अनुसार बदलते हैं।

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## context window में क्या गिना जाता है

मॉडल को प्राप्त होने वाली हर चीज़ गिनी जाती है, जिसमें शामिल हैं:

- System prompt (सभी अनुभाग)।
- Conversation history।
- Tool calls + tool results।
- Attachments/transcripts (इमेज/ऑडियो/फ़ाइलें)।
- Compaction summaries और pruning artifacts।
- प्रदाता के “wrappers” या छिपे हुए हेडर्स (दिखाई नहीं देते, फिर भी गिने जाते हैं)।

## OpenClaw system prompt कैसे बनाता है

37. System prompt **OpenClaw-owned** होता है और हर run में दोबारा बनाया जाता है। 38. इसमें शामिल है:

- टूल सूची + संक्षिप्त विवरण।
- Skills सूची (केवल मेटाडेटा; नीचे देखें)।
- Workspace स्थान।
- समय (UTC + कॉन्फ़िगर होने पर उपयोगकर्ता समय में रूपांतरण)।
- Runtime मेटाडेटा (होस्ट/OS/मॉडल/थिंकिंग)।
- **Project Context** के अंतर्गत इंजेक्ट की गई वर्कस्पेस बूटस्ट्रैप फ़ाइलें।

पूर्ण विवरण: [System Prompt](/concepts/system-prompt).

## इंजेक्ट की गई वर्कस्पेस फ़ाइलें (प्रोजेक्ट संदर्भ)

डिफ़ॉल्ट रूप से, OpenClaw वर्कस्पेस फ़ाइलों का एक निश्चित सेट इंजेक्ट करता है (यदि मौजूद हों):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (केवल पहली बार)

39. बड़े files को per-file `agents.defaults.bootstrapMaxChars` (default `20000` chars) का उपयोग करके truncate किया जाता है। 40. `/context` **raw बनाम injected** sizes और क्या truncation हुआ, यह दिखाता है।

## Skills: क्या इंजेक्ट होता है बनाम ऑन-डिमांड लोड

41. System prompt में एक compact **skills list** (name + description + location) शामिल होती है। 42. इस list का वास्तविक overhead होता है।

43. Skill instructions default रूप से शामिल **नहीं** होतीं। 44. Model से अपेक्षा की जाती है कि वह skill की `SKILL.md` को **केवल आवश्यकता होने पर** `read` करे।

## Tools: दो प्रकार की लागतें होती हैं

Tools संदर्भ को दो तरीकों से प्रभावित करते हैं:

1. system prompt में **Tool list टेक्स्ट** (जो आप “Tooling” के रूप में देखते हैं)।
2. 45. **Tool schemas** (JSON)। 46. इन्हें model को भेजा जाता है ताकि वह tools call कर सके। 47. ये context में गिने जाते हैं, भले ही आप उन्हें plain text के रूप में न देखें।

`/context detail` सबसे बड़े tool schemas का विवरण देता है ताकि आप देख सकें कि क्या प्रमुख है।

## Commands, directives, और “inline shortcuts”

48. Slash commands Gateway द्वारा handle किए जाते हैं। 49. कुछ अलग-अलग behaviors हैं:

- **Standalone commands**: ऐसा संदेश जो केवल `/...` हो, कमांड के रूप में चलता है।
- **Directives**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` मॉडल द्वारा संदेश देखने से पहले हटा दिए जाते हैं।
  - केवल-Directive वाले संदेश सत्र सेटिंग्स को बनाए रखते हैं।
  - सामान्य संदेश में inline directives प्रति-संदेश संकेत के रूप में कार्य करते हैं।
- **Inline shortcuts** (केवल allowlisted प्रेषक): सामान्य संदेश के भीतर कुछ `/...` टोकन तुरंत चल सकते हैं (उदाहरण: “hey /status”), और शेष टेक्स्ट देखने से पहले हटा दिए जाते हैं।

विवरण: [Slash commands](/tools/slash-commands).

## Sessions, compaction, और pruning (क्या स्थायी रहता है)

संदेशों के बीच क्या स्थायी रहता है, यह तंत्र पर निर्भर करता है:

- **Normal history** सत्र ट्रांसक्रिप्ट में तब तक रहती है जब तक नीति के अनुसार compacted/pruned न हो।
- **Compaction** ट्रांसक्रिप्ट में एक सारांश को स्थायी करता है और हाल के संदेशों को यथावत रखता है।
- **Pruning** किसी रन के लिए _in-memory_ prompt से पुराने tool results हटाता है, लेकिन ट्रांसक्रिप्ट को पुनर्लेखित नहीं करता।

दस्तावेज़: [Session](/concepts/session), [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning).

## `/context` वास्तव में क्या रिपोर्ट करता है

`/context` उपलब्ध होने पर नवीनतम **run-built** system prompt रिपोर्ट को प्राथमिकता देता है:

- `System prompt (run)` = अंतिम embedded (tool-capable) रन से कैप्चर किया गया और सत्र स्टोर में स्थायी किया गया।
- `System prompt (estimate)` = जब कोई रन रिपोर्ट मौजूद न हो (या ऐसे CLI बैकएंड के माध्यम से चलाते समय जो रिपोर्ट उत्पन्न नहीं करता) तब तुरंत गणना किया जाता है।

किसी भी स्थिति में, यह आकार और शीर्ष योगदानकर्ताओं की रिपोर्ट करता है; यह पूर्ण system prompt या tool schemas को **डंप नहीं** करता।


