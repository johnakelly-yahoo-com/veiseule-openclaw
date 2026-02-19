---
summary: "Hugging Face Inference sozlamalari (auth + model tanlash)"
read_when:
  - Siz OpenClaw bilan Hugging Face Inference’dan foydalanmoqchisiz
  - Sizga HF token env o‘zgaruvchisi yoki CLI auth tanlovi kerak
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) yagona router API orqali OpenAI-compatible chat completions xizmatini taqdim etadi. Bitta token orqali ko‘plab modellardan (DeepSeek, Llama va boshqalar) foydalanish imkoniga ega bo‘lasiz. OpenClaw **OpenAI-compatible endpoint** (faqat chat completions) dan foydalanadi; text-to-image, embeddings yoki speech uchun [HF inference clients](https://huggingface.co/docs/api-inference/quicktour) dan to‘g‘ridan-to‘g‘ri foydalaning.

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` yoki `HF_TOKEN` (**Make calls to Inference Providers** ruxsatiga ega fine-grained token)
- API: OpenAI-compatible (`https://router.huggingface.co/v1`)
- Billing: Bitta HF token; [narxlash](https://huggingface.co/docs/inference-providers/pricing) provayder tariflariga asoslanadi va bepul darajani o‘z ichiga oladi.

## Tez boshlash

1. **Make calls to Inference Providers** ruxsati bilan fine-grained tokenni [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) sahifasida yarating.
2. Onboarding’ni ishga tushiring va provayder ochiladigan ro‘yxatidan **Hugging Face** ni tanlang, so‘ng so‘ralganda API kalitingizni kiriting:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** ochiladigan ro‘yxatidan kerakli modelni tanlang (ro‘yxat sizda amal qiluvchi token bo‘lsa Inference API’dan yuklanadi; aks holda ichki ro‘yxat ko‘rsatiladi). Tanlovingiz standart (default) model sifatida saqlanadi.
4. Standart modelni keyinroq config’da ham o‘rnatishingiz yoki o‘zgartirishingiz mumkin:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Interaktiv bo‘lmagan misol

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Bu `huggingface/deepseek-ai/DeepSeek-R1` modelini standart model sifatida o‘rnatadi.

## Muhit haqida eslatma

Agar Gateway daemon sifatida (launchd/systemd) ishlayotgan bo‘lsa, `HUGGINGFACE_HUB_TOKEN` yoki `HF_TOKEN` o‘sha jarayon uchun mavjud ekanligiga ishonch hosil qiling
(masalan, `~/.openclaw/.env` ichida yoki
`env.shellEnv` orqali).

## Modelni aniqlash va onboarding ochiladigan ro‘yxati

OpenClaw modellarni **Inference endpoint’ga to‘g‘ridan-to‘g‘ri** murojaat qilib aniqlaydi:

```bash
GET https://router.huggingface.co/v1/models
```

(Ixtiyoriy: to‘liq ro‘yxat uchun `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` yoki `$HF_TOKEN` yuboring; ba’zi endpoint’lar auth’siz qisqartirilgan ro‘yxatni qaytaradi.) Javob OpenAI uslubida bo‘ladi `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Hugging Face API kalitini sozlaganingizda (onboarding, `HUGGINGFACE_HUB_TOKEN` yoki `HF_TOKEN` orqali), OpenClaw mavjud chat-completion modellarini aniqlash uchun ushbu GET so‘rovdan foydalanadi. **Interaktiv onboarding** jarayonida tokenni kiritganingizdan so‘ng, sizga shu ro‘yxatdan (yoki so‘rov muvaffaqiyatsiz bo‘lsa ichki katalogdan) to‘ldirilgan **Default Hugging Face model** ochiladigan ro‘yxati ko‘rsatiladi. Ish jarayonida (masalan, Gateway ishga tushganda), kalit mavjud bo‘lsa, OpenClaw katalogni yangilash uchun yana **GET** `https://router.huggingface.co/v1/models` so‘rovini yuboradi. Ro‘yxat ichki katalog bilan (masalan, kontekst oynasi va narx kabi metadata uchun) birlashtiriladi. Agar so‘rov muvaffaqiyatsiz bo‘lsa yoki kalit o‘rnatilmagan bo‘lsa, faqat ichki katalogdan foydalaniladi.

## Model nomlari va tahrirlanadigan parametrlar

- **API’dan olingan nom:** Modelning ko‘rsatiladigan nomi API `name`, `title` yoki `display_name` ni qaytarganda **GET /v1/models dan olinadi**; aks holda u model id’dan hosil qilinadi (masalan, `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Ko‘rsatiladigan nomni o‘zgartirish:** Har bir model uchun config’da maxsus yorliq (label) o‘rnatishingiz mumkin, shunda u CLI va UI’da siz xohlagan ko‘rinishda aks etadi:

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

- **Provider / policy tanlash:** Router backendni qanday tanlashini belgilash uchun **model id** ga qo‘shimcha suffiks qo‘shing:

  - **`:fastest`** — eng yuqori o‘tkazuvchanlik (router tanlaydi; provider tanlovi **bloklangan** — interaktiv backend tanlagichi yo‘q).
  - **`:cheapest`** — har bir chiqish tokeni uchun eng past narx (router tanlaydi; provider tanlovi **bloklangan**).
  - **`:provider`** — ma’lum bir backendni majburan tanlash (masalan, `:sambanova`, `:together`).

  **:cheapest** yoki **:fastest** ni tanlaganingizda (masalan, onboarding model ro‘yxatida), provider bloklanadi: router narx yoki tezlik asosida qaror qiladi va qo‘shimcha “ma’lum backendni afzal ko‘rish” bosqichi ko‘rsatilmaydi. Bularni `models.providers.huggingface.models` ichiga alohida yozuvlar sifatida qo‘shishingiz yoki `model.primary` ni suffiks bilan belgilashingiz mumkin. Shuningdek, standart tartibni [Inference Provider settings](https://hf.co/settings/inference-providers) bo‘limida sozlashingiz mumkin (suffikssiz = o‘sha tartibdan foydalaniladi).

- **Config merge:** Konfiguratsiya birlashtirilganda `models.providers.huggingface.models` dagi mavjud yozuvlar (masalan, `models.json` da) saqlab qolinadi. Shuning uchun u yerda o‘rnatgan har qanday maxsus `name`, `alias` yoki model opsiyalari saqlanib qoladi.

## Model ID va konfiguratsiya misollari

Model havolalari `huggingface/<org>/<model>` (Hub uslubidagi ID) ko‘rinishida bo‘ladi. Quyidagi ro‘yxat **GET** `https://router.huggingface.co/v1/models` dan olingan; sizning katalogingizda bundan ko‘proq modelllar bo‘lishi mumkin.

**Misol ID lar (inference endpointdan):**

| Model                                  | Ref (`huggingface/` bilan boshlanadi) |
| -------------------------------------- | -------------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                                |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                              |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                          |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                               |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                         |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                      |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                       |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                    |
| GLM 4.7                | `zai-org/GLM-4.7`                                        |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                                   |

Model id ga `:fastest`, `:cheapest` yoki `:provider` (masalan, `:together`, `:sambanova`) qo‘shishingiz mumkin. Standart tartibni [Inference Provider settings](https://hf.co/settings/inference-providers) bo‘limida sozlang; to‘liq ro‘yxat uchun [Inference Providers](https://huggingface.co/docs/inference-providers) va **GET** `https://router.huggingface.co/v1/models` ga qarang.

### To‘liq konfiguratsiya misollari

**Asosiy DeepSeek R1, Qwen fallback bilan:**

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

**Qwen sukut bo‘yicha model sifatida, :cheapest va :fastest variantlari bilan:**

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

**DeepSeek + Llama + GPT-OSS aliaslar bilan:**

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

**:provider yordamida aniq backendni majburan tanlash:**

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

**Policy suffikslari bilan bir nechta Qwen va DeepSeek modellari:**

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
