---
summary: "Konfiguracja Hugging Face Inference (uwierzytelnianie + wybór modelu)"
read_when:
  - Chcesz używać Hugging Face Inference z OpenClaw
  - Potrzebujesz zmiennej środowiskowej z tokenem HF lub wyboru uwierzytelniania w CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) oferują zgodne z OpenAI uzupełnianie czatu przez pojedyncze API routera. Uzyskujesz dostęp do wielu modeli (DeepSeek, Llama i innych) za pomocą jednego tokena. OpenClaw używa **endpointu zgodnego z OpenAI** (tylko chat completions); do text-to-image, embeddings lub mowy używaj bezpośrednio [klientów HF inference](https://huggingface.co/docs/api-inference/quicktour).

- Dostawca: `huggingface`
- Uwierzytelnianie: `HUGGINGFACE_HUB_TOKEN` lub `HF_TOKEN` (token o ograniczonym zakresie z uprawnieniem **Make calls to Inference Providers**)
- API: kompatybilne z OpenAI (`https://router.huggingface.co/v1`)
- Rozliczenia: pojedynczy token HF; [cennik](https://huggingface.co/docs/inference-providers/pricing) zgodny ze stawkami dostawców, z darmowym limitem.

## Szybki start

1. Utwórz token o ograniczonym zakresie na stronie [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) z uprawnieniem **Make calls to Inference Providers**.
2. Uruchom onboarding i wybierz **Hugging Face** z listy dostawców, a następnie wprowadź swój klucz API, gdy pojawi się monit:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. W polu **Default Hugging Face model** wybierz model, którego chcesz używać (lista jest wczytywana z Inference API, gdy masz prawidłowy token; w przeciwnym razie wyświetlana jest wbudowana lista). Twój wybór zostanie zapisany jako model domyślny.
4. Możesz również ustawić lub zmienić model domyślny później w konfiguracji:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Przykład w trybie nieinteraktywnym

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Spowoduje to ustawienie `huggingface/deepseek-ai/DeepSeek-R1` jako modelu domyślnego.

## Uwaga dotycząca środowiska

Jeśli Gateway działa jako daemon (launchd/systemd), upewnij się, że `HUGGINGFACE_HUB_TOKEN` lub `HF_TOKEN`
jest dostępny dla tego procesu (na przykład w `~/.openclaw/.env` lub przez
`env.shellEnv`).

## Wykrywanie modeli i lista rozwijana podczas onboardingu

OpenClaw wykrywa modele, wywołując **bezpośrednio endpoint Inference**:

```bash
GET https://router.huggingface.co/v1/models
```

(Opcjonalnie: wyślij `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` lub `$HF_TOKEN`, aby uzyskać pełną listę; niektóre endpointy bez uwierzytelnienia zwracają tylko podzbiór.) Odpowiedź ma format OpenAI: `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Gdy skonfigurujesz klucz API Hugging Face (przez onboarding, `HUGGINGFACE_HUB_TOKEN` lub `HF_TOKEN`), OpenClaw używa tego zapytania GET do wykrywania dostępnych modeli chat-completion. Podczas **interaktywnego onboardingu**, po wprowadzeniu tokenu zobaczysz listę rozwijaną **Default Hugging Face model** wypełnioną na podstawie tej listy (lub wbudowanego katalogu, jeśli żądanie się nie powiedzie). W czasie działania (np. przy uruchamianiu Gateway), gdy klucz jest dostępny, OpenClaw ponownie wywołuje **GET** `https://router.huggingface.co/v1/models`, aby odświeżyć katalog. Lista jest łączona z wbudowanym katalogiem (zawierającym metadane, takie jak okno kontekstu i koszt). Jeśli żądanie się nie powiedzie lub klucz nie jest ustawiony, używany jest wyłącznie wbudowany katalog.

## Nazwy modeli i edytowalne opcje

- **Nazwa z API:** Wyświetlana nazwa modelu jest **uzupełniana na podstawie GET /v1/models**, gdy API zwraca `name`, `title` lub `display_name`; w przeciwnym razie jest wyprowadzana z identyfikatora modelu (np. `deepseek-ai/DeepSeek-R1` → „DeepSeek R1”).
- **Nadpisanie nazwy wyświetlanej:** Możesz ustawić własną etykietę dla każdego modelu w konfiguracji, aby w CLI i UI był wyświetlany zgodnie z Twoimi preferencjami:

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

- **Wybór dostawcy / polityki:** Dodaj sufiks do **identyfikatora modelu**, aby określić sposób wyboru backendu przez router:

  - **`:fastest`** — najwyższa przepustowość (wybór routera; wybór dostawcy jest **zablokowany** — brak interaktywnego wyboru backendu).
  - **`:cheapest`** — najniższy koszt za token wyjściowy (wybór routera; wybór dostawcy jest **zablokowany**).
  - **`:provider`** — wymuś konkretny backend (np. `:sambanova`, `:together`).

  Gdy wybierzesz **:cheapest** lub **:fastest** (np. z listy modeli podczas onboardingu), dostawca jest zablokowany: router decyduje na podstawie kosztu lub szybkości i nie jest wyświetlany opcjonalny krok „preferuj konkretny backend”. Możesz dodać je jako osobne wpisy w `models.providers.huggingface.models` lub ustawić `model.primary` z odpowiednim sufiksem. Możesz także ustawić domyślną kolejność w [ustawieniach Inference Provider](https://hf.co/settings/inference-providers) (brak sufiksu = użyj tej kolejności).

- **Scalanie konfiguracji:** Istniejące wpisy w `models.providers.huggingface.models` (np. w `models.json`) są zachowywane podczas scalania konfiguracji. Wszelkie niestandardowe `name`, `alias` lub opcje modelu, które tam ustawisz, zostaną zachowane.

## Identyfikatory modeli i przykłady konfiguracji

Odwołania do modeli używają formatu `huggingface/<org>/<model>` (identyfikatory w stylu Hub). Poniższa lista pochodzi z **GET** `https://router.huggingface.co/v1/models`; Twój katalog może zawierać więcej pozycji.

**Przykładowe identyfikatory (z punktu końcowego inferencji):**

| Model                                  | Ref (z prefiksem `huggingface/`) |
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

Możesz dodać `:fastest`, `:cheapest` lub `:provider` (np. `:together`, `:sambanova`) do identyfikatora modelu. Ustaw domyślną kolejność w [ustawieniach Inference Provider](https://hf.co/settings/inference-providers); zobacz [Inference Providers](https://huggingface.co/docs/inference-providers) oraz **GET** `https://router.huggingface.co/v1/models`, aby uzyskać pełną listę.

### Kompletne przykłady konfiguracji

**Główny DeepSeek R1 z zapasowym Qwen:**

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

**Qwen jako domyślny, z wariantami :cheapest i :fastest:**

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

**DeepSeek + Llama + GPT-OSS z aliasami:**

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

**Wymuś określony backend za pomocą :provider:**

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

**Wiele modeli Qwen i DeepSeek z sufiksami polityk:**

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
