---
read_when:
  - Kapag pinapatakbo ang onboarding wizard o habang nagse-set up
  - Kapag nagse-set up ng bagong makina
sidebarTitle: Wizard (CLI)
summary: "CLI onboarding wizard: Interaktibong pag-setup ng Gateway, workspace, channel, at Skills"
title: Onboarding Wizard (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Onboarding Wizard (CLI)

Ang CLI onboarding wizard ang inirerekomendang paraan upang i-set up ang OpenClaw sa macOS, Linux, at Windows (sa pamamagitan ng WSL2). Kino-configure nito ang mga default na setting ng workspace, mga channel, at Skills, bukod pa sa lokal o remote na koneksyon sa Gateway.

```bash
openclaw onboard
```

<Info>
Pinakamabilis na paraan para simulan ang iyong unang chat: Buksan ang Control UI (hindi kailangan ang pag-set up ng channel). Patakbuhin ang `openclaw dashboard` upang makapag-chat sa browser. Dokumentasyon: [Dashboard](/web/dashboard).
</Info>

## Quickstart vs Advanced na Setup

Nagsisimula ang wizard sa pagpili sa pagitan ng **Quickstart** (mga default na setting) at **Advanced** (buong kontrol).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">    - Lokal na Gateway sa loopback
    - Umiiral na workspace o default na workspace
    - Gateway port `18789`
    - Awtomatikong nabubuo ang Gateway authentication token (nabubuo kahit sa loopback)
    - Naka-off ang Tailscale exposure
    - Ang Telegram at WhatsApp DM ay naka-allowlist bilang default (maaaring hingin ang paglalagay ng numero ng telepono)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">    - Ipinapakita ang kumpletong prompt flow para sa mode, workspace, Gateway, mga channel, daemon, at Skills
  
</Tab>
</Tabs>

## Mga detalye ng CLI onboarding

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">    Kumpletong paliwanag ng lokal at remote na daloy, authentication at model matrix, output ng configuration, wizard RPC, at galaw ng signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">    Mga recipe para sa non-interactive na onboarding at mga halimbawa ng awtomatikong `agents add`.
  
</Card>
</Columns>

## Mga karaniwang follow-up na command

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` ay hindi nangangahulugang non-interactive mode. Gamitin ang `--non-interactive` sa mga script.
</Note>

<Tip>
Inirerekomenda: Mag-set ng Brave Search API key upang magamit ng agent ang `web_search` (ang `web_fetch` ay gumagana kahit walang key). Pinakamadaling paraan: patakbuhin ang `openclaw configure --section web` at mase-save ang `tools.web.search.apiKey`. Dokumentasyon: [Webツール](/tools/web).
</Tip>

## Kaugnay na Dokumentasyon

- CLI command reference: [`openclaw onboard`](/cli/onboard)
- Onboarding ng macOS app: [オンボーディング](/start/onboarding)
- Mga hakbang sa unang pag-start ng agent: [エージェントブートストラップ](/start/bootstrapping)
