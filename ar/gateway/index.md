---
summary: "دليل تشغيل لخدمة Gateway ودورة حياتها وعملياتها"
read_when:
  - عند تشغيل عملية Gateway أو تصحيح أخطائها
title: "دليل تشغيل Gateway"
---

# دليل تشغيل خدمة Gateway

```
تشخيصات تبدأ من الأعراض مع سلاسل أوامر دقيقة وتواقيع السجلات.
```

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    دليل إعداد موجه حسب المهام + مرجع إعدادات كامل.
   
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Task-oriented setup guide + full configuration reference.
  
</Card>
</CardGroup>

## تشغيل محلي خلال 5 دقائق

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
Linux: `openclaw-gateway-<profile>.service`
```

خط أساس سليم: `Runtime: running` و `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
تراقب إعادة تحميل إعدادات Gateway مسار ملف الإعداد النشط (المحدد من افتراضات الملف الشخصي/الحالة، أو `OPENCLAW_CONFIG_PATH` عند تعيينه).
الوضع الافتراضي: `gateway.reload.mode="hybrid"` (تطبيق فوري للتغييرات الآمنة، وإعادة تشغيل عند الحرجة).
</Note>

## نموذج وقت التشغيل

- عملية واحدة تعمل دائمًا للتوجيه، ومستوى التحكم، واتصالات القنوات.
- تعدد الإرسال على منفذ واحد.
  - WebSocket للتحكم/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - واجهة تحكم ويب وخطافات
- وضع الربط الافتراضي: `loopback`.
- يتطلب توثيق Gateway افتراضيًا: اضبط `gateway.auth.token` (أو `OPENCLAW_GATEWAY_TOKEN`) أو `gateway.auth.password`.

### أولوية المنفذ والربط

| الإعداد                                                                                    | ترتيب الحل                                                                                                              |
| ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| المنفذ الأساسي = `gateway.port` (أو `OPENCLAW_GATEWAY_PORT` / `--port`) | أسبقية المنفذ: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > الافتراضي `18789`. |
| وضع الربط                                                                                  | CLI/override ← `gateway.bind` ← `loopback`                                                                              |

### أوضاع إعادة التحميل السريع

| التعطيل باستخدام `gateway.reload.mode="off"`. | سلوك الإبقاء حيًا                                  |
| ------------------------------------------------------------- | -------------------------------------------------- |
| `off`                                                         | بدون إعادة تحميل للإعدادات                         |
| `hot`                                                         | تطبيق التغييرات الآمنة للتحميل السريع فقط          |
| `restart`                                                     | إعادة التشغيل عند التغييرات التي تتطلب إعادة تحميل |
| `hybrid` (الافتراضي)                       | تطبيق سريع عند الأمان، وإعادة تشغيل عند الحاجة     |

## مجموعة أوامر المشغّل

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## الوصول عن بُعد

يُفضَّل Tailscale/VPN؛ وإلا فنفق SSH:
خيار احتياطي: نفق SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

يتصل العملاء بعد ذلك بـ `ws://127.0.0.1:18789` عبر النفق.

<Warning>
إذا تمت تهيئة مصادقة gateway، فلا يزال يتعين على العملاء إرسال بيانات المصادقة (`token`/`password`) حتى عبر أنفاق SSH.
</Warning>

انظر: [Remote Gateway](/gateway/remote)، [Authentication](/gateway/authentication)، [Tailscale](/gateway/tailscale).

## الإشراف ودورة حياة الخدمة

استخدم التشغيل تحت الإشراف لموثوقية شبيهة ببيئات الإنتاج.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

تسميات LaunchAgent هي `ai.openclaw.gateway` (الافتراضي) أو `ai.openclaw.<profile>` (ملف شخصي مُسمّى). يقوم `openclaw doctor` بتدقيق وإصلاح انحراف إعدادات الخدمة.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

