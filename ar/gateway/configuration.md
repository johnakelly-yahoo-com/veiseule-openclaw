---
title: "التهيئة"
---

# التهيئة

يقرأ OpenClaw ملف تهيئة اختياري بصيغة <Tooltip tip="JSON5 يدعم التعليقات والفواصل اللاحقة">**JSON5**</Tooltip> من `~/.openclaw/openclaw.json`.

إذا كان الملف مفقودًا، يستخدم OpenClaw إعدادات افتراضية آمنة. من الأسباب الشائعة لإضافة ملف تهيئة:

- ربط القنوات والتحكّم في من يمكنه مراسلة البوت
- تعيين النماذج، الأدوات، العزل (sandboxing)، أو الأتمتة (cron، hooks)
- ضبط الجلسات، الوسائط، الشبكات، أو واجهة المستخدم

اطّلع على [المرجع الكامل](/gateway/configuration-reference) لجميع الحقول المتاحة.

<Tip>
**جديد على التهيئة؟** ابدأ بالأمر `openclaw onboard` للإعداد التفاعلي، أو راجع دليل [أمثلة التهيئة](/gateway/configuration-examples) للحصول على إعدادات كاملة جاهزة للنسخ.
</Tip>

## أقل تهيئة ممكنة

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## تعديل التهيئة

