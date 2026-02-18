---
title: "WhatsApp"
---

# WhatsApp (web channel)

Status: WhatsApp Web via Baileys only. Gateway owns the session(s).

## Quick setup (beginner)

1. Use a **separate phone number** if possible (recommended).
2. Configure WhatsApp in `~/.openclaw/openclaw.json`.
3. Run `openclaw channels login` to scan the QR code (Linked Devices).
4. Start the gateway.

Minimal config:

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

## Goals

- Multiple WhatsApp accounts (multi-account) in one Gateway process.
- Deterministic routing: replies return to WhatsApp, no model routing.
- Model sees enough context to understand quoted replies.

## 1. Konfiguratsiya yozuvlari

2. Odatiy holatda, WhatsApp `/config set|unset` orqali ishga tushirilgan konfiguratsiya yangilanishlarini yozishga ruxsat etilgan ( `commands.config: true` talab etiladi).

3. O‘chirish uchun:

```json5
4. {
  channels: { whatsapp: { configWrites: false } },
}
```

## 5. Arxitektura (kim nimaga egalik qiladi)

- 6. **Gateway** Baileys soketi va inbox loop’ga egalik qiladi.
- 7. **CLI / macOS ilovasi** gateway bilan muloqot qiladi; Baileys’dan bevosita foydalanilmaydi.
- 8. Chiquvchi xabarlarni yuborish uchun **faol tinglovchi** talab etiladi; aks holda yuborish darhol muvaffaqiyatsiz bo‘ladi.

## 9. Telefon raqamini olish (ikki rejim)

10. WhatsApp tasdiqlash uchun haqiqiy mobil raqamni talab qiladi. 11. VoIP va virtual raqamlar odatda bloklanadi. 12. OpenClaw’ni WhatsApp’da ishga tushirishning ikkita qo‘llab-quvvatlanadigan usuli mavjud:

### 13. Maxsus raqam (tavsiya etiladi)

14. OpenClaw uchun **alohida telefon raqami**dan foydalaning. 15. Eng yaxshi UX, toza marshrutlash, o‘zingiz bilan chat qilishdagi noqulayliklar yo‘q. 16. Ideal sozlama: **zaxira/eski Android telefon + eSIM**. 17. Uni Wi‑Fi va quvvatga ulangan holda qoldiring va QR orqali ulang.

18. **WhatsApp Business:** Xuddi shu qurilmada boshqa raqam bilan WhatsApp Business’dan foydalanishingiz mumkin. 19. Shaxsiy WhatsApp’ingizni alohida saqlash uchun juda qulay — WhatsApp Business’ni o‘rnating va OpenClaw raqamini u yerda ro‘yxatdan o‘tkazing.

20. **Namunaviy konfiguratsiya (maxsus raqam, bitta foydalanuvchi allowlist):**

