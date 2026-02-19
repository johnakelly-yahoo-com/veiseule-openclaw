---
summary: "Gateway autentifikatsiyasini ishonchli reverse proxy’ga topshiring (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - OpenClaw’ni identity-aware proxy ortida ishga tushirish
  - OpenClaw oldida OAuth bilan Pomerium, Caddy yoki nginx’ni sozlash
  - Reverse proxy sozlamalarida WebSocket 1008 unauthorized xatolarini tuzatish
---

# Trusted Proxy Auth

> ⚠️ **Xavfsizlikka sezgir funksiya.** Ushbu rejim autentifikatsiyani to‘liq reverse proxy’ga topshiradi. Noto‘g‘ri sozlash Gateway’ni ruxsatsiz kirishlarga ochib qo‘yishi mumkin. Yoqishdan oldin ushbu sahifani diqqat bilan o‘qing.

## Qachon foydalanish kerak

`trusted-proxy` autentifikatsiya rejimidan quyidagi hollarda foydalaning:

- OpenClaw’ni **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) ortida ishga tushirsangiz
- Proxy barcha autentifikatsiyani boshqaradi va foydalanuvchi identifikatsiyasini sarlavhalar orqali uzatadi
- Siz Kubernetes yoki konteyner muhitidasiz va proxy Gateway’ga yagona kirish yo‘li bo‘lsa
- Brauzerlar WS payload’larida token uzata olmagani sababli WebSocket `1008 unauthorized` xatolariga duch kelayotgan bo‘lsangiz

## Qachon foydalanmaslik kerak

- Agar proksingiz foydalanuvchilarni autentifikatsiya qilmasa (faqat TLS terminatori yoki load balancer bo‘lsa)
- Agar Gateway’ga proksini chetlab o‘tuvchi biror yo‘l mavjud bo‘lsa (firewall’dagi ochiq joylar, ichki tarmoqdan kirish)
- Agar proksingiz forwarded header’larni to‘g‘ri olib tashlayotgani yoki ustiga yozayotganiga ishonchingiz komil bo‘lmasa
- Agar sizga faqat bitta foydalanuvchi uchun shaxsiy kirish kerak bo‘lsa (soddaroq sozlash uchun Tailscale Serve + loopback’ni ko‘rib chiqing)

## Qanday ishlaydi

1. Sizning reverse proxy’ingiz foydalanuvchilarni autentifikatsiya qiladi (OAuth, OIDC, SAML va boshqalar)
2. Proksi autentifikatsiyadan o‘tgan foydalanuvchi identifikatorini o‘z ichiga olgan header qo‘shadi (masalan, `x-forwarded-user: nick@example.com`)
3. OpenClaw so‘rov **ishonchli proksi IP manzilidan** kelganini tekshiradi (`gateway.trustedProxies` ichida sozlangan)
4. OpenClaw foydalanuvchi identifikatorini sozlangan header’dan ajratib oladi
5. Agar hammasi to‘g‘ri bo‘lsa, so‘rovga ruxsat beriladi

## Sozlash