<Tabs>
  <Tab title="المعالج التفاعلي">
    ```bash
    openclaw onboard       # معالج الإعداد الكامل
    openclaw configure     # معالج التهيئة
    ```
  </Tab>
  <Tab title="CLI (أوامر سريعة)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="واجهة التحكم">
    افتح [http://127.0.0.1:18789](http://127.0.0.1:18789) واستخدم تبويب **Config**.
    تقوم واجهة التحكم بإنشاء نموذج من مخطط التهيئة، مع محرّر **Raw JSON** كخيار متقدم.
  </Tab>
  <Tab title="تعديل مباشر">
    عدّل الملف `~/.openclaw/openclaw.json` مباشرةً. تقوم البوابة بمراقبة الملف وتطبيق التغييرات تلقائيًا (انظر [إعادة التحميل الساخن](#config-hot-reload)).
  </Tab>
</Tabs>

## التحقق الصارم

<Warning>
يقبل OpenClaw فقط التهيئات التي تطابق المخطط بالكامل. المفاتيح غير المعروفة، أو الأنواع غير الصحيحة، أو القيم غير الصالحة تجعل البوابة **ترفض البدء**. الاستثناء الوحيد على مستوى الجذر هو `$schema` (من نوع string) لتمكين المحررات من إرفاق بيانات JSON Schema.
</Warning>

عند فشل التحقق:

- لا تبدأ البوابة
- تعمل فقط أوامر التشخيص (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- شغّل `openclaw doctor` لرؤية الأخطاء بالتفصيل
- شغّل `openclaw doctor --fix` (أو `--yes`) لتطبيق الإصلاحات

## مهام شائعة

<AccordionGroup>
  <Accordion title="إعداد قناة (WhatsApp، Telegram، Discord، إلخ)">
    لكل قناة قسم خاص بها تحت `channels.<provider>`. راجع صفحة القناة المخصّصة لخطوات الإعداد:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    تشترك جميع القنوات في نفس نمط سياسة الرسائل الخاصة (DM):

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="اختيار وتكوين النماذج">
    عيّن النموذج الأساسي مع بدائل احتياطية:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - يعرّف `agents.defaults.models` كتالوج النماذج ويعمل كقائمة سماح لأمر `/model`.
    - تستخدم مراجع النماذج صيغة `provider/model` (مثال: `anthropic/claude-opus-4-6`).
    - راجع [Models CLI](/concepts/models) لتبديل النماذج داخل الدردشة و[Model Failover](/concepts/model-failover) لسلوك التبديل الاحتياطي.
    - لمزوّدين مخصّصين/مستضافين ذاتيًا، راجع [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls).

  </Accordion>

  <Accordion title="التحكم في من يمكنه مراسلة البوت">
    يتم التحكم في الوصول إلى الرسائل الخاصة لكل قناة عبر `dmPolicy`:

    - `"pairing"` (افتراضي): يحصل المرسلون غير المعروفين على رمز اقتران لمرة واحدة
    - `"allowlist"`: فقط المرسلون ضمن `allowFrom` (أو مخزن الاقتران)
    - `"open"`: السماح بجميع الرسائل الخاصة الواردة (يتطلب `allowFrom: ["*"]`)
    - `"disabled"`: تجاهل جميع الرسائل الخاصة

    للمجموعات، استخدم `groupPolicy` مع `groupAllowFrom` أو قوائم سماح خاصة بالقناة.

    راجع [المرجع الكامل](/gateway/configuration-reference#dm-and-group-access) للتفاصيل.

  </Accordion>

  <Accordion title="بوابة الإشارات في دردشات المجموعات">
    رسائل المجموعات تتطلب الإشارة افتراضيًا. قم بالتهيئة لكل وكيل:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **إشارات وصفية**: إشارات @ الأصلية في المنصة
    - **أنماط نصية**: تعابير regex في `mentionPatterns`
    - راجع [المرجع الكامل](/gateway/configuration-reference#group-chat-mention-gating).

  </Accordion>

  <Accordion title="تهيئة الجلسات وإعادة الضبط">
    تتحكم الجلسات في استمرارية المحادثة:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - راجع [Session Management](/concepts/session) و[المرجع الكامل](/gateway/configuration-reference#session).

  </Accordion>

  <Accordion title="تفعيل العزل (Sandboxing)">
    تشغيل جلسات الوكيل داخل حاويات Docker معزولة:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    أنشئ الصورة أولًا: `scripts/sandbox-setup.sh`

    راجع [Sandboxing](/gateway/sandboxing) و[المرجع الكامل](/gateway/configuration-reference#sandbox).

  </Accordion>

  <Accordion title="إعداد heartbeat (فحوصات دورية)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: مدة مثل `30m` أو `2h` (استخدم `0m` للتعطيل)
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - راجع [Heartbeat](/gateway/heartbeat).

  </Accordion>

  <Accordion title="إعداد مهام Cron">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    راجع [Cron jobs](/automation/cron-jobs).

  </Accordion>

  <Accordion title="إعداد Webhooks (Hooks)">
    تفعيل نقاط HTTP في البوابة:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    راجع [المرجع الكامل](/gateway/configuration-reference#hooks).

  </Accordion>

  <Accordion title="توجيه متعدد الوكلاء">
    تشغيل عدة وكلاء معزولين:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    راجع [Multi-Agent](/concepts/multi-agent) و[المرجع الكامل](/gateway/configuration-reference#multi-agent-routing).

  </Accordion>

  <Accordion title="تقسيم التهيئة إلى عدة ملفات (`$include`)">
    استخدم `$include` لتنظيم التهيئات الكبيرة:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **ملف واحد**: يستبدل الكائن الحاوي
    - **مصفوفة ملفات**: دمج عميق حسب الترتيب
    - **مفاتيح شقيقة**: تُدمج بعد التضمين
    - **تضمينات متداخلة**: حتى 10 مستويات
    - **مسارات نسبية**: نسبةً إلى الملف المُضمِّن
    - **معالجة الأخطاء**: رسائل واضحة للملفات المفقودة أو الدائرية

  </Accordion>
</AccordionGroup>

## إعادة التحميل الساخن للتهيئة

تراقب البوابة الملف `~/.openclaw/openclaw.json` وتطبّق التغييرات تلقائيًا — دون الحاجة لإعادة تشغيل يدوي في معظم الحالات.

### أوضاع إعادة التحميل

| الوضع                  | السلوك                                                                 |
| ---------------------- | ---------------------------------------------------------------------- |
| **`hybrid`** (افتراضي) | يطبّق التغييرات الآمنة فورًا ويعيد التشغيل تلقائيًا عند الحاجة      |
| **`hot`**              | يطبّق التغييرات الآمنة فقط ويسجّل تحذيرًا عند الحاجة لإعادة التشغيل |
| **`restart`**          | يعيد تشغيل البوابة عند أي تغيير                                       |
| **`off`**              | يعطّل المراقبة؛ تسري التغييرات بعد إعادة تشغيل يدوي                 |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

<Note>
`gateway.reload` و `gateway.remote` لا يتطلبان إعادة تشغيل.
</Note>

## متغيرات البيئة

يقرأ OpenClaw متغيرات البيئة من العملية الأب، بالإضافة إلى:

- `.env` في دليل العمل الحالي (إن وُجد)
- `~/.openclaw/.env` (احتياطي عام)

لا يتجاوز أي منهما القيم الموجودة مسبقًا. يمكن أيضًا تحديد متغيرات ضمن التهيئة:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="استيراد متغيرات من الصدفة (اختياري)">
  عند التفعيل، وإذا لم تكن المفاتيح المطلوبة مضبوطة، يشغّل OpenClaw صدفة تسجيل الدخول ويستورد المفاتيح المفقودة فقط:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

المكافئ عبر البيئة: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="استبدال متغيرات البيئة داخل التهيئة">
  يمكنك الإشارة إلى متغيرات البيئة داخل أي قيمة نصية باستخدام `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

القواعد:

- يُطابق فقط الأسماء الكبيرة: `[A-Z_][A-Z0-9_]*`
- المتغيرات المفقودة أو الفارغة تسبب خطأ عند التحميل
- استخدم `$${VAR}` لإخراج حرفي
- يعمل داخل ملفات `$include`
- مثال: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

راجع [Environment](/help/environment) للتفاصيل الكاملة.

## المرجع الكامل

للاطلاع على شرح تفصيلي لكل حقل، راجع **[Configuration Reference](/gateway/configuration-reference)**.

---

_روابط ذات صلة: [أمثلة التهيئة](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
