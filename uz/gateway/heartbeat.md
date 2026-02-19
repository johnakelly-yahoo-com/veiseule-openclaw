---
summary: "Heartbeat polling xabarlari va bildirishnoma qoidalari"
read_when:
  - Heartbeat intervali yoki xabar matnini sozlayotganda
  - Rejalashtirilgan vazifalar uchun heartbeat va cron o‘rtasida tanlov qilayotganda
title: "Heartbeat"
---

# Yurak urishi (Gateway)

> **Heartbeat yoki Cron?** Qaysi birini qachon ishlatish bo‘yicha yo‘riqnoma uchun [Cron vs Heartbeat](/automation/cron-vs-heartbeat) sahifasiga qarang.

Heartbeat asosiy sessiyada **davriy agent yurishlarini** ishga tushiradi, shunda model sizni bezovta qilmasdan e’tibor talab qiladigan narsalarni ko‘rsatib bera oladi.

Nosozliklarni aniqlash: [/automation/troubleshooting](/automation/troubleshooting)

## Tezkor boshlash (boshlovchilar uchun)

1. Heartbeat’ni yoqilgan holda qoldiring (standart `30m`, yoki Anthropic OAuth/setup-token uchun `1h`) yoki o‘zingiz interval belgilang.
2. Agent ish maydonida kichik `HEARTBEAT.md` cheklist yarating (ixtiyoriy, lekin tavsiya etiladi).
3. Heartbeat xabarlari qayerga yuborilishini belgilang (`target: "last"` — standart).
4. Ixtiyoriy: shaffoflik uchun heartbeat reasoning yetkazilishini yoqing.
5. Ixtiyoriy: heartbeat’ni faqat faol soatlar bilan cheklang (mahalliy vaqt).

Misol konfiguratsiya:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        // activeHours: { start: "08:00", end: "24:00" },
        // includeReasoning: true, // ixtiyoriy: alohida `Reasoning:` xabarini ham yuboradi
      },
    },
  },
}
```

## Standart sozlamalar

- Interval: `30m` (or `1h` when Anthropic OAuth/setup-token is the detected auth mode). Set `agents.defaults.heartbeat.every` or per-agent `agents.list[].heartbeat.every`; use `0m` to disable.
- Prompt body (configurable via `agents.defaults.heartbeat.prompt`):
  `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`
- The heartbeat prompt is sent **verbatim** as the user message. The system
  prompt includes a “Heartbeat” section and the run is flagged internally.
- Active hours (`heartbeat.activeHours`) are checked in the configured timezone.
  Outside the window, heartbeats are skipped until the next tick inside the window.

## Heartbeat prompti nima uchun kerak

Standart prompt ataylab keng qamrovli:

- **Fon vazifalari**: “Consider outstanding tasks” agentni ochiq qolgan ishlarni (inbox, kalendar, eslatmalar, navbatdagi ishlar) ko‘rib chiqishga va shoshilinchlarini ko‘rsatishga undaydi.
- **Inson bilan aloqa**: “Checkup sometimes on your human during day time” vaqti-vaqti bilan yengil “sizga nimadir kerakmi?” xabarini yuborishga undaydi, lekin kechasi spam bo‘lmasligi uchun siz sozlagan mahalliy vaqt zonasidan foydalanadi (qarang: [/concepts/timezone](/concepts/timezone)).

Agar heartbeat juda aniq bir ishni bajarishi kerak bo‘lsa (masalan, “Gmail PubSub statistikasini tekshir” yoki “gateway holatini tasdiqla”), `agents.defaults.heartbeat.prompt` (yoki `agents.list[].heartbeat.prompt`) ni maxsus matn bilan belgilang (o‘zgarishsiz yuboriladi).

## Javob shartnomasi

- Agar e’tibor talab qiladigan narsa bo‘lmasa, **`HEARTBEAT_OK`** bilan javob bering.
- During heartbeat runs, OpenClaw treats `HEARTBEAT_OK` as an ack when it appears
  at the **start or end** of the reply. The token is stripped and the reply is
  dropped if the remaining content is **≤ `ackMaxChars`** (default: 300).
- Agar `HEARTBEAT_OK` javobning **o‘rtasida** bo‘lsa, u maxsus talqin qilinmaydi.
- Ogohlantirishlar uchun **`HEARTBEAT_OK` ni qo‘shmang**; faqat ogohlantirish matnini qaytaring.

Heartbeat’dan tashqarida, xabar boshida/oxirida tasodifan kelgan `HEARTBEAT_OK` olib tashlanadi va log qilinadi; faqat `HEARTBEAT_OK` dan iborat xabar yuborilmaydi.

## Config

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // standart: 30m (0m o‘chiradi)
        model: "anthropic/claude-opus-4-6",
        includeReasoning: false, // standart: false (mavjud bo‘lsa alohida Reasoning: xabar yuboradi)
        target: "last", // last | none | <channel id> (core yoki plugin, masalan "bluebubbles")
        to: "+15551234567", // ixtiyoriy kanalga xos override
        accountId: "ops-bot", // ixtiyoriy multi-account kanal id
        prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        ackMaxChars: 300, // HEARTBEAT_OK dan keyin ruxsat etilgan maksimal belgilar soni
      },
    },
  },
}
```

