---
read_when:
  - Onboarding Wizard’ni ishga tushirish yoki sozlash vaqtida
  - Yangi mashinani sozlash vaqtida
sidebarTitle: Wizard (CLI)
summary: "CLI Onboarding Wizard: Gateway, workspace, kanallar va Skills uchun interaktiv sozlash"
title: Onboarding Wizard (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding Wizard (CLI)

CLI Onboarding Wizard macOS, Linux va Windows (WSL2 orqali) tizimlarida OpenClaw’ni sozlash uchun tavsiya etilgan yo‘ldir. U lokal Gateway yoki masofaviy Gateway ulanishidan tashqari, workspace’ning standart sozlamalari, kanallar va Skills’ni ham sozlaydi.

```bash
openclaw onboard
```

<Info>
Birinchi chatni eng tez boshlash usuli: Control UI ni oching (kanal sozlamalari talab qilinmaydi). `openclaw dashboard` ni ishga tushiring va brauzer orqali chat qiling. Hujjatlar: [Dashboard](/web/dashboard).
</Info>

## Tezkor ishga tushirish vs Batafsil sozlash

Ustoz (wizard) boshlash uchun **Tezkor ishga tushirish** (standart sozlamalar) yoki **Batafsil sozlash** (to‘liq nazorat) variantlaridan birini tanlashni taklif qiladi.

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">    - loopback dagi lokal Gateway
    - Mavjud workspace yoki standart workspace
    - Gateway porti `18789`
    - Gateway autentifikatsiya tokeni avtomatik yaratiladi (loopback da ham yaratiladi)
    - Tailscale orqali ommaga ochish o‘chiq
    - Telegram va WhatsApp DM lar standart bo‘yicha ruxsatlar ro‘yxati asosida (telefon raqamini kiritish so‘ralishi mumkin)
</Tab>
  <Tab title="詳細設定（完全な制御）">    - Rejim, workspace, Gateway, kanallar, daemon va Skills uchun to‘liq prompt oqimini ko‘rsatadi
</Tab>
</Tabs>

## CLI onboarding tafsilotlari

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">    Lokal va masofaviy jarayonlarning to‘liq tavsifi, autentifikatsiya va model matritsasi, konfiguratsiya chiqishi, wizard RPC va signal-cli ishlash prinsipi.
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">    Interaktiv bo‘lmagan onboarding retseptlari va avtomatlashtirilgan `agents add` misollari.
</Card>
</Columns>

## Ko‘p ishlatiladigan keyingi buyruqlar

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` interaktiv bo‘lmagan rejimni anglatmaydi. Skriptlarda `--non-interactive` dan foydalaning.
</Note>

<Tip>
Tavsiya: agent `web_search` dan foydalana olishi uchun Brave Search API kalitini sozlang (`web_fetch` kalitsiz ham ishlaydi). Eng oson usul: `openclaw configure --section web` ni ishga tushiring — bu `tools.web.search.apiKey` ni saqlaydi. Hujjatlar: [Webツール](/tools/web).
</Tip>

## Tegishli hujjatlar

- CLI buyruqlar ma’lumotnomasi: [`openclaw onboard`](/cli/onboard)
- macOS ilovasida onboarding: [Onboarding](/start/onboarding)
- Agentni birinchi ishga tushirish tartibi: [Agent Bootstrap](/start/bootstrapping)
