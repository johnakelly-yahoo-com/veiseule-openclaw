---
summary: "Referência da CLI para `openclaw onboard` (assistente interativo de integração inicial)"
read_when:
  - Você quer configuração guiada para gateway, workspace, autenticação, canais e Skills
title: "onboard"
---

# `openclaw onboard`

Assistente interativo de integração inicial (configuração local ou remota do Gateway).

## Guias relacionados

- Hub de integração inicial da CLI: [Onboarding Wizard (CLI)](/start/wizard)
- Visão geral do onboarding: [Onboarding Overview](/start/onboarding-overview)
- Automação da CLI: [CLI Automation](/start/wizard-cli-automation)
- Referência de integração inicial da CLI: [CLI Onboarding Reference](/start/wizard-cli-reference)
- Integração inicial no macOS: [Onboarding (macOS App)](/start/onboarding)

## Exemplos

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

Provedor personalizado não interativo:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key` é opcional no modo não interativo. Se omitido, o onboarding verifica `CUSTOM_API_KEY`.

Opções de endpoint Z.AI não interativas:

Observação: `--auth-choice zai-api-key` agora detecta automaticamente o melhor endpoint Z.AI para sua chave (prefere a API geral com `zai/glm-5`).
Se você quiser especificamente os endpoints do GLM Coding Plan, escolha `zai-coding-global` ou `zai-coding-cn`.

```bash
# Seleção de endpoint sem prompt
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# Outras opções de endpoint Z.AI:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

Notas do fluxo:

- `quickstart`: prompts mínimos, gera automaticamente um token do gateway.
- `manual`: prompts completos para porta/bind/autenticação (alias de `advanced`).
- Primeiro chat mais rápido: `openclaw dashboard` (UI de Controle, sem configuração de canal).
- Provedor personalizado: conecte qualquer endpoint compatível com OpenAI ou Anthropic,
  incluindo provedores hospedados não listados. Use Unknown para detectar automaticamente.

## Comandos comuns de acompanhamento

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` não implica modo não interativo. Use `--non-interactive` para scripts.
</Note>
