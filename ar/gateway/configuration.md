---
summary: "نظرة عامة على الإعداد: المهام الشائعة، الإعداد السريع، وروابط إلى المرجع الكامل"
read_when:
  - إعداد OpenClaw لأول مرة
  - تبحث عن أنماط إعداد شائعة
  - الانتقال إلى أقسام الإعدادات المحددة
title: "التهيئة"
---

# التهيئة 🔧

يقرأ OpenClaw ملف تهيئة **JSON5** اختياريًا من `~/.openclaw/openclaw.json` (تُسمح التعليقات والفواصل اللاحقة).

إذا كان الملف مفقودًا، يستخدم OpenClaw إعدادات افتراضية «آمنة إلى حدٍّ ما» (وكيل Pi مُضمَّن + جلسات لكل مُرسِل + مساحة عمل `~/.openclaw/workspace`). عادةً لا تحتاج إلى ملف تهيئة إلا من أجل:

- ربط القنوات والتحكم في من يمكنه مراسلة البوت
- تعيين النماذج والأدوات ووضع الحماية (sandboxing) أو الأتمتة (cron، hooks)
- ضبط الجلسات والوسائط والشبكات أو واجهة المستخدم

اطّلع على [المرجع الكامل](/gateway/configuration-reference) لكل حقل متاح.

<Tip>
**جديد على التهيئة؟** اطّلع على دليل [أمثلة التهيئة](/gateway/configuration-examples) للحصول على أمثلة كاملة مع شروح مفصّلة!
</Tip>