فعّل lingering (مطلوب كي تبقى خدمة المستخدم بعد تسجيل الخروج/الخمول):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

استخدم وحدة نظام للمضيفين متعددي المستخدمين/الدائمين التشغيل.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## بوابات متعددة (على المضيف نفسه)

غالبًا غير ضروري: يمكن لـ Gateway واحدة خدمة قنوات مراسلة ووكلاء متعددين.
استخدم بوابات متعددة فقط للتكرار أو العزل الصارم (مثل: روبوت إنقاذ).

Checklist per instance:

- `gateway.port` فريد
- `OPENCLAW_CONFIG_PATH` فريد
- `OPENCLAW_STATE_DIR` فريد
- `agents.defaults.workspace` فريد

مثال:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

الدليل الكامل: [بوابات متعددة](/gateway/multiple-gateways).

### مسار سريع لملف تعريف التطوير

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

تتضمن الإعدادات الافتراضية حالة/إعدادات معزولة ومنفذ gateway أساسي `19001`.

## البروتوكول (منظور المشغّل)

- يجب أن يكون أول إطار من العميل هو `connect`.
- يعيد Gateway لقطة `hello-ok` (`presence`، `health`، `stateVersion`، `uptimeMs`، الحدود/السياسة).
- الطلبات: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- الأحداث الشائعة: `connect.challenge`، `agent`، `chat`، `presence`، `tick`، `health`، `heartbeat`، `shutdown`.

تشغيل الوكيل يتم على مرحلتين:

1. إقرار فوري بالقبول (`status:"accepted"`)
2. استجابات `agent` على مرحلتين: أولًا تأكيد `res` `{runId,status:"accepted"}`، ثم `res` `{runId,status:"ok"|"error",summary}` النهائي بعد انتهاء التشغيل؛ ويصل الخرج المتدفق كـ `event:"agent"`.

الوثائق الكاملة: [بروتوكول Gateway](/gateway/protocol) و[بروتوكول Bridge (قديم)](/gateway/bridge-protocol).

## فحوصات تشغيلية

### التحقق من الحيوية (Liveness)

- الحيوية: افتح WS وأرسل `req:connect` → توقّع `res` مع `payload.type="hello-ok"` (مع لقطة).
- توقّع استجابة `hello-ok` مع لقطة (snapshot).

### الجاهزية

```bash
`openclaw gateway health|status` — طلب الصحة/الحالة عبر WS الخاص بـ Gateway.
```

### استعادة الفجوات

لا تتم إعادة تشغيل الأحداث. عند وجود فجوات في التسلسل، حدّث الحالة (`health`, `system-presence`) قبل المتابعة.

## أنماط الأعطال الشائعة

| النمط                                                          | المشكلة المحتملة                                   |
| -------------------------------------------------------------- | -------------------------------------------------- |
| `refusing to bind gateway ... `without auth\`                  | محاولة الربط بواجهة غير loopback دون رمز/كلمة مرور |
| `another gateway instance is already listening` / `EADDRINUSE` | تعارض في المنفذ                                    |
| `Gateway start blocked: set gateway.mode=local`                | تم ضبط الإعداد على وضع remote                      |
| `unauthorized` أثناء الاتصال                                   | عدم تطابق المصادقة بين العميل وGateway             |

للحصول على مسارات تشخيص كاملة، استخدم [Gateway Troubleshooting](/gateway/troubleshooting).

## ضمانات السلامة

- لا يوجد مسار بديل لاتصالات Baileys المباشرة؛ إذا كانت Gateway متوقفة، تفشل عمليات الإرسال سريعًا.
- تُرفَض الإطارات الأولى غير المتصلة أو JSON المشوّه ويُغلق المقبس.
- إيقاف رشيق: بث حدث `shutdown` قبل الإغلاق؛ يجب على العملاء التعامل مع الإغلاق + إعادة الاتصال.

---

ذات صلة:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

