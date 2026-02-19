---
read_when:
  - Når du kører onboarding-guiden eller under konfiguration
  - Når du opsætter en ny maskine
sidebarTitle: Wizard (CLI)
summary: "CLI-onboarding-guiden: interaktiv opsætning af Gateway, arbejdsområde, kanaler og Skills"
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

CLI-onboarding-guiden er den anbefalede metode til at opsætte OpenClaw på macOS, Linux og Windows (via WSL2). Ud over lokal Gateway eller forbindelse til en remote Gateway konfigurerer den også standardindstillinger for arbejdsområde, kanaler og Skills.

```bash
openclaw onboard
```

<Info>
Den hurtigste måde at starte din første chat: Åbn Control UI (kanalkonfiguration er ikke nødvendig). Kør `openclaw dashboard` for at chatte i din browser. Dokumentation: [Dashboard](/web/dashboard).
</Info>

## Hurtigstart vs. avanceret opsætning

Guiden starter med at lade dig vælge mellem **Hurtigstart** (standardindstillinger) og **Avanceret opsætning** (fuld kontrol).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Lokal Gateway på loopback
    - Eksisterende arbejdsområde eller standardarbejdsområde
    - Gateway-port `18789`
    - Gateway-godkendelsestoken genereres automatisk (oprettes også på loopback)
    - Tailscale-offentliggørelse er slået fra
    - Telegram- og WhatsApp-DM’er er som standard på tilladelsesliste (du kan blive bedt om at indtaste telefonnummer)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Viser hele prompt-flowet for tilstand, arbejdsområde, Gateway, kanaler, daemon og Skills
  
</Tab>
</Tabs>

## Detaljer om CLI-onboarding

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Fuld forklaring af lokale og remote flows, godkendelse og modelmatrix, konfigurationsoutput, wizard RPC og signal-cli-adfærd.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Opskrifter til ikke-interaktiv onboarding og automatiserede `agents add`-eksempler.
  
</Card>
</Columns>

## Ofte brugte opfølgningskommandoer

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` betyder ikke ikke-interaktiv tilstand. Brug `--non-interactive` i scripts.
</Note>

<Tip>
Anbefalet: Konfigurer en Brave Search API-nøgle, så agenter kan bruge `web_search` (`web_fetch` fungerer uden nøgle). Den nemmeste måde: kør `openclaw configure --section web` for at gemme `tools.web.search.apiKey`. Dokumentation: [Webværktøjer](/tools/web).
</Tip>

## Relateret dokumentation

- CLI-kommandoreference: [`openclaw onboard`](/cli/onboard)
- Onboarding af macOS-app: [Onboarding](/start/onboarding)
- Første opstart af agent: [Agent-bootstrap](/start/bootstrapping)