## إعدادات بسيطة

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## تحرير الإعدادات

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # معالج الإعداد الكامل
    openclaw configure     # معالج الإعدادات
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    افتح [http://127.0.0.1:18789](http://127.0.0.1:18789) واستخدم علامة التبويب **Config**.
    تعرض واجهة التحكم نموذجًا مبنيًا على هذا المخطط، مع محرّر **Raw JSON** كخيار طوارئ.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (أو `OPENCLAW_CONFIG_PATH`) يراقب Gateway الملف ويطبّق التغييرات تلقائيًا (راجع [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## تحقق صارم

<Warning>
لا يقبل OpenClaw إلا التهيئات التي تطابق المخطط بالكامل. المفاتيح غير المعروفة، أو الأنواع غير الصحيحة، أو القيم غير الصالحة تجعل Gateway **يرفض البدء** حفاظًا على الأمان. الاستثناء الوحيد على مستوى الجذر هو `$schema` (string)، بحيث يمكن للمحررات إرفاق بيانات JSON Schema الوصفية.
</Warning>

عند فشل التحقّق:

- لا يبدأ Gateway.
- يُسمح فقط بأوامر التشخيص (على سبيل المثال: `openclaw doctor`، `openclaw logs`، `openclaw health`، `openclaw status`، `openclaw service`، `openclaw help`).
- شغّل `openclaw doctor` لعرض المشكلات بدقّة.
- شغّل `openclaw doctor --fix` (أو `--yes`) لتطبيق عمليات الترحيل/الإصلاح.

## المهام الشائعة

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    لكل قناة قسم إعدادات خاص بها ضمن `channels.<provider>`.\`. راجع صفحة القناة المخصصة لمعرفة خطوات الإعداد:

    ```
    2. {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">
    عيّن النموذج الأساسي وبدائل اختيارية:

    ````
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
    
    - يحدد `agents.defaults.models` كتالوج النماذج ويعمل كقائمة سماح لأمر `/model`.
    - تستخدم مراجع النماذج تنسيق `provider/model` (مثل `anthropic/claude-opus-4-6`).
    - راجع [Models CLI](/concepts/models) لتبديل النماذج داخل الدردشة و[Model Failover](/concepts/model-failover) لتناوب المصادقة وسلوك التبديل الاحتياطي.
    - لمزوّدي الخدمة المخصصين/ذوي الاستضافة الذاتية، راجع [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) في المرجع.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">تجاوز لكل DM: `channels.<provider>

    ```
    - `"pairing"` (الافتراضي): يحصل المرسلون غير المعروفين على رمز اقتران لمرة واحدة للموافقة
    - `"allowlist"`: فقط المرسلون المدرجون في `allowFrom` (أو مخزن السماح المقترن)
    - `"open"`: السماح بجميع الرسائل الخاصة الواردة (يتطلب `allowFrom: ["*"]`)
    - `"disabled"`: تجاهل جميع الرسائل الخاصة
    
    بالنسبة للمجموعات، استخدم `groupPolicy` + `groupAllowFrom` أو قوائم السماح الخاصة بالقناة.
    
    راجع [المرجع الكامل](/gateway/configuration-reference#dm-and-group-access) للتفاصيل الخاصة بكل قناة.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    تفترض رسائل المجموعات افتراضيًا **اشتراط الإشارة** (إشارة وصفية أو أنماط regex). إعدادات `agents.defaults.subagents` الافتراضية:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">جلسة مناقشة منعزلة:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // موصى به لعدة مستخدمين
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: ‏`main` (مشتركة) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - راجع [Session Management](/concepts/session) لمعرفة النطاق وروابط الهوية وسياسة الإرسال.
    - راجع [المرجع الكامل](/gateway/configuration-reference#session) لجميع الحقول.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">
    تشغيل جلسات الوكيل داخل حاويات Docker معزولة:


    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">
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


    ```
    - `every`: سلسلة مدة (`30m`، `2h`). عيّن `0m` للتعطيل.
    - `target`: ‏`last` | `whatsapp` | `telegram` | `discord` | `none`
    - راجع [Heartbeat](/gateway/heartbeat) للدليل الكامل.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    انظر [Cron jobs](/automation/cron-jobs) للحصول على لمحة عامة عن الميزة وأمثلة CLI.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">تمكين نقطة نهاية HTTP بسيطة على خادم HTTP.

    ```
    34. {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/.openclaw/hooks",
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            wakeMode: "now",
            name: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            model: "openai/gpt-5.2-mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">
    تشغيل عدة وكلاء معزولين بمساحات عمل وجلسات منفصلة:


    ```
    5. {
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
      channels: {
        whatsapp: {
          accounts: {
            personal: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">
    استخدم `$include` لتنظيم ملفات الإعدادات الكبيرة:


    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## إعادة تحميل الإعدادات تلقائيًا

يراقب Gateway الملف `~/.openclaw/openclaw.json` ويطبّق التغييرات تلقائيًا — لا حاجة لإعادة التشغيل يدويًا لمعظم الإعدادات.

### أوضاع إعادة التحميل

| Bind modes:                 | سلوك الدمج                                                                                                                                                                                          |
| ------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (الافتراضي) | يطبّق التغييرات الآمنة فورًا. يعيد التشغيل تلقائيًا للتغييرات الحرجة.                                                                                               |
| **`hot`**                                   | "ساخن": فقط تطبيق التغييرات الآمنة الساخنة؛ سجل عندما يتطلب الأمر إعادة التشغيل. يسجّل تحذيرًا عند الحاجة إلى إعادة التشغيل — وتتولى أنت تنفيذ ذلك. |
| **`restart`**                               | `إعادة التشغيل`: إعادة تشغيل البوابة على أي تغيير في الإعداد.                                                                                                       |
| `{provider}`                                | يعطّل مراقبة الملف. تدخل التغييرات حيز التنفيذ عند إعادة التشغيل اليدوية التالية.                                                                                   |

```json5
{
  بوابة: {
    إعادة التحميل: {
      mode: "هجين",
      debounceMs: 300,
    },
  },
}
```

### ما الذي يُطبَّق فورًا وما الذي يتطلب إعادة تشغيل

تُطبَّق معظم الحقول فورًا دون توقف. في وضع `hybrid`، تتم معالجة التغييرات التي تتطلب إعادة تشغيل تلقائيًا.

| الفئة                                               | الحقول:                                                                  | هل يلزم إعادة التشغيل؟                |
| --------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------------------------------- |
| القنوات                                             | `channels.*`, `web` (WhatsApp) — جميع القنوات المدمجة وقنوات الإضافات | لا                                    |
| الوكيل والنماذج                                     | `agent`, `agents`, `models`, `routing`                                                   | لا                                    |
| الأتمتة                                             | `hooks`, `cron`, `agent.heartbeat`                                                       | لا                                    |
| messages                                            | `messages.inbound`                                                                       | لا                                    |
| الأدوات والوسائط                                    | `tools`, `browser`, `skills`, `audio`, `talk`                                            | لا                                    |
| واجهة المستخدم وأخرى                                | `ui`, `logging`, `identity`, `bindings`                                                  | لا                                    |
| `بوابة` (وضع خادم Gateway + ربط) | `gateway` (port/bind/auth/control UI/tailscale)                       | **الاستبدال الضمني:** |
| البنية التحتية                                      | `discovery`, `canvasHost`, `plugins`                                                     | **أنواع الإشارات:**   |

<Note>
`gateway.reload` و `gateway.remote` حالتان استثنائيتان — تغييرهما **لا** يؤدي إلى إعادة التشغيل.

- يتم دمج الكائنات بشكل تكراري
- `null` يحذف مفتاحًا
- يتم استبدال المصفوفات

المعاملات:

- `raw` (string) — ‏JSON5 يحتوي فقط على المفاتيح المطلوب تغييرها
- `baseHash` (مطلوب) — تجزئة الإعدادات من `config.get`
- `sessionKey`, `note`, `restartDelayMs` — نفس القيم في `config.apply`

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```
</Note>

## التحديثات الجزئية (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">استخدم `config.apply` للتحقّق وكتابة التهيئة كاملةً وإعادة تشغيل Gateway في خطوة واحدة.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">استخدم `config.patch` لدمج تحديث جزئي في التهيئة الحالية دون الكتابة فوق
المفاتيح غير ذات الصلة. يطبّق دلالات JSON merge patch:

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## المتغير

`~/.openclaw/.env` (قيمة احتياطية عامة)

- `.env` من دليل العمل الحالي (إن وُجد)
- يمكنك أيضًا تعيين متغيرات بيئة مضمنة داخل الإعدادات:

لا يتجاوز ملف `env` ملف env الموجود حالياً.
الإشارة إلى متغيرات البيئة في أي قيمة نصية ضمن الإعدادات باستخدام `${VAR_NAME}`:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

<Accordion title="Shell env import (optional)">خيار ملائم بالاشتراك: إذا كان مفعّلًا ولم تُضبط أي من المفاتيح المتوقعة بعد، يشغّل OpenClaw صدفة تسجيل الدخول لديك ويستورد فقط المفاتيح المتوقعة المفقودة (ولا يتجاوز القيم). هذا يعادل فعليًا استيراد ملف تعريف الصدفة لديك.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">المتغيرات المفقودة/الفارغة تؤدي إلى ظهور خطأ عند وقت التحميل

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**القواعد:**

- لا تُطابِق إلا أسماء متغيرات البيئة بالحروف الكبيرة: `[A-Z_][A-Z0-9_]*`
- الاستبدال المضمن: `"${BASE}/v1"` → `"https://api.example.com/v1"`
- الهروب باستخدام `$${VAR}` لإخراج `${VAR}` حرفيًا
- تعمل مع `$include` (تُطبّق الاستبدالات أيضًا على الملفات المُضمَّنة)
- المرجع الكامل

</Accordion>

انظر [/environment](/help/environment) للاطّلاع على الأسبقية الكاملة والمصادر.

## للاطلاع على المرجع الكامل لكل حقل، راجع **[Configuration Reference](/gateway/configuration-reference)**.

ملاحظات الأمان:

---

مثال (قائمة السماح لموفر/نموذج محدد):
