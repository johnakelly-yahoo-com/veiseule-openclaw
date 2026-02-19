---
summary: "एक WhatsApp संदेश को कई एजेंट्स तक प्रसारित करें"
read_when:
  - ब्रॉडकास्ट समूहों का विन्यास
  - WhatsApp में मल्टी-एजेंट उत्तरों का डिबग करना
status: experimental
title: "ब्रॉडकास्ट समूह"
---

# ब्रॉडकास्ट समूह

**स्थिति:** प्रायोगिक  
**संस्करण:** 2026.1.9 में जोड़ा गया

## अवलोकन

Broadcast Groups कई agents को एक ही संदेश को एक साथ प्रोसेस करने और प्रतिक्रिया देने में सक्षम बनाते हैं। यह आपको एक ही WhatsApp group या DM में साथ काम करने वाली विशेषीकृत agent टीमों को बनाने देता है — सभी एक ही फ़ोन नंबर का उपयोग करते हुए।

वर्तमान दायरा: **केवल WhatsApp** (वेब चैनल)।

Broadcast groups को channel allowlists और group activation rules के बाद मूल्यांकन किया जाता है। WhatsApp groups में, इसका अर्थ है कि broadcasts तब होते हैं जब OpenClaw सामान्यतः उत्तर देता (उदाहरण के लिए: mention पर, आपकी group settings पर निर्भर)।

## उपयोग के मामले

### 1. विशेषीकृत Agent टीमें

परमाण्विक, केंद्रित जिम्मेदारियों के साथ कई एजेंट्स तैनात करें:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

प्रत्येक एजेंट एक ही संदेश को संसाधित करता है और अपना विशेष दृष्टिकोण प्रदान करता है।

### 2. बहु-भाषा समर्थन

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. गुणवत्ता आश्वासन वर्कफ़्लो

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. कार्य स्वचालन

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## विन्यास

### बुनियादी सेटअप

शीर्ष-स्तरीय `broadcast` सेक्शन जोड़ें (`bindings` के बगल में)। Keys WhatsApp peer ids होते हैं:

