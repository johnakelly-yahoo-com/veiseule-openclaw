---
read_when:
  - Configuração inicial do zero
  - Quer o caminho mais rápido para um chat funcional
summary: Instale o OpenClaw e execute seu primeiro chat em poucos minutos.
title: Introdução
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Introdução

Objetivo: criar seu primeiro chat funcional do zero com uma configuração mínima.

<Info>
Forma mais rápida de conversar: abra a Control UI (nenhuma configuração de canal é necessária). Execute `openclaw dashboard` para conversar no navegador ou
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Host do Gateway</Tooltip>abra `http://127.0.0.1:18789/`.
Documentação: [Dashboard](/web/dashboard) e [Control UI](/web/control-ui).
</Info>

## Pré-requisitos

- Node 22 ou superior

<Tip>
Se não tiver certeza, verifique sua versão do Node com `node --version`.
</Tip>

## Configuração rápida (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    Outros métodos de instalação e requisitos: [Instalação](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    O assistente configura autenticação, o Gateway e canais opcionais.
    Consulte [Assistente de Onboarding](/start/wizard) para mais detalhes.
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Se você instalou o serviço, ele já deve estar em execução:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Se a Control UI carregar, o Gateway está pronto para uso.
</Check>

## Verificações opcionais e recursos adicionais

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Útil para testes rápidos e solução de problemas.

    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    É necessário ter um canal configurado.

    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Saiba mais

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Referência completa do assistente CLI e opções avançadas.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Fluxo de primeira execução do app macOS.
  
</Card>
</Columns>

## Estado após a conclusão

- Gateway em execução
- Autenticação configurada
- Acesso à Control UI ou canal conectado

## Próximos passos

- Segurança e aprovação de DM: [Pareamento](/channels/pairing)
- Conecte mais canais: [Canais](/channels)
- Workflows avançados e build a partir do código-fonte: [Configuração](/start/setup)
