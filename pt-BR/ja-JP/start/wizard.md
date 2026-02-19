---
read_when:
  - Ao executar ou configurar o assistente de onboarding
  - Ao configurar uma nova máquina
sidebarTitle: Wizard (CLI)
summary: "Assistente de onboarding do CLI: configuração interativa do Gateway, workspace, canais e Skills"
title: Assistente de onboarding (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# Assistente de onboarding (CLI)

O assistente de onboarding do CLI é o caminho recomendado para configurar o OpenClaw no macOS, Linux e Windows (via WSL2). Ele configura um Gateway local ou uma conexão com Gateway remoto, além das definições padrão do workspace, canais e Skills.

```bash
openclaw onboard
```

<Info>
A maneira mais rápida de iniciar seu primeiro chat: abra a Control UI (não é necessário configurar canais). Execute `openclaw dashboard` para conversar no navegador. Documentação: [Dashboard](/web/dashboard).
</Info>

## Início rápido vs Configuração detalhada

O assistente começa permitindo escolher entre **Início rápido** (configurações padrão) ou **Configuração detalhada** (controle total).

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - Gateway local em loopback
    - Workspace existente ou workspace padrão
    - Porta do Gateway `18789`
    - Token de autenticação do Gateway gerado automaticamente (mesmo em loopback)
    - Exposição via Tailscale desativada
    - DMs do Telegram e WhatsApp permitidas por padrão (pode ser solicitado o número de telefone)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - Exibe o fluxo completo de prompts para modo, workspace, Gateway, canais, daemons e Skills
  
</Tab>
</Tabs>

## Detalhes do onboarding via CLI

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    Descrição completa dos fluxos local e remoto, matriz de autenticação e modelos, saída de configuração, RPC do assistente e comportamento do signal-cli.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    Exemplos de receitas de onboarding não interativo e `agents add` automatizado.
  
</Card>
</Columns>

## Comandos de acompanhamento mais usados

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json` não significa modo não interativo. Em scripts, use `--non-interactive`.
</Note>

<Tip>
Recomendado: configure uma chave da Brave Search API para que os agentes possam usar `web_search` (`web_fetch` funciona sem chave). A forma mais simples: execute `openclaw configure --section web` para salvar `tools.web.search.apiKey`. Documentação: [Ferramentas Web](/tools/web).
</Tip>

## Documentação relacionada

- Referência de comandos CLI: [`openclaw onboard`](/cli/onboard)
- Onboarding do app macOS: [Onboarding](/start/onboarding)
- Passos para a primeira execução do agente: [Bootstrap do agente](/start/bootstrapping)
