---
title: "Shaxsiy yordamchini sozlash"
---

# OpenClaw yordamida shaxsiy yordamchi yaratish

OpenClaw — bu **Pi** agentlari uchun WhatsApp + Telegram + Discord + iMessage gateway. Pluginlar Mattermost qo‘shadi. Ushbu qo‘llanma “shaxsiy yordamchi” sozlamasi haqida: doimo faol bo‘ladigan agent sifatida ishlaydigan bitta alohida WhatsApp raqami.

## ⚠️ Avvalo xavfsizlik

Siz agentga quyidagi imkoniyatlarni berasiz:

- kompyuteringizda buyruqlar bajarish (Pi tool sozlamangizga qarab)
- ish muhitingizdagi fayllarni o‘qish/yozish
- WhatsApp/Telegram/Discord/Mattermost (plugin) orqali tashqariga xabar yuborish

Ehtiyotkorlik bilan boshlang:

- Har doim `channels.whatsapp.allowFrom` ni sozlang (shaxsiy Mac’ingizda hech qachon hammaga ochiq rejimda ishlatmang).
- Yordamchi uchun alohida WhatsApp raqamidan foydalaning.
- Heartbeat sukut bo‘yicha har 30 daqiqada ishlaydi. Sozlamaga ishonch hosil qilmaguningizcha uni `agents.defaults.heartbeat.every: "0m"` qilib o‘chirib qo‘ying.

## Talablar

- OpenClaw o‘rnatilgan va dastlabki sozlamalar bajarilgan — agar hali qilmagan bo‘lsangiz, [Getting Started](/start/getting-started) ni ko‘ring
- Yordamchi uchun ikkinchi telefon raqami (SIM/eSIM/prepaid)

## Ikki telefonli sozlama (tavsiya etiladi)

Sizga quyidagisi kerak:

```mermaid
flowchart TB
    A["<b>Sizning telefoningiz (shaxsiy)<br></b><br>Sizning WhatsApp<br>+1-555-YOU"] -- message --> B["<b>Ikkinchi telefon (yordamchi)<br></b><br>Yordamchi WA<br>+1-555-ASSIST"]
    B -- linked via QR --> C["<b>Sizning Mac (openclaw)<br></b><br>Pi agent"]
```

Agar shaxsiy WhatsApp’ingizni OpenClaw’ga ulasangiz, sizga kelgan har bir xabar “agent input” ga aylanadi. Odatda bu siz istagan narsa emas.

## 5 daqiqalik tezkor boshlash

1. WhatsApp Web’ni ulang (QR ko‘rsatadi; yordamchi telefon bilan skaner qiling):

```bash
openclaw channels login
```

2. Gateway’ni ishga tushiring (ochiq qoldiring):

```bash
openclaw gateway --port 18789
```

3. `~/.openclaw/openclaw.json` fayliga minimal konfiguratsiya yozing:

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Endi ruxsat berilgan telefoningizdan yordamchi raqamiga xabar yuboring.

Onboarding tugagach, dashboard avtomatik ochiladi va toza (tokensiz) havola ko‘rsatiladi. Agar autentifikatsiya so‘rasa, `gateway.auth.token` dagi tokenni Control UI sozlamalariga joylashtiring. Keyinroq qayta ochish uchun: `openclaw dashboard`.

## Agent uchun ish maydoni (AGENTS)

OpenClaw ish ko‘rsatmalari va “xotira”ni workspace katalogidan o‘qiydi.

Sukut bo‘yicha OpenClaw agent workspace sifatida `~/.openclaw/workspace` dan foydalanadi va uni (hamda boshlang‘ich `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` fayllarini) sozlash/agent birinchi marta ishga tushganda avtomatik yaratadi. `BOOTSTRAP.md` faqat workspace mutlaqo yangi bo‘lganda yaratiladi (o‘chirib tashlaganingizdan keyin qayta paydo bo‘lmasligi kerak). `MEMORY.md` ixtiyoriy (avtomatik yaratilmaydi); mavjud bo‘lsa, oddiy sessiyalarda yuklanadi. Subagent sessiyalari faqat `AGENTS.md` va `TOOLS.md` ni qo‘shadi.

Maslahat: bu papkani OpenClaw’ning “xotirasi” deb biling va `AGENTS.md` hamda xotira fayllaringiz zaxiralangan bo‘lishi uchun uni git reposiga (imkon qadar private) aylantiring. Agar git o‘rnatilgan bo‘lsa, yangi workspace avtomatik initsializatsiya qilinadi.

```bash
openclaw setup
```

To‘liq workspace tuzilmasi va zaxiralash qo‘llanmasi: [Agent workspace](/concepts/agent-workspace)  
Xotira jarayoni: [Memory](/concepts/memory)

Ixtiyoriy: boshqa workspace tanlash uchun `agents.defaults.workspace` dan foydalaning (`~` qo‘llab-quvvatlanadi).

