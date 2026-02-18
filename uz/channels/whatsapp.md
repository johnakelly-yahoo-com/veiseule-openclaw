---
title: "WhatsApp"
---

# WhatsApp (web channel)

Status: WhatsApp Web orqali Baileys yordamida production-ready. Gateway ulangan session(lar)ga egalik qiladi.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Noma’lum yuboruvchilar uchun standart DM siyosati — pairing.
  </Card>
  <Card title="Kanal troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallararo diagnostika va tuzatish bo‘yicha qo‘llanmalar.
  </Card>
  <Card title="Gateway konfiguratsiyasi" icon="settings" href="/gateway/configuration">
    To‘liq kanal konfiguratsiya naqshlari va misollar.
  </Card>
</CardGroup>

## Quick setup

<Steps>
  <Step title="WhatsApp access policy sozlang">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing",
      allowFrom: ["+15551234567"],
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
}
```

  </Step>

  <Step title="WhatsApp’ni ulang (QR)">

```bash
openclaw channels login --channel whatsapp
```

    Muayyan akkaunt uchun:

```bash
openclaw channels login --channel whatsapp --account work
```

  </Step>

  <Step title="Gateway’ni ishga tushiring">

```bash
openclaw gateway
```

  </Step>

  <Step title="Birinchi pairing so‘rovini tasdiqlang (agar pairing rejimi ishlatilsa)">

```bash
openclaw pairing list whatsapp
openclaw pairing approve whatsapp <CODE>
```

    Pairing so‘rovlari 1 soatdan keyin muddati tugaydi. Kutilayotgan so‘rovlar har bir kanal uchun 3 ta bilan cheklangan.

  </Step>
</Steps>

<Note>
OpenClaw imkon qadar WhatsApp’ni alohida telefon raqamida ishlatishni tavsiya qiladi. (Kanal metama’lumotlari va onboarding oqimi shu sozlama uchun optimallashtirilgan, ammo shaxsiy raqam bilan ishlash ham qo‘llab-quvvatlanadi.)
</Note>

## Deployment patterns

<AccordionGroup>
  <Accordion title="Maxsus raqam (tavsiya etiladi)">
    Eng toza operatsion rejim:

    - OpenClaw uchun alohida WhatsApp identifikatori
    - aniqroq DM allowlist va marshrutlash chegaralari
    - o‘zingiz bilan chat qilishdagi chalkashlik ehtimoli past

    Minimal siyosat namunasi:

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

  </Accordion>

  <Accordion title="Shaxsiy raqam (fallback)">
    Onboarding shaxsiy raqam rejimini qo‘llab-quvvatlaydi va self-chat uchun mos boshlang‘ich sozlamani yozadi:

    - `dmPolicy: "allowlist"`
    - `allowFrom` sizning shaxsiy raqamingizni o‘z ichiga oladi
    - `selfChatMode: true`

    Runtime’da self-chat himoyasi ulangan self raqam va `allowFrom` asosida ishlaydi.

  </Accordion>

  <Accordion title="WhatsApp Web-only kanal qamrovi">
    Joriy OpenClaw arxitekturasida xabar almashish kanali WhatsApp Web (`Baileys`) asosida ishlaydi.

    Built-in chat-channel registry’da alohida Twilio WhatsApp messaging kanali mavjud emas.
  </Accordion>
</AccordionGroup>

## Runtime model

- Gateway WhatsApp soketi va reconnect loop’ga egalik qiladi.
- Chiquvchi xabar yuborish uchun maqsad akkauntda faol WhatsApp listener bo‘lishi shart.
- Status va broadcast chatlar e’tiborsiz qoldiriladi (`@status`, `@broadcast`).
- Direct chatlar DM session qoidalaridan foydalanadi (`session.dmScope`; standart `main` barcha DM’larni agentning asosiy sessiyasiga birlashtiradi).
- Guruh sessiyalari izolyatsiyalangan (`agent:<agentId>:whatsapp:group:<jid>`).

## Access control and activation

<Tabs>
  <Tab title="DM policy">
    `channels.whatsapp.dmPolicy` direct chat kirishini boshqaradi:

    - `pairing` (standart)
    - `allowlist`
    - `open` (`allowFrom` ichida `"*"` bo‘lishi talab etiladi)
    - `disabled`

    `allowFrom` E.164 formatidagi telefon raqamlarini qabul qiladi (ichki normalizatsiya qilinadi).

    Multi-account override: `channels.whatsapp.accounts.<id>.dmPolicy` (va `allowFrom`) kanal darajasidagi standart sozlamalardan ustun turadi.

    Runtime tafsilotlari:

    - pairing’lar kanal allow-store’da saqlanadi va konfiguratsiyadagi `allowFrom` bilan birlashtiriladi
    - agar allowlist sozlanmagan bo‘lsa, ulangan self raqam sukut bo‘yicha ruxsat etiladi
    - chiquvchi `fromMe` DM’lar hech qachon avtomatik pairing qilinmaydi

  </Tab>

  <Tab title="Group policy + allowlists">
    Guruh kirishi ikki qatlamdan iborat:

    1. **Guruh membership allowlist** (`channels.whatsapp.groups`)
       - agar `groups` ko‘rsatilmagan bo‘lsa, barcha guruhlar mos keladi
       - agar `groups` mavjud bo‘lsa, u guruh allowlist sifatida ishlaydi (`"*"` ruxsat etiladi)

    2. **Guruh yuboruvchi siyosati** (`channels.whatsapp.groupPolicy` + `groupAllowFrom`)
       - `open`: yuboruvchi allowlist tekshirilmaydi
       - `allowlist`: yuboruvchi `groupAllowFrom` (yoki `*`) bilan mos bo‘lishi kerak
       - `disabled`: barcha guruh kiruvchi xabarlar bloklanadi

    Sender allowlist fallback:

    - agar `groupAllowFrom` o‘rnatilmagan bo‘lsa, mavjud bo‘lsa `allowFrom` ga fallback qilinadi

    Eslatma: agar umuman `channels.whatsapp` bloki mavjud bo‘lmasa, runtime group-policy fallback amalda `open` bo‘ladi.

  </Tab>

  <Tab title="Mentions + /activation">
    Guruh javoblari sukut bo‘yicha mention talab qiladi.

    Mention aniqlash quyidagilarni o‘z ichiga oladi:

    - bot identifikatoriga aniq WhatsApp mention
    - sozlangan mention regex pattern’lar (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - botga javob (reply) orqali implicit aniqlash (reply yuboruvchisi bot identifikatori bilan mos)

    Sessiya darajasidagi activation buyruqlari:

    - `/activation mention`
    - `/activation always`

    `activation` global konfiguratsiyani emas, sessiya holatini yangilaydi. Faqat owner tomonidan bajarilishi mumkin.

  </Tab>
</Tabs>

## Related

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)

