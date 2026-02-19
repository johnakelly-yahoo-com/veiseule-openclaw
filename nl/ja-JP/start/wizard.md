---
read_when:
  - Bij het uitvoeren van de onboardingwizard of tijdens de configuratie
  - Bij het instellen van een nieuwe machine
sidebarTitle: Wizard (CLI)
summary: "CLI-onboardingwizard: interactieve setup van Gateway, werkruimte, kanalen en Skills"
title: Onboardingwizard (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboardingwizard (CLI)

De CLI-onboardingwizard is het aanbevolen traject voor het instellen van OpenClaw op macOS, Linux en Windows (via WSL2). Naast lokale Gateway- of externe Gateway-verbindingen configureert deze de standaardinstellingen van de werkruimte, kanalen en Skills.

```bash
openclaw onboard
```

<Info>
Snelste manier om je eerste chat te starten: open de Control UI (geen kanaalconfiguratie vereist). Voer `openclaw dashboard` uit om in je browser te chatten. Documentatie: [Dashboard](/web/dashboard).
</Info>

## Snelle start vs geavanceerde configuratie

De wizard begint met de keuze tussen **Snelle start** (standaardinstellingen) of **Geavanceerde configuratie** (volledige controle).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Lokale Gateway op loopback
    - Bestaande werkruimte of standaardwerkruimte
    - Gateway-poort `18789`
    - Gateway-authenticatietoken wordt automatisch gegenereerd (ook op loopback)
    - Tailscale-publicatie uitgeschakeld
    - Telegram- en WhatsApp-DM’s standaard op allowlist (mogelijk wordt gevraagd om een telefoonnummer in te voeren)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Toont de volledige promptflow voor modus, werkruimte, Gateway, kanalen, daemon en Skills
  
</Tab>
</Tabs>

## Details van CLI-onboarding

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Volledige uitleg van lokale en externe flows, authenticatie- en modelmatrix, configuratie-output, wizard-RPC en de werking van signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Recepten voor niet-interactieve onboarding en voorbeelden van geautomatiseerde `agents add`.
  
</Card>
</Columns>

## Veelgebruikte vervolgcommando’s

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` betekent geen niet-interactieve modus. Gebruik in scripts `--non-interactive`.
</Note>

<Tip>
Aanbevolen: stel een Brave Search API-sleutel in zodat agenten `web_search` kunnen gebruiken (`web_fetch` werkt zonder sleutel). De eenvoudigste manier: voer `openclaw configure --section web` uit om `tools.web.search.apiKey` op te slaan. Documentatie: [Webtools](/tools/web).
</Tip>

## Gerelateerde documentatie

- CLI-commandoverzicht: [`openclaw onboard`](/cli/onboard)
- Onboarding van de macOS-app: [Onboarding](/start/onboarding)
- Stappen voor de eerste start van een agent: [Agent Bootstrap](/start/bootstrapping)
