---
summary: "Opsætning af Hugging Face Inference (godkendelse + modelvalg)"
read_when:
  - Du vil bruge Hugging Face Inference med OpenClaw
  - Du har brug for HF token-miljøvariablen eller CLI-godkendelsesvalget
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) tilbyder OpenAI-kompatible chat completions via én samlet router-API. Du får adgang til mange modeller (DeepSeek, Llama m.fl.) med ét token. OpenClaw bruger det **OpenAI-kompatible endpoint** (kun chat completions); til text-to-image, embeddings eller tale skal du bruge [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) direkte.

- Udbyder: `huggingface`
- Godkendelse: `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN` (finmasket token med **Make calls to Inference Providers**)
- API: OpenAI-kompatibel (`https://router.huggingface.co/v1`)
- Fakturering: Ét HF-token; [priser](https://huggingface.co/docs/inference-providers/pricing) følger udbyderens takster med et gratis niveau.

## Hurtig start

1. Opret et finmasket token på [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) med tilladelsen **Make calls to Inference Providers**.
2. Kør onboarding og vælg **Hugging Face** i udbyderens dropdown-menu, og indtast derefter din API-nøgle, når du bliver bedt om det:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. I dropdown-menuen **Default Hugging Face model** skal du vælge den model, du ønsker (listen indlæses fra Inference API, når du har et gyldigt token; ellers vises en indbygget liste). Dit valg gemmes som standardmodel.
4. Du kan også angive eller ændre standardmodellen senere i konfigurationen:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Ikke-interaktivt eksempel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Dette vil sætte `huggingface/deepseek-ai/DeepSeek-R1` som standardmodel.

## Bemærkning om miljø

Hvis Gateway kører som en daemon (launchd/systemd), skal du sikre, at `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN`
er tilgængelig for den proces (for eksempel i `~/.openclaw/.env` eller via
`env.shellEnv`).

## Modelopdagelse og onboarding-rullemenu

OpenClaw finder modeller ved at kalde **Inference-endpointet direkte**:

```bash
GET https://router.huggingface.co/v1/models
```

(Valgfrit: send `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` eller `$HF_TOKEN` for den fulde liste; nogle endpoints returnerer et udsnit uden godkendelse.) Svaret er i OpenAI-format `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Når du konfigurerer en Hugging Face API-nøgle (via onboarding, `HUGGINGFACE_HUB_TOKEN` eller `HF_TOKEN`), bruger OpenClaw denne GET til at finde tilgængelige chat-completion-modeller. Under **interaktiv onboarding**, efter du har indtastet din token, ser du en **Standard Hugging Face-model**-rullemenu, som udfyldes fra denne liste (eller det indbyggede katalog, hvis forespørgslen mislykkes). Ved kørsel (f.eks. ved opstart af Gateway), når en nøgle er til stede, kalder OpenClaw igen **GET** `https://router.huggingface.co/v1/models` for at opdatere kataloget. Listen flettes sammen med et indbygget katalog (for metadata som kontekstvindue og omkostning). Hvis forespørgslen mislykkes, eller ingen nøgle er angivet, bruges kun det indbyggede katalog.

## Modelnavne og redigerbare indstillinger

- **Navn fra API:** Modellens visningsnavn **hentes fra GET /v1/models**, når API’et returnerer `name`, `title` eller `display_name`; ellers afledes det af model-id’et (f.eks. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Tilsidesæt visningsnavn:** Du kan angive en brugerdefineret etiket pr. model i config, så den vises, som du ønsker, i CLI og UI:

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

- **Valg af udbyder / politik:** Tilføj et suffiks til **model-id’et** for at vælge, hvordan routeren vælger backend:

  - **`:fastest`** — højeste throughput (routeren vælger; valg af udbyder er **låst** — ingen interaktiv backend-vælger).
  - **`:cheapest`** — laveste pris pr. output-token (routeren vælger; valg af udbyder er **låst**).
  - **`:provider`** — gennemtving en specifik backend (f.eks. `:sambanova`, `:together`).

  Når du vælger **:cheapest** eller **:fastest** (f.eks. i onboarding-modellens rullemenu), er udbyderen låst: routeren beslutter ud fra pris eller hastighed, og der vises ikke et valgfrit trin til at “foretrække en specifik backend”. Du kan tilføje disse som separate poster i `models.providers.huggingface.models` eller angive `model.primary` med suffikset. Du kan også angive din standardrækkefølge i [Inference Provider settings](https://hf.co/settings/inference-providers) (intet suffiks = brug denne rækkefølge).

- **Sammenfletning af config:** Eksisterende poster i `models.providers.huggingface.models` (f.eks. i `models.json`) bevares, når config flettes sammen. Så alle brugerdefinerede `name`, `alias` eller modelindstillinger, du angiver der, bevares.

## Model-id’er og konfigurationseksempler

Modelreferencer bruger formen `huggingface/<org>/<model>` (Hub-stil-id’er). Listen nedenfor er fra **GET** `https://router.huggingface.co/v1/models`; dit katalog kan indeholde flere.

**Eksempel-id’er (fra inference-endpointet):**

| Model                                  | Ref (prefix med `huggingface/`) |
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

Du kan tilføje `:fastest`, `:cheapest` eller `:provider` (f.eks. `:together`, `:sambanova`) til model-id'et. Angiv din standardrækkefølge i [Inference Provider settings](https://hf.co/settings/inference-providers); se [Inference Providers](https://huggingface.co/docs/inference-providers) og **GET** `https://router.huggingface.co/v1/models` for den fulde liste.

### Komplette konfigurationseksempler

**Primær DeepSeek R1 med Qwen som fallback:**

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

**Qwen som standard med :cheapest- og :fastest-varianter:**

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

**DeepSeek + Llama + GPT-OSS med aliasser:**

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

**Tving en specifik backend med :provider:**

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

**Flere Qwen- og DeepSeek-modeller med policy-suffikser:**

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
