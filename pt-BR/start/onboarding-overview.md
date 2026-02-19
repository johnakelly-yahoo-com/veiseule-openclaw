---
summary: "Visão geral das opções e fluxos de onboarding do OpenClaw"
read_when:
  - Escolhendo um caminho de onboarding
  - Configurando um novo ambiente
title: "Visão geral do onboarding"
sidebarTitle: "Visão geral do onboarding"
---

# Visão geral do onboarding

O OpenClaw oferece múltiplos caminhos de onboarding, dependendo de onde o Gateway é executado
e de como você prefere configurar os providers.

## Escolha seu caminho de onboarding

- **Assistente CLI** para macOS, Linux e Windows (via WSL2).
- **App macOS** para uma primeira execução guiada em Macs com Apple silicon ou Intel.

## Assistente de onboarding via CLI

Execute o assistente em um terminal:

```bash
openclaw onboard
```

Use o assistente CLI quando quiser controle total do Gateway, workspace,
canais e skills. Documentação:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## Onboarding pelo app macOS

Use o app OpenClaw quando quiser uma configuração totalmente guiada no macOS. Documentação:

- [Onboarding (macOS App)](/start/onboarding)

## Provedor Personalizado

Se você precisa de um endpoint que não está listado, incluindo provedores hospedados que expõem APIs padrão OpenAI ou Anthropic, escolha **Custom Provider** no assistente da CLI. Você será solicitado a:

- Escolher compatível com OpenAI, compatível com Anthropic ou **Unknown** (detecção automática).
- Inserir uma URL base e a chave de API (se exigida pelo provedor).
- Fornecer um ID de modelo e um alias opcional.
- Escolher um Endpoint ID para que vários endpoints personalizados possam coexistir.

Para etapas detalhadas, siga a documentação de onboarding da CLI acima.

