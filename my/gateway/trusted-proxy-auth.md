---
summary: "Gateway authentication ကို ယုံကြည်စိတ်ချရသော reverse proxy (Pomerium, Caddy, nginx + OAuth) သို့ လွှဲအပ်ပါ"
read_when:
  - OpenClaw ကို identity-aware proxy နောက်ကွယ်တွင် လည်ပတ်ခြင်း
  - OpenClaw ၏ရှေ့တွင် OAuth ပါဝင်သော Pomerium, Caddy သို့မဟုတ် nginx ကို စနစ်တကျ တပ်ဆင်ခြင်း
  - reverse proxy setup များတွင် ဖြစ်ပေါ်သော WebSocket `1008 unauthorized` အမှားများကို ဖြေရှင်းခြင်း
---

# Trusted Proxy Auth

> ⚠️ **လုံခြုံရေးအရ အရေးကြီးသော feature ဖြစ်သည်။** ဤ mode သည် authentication ကို သင့် reverse proxy ထံသို့ အပြည့်အဝ လွှဲအပ်ထားသည်။ မမှန်ကန်သော configuration သည် သင့် Gateway ကို ခွင့်မပြုထားသော ဝင်ရောက်မှုများအတွက် ဖွင့်လှစ်ပေးနိုင်သည်။ ဖွင့်မီ ဤစာမျက်နှာကို သေချာစွာ ဖတ်ရှုပါ။

## ဘယ်အချိန်တွင် အသုံးပြုသင့်သလဲ

`trusted-proxy` auth mode ကို အောက်ပါအခြေအနေများတွင် အသုံးပြုပါ:

- သင်သည် OpenClaw ကို **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) နောက်ကွယ်တွင် လည်ပတ်နေသည်
- သင့် proxy သည် authentication အားလုံးကို ကိုင်တွယ်ပြီး header များမှတစ်ဆင့် user identity ကို ပို့ပေးသည်
- သင်သည် Kubernetes သို့မဟုတ် container ပတ်ဝန်းကျင်တွင် လည်ပတ်နေပြီး proxy သည် Gateway သို့ ဝင်ရောက်နိုင်သည့် တစ်ခုတည်းသော လမ်းကြောင်းဖြစ်သည်
- browser များက WS payload တွင် token မပို့နိုင်သဖြင့် WebSocket `1008 unauthorized` အမှားများ ကြုံတွေ့နေရသည်

## မအသုံးပြုသင့်သော အခြေအနေများ

- သင့် proxy သည် user များကို authenticate မလုပ်ပါက (TLS terminator သို့မဟုတ် load balancer သာဖြစ်ပါက)
- proxy ကို ကျော်လွှားပြီး Gateway သို့ ဝင်ရောက်နိုင်သော လမ်းကြောင်းရှိပါက (firewall ပေါက်များ၊ internal network access)
- သင့် proxy သည် forwarded headers များကို မှန်ကန်စွာ ဖယ်ရှား/overwrite လုပ်နေသည်ဟု မသေချာပါက
- ကိုယ်ပိုင် တစ်ဦးတည်းအသုံးပြုမှုသာ လိုအပ်ပါက (ပိုမိုရိုးရှင်းသော setup အတွက် Tailscale Serve + loopback ကို စဉ်းစားပါ)

## အလုပ်လုပ်ပုံ

1. သင့် reverse proxy သည် user များကို authenticate လုပ်သည် (OAuth, OIDC, SAML စသည်)
2. proxy သည် authenticate လုပ်ပြီးသော user identity ပါဝင်သည့် header တစ်ခု ထည့်ပေးသည် (ဥပမာ `x-forwarded-user: nick@example.com`)
3. OpenClaw သည် request သည် **trusted proxy IP** ( `gateway.trustedProxies` တွင် configure လုပ်ထားသော) မှ လာကြောင်း စစ်ဆေးသည်
4. OpenClaw သည် configure လုပ်ထားသော header မှ user identity ကို ထုတ်ယူသည်
5. အရာအားလုံး မှန်ကန်ပါက request ကို authorize လုပ်မည်

## ဖွဲ့စည်းမှု

