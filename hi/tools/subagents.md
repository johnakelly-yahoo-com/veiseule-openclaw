---
summary: "उप-एजेंट: ऐसे पृथक एजेंट रन उत्पन्न करना जो अनुरोधकर्ता चैट को परिणामों की घोषणा करते हैं"
read_when:
  - आप एजेंट के माध्यम से पृष्ठभूमि/समानांतर कार्य चाहते हैं
  - आप sessions_spawn या उप-एजेंट टूल नीति बदल रहे हैं
title: "उप-एजेंट"
---

# उप-एजेंट

Sub-agents बैकग्राउंड एजेंट रन होते हैं जो किसी मौजूदा एजेंट रन से शुरू किए जाते हैं। वे अपने स्वयं के सेशन (`agent:<agentId>:subagent:<uuid>`) में चलते हैं और पूरा होने पर अपना परिणाम अनुरोधकर्ता चैट चैनल में **announce** करते हैं।

## Slash कमांड

**वर्तमान सेशन** के लिए sub-agent रन की जाँच या नियंत्रण करने हेतु `/subagents` का उपयोग करें:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

The simplest way to use sub-agents is to ask your agent naturally:

मुख्य लक्ष्य:

- मुख्य रन को ब्लॉक किए बिना "research / लंबा कार्य / धीमा टूल" कार्यों को समानांतर करना।
- डिफ़ॉल्ट रूप से sub-agents को अलग-थलग रखना (सेशन पृथक्करण + वैकल्पिक sandboxing)।
- टूल सतह को दुरुपयोग से सुरक्षित रखना: sub-agents को डिफ़ॉल्ट रूप से सेशन टूल्स नहीं मिलते।
- orchestrator पैटर्न के लिए कॉन्फ़िगर करने योग्य nesting depth का समर्थन करना।

लागत नोट: प्रत्येक sub-agent का अपना **अलग** context और token उपयोग होता है। भारी या दोहराव वाले
कार्यों के लिए, sub-agents के लिए सस्ता मॉडल सेट करें और अपने मुख्य एजेंट को उच्च-गुणवत्ता मॉडल पर रखें।
आप इसे `agents.defaults.subagents.model` या प्रति-एजेंट ओवरराइड के माध्यम से कॉन्फ़िगर कर सकते हैं।

## टूल

`sessions_spawn` का उपयोग करें:

- एक sub-agent रन शुरू करता है (`deliver: false`, global lane: `subagent`)
- फिर एक announce चरण चलाता है और announce उत्तर को अनुरोधकर्ता चैट चैनल में पोस्ट करता है
- डिफ़ॉल्ट मॉडल: कॉलर से विरासत में मिलता है जब तक आप `agents.defaults.subagents.model` (या प्रति-एजेंट `agents.list[].subagents.model`) सेट नहीं करते; स्पष्ट `sessions_spawn.model` हमेशा प्राथमिकता लेता है।
- डिफ़ॉल्ट thinking: कॉलर से विरासत में मिलता है जब तक आप `agents.defaults.subagents.thinking` (या प्रति-एजेंट `agents.list[].subagents.thinking`) सेट नहीं करते; स्पष्ट `sessions_spawn.thinking` हमेशा प्राथमिकता लेता है।

टूल पैरामीटर:

- `task` (आवश्यक)
- `label?` (वैकल्पिक)
- `agentId?` (वैकल्पिक; अनुमति होने पर किसी अन्य एजेंट id के अंतर्गत शुरू करें)
- `model?` (वैकल्पिक; sub-agent मॉडल को ओवरराइड करता है; अमान्य मानों को छोड़ दिया जाता है और sub-agent डिफ़ॉल्ट मॉडल पर चलता है, टूल परिणाम में एक चेतावनी के साथ)
- `thinking?` (वैकल्पिक; sub-agent रन के लिए thinking स्तर को ओवरराइड करता है)
- `runTimeoutSeconds?` (डिफ़ॉल्ट `0`; सेट होने पर, sub-agent रन N सेकंड के बाद निरस्त कर दिया जाता है)
- `cleanup?` (`delete|keep`, डिफ़ॉल्ट `keep`)

उप-एजेंट बिना किसी कॉन्फ़िगरेशन के तुरंत काम करते हैं। डिफ़ॉल्ट्स:

- `agents.list[].subagents.allowAgents`: एजेंट ids की सूची जिन्हें `agentId` के माध्यम से लक्षित किया जा सकता है (`["*"]` किसी भी को अनुमति देने के लिए)। डिफ़ॉल्ट: केवल अनुरोधकर्ता एजेंट।

