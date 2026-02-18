---
title: "Google Chat"
---

# Google Chat (Chat API)

Holati: Google Chat API webhooklari (faqat HTTP) orqali DM va space’lar uchun tayyor.

## Tezkor sozlash (boshlovchilar uchun)

1. Google Cloud loyihasini yarating va **Google Chat API**ni yoqing.
   - O‘ting: [Google Chat API Credentials](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - Agar API hali yoqilmagan bo‘lsa, uni yoqing.
2. **Service Account** yarating:
   - **Create Credentials** > **Service Account** tugmasini bosing.
   - Istalgan nom bering (masalan, `openclaw-chat`).
   - Ruxsatlarni bo‘sh qoldiring (**Continue** bosing).
   - Kirish huquqiga ega prinsiplalarni bo‘sh qoldiring (**Done** bosing).
3. **JSON Key** yarating va yuklab oling:
   - Service accountlar ro‘yxatida hozir yaratganingizni bosing.
   - **Keys** bo‘limiga o‘ting.
   - **Add Key** > **Create new key** ni bosing.
   - **JSON** ni tanlang va **Create** ni bosing.
4. Yuklab olingan JSON faylni gateway hostingizda saqlang (masalan, `~/.openclaw/googlechat-service-account.json`).
5. [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) sahifasida Google Chat ilovasini yarating:
   - **Application info** bo‘limini to‘ldiring:
     - **App name**: (masalan, `OpenClaw`)
     - **Avatar URL**: (masalan, `https://openclaw.ai/logo.png`)
     - **Description**: (masalan, `Personal AI Assistant`)
   - **Interactive features** ni yoqing.
   - **Functionality** ostida **Join spaces and group conversations** ni belgilang.
   - **Connection settings** ostida **HTTP endpoint URL** ni tanlang.
   - **Triggers** ostida **Use a common HTTP endpoint URL for all triggers** ni tanlang va qiymat sifatida gateway’ning ommaviy URL manziliga `/googlechat` qo‘shib kiriting.
     - _Maslahat: Gateway’ning ommaviy URL manzilini topish uchun `openclaw status` ni ishga tushiring._
   - **Visibility** ostida **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;** ni belgilang.
   - Matn maydoniga emailingizni kiriting (masalan, `user@example.com`).
   - Pastdagi **Save** tugmasini bosing.
6. **Ilova holatini yoqing**:
   - Saqlagandan so‘ng, sahifani **yangilang**.
   - **App status** bo‘limini toping (odatda saqlagandan keyin yuqori yoki pastki qismda paydo bo‘ladi).
   - Holatini **Live - available to users** ga o‘zgartiring.
   - Yana **Save** ni bosing.
7. OpenClaw’ni service account yo‘li + webhook audience bilan sozlang:
   - Muhit o‘zgaruvchisi: `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - Yoki konfiguratsiyada: `channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`.
8. Webhook audience turini va qiymatini belgilang (Chat ilovasi sozlamalariga mos bo‘lishi kerak).
9. Gateway’ni ishga tushiring. Google Chat webhook manzilingizga POST so‘rov yuboradi.

## Google Chat’ga qo‘shish

Gateway ishga tushgan va emailingiz visibility ro‘yxatiga qo‘shilgan bo‘lsa:

1. [Google Chat](https://chat.google.com/) ga o‘ting.
2. **Direct Messages** yonidagi **+** (plus) belgisini bosing.
3. Qidiruv satriga (odatda odam qo‘shadigan joy) Google Cloud Console’da sozlagan **App name** ni kiriting.
   - **Eslatma**: Bot "Marketplace" ro‘yxatida ko‘rinmaydi, chunki u xususiy ilova. Uni nomi bo‘yicha qidirishingiz kerak.
4. Natijalardan botingizni tanlang.
5. 1:1 suhbatni boshlash uchun **Add** yoki **Chat** ni bosing.
6. Assistentni ishga tushirish uchun "Hello" deb yozing!

## Ommaviy URL (faqat Webhook)

Google Chat webhooklari ommaviy HTTPS endpoint talab qiladi. Xavfsizlik uchun **faqat `/googlechat` yo‘lini internetga oching**. OpenClaw dashboard va boshqa sezgir endpointlarni xususiy tarmog‘ingizda qoldiring.

### Variant A: Tailscale Funnel (Tavsiya etiladi)

Xususiy dashboard uchun Tailscale Serve, ommaviy webhook yo‘li uchun esa Funnel’dan foydalaning. Bu `/` ni xususiy saqlab, faqat `/googlechat` ni ochadi.

1. **Gateway qaysi manzilga bog‘langanini tekshiring:**

   ```bash
   ss -tlnp | grep 18789
   ```

   IP manzilni yozib oling (masalan, `127.0.0.1`, `0.0.0.0` yoki `100.x.x.x` kabi Tailscale IP).

2. **Dashboard’ni faqat tailnet uchun oching (8443 port):**

   ```bash
   # Agar localhost (127.0.0.1 yoki 0.0.0.0) ga bog‘langan bo‘lsa:
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # Agar faqat Tailscale IP (masalan, 100.106.161.80) ga bog‘langan bo‘lsa:
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **Faqat webhook yo‘lini ommaviy oching:**

   ```bash
   # Agar localhost (127.0.0.1 yoki 0.0.0.0) ga bog‘langan bo‘lsa:
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # Agar faqat Tailscale IP (masalan, 100.106.161.80) ga bog‘langan bo‘lsa:
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **Node uchun Funnel ruxsatini tasdiqlang:**
   Agar so‘ralsa, chiqishda ko‘rsatilgan avtorizatsiya URL manziliga o‘ting va tailnet siyosatida ushbu node uchun Funnel’ni yoqing.

5. **Sozlamani tekshiring:**

   ```bash
   tailscale serve status
   tailscale funnel status
   ```

Sizning ommaviy webhook URL manzilingiz:
`https://<node-name>.<tailnet>.ts.net/googlechat`

Xususiy dashboard faqat tailnet ichida qoladi:
`https://<node-name>.<tailnet>.ts.net:8443/`

Google Chat ilovasi sozlamasida ommaviy URL’ni (`:8443`siz) ishlating.

> Eslatma: Ushbu sozlama qayta yuklashlardan keyin ham saqlanadi. Keyinroq olib tashlash uchun `tailscale funnel reset` va `tailscale serve reset` ni ishga tushiring.

### Variant B: Reverse Proxy (Caddy)

Agar Caddy kabi reverse proxy ishlatsangiz, faqat kerakli yo‘lni proksi qiling:

```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```

Ushbu konfiguratsiyada `your-domain.com/` ga kelgan so‘rovlar e’tiborga olinmaydi yoki 404 qaytaradi, `your-domain.com/googlechat` esa xavfsiz tarzda OpenClaw’ga yo‘naltiriladi.

### Variant C: Cloudflare Tunnel

Tunnel ingress qoidalarini faqat webhook yo‘lini yo‘naltiradigan qilib sozlang:

- **Yo‘l**: `/googlechat` -> `http://localhost:18789/googlechat`
- **Standart qoida**: HTTP 404 (Topilmadi)

## Qanday ishlaydi

1. Google Chat gateway’ga webhook POST so‘rovlarini yuboradi. Har bir so‘rov `Authorization: Bearer <token>` sarlavhasini o‘z ichiga oladi.
2. OpenClaw tokenni sozlangan `audienceType` va `audience` bilan tekshiradi:
   - `audienceType: "app-url"` → audience — sizning HTTPS webhook URL manzilingiz.
   - `audienceType: "project-number"` → audience — Cloud loyiha raqami.
3. Xabarlar space bo‘yicha marshrutlanadi:
   - DM’lar uchun sessiya kaliti `agent:<agentId>:googlechat:dm:<spaceId>`.
   - Space’lar uchun sessiya kaliti `agent:<agentId>:googlechat:group:<spaceId>`.
4. DM kirishi odatda pairing orqali amalga oshadi. Noma’lum jo‘natuvchilar pairing kodi oladi; tasdiqlash:
   - `openclaw pairing approve googlechat <code>`
5. Guruh space’larda odatda @-mention talab qilinadi. Agar mention aniqlash uchun ilova foydalanuvchi nomi kerak bo‘lsa, `botUser` dan foydalaning.

## Manzillar (Targets)

Yetkazib berish va allowlist uchun quyidagi identifikatorlardan foydalaning:

- Shaxsiy xabarlar: `users/<userId>` (tavsiya etiladi) yoki oddiy email `name@example.com` (o‘zgaruvchan principal).
- Eskirgan: `users/<email>` email allowlist emas, balki user id sifatida talqin qilinadi.
- Space’lar: `spaces/<spaceId>`.

## Muhim konfiguratsiya nuqtalari

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // ixtiyoriy; mention aniqlashga yordam beradi
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only.",
        },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

