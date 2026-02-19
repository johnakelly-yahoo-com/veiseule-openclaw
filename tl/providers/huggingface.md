---
summary: "Setup ng Hugging Face Inference (auth + pagpili ng model)"
read_when:
  - Gusto mong gamitin ang Hugging Face Inference sa OpenClaw
  - Kailangan mo ang HF token env var o CLI auth choice
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

Ang [Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) ay nag-aalok ng OpenAI-compatible chat completions sa pamamagitan ng iisang router API. Makakakuha ka ng access sa maraming modelo (DeepSeek, Llama, at iba pa) gamit ang iisang token. Ginagamit ng OpenClaw ang **OpenAI-compatible endpoint** (chat completions lamang); para sa text-to-image, embeddings, o speech, direktang gamitin ang [HF inference clients](https://huggingface.co/docs/api-inference/quicktour).

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN` (fine-grained token na may **Make calls to Inference Providers**)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: Iisang HF token; ang [pricing](https://huggingface.co/docs/inference-providers/pricing) ay sumusunod sa mga rate ng provider na may libreng tier.

## Mabilis na pagsisimula

1. Gumawa ng fine-grained token sa [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) na may pahintulot na **Make calls to Inference Providers**.
2. Patakbuhin ang onboarding at piliin ang **Hugging Face** sa provider dropdown, pagkatapos ay ilagay ang iyong API key kapag hiniling:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. Sa **Default Hugging Face model** dropdown, piliin ang model na gusto mo (ang listahan ay kino-load mula sa Inference API kapag may valid token ka; kung wala, built-in na listahan ang ipapakita). Ang pinili mo ay mase-save bilang default na model.
4. Maaari mo ring itakda o baguhin ang default na model mamaya sa config:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Halimbawa na non-interactive

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Itatakda nito ang `huggingface/deepseek-ai/DeepSeek-R1` bilang default na model.

## Paalala sa environment

Kung ang Gateway ay tumatakbo bilang daemon (launchd/systemd), siguraduhing ang `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN`
ay available sa prosesong iyon (halimbawa, sa `~/.openclaw/.env` o sa pamamagitan ng
`env.shellEnv`).

## Pagdiskubre ng model at onboarding dropdown

Dinidiskubre ng OpenClaw ang mga model sa pamamagitan ng direktang pagtawag sa **Inference endpoint**:

```bash
GET https://router.huggingface.co/v1/models
```

(Opsyonal: magpadala ng `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` o `$HF_TOKEN` para sa kumpletong listahan; may ilang endpoint na nagbabalik lamang ng subset kapag walang auth.) Ang tugon ay OpenAI-style `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Kapag nag-configure ka ng Hugging Face API key (sa pamamagitan ng onboarding, `HUGGINGFACE_HUB_TOKEN`, o `HF_TOKEN`), ginagamit ng OpenClaw ang GET na ito upang tuklasin ang mga available na chat-completion model. Sa panahon ng **interactive onboarding**, pagkatapos mong ilagay ang iyong token makikita mo ang **Default Hugging Face model** dropdown na pinupunan mula sa listahang iyon (o mula sa built-in catalog kung mag-fail ang request). Sa runtime (hal. sa startup ng Gateway), kapag may key, muling tinatawagan ng OpenClaw ang **GET** `https://router.huggingface.co/v1/models` upang i-refresh ang catalog. Ang listahan ay pinagsasama sa isang built-in catalog (para sa metadata tulad ng context window at cost). Kung mag-fail ang request o walang nakatakdang key, ang built-in catalog lamang ang gagamitin.

## Mga pangalan ng model at mga nae-edit na opsyon

- **Pangalan mula sa API:** Ang display name ng model ay **kinukuha mula sa GET /v1/models** kapag ibinalik ng API ang `name`, `title`, o `display_name`; kung wala, ito ay hinango mula sa model id (hal. `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **I-override ang display name:** Maaari kang magtakda ng custom na label kada model sa config upang lumabas ito ayon sa gusto mo sa CLI at UI:

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

- **Pagpili ng Provider / policy:** Magdagdag ng suffix sa **model id** upang piliin kung paano pipiliin ng router ang backend:

  - **`:fastest`** — pinakamataas na throughput (ang router ang pipili; ang pagpili ng provider ay **naka-lock** — walang interactive backend picker).
  - **`:cheapest`** — pinakamababang gastos bawat output token (ang router ang pipili; ang pagpili ng provider ay **naka-lock**).
  - **`:provider`** — pilitin ang isang partikular na backend (hal. `:sambanova`, `:together`).

  Kapag pinili mo ang **:cheapest** o **:fastest** (hal. sa onboarding model dropdown), naka-lock ang provider: ang router ang magpapasya batay sa cost o bilis at walang ipapakitang opsyonal na “prefer specific backend” na hakbang. Maaari mong idagdag ang mga ito bilang hiwalay na entries sa `models.providers.huggingface.models` o itakda ang `model.primary` na may suffix. Maaari mo ring itakda ang iyong default na order sa [Inference Provider settings](https://hf.co/settings/inference-providers) (walang suffix = gamitin ang order na iyon).

- **Config merge:** Ang mga umiiral na entries sa `models.providers.huggingface.models` (hal. sa `models.json`) ay pinananatili kapag pinagsama ang config. Kaya anumang custom na `name`, `alias`, o mga opsyon ng model na itinakda mo roon ay mapapanatili.

## Mga Model ID at mga halimbawa ng configuration

Ang mga model ref ay gumagamit ng anyong `huggingface/<org>/<model>` (Hub-style IDs). Ang listahan sa ibaba ay mula sa **GET** `https://router.huggingface.co/v1/models`; maaaring mas marami pa ang nasa iyong catalog.

**Mga Halimbawang ID (mula sa inference endpoint):**

| Model                                  | Ref (lagyan ng prefix na `huggingface/`) |
| -------------------------------------- | ----------------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                                   |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                                 |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                             |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                                  |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                            |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                         |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                          |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                       |
| GLM 4.7                | `zai-org/GLM-4.7`                                           |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                      |

Maaari mong idagdag ang `:fastest`, `:cheapest`, o `:provider` (hal. `:together`, `:sambanova`) sa model id. Itakda ang iyong default na pagkakasunod-sunod sa [Inference Provider settings](https://hf.co/settings/inference-providers); tingnan ang [Inference Providers](https://huggingface.co/docs/inference-providers) at **GET** `https://router.huggingface.co/v1/models` para sa kumpletong listahan.

### Mga kumpletong halimbawa ng configuration

**Pangunahing DeepSeek R1 na may Qwen bilang fallback:**

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

**Qwen bilang default, na may mga variant na :cheapest at :fastest:**

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

**DeepSeek + Llama + GPT-OSS na may mga alias:**

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

**Piliting gumamit ng partikular na backend gamit ang :provider:**

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

**Maramihang Qwen at DeepSeek models na may mga policy suffix:**

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
