---
summary: "شغّل OpenClaw داخل حاوية Podman بدون صلاحيات root"
read_when:
  - تريد بوابة مُحاوية باستخدام Podman بدلًا من Docker
title: "Podman"
---

# Podman

شغّل بوابة OpenClaw داخل حاوية Podman **بدون صلاحيات root**. يستخدم نفس الصورة الخاصة بـ Docker (قم بالبناء من المستودع [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## المتطلبات

- Podman (بدون صلاحيات root)
- Sudo للإعداد لمرة واحدة (إنشاء مستخدم، بناء الصورة)

## البدء السريع

**1. إعداد لمرة واحدة** (من جذر المستودع؛ ينشئ المستخدم، ويبني الصورة، ويثبت سكربت التشغيل):

```bash
./setup-podman.sh
```

يؤدي هذا أيضًا إلى إنشاء ملف `~openclaw/.openclaw/openclaw.json` بسيط (يضبط `gateway.mode="local"`) حتى يتمكن Gateway من البدء دون تشغيل معالج الإعداد.

بشكل افتراضي، لا يتم تثبيت الحاوية كخدمة systemd، بل تقوم بتشغيلها يدويًا (انظر أدناه). لإعداد بنمط الإنتاج مع بدء تلقائي وإعادة تشغيل، قم بتثبيتها كخدمة مستخدم systemd Quadlet بدلًا من ذلك:

```bash
./setup-podman.sh --quadlet
```

(أو عيّن `OPENCLAW_PODMAN_QUADLET=1`؛ استخدم `--container` لتثبيت الحاوية وسكربت التشغيل فقط.)

**2. تشغيل Gateway** (يدويًا، لاختبار سريع):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. معالج الإعداد** (على سبيل المثال لإضافة قنوات أو مزودين):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

ثم افتح `http://127.0.0.1:18789/` واستخدم الرمز من `~openclaw/.openclaw/.env` (أو القيمة التي طُبعت بواسطة الإعداد).

## Systemd (Quadlet، اختياري)

إذا قمت بتشغيل `./setup-podman.sh --quadlet` (أو `OPENCLAW_PODMAN_QUADLET=1`)، فسيتم تثبيت وحدة [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) بحيث يعمل Gateway كخدمة مستخدم systemd لمستخدم openclaw. يتم تمكين الخدمة وبدؤها في نهاية الإعداد.

- **بدء:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **إيقاف:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **الحالة:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **السجلات:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

يوجد ملف quadlet في `~openclaw/.config/containers/systemd/openclaw.container`. لتغيير المنافذ أو متغيرات البيئة، عدّل ذلك الملف (أو ملف `.env` الذي يستورده)، ثم نفّذ `sudo systemctl --machine openclaw@ --user daemon-reload` وأعد تشغيل الخدمة. عند الإقلاع، تبدأ الخدمة تلقائيًا إذا كان lingering مفعّلًا لمستخدم openclaw (يقوم الإعداد بذلك عند توفر loginctl).

لإضافة quadlet **بعد** إعداد أولي لم يستخدمه، أعد التشغيل: `./setup-podman.sh --quadlet`.

## مستخدم openclaw (بدون تسجيل دخول)

يقوم `setup-podman.sh` بإنشاء مستخدم نظام مخصص باسم `openclaw`:

- **Shell:** `nologin` — لا يوجد تسجيل دخول تفاعلي؛ مما يقلل من سطح الهجوم.

- **Home:** على سبيل المثال `/home/openclaw` — يحتوي على `~/.openclaw` (الإعدادات، مساحة العمل) وسكربت التشغيل `run-openclaw-podman.sh`.

- **Rootless Podman:** يجب أن يكون لدى المستخدم نطاق **subuid** و **subgid**. تقوم العديد من التوزيعات بتعيين هذه القيم تلقائيًا عند إنشاء المستخدم. إذا طبع الإعداد تحذيرًا، فأضف الأسطر التالية إلى `/etc/subuid` و `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  ثم شغّل Gateway باسم ذلك المستخدم (على سبيل المثال من cron أو systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **الإعدادات:** يمكن فقط للمستخدم `openclaw` و root الوصول إلى `/home/openclaw/.openclaw`. لتعديل الإعدادات: استخدم واجهة التحكم (Control UI) بعد تشغيل gateway، أو `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## البيئة والإعدادات

- **الرمز (Token):** يتم تخزينه في `~openclaw/.openclaw/.env` باسم `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` و `run-openclaw-podman.sh` ينشئانه إذا لم يكن موجودًا (باستخدام `openssl` أو `python3` أو `od`).
- **اختياري:** في ملف `.env` هذا يمكنك تعيين مفاتيح المزود (مثل `GROQ_API_KEY` و `OLLAMA_API_KEY`) ومتغيرات البيئة الأخرى الخاصة بـ OpenClaw.
- **منافذ المضيف (Host):** بشكل افتراضي يقوم السكربت بربط `18789` (gateway) و `18790` (bridge). يمكنك تجاوز تعيين منفذ **المضيف** باستخدام `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` و `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` عند التشغيل.
- **المسارات:** يتم تعيين إعدادات المضيف ومساحة العمل افتراضيًا إلى `~openclaw/.openclaw` و `~openclaw/.openclaw/workspace`. يمكنك تجاوز مسارات المضيف المستخدمة في سكربت التشغيل باستخدام `OPENCLAW_CONFIG_DIR` و `OPENCLAW_WORKSPACE_DIR`.

## أوامر مفيدة

- **السجلات (Logs):** باستخدام quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. باستخدام السكربت: `sudo -u openclaw podman logs -f openclaw`
- **الإيقاف (Stop):** باستخدام quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. باستخدام السكربت: `sudo -u openclaw podman stop openclaw`
- **التشغيل مرة أخرى:** باستخدام quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. باستخدام السكربت: أعد تشغيل سكربت الإطلاق أو `podman start openclaw`
- **إزالة الحاوية (Container):** `sudo -u openclaw podman rm -f openclaw` — سيتم الاحتفاظ بالإعدادات ومساحة العمل على المضيف

## استكشاف الأخطاء وإصلاحها

- **تم رفض الإذن (EACCES) على config أو auth-profiles:** تستخدم الحاوية افتراضيًا `--userns=keep-id` وتعمل بنفس uid/gid الخاص بمستخدم المضيف الذي يشغّل السكربت. تأكد من أن `OPENCLAW_CONFIG_DIR` و `OPENCLAW_WORKSPACE_DIR` على المضيف مملوكان لذلك المستخدم.
- **تم منع تشغيل Gateway (غياب `gateway.mode=local`):** تأكد من وجود `~openclaw/.openclaw/openclaw.json` وأنه يعيّن `gateway.mode="local"`. `setup-podman.sh` ينشئ هذا الملف إذا لم يكن موجودًا.
- **فشل Rootless Podman لمستخدم openclaw:** تحقق من أن `/etc/subuid` و `/etc/subgid` يحتويان على سطر للمستخدم `openclaw` (مثل `openclaw:100000:65536`). أضفه إذا كان مفقودًا ثم أعد التشغيل.
- **اسم الحاوية قيد الاستخدام:** يستخدم سكربت الإطلاق `podman run --replace`، لذلك يتم استبدال الحاوية الحالية عند بدء التشغيل مرة أخرى. للتنظيف يدويًا: `podman rm -f openclaw`.
- **لم يتم العثور على السكربت عند التشغيل كمستخدم openclaw:** تأكد من تشغيل `setup-podman.sh` بحيث يتم نسخ `run-openclaw-podman.sh` إلى مجلد home الخاص بـ openclaw (مثل `/home/openclaw/run-openclaw-podman.sh`).
- **خدمة Quadlet غير موجودة أو تفشل في البدء:** شغّل `sudo systemctl --machine openclaw@ --user daemon-reload` بعد تعديل ملف `.container`. يتطلب Quadlet cgroups v2: يجب أن يعرض `podman info --format '{{.Host.CgroupsVersion}}'` القيمة `2`.

## اختياري: التشغيل كمستخدمك الخاص

لتشغيل gateway كمستخدمك العادي (بدون مستخدم openclaw مخصص): قم ببناء الصورة، وأنشئ `~/.openclaw/.env` يحتوي على `OPENCLAW_GATEWAY_TOKEN`، ثم شغّل الحاوية باستخدام `--userns=keep-id` مع ربط المجلدات إلى `~/.openclaw` الخاص بك. تم تصميم سكربت الإطلاق لآلية مستخدم openclaw؛ لإعداد مستخدم واحد يمكنك بدلاً من ذلك تشغيل أمر `podman run` من السكربت يدويًا، مع توجيه config ومساحة العمل إلى مجلد home الخاص بك. موصى به لمعظم المستخدمين: استخدم `setup-podman.sh` وشغّل كـمستخدم openclaw بحيث تكون الإعدادات والعملية معزولة.
