---
summary: "Konfiguracja Together AI (uwierzytelnianie + wybór modelu)"
read_when:
  - Chcesz używać Together AI z OpenClaw
  - Potrzebujesz zmiennej środowiskowej z kluczem API lub wyboru uwierzytelniania w CLI
---

# Together AI

[Together AI](https://together.ai) zapewnia dostęp do wiodących modeli open-source, w tym Llama, DeepSeek, Kimi i innych, poprzez ujednolicone API.

- Provider: `together`
- Auth: `TOGETHER_API_KEY`
- API: kompatybilne z OpenAI

## Szybki start

1. Ustaw klucz API (zalecane: zapisz go dla Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Ustaw domyślny model:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Przykład bez trybu interaktywnego

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Spowoduje to ustawienie `together/moonshotai/Kimi-K2.5` jako domyślnego modelu.

## Uwaga dotycząca środowiska

Jeśli Gateway działa jako demon (launchd/systemd), upewnij się, że `TOGETHER_API_KEY`
jest dostępny dla tego procesu (na przykład w `~/.clawdbot/.env` lub przez
`env.shellEnv`).

## Dostępne modele

Together AI zapewnia dostęp do wielu popularnych modeli open-source:

- **GLM 4.7 Fp8** - Domyślny model z oknem kontekstu 200K
- **Llama 3.3 70B Instruct Turbo** - Szybkie i wydajne wykonywanie poleceń
- **Llama 4 Scout** - Model wizyjny z rozumieniem obrazów
- **Llama 4 Maverick** - Zaawansowany model wizyjny i rozumowania
- **DeepSeek V3.1** - Wydajny model do kodowania i rozumowania
- **DeepSeek R1** - Zaawansowany model rozumowania
- **Kimi K2 Instruct** - Model o wysokiej wydajności z oknem kontekstu 262K

Wszystkie modele obsługują standardowe chat completions i są kompatybilne z OpenAI API.