### Qamrov va ustuvorlik

- `agents.defaults.heartbeat` global heartbeat xatti-harakatini belgilaydi.
- `agents.list[].heartbeat` ustiga qo‘shiladi; agar biror agentda `heartbeat` bloki bo‘lsa, **faqat o‘sha agentlar** heartbeat ishga tushiradi.
- `channels.defaults.heartbeat` barcha kanallar uchun ko‘rinish standartlarini belgilaydi.
- `channels.<channel>.heartbeat` overrides channel defaults.
- `channels.<channel>.accounts.<id>.heartbeat` (multi-account channels) overrides per-channel settings.

### Har bir agent uchun heartbeat

If any `agents.list[]` entry includes a `heartbeat` block, **only those agents**
run heartbeats. The per-agent block merges on top of `agents.defaults.heartbeat`
(so you can set shared defaults once and override per agent).

Misol: ikki agent, faqat ikkinchi agent heartbeat ishga tushiradi.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
    list: [
      { id: "main", default: true },
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "whatsapp",
          to: "+15551234567",
          prompt: "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        },
      },
    ],
  },
}
```

### Faol soatlar misoli

Heartbeat’ni ma’lum vaqt zonasidagi ish soatlari bilan cheklash:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
        activeHours: {
          start: "09:00",
          end: "22:00",
          timezone: "America/New_York", // ixtiyoriy; agar userTimezone o‘rnatilgan bo‘lsa o‘sha, aks holda host tz
        },
      },
    },
  },
}
```

Outside this window (before 9am or after 10pm Eastern), heartbeats are skipped. The next scheduled tick inside the window will run normally.

### Multi account misoli

Telegram kabi multi-account kanallarda ma’lum akkauntni tanlash uchun `accountId` dan foydalaning:

```json5
{
  agents: {
    list: [
      {
        id: "ops",
        heartbeat: {
          every: "1h",
          target: "telegram",
          to: "12345678",
          accountId: "ops-bot",
        },
      },
    ],
  },
  channels: {
    telegram: {
      accounts: {
        "ops-bot": { botToken: "YOUR_TELEGRAM_BOT_TOKEN" },
      },
    },
  },
}
```

### Maydonlar bo‘yicha izohlar

- `every`: heartbeat intervali (davomiylik satri; standart birlik = daqiqa).
- `model`: heartbeat ishga tushirishlari uchun ixtiyoriy model override (`provider/model`).
- `includeReasoning`: yoqilganda, mavjud bo‘lsa alohida `Reasoning:` xabarini ham yuboradi (`/reasoning on` bilan bir xil format).
- `session`: heartbeat ishga tushirishlari uchun ixtiyoriy sessiya kaliti.
  - `main` (standart): agentning asosiy sessiyasi.
  - Aniq sessiya kaliti (`openclaw sessions --json` yoki [sessions CLI](/cli/sessions) orqali).
  - Sessiya kaliti formatlari: qarang [Sessions](/concepts/session) va [Groups](/channels/groups).