- group chats: group JID (उदा. `120363403215116621@g.us`)
- DMs: E.164 फ़ोन नंबर (उदा. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**परिणाम:** जब OpenClaw इस चैट में उत्तर देगा, तब यह सभी तीन एजेंट्स चलाएगा।

### प्रसंस्करण रणनीति

यह नियंत्रित करें कि एजेंट्स संदेशों को कैसे संसाधित करें:

#### समानांतर (डिफ़ॉल्ट)

सभी एजेंट्स एक साथ संसाधित करते हैं:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### क्रमिक

एजेंट्स क्रम में संसाधित करते हैं (एक, पिछले के समाप्त होने की प्रतीक्षा करता है):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### पूर्ण उदाहरण

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## यह कैसे काम करता है

### संदेश प्रवाह

1. **आने वाला संदेश** WhatsApp समूह में आता है
2. **ब्रॉडकास्ट जाँच**: सिस्टम जाँचता है कि peer ID `broadcast` में है या नहीं
3. **यदि ब्रॉडकास्ट सूची में है**:
   - सूचीबद्ध सभी एजेंट्स संदेश को संसाधित करते हैं
   - प्रत्येक एजेंट की अपनी सत्र कुंजी और पृथक संदर्भ होता है
   - एजेंट्स समानांतर (डिफ़ॉल्ट) या क्रमिक रूप से संसाधित करते हैं
4. **यदि ब्रॉडकास्ट सूची में नहीं है**:
   - सामान्य रूटिंग लागू होती है (पहला मिलान करने वाला बाइंडिंग)

नोट: ब्रॉडकास्ट समूह चैनल allowlists या समूह सक्रियण नियमों (mentions/commands/etc) को बायपास नहीं करते। वे केवल यह बदलते हैं कि जब कोई संदेश प्रोसेसिंग के लिए पात्र होता है तो _कौन‑से एजेंट चलते हैं_।

### सत्र पृथक्करण

ब्रॉडकास्ट समूह में प्रत्येक एजेंट पूरी तरह से अलग बनाए रखता है:

- **सत्र कुंजियाँ** (`agent:alfred:whatsapp:group:120363...` बनाम `agent:baerbel:whatsapp:group:120363...`)
- **वार्तालाप इतिहास** (एजेंट अन्य एजेंट्स के संदेश नहीं देखता)
- **वर्कस्पेस** (यदि विन्यस्त हो तो अलग-अलग sandbox)
- **टूल पहुँच** (भिन्न allow/deny सूचियाँ)
- **मेमोरी/संदर्भ** (अलग IDENTITY.md, SOUL.md, आदि)
- **समूह संदर्भ बफ़र** (संदर्भ हेतु प्रयुक्त हालिया समूह संदेश) प्रति peer साझा होता है, इसलिए सभी ब्रॉडकास्ट एजेंट्स ट्रिगर होने पर समान संदर्भ देखते हैं

इससे प्रत्येक एजेंट के पास हो सकता है:

- अलग-अलग व्यक्तित्व
- अलग-अलग टूल पहुँच (जैसे, केवल-पठन बनाम पठन-लेखन)
- अलग-अलग मॉडल (जैसे, opus बनाम sonnet)
- अलग-अलग Skills इंस्टॉल

### उदाहरण: पृथक सत्र

समूह `120363403215116621@g.us` में एजेंट्स `["alfred", "baerbel"]` के साथ:

**Alfred का संदर्भ:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel का संदर्भ:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## सर्वोत्तम प्रथाएँ

### 1. एजेंट्स को केंद्रित रखें

प्रत्येक एजेंट को एकल, स्पष्ट जिम्मेदारी के साथ डिज़ाइन करें:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **अच्छा:** प्रत्येक एजेंट का एक काम  
❌ **खराब:** एक सामान्य "dev-helper" एजेंट

### 2. वर्णनात्मक नामों का उपयोग करें

यह स्पष्ट करें कि प्रत्येक एजेंट क्या करता है:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. विभिन्न टूल एक्सेस कॉन्फ़िगर करें

एजेंट्स को केवल वही टूल दें जिनकी उन्हें आवश्यकता है:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. प्रदर्शन की निगरानी करें

कई एजेंट्स के साथ, विचार करें:

- गति के लिए `"strategy": "parallel"` (डिफ़ॉल्ट) का उपयोग
- ब्रॉडकास्ट समूहों को 5–10 एजेंट्स तक सीमित करना
- सरल एजेंट्स के लिए तेज़ मॉडल का उपयोग

### 5. विफलताओं को शालीनता से संभालें

एजेंट स्वतंत्र रूप से विफल होते हैं। एक एजेंट की त्रुटि दूसरों को ब्लॉक नहीं करती:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## संगतता

### प्रदाता

ब्रॉडकास्ट समूह वर्तमान में इनके साथ काम करते हैं:

- ✅ WhatsApp (कार्यान्वित)
- 🚧 Telegram (योजनाबद्ध)
- 🚧 Discord (योजनाबद्ध)
- 🚧 Slack (योजनाबद्ध)

### रूटिंग

ब्रॉडकास्ट समूह मौजूदा रूटिंग के साथ मिलकर काम करते हैं:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: केवल alfred उत्तर देता है (सामान्य रूटिंग)
- `GROUP_B`: agent1 और agent2 दोनों उत्तर देते हैं (ब्रॉडकास्ट)

**प्राथमिकता:** `broadcast` को `bindings` पर प्राथमिकता मिलती है।

## समस्या-निवारण

### एजेंट्स उत्तर नहीं दे रहे

**जाँचें:**

1. `agents.list` में एजेंट IDs मौजूद हैं
2. Peer ID फ़ॉर्मैट सही है (उदाहरण: `120363403215116621@g.us`)
3. एजेंट्स deny सूचियों में नहीं हैं

**डिबग:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### केवल एक एजेंट उत्तर दे रहा है

**कारण:** Peer ID संभवतः `bindings` में है लेकिन `broadcast` में नहीं।

**समाधान:** ब्रॉडकास्ट विन्यास में जोड़ें या bindings से हटाएँ।

### प्रदर्शन संबंधी समस्याएँ

**यदि कई एजेंट्स के साथ धीमा हो:**

- प्रति समूह एजेंट्स की संख्या कम करें
- हल्के मॉडल का उपयोग करें (opus के बजाय sonnet)
- sandbox स्टार्टअप समय जाँचें

## उदाहरण

### उदाहरण 1: कोड समीक्षा टीम

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**उपयोगकर्ता भेजता है:** कोड स्निपेट  
**उत्तर:**

- code-formatter: "इंडेंटेशन ठीक किया और टाइप हिंट्स जोड़े"
- security-scanner: "⚠️ पंक्ति 12 में SQL injection भेद्यता"
- test-coverage: "कवरेज 45% है, त्रुटि मामलों के लिए परीक्षण गायब हैं"
- docs-checker: "फ़ंक्शन `process_data` के लिए docstring गायब है"

### उदाहरण 2: बहु-भाषा समर्थन

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## API संदर्भ

### विन्यास स्कीमा

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### फ़ील्ड्स

- `strategy` (वैकल्पिक): एजेंट्स को कैसे संसाधित किया जाए
  - `"parallel"` (डिफ़ॉल्ट): सभी एजेंट्स एक साथ संसाधित करते हैं
  - `"sequential"`: एजेंट्स ऐरे क्रम में संसाधित करते हैं
- `[peerId]`: WhatsApp समूह JID, E.164 नंबर, या अन्य peer ID
  - मान: उन एजेंट IDs की ऐरे जो संदेशों को संसाधित करें

## सीमाएँ

1. **अधिकतम एजेंट्स:** कोई कठोर सीमा नहीं, लेकिन 10+ एजेंट्स धीमे हो सकते हैं
2. **साझा संदर्भ:** एजेंट्स एक-दूसरे के उत्तर नहीं देखते (डिज़ाइन के अनुसार)
3. **संदेश क्रम:** समानांतर उत्तर किसी भी क्रम में आ सकते हैं
4. **दर सीमाएँ:** सभी एजेंट्स WhatsApp दर सीमाओं में गिने जाते हैं

## भविष्य के संवर्द्धन

योजनाबद्ध विशेषताएँ:

- [ ] साझा संदर्भ मोड (एजेंट्स एक-दूसरे के उत्तर देखें)
- [ ] एजेंट समन्वय (एजेंट्स एक-दूसरे को संकेत दे सकें)
- [ ] गतिशील एजेंट चयन (संदेश सामग्री के आधार पर एजेंट चुनना)
- [ ] एजेंट प्राथमिकताएँ (कुछ एजेंट्स पहले उत्तर दें)

## यह भी देखें

- [मल्टी-एजेंट विन्यास](/tools/multi-agent-sandbox-tools)
- [रूटिंग विन्यास](/channels/channel-routing)
- [सत्र प्रबंधन](/concepts/sessions)

