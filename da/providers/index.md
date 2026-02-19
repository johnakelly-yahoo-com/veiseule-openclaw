---
summary: "Modeludbydere (LLM'er) understøttet af OpenClaw"
read_when:
  - Du vil vælge en modeludbyder
  - Du har brug for et hurtigt overblik over understøttede LLM-backends
title: "Modeludbydere"
---

# Modeludbydere

OpenClaw kan bruge mange LLM udbydere. Vælg en udbyder, autentisk, og angiv derefter standardmodellen
som `udbyder/model`.

Leder du efter chat channel docs (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/etc.)? Se [Channels](/channels).

## Højdepunkt: Venice (Venice AI)

Venice er vores anbefalede Venice AI-opsætning til privatlivs-først-inferens med mulighed for at bruge Opus til krævende opgaver.

- Standard: `venice/llama-3.3-70b`
- Bedst samlet set: `venice/claude-opus-45` (Opus er fortsat den stærkeste)

Se [Venice AI](/providers/venice).

## Hurtig start

1. Autentificér med udbyderen (normalt via `openclaw onboard`).
2. Sæt standardmodellen:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Udbyderdokumentation

- [OpenAI (API + Codex)](/providers/openai)
- [Anthropic (API + Claude Code CLI)](/providers/anthropic)
- [Qwen (OAuth)](/providers/qwen)
- [OpenRouter](/providers/openrouter)
- [LiteLLM (unified gateway)](/providers/litellm)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Together AI](/providers/together)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
- [OpenCode Zen](/providers/opencode)
- [Amazon Bedrock](/providers/bedrock)
- [Z.AI](/providers/zai)
- [Xiaomi](/providers/xiaomi)
- [Venice (Venice AI, privatlivsfokuseret)](/providers/venice)
- [MiniMax](/providers/minimax)
- [Venice (Venice AI, privatlivsfokuseret)](/providers/venice)
- [Hugging Face (Inference)](/providers/huggingface)
- [Ollama (lokale modeller)](/providers/ollama)
- [vLLM (local models)](/providers/vllm)
- [Qianfan](/providers/qianfan)
- [NVIDIA](/providers/nvidia)

## Transskriptionsudbydere

- [Deepgram (lydtransskription)](/providers/deepgram)

## Community-værktøjer

- [Claude Max API Proxy](/providers/claude-max-api-proxy) – Brug Claude Max/Pro-abonnement som et OpenAI-kompatibelt API-endpoint

For hele udbyderkatalog (xAI, Groq, Mistral, osv.) og avanceret konfiguration,
se [Modeludbydere](/concepts/model-providers).

