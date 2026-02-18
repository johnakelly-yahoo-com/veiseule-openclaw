---
title: "الوكلاء الفرعيون"
---

# الوكلاء الفرعيون

تتيح لك الوكلاء الفرعيون تشغيل مهام في الخلفية دون حجب المحادثة الرئيسية. عند إنشاء وكيل فرعي، يعمل في جلسة معزولة خاصة به، وينفذ عمله، ثم يعلن النتيجة في الدردشة عند الانتهاء.

**حالات الاستخدام:**

- البحث في موضوع ما بينما يواصل الوكيل الرئيسي الإجابة عن الأسئلة
- تشغيل عدة مهام طويلة بالتوازي (استخلاص الويب، تحليل الشيفرة، معالجة الملفات)
- تفويض المهام إلى وكلاء متخصصين في إعداد متعدد الوكلاء

## البدء السريع

أبسط طريقة لاستخدام الوكلاء الفرعيين هي أن تطلب من وكيلك بشكل طبيعي:

> "أنشئ وكيلًا فرعيًا للبحث في أحدث ملاحظات إصدار Node.js"

سيستدعي الوكيل أداة `sessions_spawn` في الخلفية. عندما ينتهي الوكيل الفرعي، يعلن نتائجه في دردشتك.

يمكنك أيضًا تحديد الخيارات بشكل صريح:

> "أنشئ وكيلًا فرعيًا لتحليل سجلات الخادم لليوم.
> استخدم gpt-5.2 واضبط مهلة قدرها 5 دقائق." يستدعي الوكيل الرئيسي `sessions_spawn` مع وصف للمهمة.

## كيف يعمل

