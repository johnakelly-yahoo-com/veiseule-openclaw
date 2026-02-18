---
title: "Guruh xabarlari"
---

# Guruh xabarlari (WhatsApp web kanali)

Maqsad: Clawd’ni WhatsApp guruhlariga qo‘shish, faqat chaqirilganda uyg‘otish va ushbu mavzuni shaxsiy DM sessiyasidan alohida saqlash.

Eslatma: `agents.list[].groupChat.mentionPatterns` endi Telegram/Discord/Slack/iMessage uchun ham qo‘llaniladi; ushbu hujjat WhatsApp’ga xos xatti-harakatlarga qaratilgan. Ko‘p agentli sozlamalarda har bir agent uchun `agents.list[].groupChat.mentionPatterns` ni belgilang (yoki global zaxira sifatida `messages.groupChat.mentionPatterns` dan foydalaning).

## Nimalar joriy etilgan (2025-12-03)

- Faollashtirish rejimlari: `mention` (standart) yoki `always`. `mention` ping talab qiladi (haqiqiy WhatsApp @-eslatmalari `mentionedJids` orqali, regex andozalari yoki botning E.164 raqami matnning istalgan joyida). `always` har bir xabarda agentni uyg‘otadi, biroq u faqat mazmunli foyda qo‘sha olganda javob berishi kerak; aks holda `NO_REPLY` jimlik tokenini qaytaradi. Standart qiymatlar konfiguratsiyada (`channels.whatsapp.groups`) o‘rnatiladi va har bir guruh uchun `/activation` orqali bekor qilinishi mumkin. `channels.whatsapp.groups` o‘rnatilganda, u guruh allowlist sifatida ham ishlaydi (`"*"` qo‘shilsa barcha guruhlarga ruxsat beriladi).
- Guruh siyosati: `channels.whatsapp.groupPolicy` guruh xabarlari qabul qilinishini boshqaradi (`open|disabled|allowlist`). `allowlist` uchun `channels.whatsapp.groupAllowFrom` ishlatiladi (zaxira: aniq ko‘rsatilgan `channels.whatsapp.allowFrom`). Standart — `allowlist` (jo‘natuvchilarni qo‘shmaguningizcha bloklanadi).
- Har bir guruh uchun alohida sessiyalar: sessiya kalitlari `agent:<agentId>:whatsapp:group:<jid>` ko‘rinishida bo‘ladi, shuning uchun `/verbose on` yoki `/think high` kabi buyruqlar (alohida xabar sifatida yuborilganda) faqat shu guruhga taalluqli bo‘ladi; shaxsiy DM holati o‘zgarmaydi. Heartbeat’lar guruh mavzulari uchun o‘tkazib yuboriladi.
- Kontekstni kiritish: ishga tushirishni _qo‘zg‘atmagan_ **faqat kutilayotgan** guruh xabarlari (standart 50 ta) `[Chat messages since your last reply - for context]` ostida prefiks qilinadi, qo‘zg‘atuvchi qator esa `[Current message - respond to this]` ostida beriladi. Sessiyada allaqachon mavjud xabarlar qayta kiritilmaydi.
- Jo‘natuvchini ko‘rsatish: har bir guruh batch’i endi `[from: Sender Name (+E164)]` bilan yakunlanadi, shunda Pi kim gapirayotganini biladi.
- Ephemeral/view-once: matn/mentionlarni ajratib olishdan oldin ular ochiladi, shuning uchun ichidagi pinglar ham ishga tushiradi.
- Guruh system prompt’i: guruh sessiyasining birinchi bosqichida (va har safar `/activation` rejimi o‘zgarganda) system prompt’ga qisqa izoh qo‘shiladi, masalan: `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` Agar metadata mavjud bo‘lmasa ham, agentga bu guruh chat ekanini bildiramiz.

## Konfiguratsiya namunasi (WhatsApp)

WhatsApp matn tanasida vizual `@` belgisi olib tashlangan holatda ham ko‘rinadigan ism orqali ping ishlashi uchun `~/.openclaw/openclaw.json` fayliga `groupChat` blokini qo‘shing:

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: ["@?openclaw", "\\+?15555550123"],
        },
      },
    ],
  },
}
```

Eslatmalar:

- Regex’lar katta-kichik harflarga sezgir emas; ular `@openclaw` kabi ko‘rinadigan ism orqali pingni va `+`/bo‘sh joy bilan yoki ularsiz xom raqamni qamrab oladi.
- Kimdir kontaktni bosganda WhatsApp baribir `mentionedJids` orqali kanonik mention yuboradi, shuning uchun raqam bo‘yicha zaxira kamdan-kam kerak bo‘ladi, ammo foydali xavfsizlik chorasi hisoblanadi.

### Faollashtirish buyrug‘i (faqat egasi)

Guruh chat buyrug‘idan foydalaning:

- `/activation mention`
- `/activation always`

Buni faqat egasining raqami (`channels.whatsapp.allowFrom`, yoki o‘rnatilmagan bo‘lsa botning o‘z E.164 raqami) o‘zgartira oladi. Joriy faollashtirish rejimini ko‘rish uchun guruhga `/status` ni alohida xabar sifatida yuboring.

## Qanday foydalaniladi

1. OpenClaw ishlayotgan WhatsApp akkauntini guruhga qo‘shing.
2. `@openclaw …` deb yozing (yoki raqamni kiriting). `groupPolicy: "open"` o‘rnatilmagan bo‘lsa, faqat allowlist’dagi jo‘natuvchilar ishga tushira oladi.
3. Agent prompt’i yaqindagi guruh konteksti va oxiridagi `[from: …]` belgisini o‘z ichiga oladi, shuning uchun to‘g‘ri shaxsga murojaat qila oladi.
4. Sessiya darajasidagi direktivalar (`/verbose on`, `/think high`, `/new` yoki `/reset`, `/compact`) faqat shu guruh sessiyasiga taalluqli; ro‘yxatdan o‘tishi uchun ularni alohida xabar sifatida yuboring. Shaxsiy DM sessiyangiz mustaqil qoladi.

## Sinov / tekshirish

- Qo‘lda tezkor sinov:
  - Guruhda `@openclaw` ping yuboring va jo‘natuvchi nomini tilga olgan javobni tasdiqlang.
  - Ikkinchi ping yuboring va history bloki kiritilganini, keyingi bosqichda esa tozalanganini tekshiring.
- Gateway loglarini tekshiring (`--verbose` bilan ishga tushiring) — `inbound web message` yozuvlarida `from: <groupJid>` va `[from: …]` suffiksi ko‘rinadi.

## Ma’lum jihatlar

- Shovqinli e’lonlarning oldini olish uchun heartbeat’lar guruhlar uchun ataylab o‘chirib qo‘yilgan.
- Echo suppression birlashtirilgan batch satridan foydalanadi; agar mentionlarsiz bir xil matnni ikki marta yuborsangiz, faqat birinchisiga javob beriladi.
- Sessiya saqlash yozuvlari sessiya omborida (`~/.openclaw/agents/<agentId>/sessions/sessions.json` standart bo‘yicha) `agent:<agentId>:whatsapp:group:<jid>` ko‘rinishida paydo bo‘ladi; yozuv yo‘qligi guruh hali ishga tushirmaganini anglatadi.
- Guruhlardagi typing indikatorlari `agents.defaults.typingMode` ga amal qiladi (standart: mention qilinmaganda `message`).

