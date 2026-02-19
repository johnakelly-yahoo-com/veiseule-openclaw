---
summary: "Together AI-opsætning (auth + modelvalg)"
read_when:
  - Du vil bruge Together AI med OpenClaw
  - Du skal bruge API-nøglens miljøvariabel eller CLI auth-valg
---

# Together AI

[Together AI](https://together.ai) giver adgang til førende open source-modeller, herunder Llama, DeepSeek, Kimi m.fl., via en samlet API.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-kompatibel

## Hurtig start

1. Sæt API-nøglen (anbefalet: gem den til Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Angiv en standardmodel:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Ikke-interaktivt eksempel

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Dette vil sætte `together/moonshotai/Kimi-K2.5` som standardmodel.

## Miljøbemærkning

Hvis Gateway kører som en daemon (launchd/systemd), skal du sikre dig, at `TOGETHER_API_KEY` er tilgængelig for den proces (for eksempel i `~/.clawdbot/.env` eller via `env.shellEnv`).

## Tilgængelige modeller

Together AI giver adgang til mange populære open-source-modeller:

- **GLM 4.7 Fp8** - Standardmodel med 200K kontekstvindue
- **Llama 3.3 70B Instruct Turbo** - Hurtig og effektiv til at følge instruktioner
- **Llama 4 Scout** - Vision-model med billedforståelse
- **Llama 4 Maverick** - Avanceret vision og ræsonnering
- **DeepSeek V3.1** - Kraftfuld model til kodning og ræsonnering
- **DeepSeek R1** - Avanceret ræsonneringsmodel
- **Kimi K2 Instruct** - Højtydende model med 262K kontekstvindue

Alle modeller understøtter standard chat completions og er kompatible med OpenAI API.
