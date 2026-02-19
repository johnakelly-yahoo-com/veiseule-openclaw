---
summary: "Together AI setup (auth + pagpili ng model)"
read_when:
  - Gusto mong gamitin ang Together AI sa OpenClaw
  - Kailangan mo ang API key env var o CLI auth choice
---

# Together AI

Ang [Together AI](https://together.ai) ay nagbibigay ng access sa mga nangungunang open-source model kabilang ang Llama, DeepSeek, Kimi, at iba pa sa pamamagitan ng isang unified API.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-compatible

## Mabilis na pagsisimula

1. Itakda ang API key (inirerekomenda: i-store ito para sa Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Magtakda ng default na model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Halimbawa na non-interactive

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Itatakda nito ang `together/moonshotai/Kimi-K2.5` bilang default na model.

## Tala sa environment

Kung ang Gateway ay tumatakbo bilang daemon (launchd/systemd), tiyaking ang `TOGETHER_API_KEY`
ay available sa prosesong iyon (halimbawa, sa `~/.clawdbot/.env` o sa pamamagitan ng
`env.shellEnv`).

## Mga available na model

Nagbibigay ang Together AI ng access sa maraming sikat na open-source na model:

- **GLM 4.7 Fp8** - Default na model na may 200K context window
- **Llama 3.3 70B Instruct Turbo** - Mabilis at episyenteng pagsunod sa mga instruction
- **Llama 4 Scout** - Vision model na may kakayahang umunawa ng imahe
- **Llama 4 Maverick** - Advanced na vision at reasoning
- **DeepSeek V3.1** - Makapangyarihang model para sa coding at reasoning
- **DeepSeek R1** - Advanced na reasoning model
- **Kimi K2 Instruct** - High-performance na model na may 262K context window

Lahat ng model ay sumusuporta sa standard chat completions at compatible sa OpenAI API.
