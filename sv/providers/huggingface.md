---
summary: "Konfiguration av Hugging Face Inference (autentisering + modellval)"
read_when:
  - Du vill använda Hugging Face Inference med OpenClaw
  - Du behöver HF-tokenens miljövariabel eller CLI-autentiseringsval
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) erbjuder OpenAI-kompatibla chat completions via ett enda router-API. Du får tillgång till många modeller (DeepSeek, Llama med flera) med en enda token. OpenClaw använder den **OpenAI-kompatibla endpointen** (endast chat completions); för text-till-bild, embeddings eller tal, använd [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) direkt.

- Leverantör: `huggingface`
- Autentisering: `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN` (finjusterad token med behörigheten **Make calls to Inference Providers**)
- API: OpenAI-kompatibelt (`https://router.huggingface.co/v1`)
- Fakturering: En enda HF-token; [prissättning](https://huggingface.co/docs/inference-providers/pricing) följer leverantörens priser med en gratisnivå.

## Snabbstart

1. Skapa en finjusterad token på [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) med behörigheten **Make calls to Inference Providers**.
2. Kör onboarding och välj **Hugging Face** i leverantörens rullgardinsmeny, och ange sedan din API-nyckel när du uppmanas:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. I rullgardinsmenyn **Default Hugging Face model**, välj den modell du vill ha (listan laddas från Inference API när du har en giltig token; annars visas en inbyggd lista). Ditt val sparas som standardmodell.
4. Du kan också ange eller ändra standardmodellen senare i konfigurationen:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Icke-interaktivt exempel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Detta kommer att ange `huggingface/deepseek-ai/DeepSeek-R1` som standardmodell.

## Miljöanmärkning

Om Gateway körs som en daemon (launchd/systemd), se till att `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN`
är tillgänglig för den processen (till exempel i `~/.openclaw/.env` eller via
`env.shellEnv`).

## Modellidentifiering och rullgardinsmeny i onboarding

OpenClaw identifierar modeller genom att anropa **Inference-endpointen direkt**:

```bash
GET https://router.huggingface.co/v1/models
```

(Valfritt: skicka `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` eller `$HF_TOKEN` för hela listan; vissa endpoints returnerar en delmängd utan autentisering.) Svaret är i OpenAI-stil `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

När du konfigurerar en Hugging Face API-nyckel (via onboarding, `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN`) använder OpenClaw denna GET för att upptäcka tillgängliga chat-completion-modeller. Under **interaktiv onboarding**, efter att du har angett din token ser du en rullgardinsmeny för **Default Hugging Face model** som fylls med data från den listan (eller den inbyggda katalogen om begäran misslyckas). Vid körning (t.ex. vid Gateway-start), när en nyckel finns, anropar OpenClaw återigen **GET** `https://router.huggingface.co/v1/models` för att uppdatera katalogen. Listan slås samman med en inbyggd katalog (för metadata som kontextfönster och kostnad). Om begäran misslyckas eller om ingen nyckel är angiven används endast den inbyggda katalogen.

## Modellnamn och redigerbara alternativ

- **Namn från API:** Modellens visningsnamn **hämtas från GET /v1/models** när API:et returnerar `name`, `title` eller `display_name`; annars härleds det från modell-id:t (t.ex. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Åsidosätt visningsnamn:** Du kan ange en anpassad etikett per modell i konfigurationen så att den visas som du vill i CLI och UI:

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

- **Leverantörs- / policyval:** Lägg till ett suffix till **model id** för att välja hur routern ska välja backend:

  - **`:fastest`** — högsta genomströmning (routern väljer; leverantörsvalet är **låst** — ingen interaktiv backend-väljare).
  - **`:cheapest`** — lägsta kostnad per output-token (routern väljer; leverantörsvalet är **låst**).
  - **`:provider`** — tvinga en specifik backend (t.ex. `:sambanova`, `:together`).

  När du väljer **:cheapest** eller **:fastest** (t.ex. i modellrullgardinsmenyn under onboarding) är leverantören låst: routern avgör baserat på kostnad eller hastighet och inget valfritt steg för att ”föredra specifik backend” visas. Du kan lägga till dessa som separata poster i `models.providers.huggingface.models` eller ange `model.primary` med suffixet. Du kan också ange din standardordning i [Inference Provider settings](https://hf.co/settings/inference-providers) (inget suffix = använd den ordningen).

- **Konfigurationssammanfogning:** Befintliga poster i `models.providers.huggingface.models` (t.ex. i `models.json`) behålls när konfigurationen slås samman. Så alla anpassade `name`, `alias` eller modellalternativ som du anger där bevaras.

## Modell-ID:n och konfigurationsexempel

Modellreferenser använder formatet `huggingface/<org>/<model>` (Hub-stil-ID:n). Listan nedan är från **GET** `https://router.huggingface.co/v1/models`; din katalog kan innehålla fler.

**Exempel-ID:n (från inference-endpointen):**

| Modell                                 | Ref (prefixa med `huggingface/`) |
| -------------------------------------- | --------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                         |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                          |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                 |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                  |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                               |
| GLM 4.7                | `zai-org/GLM-4.7`                                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                              |

Du kan lägga till `:fastest`, `:cheapest` eller `:provider` (t.ex. `:together`, `:sambanova`) till modell-id:t. Ställ in din standardordning i [Inference Provider settings](https://hf.co/settings/inference-providers); se [Inference Providers](https://huggingface.co/docs/inference-providers) och **GET** `https://router.huggingface.co/v1/models` för den fullständiga listan.

### Fullständiga konfigurationsexempel

**Primär DeepSeek R1 med Qwen som fallback:**

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

**Qwen som standard, med :cheapest- och :fastest-varianter:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (billigast)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (snabbast)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS med alias:**

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

**Tvinga en specifik backend med :provider:**

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

**Flera Qwen- och DeepSeek-modeller med policy-suffix:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (billig)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (snabb)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```

