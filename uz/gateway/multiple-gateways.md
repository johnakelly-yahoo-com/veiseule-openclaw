---
title: "Bir nechta Gateway"
---

# Bir nechta Gateway (bir xil xost)

Ko‘pchilik holatlarda bitta Gateway yetarli bo‘ladi, chunki u bir nechta xabar almashish ulanishlari va agentlarni boshqara oladi. Agar sizga kuchliroq izolyatsiya yoki zaxira (masalan, qutqaruv boti) kerak bo‘lsa, alohida profil/portlarga ega mustaqil Gateway’larni ishga tushiring.

## Izolyatsiya ro‘yxati (majburiy)

- `OPENCLAW_CONFIG_PATH` — har bir instansiya uchun alohida config fayl
- `OPENCLAW_STATE_DIR` — har bir instansiya uchun sessiyalar, credential’lar, keshlar
- `agents.defaults.workspace` — har bir instansiya uchun workspace ildiz papkasi
- `gateway.port` (yoki `--port`) — har bir instansiya uchun noyob
- Hosila (derived) portlar (browser/canvas) bir-biriga to‘qnashmasligi kerak

Agar bular umumiy bo‘lsa, config kolliziyalari va port mojarolariga duch kelasiz.

## Tavsiya etiladi: profillar (`--profile`)

Profillar `OPENCLAW_STATE_DIR` va `OPENCLAW_CONFIG_PATH` ni avtomatik ravishda ajratadi hamda servis nomlariga suffiks qo‘shadi.

```bash
# main
openclaw --profile main setup
openclaw --profile main gateway --port 18789

# rescue
openclaw --profile rescue setup
openclaw --profile rescue gateway --port 19001
```

Har bir profil uchun servislar:

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

## Qutqaruv-bot qo‘llanmasi

Bir xil xostda ikkinchi Gateway’ni quyidagilar bilan ishga tushiring:

- alohida profil/config
- alohida state papkasi
- alohida workspace
- alohida asosiy port (va unga bog‘liq hosila portlar)

Bu qutqaruv botini asosiy botdan izolyatsiya qiladi, shunda asosiy bot ishlamay qolsa, uni diagnostika qilish yoki config o‘zgartirish mumkin bo‘ladi.

Port oralig‘i: asosiy portlar orasida kamida 20 ta port farqi qoldiring, shunda hosila browser/canvas/CDP portlari hech qachon to‘qnashmaydi.

### O‘rnatish tartibi (qutqaruv bot)

```bash
# Asosiy bot (mavjud yoki yangi, --profile parametrisiz)
# 18789 portda + Chrome CDC/Canvas/... portlari bilan ishlaydi
openclaw onboard
openclaw gateway install

# Qutqaruv bot (izolyatsiyalangan profil + portlar)
openclaw --profile rescue onboard
# Eslatmalar:
# - workspace nomi odatda -rescue suffiksi bilan tugaydi
# - Port kamida 18789 + 20 port bo‘lishi kerak,
#   yaxshisi butunlay boshqa asosiy port tanlang, masalan 19789,
# - qolgan onboarding jarayoni odatdagidek

# Servisni o‘rnatish (agar onboarding paytida avtomatik bo‘lmagan bo‘lsa)
openclaw --profile rescue gateway install
```

## Port xaritasi (hosila)

Asosiy port = `gateway.port` (yoki `OPENCLAW_GATEWAY_PORT` / `--port`).

- browser boshqaruv servisi porti = asosiy + 2 (faqat loopback)
- canvas xosti Gateway HTTP serverida xizmat ko‘rsatiladi (`gateway.port` bilan bir xil port)
- Browser profil CDP portlari `browser.controlPort + 9 .. + 108` oralig‘idan avtomatik ajratiladi

Agar config yoki env orqali ulardan birini o‘zgartirsangiz, har bir instansiya uchun noyobligini saqlashingiz shart.

## Browser/CDP bo‘yicha eslatmalar (keng tarqalgan xato)

- `browser.cdpUrl` ni bir nechta instansiyada bir xil qiymatga **mahkamlab qo‘ymang**.
- Har bir instansiya o‘zining browser control porti va CDP oralig‘iga ega bo‘lishi kerak (gateway portidan hosil qilinadi).
- Agar aniq CDP portlar kerak bo‘lsa, har bir instansiya uchun `browser.profiles.<name>.cdpPort` ni belgilang.
- Masofaviy Chrome: `browser.profiles.<name>.cdpUrl` dan foydalaning (har profil, har instansiya uchun).

## Qo‘lda env misoli

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/main.json \
OPENCLAW_STATE_DIR=~/.openclaw-main \
openclaw gateway --port 18789

OPENCLAW_CONFIG_PATH=~/.openclaw/rescue.json \
OPENCLAW_STATE_DIR=~/.openclaw-rescue \
openclaw gateway --port 19001
```

## Tezkor tekshiruvlar

```bash
openclaw --profile main status
openclaw --profile rescue status
openclaw --profile rescue browser status
```
