---
summary: "CLI-Referenz für `openclaw onboard` (interaktiver Onboarding-Assistent)"
read_when:
  - Sie möchten eine geführte Einrichtung für Gateway, Workspace, Authentifizierung, Kanäle und Skills
title: "onboard"
---

# `openclaw onboard`

Interaktiver Onboarding-Assistent (lokale oder entfernte Gateway-Einrichtung).

## Zugehörige Anleitungen

- CLI-Onboarding-Hub: [Onboarding Wizard (CLI)](/start/wizard)
- Onboarding-Überblick: [Onboarding Overview](/start/onboarding-overview)
- CLI-Automatisierung: [CLI Automation](/start/wizard-cli-automation)
- CLI-Onboarding-Referenz: [CLI Onboarding Reference](/start/wizard-cli-reference)
- macOS-Onboarding: [Onboarding (macOS App)](/start/onboarding)

## Beispiele

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Nicht interaktiver benutzerdefinierter Provider:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` ist im nicht interaktiven Modus optional. Wenn nicht angegeben, prüft das Onboarding `CUSTOM_API_KEY`.

Nicht interaktive Z.AI-Endpunktoptionen:

Hinweis: `--auth-choice zai-api-key` erkennt jetzt automatisch den besten Z.AI-Endpunkt für Ihren Schlüssel (bevorzugt die allgemeine API mit `zai/glm-5`).
Wenn Sie speziell die GLM Coding Plan-Endpunkte verwenden möchten, wählen Sie `zai-coding-global` oder `zai-coding-cn`.

```bash
# Endpunktauswahl ohne Prompt
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Weitere Z.AI-Endpunktoptionen:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Flow Notizen:

- `quickstart`: minimale Abfragen, generiert automatisch ein Gateway-Token.
- `manual`: vollständige Abfragen für Port/Bind/Auth (Alias von `advanced`).
- Schnellster erster Chat: `openclaw dashboard` (Control-UI, keine Kanaleinrichtung).
- Custom Provider: Verbinden Sie jeden OpenAI- oder Anthropic-kompatiblen Endpunkt,
  einschließlich nicht aufgeführter gehosteter Anbieter. Verwenden Sie Unknown zur automatischen Erkennung.

## Häufige Folgekommandos

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` impliziert keinen nicht-interaktiven Modus. Verwenden Sie `--non-interactive` für Skripte.
</Note>