```json5
{
  gateway: {
    // ကွန်ယက် interface သို့ bind လုပ်ရမည် (loopback မဟုတ်ပါ)
    bind: "lan",

    // အရေးကြီးသည် - သင်၏ proxy IP(များ) ကိုသာ ဤနေရာတွင် ထည့်ပါ
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // အတည်ပြုပြီးသော အသုံးပြုသူ၏ identity ပါဝင်သည့် Header (လိုအပ်သည်)
        userHeader: "x-forwarded-user",

        // ရွေးချယ်နိုင်သည် - (proxy စစ်ဆေးရန်) မဖြစ်မနေ ပါရှိရမည့် headers များ
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // ရွေးချယ်နိုင်သည် - သတ်မှတ်ထားသော အသုံးပြုသူများကိုသာ ခွင့်ပြုမည် (empty = အားလုံးခွင့်ပြု)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### ဖွဲ့စည်းမှု အညွှန်း

| အကွက်                                       | လိုအပ်သည် | ဖော်ပြချက်                                                                                                                  |
| ------------------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | ဟုတ်သည်   | ယုံကြည်စိတ်ချရသော proxy IP လိပ်စာများ၏ array ဖြစ်သည်။ အခြား IP များမှ လာသော request များကို ပယ်ချပါသည်။                     |
| `gateway.auth.mode`                         | ဟုတ်သည်   | `"trusted-proxy"` ဖြစ်ရမည်                                                                                                  |
| `gateway.auth.trustedProxy.userHeader`      | ဟုတ်သည်   | အတည်ပြုပြီးသော အသုံးပြုသူ၏ identity ပါဝင်သည့် Header အမည်                                                                   |
| `gateway.auth.trustedProxy.requiredHeaders` | မလိုအပ်ပါ | request ကို ယုံကြည်စိတ်ချနိုင်ရန် မဖြစ်မနေ ပါရှိရမည့် အပို headers များ                                                     |
| `gateway.auth.trustedProxy.allowUsers`      | မလိုအပ်ပါ | ခွင့်ပြုထားသော အသုံးပြုသူ identity များ၏ စာရင်း။ Empty ဖြစ်ပါက အတည်ပြုပြီးသော အသုံးပြုသူအားလုံးကို ခွင့်ပြုသည်ဟု ဆိုလိုသည်။ |

## Proxy Setup နမူနာများ

### Pomerium

Pomerium သည် identity ကို `x-pomerium-claim-email` (သို့မဟုတ် အခြား claim headers များ) တွင် ပို့ဆောင်ပြီး JWT ကို `x-pomerium-jwt-assertion` တွင် ပေးပို့ပါသည်။

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium ၏ IP
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

Pomerium config အပိုင်းနမူနာ -

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

### OAuth ဖြင့် Caddy

`caddy-security` plugin ပါသော Caddy သည် အသုံးပြုသူများကို authenticate လုပ်ပြီး identity headers များကို ပို့ဆောင်နိုင်သည်။

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy ၏ IP (host တူညီပါက)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile အပိုင်းနမူနာ -

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

oauth2-proxy သည် အသုံးပြုသူများကို authenticate လုပ်ပြီး identity ကို `x-auth-request-email` တွင် ပို့ဆောင်ပါသည်။

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

nginx config အပိုင်းနမူနာ -

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

### Forward Auth ဖြင့် Traefik

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

## လုံခြုံရေး စစ်ဆေးရန် စာရင်း

trusted-proxy auth ကို ဖွင့်မီ အောက်ပါအချက်များကို စစ်ဆေးပါ:

- [ ] **Proxy သည် တစ်ခုတည်းသော လမ်းကြောင်း ဖြစ်ရမည်**: Gateway port ကို သင့် proxy မှ လွဲ၍ အခြားအရာအားလုံးမှ firewall ဖြင့် ပိတ်ထားရမည်
- [ ] **trustedProxies ကို အနည်းဆုံးသာ သတ်မှတ်ထားရမည်**: subnet တစ်ခုလုံးမဟုတ်ဘဲ သင့် proxy ၏ အမှန်တကယ် IP များသာ ထည့်ပါ
- [ ] **Proxy သည် headers များကို ဖယ်ရှားပြီး ပြန်ရေးသားရမည်**: client များမှ လာသော `x-forwarded-*` headers များကို append မလုပ်ဘဲ overwrite လုပ်ရမည်
- [ ] **TLS termination**: သင့် proxy က TLS ကို ကိုင်တွယ်ရမည်၊ အသုံးပြုသူများသည် HTTPS မှတစ်ဆင့် ချိတ်ဆက်ရမည်
- [ ] **allowUsers ကို သတ်မှတ်ထားပါ** (အကြံပြုသည်): authentication ပြုလုပ်ထားသူအားလုံးကို ခွင့်ပြုမည့်အစား သိရှိထားသော အသုံးပြုသူများကိုသာ ကန့်သတ်ပါ

## လုံခြုံရေး စစ်ဆေးမှု

`openclaw security audit` သည် trusted-proxy auth ကို **critical** အဆင့် ပြဿနာအဖြစ် သတိပေးမည်။ ဤအရာသည် ရည်ရွယ်ထားသော အပြုအမူဖြစ်သည် — သင့် proxy setup ပေါ်တွင် လုံခြုံရေးကို အပ်နှင်းထားကြောင်း သတိပေးခြင်းဖြစ်သည်။

Audit သည် အောက်ပါအချက်များကို စစ်ဆေးသည်:

- `trustedProxies` configuration မရှိခြင်း
- `userHeader` configuration မရှိခြင်း
- `allowUsers` အလွတ် ဖြစ်ခြင်း (authentication ပြုလုပ်ထားသော မည်သူမဆို ဝင်ရောက်ခွင့်ရှိ)

## ပြဿနာဖြေရှင်းခြင်း

### "trusted_proxy_untrusted_source"

Request သည် `gateway.trustedProxies` ထဲရှိ IP မှ မလာခဲ့ပါ။ စစ်ဆေးပါ:

- Proxy IP မှန်ကန်ပါသလား? (Docker container IP များ ပြောင်းလဲနိုင်သည်)
- သင့် proxy အရှေ့ဘက်တွင် load balancer တစ်ခု ရှိပါသလား?
- အမှန်တကယ် IP များကို ရှာဖွေရန် `docker inspect` သို့မဟုတ် `kubectl get pods -o wide` ကို အသုံးပြုပါ

### "trusted_proxy_user_missing"

User header သည် အလွတ် ဖြစ်နေသည် သို့မဟုတ် မရှိပါ။ စစ်ဆေးပါ:

- သင့် proxy တွင် identity headers များကို ပို့ရန် configure လုပ်ထားပါသလား?
- Header အမည် မှန်ကန်ပါသလား? (case-insensitive ဖြစ်သော်လည်း စာလုံးပေါင်း မှန်ရမည်)
- အသုံးပြုသူသည် proxy တွင် အမှန်တကယ် authentication ပြုလုပ်ထားပါသလား?

### "trusted_proxy_missing_header_\*"

လိုအပ်သော header တစ်ခု မပါရှိပါ။ စစ်ဆေးပါ:

- အဆိုပါ header များအတွက် သင့် proxy configuration
- Chain တစ်လျှောက် တစ်နေရာရာတွင် header များကို ဖယ်ရှားနေပါသလား

### "trusted_proxy_user_not_allowed"

အသုံးပြုသူသည် authentication ပြုလုပ်ထားသော်လည်း `allowUsers` ထဲတွင် မပါရှိပါ။ ၎င်းတို့ကို ထည့်ပါ သို့မဟုတ် allowlist ကို ဖယ်ရှားပါ။

### WebSocket သည် မအောင်မြင်သေးပါ

သင့် proxy တွင် အောက်ပါအချက်များကို သေချာပါ:

- WebSocket အဆင့်မြှင့်တင်မှုများကို ပံ့ပိုးသည် (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket အဆင့်မြှင့်တင်မှု တောင်းဆိုချက်များတွင် (HTTP မသာ) identity headers များကို ပို့ဆောင်ပေးသည်
- WebSocket ချိတ်ဆက်မှုများအတွက် သီးသန့် auth လမ်းကြောင်း မရှိပါ

## Token Auth မှ ပြောင်းရွှေ့ခြင်း

token auth မှ trusted-proxy သို့ ပြောင်းရွှေ့နေပါက:

1. သင့် proxy တွင် အသုံးပြုသူများကို authenticate လုပ်ပြီး headers များကို ပို့ဆောင်ရန် ပြင်ဆင်ပါ
2. proxy setup ကို သီးသန့် စမ်းသပ်ပါ (headers နှင့် curl အသုံးပြု၍)
3. OpenClaw config ကို trusted-proxy auth ဖြင့် အပ်ဒိတ်လုပ်ပါ
4. Gateway ကို ပြန်လည်စတင်ပါ
5. Control UI မှ WebSocket ချိတ်ဆက်မှုများကို စမ်းသပ်ပါ
6. `openclaw security audit` ကို လည်ပတ်ပြီး တွေ့ရှိချက်များကို ပြန်လည်သုံးသပ်ပါ

## ဆက်စပ်သောအကြောင်းအရာများ

- [Security](/gateway/security) — လုံခြုံရေး လမ်းညွှန်အပြည့်အစုံ
- [Configuration](/gateway/configuration) — config ကိုးကားချက်
- [Remote Access](/gateway/remote) — အခြား remote access ပုံစံများ
- [Tailscale](/gateway/tailscale) — tailnet-only အသုံးပြုမှုအတွက် ပိုမိုရိုးရှင်းသော အခြားရွေးချယ်မှု