Izohlar:

- Service account ma’lumotlari `serviceAccount` (JSON string) orqali ham bevosita uzatilishi mumkin.
- Agar `webhookPath` ko‘rsatilmagan bo‘lsa, standart qiymat `/googlechat`.
- Agar `actions.reactions` yoqilgan bo‘lsa, reaksiyalar `reactions` vositasi va `channels action` orqali mavjud.
- `typingIndicator` quyidagilarni qo‘llab-quvvatlaydi: `none`, `message` (standart), va `reaction` (`reaction` uchun user OAuth talab qilinadi).
- Ilovalar (attachment) Chat API orqali yuklab olinadi va media pipeline’da saqlanadi (`mediaMaxMb` bilan hajm cheklangan).

## Muammolarni bartaraf etish

### 405 Method Not Allowed

Agar Google Cloud Logs Explorer’da quyidagi xatolar ko‘rinsa:

```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

Bu webhook handler ro‘yxatdan o‘tmaganini bildiradi. Odatdagi sabablar:

1. **Kanal sozlanmagan**: Konfiguratsiyada `channels.googlechat` bo‘limi yo‘q. Tekshiring:

   ```bash
   openclaw config get channels.googlechat
   ```

   Agar "Config path not found" qaytsa, konfiguratsiyani qo‘shing (qarang: [Muhim konfiguratsiya nuqtalari](#muhim-konfiguratsiya-nuqtalari)).

2. **Plugin yoqilmagan**: Plugin holatini tekshiring:

   ```bash
   openclaw plugins list | grep googlechat
   ```

   Agar "disabled" ko‘rsatsa, konfiguratsiyaga `plugins.entries.googlechat.enabled: true` qo‘shing.

3. **Gateway qayta ishga tushirilmagan**: Konfiguratsiya qo‘shilgandan so‘ng gateway’ni qayta ishga tushiring:

   ```bash
   openclaw gateway restart
   ```

Kanal ishlayotganini tekshiring:

```bash
openclaw channels status
# Ko‘rsatishi kerak: Google Chat default: enabled, configured, ...
```

### Boshqa muammolar

- Autentifikatsiya xatolari yoki audience sozlamasi yetishmasligini tekshirish uchun `openclaw channels status --probe` dan foydalaning.
- Xabarlar kelmasa, Chat ilovasining webhook URL va event subscription’larini tekshiring.
- Agar mention talabi javoblarni bloklasa, `botUser` ni ilovaning user resource nomiga o‘rnating va `requireMention` ni tekshiring.
- So‘rovlar gateway’ga yetib kelayotganini ko‘rish uchun test xabar yuborayotganda `openclaw logs --follow` ni ishga tushiring.

Tegishli hujjatlar:

- [Gateway konfiguratsiyasi](/gateway/configuration)
- [Xavfsizlik](/gateway/security)
- [Reaksiyalar](/tools/reactions)