खोज (Discovery):

- यह देखने के लिए `agents_list` का उपयोग करें कि वर्तमान में `sessions_spawn` के लिए कौन-से एजेंट ids अनुमत हैं।

Auto-archive:

- Sub-agent सत्र `agents.defaults.subagents.archiveAfterMinutes` (डिफ़ॉल्ट: 60) के बाद स्वचालित रूप से आर्काइव कर दिए जाते हैं।
- Archive `sessions.delete` का उपयोग करता है और ट्रांसक्रिप्ट का नाम बदलकर `*.deleted.<timestamp>` कर देता है\` (उसी फ़ोल्डर में)।
- `cleanup: "delete"` announce के तुरंत बाद आर्काइव कर देता है (फिर भी rename के माध्यम से ट्रांसक्रिप्ट सुरक्षित रहता है)।
- Auto-archive best-effort आधार पर काम करता है; यदि gateway पुनः शुरू होता है तो लंबित टाइमर खो जाते हैं।
- `runTimeoutSeconds` **auto-archive** नहीं करता; यह केवल रन को रोकता है। सत्र auto-archive होने तक बना रहता है।
- Auto-archive depth-1 और depth-2 सत्रों पर समान रूप से लागू होता है।

## Nested Sub-Agents

डिफ़ॉल्ट रूप से, sub-agents अपने स्वयं के sub-agents शुरू नहीं कर सकते (`maxSpawnDepth: 1`)। आप `maxSpawnDepth: 2` सेट करके एक स्तर की nesting सक्षम कर सकते हैं, जो **orchestrator pattern** की अनुमति देता है: main → orchestrator sub-agent → worker sub-sub-agents।

### सक्षम कैसे करें

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### समवर्तीता

| Depth | Session key आकार                             | भूमिका                                                                 | क्या spawn कर सकता है?        |
| ----- | -------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------- |
| 0     | `agent:&lt;id&gt;:main`                            | Main agent                                                             | हमेशा                         |
| 1     | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (जब depth 2 की अनुमति हो तो orchestrator) | केवल यदि `maxSpawnDepth >= 2` |
| 2     | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (leaf worker)                         | कभी नहीं                      |

### Announce श्रृंखला

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

1. Depth-2 worker समाप्त होता है → अपने parent (depth-1 orchestrator) को announce करता है
2. Depth-1 orchestrator announce प्राप्त करता है, परिणामों को संयोजित करता है, समाप्त होता है → main को announce करता है
3. Main agent announce प्राप्त करता है और उपयोगकर्ता को प्रदान करता है

Sub-agent sessions are automatically archived after a configurable period:

### Depth के अनुसार Tool नीति

- **Depth 1 (orchestrator, जब `maxSpawnDepth >= 2`)**: `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` प्राप्त करता है ताकि वह अपने children को प्रबंधित कर सके। अन्य session/system tools प्रतिबंधित रहते हैं।
- **Depth 1 (leaf, जब `maxSpawnDepth == 1`)**: कोई session tools नहीं (वर्तमान डिफ़ॉल्ट व्यवहार)।
- **Depth 2 (leaf worker)**: कोई session tools नहीं — depth 2 पर `sessions_spawn` हमेशा प्रतिबंधित रहता है। आगे और children spawn नहीं कर सकता।

### The `sessions_spawn` Tool

प्रत्येक agent सत्र (किसी भी depth पर) में एक समय में अधिकतम `maxChildrenPerAgent` (डिफ़ॉल्ट: 5) सक्रिय children हो सकते हैं। यह एक ही orchestrator से अनियंत्रित fan-out को रोकता है।

### पैरामीटर

गहराई-1 orchestrator को रोकने पर उसके सभी गहराई-2 child स्वतः रुक जाते हैं:

- `/stop` को मुख्य चैट में भेजने से सभी गहराई-1 agents रुक जाते हैं और यह उनके गहराई-2 children तक क्रमिक रूप से लागू होता है।
- `/subagents kill <id>` किसी विशिष्ट sub-agent को रोकता है और उसके children तक क्रमिक रूप से लागू होता है।
- `/subagents kill all` अनुरोधकर्ता के सभी sub-agents को रोकता है और क्रमिक रूप से लागू होता है।

## प्रमाणीकरण

उप-एजेंट प्रमाणीकरण **एजेंट आईडी** द्वारा हल किया जाता है, सत्र प्रकार द्वारा नहीं:

- sub-agent session key है `agent:<agentId>:subagent:<uuid>`।
- auth store उस agent के `agentDir` से लोड किया जाता है।
- मुख्य agent के auth profiles को **fallback** के रूप में मर्ज किया जाता है; टकराव होने पर agent profiles, main profiles को ओवरराइड करते हैं।

नोट: यह मर्ज additive है, इसलिए main profiles हमेशा fallback के रूप में उपलब्ध रहते हैं। प्रत्येक agent के लिए पूर्णतः अलग-थलग auth अभी समर्थित नहीं है।

## घोषणा

Sub-agents एक announce step के माध्यम से वापस रिपोर्ट करते हैं:

- announce step sub-agent session के अंदर चलता है (अनुरोधकर्ता session में नहीं)।
- यदि sub-agent ठीक-ठीक `ANNOUNCE_SKIP` उत्तर देता है, तो कुछ भी पोस्ट नहीं किया जाता।
- अन्यथा announce का उत्तर अनुरोधकर्ता के चैट चैनल में एक follow-up `agent` कॉल (`deliver=true`) के माध्यम से पोस्ट किया जाता है।
- उपलब्ध होने पर घोषणा उत्तर थ्रेड/टॉपिक रूटिंग को संरक्षित रखते हैं (Slack थ्रेड्स, Telegram टॉपिक्स, Matrix थ्रेड्स)।
- Announce संदेशों को एक स्थिर टेम्पलेट में सामान्यीकृत किया जाता है:
  - `Status:` रन के परिणाम (`success`, `error`, `timeout`, या `unknown`) से प्राप्त।
  - `Result:` announce step की सारांश सामग्री (या अनुपलब्ध होने पर `(not available)`)।
  - `Notes:` त्रुटि विवरण और अन्य उपयोगी संदर्भ।
- `Status` मॉडल आउटपुट से अनुमानित नहीं होता; यह runtime outcome signals से आता है।

Announce payloads के अंत में एक stats लाइन शामिल होती है (भले ही वे wrapped हों):

- Runtime (उदाहरण के लिए, `runtime 5m12s`)
- टोकन उपयोग (इनपुट/आउटपुट/कुल)
- जब model pricing कॉन्फ़िगर हो (`models.providers.*.models[].cost`) तो अनुमानित लागत
- `sessionKey`, `sessionId`, और transcript path (ताकि मुख्य agent `sessions_history` के माध्यम से इतिहास प्राप्त कर सके या डिस्क पर फ़ाइल की जाँच कर सके)

## Managing Sub-Agents (`/subagents`)

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

config के माध्यम से ओवरराइड करें:

`````json5
````
```
🧭 Subagents (current session)
Active: 1 · Done: 2
1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
```

