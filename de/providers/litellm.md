---
summary: "Führen Sie OpenClaw über den LiteLLM Proxy aus, um einen einheitlichen Modellzugriff und Kostenverfolgung zu ermöglichen"
read_when:
  - Sie möchten OpenClaw über einen LiteLLM-Proxy leiten
  - Sie benötigen Kostenverfolgung, Logging oder Modell-Routing über LiteLLM
---

# LiteLLM

[LiteLLM](https://litellm.ai) ist ein Open-Source-LLM-Gateway, das eine einheitliche API für über 100 Modellanbieter bereitstellt. Leite OpenClaw über LiteLLM, um zentrale Kostenverfolgung, Logging und die Flexibilität zu erhalten, Backends zu wechseln, ohne deine OpenClaw-Konfiguration zu ändern.

## Warum LiteLLM mit OpenClaw verwenden?

- **Kostenverfolgung** — Sieh genau, was OpenClaw über alle Modelle hinweg ausgibt
- **Modell-Routing** — Wechsle zwischen Claude, GPT-4, Gemini, Bedrock ohne Konfigurationsänderungen
- **Virtuelle Schlüssel** — Erstelle Schlüssel mit Ausgabenlimits für OpenClaw
- **Logging** — Vollständige Request-/Response-Logs für das Debugging
- **Fallbacks** — Automatisches Failover, wenn dein primärer Anbieter ausfällt

## Schnellstart

### Über das Onboarding

```bash
openclaw onboard --auth-choice litellm-api-key
```

### Manuelle Einrichtung

1. Starte den LiteLLM-Proxy:

```bash
pip install 'litellm[proxy]'
litellm --model claude-opus-4-6
```

2. Verbinde OpenClaw mit LiteLLM:

```bash
export LITELLM_API_KEY="your-litellm-key"

openclaw
```

Das ist alles. OpenClaw wird nun über LiteLLM geroutet.

## Konfiguration

### Umgebungsvariablen

```bash
export LITELLM_API_KEY="sk-litellm-key"
```

### Konfigurationsdatei

```json5
{
  models: {
    providers: {
      litellm: {
        baseUrl: "http://localhost:4000",
        apiKey: "${LITELLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "claude-opus-4-6",
            name: "Claude Opus 4.6",
            reasoning: true,
            input: ["text", "image"],
            contextWindow: 200000,
            maxTokens: 64000,
          },
          {
            id: "gpt-4o",
            name: "GPT-4o",
            reasoning: false,
            input: ["text", "image"],
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "litellm/claude-opus-4-6" },
    },
  },
}
```

## Virtuelle Schlüssel

Erstelle einen dedizierten Schlüssel für OpenClaw mit Ausgabenlimits:

```bash
curl -X POST "http://localhost:4000/key/generate" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "key_alias": "openclaw",
    "max_budget": 50.00,
    "budget_duration": "monthly"
  }'
```

Verwende den generierten Schlüssel als `LITELLM_API_KEY`.

## Modell-Routing

LiteLLM kann Modellanfragen an verschiedene Backends weiterleiten. Konfiguriere dies in deiner LiteLLM-`config.yaml`:

```yaml
model_list:
  - model_name: claude-opus-4-6
    litellm_params:
      model: claude-opus-4-6
      api_key: os.environ/ANTHROPIC_API_KEY

  - model_name: gpt-4o
    litellm_params:
      model: gpt-4o
      api_key: os.environ/OPENAI_API_KEY
```

OpenClaw fordert weiterhin `claude-opus-4-6` an — LiteLLM übernimmt das Routing.

## Nutzung anzeigen

Überprüfe das LiteLLM-Dashboard oder die API:

```bash
# Schlüsselinformationen
curl "http://localhost:4000/key/info" \
  -H "Authorization: Bearer sk-litellm-key"

# Ausgabenprotokolle
curl "http://localhost:4000/spend/logs" \
  -H "Authorization: Bearer $LITELLM_MASTER_KEY"
```

## Hinweise

- LiteLLM läuft standardmäßig auf `http://localhost:4000`
- OpenClaw verbindet sich über den OpenAI-kompatiblen `/v1/chat/completions`-Endpoint
- Alle OpenClaw-Funktionen funktionieren über LiteLLM — keine Einschränkungen

## Siehe auch

- [LiteLLM Docs](https://docs.litellm.ai)
- [Modellanbieter](/concepts/model-providers)

