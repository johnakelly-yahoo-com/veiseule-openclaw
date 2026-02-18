---
title: Feishu
---

# Feishu boti

Feishu (Lark) — bu kompaniyalar xabar almashish va hamkorlik uchun foydalanadigan jamoaviy chat platformasi. Ushbu plagin OpenClaw’ni Feishu/Lark botiga platformaning WebSocket hodisalar obunasi orqali ulaydi, shuning uchun ochiq webhook URL’ni oshkor qilmasdan xabarlarni qabul qilish mumkin.

---

## Kerakli plagin

Feishu plaginini o‘rnating:

```bash
openclaw plugins install @openclaw/feishu
```

Lokal checkout (git repozitoriydan ishga tushirilganda):

```bash
openclaw plugins install ./extensions/feishu
```

---

## Tezkor boshlash

Feishu kanalini qo‘shishning ikki usuli mavjud:

### 1-usul: onboarding ustasi (tavsiya etiladi)

Agar siz OpenClaw’ni endigina o‘rnatgan bo‘lsangiz, ustani ishga tushiring:

```bash
openclaw onboard
```

Usta quyidagilar bo‘yicha yo‘l-yo‘riq beradi:

1. Feishu ilovasini yaratish va ma’lumotlarni yig‘ish
2. Ilova ma’lumotlarini OpenClaw’da sozlash
3. Gateway’ni ishga tushirish

✅ **Sozlangandan so‘ng**, gateway holatini tekshiring:

- `openclaw gateway status`
- `openclaw logs --follow`

### 2-usul: CLI orqali sozlash

Agar dastlabki o‘rnatishni allaqachon tugatgan bo‘lsangiz, kanalni CLI orqali qo‘shing:

```bash
openclaw channels add
```

**Feishu** ni tanlang, so‘ng App ID va App Secret’ni kiriting.

✅ **Sozlangandan so‘ng**, gateway’ni boshqaring:

- `openclaw gateway status`
- `openclaw gateway restart`
- `openclaw logs --follow`

---

## 1-qadam: Feishu ilovasini yaratish

### 1. Feishu Open Platform’ni oching

