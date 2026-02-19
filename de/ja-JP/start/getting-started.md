---
read_when:
  - Ersteinrichtung von Grund auf
  - Den schnellsten Weg zu einem funktionierenden Chat kennenlernen
summary: Installieren Sie OpenClaw und führen Sie in wenigen Minuten Ihren ersten Chat aus。
title: Einführung
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Einführung

Ziel: Mit einer minimalen Einrichtung von Grund auf den ersten funktionierenden Chat erreichen。

<Info>
最速のチャット方法：Control UIを開く（チャンネル設定は不要）。`openclaw dashboard`を実行してブラウザでチャットするか、<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway-Host</Tooltip>und öffnen Sie `http://127.0.0.1:18789/`.
Dokumentation: [Dashboard](/web/dashboard) und [Control UI](/web/control-ui)。
</Info>

## Voraussetzungen

- Node 22 oder höher

<Tip>
Wenn Sie unsicher sind, überprüfen Sie die Node-Version mit `node --version`。
</Tip>

## Schnelleinrichtung (CLI)

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
    Weitere Installationsmethoden und Anforderungen: [Installieren](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Der Assistent konfiguriert die Authentifizierung, die Gateway-Einstellungen sowie optionale Kanäle.
    Weitere Details finden Sie im [Onboarding-Assistenten](/start/wizard)。
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Wenn Sie den Dienst installiert haben, sollte er bereits ausgeführt werden：

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
Wenn die Control UI geladen wird, ist das Gateway einsatzbereit。
</Check>

## Optionale Überprüfung und zusätzliche Funktionen

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Nützlich für Schnelltests oder zur Fehlerbehebung。

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Ein konfigurierter Kanal ist erforderlich。

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Weitere Details

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Vollständige CLI-Wizard-Referenz und erweiterte Optionen.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Erster Ausführungsablauf der macOS-App.
  
</Card>
</Columns>

## Status nach Abschluss

- Laufender Gateway
- Konfigurierte Authentifizierung
- Control UI-Zugriff oder verbundene Kanäle

## Nächste Schritte

- Sicherheit und Freigabe von DMs: [Pairing](/channels/pairing)
- Weitere Kanäle verbinden: [Kanäle](/channels)
- Erweiterte Workflows und Build aus dem Quellcode: [Setup](/start/setup)