- `target`:
  - `last` (standart): oxirgi ishlatilgan tashqi kanalga yuboradi.
  - aniq kanal: `whatsapp` / `telegram` / `discord` / `googlechat` / `slack` / `msteams` / `signal` / `imessage`.
  - `none`: heartbeat ishga tushadi, lekin **tashqariga yuborilmaydi**.
- `to`: ixtiyoriy qabul qiluvchi override (kanalga xos id, masalan WhatsApp uchun E.164 yoki Telegram chat id).
- `accountId`: optional account id for multi-account channels. When `target: "last"`, the account id applies to the resolved last channel if it supports accounts; otherwise it is ignored. If the account id does not match a configured account for the resolved channel, delivery is skipped.
- `prompt`: standart prompt matnini to‘liq almashtiradi (merge qilinmaydi).
- `ackMaxChars`: `HEARTBEAT_OK` dan keyin yuborishga ruxsat etilgan maksimal belgilar soni.
- `activeHours`: restricts heartbeat runs to a time window. Object with `start` (HH:MM, inclusive), `end` (HH:MM exclusive; `24:00` allowed for end-of-day), and optional `timezone`.
  - Kiritilmagan yoki `"user"`: agar mavjud bo‘lsa `agents.defaults.userTimezone`, aks holda host tizim vaqt zonasi.
  - `"local"`: har doim host tizim vaqt zonasi.
  - Har qanday IANA identifikatori (masalan `America/New_York`): bevosita ishlatiladi; noto‘g‘ri bo‘lsa `"user"` xatti-harakatiga qaytadi.
  - Faol oynadan tashqarida heartbeat’lar keyingi ichki tick’gacha o‘tkazib yuboriladi.

## Yetkazish xatti-harakati

- Heartbeats run in the agent’s main session by default (`agent:<id>:<mainKey>`),
  or `global` when `session.scope = "global"`. Set `session` to override to a
  specific channel session (Discord/WhatsApp/etc.).
- `session` faqat ishga tushirish kontekstiga ta’sir qiladi; yuborish `target` va `to` orqali boshqariladi.
- To deliver to a specific channel/recipient, set `target` + `to`. With
  `target: "last"`, delivery uses the last external channel for that session.
- Agar asosiy navbat band bo‘lsa, heartbeat o‘tkazib yuboriladi va keyinroq qayta urinadi.
- Agar `target` tashqi manzilga mos kelmasa, ishga tushirish amalga oshadi, lekin tashqi xabar yuborilmaydi.
- Faqat heartbeat javoblari sessiyani faol saqlab turmaydi; oxirgi `updatedAt` tiklanadi, shunda idle muddati odatdagidek ishlaydi.

## Ko‘rinish boshqaruvlari

By default, `HEARTBEAT_OK` acknowledgments are suppressed while alert content is
delivered. You can adjust this per channel or per account:

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false # HEARTBEAT_OK ni yashirish (standart)
      showAlerts: true # Ogohlantirish xabarlarini ko‘rsatish (standart)
      useIndicator: true # UI holat indikatorlarini yuborish (standart)
  telegram:
    heartbeat:
      showOk: true # Telegram’da OK tasdiqlarini ko‘rsatish
  whatsapp:
    accounts:
      work:
        heartbeat:
          showAlerts: false # Ushbu akkaunt uchun ogohlantirish yuborishni o‘chirish
```

Ustuvorlik: per-account → per-channel → kanal standartlari → ichki standartlar.

### Har bir flag nima qiladi

- `showOk`: model faqat OK javob qaytarsa, `HEARTBEAT_OK` tasdig‘ini yuboradi.
- `showAlerts`: model OK bo‘lmagan javob qaytarsa, ogohlantirish mazmunini yuboradi.
- `useIndicator`: UI holati uchun indikator hodisalarini yuboradi.

Agar **uchalasi ham** false bo‘lsa, OpenClaw heartbeat ishga tushirishni butunlay o‘tkazib yuboradi (model chaqirilmaydi).

### Per-channel va per-account misollar

```yaml
channels:
  defaults:
    heartbeat:
      showOk: false
      showAlerts: true
      useIndicator: true
  slack:
    heartbeat:
      showOk: true # barcha Slack akkauntlari
    accounts:
      ops:
        heartbeat:
          showAlerts: false # faqat ops akkaunti uchun ogohlantirishlarni o‘chirish
  telegram:
    heartbeat:
      showOk: true
