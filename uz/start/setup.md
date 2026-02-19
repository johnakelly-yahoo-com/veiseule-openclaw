---
summary: "OpenClaw uchun ilg‘or o‘rnatish va ishlab chiqish ish jarayonlari"
read_when:
  - Yangi mashina sozlayotganda
  - Shaxsiy sozlamalaringizni buzmasdan “eng so‘nggi va eng zo‘r” versiyani xohlasangiz
title: "O‘rnatish"
---

# O‘rnatish

<Note>
If you are setting up for the first time, start with [Getting Started](/start/getting-started).
For wizard details, see [Onboarding Wizard](/start/wizard).
</Note>

Oxirgi yangilanish: 2026-01-01

## Qisqacha

- **Moslashtirish repodan tashqarida saqlanadi:** `~/.openclaw/workspace` (workspace) + `~/.openclaw/openclaw.json` (config).
- **Barqaror ish jarayoni:** macOS ilovasini o‘rnating; u o‘rnatilgan Gateway’ni o‘zi ishga tushiradi.
- **Eng so‘nggi (bleeding edge) ish jarayoni:** Gateway’ni `pnpm gateway:watch` orqali o‘zingiz ishga tushiring, so‘ng macOS ilovasi Local rejimida ulanadi.

## Talablar (manbadan yig‘ish uchun)

- Node `>=22`
- `pnpm`
- Docker (ixtiyoriy; faqat container asosidagi o‘rnatish/e2e uchun — qarang [Docker](/install/docker))

## Moslashtirish strategiyasi (yangilanishlar zarar qilmasligi uchun)

Agar “100% o‘zimga mos” sozlashni _va_ oson yangilanishlarni xohlasangiz, sozlamalaringizni quyida saqlang:

- **Config:** `~/.openclaw/openclaw.json` (JSON/JSON5-ga o‘xshash)
- **Workspace:** `~/.openclaw/workspace` (skills, promptlar, xotiralar; uni shaxsiy git repo qiling)

Bir marta ishga tushiring:

```bash
openclaw setup
```

Ushbu repodan foydalanayotganda, lokal CLI kirish nuqtasidan foydalaning:

```bash
openclaw setup
```

Agar global o‘rnatish hali bo‘lmasa, `pnpm openclaw setup` orqali ishga tushiring.

## Gateway’ni ushbu repodan ishga tushirish

`pnpm build` dan so‘ng, paketlangan CLI’ni to‘g‘ridan-to‘g‘ri ishga tushirishingiz mumkin:

```bash
node openclaw.mjs gateway --port 18789 --verbose
```

## Barqaror ish jarayoni (macOS ilovasi birinchi)

1. **OpenClaw.app** ni o‘rnating va ishga tushiring (menyu paneli).
2. Onboarding/ruxsatlar ro‘yxatini yakunlang (TCC so‘rovlari).
3. Gateway **Local** rejimida va ishga tushganligiga ishonch hosil qiling (ilova boshqaradi).
4. Kanallarni ulang (masalan: WhatsApp):

```bash
openclaw channels login
```

5. Tekshiruv:

```bash
openclaw health
```

Agar onboarding sizning build versiyangizda mavjud bo‘lmasa:

- `openclaw setup`, so‘ng `openclaw channels login` ni ishga tushiring, keyin Gateway’ni qo‘lda ishga tushiring (`openclaw gateway`).

## Eng so‘nggi (bleeding edge) ish jarayoni (Gateway terminalda)

Maqsad: TypeScript Gateway ustida ishlash, hot reload olish va macOS ilovasi UI’sini ulangan holda saqlash.

### 0. (Ixtiyoriy) macOS ilovasini ham manbadan ishga tushirish

Agar macOS ilovasini ham eng so‘nggi versiyada ishlatmoqchi bo‘lsangiz:

```bash
./scripts/restart-mac.sh
```

### 1. Dev Gateway’ni ishga tushiring

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` gateway’ni watch rejimida ishga tushiradi va TypeScript o‘zgarishlarida qayta yuklaydi.

### 2. macOS ilovasini ishlayotgan Gateway’ga yo‘naltiring

**OpenClaw.app** ichida:

- Ulanish rejimi: **Mahalliy**
  Ilova sozlangan port orqali ishlayotgan gateway’ga ulanadi.

### 3. Tekshirish

- Ilova ichida Gateway holati **“Using existing gateway …”** deb ko‘rsatilishi kerak.
- Yoki CLI orqali:

```bash
openclaw health
```

### Keng tarqalgan xatolar

- **Noto‘g‘ri port:** Gateway WS odatda `ws://127.0.0.1:18789`; ilova va CLI bir xil portdan foydalanayotganini tekshiring.
- **Holat (state) qayerda saqlanadi:**
  - Credential’lar: `~/.openclaw/credentials/`
  - Sessiyalar: `~/.openclaw/agents/<agentId>/sessions/`
  - Loglar: `/tmp/openclaw/`

## Credential saqlash xaritasi

Autentifikatsiya muammolarini tekshirish yoki nimani zaxiralash kerakligini aniqlashda foydalaning:

- **WhatsApp**: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
- **Telegram bot tokeni**: config/env yoki `channels.telegram.tokenFile`
- **Discord bot tokeni**: config/env (token fayli hali qo‘llab-quvvatlanmaydi)
- **Slack tokenlari**: config/env (`channels.slack.*`)
- **Pairing allowlist’lar**: `~/.openclaw/credentials/<channel>-allowFrom.json`
- **Model auth profillari**: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`
- **Eski OAuth importi**: `~/.openclaw/credentials/oauth.json`
  Batafsil: [Security](/gateway/security#credential-storage-map).

## Yangilash (sozlamalaringizni buzmasdan)

- `~/.openclaw/workspace` va `~/.openclaw/` ni “o‘zingizga tegishli ma’lumotlar” sifatida saqlang; shaxsiy prompt/config’ni `openclaw` reposiga joylamang.
- Manbani yangilash: `git pull` + `pnpm install` (agar lockfile o‘zgargan bo‘lsa) + `pnpm gateway:watch` dan foydalanishda davom eting.

## Linux (systemd user xizmati)

Linux o‘rnatmalari systemd **user** xizmatidan foydalanadi. Sukut bo‘yicha systemd foydalanuvchi xizmatlarini logout/idle’da to‘xtatadi, bu esa Gateway’ni o‘chiradi. Onboarding siz uchun lingering’ni yoqishga harakat qiladi (sudo so‘rashi mumkin). Agar hali ham o‘chiq bo‘lsa, ishga tushiring:

```bash
sudo loginctl enable-linger $USER
```

Doimiy ishlash yoki ko‘p foydalanuvchili serverlar uchun **system** xizmatini **user** xizmati o‘rniga ko‘rib chiqing (lingering kerak bo‘lmaydi). systemd bo‘yicha eslatmalar uchun [Gateway runbook](/gateway) ga qarang.

## Bog‘liq hujjatlar

- [Gateway runbook](/gateway) (flaglar, nazorat, portlar)
- [Gateway configuration](/gateway/configuration) (config sxemasi + misollar)
- [Discord](/channels/discord) va [Telegram](/channels/telegram) (reply teglari + replyToMode sozlamalari)
- [OpenClaw assistantini sozlash](/start/openclaw)
- [macOS app](/platforms/macos) (gateway hayotiy sikli)

