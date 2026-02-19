---
summary: "OpenClaw mit vLLM ausführen (OpenAI-kompatibler lokaler Server)"
read_when:
  - Sie möchten OpenClaw mit einem lokalen vLLM-Server ausführen
  - Sie möchten OpenAI-kompatible `/v1`-Endpunkte mit Ihren eigenen Modellen verwenden
title: "vLLM"
---

# vLLM

vLLM kann Open-Source- (und einige benutzerdefinierte) Modelle über eine **OpenAI-kompatible** HTTP-API bereitstellen. OpenClaw kann sich über die `openai-completions` API mit vLLM verbinden.

OpenClaw kann verfügbare Modelle auch **automatisch erkennen**, wenn Sie sich mit `VLLM_API_KEY` dafür entscheiden (beliebiger Wert funktioniert, wenn Ihr Server keine Authentifizierung erzwingt) und keinen expliziten `models.providers.vllm`-Eintrag definieren.

## Schnellstart

1. Starten Sie vLLM mit einem OpenAI-kompatiblen Server.

Ihre Basis-URL sollte `/v1`-Endpunkte bereitstellen (z. B. `/v1/models`, `/v1/chat/completions`). vLLM läuft üblicherweise auf:

- `http://127.0.0.1:8000/v1`

2. Opt-in (beliebiger Wert funktioniert, wenn keine Authentifizierung konfiguriert ist):

```bash
export VLLM_API_KEY="vllm-local"
```

3. Wählen Sie ein Modell aus (ersetzen Sie es durch eine Ihrer vLLM-Modell-IDs):

```json5
{
  agents: {
    defaults: {
      model: { primary: "vllm/your-model-id" },
    },
  },
}
```

## Modellerkennung (impliziter Provider)

Wenn `VLLM_API_KEY` gesetzt ist (oder ein Auth-Profil existiert) und Sie `models.providers.vllm` **nicht** definieren, führt OpenClaw folgende Anfrage aus:

- `GET http://127.0.0.1:8000/v1/models`

…und wandelt die zurückgegebenen IDs in Modelleinträge um.

Wenn Sie `models.providers.vllm` explizit festlegen, wird die automatische Erkennung übersprungen und Sie müssen Modelle manuell definieren.

## Explizite Konfiguration (manuelle Modelle)

Verwenden Sie eine explizite Konfiguration, wenn:

- vLLM läuft auf einem anderen Host/Port.
- Sie möchten `contextWindow`-/`maxTokens`-Werte festlegen.
- Ihr Server erfordert einen echten API-Schlüssel (oder Sie möchten Header kontrollieren).

```json5
{
  models: {
    providers: {
      vllm: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "${VLLM_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "your-model-id",
            name: "Local vLLM Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## Fehlerbehebung

- Überprüfen Sie, ob der Server erreichbar ist:

```bash
curl http://127.0.0.1:8000/v1/models
```

- Wenn Anfragen mit Authentifizierungsfehlern fehlschlagen, setzen Sie einen echten `VLLM_API_KEY`, der Ihrer Serverkonfiguration entspricht, oder konfigurieren Sie den Provider explizit unter `models.providers.vllm`.

