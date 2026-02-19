---
summary: "Überblick über OpenClaw-Onboarding-Optionen und -Abläufe"
read_when:
  - Auswahl eines Onboarding-Pfads
  - Einrichten einer neuen Umgebung
title: "Onboarding-Überblick"
sidebarTitle: "Onboarding-Überblick"
---

# Onboarding-Überblick

OpenClaw unterstützt mehrere Onboarding-Pfade, je nachdem, wo das Gateway läuft
und wie Sie Provider konfigurieren möchten.

## Wählen Sie Ihren Onboarding-Pfad

- **CLI-Assistent** für macOS, Linux und Windows (über WSL2).
- **macOS-App** für einen geführten ersten Start auf Apple Silicon oder Intel-Macs.

## CLI-Onboarding-Assistent

Führen Sie den Assistenten in einem Terminal aus:

```bash
openclaw onboard
```

Verwende den CLI-Wizard, wenn du die volle Kontrolle über das Gateway, den Workspace,
Channels und Skills haben möchtest. Dokumentation:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS-App-Onboarding

Verwende die OpenClaw-App, wenn du eine vollständig geführte Einrichtung unter macOS wünschst. Dokumentation:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

Wenn du einen Endpunkt benötigst, der nicht aufgeführt ist – einschließlich gehosteter Anbieter, die standardmäßige OpenAI- oder Anthropic-APIs bereitstellen – wähle im CLI-Wizard **Custom Provider** aus. Du wirst aufgefordert:

- OpenAI-kompatibel, Anthropic-kompatibel oder **Unknown** (automatische Erkennung) auszuwählen.
- Eine Basis-URL und einen API-Schlüssel einzugeben (falls vom Anbieter erforderlich).
- Eine Modell-ID und einen optionalen Alias anzugeben.
- Eine Endpoint-ID auszuwählen, damit mehrere benutzerdefinierte Endpunkte koexistieren können.

Für detaillierte Schritte folge der oben verlinkten CLI-Onboarding-Dokumentation.

