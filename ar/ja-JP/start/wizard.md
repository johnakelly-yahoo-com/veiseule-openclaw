---
read_when:
  - عند تشغيل معالج الإعداد أو أثناء التكوين
  - عند إعداد جهاز جديد
sidebarTitle: Wizard (CLI)
summary: "معالج الإعداد عبر CLI: إعداد تفاعلي لـ Gateway ومساحات العمل والقنوات وSkills"
title: معالج الإعداد (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# معالج الإعداد الأولي (CLI)

يُعد معالج الإعداد الأولي عبر CLI المسار الموصى به لإعداد OpenClaw على macOS وLinux وWindows (عبر WSL2). بالإضافة إلى الاتصال بـ Gateway محلي أو بعيد، يقوم أيضًا بتكوين الإعدادات الافتراضية لمساحة العمل والقنوات وSkills.

```bash
openclaw onboard
```

<Info>
أسرع طريقة لبدء أول محادثة: افتح Control UI (لا يلزم إعداد القنوات). شغّل `openclaw dashboard` للدردشة عبر المتصفح. الوثائق: [Dashboard](/web/dashboard).
</Info>

## البدء السريع مقابل الإعدادات المتقدمة

يبدأ المعالج باختيار **البدء السريع** (إعدادات افتراضية) أو **الإعدادات المتقدمة** (تحكم كامل).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Gateway محلي على loopback
    - مساحة عمل موجودة أو مساحة العمل الافتراضية
    - منفذ Gateway `18789`
    - يتم إنشاء رمز مصادقة Gateway تلقائيًا (حتى على loopback)
    - نشر Tailscale معطل
    - رسائل Telegram وWhatsApp الخاصة (DM) ضمن قائمة السماح افتراضيًا (قد يُطلب إدخال رقم الهاتف)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - عرض تدفق المطالبات الكامل للوضع، ومساحة العمل، وGateway، والقنوات، والخدمات، وSkills
  
</Tab>
</Tabs>

## تفاصيل الإعداد الأولي عبر CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    شرح كامل للتدفقات المحلية والبعيدة، ومصفوفة المصادقة والنماذج، ومخرجات الإعداد، وRPC الخاص بالمعالج، وسلوك signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    وصفات للإعداد الأولي غير التفاعلي وأمثلة مؤتمتة لـ `agents add`.
  
</Card>
</Columns>

## أوامر المتابعة الشائعة الاستخدام

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` لا يعني الوضع غير التفاعلي. في السكربتات استخدم `--non-interactive`.
</Note>

<Tip>
موصى به: قم بتعيين مفتاح Brave Search API لتمكين الوكيل من استخدام `web_search` (يعمل `web_fetch` بدون مفتاح). أسهل طريقة: شغّل `openclaw configure --section web` لحفظ `tools.web.search.apiKey`. الوثائق: [أدوات الويب](/tools/web).
</Tip>

## وثائق ذات صلة

- مرجع أوامر CLI: [`openclaw onboard`](/cli/onboard)
- الإعداد الأولي لتطبيق macOS: [الإعداد الأولي](/start/onboarding)
- خطوات التشغيل الأولي للوكيل: [التهيئة الأولية للوكيل](/start/bootstrapping)