```
/subagents stop 3
```

```
⚙️ Stop requested for deploy staging.
```
````
`````

## समवर्तीता

Sub-agents एक समर्पित in-process queue lane का उपयोग करते हैं:

- Lane नाम: `subagent`
- Concurrency: `agents.defaults.subagents.maxConcurrent` (डिफ़ॉल्ट `8`)

## रोकना

- अनुरोधकर्ता चैट में `/stop` भेजने से अनुरोधकर्ता session निरस्त हो जाता है और उससे शुरू किए गए सभी सक्रिय sub-agent runs रुक जाते हैं, तथा nested children तक क्रमिक रूप से लागू होता है।
- `/subagents kill <id>` किसी विशिष्ट sub-agent को रोकता है और उसके children तक क्रमिक रूप से लागू होता है।

## सीमाएँ

- Sub-agent announce **best-effort** है। यदि gateway पुनः प्रारंभ होता है, तो लंबित "announce back" कार्य खो जाता है।
- Sub-agents अभी भी वही gateway process संसाधन साझा करते हैं; `maxConcurrent` को एक सुरक्षा वाल्व के रूप में मानें।
- `sessions_spawn` हमेशा non-blocking है: यह तुरंत `{ status: "accepted", runId, childSessionKey }` लौटाता है।
- Sub-agent context केवल `AGENTS.md` + `TOOLS.md` inject करता है (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, या `BOOTSTRAP.md` नहीं)।
- अधिकतम nesting depth 5 है (`maxSpawnDepth` सीमा: 1–5)। अधिकांश उपयोग मामलों के लिए Depth 2 अनुशंसित है।
- `maxChildrenPerAgent` प्रति सत्र सक्रिय children की सीमा निर्धारित करता है (default: 5, range: 1–20)।

