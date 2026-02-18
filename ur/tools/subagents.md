---
title: "ذیلی ایجنٹس"
---

# ذیلی ایجنٹس

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

**Use cases:**

- Research a topic while the main agent continues answering questions
- Run multiple long tasks in parallel (web scraping, code analysis, file processing)
- Delegate tasks to specialized agents in a multi-agent setup

## فوری آغاز

The simplest way to use sub-agents is to ask your agent naturally:

> "Spawn a sub-agent to research the latest Node.js release notes"

The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.

You can also be explicit about options:

> "Spawn a sub-agent to analyze the server logs from today. Use gpt-5.2 and set a 5-minute timeout."

## یہ کیسے کام کرتا ہے

<Steps>
  <Step title="Main agent spawns">
    The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
  </Step>
  <Step title="Sub-agent runs in the background">
    A new isolated session is created (`agent:<agentId>:subagent:<uuid>`) on the dedicated `subagent` queue lane.
  </Step>
  <Step title="Result is announced">
    When the sub-agent finishes, it announces its findings back to the requester chat. The main agent posts a natural-language summary.
  </Step>
  <Step title="Session is archived">
    The sub-agent session is auto-archived after 60 minutes (configurable). Transcripts are preserved.
  </Step>
</Steps>

<Tip>
Each sub-agent has its **own** context and token usage. Set a cheaper model for sub-agents to save costs — see [Setting a Default Model](#setting-a-default-model) below.
</Tip>

## Configuration

Sub-agents work out of the box with no configuration. بطورِ طے شدہ:

- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Max concurrent: 8
- Auto-archive: after 60 minutes

### Setting a Default Model

Use a cheaper model for sub-agents to save on token costs:

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Setting a Default Thinking Level

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### فی-ایجنٹ اوور رائیڈز

ملٹی-ایجنٹ سیٹ اپ میں، آپ ہر ایجنٹ کے لیے سب-ایجنٹ ڈیفالٹس سیٹ کر سکتے ہیں:

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

### ہم وقتیّت

یہ کنٹرول کریں کہ ایک ہی وقت میں کتنے سب-ایجنٹس چل سکتے ہیں:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

سب-ایجنٹس ایک مخصوص کیو لین (`subagent`) استعمال کرتے ہیں جو مین ایجنٹ کیو سے الگ ہوتی ہے، اس لیے سب-ایجنٹ رنز آنے والے جوابات کو بلاک نہیں کرتے۔

### خودکار آرکائیو

سب-ایجنٹ سیشنز ایک قابلِ ترتیب مدت کے بعد خودکار طور پر آرکائیو ہو جاتے ہیں:

```json5
{
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>آرکائیو ٹرانسکرپٹ کا نام `*.deleted.` میں تبدیل کر دیتا ہے<timestamp>` (اسی فولڈر میں) — ٹرانسکرپٹس محفوظ رہتی ہیں، حذف نہیں کی جاتیں۔ خودکار آرکائیو ٹائمرز بہترین کوشش کی بنیاد پر ہوتے ہیں؛ اگر گیٹ وے دوبارہ شروع ہو جائے تو زیرِ التوا ٹائمرز ضائع ہو جاتے ہیں۔
</Note>

## `sessions_spawn` ٹول

یہ وہ ٹول ہے جسے ایجنٹ سب-ایجنٹس بنانے کے لیے کال کرتا ہے۔

### Parameters

| پیرامیٹر            | قسم                  | ڈیفالٹ                                | Description                                                                          |
| ------------------- | -------------------- | ------------------------------------- | ------------------------------------------------------------------------------------ |
| `task`              | string               | _(درکار)_          | سب-ایجنٹ کو کیا کرنا چاہیے                                                           |
| `label`             | string               | —                                     | شناخت کے لیے مختصر لیبل                                                              |
| `agentId`           | string               | _(کالر کا ایجنٹ)_  | کسی مختلف ایجنٹ آئی ڈی کے تحت اسپان کریں (اجازت ہونی چاہیے)       |
| `model`             | string               | _(اختیاری)_        | اس سب-ایجنٹ کے لیے ماڈل اوور رائیڈ کریں                                              |
| `thinking`          | string               | _(اختیاری)_        | سوچ کی سطح اوور رائیڈ کریں (`off`, `low`, `medium`, `high` وغیرہ) |
| `runTimeoutSeconds` | عدد                  | `0` (کوئی حد نہیں) | N سیکنڈ کے بعد سب-ایجنٹ کو منسوخ کریں                                                |
| `صفائی`             | "delete" \\| "keep" | "keep"                                | "delete" اعلان کے فوراً بعد آرکائیو کر دیتا ہے                                       |

### ماڈل ریزولوشن آرڈر

سب-ایجنٹ ماڈل اس ترتیب میں حل ہوتا ہے (پہلا میچ جیتتا ہے):

1. `sessions_spawn` کال میں واضح `model` پیرامیٹر
2. فی-ایجنٹ کنفیگریشن: `agents.list[].subagents.model`
3. عالمی ڈیفالٹ: `agents.defaults.subagents.model`
4. اس نئے سیشن کے لیے ہدف ایجنٹ کی معمول کی ماڈل ریزولوشن

سوچ کی سطح اس ترتیب میں حل ہوتی ہے:

1. `sessions_spawn` کال میں واضح `thinking` پیرامیٹر
2. فی-ایجنٹ کنفیگریشن: `agents.list[].subagents.thinking`
3. عالمی ڈیفالٹ: `agents.defaults.subagents.thinking`
4. ورنہ سب-ایجنٹ کے لیے مخصوص سوچ کی کوئی اوور رائیڈ لاگو نہیں ہوتی

<Note>غلط ماڈل ویلیوز کو خاموشی سے چھوڑ دیا جاتا ہے — سب-ایجنٹ اگلے درست ڈیفالٹ پر ایک وارننگ کے ساتھ ٹول رزلٹ میں چلتا ہے۔</Note>

### کراس-ایجنٹ اسپاننگ

ڈیفالٹ طور پر، سب-ایجنٹس صرف اپنی ہی ایجنٹ آئی ڈی کے تحت اسپان ہو سکتے ہیں۔ 1. کسی ایجنٹ کو دوسرے ایجنٹ آئی ڈیز کے تحت ذیلی ایجنٹس بنانے کی اجازت دینے کے لیے:

```json5
2. {
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // یا کسی بھی کی اجازت دینے کے لیے ["*"]
        },
      },
    ],
  },
}
```

<Tip>3. `sessions_spawn` کے لیے فی الحال کن ایجنٹ آئی ڈیز کی اجازت ہے یہ جاننے کے لیے `agents_list` ٹول استعمال کریں۔</Tip>

## 4. ذیلی ایجنٹس کا انتظام (`/subagents`)

5. موجودہ سیشن کے لیے ذیلی ایجنٹ رنز کا معائنہ اور کنٹرول کرنے کے لیے `/subagents` سلیش کمانڈ استعمال کریں:

| کمانڈ                                      | Description                                                                                          |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------- |
| `/subagents list`                          | 6. تمام ذیلی ایجنٹ رنز کی فہرست بنائیں (فعال اور مکمل شدہ) |
| `/subagents stop <id\\|#\\|all>`         | 7. چلتے ہوئے ذیلی ایجنٹ کو روکیں                                              |
| `/subagents log <id\\|#> [limit] [tools]` | 8. ذیلی ایجنٹ کی ٹرانسکرپٹ دیکھیں                                             |
| `/subagents info <id\\|#>`                | 9. رن کی تفصیلی میٹاڈیٹا دکھائیں                                              |
| `/subagents send <id\\|#> <message>`      | 10. چلتے ہوئے ذیلی ایجنٹ کو پیغام بھیجیں                                      |

11. آپ ذیلی ایجنٹس کو فہرست کے انڈیکس (`1`, `2`)، رن آئی ڈی کے ابتدائی حصے، مکمل سیشن کی، یا `last` کے ذریعے حوالہ دے سکتے ہیں۔

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">12. 
    ```
    /subagents list
    ```

    ````
    13. ```
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

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">14. 
    ```
    /subagents info 1
    ```

    ````
    15. ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">16. 
    ```
    /subagents log 1 10
    ```

    ````
    17. ذیلی ایجنٹ کی ٹرانسکرپٹ سے آخری 10 پیغامات دکھاتا ہے۔ ٹول کال پیغامات شامل کرنے کے لیے `tools` شامل کریں:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">18. 
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    19. چلتے ہوئے ذیلی ایجنٹ کے سیشن میں ایک پیغام بھیجتا ہے اور جواب کے لیے 30 سیکنڈ تک انتظار کرتا ہے۔
    ```

  </Accordion>
</AccordionGroup>

## 20. اعلان (نتائج کیسے واپس آتے ہیں)

21. جب کوئی ذیلی ایجنٹ مکمل ہو جاتا ہے، تو وہ **announce** مرحلے سے گزرتا ہے:

1. 22. ذیلی ایجنٹ کا آخری جواب محفوظ کر لیا جاتا ہے
2. 23. نتیجہ، اسٹیٹس اور اعداد و شمار کے ساتھ ایک خلاصہ پیغام مرکزی ایجنٹ کے سیشن کو بھیجا جاتا ہے
3. 24. مرکزی ایجنٹ آپ کی چیٹ میں قدرتی زبان میں خلاصہ پوسٹ کرتا ہے

اعلان کے جوابات دستیاب ہونے پر تھریڈ/موضوع کی روٹنگ برقرار رکھتے ہیں (Slack تھریڈز، Telegram ٹاپکس، Matrix تھریڈز)۔

### 25. اعلان کے اعداد و شمار

26. ہر اعلان میں ایک اسٹیٹس لائن شامل ہوتی ہے جس میں:

- 27. رن ٹائم کی مدت
- ٹوکن استعمال (ان پٹ/آؤٹ پٹ/کل)
- 28. تخمینی لاگت (جب ماڈل کی قیمتیں `models.providers.*.models[].cost` کے ذریعے ترتیب دی گئی ہوں)
- 29. سیشن کی، سیشن آئی ڈی، اور ٹرانسکرپٹ کا راستہ

### 30. اعلان کی حیثیت

31. اعلان کے پیغام میں رن ٹائم کے نتیجے سے اخذ کردہ حیثیت شامل ہوتی ہے (ماڈل آؤٹ پٹ سے نہیں):

- 32. **successful completion** (`ok`) — کام معمول کے مطابق مکمل ہوا
- 33. **error** — کام ناکام ہو گیا (غلطی کی تفصیلات نوٹس میں)
- 34. **timeout** — کام نے `runTimeoutSeconds` سے تجاوز کیا
- 35. **unknown** — حیثیت کا تعین نہیں ہو سکا

<Tip>
36. اگر صارف کے سامنے اعلان کی ضرورت نہ ہو، تو مرکزی ایجنٹ کا summarize مرحلہ `NO_REPLY` واپس کر سکتا ہے اور کچھ بھی پوسٹ نہیں کیا جاتا۔
37. یہ `ANNOUNCE_SKIP` سے مختلف ہے، جو ایجنٹ سے ایجنٹ اعلان کے فلو (`sessions_send`) میں استعمال ہوتا ہے۔
</Tip>

## 38. ٹول پالیسی

39. بطور ڈیفالٹ، ذیلی ایجنٹس کو **تمام ٹولز ملتے ہیں سوائے** ان ٹولز کے جنہیں غیر محفوظ یا پس منظر کے کاموں کے لیے غیر ضروری ہونے کی وجہ سے منع کیا گیا ہو:

<AccordionGroup>
  <Accordion title="Default denied tools">40. 
    | Denied tool | Reason |
    |-------------|--------|
    | `sessions_list` | سیشن مینجمنٹ — مرکزی ایجنٹ نظم و نسق کرتا ہے |
    | `sessions_history` | سیشن مینجمنٹ — مرکزی ایجنٹ نظم و نسق کرتا ہے |
    | `sessions_send` | سیشن مینجمنٹ — مرکزی ایجنٹ نظم و نسق کرتا ہے |
    | `sessions_spawn` | نیسٹڈ فین آؤٹ نہیں (ذیلی ایجنٹس مزید ذیلی ایجنٹس نہیں بنا سکتے) |
    | `gateway` | سسٹم ایڈمن — ذیلی ایجنٹ سے خطرناک |
    | `agents_list` | سسٹم ایڈمن |
    | `whatsapp_login` | انٹرایکٹو سیٹ اپ — کوئی ٹاسک نہیں |
    | `session_status` | حیثیت/شیڈولنگ — مرکزی ایجنٹ ہم آہنگی کرتا ہے |
    | `cron` | حیثیت/شیڈولنگ — مرکزی ایجنٹ ہم آہنگی کرتا ہے |
    | `memory_search` | اس کے بجائے متعلقہ معلومات spawn پرامپٹ میں دیں |
    | `memory_get` | اس کے بجائے متعلقہ معلومات spawn پرامپٹ میں دیں |</Accordion>
</AccordionGroup>

### 41. ذیلی ایجنٹ ٹولز کو حسبِ ضرورت بنانا

42. آپ ذیلی ایجنٹ کے ٹولز کو مزید محدود کر سکتے ہیں:

```json5
43. {
  tools: {
    subagents: {
      tools: {
        // deny ہمیشہ allow پر غالب رہتا ہے
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

44. ذیلی ایجنٹس کو **صرف** مخصوص ٹولز تک محدود کرنے کے لیے:

```json5
45. {
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // اگر set ہو تو deny پھر بھی غالب رہے گا
      },
    },
  },
}
```

<Note>
46. کسٹم deny اندراجات ڈیفالٹ deny فہرست میں **شامل** کر دیے جاتے ہیں۔ 47. اگر `allow` سیٹ ہو، تو صرف وہی ٹولز دستیاب ہوں گے (ڈیفالٹ deny فہرست اس کے اوپر لاگو رہے گی)۔
</Note>

## تصدیق

ذیلی ایجنٹ کی تصدیق **ایجنٹ آئی ڈی** کی بنیاد پر حل ہوتی ہے، نہ کہ سیشن کی قسم پر:

- 48. auth اسٹور ہدف ایجنٹ کے `agentDir` سے لوڈ کیا جاتا ہے
- 49. مرکزی ایجنٹ کے auth پروفائلز کو **fallback** کے طور پر ضم کیا جاتا ہے (تنازع کی صورت میں ایجنٹ کے پروفائلز غالب رہتے ہیں)
- 50. یہ ضم کرنا additive ہے — مرکزی پروفائلز ہمیشہ fallback کے طور پر دستیاب رہتے ہیں

<Note>1. 
ہر ذیلی ایجنٹ کے لیے مکمل طور پر الگ تھلگ تصدیق (auth) فی الحال معاونت یافتہ نہیں ہے۔</Note>

## 2. سیاق و سباق اور سسٹم پرامپٹ

3. ذیلی ایجنٹس کو مرکزی ایجنٹ کے مقابلے میں کم شدہ سسٹم پرامپٹ موصول ہوتا ہے:

- 4. **شامل:** Tooling، Workspace، Runtime حصے، نیز `AGENTS.md` اور `TOOLS.md`
- 5. **شامل نہیں:** `SOUL.md`، `IDENTITY.md`، `USER.md`، `HEARTBEAT.md`، `BOOTSTRAP.md`

6. ذیلی ایجنٹ کو ایک کام پر مرکوز سسٹم پرامپٹ بھی ملتا ہے جو اسے تفویض کردہ کام پر توجہ مرکوز رکھنے، اسے مکمل کرنے، اور مرکزی ایجنٹ کے طور پر عمل نہ کرنے کی ہدایت دیتا ہے۔

## 7. ذیلی ایجنٹس کو روکنا

| 8. طریقہ                   | 9. اثر                                                                                   |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| چیٹ میں `/stop`                                   | 11. مرکزی سیشن **اور** اس سے شروع ہونے والی تمام فعال ذیلی ایجنٹ رنز کو منسوخ کر دیتا ہے |
| 12. `/subagents stop <id>` | مرکزی سیشن کو متاثر کیے بغیر ایک مخصوص ذیلی ایجنٹ کو روکتا ہے                                                   |
| 14. `runTimeoutSeconds`    | 15. مقررہ وقت کے بعد ذیلی ایجنٹ رن کو خودکار طور پر منسوخ کر دیتا ہے                     |

<Note>
16. `runTimeoutSeconds` سیشن کو خودکار طور پر آرکائیو **نہیں** کرتا۔ 17. سیشن اس وقت تک برقرار رہتا ہے جب تک معمول کا آرکائیو ٹائمر فعال نہ ہو جائے۔
</Note>

## 18. مکمل کنفیگریشن کی مثال

<Accordion title="Complete sub-agent configuration">19. 
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops can spawn sub-agents under "main"
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agents can't use the browser
      },
    },
  },
}
```</Accordion>

