---
summary: "Hugging Face Inference-instelling (auth + modelselectie)"
read_when:
  - Je wilt Hugging Face Inference gebruiken met OpenClaw
  - Je hebt de HF-token env-var of CLI-auth-keuze nodig
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) bieden OpenAI-compatibele chat completions via één enkele router-API. Je krijgt toegang tot veel modellen (DeepSeek, Llama en meer) met één token. OpenClaw gebruikt de **OpenAI-compatibele endpoint** (alleen chat completions); voor text-to-image, embeddings of spraak gebruik je de [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) rechtstreeks.

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` of `HF_TOKEN` (fijnmazige token met **Make calls to Inference Providers**)
- API: OpenAI-compatibel (`https://router.huggingface.co/v1`)
- Facturatie: Eén HF-token; [prijzen](https://huggingface.co/docs/inference-providers/pricing) volgen de tarieven van de provider met een gratis tier.

## Snelstart

1. Maak een fijnmazige token aan via [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) met de permissie **Make calls to Inference Providers**.
2. Voer onboarding uit en kies **Hugging Face** in de provider-dropdown, en voer vervolgens je API-sleutel in wanneer daarom wordt gevraagd:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. Kies in de dropdown **Default Hugging Face model** het gewenste model (de lijst wordt geladen vanuit de Inference API wanneer je een geldige token hebt; anders wordt een ingebouwde lijst getoond). Je keuze wordt opgeslagen als het standaardmodel.
4. Je kunt het standaardmodel later ook instellen of wijzigen in de configuratie:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Niet-interactief voorbeeld

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Dit stelt `huggingface/deepseek-ai/DeepSeek-R1` in als het standaardmodel.

## Omgevingsopmerking

Als de Gateway als een daemon draait (launchd/systemd), zorg er dan voor dat `HUGGINGFACE_HUB_TOKEN` of `HF_TOKEN`
beschikbaar is voor dat proces (bijvoorbeeld in `~/.openclaw/.env` of via
`env.shellEnv`).

## Modeldetectie en onboarding-dropdown

OpenClaw detecteert modellen door de **Inference-endpoint rechtstreeks** aan te roepen:

```bash
GET https://router.huggingface.co/v1/models
```

(Optioneel: stuur `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` of `$HF_TOKEN` mee voor de volledige lijst; sommige endpoints geven zonder auth slechts een subset terug.) De response heeft OpenAI-stijl `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Wanneer je een Hugging Face API-sleutel configureert (via onboarding, `HUGGINGFACE_HUB_TOKEN` of `HF_TOKEN`), gebruikt OpenClaw deze GET om beschikbare chat-completion-modellen te ontdekken. Tijdens **interactieve onboarding**, nadat je je token hebt ingevoerd, zie je een **Standaard Hugging Face-model** dropdown die wordt gevuld vanuit die lijst (of de ingebouwde catalogus als het verzoek mislukt). Tijdens runtime (bijv. bij het opstarten van Gateway), wanneer een sleutel aanwezig is, roept OpenClaw opnieuw **GET** `https://router.huggingface.co/v1/models` aan om de catalogus te verversen. De lijst wordt samengevoegd met een ingebouwde catalogus (voor metadata zoals contextvenster en kosten). Als het verzoek mislukt of er geen sleutel is ingesteld, wordt alleen de ingebouwde catalogus gebruikt.

## Modelnamen en bewerkbare opties

- **Naam van API:** De weergavenaam van het model wordt **gevuld via GET /v1/models** wanneer de API `name`, `title` of `display_name` retourneert; anders wordt deze afgeleid van de model-id (bijv. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Weergavenaam overschrijven:** Je kunt per model een aangepast label instellen in de configuratie zodat het in de CLI en UI wordt weergegeven zoals jij wilt:

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

- **Provider- / beleidsselectie:** Voeg een suffix toe aan de **model-id** om te bepalen hoe de router de backend kiest:

  - **`:fastest`** — hoogste throughput (router kiest; providerkeuze is **vergrendeld** — geen interactieve backendkiezer).
  - **`:cheapest`** — laagste kosten per output-token (router kiest; providerkeuze is **vergrendeld**).
  - **`:provider`** — forceer een specifieke backend (bijv. `:sambanova`, `:together`).

  Wanneer je **:cheapest** of **:fastest** selecteert (bijv. in de model-dropdown tijdens onboarding), is de provider vergrendeld: de router beslist op basis van kosten of snelheid en er wordt geen optionele stap “specifieke backend prefereren” getoond. Je kunt deze als afzonderlijke items toevoegen in `models.providers.huggingface.models` of `model.primary` instellen met de suffix. Je kunt ook je standaardvolgorde instellen in [Inference Provider settings](https://hf.co/settings/inference-providers) (geen suffix = gebruik die volgorde).

- **Config-samenvoeging:** Bestaande items in `models.providers.huggingface.models` (bijv. in `models.json`) blijven behouden wanneer de configuratie wordt samengevoegd. Dus alle aangepaste `name`, `alias` of modelopties die je daar instelt, blijven behouden.

## Model-ID’s en configuratievoorbeelden

Modelreferenties gebruiken de vorm `huggingface/<org>/<model>` (Hub-stijl ID’s). De onderstaande lijst is afkomstig van **GET** `https://router.huggingface.co/v1/models`; jouw catalogus kan meer bevatten.

**Voorbeeld-ID’s (van het inference-endpoint):**

| Model                                  | Ref (prefix met `huggingface/`) |
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

Je kunt `:fastest`, `:cheapest` of `:provider` (bijv. `:together`, `:sambanova`) toevoegen aan de model-id. Stel je standaardvolgorde in bij [Inference Provider settings](https://hf.co/settings/inference-providers); zie [Inference Providers](https://huggingface.co/docs/inference-providers) en **GET** `https://router.huggingface.co/v1/models` voor de volledige lijst.

### Volledige configuratievoorbeelden

**Primair DeepSeek R1 met Qwen als fallback:**

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

**Qwen als standaard, met :cheapest- en :fastest-varianten:**

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

**DeepSeek + Llama + GPT-OSS met aliassen:**

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

**Forceer een specifieke backend met :provider:**

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

**Meerdere Qwen- en DeepSeek-modellen met policy-suffixen:**

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
