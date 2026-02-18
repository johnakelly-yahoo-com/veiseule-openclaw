---
title: "OpenAI"
---

# OpenAI

OpenAI GPT modellari uchun dasturchilar APIlarini taqdim etadi. Codex obuna uchun **ChatGPT orqali tizimga kirish**ni qo‘llab-quvvatlaydi
access or **API key** sign-in for usage-based access. Codex cloud requires ChatGPT sign-in.

## Variant A: OpenAI API kaliti (OpenAI Platform)

**Eng mos:** to‘g‘ridan-to‘g‘ri API kirishi va foydalanishga asoslangan to‘lov.
Get your API key from the OpenAI dashboard.

### CLI sozlamasi

```bash
openclaw onboard --auth-choice openai-api-key
# or non-interactive
openclaw onboard --openai-api-key "$OPENAI_API_KEY"
```

### Konfiguratsiya namunasi

```json5
{
  env: { OPENAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "openai/gpt-5.1-codex" } } },
}
```

## Variant B: OpenAI Code (Codex) obunasi

**Eng mos:** API kaliti o‘rniga ChatGPT/Codex obuna kirishidan foydalanish.
Codex cloud requires ChatGPT sign-in, while the Codex CLI supports ChatGPT or API key sign-in.

### CLI setup (Codex OAuth)

```bash
# Run Codex OAuth in the wizard
openclaw onboard --auth-choice openai-codex

# Or run OAuth directly
openclaw models auth login --provider openai-codex
```

### 1. Konfiguratsiya parchasi (Codex obunasi)

```json5
{
  agents: { defaults: { model: { primary: "openai-codex/gpt-5.3-codex" } } },
}
```

## 3. Eslatmalar

- 4. Model havolalari doimo `provider/model` formatidan foydalanadi (qarang: [/concepts/models](/concepts/models)).
- 5. Autentifikatsiya tafsilotlari va qayta foydalanish qoidalari [/concepts/oauth](/concepts/oauth) sahifasida.


