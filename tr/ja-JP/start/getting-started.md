---
read_when:
  - Sıfırdan ilk kurulum
  - Çalışan bir sohbete giden en kısa yolu öğrenin
summary: OpenClaw’u yükleyin ve birkaç dakika içinde ilk sohbetinizi başlatın。
title: Başlarken
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Giriş

Amaç: Sıfırdan, minimum kurulumla ilk çalışan sohbeti gerçekleştirmek.

<Info>
En hızlı sohbet yöntemi: Control UI'yi açın (kanal ayarı gerekmez). `openclaw dashboard` çalıştırarak tarayıcıda sohbet edin veya,<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway ana makinesi</Tooltip>üzerinde `http://127.0.0.1:18789/` adresini açın.
Dokümantasyon: [Dashboard](/web/dashboard) ve [Control UI](/web/control-ui)。
</Info>

## Ön koşullar

- Node 22 veya üzeri

<Tip>
Emin değilseniz, `node --version` komutuyla Node sürümünü kontrol edin.
</Tip>

## Hızlı kurulum (CLI)

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
    Diğer kurulum yöntemleri ve gereksinimler için: [Kurulum](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Sihirbaz kimlik doğrulamayı, Gateway yapılandırmasını ve isteğe bağlı kanalları yapılandırır.
    Ayrıntılar için [Onboarding Sihirbazı](/start/wizard) sayfasına bakın。
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Hizmeti kurduysanız, zaten çalışıyor olmalıdır:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Control UI yüklenirse, Gateway kullanıma hazırdır。
</Check>

## İsteğe bağlı doğrulama ve ek özellikler

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Hızlı testler ve sorun giderme için kullanışlıdır。

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Yapılandırılmış bir kanal gerektirir。

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Daha fazla bilgi

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Tam CLI sihirbaz referansı ve gelişmiş seçenekler。
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS uygulamasının ilk çalıştırma akışı。
  
</Card>
</Columns>

## Tamamlandıktan sonraki durum

- Çalışan Gateway
- Yapılandırılmış kimlik doğrulama
- Control UI erişimi veya bağlı bir kanal

## Sonraki adımlar

- DM güvenliği ve onay: [Eşleştirme](/channels/pairing)
- Daha fazla kanal bağlayın: [Kanallar](/channels)
- Gelişmiş iş akışları ve kaynaktan derleme: [Kurulum](/start/setup)
