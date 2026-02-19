---
summary: "Together AI-installation (auth + modellval)"
read_when:
  - Du vill använda Together AI med OpenClaw
  - Du behöver miljövariabeln för API-nyckeln eller CLI-auth-valet
---

# Together AI

[Together AI](https://together.ai) ger åtkomst till ledande open source-modeller inklusive Llama, DeepSeek, Kimi med flera via ett enhetligt API.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-kompatibelt

## Snabbstart

1. Ange API-nyckeln (rekommenderas: spara den för Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Ange en standardmodell:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Icke-interaktivt exempel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Detta kommer att ange `together/moonshotai/Kimi-K2.5` som standardmodell.

## Miljöanmärkning

Om Gateway körs som en daemon (launchd/systemd), se till att `TOGETHER_API_KEY`
är tillgänglig för den processen (till exempel i `~/.clawdbot/.env` eller via
`env.shellEnv`).

## Tillgängliga modeller

Together AI ger tillgång till många populära open-source-modeller:

- **GLM 4.7 Fp8** - Standardmodell med 200K kontextfönster
- **Llama 3.3 70B Instruct Turbo** - Snabb och effektiv på att följa instruktioner
- **Llama 4 Scout** - Visionsmodell med bildförståelse
- **Llama 4 Maverick** - Avancerad vision och resonemang
- **DeepSeek V3.1** - Kraftfull modell för kodning och resonemang
- **DeepSeek R1** - Avancerad resonemangsmodell
- **Kimi K2 Instruct** - Högpresterande modell med 262K kontextfönster

Alla modeller stöder standard chat completions och är kompatibla med OpenAI API.

