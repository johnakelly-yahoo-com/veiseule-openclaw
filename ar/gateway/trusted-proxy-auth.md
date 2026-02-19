---
summary: "فوّض مصادقة gateway إلى reverse proxy موثوق (Pomerium، Caddy، nginx + OAuth)"
read_when:
  - تشغيل OpenClaw خلف proxy واعٍ بالهوية
  - إعداد Pomerium أو Caddy أو nginx مع OAuth أمام OpenClaw
  - إصلاح أخطاء WebSocket 1008 غير المصرح بها في إعدادات reverse proxy
---

# Trusted Proxy Auth

> ⚠️ **ميزة حساسة أمنيًا.** يفوّض هذا الوضع المصادقة بالكامل إلى reverse proxy الخاص بك. قد يؤدي سوء الإعداد إلى تعريض Gateway للوصول غير المصرح به. اقرأ هذه الصفحة بعناية قبل التفعيل.

## متى يتم الاستخدام

استخدم وضع المصادقة `trusted-proxy` عندما:

- تقوم بتشغيل OpenClaw خلف **وكيل مدرك للهوية** (Pomerium، Caddy + OAuth، nginx + oauth2-proxy، Traefik + forward auth)
- يتولى الوكيل جميع عمليات المصادقة ويمرّر هوية المستخدم عبر الرؤوس (headers)
- تعمل ضمن بيئة Kubernetes أو حاويات حيث يكون الوكيل هو المسار الوحيد إلى Gateway
- تواجه أخطاء WebSocket `1008 unauthorized` لأن المتصفحات لا يمكنها تمرير الرموز في حمولة WS

## متى لا يجب استخدامه

- إذا كان الوكيل لا يقوم بمصادقة المستخدمين (مجرد مُنهي TLS أو موزّع أحمال)
- إذا كان هناك أي مسار إلى Gateway يتجاوز الوكيل (ثغرات في الجدار الناري، وصول من الشبكة الداخلية)
- إذا لم تكن متأكدًا مما إذا كان الوكيل لديك يقوم بإزالة/استبدال الرؤوس المُعاد توجيهها بشكل صحيح
- إذا كنت تحتاج فقط إلى وصول شخصي لمستخدم واحد (فكّر في Tailscale Serve + loopback لإعداد أبسط)

## كيف يعمل

1. يقوم الوكيل العكسي بمصادقة المستخدمين (OAuth، OIDC، SAML، إلخ)
2. يضيف الوكيل رأسًا يحتوي على هوية المستخدم المُصادق عليه (مثلًا، `x-forwarded-user: nick@example.com`)
3. يتحقق OpenClaw من أن الطلب جاء من **عنوان IP لوكيل موثوق** (مُحدد في `gateway.trustedProxies`)
4. يستخرج OpenClaw هوية المستخدم من الرأس المُكوَّن
5. إذا كان كل شيء صحيحًا، يتم تفويض الطلب

## الإعداد

```json5
{
  gateway: {
    // Must bind to network interface (not loopback)
    bind: "lan",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### مرجع الإعداد

| الحقل                                       | مطلوب | الوصف                                                                                                                     |
| ------------------------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | نعم   | مصفوفة بعناوين IP الخاصة بالوكلاء الموثوقين. يتم رفض الطلبات القادمة من عناوين IP أخرى.   |
| `gateway.auth.mode`                         | نعم   | يجب أن يكون `"trusted-proxy"`                                                                                             |
| `gateway.auth.trustedProxy.userHeader`      | نعم   | اسم الرأس الذي يحتوي على هوية المستخدم المُصادق عليه                                                                      |
| `gateway.auth.trustedProxy.requiredHeaders` | لا    | رؤوس إضافية يجب أن تكون موجودة حتى يُعتبر الطلب موثوقًا                                                                   |
| `gateway.auth.trustedProxy.allowUsers`      | لا    | قائمة سماح بهويات المستخدمين. القيمة الفارغة تعني السماح لجميع المستخدمين المُصادق عليهم. |

## أمثلة على إعداد الوكيل

### Pomerium

يقوم Pomerium بتمرير هوية المستخدم في `x-pomerium-claim-email` (أو ترويسات claim أخرى) ويمرر رمز JWT في `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // عنوان IP الخاص بـ Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

مقتطف إعداد Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy مع OAuth

يمكن لـ Caddy باستخدام إضافة `caddy-security` مصادقة المستخدمين وتمرير ترويسات الهوية.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // عنوان IP الخاص بـ Caddy (إذا كان على نفس المضيف)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

