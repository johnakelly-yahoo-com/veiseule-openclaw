---
read_when:
  - Eerste installatie vanaf nul
  - Wil je de snelste route naar een werkende chat weten
summary: Installeer OpenClaw en start je eerste chat binnen enkele minuten.
title: Aan de slag
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Inleiding

Doel: vanaf nul je eerste werkende chat realiseren met een minimale setup.

<Info>
Snelste manier om te chatten: open de Control UI (geen kanaalconfiguratie nodig). Voer `openclaw dashboard` uit om in je browser te chatten, of<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway-host</Tooltip>door `http://127.0.0.1:18789/` te openen.
Documentatie: [Dashboard](/web/dashboard) en [Control UI](/web/control-ui).
</Info>

## Vereisten

- Node 22 of hoger

<Tip>
Als je het niet zeker weet, controleer je Node-versie met `node --version`.
</Tip>

## Snelle installatie (CLI)

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
    Andere installatiemethoden en vereisten: [Installatie](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    De wizard configureert authenticatie, de Gateway-instellingen en optionele kanalen.
    Zie [Onboarding-wizard](/start/wizard) voor meer informatie.
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Als je de service hebt geïnstalleerd, zou deze al moeten draaien:

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
Als de Control UI wordt geladen, is de Gateway klaar voor gebruik.
</Check>

## Optionele controles en extra functies

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Handig voor snelle tests en probleemoplossing.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Vereist een geconfigureerd kanaal.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Meer informatie

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Volledige CLI-wizardreferentie en geavanceerde opties.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Eerste opstartproces van de macOS-app.
  
</Card>
</Columns>

## Status na voltooiing

- Draaiende Gateway
- Geconfigureerde authenticatie
- Toegang tot Control UI of verbonden kanalen

## Volgende stappen

- Veiligheid en goedkeuring van DM’s: [Koppelen](/channels/pairing)
- Verbind meer kanalen: [Kanalen](/channels)
- Geavanceerde workflows en bouwen vanuit broncode: [Setup](/start/setup)
