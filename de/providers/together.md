---
summary: "Together AI-Einrichtung (Authentifizierung + Modellauswahl)"
read_when:
  - Sie möchten Together AI mit OpenClaw verwenden
  - Sie benötigen die API-Schlüssel-Umgebungsvariable oder die CLI-Authentifizierungsoption
---

# Together AI

[Together AI](https://together.ai) bietet über eine einheitliche API Zugriff auf führende Open-Source-Modelle, darunter Llama, DeepSeek, Kimi und weitere.

- Provider: `together`
- Authentifizierung: `TOGETHER_API_KEY`
- API: OpenAI-kompatibel

## Schnellstart

1. Setzen Sie den API-Schlüssel (empfohlen: für das Gateway speichern):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Legen Sie ein Standardmodell fest:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Nicht-interaktives Beispiel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Dadurch wird `together/moonshotai/Kimi-K2.5` als Standardmodell festgelegt.

## Umgebungshinweis

Wenn das Gateway als Daemon (launchd/systemd) läuft, stellen Sie sicher, dass `TOGETHER_API_KEY`
für diesen Prozess verfügbar ist (zum Beispiel in `~/.clawdbot/.env` oder über
`env.shellEnv`).

## Verfügbare Modelle

Together AI bietet Zugriff auf viele beliebte Open-Source-Modelle:

- **GLM 4.7 Fp8** - Standardmodell mit 200K Kontextfenster
- **Llama 3.3 70B Instruct Turbo** - Schnell, effizient bei der Befolgung von Anweisungen
- **Llama 4 Scout** - Vision-Modell mit Bildverständnis
- **Llama 4 Maverick** - Erweiterte Vision- und Reasoning-Fähigkeiten
- **DeepSeek V3.1** - Leistungsstarkes Modell für Coding und Reasoning
- **DeepSeek R1** - Fortschrittliches Reasoning-Modell
- **Kimi K2 Instruct** - Hochleistungsmodell mit 262K Kontextfenster

Alle Modelle unterstützen standardmäßige Chat-Completions und sind OpenAI-API-kompatibel.