[Feishu Open Platform](https://open.feishu.cn/app) sahifasiga o‘ting va tizimga kiring.

Lark (global) tenantlar [https://open.larksuite.com/app](https://open.larksuite.com/app) manzilidan foydalanishi va Feishu konfiguratsiyasida `domain: "lark"` ni o‘rnatishi kerak.

### 2. Ilova yarating

1. **Create enterprise app** tugmasini bosing
2. Ilova nomi va tavsifini kiriting
3. Ilova ikonkasini tanlang

![Korxona ilovasini yaratish](../images/feishu-step2-create-app.png)

### 3. Ma’lumotlarni nusxalash

**Credentials & Basic Info** bo‘limidan quyidagilarni nusxalang:

- **Ilova ID** (format: `cli_xxx`)
- **Ilova siri**

❗ **Muhim:** App Secret’ni maxfiy saqlang.

![Ma’lumotlarni olish](../images/feishu-step3-credentials.png)

### 4. Ruxsatlarni sozlash

**Permissions** bo‘limida **Batch import** ni bosing va quyidagini joylashtiring:

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![Ruxsatlarni sozlash](../images/feishu-step4-permissions.png)

### 5. Bot imkoniyatini yoqish

**App Capability** > **Bot** bo‘limida:

1. Bot imkoniyatini yoqing
2. Bot nomini belgilang

![Bot imkoniyatini yoqish](../images/feishu-step5-bot-capability.png)

### 6. Hodisalar obunasini sozlash

⚠️ **Muhim:** hodisalar obunasini sozlashdan oldin quyidagilarga ishonch hosil qiling:

1. Siz allaqachon Feishu uchun `openclaw channels add` ni ishga tushirgansiz
2. Gateway ishlamoqda (`openclaw gateway status`)

**Event Subscription** bo‘limida:

1. **Use long connection to receive events** (WebSocket) ni tanlang
2. `im.message.receive_v1` hodisasini qo‘shing

⚠️ Agar gateway ishlamayotgan bo‘lsa, long-connection sozlamasi saqlanmasligi mumkin.

![Hodisalar obunasini sozlash](../images/feishu-step6-event-subscription.png)

### 7. Ilovani nashr qilish

1. **Version Management & Release** bo‘limida versiya yarating
2. Ko‘rib chiqish uchun yuboring va nashr qiling
3. Administrator tasdig‘ini kuting (korxona ilovalari odatda avtomatik tasdiqlanadi)

---

## 2-qadam: OpenClaw’ni sozlash

### Usta orqali sozlash (tavsiya etiladi)

```bash
openclaw channels add
```

**Feishu** ni tanlang va App ID hamda App Secret’ni kiriting.

### Konfiguratsiya fayli orqali sozlash

`~/.openclaw/openclaw.json` faylini tahrir qiling:

```json5
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

### Muhit o‘zgaruvchilari orqali sozlash

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark (global) domeni

Agar tenantingiz Lark (xalqaro) da bo‘lsa, domenni `lark` (yoki to‘liq domen satri) ga o‘rnating. Buni `channels.feishu.domain` da yoki har bir akkaunt uchun (`channels.feishu.accounts.<id>.domain`) belgilashingiz mumkin.

```json5
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

---

## 3-qadam: Ishga tushirish va sinov

### 1. Gateway’ni ishga tushiring

```bash
openclaw gateway
```

### 2. Sinov xabari yuboring

Feishu’da botingizni toping va xabar yuboring.

### 3. Pairing’ni tasdiqlang

Standart holatda bot pairing kodi bilan javob beradi. Uni tasdiqlang:

```bash
openclaw pairing approve feishu <CODE>
```

Tasdiqlangandan so‘ng odatdagidek suhbatlashishingiz mumkin.

---

## Umumiy ko‘rinish

- **Feishu bot kanali**: gateway tomonidan boshqariladigan Feishu bot
- **Deterministik yo‘naltirish**: javoblar har doim Feishu’ga qaytadi
- **Sessiya izolyatsiyasi**: DM’lar bitta asosiy sessiyani ulashadi; guruhlar alohida
- **WebSocket ulanishi**: Feishu SDK orqali long connection, ochiq URL talab qilinmaydi

---

## Kirishni boshqarish

### Shaxsiy xabarlar (DM)

- **Standart**: `dmPolicy: "pairing"` (noma’lum foydalanuvchilar pairing kodi oladi)
- **Pairing’ni tasdiqlash**:

  ```bash
  openclaw pairing list feishu
  openclaw pairing approve feishu <CODE>
  ```

- **Allowlist rejimi**: ruxsat etilgan Open ID’larni `channels.feishu.allowFrom` da belgilang

### Guruh chatlari

**1. Guruh siyosati** (`channels.feishu.groupPolicy`):

- `"open"` = guruhlarda hammaga ruxsat (standart)
- `"allowlist"` = faqat `groupAllowFrom` dagilarga ruxsat
- `"disabled"` = guruh xabarlarini o‘chirish

**2. Mention talabi** (`channels.feishu.groups.<chat_id>.requireMention`):

- `true` = @mention talab qilinadi (standart)
- `false` = mentionsiz javob beradi

---

## Guruh sozlamalari misollari

### Barcha guruhlarga ruxsat, @mention talab qilinadi (standart)

```json5
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // Standart requireMention: true
    },
  },
}
```

### Barcha guruhlarga ruxsat, @mention talab qilinmaydi

```json5
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### Faqat ma’lum foydalanuvchilarga guruhlarda ruxsat

```json5
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["ou_xxx", "ou_yyy"],
    },
  },
}
```

---

## Guruh/foydalanuvchi ID’larini olish

### Guruh ID’lari (chat_id)

Guruh ID’lari `oc_xxx` ko‘rinishida bo‘ladi.

**1-usul (tavsiya etiladi)**

1. Gateway’ni ishga tushiring va guruhda botni @mention qiling
2. `openclaw logs --follow` ni ishga tushiring va `chat_id` ni toping

**2-usul**

Feishu API debugger’dan foydalanib guruh chatlarini ro‘yxatini oling.

### Foydalanuvchi ID’lari (open_id)

Foydalanuvchi ID’lari `ou_xxx` ko‘rinishida bo‘ladi.

**1-usul (tavsiya etiladi)**

1. Gateway’ni ishga tushiring va botga DM yuboring
2. `openclaw logs --follow` ni ishga tushiring va `open_id` ni toping

**2-usul**

Foydalanuvchi Open ID’larini pairing so‘rovlaridan tekshiring:

```bash
openclaw pairing list feishu
```

---

## Keng tarqalgan buyruqlar

| Command   | Tavsif              |
| --------- | ------------------- |
| `/status` | Bot holatini ko‘rsatish |
| `/reset`  | Sessiyani tiklash   |
| `/model`  | Modelni ko‘rsatish/almashtirish |

> Eslatma: Feishu hozircha native buyruq menyularini qo‘llab-quvvatlamaydi, shuning uchun buyruqlar matn ko‘rinishida yuborilishi kerak.

## Gateway boshqaruv buyruqlari

| Command                    | Tavsif                         |
| -------------------------- | ------------------------------ |
| `openclaw gateway status`  | Gateway holatini ko‘rsatish    |
| `openclaw gateway install` | Gateway xizmatini o‘rnatish/ishga tushirish |
| `openclaw gateway stop`    | Gateway xizmatini to‘xtatish   |
| `openclaw gateway restart` | Gateway xizmatini qayta ishga tushirish |
| `openclaw logs --follow`   | Gateway loglarini kuzatish     |

---

## Muammolarni bartaraf etish

### Bot guruh chatlarida javob bermaydi

1. Bot guruhga qo‘shilganini tekshiring
2. Botni @mention qilganingizga ishonch hosil qiling (standart xatti-harakat)
3. `groupPolicy` `"disabled"` ga o‘rnatilmaganini tekshiring
4. Loglarni tekshiring: `openclaw logs --follow`

### Bot xabarlarni qabul qilmaydi

1. Ilova nashr qilingan va tasdiqlanganini tekshiring
2. Hodisalar obunasida `im.message.receive_v1` mavjudligini tekshiring
3. **Long connection** yoqilganini tekshiring
4. Ilova ruxsatlari to‘liq ekanini tekshiring
5. Gateway ishlayotganini tekshiring: `openclaw gateway status`
6. Loglarni tekshiring: `openclaw logs --follow`

### App Secret oshkor bo‘ldi

1. Feishu Open Platform’da App Secret’ni yangilang
2. Konfiguratsiyada App Secret’ni yangilang
3. Gateway’ni qayta ishga tushiring

### Xabar yuborishda xatoliklar

1. Ilovada `im:message:send_as_bot` ruxsati borligini tekshiring
2. Ilova nashr qilinganini tekshiring
3. Loglarda batafsil xatolarni tekshiring

---

## Kengaytirilgan sozlamalar

### Bir nechta akkaunt

```json5
{
  channels: {
    feishu: {
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

### Xabar cheklovlari

- `textChunkLimit`: chiqish matni bo‘lak hajmi (standart: 2000 belgi)
- `mediaMaxMb`: media yuklash/yuklab olish limiti (standart: 30MB)

### Streaming

Feishu interaktiv kartalar orqali streaming javoblarni qo‘llab-quvvatlaydi. Yoqilganda, bot matn generatsiya qilinayotganda kartani yangilab boradi.

```json5
{
  channels: {
    feishu: {
      streaming: true, // streaming karta chiqishini yoqish (standart true)
      blockStreaming: true, // blok darajasida streaming (standart true)
    },
  },
}
```

To‘liq javob yuborilishini kutish uchun `streaming: false` ni o‘rnating.

### Ko‘p-agentli yo‘naltirish

Feishu DM yoki guruhlarini turli agentlarga yo‘naltirish uchun `bindings` dan foydalaning.

```json5
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

Yo‘naltirish maydonlari:

- `match.channel`: `"feishu"`
- `match.peer.kind`: `"direct"` yoki `"group"`
- `match.peer.id`: foydalanuvchi Open ID (`ou_xxx`) yoki guruh ID (`oc_xxx`)

ID’larni aniqlash bo‘yicha maslahatlar uchun [Guruh/foydalanuvchi ID’larini olish](#guruhfoydalanuvchi-idlarini-olish) ga qarang.

---

## Konfiguratsiya ma’lumotnomasi

To‘liq konfiguratsiya: [Gateway configuration](/gateway/configuration)

Asosiy sozlamalar:

| Setting                                           | Tavsif                          | Default   |
| ------------------------------------------------- | ------------------------------- | --------- |
| `channels.feishu.enabled`                         | Kanalni yoqish/o‘chirish        | `true`    |
| `channels.feishu.domain`                          | API domeni (`feishu` yoki `lark`) | `feishu`  |
| `channels.feishu.accounts.<id>.appId`             | App ID                          | -         |
| `channels.feishu.accounts.<id>.appSecret`         | App Secret                      | -         |
| `channels.feishu.accounts.<id>.domain`            | Har bir akkaunt uchun API domeni | `feishu`  |
| `channels.feishu.dmPolicy`                        | DM siyosati                     | `pairing` |
| `channels.feishu.allowFrom`                       | DM allowlist (open_id ro‘yxati) | -         |
| `channels.feishu.groupPolicy`                     | Guruh siyosati                  | `open`    |
| `channels.feishu.groupAllowFrom`                  | Guruh allowlist                 | -         |
| `channels.feishu.groups.<chat_id>.requireMention` | @mention talab qilish           | `true`    |
| `channels.feishu.groups.<chat_id>.enabled`        | Guruhni yoqish                  | `true`    |
| `channels.feishu.textChunkLimit`                  | Xabar bo‘lagi hajmi             | `2000`    |
| `channels.feishu.mediaMaxMb`                      | Media hajmi limiti              | `30`      |
| `channels.feishu.streaming`                       | Streaming karta chiqishini yoqish | `true`    |
| `channels.feishu.blockStreaming`                  | Blokli streaming’ni yoqish      | `true`    |

---

## dmPolicy ma’lumotnomasi

| Value         | Xatti-harakat                                                  |
| ------------- | -------------------------------------------------------------- |
| `"pairing"`   | **Standart.** Noma’lum foydalanuvchilar pairing kodi oladi; tasdiqlash kerak |
| `"allowlist"` | Faqat `allowFrom` dagi foydalanuvchilar suhbatlasha oladi     |
| `"open"`      | Barcha foydalanuvchilarga ruxsat (allowFrom’da `"*"` talab qilinadi) |
| `"disabled"`  | DM’larni o‘chirish                                             |

---

## Qo‘llab-quvvatlanadigan xabar turlari

### Qabul qilish

- ✅ Matn
- ✅ Boy matn (post)
- ✅ Rasmlar
- ✅ Fayllar
- ✅ Audio
- ✅ Video
- ✅ Stikerlar

### Yuborish

- ✅ Matn
- ✅ Rasmlar
- ✅ Fayllar
- ✅ Audio
- ⚠️ Boy matn (qisman qo‘llab-quvvatlanadi)

