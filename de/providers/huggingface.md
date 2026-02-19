---
summary: "Hugging Face Inference Einrichtung (Authentifizierung + Modellauswahl)"
read_when:
  - Sie möchten Hugging Face Inference mit OpenClaw verwenden
  - Sie benötigen die HF-Token-Umgebungsvariable oder die CLI-Authentifizierungsoption
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) bieten OpenAI-kompatible Chat-Completions über eine einzelne Router-API an. Sie erhalten Zugriff auf viele Modelle (DeepSeek, Llama und mehr) mit einem einzigen Token. OpenClaw verwendet den **OpenAI-kompatiblen Endpunkt** (nur Chat-Completions); für Text-zu-Bild, Embeddings oder Sprache verwenden Sie direkt die [HF inference clients](https://huggingface.co/docs/api-inference/quicktour).

- Provider: `huggingface`
- Authentifizierung: `HUGGINGFACE_HUB_TOKEN` oder `HF_TOKEN` (feingranulares Token mit **Make calls to Inference Providers**)
- API: OpenAI-kompatibel (`https://router.huggingface.co/v1`)
- Abrechnung: Einzelnes HF-Token; [Preise](https://huggingface.co/docs/inference-providers/pricing) richten sich nach den Anbietertarifen mit einer kostenlosen Stufe.

## Schnellstart

1. Erstellen Sie ein feingranulares Token unter [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) mit der Berechtigung **Make calls to Inference Providers**.
2. Führen Sie das Onboarding aus und wählen Sie im Provider-Dropdown **Hugging Face** aus. Geben Sie anschließend Ihren API-Schlüssel ein, wenn Sie dazu aufgefordert werden:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. Wählen Sie im Dropdown **Default Hugging Face model** das gewünschte Modell aus (die Liste wird aus der Inference-API geladen, wenn Sie ein gültiges Token haben; andernfalls wird eine integrierte Liste angezeigt). Ihre Auswahl wird als Standardmodell gespeichert.
4. Sie können das Standardmodell später auch in der Konfiguration festlegen oder ändern:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Nicht-interaktives Beispiel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Dadurch wird `huggingface/deepseek-ai/DeepSeek-R1` als Standardmodell festgelegt.

## Umgebungshinweis

Wenn das Gateway als Daemon (launchd/systemd) läuft, stellen Sie sicher, dass `HUGGINGFACE_HUB_TOKEN` oder `HF_TOKEN`
für diesen Prozess verfügbar ist (zum Beispiel in `~/.openclaw/.env` oder über
`env.shellEnv`).

## Modellerkennung und Onboarding-Dropdown

OpenClaw erkennt Modelle durch direkten Aufruf des **Inference-Endpunkts**:

```bash
GET https://router.huggingface.co/v1/models
```

(Optional: Senden Sie `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` oder `$HF_TOKEN` für die vollständige Liste; einige Endpunkte liefern ohne Authentifizierung nur eine Teilmenge zurück.) Die Antwort ist im OpenAI-Stil `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Wenn Sie einen Hugging Face API-Schlüssel konfigurieren (über das Onboarding, `HUGGINGFACE_HUB_TOKEN` oder `HF_TOKEN`), verwendet OpenClaw diesen GET-Aufruf, um verfügbare Chat-Completion-Modelle zu ermitteln. Während des **interaktiven Onboardings** sehen Sie nach Eingabe Ihres Tokens ein Dropdown **Default Hugging Face model**, das mit dieser Liste befüllt wird (oder mit dem integrierten Katalog, falls die Anfrage fehlschlägt). Zur Laufzeit (z. B. beim Start des Gateway) ruft OpenClaw bei vorhandenem Schlüssel erneut **GET** `https://router.huggingface.co/v1/models` auf, um den Katalog zu aktualisieren. Die Liste wird mit einem integrierten Katalog zusammengeführt (für Metadaten wie Kontextfenster und Kosten). Wenn die Anfrage fehlschlägt oder kein Schlüssel gesetzt ist, wird nur der integrierte Katalog verwendet.

## Modellnamen und bearbeitbare Optionen

- **Name aus der API:** Der angezeigte Modellname wird aus **GET /v1/models** übernommen, wenn die API `name`, `title` oder `display_name` zurückgibt; andernfalls wird er aus der Modell-ID abgeleitet (z. B. `deepseek-ai/DeepSeek-R1` → „DeepSeek R1“).
- **Anzeigenamen überschreiben:** Sie können pro Modell in der Konfiguration eine benutzerdefinierte Bezeichnung festlegen, sodass es in der CLI und UI wie gewünscht angezeigt wird:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider- / Richtlinienauswahl:** Hängen Sie ein Suffix an die **Modell-ID** an, um festzulegen, wie der Router das Backend auswählt:

  - **`:fastest`** — höchster Durchsatz (Auswahl durch den Router; Provider-Auswahl ist **gesperrt** — kein interaktiver Backend-Selektor).
  - **`:cheapest`** — niedrigste Kosten pro Ausgabetoken (Auswahl durch den Router; Provider-Auswahl ist **gesperrt**).
  - **`:provider`** — erzwingt ein bestimmtes Backend (z. B. `:sambanova`, `:together`).

  Wenn Sie **:cheapest** oder **:fastest** auswählen (z. B. im Modell-Dropdown während des Onboardings), ist der Provider gesperrt: Der Router entscheidet nach Kosten oder Geschwindigkeit, und es wird kein optionaler Schritt „bestimmtes Backend bevorzugen“ angezeigt. Sie können diese als separate Einträge in `models.providers.huggingface.models` hinzufügen oder `model.primary` mit dem Suffix festlegen. Sie können Ihre Standardreihenfolge auch in den [Inference Provider settings](https://hf.co/settings/inference-providers) festlegen (kein Suffix = diese Reihenfolge verwenden).

- **Konfigurationszusammenführung:** Bestehende Einträge in `models.providers.huggingface.models` (z. B. in `models.json`) bleiben bei der Zusammenführung der Konfiguration erhalten. Alle dort festgelegten benutzerdefinierten `name`-, `alias`- oder Modelloptionen bleiben somit erhalten.

## Modell-IDs und Konfigurationsbeispiele

Modellreferenzen verwenden das Format `huggingface/<org>/<model>` (Hub-IDs). Die folgende Liste stammt von **GET** `https://router.huggingface.co/v1/models`; Ihr Katalog kann weitere Einträge enthalten.

**Beispiel-IDs (vom Inference-Endpunkt):**

| Modell                                 | Ref (mit Präfix `huggingface/`) |
| -------------------------------------- | -------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                          |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                        |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                    |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                         |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                   |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                 |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                              |
| GLM 4.7                | `zai-org/GLM-4.7`                                  |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                             |

Sie können `:fastest`, `:cheapest` oder `:provider` (z. B. `:together`, `:sambanova`) an die Modell-ID anhängen. Legen Sie Ihre Standardreihenfolge in den [Inference Provider settings](https://hf.co/settings/inference-providers) fest; siehe [Inference Providers](https://huggingface.co/docs/inference-providers) und **GET** `https://router.huggingface.co/v1/models` für die vollständige Liste.

### Vollständige Konfigurationsbeispiele

**Primäres DeepSeek R1 mit Qwen als Fallback:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen als Standard mit :cheapest- und :fastest-Varianten:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS mit Aliasen:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Einen bestimmten Backend mit :provider erzwingen:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Mehrere Qwen- und DeepSeek-Modelle mit Policy-Suffixen:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```
