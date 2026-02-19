---
summary: "Suporte ao canal WhatsApp, controles de acesso, comportamento de entrega e operações"
read_when:
  - Trabalhando no comportamento do canal WhatsApp/web ou no roteamento do inbox
title: "WhatsApp"
---

# WhatsApp (canal web)

Status: apenas WhatsApp Web via Baileys. O Gateway é dono da(s) sessão(ões).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing"> 
    A política padrão de DM é emparelhamento para remetentes desconhecidos.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting"> 
    Diagnósticos entre canais e playbooks de reparo.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration"> 
    Padrões completos de configuração de canal e exemplos.
  
</Card>
</CardGroup>

## Configuração rápida (iniciante)

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Para uma conta específica:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Aprove com: `openclaw pairing approve whatsapp <code>` (liste com `openclaw pairing list whatsapp`).
```

    ```
    Os códigos expiram após 1 hora; solicitações pendentes são limitadas a 3 por canal.
    ```

  
</Step>
</Steps>

<Note>
Ótimo para manter seu WhatsApp pessoal separado — instale o WhatsApp Business e registre o número do OpenClaw ali. (Os metadados do canal e o fluxo de onboarding são otimizados para essa configuração, mas configurações com número pessoal também são suportadas.)
</Note>

## Padrões de implantação

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)"> 
    Este é o modo operacional mais limpo:
  

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback"> 
    O onboarding oferece suporte ao modo com número pessoal e grava uma linha de base amigável para autochat:
  

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope"> 
    O canal da plataforma de mensagens é baseado no WhatsApp Web (`Baileys`) na arquitetura atual de canais do OpenClaw.
  

    ```
    Não há um canal separado de mensagens WhatsApp via Twilio no registro integrado de canais de chat.
    ```

  
</Accordion>
</AccordionGroup>

## Modelo de runtime

- O Gateway é responsável pelo socket do WhatsApp e pelo loop de reconexão.
- Envios de saída exigem um listener ativo do WhatsApp para a conta de destino.
- Chats de status/broadcast são ignorados.
- Conversas diretas usam as regras de sessão de DM (`session.dmScope`; o padrão `main` consolida DMs na sessão principal do agente).
- Grupos mapeiam para sessões `agent:<agentId>:whatsapp:group:<jid>`.

## Controle de acesso e ativação

<Tabs>
  <Tab title="DM policy">**Política de DM**: `channels.whatsapp.dmPolicy` controla o acesso a chats diretos (padrão: `pairing`).

    ```
    Aberto: requer `channels.whatsapp.allowFrom` incluir `"*"`.
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    O acesso a grupos tem duas camadas:

    ```
    `channels.whatsapp.groups` (allowlist de grupo + padrões de bloqueio por menção; use `"*"` para permitir todos)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Respostas em grupo exigem menção por padrão.

A detecção de menção inclui:

- menções explícitas ao bot no WhatsApp
- padrões de regex de menção configurados (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
- detecção implícita de resposta ao bot (remetente da resposta corresponde à identidade do bot)

Comando de ativação em nível de sessão:

- `/activation mention`
- `/activation always`

`activation` atualiza o estado da sessão (não a configuração global). É restrito ao proprietário.

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## Quando o próprio número vinculado também está presente em `allowFrom`, as proteções de conversa consigo mesmo do WhatsApp são ativadas:

ignorar confirmações de leitura em interações de conversa consigo mesmo

- ignorar o comportamento de disparo automático por menção-JID que, de outra forma, mencionaria você mesmo
- ```
  Mensagens recebidas do WhatsApp são encapsuladas no envelope de entrada compartilhado.
  ```
- As respostas em autochat usam por padrão `[{identity.name}]` quando definido (caso contrário `[openclaw]`)
  se `messages.responsePrefix` não estiver definido.

## Normalização de mensagens (o que o modelo vê)

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">Se existir uma resposta citada, o contexto é acrescentado neste formato:

```text
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

