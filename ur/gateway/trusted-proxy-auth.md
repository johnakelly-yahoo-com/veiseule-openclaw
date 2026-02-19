---
summary: "Gateway کی توثیق کو کسی قابلِ اعتماد reverse proxy (Pomerium, Caddy, nginx + OAuth) کے سپرد کریں"
read_when:
  - OpenClaw کو ایک identity-aware proxy کے پیچھے چلانا
  - OpenClaw کے سامنے Pomerium، Caddy، یا nginx کو OAuth کے ساتھ سیٹ اپ کرنا
  - reverse proxy سیٹ اپ کے ساتھ WebSocket 1008 غیر مجاز غلطیوں کو درست کرنا
---

# Trusted Proxy Auth

> ⚠️ **سیکیورٹی سے متعلق حساس فیچر۔** یہ موڈ مکمل طور پر توثیق کو آپ کے reverse proxy کے سپرد کرتا ہے۔ غلط کنفیگریشن آپ کے Gateway کو غیر مجاز رسائی کے لیے بے نقاب کر سکتی ہے۔ فعال کرنے سے پہلے اس صفحے کو غور سے پڑھیں۔

## کب استعمال کریں

`trusted-proxy` auth موڈ استعمال کریں جب:

- آپ OpenClaw کو ایک **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) کے پیچھے چلا رہے ہوں
- آپ کا proxy تمام توثیق سنبھالتا ہے اور ہیڈرز کے ذریعے صارف کی شناخت آگے بھیجتا ہے
- آپ Kubernetes یا کنٹینر ماحول میں ہیں جہاں Gateway تک رسائی کا واحد راستہ proxy ہے
- آپ کو WebSocket `1008 unauthorized` غلطیاں آ رہی ہیں کیونکہ براؤزر WS payload میں ٹوکن نہیں بھیج سکتے

## کب استعمال نہ کریں

- اگر آپ کا proxy صارفین کی توثیق نہیں کرتا (صرف TLS ختم کرنے والا یا load balancer ہے)
- اگر Gateway تک کوئی ایسا راستہ موجود ہے جو proxy کو بائی پاس کرتا ہو (فائر وال کی خامیاں، اندرونی نیٹ ورک رسائی)
- اگر آپ کو یقین نہیں کہ آپ کا proxy forwarded headers کو درست طریقے سے ہٹاتا/اوور رائٹ کرتا ہے
- اگر آپ کو صرف ذاتی واحد صارف رسائی درکار ہے (آسان سیٹ اپ کے لیے Tailscale Serve + loopback پر غور کریں)

## یہ کیسے کام کرتا ہے

1. آپ کا reverse proxy صارفین کی توثیق کرتا ہے (OAuth, OIDC, SAML, وغیرہ)
2. Proxy مستند شدہ صارف کی شناخت کے ساتھ ایک ہیڈر شامل کرتا ہے (مثلاً، `x-forwarded-user: nick@example.com`)
3. OpenClaw تصدیق کرتا ہے کہ درخواست ایک **trusted proxy IP** سے آئی ہے (جو `gateway.trustedProxies` میں کنفیگر ہوتی ہے)
4. OpenClaw کنفیگر کردہ ہیڈر سے صارف کی شناخت حاصل کرتا ہے
5. اگر سب کچھ درست ہو تو درخواست مجاز قرار دی جاتی ہے

## کنفیگریشن

