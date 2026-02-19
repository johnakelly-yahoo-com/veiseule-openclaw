---
read_when:
  - Pierwsza konfiguracja od zera
  - Chcę poznać najszybszą drogę do działającego czatu
summary: Zainstaluj OpenClaw i uruchom swój pierwszy czat w kilka minut.
title: Wprowadzenie
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Wprowadzenie

Cel: od zera, przy minimalnej konfiguracji, uruchomić pierwszy działający czat.

<Info>
Najszybszy sposób na czat: otwórz Control UI (konfiguracja kanału nie jest wymagana). Uruchom `openclaw dashboard`, aby czatować w przeglądarce, lub
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Host Gateway</Tooltip> pod adresem `http://127.0.0.1:18789/`.
Dokumentacja: [Dashboard](/web/dashboard) i [Control UI](/web/control-ui).
</Info>

## Wymagania wstępne

- Node 22 lub nowszy

<Tip>
Jeśli nie masz pewności, sprawdź wersję Node za pomocą `node --version`.
</Tip>

## Szybka konfiguracja (CLI)

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
    Inne metody instalacji i wymagania: [Instalacja](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Kreator skonfiguruje uwierzytelnianie, ustawienia Gateway oraz opcjonalne kanały.
    Szczegóły znajdziesz w [Kreatorze onboardingu](/start/wizard).
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Jeśli zainstalowałeś usługę, powinna już działać:

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
Jeśli Control UI się załaduje, Gateway jest gotowy do użycia.
</Check>

## Sprawdzanie opcji i funkcji dodatkowych

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Przydatne do szybkich testów i rozwiązywania problemów.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Wymagany jest skonfigurowany kanał.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Dowiedz się więcej

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Pełna referencja kreatora CLI i zaawansowane opcje.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Przebieg pierwszego uruchomienia aplikacji macOS.
  
</Card>
</Columns>

## Stan po zakończeniu

- Działający Gateway
- Skonfigurowane uwierzytelnianie
- Dostęp do Control UI lub podłączony kanał

## Następne kroki

- Bezpieczeństwo i zatwierdzanie DM: [Parowanie](/channels/pairing)
- Podłącz więcej kanałów: [Kanały](/channels)
- Zaawansowane przepływy pracy i budowanie ze źródła: [Konfiguracja](/start/setup)