```json5
21. {
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

22. **Pairing rejimi (ixtiyoriy):**
    Allowlist o‘rniga pairing ishlatmoqchi bo‘lsangiz, `channels.whatsapp.dmPolicy` ni `pairing` ga o‘rnating. 23. Noma’lum yuboruvchilar pairing kodini oladi; tasdiqlash uchun:
    `openclaw pairing approve whatsapp <code>`

### 24. Shaxsiy raqam (zaxira variant)

25. Tezkor zaxira: OpenClaw’ni **o‘zingizning raqamingizda** ishga tushiring. 26. Kontaktlarni spam qilmaslik uchun test paytida o‘zingizga xabar yuboring (WhatsApp “Message yourself”). 27. Sozlash va tajribalar vaqtida asosiy telefoningizda tasdiqlash kodlarini o‘qishga tayyor bo‘ling. 28. **Self-chat rejimini yoqish shart.**
    Usta (wizard) sizdan shaxsiy WhatsApp raqamingizni so‘raganda, yordamchi raqamini emas, balki siz xabar yuboradigan telefonni (egasi/yuboruvchi) kiriting.

29. **Namunaviy konfiguratsiya (shaxsiy raqam, self-chat):**

```json
30. {
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

31. Self-chat javoblari, agar `messages.responsePrefix` o‘rnatilmagan bo‘lsa, `[{identity.name}]` ga sukut bo‘yicha o‘rnatiladi (aks holda `[openclaw]`). 32. Prefiksni moslashtirish yoki o‘chirish uchun uni aniq belgilang
    (olib tashlash uchun `""` dan foydalaning).

### 33. Raqam manbasi bo‘yicha maslahatlar

- 34. Mamlakatingiz mobil operatoridan **mahalliy eSIM** (eng ishonchli)
  - 35. Avstriya: [hot.at](https://www.hot.at)
  - 36. Buyuk Britaniya: [giffgaff](https://www.giffgaff.com) — bepul SIM, shartnomasiz
- 37. **Oldindan to‘lanadigan SIM** — arzon, faqat tasdiqlash uchun bitta SMS qabul qila olishi kifoya

38. **Qoching:** TextNow, Google Voice, ko‘pchilik “bepul SMS” xizmatlari — WhatsApp bularni juda qat’iy bloklaydi.

39. **Maslahat:** Raqam faqat bitta tasdiqlash SMS’ini qabul qilishi kerak. 40. Shundan so‘ng, WhatsApp Web sessiyalari `creds.json` orqali saqlanib qoladi.

## 41. Nega Twilio emas?

- 42. OpenClaw’ning dastlabki versiyalari Twilio’ning WhatsApp Business integratsiyasini qo‘llab-quvvatlagan.
- 43. WhatsApp Business raqamlari shaxsiy yordamchi uchun mos emas.
- 44. Meta 24 soatlik javob oynasini majburiy qiladi; agar oxirgi 24 soat ichida javob bermagan bo‘lsangiz, biznes raqam yangi xabarlarni boshlay olmaydi.
- 45. Yuqori hajmli yoki “sergap” foydalanish agressiv bloklashni keltirib chiqaradi, chunki biznes akkauntlar o‘nlab shaxsiy yordamchi xabarlarini yuborish uchun mo‘ljallanmagan.
- 46. Natija: yetkazib berish ishonchsiz va tez-tez bloklanadi, shu sababli qo‘llab-quvvatlash olib tashlandi.

## 47. Kirish + hisob ma’lumotlari

- 48. Kirish buyrug‘i: `openclaw channels login` (QR orqali Linked Devices).
- 49. Ko‘p akkauntli kirish: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- 50. Standart akkaunt (`--account` ko‘rsatilmaganida): agar mavjud bo‘lsa `default`, aks holda sozlangan akkaunt id’laridan birinchisi (saralangan).
- Credentials stored in `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Backup copy at `creds.json.bak` (restored on corruption).
- Legacy compatibility: older installs stored Baileys files directly in `~/.openclaw/credentials/`.
- Logout: `openclaw channels logout` (or `--account <id>`) deletes WhatsApp auth state (but keeps shared `oauth.json`).
- Logged-out socket => error instructs re-link.

## Inbound flow (DM + group)

- WhatsApp events come from `messages.upsert` (Baileys).
- Inbox listeners are detached on shutdown to avoid accumulating event handlers in tests/restarts.
- Status/broadcast chats are ignored.
- Direct chats use E.164; groups use group JID.
- **DM policy**: `channels.whatsapp.dmPolicy` controls direct chat access (default: `pairing`).
  - Pairing: unknown senders get a pairing code (approve via `openclaw pairing approve whatsapp <code>`; codes expire after 1 hour).
  - Open: requires `channels.whatsapp.allowFrom` to include `"*"`.
  - Your linked WhatsApp number is implicitly trusted, so self messages skip ⁠`channels.whatsapp.dmPolicy` and `channels.whatsapp.allowFrom` checks.

### Personal-number mode (fallback)

If you run OpenClaw on your **personal WhatsApp number**, enable `channels.whatsapp.selfChatMode` (see sample above).

Behavior:

- Outbound DMs never trigger pairing replies (prevents spamming contacts).
- Inbound unknown senders still follow `channels.whatsapp.dmPolicy`.
- Self-chat mode (allowFrom includes your number) avoids auto read receipts and ignores mention JIDs.
- Read receipts sent for non-self-chat DMs.

## Read receipts

By default, the gateway marks inbound WhatsApp messages as read (blue ticks) once they are accepted.

Disable globally:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Disable per account:

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false },
      },
    },
  },
}
```

Notes:

- Self-chat mode always skips read receipts.

## WhatsApp FAQ: sending messages + pairing

**Will OpenClaw message random contacts when I link WhatsApp?**  
No. Default DM policy is **pairing**, so unknown senders only get a pairing code and their message is **not processed**. OpenClaw only replies to chats it receives, or to sends you explicitly trigger (agent/CLI).

**How does pairing work on WhatsApp?**  
Pairing is a DM gate for unknown senders:

- First DM from a new sender returns a short code (message is not processed).
- Approve with: `openclaw pairing approve whatsapp <code>` (list with `openclaw pairing list whatsapp`).
- Codes expire after 1 hour; pending requests are capped at 3 per channel.

40. Javoblar baribir **bir xil WhatsApp akkauntidan** keladi va to‘g‘ridan-to‘g‘ri chatlar har bir agentning asosiy sessiyasiga birlashadi, shuning uchun **har bir odam uchun bitta agent**dan foydalaning. 41. %%{init: {
    'theme': 'base',
    'themeVariables': {
    'primaryColor': '#ffffff',
    'primaryTextColor': '#000000',
    'primaryBorderColor': '#000000',
    'lineColor': '#000000',
    'secondaryColor': '#f9f9fb',
    'tertiaryColor': '#ffffff',
    'clusterBkg': '#f9f9fb',
    'clusterBorder': '#000000',
    'nodeBorder': '#000000',
    'mainBkg': '#ffffff',
    'edgeLabelBackground': '#ffffff'
    }
    }}%%
    sequenceDiagram
    participant Client
    participant Gateway

    Client->>Gateway: req:connect
    Gateway-->>Client: res (ok)
    Note right of Gateway: yoki res error + close
    Note left of Client: payload=hello-ok<br>snapshot: presence + health

    Gateway-->>Client: event:presence
    Gateway-->>Client: event:tick

    Client->>Gateway: req:agent
    Gateway-->>Client: res:agent<br>ack {runId, status:"accepted"}
    Gateway-->>Client: event:agent<br>(oqimda)
    Gateway-->>Client: res:agent<br>final {runId, status, summary} DM access control (`dmPolicy`/`allowFrom`) is global per WhatsApp account. See [Multi-Agent Routing](/concepts/multi-agent).

**Why do you ask for my phone number in the wizard?**  
The wizard uses it to set your **allowlist/owner** so your own DMs are permitted. It’s not used for auto-sending. If you run on your personal WhatsApp number, use that same number and enable `channels.whatsapp.selfChatMode`.

## Message normalization (what the model sees)

- `Body` is the current message body with envelope.

- Quoted reply context is **always appended**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Reply metadata also set:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = quoted body or media placeholder
  - `ReplyToSender` = E.164 when known

- Faqat media bo‘lgan kiruvchi xabarlar plaseholderlardan foydalanadi:
  - <media:image|video|audio|document|sticker>

## Guruhlar

- Guruhlar `agent:<agentId>:whatsapp:group:<jid>` sessiyalariga mos keladi.
- Guruh siyosati: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (standart `allowlist`).
- Faollashtirish rejimlari:
  - `mention` (standart): @mention yoki regex mosligini talab qiladi.
  - `always`: har doim ishga tushadi.
- `/activation mention|always` faqat egaga ruxsat etilgan va alohida xabar sifatida yuborilishi kerak.
- Ega = `channels.whatsapp.allowFrom` (yoki agar sozlanmagan bo‘lsa, o‘zingizning E.164).
- **Tarixni kiritish** (faqat kutilayotganlar):
  - So‘nggi _qayta ishlanmagan_ xabarlar (standart 50) quyida kiritiladi:
    `[Chat messages since your last reply - for context]` (sessiyada allaqachon mavjud xabarlar qayta kiritilmaydi)
  - Joriy xabar quyida:
    `[Current message - respond to this]`
  - Yuboruvchi suffiksi qo‘shiladi: `[from: Name (+E164)]`
- Guruh metama’lumotlari 5 daqiqa keshlanadi (mavzu + ishtirokchilar).

## Javobni yetkazish (threading)

- WhatsApp Web standart xabarlarni yuboradi (joriy shlyuzda iqtibosli javob threading yo‘q).
- Javob teglari ushbu kanalda e’tiborsiz qoldiriladi.

## Tasdiqlovchi reaksiyalar (qabul qilinganda avtomatik reaksiya)

WhatsApp kiruvchi xabarlarga javob bot javob yaratishidan oldin, qabul qilingan zahoti emoji reaksiyalarini avtomatik yuborishi mumkin. Bu foydalanuvchilarga ularning xabari qabul qilinganini darhol bildiradi.

**Konfiguratsiya:**

```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**Variantlar:**

- `emoji` (string): Tasdiqlash uchun ishlatiladigan emoji (masalan, "👀", "✅", "📨"). Bo‘sh yoki ko‘rsatilmagan = funksiya o‘chirilgan.
- `direct` (boolean, standart: `true`): To‘g‘ridan-to‘g‘ri/DM chatlarda reaksiyalarni yuborish.
- `group` (string, standart: `"mentions"`): Guruh chatlaridagi xatti-harakat:
  - `"always"`: Barcha guruh xabarlariga reaksiya qilish (@mention bo‘lmasa ham)
  - `"mentions"`: Faqat bot @mentioned qilinganda reaksiya qilish
  - `"never"`: Guruhlarda hech qachon reaksiya qilmaslik

**Har bir akkaunt uchun alohida override:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Xulq-atvor bo‘yicha eslatmalar:**

- Reaksiyalar xabar qabul qilingach **darhol**, yozish indikatorlari yoki bot javoblaridan oldin yuboriladi.
- `requireMention: false` (faollashtirish: always) bo‘lgan guruhlarda `group: "mentions"` barcha xabarlarga reaksiya qiladi (faqat @mentionlarga emas).
- Fire-and-forget: reaksiya yuborishdagi xatolar loglanadi, lekin bot javob berishiga to‘sqinlik qilmaydi.
- Guruh reaksiyalari uchun ishtirokchi JID avtomatik ravishda qo‘shiladi.
- WhatsApp `messages.ackReaction` ni e’tiborsiz qoldiradi; uning o‘rniga `channels.whatsapp.ackReaction` dan foydalaning.

## Agent vositasi (reaksiyalar)

- Vosita: `whatsapp` `react` amali bilan (`chatJid`, `messageId`, `emoji`, ixtiyoriy `remove`).
- Ixtiyoriy: `participant` (guruh yuboruvchisi), `fromMe` (o‘zingizning xabaringizga reaksiya), `accountId` (ko‘p akkauntli).
- Reaksiyani olib tashlash semantikasi: qarang [/tools/reactions](/tools/reactions).
- Vosita cheklovi: `channels.whatsapp.actions.reactions` (standart: yoqilgan).

## Cheklovlar

- Chiquvchi matn `channels.whatsapp.textChunkLimit` ga bo‘linadi (standart 4000).
- Ixtiyoriy yangi qator bo‘yicha bo‘lish: `channels.whatsapp.chunkMode="newline"` ni o‘rnating, shunda uzunlik bo‘yicha bo‘lishdan oldin bo‘sh qatorlar (paragraf chegaralari) bo‘yicha bo‘linadi.
- Kiruvchi media saqlash hajmi `channels.whatsapp.mediaMaxMb` bilan cheklanadi (standart 50 MB).
- Chiquvchi media elementlari `agents.defaults.mediaMaxMb` bilan cheklanadi (standart 5 MB).

## Chiquvchi yuborish (matn + media)

- Faol veb tinglovchidan foydalanadi; shlyuz ishlamayotgan bo‘lsa xato yuz beradi.
- Matnni bo‘laklash: har bir xabar uchun 4k maksimal ( `channels.whatsapp.textChunkLimit` orqali sozlanadi, ixtiyoriy `channels.whatsapp.chunkMode`).
- Media:
  - Rasm/video/audio/hujjat qo‘llab-quvvatlanadi.
  - Audio PTT sifatida yuboriladi; `audio/ogg` => `audio/ogg; codecs=opus`.
  - Sarlavha faqat birinchi media elementida bo‘ladi.
  - Media yuklab olish HTTP(S) va lokal yo‘llarni qo‘llab-quvvatlaydi.
  - Animatsiyalangan GIFlar: WhatsApp inline aylanish uchun `gifPlayback: true` bilan MP4 kutadi.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send` parametrlari `gifPlayback: true` ni o‘z ichiga oladi

## Ovozli xabarlar (PTT audio)

WhatsApp audio-ni **ovozli xabarlar** (PTT pufagi) sifatida yuboradi.

- Eng yaxshi natija: OGG/Opus. OpenClaw `audio/ogg` ni `audio/ogg; codecs=opus` ga qayta yozadi.
- `[[audio_as_voice]]` WhatsApp uchun e’tiborga olinmaydi (audio allaqachon ovozli xabar sifatida yuboriladi).

## Media limitlari + optimallashtirish

- Standart chiqish limiti: 5 MB (har bir media elementi uchun).
- Bekor qilish: `agents.defaults.mediaMaxMb`.
- Rasmlar limit ostida JPEG ga avtomatik optimallashtiriladi (o‘lchamni o‘zgartirish + sifatni tanlash).
- Haddan katta media => xato; media javobi matnli ogohlantirishga qaytadi.

## Heartbeatlar

- **Gateway heartbeat** ulanish holatini loglaydi (`web.heartbeatSeconds`, standart 60s).
- **Agent heartbeat** har bir agent uchun (`agents.list[].heartbeat`) yoki global tarzda
  `agents.defaults.heartbeat` orqali sozlanishi mumkin (agentga xos yozuvlar bo‘lmasa zaxira).
  - Sozlangan heartbeat promptidan foydalanadi (standart: `Read HEARTBEAT.md if it exists (workspace context). Unga qat’iy amal qiling. Oldingi chatlardagi eski vazifalarni taxmin qilmang va takrorlamang. Agar e’tibor talab qilinadigan narsa bo‘lmasa, HEARTBEAT_OK deb javob bering.`) + `HEARTBEAT_OK` o‘tkazib yuborish xatti-harakati.
  - Yetkazib berish sukut bo‘yicha oxirgi ishlatilgan kanalga (yoki sozlangan maqsadga) amalga oshiriladi.

## Qayta ulanish xatti-harakati

- Backoff siyosati: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Agar maxAttempts ga yetilsa, veb monitoring to‘xtaydi (pasaytirilgan holat).
- Hisobdan chiqilgan => to‘xtaydi va qayta bog‘lashni talab qiladi.

## Konfiguratsiya tezkor xaritasi

- `channels.whatsapp.dmPolicy` (DM siyosati: pairing/allowlist/open/disabled).
- `channels.whatsapp.selfChatMode` (bir telefonli sozlama; bot shaxsiy WhatsApp raqamingizdan foydalanadi).
- `channels.whatsapp.allowFrom` (DM ruxsat etilganlar ro‘yxati). WhatsApp E.164 telefon raqamlaridan foydalanadi (foydalanuvchi nomlari yo‘q).
- `channels.whatsapp.mediaMaxMb` (kiruvchi mediani saqlash limiti).
- `channels.whatsapp.ackReaction` (xabar qabul qilinganda avtomatik reaksiya: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (har bir hisob uchun sozlamalar + ixtiyoriy `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (har bir hisob uchun kiruvchi media limiti).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (har bir hisob uchun ack reaksiya bekor qilinishi).
- `channels.whatsapp.groupAllowFrom` (guruh yuboruvchilari uchun ruxsat ro‘yxati).
- `channels.whatsapp.groupPolicy` (guruh siyosati).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (guruh tarixiy konteksti; `0` o‘chiradi).
- `channels.whatsapp.dmHistoryLimit` (DM history limit in user turns). Per-user overrides: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (group allowlist + mention gating defaults; use `"*"` to allow all)
- `channels.whatsapp.actions.reactions` (gate WhatsApp tool reactions).
- `agents.list[].groupChat.mentionPatterns` (or `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (inbound prefix; per-account: `channels.whatsapp.accounts.<accountId>.messagePrefix`; deprecated: `messages.messagePrefix`)
- `messages.responsePrefix` (outbound prefix)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (optional override)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (per-agent overrides)
- `session.*` (scope, idle, store, mainKey)
- `web.enabled` (disable channel startup when false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Logs + troubleshooting

- Subsystems: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Log file: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurable).
- Troubleshooting guide: [Gateway troubleshooting](/gateway/troubleshooting).

## Troubleshooting (quick)

**Not linked / QR login required**

- Symptom: `channels status` shows `linked: false` or warns “Not linked”.
- Fix: run `openclaw channels login` on the gateway host and scan the QR (WhatsApp → Settings → Linked Devices).

**Linked but disconnected / reconnect loop**

- Symptom: `channels status` shows `running, disconnected` or warns “Linked but disconnected”.
- Fix: `openclaw doctor` (or restart the gateway). If it persists, relink via `channels login` and inspect `openclaw logs --follow`.

**Bun runtime**

- Bun is **not recommended**. WhatsApp (Baileys) and Telegram are unreliable on Bun.
  Run the gateway with **Node**. (See Getting Started runtime note.)
