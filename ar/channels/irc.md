---
title: IRC
description: Connect OpenClaw to IRC channels and direct messages.
---

Use IRC when you want OpenClaw in classic channels (`#room`) and direct messages.
IRC ships as an extension plugin, but it is configured in the main config under `channels.irc`.

## البدء السريع

1. قم بتمكين إعدادات IRC في `~/.openclaw/openclaw.json`.
2. قم بتعيين ما لا يقل عن:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. ابدأ/أعد تشغيل البوابة:

```bash
openclaw gateway run
```

## إعدادات الأمان الافتراضية

- القيمة الافتراضية لـ `channels.irc.dmPolicy` هي `"pairing"`.
- القيمة الافتراضية لـ `channels.irc.groupPolicy` هي `"allowlist"`.
- عند تعيين `groupPolicy="allowlist"`، قم بتحديد `channels.irc.groups` لتعريف القنوات المسموح بها.
- استخدم TLS (`channels.irc.tls=true`) ما لم تكن تقبل عمداً النقل بنص غير مشفر.

## التحكم في الوصول

توجد «بوابتان» منفصلتان لقنوات IRC:

1. **الوصول إلى القناة** (`groupPolicy` + `groups`): ما إذا كان البوت يقبل الرسائل من القناة أساساً.
2. **وصول المُرسِل** (`groupAllowFrom` / لكل قناة `groups["#channel"].allowFrom`): من يُسمح له بتفعيل البوت داخل تلك القناة.

مفاتيح الإعداد:

- قائمة السماح للرسائل الخاصة DM (وصول مُرسِل DM): `channels.irc.allowFrom`
- قائمة السماح لمرسلي المجموعات (وصول مُرسِل القناة): `channels.irc.groupAllowFrom`
- عناصر التحكم لكل قناة (القناة + المُرسِل + قواعد الإشارة): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` يسمح بالقنوات غير المُعدّة (**لا يزال يتطلب الإشارة افتراضياً**)

يمكن أن تستخدم عناصر قائمة السماح اسم المستخدم (nick) أو الصيغة `nick!user@host`.

### مشكلة شائعة: `allowFrom` مخصص للرسائل الخاصة DMs وليس للقنوات

إذا رأيت سجلات مثل:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…فهذا يعني أن المُرسِل غير مسموح له بإرسال رسائل **المجموعة/القناة**. قم بإصلاح ذلك إما عن طريق:

- تعيين `channels.irc.groupAllowFrom` (عام لجميع القنوات)، أو
- تعيين قوائم سماح للمرسلين لكل قناة: `channels.irc.groups["#channel"].allowFrom`

مثال (السماح لأي شخص في `#tuirc-dev` بالتحدث إلى البوت):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## تفعيل الردود (الإشارات)

حتى إذا كانت القناة مسموحاً بها (عبر `groupPolicy` + `groups`) وكان المُرسِل مسموحاً به، فإن OpenClaw يعتمد افتراضياً **اشتراط الإشارة** في سياقات المجموعات.

هذا يعني أنك قد ترى سجلات مثل `drop channel … (missing-mention)` ما لم تتضمن الرسالة نمط إشارة يطابق البوت.

لجعل البوت يرد في قناة IRC **دون الحاجة إلى إشارة**، قم بتعطيل اشتراط الإشارة لتلك القناة:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

أو للسماح **بجميع** قنوات IRC (بدون قائمة سماح لكل قناة) مع الرد دون إشارات:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## ملاحظة أمنية (موصى بها للقنوات العامة)

إذا سمحت بـ `allowFrom: ["*"]` في قناة عامة، فيمكن لأي شخص إرسال أوامر إلى البوت.
لتقليل المخاطر، قم بتقييد الأدوات لتلك القناة.

### نفس الأدوات للجميع في القناة

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### أدوات مختلفة لكل مُرسل (المالك يحصل على صلاحيات أكبر)

استخدم `toolsBySender` لتطبيق سياسة أكثر صرامة على `"*"` وسياسة أكثر مرونة على اسمك المستعار:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

ملاحظات:

- يمكن أن تكون مفاتيح `toolsBySender` اسمًا مستعارًا (مثل `"eigen"`) أو hostmask كاملًا (`"eigen!~eigen@174.127.248.171"`) لمطابقة هوية أقوى.
- تُطبَّق أول سياسة مطابقة للمُرسل؛ ويُستخدم `"*"` كخيار احتياطي عام (wildcard).

لمزيد من المعلومات حول الوصول إلى المجموعات مقابل تقييد الإشارات (وكيفية تفاعلهما)، راجع: [/channels/groups](/channels/groups).

## NickServ

للتعريف باستخدام NickServ بعد الاتصال:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

تسجيل اختياري لمرة واحدة عند الاتصال:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

قم بتعطيل `register` بعد تسجيل الاسم المستعار لتجنب محاولات REGISTER المتكررة.

## متغيرات البيئة

يدعم الحساب الافتراضي:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (مفصولة بفواصل)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## استكشاف الأخطاء وإصلاحها

- إذا اتصل البوت لكنه لا يرد أبدًا في القنوات، فتحقق من `channels.irc.groups` **وأيضًا** مما إذا كان تقييد الإشارات يحذف الرسائل (`missing-mention`). إذا كنت تريده أن يرد دون إشارات (pings)، فعيّن `requireMention:false` للقناة.
- إذا فشل تسجيل الدخول، فتحقق من توفر الاسم المستعار وكلمة مرور الخادم.
- إذا فشل TLS على شبكة مخصصة، فتحقق من إعدادات المضيف/المنفذ وإعداد الشهادة.