```json5
{
  gateway: {
    // Tarmoq interfeysiga bog‘lanishi kerak (loopback emas)
    bind: "lan",

    // MUHIM: Bu yerga faqat proksingizning IP manzil(lar)i qo‘shing
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Autentifikatsiyadan o‘tgan foydalanuvchi identifikatori joylashgan header (majburiy)
        userHeader: "x-forwarded-user",

        // Ixtiyoriy: albatta mavjud bo‘lishi kerak bo‘lgan header’lar (proksini tekshirish)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Ixtiyoriy: faqat ma’lum foydalanuvchilarga ruxsat berish (bo‘sh = barchaga ruxsat)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Sozlamalar bo‘yicha ma’lumotnoma

| Maydon                                      | Majburiy | Tavsif                                                                                                                                                                        |
| ------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Ha       | Ishonch bildiriladigan proksi IP manzillari ro‘yxati. Boshqa IP manzillardan kelgan so‘rovlar rad etiladi.                                    |
| `gateway.auth.mode`                         | Ha       | Qiymati `"trusted-proxy"` bo‘lishi kerak                                                                                                                                      |
| `gateway.auth.trustedProxy.userHeader`      | Ha       | Autentifikatsiyadan o‘tgan foydalanuvchi identifikatori joylashgan header nomi                                                                                                |
| `gateway.auth.trustedProxy.requiredHeaders` | Yo‘q     | So‘rov ishonchli deb qabul qilinishi uchun mavjud bo‘lishi shart bo‘lgan qo‘shimcha header’lar                                                                                |
| `gateway.auth.trustedProxy.allowUsers`      | Yo‘q     | Ruxsat berilgan foydalanuvchi identifikatorlari ro‘yxati. Bo‘sh bo‘lsa, autentifikatsiyadan o‘tgan barcha foydalanuvchilarga ruxsat beriladi. |

## Proksi sozlash misollari

### Pomerium

Pomerium identifikatorni `x-pomerium-claim-email` (yoki boshqa claim header’lari) orqali va JWT’ni `x-pomerium-jwt-assertion` orqali uzatadi.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium IP manzili
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

Pomerium konfiguratsiya namunasi:

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

### OAuth bilan Caddy

`caddy-security` plaginiga ega Caddy foydalanuvchilarni autentifikatsiya qilishi va identifikatsiya sarlavhalarini uzatishi mumkin.

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

Caddyfile namunasi:

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

oauth2-proxy foydalanuvchilarni autentifikatsiya qiladi va identifikatsiyani `x-auth-request-email` orqali uzatadi.

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

nginx konfiguratsiya namunasi:

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

### Forward Auth bilan Traefik

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

## Xavfsizlik bo‘yicha tekshiruv ro‘yxati

trusted-proxy autentifikatsiyasini yoqishdan oldin quyidagilarni tekshiring:

- [ ] **Proxy yagona kirish yo‘li**: Gateway porti proxy’dan tashqari hamma uchun firewall bilan yopilgan bo‘lishi kerak
- [ ] **trustedProxies minimal bo‘lsin**: Butun subnetlar emas, faqat haqiqiy proxy IP manzillari
- [ ] **Proxy sarlavhalarni tozalaydi**: Proxy mijozlardan kelgan `x-forwarded-*` sarlavhalarini qo‘shmaydi, balki ustiga yozadi
- [ ] **TLS yakunlanishi**: Proxy TLS’ni boshqaradi; foydalanuvchilar HTTPS orqali ulanadi
- [ ] **allowUsers o‘rnatilgan** (tavsiya etiladi): Autentifikatsiyadan o‘tgan istalgan foydalanuvchiga ruxsat berish o‘rniga faqat ma’lum foydalanuvchilarni cheklang

## Xavfsizlik auditi

`openclaw security audit` trusted-proxy autentifikatsiyasini **kritik** darajadagi muammo sifatida belgilaydi. Bu ataylab qilingan — bu siz xavfsizlikni proxy sozlamalaringizga topshirayotganingizni eslatib turadi.

Audit quyidagilarni tekshiradi:

- `trustedProxies` konfiguratsiyasi mavjud emas
- `userHeader` konfiguratsiyasi mavjud emas
- Bo‘sh `allowUsers` (har qanday autentifikatsiyadan o‘tgan foydalanuvchiga ruxsat beradi)

## Muammolarni bartaraf etish

### trusted_proxy_untrusted_source

So‘rov `gateway.trustedProxies` ichidagi IP manzildan kelmagan. Tekshiring:

- Proxy IP manzili to‘g‘rimi? (Docker konteyner IP manzillari o‘zgarishi mumkin)
- Proxy oldida load balancer bormi?
- Haqiqiy IP manzillarni aniqlash uchun `docker inspect` yoki `kubectl get pods -o wide` dan foydalaning

### trusted_proxy_user_missing

Foydalanuvchi sarlavhasi bo‘sh yoki mavjud emas edi. Tekshiring:

- Proxy identifikatsiya sarlavhalarini uzatishga sozlanganmi?
- Sarlavha nomi to‘g‘rimi? (katta-kichik harflar farqi yo‘q, ammo yozilishi muhim)
- Foydalanuvchi proxy’da haqiqatan ham autentifikatsiyadan o‘tganmi?

### "trusted_proxy_missing_header_\*"

Talab qilinadigan sarlavha mavjud emas. Tekshiring:

- Ushbu aniq sarlavhalar uchun proxy sozlamalaringizni
- Sarlavhalar zanjirning biror joyida olib tashlanmayotganini

### "trusted_proxy_user_not_allowed"

Foydalanuvchi autentifikatsiyadan o‘tgan, lekin `allowUsers` ro‘yxatida yo‘q. Uni qo‘shing yoki allowlist’ni olib tashlang.

### WebSocket hali ham ishlamayapti

Proxingiz quyidagilarga ishonch hosil qiling:

- WebSocket upgrade’larini qo‘llab-quvvatlaydi (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket upgrade so‘rovlarida (faqat HTTP emas) identifikatsiya sarlavhalarini uzatadi
- WebSocket ulanishlari uchun alohida autentifikatsiya yo‘li mavjud emas

## Token autentifikatsiyasidan migratsiya

Agar token autentifikatsiyasidan trusted-proxy’ga o‘tayotgan bo‘lsangiz:

1. Proxingizni foydalanuvchilarni autentifikatsiya qilish va sarlavhalarni uzatish uchun sozlang
2. Proxy sozlamasini mustaqil ravishda sinab ko‘ring (sarlavhalar bilan curl)
3. OpenClaw konfiguratsiyasini trusted-proxy autentifikatsiyasi bilan yangilang
4. Gateway’ni qayta ishga tushiring
5. Control UI’dan WebSocket ulanishlarini sinab ko‘ring
6. `openclaw security audit` ni ishga tushiring va natijalarni ko‘rib chiqing

## Bog‘liq

- [Security](/gateway/security) — to‘liq xavfsizlik qo‘llanmasi
- [Configuration](/gateway/configuration) — konfiguratsiya ma’lumotnomasi
- [Remote Access](/gateway/remote) — boshqa masofaviy kirish usullari
- [Tailscale](/gateway/tailscale) — faqat tailnet uchun kirishdagi sodda alternativ
