# OpenClaw Tahdidlar Modeli v1.0

## MITRE ATLAS Freymvorki

**Versiya:** 1.0-draft  
**Oxirgi yangilanish:** 2026-02-04  
**Metodologiya:** MITRE ATLAS + Ma’lumotlar oqimi diagrammalari  
**Freymvork:** [MITRE ATLAS](https://atlas.mitre.org/) (AI tizimlari uchun adversarial tahdidlar landshafti)

### Freymvork manbasi

Ushbu tahdidlar modeli [MITRE ATLAS](https://atlas.mitre.org/) asosida ishlab chiqilgan bo‘lib, u AI/ML tizimlariga qarshi adversarial tahdidlarni hujjatlashtirish uchun sanoat standarti hisoblanadi. ATLAS [MITRE](https://www.mitre.org/) tomonidan AI xavfsizlik hamjamiyati bilan hamkorlikda yuritiladi.

**ATLAS’ning asosiy resurslari:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [ATLAS’ga hissa qo‘shish](https://atlas.mitre.org/resources/contribute)

### Ushbu tahdidlar modeliga hissa qo‘shish

Bu OpenClaw hamjamiyati tomonidan yuritiladigan doimiy yangilanib boruvchi hujjatdir. Hissa qo‘shish bo‘yicha ko‘rsatmalar uchun [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) fayliga qarang:

- Yangi tahdidlar haqida xabar berish
- Mavjud tahdidlarni yangilash
- Hujum zanjirlarini taklif qilish
- Himoya choralarini taklif qilish

---

## 1. Kirish

### 1.1 Maqsad

Ushbu tahdidlar modeli OpenClaw AI agent platformasi va ClawHub skill bozori uchun adversarial tahdidlarni, aynan AI/ML tizimlari uchun mo‘ljallangan MITRE ATLAS freymvorki asosida hujjatlashtiradi.

### 1.2 Qamrov

| Komponent              | Qamrovda | Izoh                                              |
| ---------------------- | -------- | ------------------------------------------------- |
| OpenClaw Agent Runtime | Ha       | Asosiy agent bajarilishi, tool chaqiruvlari, sessiyalar |
| Gateway                | Ha       | Autentifikatsiya, marshrutlash, kanal integratsiyasi |
| Kanal integratsiyalari | Ha       | WhatsApp, Telegram, Discord, Signal, Slack va boshqalar |
| ClawHub Marketplace    | Ha       | Skill nashri, moderatsiya, tarqatish             |
| MCP Serverlar          | Ha       | Tashqi tool provayderlari                         |
| Foydalanuvchi qurilmalari | Qisman  | Mobil ilovalar, desktop mijozlar                 |

### 1.3 Qamrovdan tashqari

Ushbu tahdidlar modeli uchun hech narsa aniq ravishda qamrovdan tashqari deb belgilanmagan.

---

## 2. Tizim arxitekturasi

### 2.1 Ishonch chegaralari

```
┌─────────────────────────────────────────────────────────────────┐
│                    ISHONCHSIZ HUDUD                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│          ISHONCH CHEGARASI 1: Kanalga kirish                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Qurilma juftlash (30 soniyalik imtiyozli muddat)       │   │
│  │  • AllowFrom / AllowList tekshiruvi                      │   │
│  │  • Token/Password/Tailscale autentifikatsiyasi           │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          ISHONCH CHEGARASI 2: Sessiya izolyatsiyasi              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT SESSIYALARI                       │   │
│  │  • Sessiya kaliti = agent:channel:peer                   │   │
│  │  • Har bir agent uchun tool siyosatlari                  │   │
│  │  • Transkriptlarni loglash                                │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          ISHONCH CHEGARASI 3: Tool bajarilishi                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  BAJARISH SANDBOXI                       │   │
│  │  • Docker sandbox YOKI Host (exec-approvals)             │   │
│  │  • Node masofaviy bajarilishi                            │   │
│  │  • SSRF himoyasi (DNS pinning + IP bloklash)             │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          ISHONCH CHEGARASI 4: Tashqi kontent                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           YUKLAB OLINGAN URL / EMAIL / WEBHOOK          │   │
│  │  • Tashqi kontentni o‘rash (XML teglar)                  │   │
│  │  • Xavfsizlik ogohlantirishini qo‘shish                  │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          ISHONCH CHEGARASI 5: Ta’minot zanjiri                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Skill nashri (semver, SKILL.md majburiy)              │   │
│  │  • Andozaga asoslangan moderatsiya flaglari              │   │
│  │  • VirusTotal skanerlash (tez orada)                     │   │
│  │  • GitHub akkaunt yoshini tekshirish                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Ma’lumotlar oqimlari

| Oqim | Manba   | Manzil      | Ma’lumot              | Himoya                 |
| ---- | ------- | ----------- | --------------------- | ---------------------- |
| F1   | Kanal   | Gateway     | Foydalanuvchi xabarlari | TLS, AllowFrom         |
| F2   | Gateway | Agent       | Marshrutlangan xabarlar | Sessiya izolyatsiyasi  |
| F3   | Agent   | Tools       | Tool chaqiruvlari     | Siyosatni majburiy qo‘llash |
| F4   | Agent   | Tashqi      | web_fetch so‘rovlari  | SSRF bloklash          |
| F5   | ClawHub | Agent       | Skill kodi            | Moderatsiya, skanerlash |
| F6   | Agent   | Kanal       | Javoblar              | Chiqish filtratsiyasi  |

---

## 3. ATLAS taktikasi bo‘yicha tahdidlar tahlili

### 3.1 Razvedka (AML.TA0002)

#### T-RECON-001: Agent endpointlarini aniqlash

| Atribut                | Qiymat                                                               |
| ---------------------- | -------------------------------------------------------------------- |
| **ATLAS ID**           | AML.T0006 - Faol skanerlash                                          |
| **Tavsif**             | Hujumchi ochiq OpenClaw gateway endpointlarini skaner qiladi        |
| **Hujum vektori**      | Tarmoq skanerlash, shodan so‘rovlari, DNS enumeratsiyasi            |
| **Ta’sirlangan komponentlar** | Gateway, ochiq API endpointlari                              |
| **Joriy himoya choralar** | Tailscale autentifikatsiyasi opsiyasi, sukut bo‘yicha loopback’ga bog‘lash |
| **Qolgan xavf**        | O‘rta - Ommaviy gateway’lar aniqlanishi mumkin                      |
| **Tavsiyalar**         | Xavfsiz joylashtirishni hujjatlashtirish, aniqlash endpointlariga rate limiting qo‘shish |

#### T-RECON-002: Kanal integratsiyasini tekshirish

| Atribut                | Qiymat                                                              |
| ---------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**           | AML.T0006 - Faol skanerlash                                         |
| **Tavsif**             | Hujumchi AI boshqaruvidagi akkauntlarni aniqlash uchun xabar kanallarini tekshiradi |
| **Hujum vektori**      | Test xabarlar yuborish, javob andozalarini kuzatish                |
| **Ta’sirlangan komponentlar** | Barcha kanal integratsiyalari                              |
| **Joriy himoya choralar** | Maxsus choralar yo‘q                                            |
| **Qolgan xavf**        | Past - Faqat aniqlashning o‘zi katta qiymat bermaydi               |
| **Tavsiyalar**         | Javob vaqtini tasodifiylashtirishni ko‘rib chiqish                 |

---