مقتطف Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

يقوم oauth2-proxy بمصادقة المستخدمين وتمرير الهوية في `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // عنوان IP الخاص بـ nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

مقتطف إعداد nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik مع Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // عنوان IP الخاص بحاوية Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## قائمة التحقق الأمنية

قبل تفعيل مصادقة trusted-proxy، تحقق من:

- [ ] **الوكيل هو المسار الوحيد**: منفذ Gateway محمي بجدار ناري يمنع الوصول من أي جهة باستثناء الوكيل الخاص بك
- [ ] **trustedProxies محددة بأقل قدر ممكن**: فقط عناوين IP الفعلية للوكيل، وليس شبكات فرعية كاملة
- [ ] **الوكيل يزيل الترويسات**: يقوم الوكيل بالكتابة فوق (وليس إضافة) ترويسات `x-forwarded-*` القادمة من العملاء
- [ ] **إنهاء TLS**: يتولى الوكيل معالجة TLS؛ ويتصل المستخدمون عبر HTTPS
- [ ] **تم تعيين allowUsers** (موصى به): تقييد الوصول على مستخدمين معروفين بدلاً من السماح لأي مستخدم تمت مصادقته

## تدقيق أمني

سيقوم الأمر `openclaw security audit` بالإبلاغ عن مصادقة trusted-proxy بدرجة خطورة **حرجة**. هذا مقصود — فهو تذكير بأنك تفوض الأمان إلى إعدادات الوكيل الخاصة بك.

يتحقق التدقيق من:

- غياب إعداد `trustedProxies`
- غياب إعداد `userHeader`
- `allowUsers` فارغة (يسمح لأي مستخدم تمت مصادقته)

## استكشاف الأخطاء وإصلاحها

### "trusted_proxy_untrusted_source"

لم يأتِ الطلب من عنوان IP موجود في `gateway.trustedProxies`. تحقق من:

- هل عنوان IP الخاص بالوكيل صحيح؟ (قد تتغير عناوين IP لحاويات Docker)
- هل يوجد موازن تحميل أمام الوكيل الخاص بك؟
- استخدم `docker inspect` أو `kubectl get pods -o wide` للعثور على عناوين IP الفعلية

### "trusted_proxy_user_missing"

كان ترويسة المستخدم فارغة أو مفقودة. تحقق من:

- هل تم تكوين الوكيل (proxy) لتمرير ترويسات الهوية؟
- هل اسم الترويسة صحيح؟ (غير حساس لحالة الأحرف، لكن يجب أن يكون الإملاء صحيحًا)
- هل المستخدم مُصادق عليه فعليًا في الوكيل؟

### "trusted_proxy_missing_header_\*"

ترويسة مطلوبة غير موجودة. تحقق من:

- إعدادات الوكيل الخاصة بك لتلك الترويسات المحددة
- ما إذا كانت الترويسات يتم حذفها في مكان ما ضمن سلسلة الاتصال

### "trusted_proxy_user_not_allowed"

المستخدم مُصادق عليه ولكن غير موجود في `allowUsers`. إما إضافته أو إزالة قائمة السماح.

### فشل WebSocket لا يزال مستمرًا

تأكد من أن الوكيل الخاص بك:

- يدعم ترقيات WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- يمرر ترويسات الهوية في طلبات ترقية WebSocket (وليس فقط HTTP)
- لا يحتوي على مسار مصادقة منفصل لاتصالات WebSocket

## الانتقال من مصادقة Token

إذا كنت تنتقل من مصادقة token إلى trusted-proxy:

1. قم بتكوين الوكيل لمصادقة المستخدمين وتمرير الترويسات
2. اختبر إعداد الوكيل بشكل مستقل (curl مع الترويسات)
3. حدّث إعدادات OpenClaw لاستخدام مصادقة trusted-proxy
4. أعد تشغيل Gateway
5. اختبر اتصالات WebSocket من واجهة Control UI
6. شغّل `openclaw security audit` وراجع النتائج

## ذات صلة

- [Security](/gateway/security) — دليل الأمان الكامل
- [Configuration](/gateway/configuration) — مرجع الإعدادات
- [Remote Access](/gateway/remote) — أنماط وصول عن بُعد أخرى
- [Tailscale](/gateway/tailscale) — بديل أبسط للوصول ضمن tailnet فقط
