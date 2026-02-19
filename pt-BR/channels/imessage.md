---
summary: "Suporte legado ao iMessage via imsg (JSON-RPC sobre stdio). Novas configurações devem usar o BlueBubbles."
read_when:
  - Configurando suporte ao iMessage
  - Depurando envio/recebimento do iMessage
title: "iMessage"
---

# iMessage (legado: imsg)

<Warning>
**Recomendado:** Use [BlueBubbles](/channels/bluebubbles) para novas configurações do iMessage.

O canal `imsg` é uma integração legada via CLI externa e pode ser removido em uma versão futura. 
</Warning>

Status: integração legada via CLI externa. O Gateway inicia `imsg rpc` (JSON-RPC sobre stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Caminho preferido do iMessage para novas configurações.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">O pareamento é a troca de tokens padrão para DMs do iMessage.
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Referência completa dos campos do iMessage.
  
</Card>
</CardGroup>

## Configuração (caminho rápido)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="Configurar OpenClaw">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Iniciar gateway">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Aprovar o primeiro pareamento por DM (dmPolicy padrão)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            As solicitações de pareamento expiram após 1 hora.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">`channels.imessage.cliPath` pode apontar para qualquer comando que faça proxy de stdin/stdout (por exemplo, um script wrapper que conecta via SSH a outro Mac e executa `imsg rpc`).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Configuração recomendada quando anexos estão habilitados:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Se `remoteHost` não estiver definido, o OpenClaw tenta detectá-lo automaticamente analisando o comando SSH no seu script wrapper.
    ```

  
</Tab>
</Tabs>

## Requisitos e permissões (macOS)

- Garanta que o Messages esteja conectado neste Mac.
- Acesso Total ao Disco para o OpenClaw + `imsg` (acesso ao DB do Messages).
- Permissão de Automação ao enviar.

<Tip>
O macOS concede permissões TCC por app/contexto de processo. Aprove os prompts no mesmo contexto que executa `imsg` (por exemplo, Terminal/iTerm, uma sessão LaunchAgent ou um processo iniciado via SSH).

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Controle de acesso e roteamento

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` controla mensagens diretas:


    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (padrão: pareamento).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">
    `channels.imessage.groupPolicy` controla o tratamento de grupos:


    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DMs usam roteamento direto; grupos usam roteamento de grupo.
    - Com o padrão `session.dmScope=main`, DMs do iMessage são consolidadas na sessão principal do agente.
    DMs compartilham a sessão principal do agente; grupos são isolados (`agent:<agentId>:imessage:group:<chat_id>`).<agentId>Grupos:<chat_id>`).
    - As respostas retornam ao iMessage usando os metadados do canal/destino de origem.

    ```
    Se um tópico com múltiplos participantes chegar com `is_group=false`, você ainda pode isolá-lo `chat_id` usando `channels.imessage.groups` (veja “Tópicos tipo grupo” abaixo).
    ```

  
</Tab>
</Tabs>

## Padrões de implantação

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Se você quiser que o bot envie a partir de uma **identidade iMessage separada** (e manter seus Messages pessoais limpos), use um Apple ID dedicado + um usuário macOS dedicado.

    ```
    Abra o Messages nesse usuário macOS e entre no iMessage usando o Apple ID do bot.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Topologia comum:


    ```
    Se o Gateway roda em um host/VM Linux, mas o iMessage precisa rodar em um Mac, o Tailscale é a ponte mais simples: o Gateway se comunica com o Mac pela tailnet, executa `imsg` via SSH e copia anexos de volta via SCP.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Use chaves SSH para que `ssh bot@mac-mini.tailnet-1234.ts.net` funcione sem prompts.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Para configurações de conta única, use opções simples (`channels.imessage.cliPath`, `channels.imessage.dbPath`) em vez do mapa `accounts`.

    ```
    Cada conta pode substituir campos como `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` e configurações de histórico.
    ```

  
</Accordion>
</AccordionGroup>

## Mídia, fragmentação e destinos de entrega

<AccordionGroup>
  <Accordion title="Attachments and media">Uploads de mídia são limitados por `channels.imessage.mediaMaxMb` (padrão 16).
</Accordion>

  <Accordion title="Outbound chunking">Divisão opcional por nova linha: defina `channels.imessage.chunkMode="newline"` para dividir em linhas em branco (limites de parágrafo) antes da divisão por tamanho.
</Accordion>

  <Accordion title="Addressing formats">
    Destinos explícitos preferidos:


    ```
    - `chat_id:123` (recomendado para roteamento estável)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Destinos por identificador também são suportados:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Escritas de configuração

Por padrão, o iMessage tem permissão para escrever atualizações de configuração acionadas por `/config set|unset` (requer `commands.config: true`).

Desative com:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Solução de problemas

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Valide o binário e o suporte a RPC:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Se a sondagem reportar RPC não suportado, atualize `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Aprovar via:

    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (padrão: lista de permissões).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Checklist:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Notas:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">Execute um comando interativo único em um terminal com GUI para forçar o prompt e, em seguida, tente novamente:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Inicie o gateway e aprove quaisquer prompts do macOS (Automação + Acesso Total ao Disco).
    ```

  
</Accordion>
</AccordionGroup>

## Referências de configuração

- Referência de configuração (iMessage)
- Configuração completa: [Configuração](/gateway/configuration)
- Detalhes: [Pareamento](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
