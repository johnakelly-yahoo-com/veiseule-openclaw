---
summary: "Configuração do Together AI (auth + seleção de modelo)"
read_when:
  - Você deseja usar Together AI com OpenClaw
  - Você precisa da variável de ambiente da chave de API ou da opção de autenticação na CLI
---

# Together AI

O [Together AI](https://together.ai) fornece acesso aos principais modelos open-source, incluindo Llama, DeepSeek, Kimi e outros, por meio de uma API unificada.

- Provedor: `together`
- Auth: `TOGETHER_API_KEY`
- API: compatível com OpenAI

## Início rápido

1. Defina a chave de API (recomendado: armazená-la para o Gateway):

```bash
openclaw onboard --auth-choice together-api-key
```

2. Defina um modelo padrão:

```json5
{
  agents: {
    defaults: {
      model: { primary: "together/moonshotai/Kimi-K2.5" },
    },
  },
}
```

## Exemplo não interativo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice together-api-key \
  --together-api-key "$TOGETHER_API_KEY"
```

Isso definirá `together/moonshotai/Kimi-K2.5` como o modelo padrão.

## Observação sobre o ambiente

Se o Gateway estiver em execução como um daemon (launchd/systemd), certifique-se de que `TOGETHER_API_KEY`
esteja disponível para esse processo (por exemplo, em `~/.clawdbot/.env` ou via
`env.shellEnv`).

## Modelos disponíveis

Together AI fornece acesso a muitos modelos open-source populares:

- **GLM 4.7 Fp8** - Modelo padrão com janela de contexto de 200K
- **Llama 3.3 70B Instruct Turbo** - Rápido e eficiente no seguimento de instruções
- **Llama 4 Scout** - Modelo de visão com compreensão de imagens
- **Llama 4 Maverick** - Visão e raciocínio avançados
- **DeepSeek V3.1** - Modelo poderoso para código e raciocínio
- **DeepSeek R1** - Modelo avançado de raciocínio
- **Kimi K2 Instruct** - Modelo de alto desempenho com janela de contexto de 262K

Todos os modelos oferecem suporte a chat completions padrão e são compatíveis com a API da OpenAI.

