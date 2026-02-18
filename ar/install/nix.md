---
title: "Nix"
---

# تثبيت Nix

الطريقة الموصى بها لتشغيل OpenClaw باستخدام Nix هي عبر **[nix-openclaw](https://github.com/openclaw/nix-openclaw)** — وحدة Home Manager متكاملة «تشمل كل ما يلزم».

## البدء السريع

الصق هذا في وكيل الذكاء الاصطناعي لديك (Claude، Cursor، إلخ):

```text
I want to set up nix-openclaw on my Mac.
Repository: github:openclaw/nix-openclaw

What I need you to do:
1. Check if Determinate Nix is installed (if not, install it)
2. Create a local flake at ~/code/openclaw-local using templates/agent-first/flake.nix
3. Help me create a Telegram bot (@BotFather) and get my chat ID (@userinfobot)
4. Set up secrets (bot token, Anthropic key) - plain files at ~/.secrets/ is fine
5. Fill in the template placeholders and run home-manager switch
6. Verify: launchd running, bot responds to messages

Reference the nix-openclaw README for module options.
```

> **📦 الدليل الكامل: [github.com/openclaw/nix-openclaw](https://github.com/openclaw/nix-openclaw)**
>
> يُعد مستودع nix-openclaw المصدر المعتمد لتثبيت Nix. هذه الصفحة مجرد نظرة سريعة.

## ما الذي ستحصل عليه

- Gateway (البوابة) + تطبيق macOS + أدوات (whisper، spotify، cameras) — جميعها مُثبّتة الإصدارات
- خدمة Launchd تستمر عبر عمليات إعادة التشغيل
- نظام إضافات مع تهيئة تصريحية
- تراجع فوري: `home-manager switch --rollback`

---

## سلوك وقت التشغيل في وضع Nix

عند تعيين `OPENCLAW_NIX_MODE=1` (يتم تلقائيًا مع nix-openclaw):

يدعم OpenClaw **وضع Nix** الذي يجعل التهيئة حتمية ويعطّل تدفقات التثبيت التلقائي.
يمكنك تمكينه عبر التصدير:

```bash
OPENCLAW_NIX_MODE=1
```

على macOS، لا يرث تطبيق الواجهة الرسومية تلقائيًا متغيرات بيئة الصدفة. يمكنك
أيضًا تمكين وضع Nix عبر defaults:

```bash
defaults write bot.molt.mac openclaw.nixMode -bool true
```

### مسارات التهيئة والحالة

يقرأ OpenClaw تهيئة JSON5 من `OPENCLAW_CONFIG_PATH` ويخزّن البيانات القابلة للتغيير في `OPENCLAW_STATE_DIR`.
19. عند الحاجة، يمكنك أيضاً تعيين `OPENCLAW_HOME` للتحكم في دليل المنزل الأساسي المستخدم لحل المسارات الداخلية.

- 20. `OPENCLAW_HOME` (أولوية افتراضية: `HOME` / `USERPROFILE` / `os.homedir()`)
- `OPENCLAW_STATE_DIR` (الافتراضي: `~/.openclaw`)
- `OPENCLAW_CONFIG_PATH` (الافتراضي: `$OPENCLAW_STATE_DIR/openclaw.json`)

عند التشغيل تحت Nix، اضبط هذه القيم صراحةً إلى مواقع مُدارة بواسطة Nix بحيث تبقى حالة وقت التشغيل والتهيئة
خارج المخزن غير القابل للتغيير.

### سلوك وقت التشغيل في وضع Nix

- تعطيل تدفقات التثبيت التلقائي والتحوير الذاتي
- إظهار رسائل معالجة خاصة بـ Nix عند غياب الاعتمادات
- تعرض الواجهة شريط وضع Nix للقراءة فقط عند توفره

## ملاحظة التعبئة (macOS)

يتوقع مسار تعبئة macOS قالب Info.plist ثابتًا في:

```
apps/macos/Sources/OpenClaw/Resources/Info.plist
```

يقوم [`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) بنسخ هذا القالب إلى حزمة التطبيق وترقيع الحقول الديناميكية
(معرّف الحزمة، الإصدار/البناء، Git SHA، مفاتيح Sparkle). يحافظ ذلك على حتمية ملف plist لتعبئة SwiftPM
وبُنى Nix (التي لا تعتمد على سلسلة أدوات Xcode كاملة).

## ذو صلة

- [nix-openclaw](https://github.com/openclaw/nix-openclaw) — دليل الإعداد الكامل
- [Wizard](/start/wizard) — إعداد CLI غير قائم على Nix
- [Docker](/install/docker) — إعداد مُحَوْسَب بالحاويات
