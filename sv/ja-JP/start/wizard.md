---
read_when:
  - När du kör onboarding-guiden eller konfigurerar
  - När du konfigurerar en ny maskin
sidebarTitle: Wizard (CLI)
summary: "CLI-onboarding-guiden: Interaktiv konfiguration av Gateway, arbetsytor, kanaler och Skills"
title: Onboarding-guide (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding-guide (CLI)

CLI-onboarding-guiden är den rekommenderade vägen för att konfigurera OpenClaw på macOS, Linux och Windows (via WSL2). Den konfigurerar en lokal Gateway eller ansluter till en fjärr-Gateway, samt ställer in standardinställningar för arbetsytan, kanaler och Skills.

```bash
openclaw onboard
```

<Info>
Snabbaste sättet att starta din första chatt: öppna Control UI (ingen kanalkonfiguration krävs). Kör `openclaw dashboard` för att chatta i webbläsaren. Dokumentation: [Dashboard](/web/dashboard).
</Info>

## Snabbstart vs avancerad konfiguration

Guiden börjar med att låta dig välja mellan **Snabbstart** (standardinställningar) eller **Avancerad konfiguration** (full kontroll).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Lokal Gateway på loopback
    - Befintlig arbetsyta eller standardarbetsyta
    - Gateway-port `18789`
    - Gateway-autentiseringstoken genereras automatiskt (skapas även på loopback)
    - Tailscale-publicering är avstängd
    - Telegram- och WhatsApp-DM är tillåtna som standard (du kan bli ombedd att ange telefonnummer)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Visar hela promptflödet för läge, arbetsyta, Gateway, kanaler, daemoner och Skills
  
</Tab>
</Tabs>

## Detaljer om CLI-onboarding

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Fullständig genomgång av lokala och fjärrflöden, autentisering och modellmatris, konfigurationsutdata, wizard-RPC och signal-cli-beteende.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Recept för icke-interaktiv onboarding och automatiserade exempel med `agents add`.
  
</Card>
</Columns>

## Vanliga uppföljningskommandon

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` betyder inte icke-interaktivt läge. Använd `--non-interactive` i skript.
</Note>

<Tip>
Rekommenderat: konfigurera en Brave Search API-nyckel så att agenten kan använda `web_search` (`web_fetch` fungerar utan nyckel). Enklaste sättet: kör `openclaw configure --section web` för att spara `tools.web.search.apiKey`. Dokumentation: [Webbverktyg](/tools/web).
</Tip>

## Relaterad dokumentation

- CLI-kommandoreferens: [`openclaw onboard`](/cli/onboard)
- Onboarding för macOS-appen: [Onboarding](/start/onboarding)
- Steg för första uppstart av agent: [Agent-bootstrap](/start/bootstrapping)