## حدود

<Warning>
20. - **بہترین کوشش کے ساتھ اعلان:** اگر گیٹ وے دوبارہ شروع ہو جائے تو زیرِ التواء اعلان کا کام ضائع ہو جاتا ہے۔
21. - **نیسٹڈ اسپاننگ نہیں:** ذیلی ایجنٹس اپنے ذیلی ایجنٹس پیدا نہیں کر سکتے۔
22. - **مشترکہ وسائل:** ذیلی ایجنٹس گیٹ وے پروسیس کو مشترکہ طور پر استعمال کرتے ہیں؛ `maxConcurrent` کو حفاظتی حد کے طور پر استعمال کریں۔
23. - **خودکار آرکائیو بہترین کوشش پر مبنی ہے:** گیٹ وے کے دوبارہ شروع ہونے پر زیرِ التواء آرکائیو ٹائمرز ضائع ہو جاتے ہیں۔
</Warning>

## یہ بھی دیکھیں

- [سیشن ٹولز](/concepts/session-tool) — `sessions_spawn` اور دیگر سیشن ٹولز کی تفصیلات
- [ملٹی ایجنٹ سینڈ باکس اور ٹولز](/tools/multi-agent-sandbox-tools) — فی ایجنٹ ٹول پابندیاں اور سینڈ باکسنگ
- 26. [Configuration](/gateway/configuration) — `agents.defaults.subagents` حوالہ
- 27. [Queue](/concepts/queue) — `subagent` لین کس طرح کام کرتی ہے