```json5
{
  agent: {
    workspace: "~/.openclaw/workspace",
  },
}
```

Agar siz allaqachon o‘z workspace fayllaringizni repodan yetkazib bersangiz, bootstrap fayllarini yaratishni to‘liq o‘chirishingiz mumkin:

```json5
{
  agent: {
    skipBootstrap: true,
  },
}
```

## Uni “yordamchi”ga aylantiradigan konfiguratsiya

OpenClaw sukut bo‘yicha yaxshi yordamchi sozlamalari bilan keladi, ammo odatda quyidagilarni moslashtirasiz:

- `SOUL.md` dagi persona/ko‘rsatmalar
- thinking default sozlamalari (agar kerak bo‘lsa)
- heartbeat (ishga ishonch hosil qilganingizdan so‘ng)

Misol:

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-6",
    workspace: "~/.openclaw/workspace",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // Avval 0 dan boshlang; keyinroq yoqing.
    heartbeat: { every: "0m" },
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@openclaw", "openclaw"],
    },
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080,
    },
  },
}
```

## Sessiyalar va xotira

- Sessiya fayllari: `~/.openclaw/agents/<agentId>/sessions/{{SessionId}}.jsonl`
- Sessiya metama’lumotlari (token sarfi, oxirgi route va boshqalar): `~/.openclaw/agents/<agentId>/sessions/sessions.json` (legacy: `~/.openclaw/sessions/sessions.json`)
- `/new` yoki `/reset` ushbu chat uchun yangi sessiyani boshlaydi (`resetTriggers` orqali sozlanadi). Agar yolg‘iz yuborilsa, agent reset tasdiqlash uchun qisqa salom bilan javob beradi.
- `/compact [instructions]` sessiya kontekstini qisqartiradi va qolgan kontekst budjetini ko‘rsatadi.

## Heartbeat (proaktiv rejim)

Sukut bo‘yicha OpenClaw har 30 daqiqada quyidagi prompt bilan heartbeat ishga tushiradi:  
`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`  
O‘chirish uchun `agents.defaults.heartbeat.every: "0m"` ni sozlang.

- Agar `HEARTBEAT.md` mavjud bo‘lsa, lekin amalda bo‘sh (faqat bo‘sh qatorlar va `# Heading` kabi markdown sarlavhalardan iborat) bo‘lsa, OpenClaw API chaqiruvlarini tejash uchun heartbeat’ni o‘tkazib yuboradi.
- Agar fayl mavjud bo‘lmasa, heartbeat baribir ishlaydi va model nima qilishni o‘zi hal qiladi.
- Agar agent `HEARTBEAT_OK` bilan javob bersa (ixtiyoriy qisqa qo‘shimcha matn bilan; qarang `agents.defaults.heartbeat.ackMaxChars`), OpenClaw ushbu heartbeat uchun tashqi yuborishni bekor qiladi.
- Heartbeat to‘liq agent turn sifatida ishlaydi — qisqaroq interval ko‘proq token sarflaydi.

```json5
{
  agent: {
    heartbeat: { every: "30m" },
  },
}
```

## Media kirish va chiqish

Kirishdagi biriktirmalar (rasmlar/audio/hujjatlar) buyruqingizga template’lar orqali uzatilishi mumkin:

- `{{MediaPath}}` (lokal vaqtinchalik fayl yo‘li)
- `{{MediaUrl}}` (pseudo-URL)
- `{{Transcript}}` (agar audio transkripsiya yoqilgan bo‘lsa)

Agentdan chiqishdagi biriktirmalar: alohida qatorda `MEDIA:<path-or-url>` ni kiriting (bo‘sh joysiz). Misol:

```
Mana skrinshot.
MEDIA:https://example.com/screenshot.png
```

OpenClaw ularni ajratib oladi va matn bilan birga media sifatida yuboradi.

## Operatsion tekshiruv ro‘yxati

```bash
openclaw status          # lokal holat (creds, sessiyalar, navbatdagi eventlar)
openclaw status --all    # to‘liq diagnostika (faqat o‘qish, ulashish mumkin)
openclaw status --deep   # gateway health probe qo‘shadi (Telegram + Discord)
openclaw health --json   # gateway health snapshot (WS)
```

Loglar `/tmp/openclaw/` ostida joylashadi (sukut bo‘yicha: `openclaw-YYYY-MM-DD.log`).

## Keyingi qadamlar

- WebChat: [WebChat](/web/webchat)
- Gateway operatsiyalari: [Gateway operatsion qo'llanmasi](/gateway)
- Cron + wakeups: [Cron jobs](/automation/cron-jobs)
- macOS menyu paneli ilovasi: [OpenClaw macOS app](/platforms/macos)
- iOS node ilovasi: [iOS app](/platforms/ios)
- Android node ilovasi: [Android app](/platforms/android)
- Windows holati: [Windows (WSL2)](/platforms/windows)
- Linux holati: [Linux app](/platforms/linux)
- Xavfsizlik: [Security](/gateway/security)

