---
read_when:
  - Första installationen från grunden
  - Vill du veta den snabbaste vägen till en fungerande chatt
summary: Installera OpenClaw och kör din första chatt på några minuter.
title: Kom igång
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Introduktion

Mål: Att från grunden, med minimal installation, få din första fungerande chatt att köra.

<Info>
Snabbaste sättet att chatta: Öppna Control UI (kanalkonfiguration krävs inte). Kör `openclaw dashboard` för att chatta i webbläsaren, eller
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gatewayホスト</Tooltip>öppna `http://127.0.0.1:18789/`.
Dokumentation: [Dashboard](/web/dashboard) och [Control UI](/web/control-ui)。
</Info>

## Förutsättningar

- Node 22 eller senare

<Tip>
Om du är osäker, kontrollera din Node-version med `node --version`.
</Tip>

## Snabbinstallation (CLI)

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
    Andra installationsmetoder och krav: [Installation](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Guiden konfigurerar autentisering, Gateway-inställningar och valfria kanaler.
    Se [Onboarding-guiden](/start/wizard) för mer information.
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Om du har installerat tjänsten bör den redan vara igång:

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
När Control UI har laddats är Gateway redo att användas.
</Check>

## Valfria kontroller och ytterligare funktioner

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Användbart för snabba tester och felsökning.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Kräver en konfigurerad kanal.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Läs mer

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Fullständig referens för CLI-guiden och avancerade alternativ.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Första uppstartsflödet för macOS-appen.
  
</Card>
</Columns>

## Status efter slutförande

- Gateway körs
- Konfigurerad autentisering
- Åtkomst till Control UI eller anslutna kanaler

## Nästa steg

- Säkerhet och godkännande för DM: [Parkoppling](/channels/pairing)
- Anslut fler kanaler: [Kanaler](/channels)
- Avancerade arbetsflöden och bygg från källkod: [Installation](/start/setup)