<Steps>
  <Step title="Main agent spawns">
    الاستدعاء **غير حاجب** — يحصل الوكيل الرئيسي فورًا على `{ status: "accepted", runId, childSessionKey }`. 
    يتم إنشاء جلسة معزولة جديدة (`agent:
    :subagent:
    `) على مسار قائمة الانتظار المخصص `subagent`.
  
  </Step>
  <Step title="Sub-agent runs in the background">عندما ينتهي الوكيل الفرعي، يعلن نتائجه مرة أخرى في دردشة الطالب.<agentId>ينشر الوكيل الرئيسي ملخصًا بلغة طبيعية.<uuid>تتم أرشفة جلسة الوكيل الفرعي تلقائيًا بعد 60 دقيقة (قابل للضبط).</Step>
  <Step title="Result is announced">
    يتم الاحتفاظ بنصوص المحادثات. لكل وكيل فرعي سياقه **الخاص** واستهلاك الرموز الخاص به.
  </Step>
  <Step title="Session is archived">
    عيّن نموذجًا أقل تكلفة للوكلاء الفرعيين لتوفير التكاليف — راجع [Setting a Default Model](#setting-a-default-model) أدناه. تعمل الوكلاء الفرعيون مباشرة دون أي إعداد.
  </Step>
</Steps>

<Tip>
النموذج: اختيار النموذج الافتراضي للوكيل الهدف (ما لم يتم تعيين `subagents.model`) التفكير: لا يوجد تجاوز للوكيل الفرعي (ما لم يتم تعيين `subagents.thinking`)
</Tip>

## التهيئة

الحد الأقصى للتزامن: 8 القيم الافتراضية:

- الأرشفة التلقائية: بعد 60 دقيقة
- تعيين نموذج افتراضي
- استخدم نموذجًا أقل تكلفة للوكلاء الفرعيين لتقليل تكاليف الرموز:
- {
  agents: {
  defaults: {
  subagents: {
  model: "minimax/MiniMax-M2.1",
  },
  },
  },
  }

### تعيين مستوى تفكير افتراضي

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

### Per-Agent Overrides

In a multi-agent setup, you can set sub-agent defaults per agent:

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

### التزامن

Control how many sub-agents can run at the same time:

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

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

### Auto-Archive

Sub-agent sessions are automatically archived after a configurable period:

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

<Note>
Archive renames the transcript to `*.deleted.<timestamp>` (same folder) — transcripts are preserved, not deleted. Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
</Note>

## The `sessions_spawn` Tool

This is the tool the agent calls to create sub-agents.

### المعلمات

| Parameter           | النوع                    | الافتراضي                             | الوصف                                                                                             |
| ------------------- | ------------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `task`              | string                   | _(required)_       | What the sub-agent should do                                                                      |
| `التسمية`           | string                   | —                                     | Short label for identification                                                                    |
| `agentId`           | string                   | _(caller's agent)_ | Spawn under a different agent id (must be allowed)                             |
| `النموذج`           | string                   | _(optional)_       | Override the model for this sub-agent                                                             |
| `thinking`          | string                   | _(optional)_       | Override thinking level (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds` | number                   | `0` (no limit)     | Abort the sub-agent after N seconds                                                               |
| `التنظيف`           | `"delete"` \\| `"keep"` | `"keep"`                              | `"delete"` archives immediately after announce                                                    |

### Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

1. Explicit `model` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.model`
3. Global default: `agents.defaults.subagents.model`
4. Target agent’s normal model resolution for that new session

Thinking level is resolved in this order:

1. Explicit `thinking` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.thinking`
3. Global default: `agents.defaults.subagents.thinking`
4. Otherwise no sub-agent-specific thinking override is applied

<Note>
Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
</Note>

### Cross-Agent Spawning

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>
Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.
</Tip>

## Managing Sub-Agents (`/subagents`)

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

| الأمر                                      | الوصف                                                             |
| ------------------------------------------ | ----------------------------------------------------------------- |
| `/subagents list`                          | List all sub-agent runs (active and completed) |
| `/subagents stop <id\\|#\\|all>`         | Stop a running sub-agent                                          |
| `/subagents log <id\\|#> [limit] [tools]` | View sub-agent transcript                                         |
| `/subagents info <id\\|#>`                | Show detailed run metadata                                        |
| `/subagents send <id\\|#> <message>`      | Send a message to a running sub-agent                             |

You can reference sub-agents by list index (`1`, `2`), run id prefix, full session key, or `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">
    ```
    /subagents list
    ```

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

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">
    ```
    /subagents info 1
    ```

    ````
    ```
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
  <Accordion title="Example: view sub-agent log">
    ```
    /subagents log 1 10
    ```

    ````
    Shows the last 10 messages from the sub-agent's transcript. Add `tools` to include tool call messages:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Sends a message into the running sub-agent's session and waits up to 30 seconds for a reply.
    ```

  </Accordion>
</AccordionGroup>

## Announce (How Results Come Back)

When a sub-agent finishes, it goes through an **announce** step:

1. The sub-agent's final reply is captured
2. A summary message is sent to the main agent's session with the result, status, and stats
3. The main agent posts a natural-language summary to your chat

تحافظ ردود الإعلان على توجيه الخيوط/الموضوعات عند توفره (خيوط Slack، موضوعات Telegram، خيوط Matrix).

### Announce Stats

Each announce includes a stats line with:

- Runtime duration
- استخدام الرموز (إدخال/إخراج/إجمالي)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- Session key, session id, and transcript path

### Announce Status

The announce message includes a status derived from the runtime outcome (not from model output):

- **successful completion** (`ok`) — task completed normally
- **error** — task failed (error details in notes)
- **timeout** — task exceeded `runTimeoutSeconds`
- **unknown** — status could not be determined

<Tip>
If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted.
This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
</Tip>

## Tool Policy

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

<AccordionGroup>
  <Accordion title="Default denied tools">
    | Denied tool | Reason |
    |-------------|--------|
    | `sessions_list` | Session management — main agent orchestrates |
    | `sessions_history` | Session management — main agent orchestrates |
    | `sessions_send` | Session management — main agent orchestrates |
    | `sessions_spawn` | No nested fan-out (sub-agents cannot spawn sub-agents) |
    | `gateway` | System admin — dangerous from sub-agent |
    | `agents_list` | System admin |
    | `whatsapp_login` | Interactive setup — not a task |
    | `session_status` | Status/scheduling — main agent coordinates |
    | `cron` | Status/scheduling — main agent coordinates |
    | `memory_search` | Pass relevant info in spawn prompt instead |
    | `memory_get` | Pass relevant info in spawn prompt instead |
  </Accordion>
</AccordionGroup>

### Customizing Sub-Agent Tools

You can further restrict sub-agent tools:

```json5
{
  tools: {
    subagents: {
      tools: {
        // deny always wins over allow
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

To restrict sub-agents to **only** specific tools:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

<Note>
Custom deny entries are **added to** the default deny list. If `allow` is set, only those tools are available (the default deny list still applies on top).
</Note>

## المصادقة

تُحل مصادقة الوكيل الفرعي بحسب **معرّف الوكيل**، وليس بحسب نوع الجلسة:

- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)
- The merge is additive — main profiles are always available as fallbacks

<Note>
Fully isolated auth per sub-agent is not currently supported.
</Note>

## Context and System Prompt

Sub-agents receive a reduced system prompt compared to the main agent:

- **Included:** Tooling, Workspace, Runtime sections, plus `AGENTS.md` and `TOOLS.md`
- **Not included:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

The sub-agent also receives a task-focused system prompt that instructs it to stay focused on the assigned task, complete it, and not act as the main agent.

## Stopping Sub-Agents

| Method                 | Effect                                                                    |
| ---------------------- | ------------------------------------------------------------------------- |
| `/stop` in the chat    | Aborts the main session **and** all active sub-agent runs spawned from it |
| `/subagents stop <id>` | Stops a specific sub-agent without affecting the main session             |
| `runTimeoutSeconds`    | Automatically aborts the sub-agent run after the specified time           |

<Note>
`runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
</Note>

## Full Configuration Example

<Accordion title="Complete sub-agent configuration">
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
```
</Accordion>

## القيود

<Warning>
- **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
</Warning>

## See Also

- [Session Tools](/concepts/session-tool) — details on `sessions_spawn` and other session tools
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — per-agent tool restrictions and sandboxing
- [Configuration](/gateway/configuration) — `agents.defaults.subagents` reference
- [Queue](/concepts/queue) — how the `subagent` lane works
