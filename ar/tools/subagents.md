---
summary: "الوكلاء الفرعيون: إنشاء تشغيلات وكلاء معزولة تُعلن النتائج مرة أخرى إلى قناة الدردشة الخاصة بالطالب"
read_when:
  - تريد تنفيذ عمل في الخلفية/بالتوازي عبر الوكيل
  - تقوم بتغيير سياسة sessions_spawn أو أداة الوكيل الفرعي
title: "الوكلاء الفرعيون"
---

# الوكلاء الفرعيون

الوكلاء الفرعيون هم عمليات وكيل تعمل في الخلفية ويتم إنشاؤها من عملية وكيل موجودة. عند إنشاء وكيل فرعي، يعمل في جلسة معزولة خاصة به، وينفذ عمله، ثم يعلن النتيجة في الدردشة عند الانتهاء.

## أمر بشرطة مائلة

Use the `/subagents` slash command to inspect and control sub-agent runs for the current session:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` يعرض بيانات التشغيل الوصفية (الحالة، الطوابع الزمنية، معرّف الجلسة، مسار السجل، التنظيف).

الأهداف الرئيسية:

- تنفيذ مهام "البحث / المهام الطويلة / الأدوات البطيئة" بشكل متوازٍ دون حظر التشغيل الرئيسي.
- إبقاء الوكلاء الفرعيين معزولين افتراضيًا (فصل الجلسات + عزل اختياري).
- جعل نطاق الأدوات صعب إساءة الاستخدام: لا يحصل الوكلاء الفرعيون على أدوات الجلسة **افتراضيًا**.
- دعم عمق تداخل قابل للتكوين لأنماط orchestrator.

ملاحظة حول التكلفة: لكل وكيل فرعي سياق واستهلاك رموز **خاص به**. بالنسبة للمهام الثقيلة أو المتكررة
قم بتعيين نموذج أقل تكلفة للوكلاء الفرعيين، وأبقِ وكيلك الرئيسي على نموذج بجودة أعلى.
In a multi-agent setup, you can set sub-agent defaults per agent:

## #> [limit] [tools]\`

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- ثم يقوم بتشغيل خطوة إعلان وينشر رد الإعلان في قناة دردشة الجهة الطالبة
- النموذج: اختيار النموذج الافتراضي للوكيل الهدف (ما لم يتم تعيين `subagents.model`) التفكير: لا يوجد تجاوز للوكيل الفرعي (ما لم يتم تعيين `subagents.thinking`)
- Per-agent config: `agents.list[].subagents.thinking`

معلمات الأداة:

- `task`
- _(optional)_
- Spawn under a different agent id (must be allowed)
- Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
- `thinking?` (اختياري؛ يتجاوز مستوى التفكير لتشغيل الوكيل الفرعي)
- Abort the sub-agent after N seconds
- `"delete"` \\

قائمة السماح:

- Per-agent config: `agents.list[].subagents.model` الافتراضي: وكيل الجهة الطالبة فقط.

الاكتشاف:

- Use the `agents_list` tool to discover which agent ids are currently allowed for `sessions_spawn`.

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted.` (نفس المجلد).
- `"delete"` archives immediately after announce
- Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- يتم تطبيق الأرشفة التلقائية بالتساوي على جلسات العمق 1 والعمق 2.

## Stopping Sub-Agents

By default, sub-agents can only spawn under their own agent id. To allow an agent to spawn sub-agents under other agent ids:

### كيفية التمكين

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

### مستويات العمق

| العمق | صيغة مفتاح الجلسة                                                                                                                                                                                                                                                                                                                                                                                                            | الدور                                                           | هل يمكنه الإنشاء؟                |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | -------------------------------- |
| 0     | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | Per-Agent Overrides                                             | دائمًا                           |
| 1     | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | وكيل فرعي (orchestrator عند السماح بالعمق 2) | فقط إذا كان `maxSpawnDepth >= 2` |
| 2     | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Managing Sub-Agents (`/subagents`)           | أبدًا                            |

### سلسلة الإعلان

تتدفق النتائج عائدًا عبر السلسلة:

1. ينهي عامل العمق-2 عمله ← يعلن إلى والده (orchestrator بالعمق-1)
2. يتلقى orchestrator بالعمق-1 الإعلان، ويُركّب النتائج، وينهي العمل ← يعلن إلى الرئيسي
3. يتلقى الوكيل الرئيسي الإعلان ويسلّمه إلى المستخدم

كل مستوى يرى فقط الإعلانات من أبنائه المباشرين.

### Tool Policy

