---
title: "Mga Model Provider"
---

# Mga Model Provider

Maaaring gumamit ang OpenClaw ng maraming LLM provider. Pumili ng provider, mag-authenticate, pagkatapos ay itakda ang
default model as `provider/model`.

Naghahanap ng mga dokumento para sa chat channel (WhatsApp/Telegram/Discord/Slack/Mattermost (plugin)/atbp.)? Tingnan ang [Channels](/channels).

## Tampok: Venice (Venice AI)

Ang Venice ang aming inirerekomendang Venice AI setup para sa privacy-first inference, na may opsyong gumamit ng Opus para sa mahihirap na gawain.

- Default: `venice/llama-3.3-70b`
- Pinakamahusay sa pangkalahatan: `venice/claude-opus-45` (nanatiling pinakamalakas ang Opus)

Tingnan ang [Venice AI](/providers/venice).

## Mabilis na pagsisimula

1. Mag-authenticate sa provider (karaniwan sa pamamagitan ng `openclaw onboard`).
2. Itakda ang default na model:

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## Mga doc ng provider

- [OpenAI (API + Codex)](/providers/openai)
- [Anthropic (API + Claude Code CLI)](/providers/anthropic)
- [Qwen (OAuth)](/providers/qwen)
- [OpenRouter](/providers/openrouter)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
- [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
- [OpenCode Zen](/providers/opencode)
- [Amazon Bedrock](/providers/bedrock)
- [Z.AI](/providers/zai)
- [Xiaomi](/providers/xiaomi)
- [GLM models](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venice (Venice AI, nakatuon sa privacy)](/providers/venice)
- [Ollama (mga lokal na model)](/providers/ollama)
- [Qianfan](/providers/qianfan)

## Mga transcription provider

- [Deepgram (audio transcription)](/providers/deepgram)

## Mga community tool

- [Claude Max API Proxy](/providers/claude-max-api-proxy) - Gamitin ang Claude Max/Pro subscription bilang OpenAI-compatible na API endpoint

For the full provider catalog (xAI, Groq, Mistral, etc.) and advanced configuration,
see [Model providers](/concepts/model-providers).
