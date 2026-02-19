---
read_when:
  - Beim Ausführen oder Konfigurieren des Onboarding-Wizards
  - Beim Einrichten eines neuen Rechners
sidebarTitle: Wizard (CLI)
summary: "CLI-Onboarding-Wizard: Interaktive Einrichtung von Gateway, Workspace, Kanälen und Skills"
title: Onboarding-Wizard (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding-Wizard (CLI)

Der CLI-Onboarding-Wizard ist der empfohlene Weg, um OpenClaw unter macOS, Linux und Windows (über WSL2) einzurichten. Neben der Verbindung zu einem lokalen oder entfernten Gateway konfiguriert er die Standard-Workspace-Einstellungen, Kanäle und Skills.

```bash
openclaw onboard
```

<Info>
Der schnellste Weg, um den ersten Chat zu starten: Öffnen Sie die Control UI (keine Kanalkonfiguration erforderlich). Führen Sie `openclaw dashboard` aus, um im Browser zu chatten. Dokumentation: [Dashboard](/web/dashboard).
</Info>

## Schnellstart vs. Erweiterte Einrichtung

Der Wizard startet mit der Auswahl zwischen **Schnellstart** (Standardeinstellungen) und **Erweiterte Einrichtung** (volle Kontrolle).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Lokaler Gateway auf loopback
    - Bestehender Workspace oder Standard-Workspace
    - Gateway-Port `18789`
    - Gateway-Authentifizierungstoken wird automatisch generiert (auch auf loopback)
    - Tailscale-Veröffentlichung deaktiviert
    - Telegram- und WhatsApp-DMs standardmäßig auf Allowlist (möglicherweise wird die Eingabe einer Telefonnummer angefordert)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Vollständiger Prompt-Ablauf für Modus, Workspace, Gateway, Kanäle, Daemons und Skills anzeigen
  
</Tab>
</Tabs>

## Details zum CLI-Onboarding

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Vollständige Beschreibung lokaler und Remote-Flows, Authentifizierungs- und Modellmatrix, Konfigurationsausgabe, Wizard-RPC und Verhalten von signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Rezepte für nicht-interaktives Onboarding und automatisierte `agents add`-Beispiele.
  
</Card>
</Columns>

## Häufig verwendete Folgekommandos

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` bedeutet nicht nicht-interaktiver Modus. Verwenden Sie in Skripten `--non-interactive`.
</Note>

<Tip>
Empfohlen: Richten Sie einen Brave Search API-Schlüssel ein, damit Agenten `web_search` verwenden können (`web_fetch` funktioniert ohne Schlüssel). Am einfachsten: Führen Sie `openclaw configure --section web` aus, um `tools.web.search.apiKey` zu speichern. Dokumentation: [Web-Tools](/tools/web).
</Tip>

## Verwandte Dokumentation

- CLI-Befehlsreferenz: [`openclaw onboard`](/cli/onboard)
- Onboarding der macOS-App: [Onboarding](/start/onboarding)
- Anleitung für den ersten Start des Agenten: [Agent-Bootstrap](/start/bootstrapping)
