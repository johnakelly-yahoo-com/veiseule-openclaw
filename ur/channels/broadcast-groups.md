---
status: experimental
title: "براڈکاسٹ گروپس"
---

# براڈکاسٹ گروپس

**حالت:** تجرباتی  
**ورژن:** 2026.1.9 میں شامل کیا گیا

## جائزہ

براڈکاسٹ گروپس متعدد ایجنٹس کو ایک ہی پیغام کو بیک وقت پراسیس کرنے اور اس کا جواب دینے کی سہولت دیتے ہیں۔ اس سے آپ ماہر ایجنٹس کی ایسی ٹیمیں بنا سکتے ہیں جو ایک ہی WhatsApp گروپ یا DM میں مل کر کام کریں — اور وہ بھی ایک ہی فون نمبر کے ذریعے۔

موجودہ دائرہ کار: **صرف WhatsApp** (ویب چینل)۔

براڈکاسٹ گروپس کا جائزہ چینل allowlists اور گروپ ایکٹیویشن قواعد کے بعد لیا جاتا ہے۔ WhatsApp گروپس میں اس کا مطلب یہ ہے کہ براڈکاسٹ اسی وقت ہوتا ہے جب OpenClaw عام طور پر جواب دیتا ہے (مثال کے طور پر: مینشن پر، آپ کی گروپ سیٹنگز کے مطابق)۔

## استعمال کے معاملات

### 1. ماہر ایجنٹس کی ٹیمیں

ایٹامک، مرکوز ذمہ داریوں کے ساتھ متعدد ایجنٹس تعینات کریں:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

ہر ایجنٹ ایک ہی پیغام کو پروسیس کرتا ہے اور اپنی مخصوص نقطۂ نظر فراہم کرتا ہے۔

### 2. کثیر لسانی معاونت

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. کوالٹی اشورنس ورک فلو

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. ٹاسک آٹومیشن

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## کنفیگریشن

### بنیادی سیٹ اپ

اوپر کی سطح پر ایک `broadcast` سیکشن شامل کریں ( `bindings` کے ساتھ)۔ کیز WhatsApp peer IDs ہیں:

