---
title: "Brauzerga kirish"
---

# Brauzerga kirish + X/Twitter joylash

## Qo‘lda kirish (tavsiya etiladi)

Agar sayt kirishni talab qilsa, **host** brauzer profilida (openclaw brauzeri) **qo‘lda tizimga kiring**.

Modelga hisob ma’lumotlaringizni **bermang**. Avtomatlashtirilgan kirishlar ko‘pincha anti-bot himoyalarini ishga tushiradi va hisob bloklanishiga olib kelishi mumkin.

Asosiy brauzer hujjatlariga qaytish: [Browser](/tools/browser).

## Qaysi Chrome profili ishlatiladi?

OpenClaw **maxsus Chrome profilini** boshqaradi (`openclaw` nomli, interfeysi to‘q sariq rangda). Bu sizning kundalik brauzer profilingizdan alohida.

Unga kirishning ikki oson yo‘li:

1. **Agentdan brauzerni ochishni so‘rang** va keyin o‘zingiz tizimga kiring.
2. **CLI orqali ochish**:

```bash
openclaw browser start
openclaw browser open https://x.com
```

Agar bir nechta profilingiz bo‘lsa, `--browser-profile <name>` ni bering (standart — `openclaw`).

## X/Twitter: tavsiya etilgan ish jarayoni

- **O‘qish/qidirish/iplar:** **host** brauzerdan foydalaning (qo‘lda login).
- **Post joylash:** **host** brauzerdan foydalaning (qo‘lda login).

## Sandboxlash + host brauzerga kirish

Sandboxlangan brauzer sessiyalari bot aniqlashni **ko‘proq ehtimol** bilan ishga tushiradi. X/Twitter (va boshqa qat’iy saytlar) uchun **host** brauzerni afzal ko‘ring.

Agar agent sandboxlangan bo‘lsa, brauzer vositasi standart holatda sandboxga yo‘naltiriladi. Host boshqaruviga ruxsat berish uchun:

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

So‘ng host brauzerni nishonga oling:

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

Yoki yangilanishlar joylaydigan agent uchun sandboxlashni o‘chirib qo‘ying.