Os campos de metadados de resposta também são preenchidos quando disponíveis (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Cargas de localização e contato são normalizadas em contexto textual antes do roteamento.
    ```

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Mensagens recebidas apenas com mídia usam placeholders:

    ```
    
        Para grupos, mensagens não processadas podem ser armazenadas em buffer e injetadas como contexto quando o bot for finalmente acionado.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Confirmações de leitura são ativadas por padrão para mensagens recebidas do WhatsApp que forem aceitas.

    ```
    Mensagens recentes _não processadas_ (padrão 50) inseridas em:
    `[Chat messages since your last reply - for context]` (mensagens já na sessão não são reinjetadas)
    ```

  
</Accordion>

  <Accordion title="Read receipts">  
</Accordion>

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

```
- suporta cargas de imagem, vídeo, áudio (nota de voz PTT) e documento
- `audio/ogg` é reescrito para `audio/ogg; codecs=opus` para compatibilidade com notas de voz
- a reprodução de GIF animado é suportada via `gifPlayback: true` em envios de vídeo
- legendas são aplicadas ao primeiro item de mídia ao enviar respostas com múltiplas mídias
- a origem da mídia pode ser HTTP(S), `file://` ou caminhos locais
```
---

<AccordionGroup>
  <Accordion title="Text chunking">Fragmentação opcional por nova linha: defina `channels.whatsapp.chunkMode="newline"` para dividir em linhas em branco (limites de parágrafo) antes da fragmentação por comprimento.
</Accordion>

  <Accordion title="Outbound media behavior">
    - limite de salvamento de mídia recebida: `channels.whatsapp.mediaMaxMb` (padrão `50`)
    - limite de mídia de saída para respostas automáticas: `agents.defaults.mediaMaxMb` (padrão `5MB`)
    - imagens são otimizadas automaticamente (redimensionamento/ajuste de qualidade) para se enquadrar nos limites
    - em caso de falha no envio de mídia, o fallback do primeiro item envia um aviso em texto em vez de descartar a resposta silenciosamente
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">Reações de confirmação
</Accordion>
</AccordionGroup>

## enviadas imediatamente após a entrada ser aceita (pré-resposta)

`channels.whatsapp.ackReaction` (auto-reação ao receber mensagem: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

Comportamento:

- falhas são registradas em log, mas não bloqueiam a entrega normal da resposta
- o modo de grupo `mentions` reage em interações acionadas por menção; a ativação de grupo `always` atua como bypass dessa verificação
- Multi-conta e credenciais
- O WhatsApp ignora `messages.ackReaction`; use `channels.whatsapp.ackReaction` em vez disso.

## ]\` limpa o estado de autenticação do WhatsApp para essa conta.

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (configurações por conta + `authDir` opcional).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Logout: `openclaw channels logout` (ou `--account <id>`) apaga o estado de autenticação do WhatsApp (mas mantém o `oauth.json` compartilhado).<accountId>Credenciais armazenadas em `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
</Accordion>

  <Accordion title="Logout behavior">`channels.whatsapp.messagePrefix` (prefixo de entrada; por conta: `channels.whatsapp.accounts.<accountId><id>Em diretórios de autenticação legados, `oauth.json` é preservado enquanto os arquivos de autenticação do Baileys são removidos.

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## Bloqueios de ação:

- Ferramenta: `whatsapp` com a ação `react` (`chatJid`, `messageId`, `emoji`, `remove` opcional).
- Correção:```bash
  openclaw channels login --channel whatsapp
  openclaw channels status
  ```
  - `channels.whatsapp.actions.reactions` (bloquear reações de ferramenta do WhatsApp).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Solução de problemas (rápida)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Sintoma: `channels status` mostra `linked: false` ou avisa “Not linked”.

    ```
    
        Sintoma: conta vinculada com desconexões repetidas ou tentativas de reconexão.
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Envios de saída falham imediatamente quando não existe um listener ativo do gateway para a conta de destino.

    ```
    Correção: `openclaw doctor` (ou reinicie o gateway). Se persistir, relinque via `channels login` e inspecione `openclaw logs --follow`.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">Certifique-se de que o gateway está em execução e que a conta está vinculada.

    ```
    
        Verifique nesta ordem:
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">O runtime do gateway do WhatsApp deve usar Node.

    ```
    Política de grupos: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (padrão `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Bun é sinalizado como incompatível para operação estável do gateway do WhatsApp/Telegram. Referências de configuração
  
</Accordion>
</AccordionGroup>

## Referência principal:

Primary reference:

- [Referência de configuração - WhatsApp](/gateway/configuration-reference#whatsapp)

Campos de alto impacto do WhatsApp:

- acesso: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- entrega: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- Login multi-account: `openclaw channels login --account <id>` (`<id>` = `accountId`)..enabled`, `accounts.<id>`.authDir`, substituições no nível da conta
- operações: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- comportamento da sessão: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Relacionado

- [Pareamento](/channels/pairing)
- [Roteamento de canal](/channels/channel-routing)
- Guia de solução de problemas: [Solução de problemas do Gateway](/gateway/troubleshooting).
