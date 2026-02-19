---
read_when:
  - Første opsætning fra bunden
  - Vil du kende den hurtigste vej til en fungerende chat?
summary: Installer OpenClaw og kør din første chat på få minutter.
title: Introduktion
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Introduktion

Mål: At opnå din første fungerende chat fra bunden med minimal opsætning.

<Info>
Hurtigste måde at chatte på: Åbn Control UI (ingen kanalopsætning nødvendig). Kør `openclaw dashboard` for at chatte i browseren, eller
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway-vært</Tooltip>ved at åbne `http://127.0.0.1:18789/`.
Dokumentation: [Dashboard](/web/dashboard) og [Control UI](/web/control-ui).
</Info>

## Forudsætninger

- Node 22 eller nyere

<Tip>
Hvis du er i tvivl, kan du tjekke din Node-version med `node --version`.
</Tip>

## Hurtig opsætning (CLI)

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
    Andre installationsmetoder og krav: [Installation](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Guiden konfigurerer godkendelse, Gateway-indstillinger og valgfrie kanaler.
    Se [Onboarding-guiden](/start/wizard) for flere detaljer.
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Hvis du har installeret tjenesten, bør den allerede køre:


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
Når Control UI indlæses, er Gateway klar til brug.
</Check>

## Valgfri verifikation og ekstra funktioner

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Praktisk til hurtige tests og fejlfinding.


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Kræver en konfigureret kanal.


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Mere information

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Fuld CLI-guide og avancerede indstillinger.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Førstegangsforløb for macOS-appen.
  
</Card>
</Columns>

## Status efter fuldførelse

- Gateway kører
- Konfigureret godkendelse
- Adgang til Control UI eller tilsluttede kanaler

## Næste trin

- Sikkerhed og godkendelse af DM’er: [Parring](/channels/pairing)
- Tilslut flere kanaler: [Kanaler](/channels)
- Avancerede workflows og byg fra kilde: [Opsætning](/start/setup)