- **العمق 1 (orchestrator، عندما `maxSpawnDepth >= 2`)**: يحصل على `sessions_spawn` و`subagents` و`sessions_list` و`sessions_history` حتى يتمكن من إدارة أبنائه. تبقى أدوات الجلسة/النظام الأخرى مرفوضة.
- **العمق 1 (leaf، عندما `maxSpawnDepth == 1`)**: لا توجد أدوات جلسة (السلوك الافتراضي الحالي).
- **العمق 2 (leaf worker)**: لا توجد أدوات جلسة — يتم دائمًا رفض `sessions_spawn` عند العمق 2. لا يمكنه إنشاء أبناء إضافيين.

### Cross-Agent Spawning

يمكن لكل جلسة وكيل (بأي عمق) أن يكون لديها بحد أقصى `maxChildrenPerAgent` (الافتراضي: 5) من الأبناء النشطين في الوقت نفسه. يمنع هذا الانتشار غير المنضبط الناتج عن مُنسِّق واحد.

### إيقاف متسلسل

إيقاف مُنسِّق بعمق 1 يؤدي تلقائيًا إلى إيقاف جميع العناصر التابعة له بعمق 2:

- `/stop` في الدردشة الرئيسية يوقف جميع الوكلاء بعمق 1 ويتسلسل إلى العناصر التابعة لهم بعمق 2.
- `/subagents stop <id>`
- `/subagents kill all` يوقف جميع الوكلاء الفرعيين للطالب ويتسلسل.

## المصادقة

تُحل مصادقة الوكيل الفرعي بحسب **معرّف الوكيل**، وليس بحسب نوع الجلسة:

- مفتاح جلسة الوكيل الفرعي هو `agent:<agentId>:subagent:<uuid>`.
- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)

The merge is additive — main profiles are always available as fallbacks
Fully isolated auth per sub-agent is not currently supported.

## Announce Status

يقوم الوكلاء الفرعيون بالإبلاغ عبر خطوة إعلان:

- تعمل خطوة الإعلان داخل جلسة الوكيل الفرعي (وليس جلسة الطالب).
- If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted. This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
- وإلا يتم نشر رد الإعلان في قناة دردشة الطالب عبر استدعاء متابعة `agent` (`deliver=true`).
- تحافظ ردود الإعلان على توجيه الخيوط/الموضوعات عند توفره (خيوط Slack، موضوعات Telegram، خيوط Matrix).
- يتم توحيد رسائل الإعلان ضمن قالب ثابت:
  - `Status:` مشتق من نتيجة التشغيل (`success` أو `error` أو `timeout` أو `unknown`).
  - `Result:` محتوى الملخص من خطوة الإعلان (أو `(not available)` إذا لم يكن متوفرًا).
  - `Notes:` تفاصيل الخطأ وسياق مفيد آخر.
- The announce message includes a status derived from the runtime outcome (not from model output):

Each announce includes a stats line with:

- مدة التشغيل (على سبيل المثال، `runtime 5m12s`)
- استخدام الرموز (إدخال/إخراج/إجمالي)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- `sessionKey` و`sessionId` ومسار السجل (بحيث يمكن للوكيل الرئيسي جلب السجل عبر `sessions_history` أو فحص الملف على القرص)

## Customizing Sub-Agent Tools

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

عندما يكون `maxSpawnDepth >= 2`، يتلقى الوكلاء الفرعيون من المستوى 1 (المُنسِّقون) أيضًا `sessions_spawn` و`subagents` و`sessions_list` و`sessions_history` حتى يتمكنوا من إدارة العناصر التابعة لهم.

تجاوز عبر الإعدادات:

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

## التزامن

يستخدم الوكلاء الفرعيون مسار قائمة انتظار مخصصًا داخل العملية:

- اسم المسار: `subagent`
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## الإيقاف

- إرسال `/stop` في دردشة الطالب يؤدي إلى إلغاء جلسة الطالب وإيقاف أي تشغيلات نشطة لوكلاء فرعيين تم إنشاؤها منها، مع التسلسل إلى العناصر المتداخلة.
- Aborts the main session **and** all active sub-agent runs spawned from it

## القيود

- إعلان الوكيل الفرعي هو **بأفضل جهد ممكن**. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- الاستدعاء **غير حاجب** — يحصل الوكيل الرئيسي فورًا على `{ status: "accepted", runId, childSessionKey }`.
- سياق الوكيل الفرعي يحقن فقط `AGENTS.md` + `TOOLS.md` (بدون `SOUL.md` أو `IDENTITY.md` أو `USER.md` أو `HEARTBEAT.md` أو `BOOTSTRAP.md`).
- أقصى عمق للتداخل هو 5 (`maxSpawnDepth` النطاق: 1–5). يوصى بالعمق 2 لمعظم حالات الاستخدام.
- `maxChildrenPerAgent` يحدد الحد الأقصى لعدد العناصر التابعة النشطة لكل جلسة (الافتراضي: 5، النطاق: 1–20).