```

### Keng tarqalgan andozalar

| Maqsad                                                                             | Config                                                                                   |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Standart xatti-harakat (OK yashirin, ogohlantirishlar yoqilgan) | _(config talab qilinmaydi)_                                           |
| To‘liq jim (xabar yo‘q, indikator yo‘q)                         | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: false }` |
| Faqat indikator (xabar yo‘q)                                    | `channels.defaults.heartbeat: { showOk: false, showAlerts: false, useIndicator: true }`  |
| Faqat bitta kanalda OK ko‘rsatish                                                  | `channels.telegram.heartbeat: { showOk: true }`                                          |

## HEARTBEAT.md (ixtiyoriy)

If a `HEARTBEAT.md` file exists in the workspace, the default prompt tells the
agent to read it. Think of it as your “heartbeat checklist”: small, stable, and
safe to include every 30 minutes.

If `HEARTBEAT.md` exists but is effectively empty (only blank lines and markdown
headers like `# Heading`), OpenClaw skips the heartbeat run to save API calls.
If the file is missing, the heartbeat still runs and the model decides what to do.

Prompt hajmi oshib ketmasligi uchun uni kichik saqlang (qisqa cheklist yoki eslatmalar).

Misol `HEARTBEAT.md`:

```md
# Heartbeat checklist

- Quick scan: anything urgent in inboxes?
- If it’s daytime, do a lightweight check-in if nothing else is pending.
- If a task is blocked, write down _what is missing_ and ask Peter next time.
```

### Agent HEARTBEAT.md ni yangilay oladimi?

Ha — agar siz undan so‘rasangiz.

`HEARTBEAT.md` agent ish maydonidagi oddiy fayl, shuning uchun oddiy chatda shunday deyishingiz mumkin:

- “`HEARTBEAT.md` ga har kunlik kalendar tekshiruvini qo‘sh.”
- “`HEARTBEAT.md` ni qisqaroq qilib, inbox follow-up’lariga yo‘naltirib qayta yoz.”

Agar bu jarayon proaktiv bo‘lishini istasangiz, heartbeat promptiga shunday qatorda qo‘shishingiz mumkin: “Agar cheklist eskirsa, HEARTBEAT.md ni yaxshiroq variant bilan yangila.”

Xavfsizlik eslatmasi: maxfiy ma’lumotlarni (API kalitlari, telefon raqamlari, private tokenlar) `HEARTBEAT.md` ichiga qo‘ymang — u prompt kontekstining bir qismiga aylanadi.

## Qo‘lda uyg‘otish (talab bo‘yicha)

Quyidagi buyruq bilan tizim hodisasini navbatga qo‘yib, darhol heartbeat ishga tushirishingiz mumkin:

```bash
openclaw system event --text "Check for urgent follow-ups" --mode now
```

Agar bir nechta agentda `heartbeat` sozlangan bo‘lsa, qo‘lda uyg‘otish ularning har birini darhol ishga tushiradi.

Keyingi rejalashtirilgan tick’ni kutish uchun `--mode next-heartbeat` dan foydalaning.

## Reasoning yetkazish (ixtiyoriy)

Standart bo‘yicha, heartbeat faqat yakuniy “answer” yuklamasini yuboradi.

Shaffoflik kerak bo‘lsa, quyidagini yoqing:

- `agents.defaults.heartbeat.includeReasoning: true`

When enabled, heartbeats will also deliver a separate message prefixed
`Reasoning:` (same shape as `/reasoning on`). This can be useful when the agent
is managing multiple sessions/codexes and you want to see why it decided to ping
you — but it can also leak more internal detail than you want. Prefer keeping it
off in group chats.

## Xarajatni hisobga olish

Heartbeats run full agent turns. Shorter intervals burn more tokens. Keep
`HEARTBEAT.md` small and consider a cheaper `model` or `target: "none"` if you
only want internal state updates.

