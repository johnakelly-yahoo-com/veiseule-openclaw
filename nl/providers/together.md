---
summary: "Together AI-installatie (auth + modelselectie)"
read_when:
  - Je wilt Together AI gebruiken met OpenClaw
  - Je hebt de API-sleutel-omgevingsvariabele of de CLI-auth-keuze nodig
---

# Together AI

De [Together AI](https://together.ai) biedt via een uniforme API toegang tot toonaangevende open-sourcemodellen, waaronder Llama, DeepSeek, Kimi en meer.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: OpenAI-compatibel

## Snelstart

1. Stel de API-sleutel in (aanbevolen: sla deze op voor de Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Stel een standaardmodel in:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Niet-interactief voorbeeld

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Dit stelt `together/moonshotai/Kimi-K2.5` in als het standaardmodel.

## Omgevingsopmerking

Als de Gateway als een daemon (launchd/systemd) draait, zorg er dan voor dat `TOGETHER_API_KEY`
beschikbaar is voor dat proces (bijvoorbeeld in `~/.clawdbot/.env` of via
`env.shellEnv`).

## Beschikbare modellen

Together AI biedt toegang tot veel populaire open-sourcemodellen:

- **GLM 4.7 Fp8** - Standaardmodel met een contextvenster van 200K
- **Llama 3.3 70B Instruct Turbo** - Snel en efficiënt in het opvolgen van instructies
- **Llama 4 Scout** - Visueel model met beeldbegrip
- **Llama 4 Maverick** - Geavanceerd zicht en redeneervermogen
- **DeepSeek V3.1** - Krachtig model voor coderen en redeneren
- **DeepSeek R1** - Geavanceerd redeneermodel
- **Kimi K2 Instruct** - High-performance model met een contextvenster van 262K

Alle modellen ondersteunen standaard chatcompletions en zijn compatibel met de OpenAI API.
