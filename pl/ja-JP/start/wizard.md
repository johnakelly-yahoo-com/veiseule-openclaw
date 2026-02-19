---
read_when:
  - Podczas uruchamiania kreatora onboardingu lub konfiguracji
  - Podczas konfiguracji nowej maszyny
sidebarTitle: Wizard (CLI)
summary: "Kreator onboardingu CLI: interaktywna konfiguracja Gateway, obszaru roboczego, kanałów i Skills"
title: Kreator onboardingu (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Kreator onboardingu (CLI)

Kreator onboardingu CLI to zalecana ścieżka konfiguracji OpenClaw na macOS, Linux i Windows (przez WSL2). Oprócz lokalnego Gateway lub połączenia ze zdalnym Gateway konfiguruje domyślne ustawienia obszaru roboczego, kanały oraz Skills.

```bash
openclaw onboard
```

<Info>
Najszybszy sposób na rozpoczęcie pierwszego czatu: otwórz Control UI (konfiguracja kanałów nie jest wymagana). Uruchom `openclaw dashboard`, aby czatować w przeglądarce. Dokumentacja: [Dashboard](/web/dashboard).
</Info>

## Szybki start vs konfiguracja szczegółowa

Kreator rozpoczyna się od wyboru między **Szybkim startem** (ustawienia domyślne) a **Konfiguracją szczegółową** (pełna kontrola).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - lokalny Gateway na loopback
    - istniejący obszar roboczy lub domyślny obszar roboczy
    - port Gateway `18789`
    - token uwierzytelniający Gateway generowany automatycznie (również na loopback)
    - publikacja przez Tailscale wyłączona
    - DM w Telegram i WhatsApp domyślnie na liście dozwolonych (może być wymagane podanie numeru telefonu)
   
</Tab>
  <Tab title="詳細設定（完全な制御）">- Wyświetla pełny przepływ promptów dla trybów, obszarów roboczych, Gateway, kanałów, demonów i Skills
</Tab>
</Tabs>

## Szczegóły onboardingu CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">Pełne wyjaśnienie przepływów lokalnych i zdalnych, uwierzytelniania i macierzy modeli, wyników konfiguracji, RPC kreatora oraz działania signal-cli.
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">Przepisy na onboarding bez trybu interaktywnego oraz zautomatyzowane przykłady `agents add`.
</Card>
</Columns>

## Najczęściej używane polecenia uzupełniające

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` nie oznacza trybu nieinteraktywnego. W skryptach używaj `--non-interactive`.
</Note>

<Tip>
Zalecane: skonfiguruj klucz Brave Search API, aby agent mógł używać `web_search` (`web_fetch` działa bez klucza). Najprostszy sposób: uruchom `openclaw configure --section web`, co zapisze `tools.web.search.apiKey`. Dokumentacja: [Webツール](/tools/web).
</Tip>

## Powiązana dokumentacja

- Dokumentacja poleceń CLI: [`openclaw onboard`](/cli/onboard)
- Onboarding aplikacji macOS: [オンボーディング](/start/onboarding)
- Instrukcja pierwszego uruchomienia agenta: [エージェントブートストラップ](/start/bootstrapping)