```json5
{
  gateway: {
    // نیٹ ورک انٹرفیس سے بائنڈ ہونا ضروری ہے (لوپ بیک نہیں)
    bind: "lan",

    // نہایت اہم: یہاں صرف اپنے proxy کے IP ایڈریس شامل کریں
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // مستند شدہ صارف کی شناخت رکھنے والا ہیڈر (لازمی)
        userHeader: "x-forwarded-user",

        // اختیاری: ایسے ہیڈرز جو لازمی موجود ہوں (proxy کی تصدیق)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // اختیاری: مخصوص صارفین تک محدود کریں (خالی = سب کو اجازت)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### کنفیگریشن حوالہ

| فیلڈ                                        | لازمی | تفصیل                                                                                               |
| ------------------------------------------- | ----- | --------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | ہاں   | بھروسہ کرنے کے لیے پراکسی IP ایڈریسز کی فہرست۔ دیگر IPs سے آنے والی درخواستیں مسترد کر دی جاتی ہیں۔ |
| `gateway.auth.mode`                         | ہاں   | ضروری ہے کہ یہ `"trusted-proxy"` ہو                                                                 |
| `gateway.auth.trustedProxy.userHeader`      | ہاں   | وہ ہیڈر نام جس میں تصدیق شدہ صارف کی شناخت شامل ہو                                                  |
| `gateway.auth.trustedProxy.requiredHeaders` | نہیں  | اضافی ہیڈرز جو درخواست کے قابلِ اعتماد ہونے کے لیے موجود ہونا لازمی ہیں                             |
| `gateway.auth.trustedProxy.allowUsers`      | نہیں  | صارف کی شناختوں کی اجازت شدہ فہرست۔ خالی ہونے کا مطلب ہے کہ تمام تصدیق شدہ صارفین کو اجازت ہے۔      |

## پراکسی سیٹ اپ کی مثالیں

### Pomerium

Pomerium شناخت کو `x-pomerium-claim-email` (یا دیگر claim ہیڈرز) میں اور ایک JWT کو `x-pomerium-jwt-assertion` میں بھیجتا ہے۔

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
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

Pomerium کنفیگ کا حصہ:

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

### OAuth کے ساتھ Caddy

`caddy-security` پلگ ان کے ساتھ Caddy صارفین کی تصدیق کر سکتا ہے اور شناختی ہیڈرز بھیج سکتا ہے۔

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile کا حصہ:

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

oauth2-proxy صارفین کی تصدیق کرتا ہے اور شناخت کو `x-auth-request-email` میں بھیجتا ہے۔

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx کنفیگ کا حصہ:

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

### Forward Auth کے ساتھ Traefik

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## سیکیورٹی چیک لسٹ

trusted-proxy auth کو فعال کرنے سے پہلے، تصدیق کریں:

- [ ] **پراکسی ہی واحد راستہ ہو**: Gateway پورٹ آپ کی پراکسی کے علاوہ سب کے لیے فائر وال سے محفوظ ہو
- [ ] **trustedProxies کم سے کم ہوں**: صرف آپ کی اصل پراکسی IPs شامل ہوں، مکمل سب نیٹس نہیں
- [ ] **پراکسی ہیڈرز کو ہٹا دے**: آپ کی پراکسی کلائنٹس سے آنے والے `x-forwarded-*` ہیڈرز کو اوور رائٹ کرے (ضم نہ کرے)
- [ ] **TLS termination**: آپ کی پراکسی TLS سنبھالے؛ صارفین HTTPS کے ذریعے جڑیں
- [ ] **allowUsers سیٹ ہو** (تجویز کردہ): کسی بھی تصدیق شدہ شخص کو اجازت دینے کے بجائے معلوم صارفین تک محدود کریں

## سیکیورٹی آڈٹ

`openclaw security audit` قابلِ اعتماد پراکسی auth کو **critical** شدت کے ساتھ نشان زد کرے گا۔ یہ جان بوجھ کر کیا گیا ہے — یہ اس بات کی یاد دہانی ہے کہ آپ سیکیورٹی کو اپنی پراکسی سیٹ اپ کے حوالے کر رہے ہیں۔

آڈٹ درج ذیل چیزوں کی جانچ کرتا ہے:

- `trustedProxies` کنفیگریشن کی عدم موجودگی
- `userHeader` کنفیگریشن کی عدم موجودگی
- خالی `allowUsers` (کسی بھی مستند صارف کو اجازت دیتا ہے)

## مسائل کا حل

### "trusted_proxy_untrusted_source"

درخواست `gateway.trustedProxies` میں موجود IP سے نہیں آئی۔ چیک کریں:

- کیا پراکسی IP درست ہے؟ (Docker کنٹینر کے IP تبدیل ہو سکتے ہیں)
- کیا آپ کی پراکسی کے سامنے کوئی لوڈ بیلنسر ہے؟
- حقیقی IPs معلوم کرنے کے لیے `docker inspect` یا `kubectl get pods -o wide` استعمال کریں

### "trusted_proxy_user_missing"

صارف کا ہیڈر خالی تھا یا موجود نہیں تھا۔ چیک کریں:

- کیا آپ کی پراکسی شناختی ہیڈرز پاس کرنے کے لیے کنفیگر ہے؟
- کیا ہیڈر کا نام درست ہے؟ (کیس حساس نہیں ہے، لیکن ہجے درست ہونا ضروری ہے)
- کیا صارف واقعی پراکسی پر مستند ہے؟

### "trusted_proxy_missing_header_\*"

ایک مطلوبہ ہیڈر موجود نہیں تھا۔ چیک کریں:

- ان مخصوص ہیڈرز کے لیے اپنی پراکسی کنفیگریشن
- کیا چین میں کہیں ہیڈرز کو ہٹایا تو نہیں جا رہا

### "trusted_proxy_user_not_allowed"

صارف مستند ہے لیکن `allowUsers` میں شامل نہیں ہے۔ یا تو انہیں شامل کریں یا allowlist کو ہٹا دیں۔

### WebSocket اب بھی ناکام ہو رہا ہے

یقینی بنائیں کہ آپ کی پراکسی:

- WebSocket اپ گریڈز کو سپورٹ کرتی ہو (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket اپ گریڈ درخواستوں پر شناختی ہیڈرز پاس کرتی ہو (صرف HTTP نہیں)
- WebSocket کنکشنز کے لیے علیحدہ auth راستہ نہ رکھتی ہو

## Token Auth سے منتقلی

اگر آپ token auth سے trusted-proxy کی طرف جا رہے ہیں:

1. اپنی پراکسی کو صارفین کی توثیق کرنے اور ہیڈرز پاس کرنے کے لیے کنفیگر کریں
2. پراکسی سیٹ اپ کو علیحدہ طور پر ٹیسٹ کریں (ہیڈرز کے ساتھ curl)
3. OpenClaw کنفیگ کو trusted-proxy auth کے ساتھ اپڈیٹ کریں
4. Gateway کو دوبارہ شروع کریں
5. Control UI سے WebSocket کنکشنز کی جانچ کریں
6. `openclaw security audit` چلائیں اور نتائج کا جائزہ لیں

## متعلقہ

- [Security](/gateway/security) — مکمل سیکیورٹی گائیڈ
- [Configuration](/gateway/configuration) — کنفیگریشن ریفرنس
- [Remote Access](/gateway/remote) — دیگر ریموٹ ایکسیس طریقے
- [Tailscale](/gateway/tailscale) — صرف tailnet ایکسیس کے لیے آسان متبادل

