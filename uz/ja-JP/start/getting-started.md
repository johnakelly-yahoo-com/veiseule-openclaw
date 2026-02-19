---
read_when:
  - Noldan birinchi sozlash
  - Ishlaydigan chatga eng qisqa yo‘lni bilmoqchimisiz
summary: OpenClaw’ni o‘rnating va bir necha daqiqada birinchi chatni ishga tushiring.
title: Boshlash
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Boshlash

Maqsad: noldan minimal sozlama bilan birinchi ishlaydigan chatni ishga tushirish.

<Info>
Eng tez chat usuli: Control UI’ni oching (kanal sozlamasi talab qilinmaydi). `openclaw dashboard` ni ishga tushirib brauzerda chat qiling yoki,
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway xosti</Tooltip>da `http://127.0.0.1:18789/` ni oching.
Hujjatlar: [Dashboard](/web/dashboard) va [Control UI](/web/control-ui).
</Info>

## Talablar

- Node 22 yoki undan yuqori

<Tip>
Noma’lum bo‘lsa, `node --version` bilan Node versiyasini tekshiring.
</Tip>

## Tezkor sozlash (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    Boshqa o‘rnatish usullari va talablar: [O‘rnatish](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">```bash
    openclaw onboard --install-daemon
    ```

    ```
    Sehrgar autentifikatsiya, Gateway sozlamalari va ixtiyoriy kanallarni sozlaydi.
    Batafsil ma’lumot uchun [Onboarding Wizard](/start/wizard) sahifasiga qarang.
    ```

  
</Step>
  <Step title="Gatewayを確認">Xizmatni o‘rnatgan bo‘lsangiz, u allaqachon ishga tushgan bo‘lishi kerak:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">```bash
    openclaw dashboard
    ```
</Step>
</Steps>

<Check>
Agar Control UI yuklansa, Gateway foydalanishga tayyor.
</Check>

## Ixtiyoriy tekshiruvlar va qo‘shimcha funksiyalar

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">Tezkor test va nosozliklarni bartaraf etish uchun qulay.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">Oldindan sozlangan kanal talab qilinadi.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Batafsil ma’lumot

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">To‘liq CLI wizard qo‘llanmasi va kengaytirilgan variantlar.
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">macOS ilovasining birinchi ishga tushirish jarayoni.
</Card>
</Columns>

## Yakunlangandan keyingi holat

- Ishlayotgan Gateway
- Sozlangan autentifikatsiya
- Control UI kirishi yoki ulangan kanallar

## Keyingi qadamlar

- DM xavfsizligi va tasdiqlash: [Pairing](/channels/pairing)
- Qo‘shimcha kanallarni ulash: [Channels](/channels)
- Kengaytirilgan ish jarayonlari va manbadan yig‘ish: [Setup](/start/setup)