- گروپ چیٹس: گروپ JID (مثلاً `120363403215116621@g.us`)
- DMs: E.164 فون نمبر (مثلاً `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**نتیجہ:** جب OpenClaw اس چیٹ میں جواب دے گا، تو یہ تینوں ایجنٹس کو چلائے گا۔

### پروسیسنگ اسٹریٹیجی

یہ کنٹرول کریں کہ ایجنٹس پیغامات کیسے پروسیس کریں:

#### متوازی (بطورِ طے شدہ)

تمام ایجنٹس بیک وقت پروسیس کرتے ہیں:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### ترتیبی

ایجنٹس ترتیب وار پروسیس کرتے ہیں (ایک مکمل ہونے کا انتظار کرتا ہے):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### مکمل مثال

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

## یہ کیسے کام کرتا ہے

### پیغام کا بہاؤ

1. **آنے والا پیغام** WhatsApp گروپ میں موصول ہوتا ہے
2. **براڈکاسٹ چیک**: سسٹم دیکھتا ہے کہ پیئر آئی ڈی `broadcast` میں موجود ہے یا نہیں
3. **اگر براڈکاسٹ فہرست میں ہو**:
   - فہرست میں شامل تمام ایجنٹس پیغام پروسیس کرتے ہیں
   - ہر ایجنٹ کے پاس اپنی سیشن کلید اور الگ تھلگ سیاق ہوتا ہے
   - ایجنٹس متوازی (بطورِ طے شدہ) یا ترتیبی طور پر پروسیس کرتے ہیں
4. **اگر براڈکاسٹ فہرست میں نہ ہو**:
   - معمول کی روٹنگ لاگو ہوتی ہے (پہلا مماثل بائنڈنگ)

نوٹ: براڈکاسٹ گروپس چینل allowlists یا گروپ ایکٹیویشن قواعد (مینشنز/کمانڈز وغیرہ) کو بائی پاس نہیں کرتے۔ یہ صرف اس بات کو تبدیل کرتے ہیں کہ جب کوئی پیغام پراسیسنگ کے لیے اہل ہو تو _کون سے ایجنٹس چلیں گے_۔

### سیشن آئسولیشن

براڈکاسٹ گروپ میں ہر ایجنٹ مکمل طور پر الگ رکھتا ہے:

- **سیشن کلیدیں** (`agent:alfred:whatsapp:group:120363...` بمقابلہ `agent:baerbel:whatsapp:group:120363...`)
- **گفتگو کی تاریخ** (ایجنٹ دوسرے ایجنٹس کے پیغامات نہیں دیکھتا)
- **ورک اسپیس** (اگر کنفیگر ہو تو علیحدہ sandboxes)
- **ٹول رسائی** (مختلف اجازت/انکار فہرستیں)
- **میموری/سیاق** (الگ IDENTITY.md، SOUL.md، وغیرہ)
- **گروپ سیاق بفر** (سیاق کے لیے حالیہ گروپ پیغامات) پیئر کے حساب سے مشترک ہوتا ہے، اس لیے ٹرگر ہونے پر تمام براڈکاسٹ ایجنٹس ایک ہی سیاق دیکھتے ہیں

اس سے ہر ایجنٹ کو یہ سہولت ملتی ہے کہ اس کے پاس ہوں:

- مختلف شخصیات
- مختلف ٹول رسائی (مثلاً، صرف پڑھنے کے لیے بمقابلہ پڑھنے-لکھنے کے لیے)
- مختلف ماڈلز (مثلاً، opus بمقابلہ sonnet)
- مختلف Skills نصب شدہ

### مثال: الگ تھلگ سیشنز

گروپ `120363403215116621@g.us` میں ایجنٹس `["alfred", "baerbel"]` کے ساتھ:

**Alfred کا سیاق:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel کا سیاق:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## بہترین طریقۂ کار

### 1. ایجنٹس کو مرکوز رکھیں

ہر ایجنٹ کو ایک واحد، واضح ذمہ داری کے ساتھ ڈیزائن کریں:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **اچھا:** ہر ایجنٹ کا ایک کام  
❌ **برا:** ایک عمومی "dev-helper" ایجنٹ

### 2. وضاحتی نام استعمال کریں

یہ واضح کریں کہ ہر ایجنٹ کیا کرتا ہے:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Configure Different Tool Access

ایجنٹس کو صرف وہی ٹولز دیں جن کی انہیں ضرورت ہے:

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

### 4. Monitor Performance

زیادہ ایجنٹس کے ساتھ، ان باتوں پر غور کریں:

- رفتار کے لیے `"strategy": "parallel"` (بطورِ طے شدہ) استعمال کریں
- براڈکاسٹ گروپس کو 5–10 ایجنٹس تک محدود رکھیں
- سادہ ایجنٹس کے لیے تیز ماڈلز استعمال کریں

### 5. Handle Failures Gracefully

Agents fail independently. One agent's error doesn't block others:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## مطابقت

### فراہم کنندگان

براڈکاسٹ گروپس فی الحال ان کے ساتھ کام کرتے ہیں:

- ✅ WhatsApp (نافذ شدہ)
- 🚧 Telegram (منصوبہ بند)
- 🚧 Discord (منصوبہ بند)
- 🚧 Slack (منصوبہ بند)

### روٹنگ

براڈکاسٹ گروپس موجودہ روٹنگ کے ساتھ مل کر کام کرتے ہیں:

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

- `GROUP_A`: صرف alfred جواب دیتا ہے (معمول کی روٹنگ)
- `GROUP_B`: agent1 اور agent2 دونوں جواب دیتے ہیں (براڈکاسٹ)

**ترجیح:** `broadcast` کو `bindings` پر فوقیت حاصل ہے۔

## خرابیوں کا ازالہ

### ایجنٹس جواب نہیں دے رہے

**چیک کریں:**

1. ایجنٹ آئی ڈیز `agents.list` میں موجود ہوں
2. پیئر آئی ڈی فارمیٹ درست ہو (مثلاً، `120363403215116621@g.us`)
3. ایجنٹس deny فہرستوں میں نہ ہوں

**ڈیبگ:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### صرف ایک ایجنٹ جواب دے رہا ہے

**وجہ:** ممکن ہے پیئر آئی ڈی `bindings` میں ہو لیکن `broadcast` میں نہ ہو۔

**حل:** براڈکاسٹ کنفیگ میں شامل کریں یا بائنڈنگز سے ہٹا دیں۔

### کارکردگی کے مسائل

**اگر زیادہ ایجنٹس کے ساتھ سست ہو:**

- فی گروپ ایجنٹس کی تعداد کم کریں
- ہلکے ماڈلز استعمال کریں (opus کے بجائے sonnet)
- sandbox اسٹارٹ اپ وقت چیک کریں

## مثالیں

### مثال 1: کوڈ ریویو ٹیم

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

**یوزر بھیجتا ہے:** کوڈ اسنیپٹ  
**جوابات:**

- code-formatter: "انڈینٹیشن درست کی اور ٹائپ ہِنٹس شامل کیے"
- security-scanner: "⚠️ لائن 12 میں SQL انجیکشن کی کمزوری"
- test-coverage: "کوریج 45% ہے، ایرر کیسز کے لیے ٹیسٹس غائب ہیں"
- docs-checker: "فنکشن `process_data` کے لیے ڈاک اسٹرنگ غائب ہے"

### مثال 2: کثیر لسانی معاونت

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

## API حوالہ

### کنفیگ اسکیما

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### فیلڈز

- `strategy` (اختیاری): ایجنٹس کو کیسے پروسیس کیا جائے
  - `"parallel"` (بطورِ طے شدہ): تمام ایجنٹس بیک وقت پروسیس کرتے ہیں
  - `"sequential"`: ایجنٹس ارے کی ترتیب کے مطابق پروسیس کرتے ہیں
- `[peerId]`: WhatsApp گروپ JID، E.164 نمبر، یا دیگر پیئر آئی ڈی
  - ویلیو: ایجنٹ آئی ڈیز کی ارے جو پیغامات پروسیس کریں

## حدود

1. **زیادہ سے زیادہ ایجنٹس:** کوئی سخت حد نہیں، لیکن 10+ ایجنٹس سست ہو سکتے ہیں
2. **مشترک سیاق:** ایجنٹس ایک دوسرے کے جوابات نہیں دیکھتے (ڈیزائن کے مطابق)
3. **پیغام کی ترتیب:** متوازی جوابات کسی بھی ترتیب میں آ سکتے ہیں
4. **ریٹ لمٹس:** تمام ایجنٹس WhatsApp ریٹ لمٹس میں شمار ہوتے ہیں

## مستقبل کی بہتریاں

منصوبہ بند خصوصیات:

- [ ] مشترک سیاق موڈ (ایجنٹس ایک دوسرے کے جوابات دیکھ سکیں)
- [ ] ایجنٹ کوآرڈینیشن (ایجنٹس ایک دوسرے کو سگنل دے سکیں)
- [ ] ڈائنامک ایجنٹ سلیکشن (پیغام کے مواد کی بنیاد پر ایجنٹس منتخب ہوں)
- [ ] ایجنٹ ترجیحات (کچھ ایجنٹس دوسروں سے پہلے جواب دیں)

## یہ بھی دیکھیں

- [Multi-Agent Configuration](/tools/multi-agent-sandbox-tools)
- [Routing Configuration](/channels/channel-routing)
- [Session Management](/concepts/sessions)

