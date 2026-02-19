---
read_when:
  - Onboarding sihirbazını çalıştırırken veya yapılandırırken
  - Yeni bir makine kurulumu sırasında
sidebarTitle: Wizard (CLI)
summary: "CLI onboarding sihirbazı: Gateway, çalışma alanı, kanallar ve Skills için etkileşimli kurulum"
title: Onboarding Sihirbazı (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding Sihirbazı (CLI)

CLI onboarding sihirbazı, macOS, Linux ve Windows (WSL2 üzerinden) üzerinde OpenClaw kurulumunda önerilen yoldur. Yerel veya uzak Gateway bağlantısının yanı sıra, çalışma alanı varsayılan ayarlarını, kanalları ve Skills yapılandırmasını yapar.

```bash
openclaw onboard
```

<Info>
İlk sohbeti en hızlı şekilde başlatmanın yolu: Control UI’yi açın (kanal yapılandırması gerekmez). `openclaw dashboard` komutunu çalıştırarak tarayıcıda sohbet edebilirsiniz. Dokümantasyon: [Dashboard](/web/dashboard).
</Info>

## Hızlı Başlangıç vs Ayrıntılı Kurulum

Sihirbaz, **Hızlı Başlangıç** (varsayılan ayarlar) veya **Ayrıntılı Kurulum** (tam kontrol) seçeneklerinden biriyle başlar.

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback üzerinde yerel Gateway
    - Mevcut bir çalışma alanı veya varsayılan çalışma alanı
    - Gateway portu `18789`
    - Gateway kimlik doğrulama belirteci otomatik oluşturulur (loopback üzerinde de oluşturulur)
    - Tailscale yayını kapalı
    - Telegram ve WhatsApp DM’leri varsayılan olarak izin listesine alınır (telefon numarası girmeniz istenebilir)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Mod, çalışma alanı, Gateway, kanallar, daemon ve Skills için tam istem akışını gösterir
  
</Tab>
</Tabs>

## CLI Onboarding Ayrıntıları

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Yerel ve uzak akışların tam açıklaması, kimlik doğrulama ve model matrisi, yapılandırma çıktısı, sihirbaz RPC’si ve signal-cli davranışı.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Etkileşimsiz onboarding reçeteleri ve otomatikleştirilmiş `agents add` örnekleri.
  
</Card>
</Columns>

## Sık kullanılan takip komutları

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` etkileşimsiz mod anlamına gelmez. Scriptlerde `--non-interactive` kullanın.
</Note>

<Tip>
Önerilir: Ajanın `web_search` kullanabilmesi için Brave Search API anahtarını yapılandırın (`web_fetch` anahtar olmadan çalışır). En kolay yol: `openclaw configure --section web` komutunu çalıştırarak `tools.web.search.apiKey` değerini kaydetmek. Dokümantasyon: [Web araçları](/tools/web).
</Tip>

## İlgili Dokümantasyon

- CLI komut referansı: [`openclaw onboard`](/cli/onboard)
- macOS uygulaması onboarding: [Onboarding](/start/onboarding)
- Ajan ilk başlatma adımları: [Ajan Bootstrapping](/start/bootstrapping)
