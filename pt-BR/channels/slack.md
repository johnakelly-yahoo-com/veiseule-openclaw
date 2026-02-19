---
summary: "Configuração do Slack para modo socket ou webhook HTTP"
read_when:
  - Ao configurar o Slack ou depurar o modo socket/HTTP do Slack
title: "Slack"
---

# Slack

Status: pronto para produção para DMs + canais via integrações de app do Slack. Modo HTTP (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    DMs do Slack usam o modo de pareamento por padrão.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Comportamento de comandos nativos e catálogo de comandos.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnósticos entre canais e playbooks de reparo.
  
</Card>
</CardGroup>

## Configuração rápida (iniciante)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Nas configurações do app Slack:

        ```
        **OAuth & Permissions** → instale o app e copie o **Bot User OAuth Token** (`xoxb-...`).
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Fallback de env (apenas conta padrão):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subscribe app events">
          Inscreva eventos do bot para:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Também habilite a **Aba Mensagens** do App Home para DMs.
        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        Use o modo webhook HTTP quando seu Gateway for acessível pelo Slack via HTTPS (típico para implantações em servidor). O modo HTTP usa a Events API + Interactivity + Slash Commands com uma URL de requisição compartilhada.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      Modo HTTP com múltiplas contas: defina `channels.slack.accounts.<id> .mode = "http"` e forneça um
      `webhookPath` exclusivo por conta para que cada app do Slack possa apontar para sua própria URL.

  
</Tab>
</Tabs>

## Modelo de token

- `botToken` + `appToken` são obrigatórios para o Socket Mode.
- O modo HTTP requer `botToken` + `signingSecret`.
- Tokens definidos na configuração substituem o fallback de env.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` como fallback de env se aplicam apenas à conta padrão.
- `userToken` (`xoxp-...`) é apenas de configuração (sem fallback de env) e o padrão é comportamento somente leitura (`userTokenReadOnly: true`).
- Opcional: adicione `chat:write.customize` se quiser que as mensagens enviadas usem a identidade ativa do agente (com `username` personalizado e ícone). `icon_emoji` usa a sintaxe `:emoji_name:`.

<Tip>
Para ações/leituras de diretório, o user token pode ser preferido quando configurado. Mesmo com `userTokenReadOnly: false`, o token do bot continua
preferido para gravações quando está disponível.
</Tip>

## Controle de acesso e roteamento

<Tabs>
  <Tab title="DM policy">Para permitir qualquer pessoa: defina `channels.slack.dm.policy="open"` e `channels.slack.dm.allowFrom=["*"]`.

    ```
    - `pairing` (padrão)
    - `allowlist`
    - `open` (requer que `channels.slack.allowFrom` inclua `"*"`; legado: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    Flags de DM:
    
    - `dm.enabled` (padrão true)
    - `channels.slack.allowFrom` (preferido)
    - `dm.allowFrom` (legado)
    - `dm.groupEnabled` (DMs em grupo padrão false)
    - `dm.groupChannels` (allowlist opcional de MPIM)
    
    O pareamento em DMs usa `openclaw pairing approve slack <code>`.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` controla o tratamento de canais (`open|disabled|allowlist`).

    ```
    Se você definir apenas `SLACK_BOT_TOKEN`/`SLACK_APP_TOKEN` e nunca criar uma seção `channels.slack`,
      o runtime define `groupPolicy` como `open` por padrão. Adicione `channels.slack.groupPolicy`,
    `channels.defaults.groupPolicy` ou uma lista de permissões de canais para restringir.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Mensagens em canais são restritas a menções por padrão.
  

    ```
    .allowBots=true`), evite loops de bot-para-bot com allowlists `requireMention`, `channels.slack.channels.<id> .users` e/ou proteções claras em `AGENTS.md` e `SOUL.md`.
    ```

  
</Tab>
</Tabs>

## Comandos e comportamento de slash

- O modo nativo vem desativado por padrão para o Slack, a menos que você defina `channels.slack.commands.native: true` (o `commands.native` global é `"auto"`, que mantém o Slack desativado).
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Quando os comandos nativos estiverem habilitados, registre os slash commands correspondentes no Slack (nomes `/<command>`).
- Se os comandos nativos não estiverem habilitados, você pode executar um único slash command configurado via `channels.slack.slashCommand`.

Configurações padrão de slash command:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Sessões de slash usam chaves isoladas:

- Slash commands usam sessões `agent:<agentId>:slack:slash:<userId>` (prefixo configurável via `channels.slack.slashCommand.sessionPrefix`).

e ainda roteiam a execução do comando para a sessão de conversa de destino (`CommandTargetSessionKey`).

## Threads, sessões e tags de resposta

- DMs são roteadas como `direct`; canais como `channel`; MPIMs como `group`.
- Com o padrão `session.dmScope=main`, DMs do Slack são consolidadas na sessão principal do agente.
- Canais mapeiam para sessões `agent:<agentId>:slack:channel:<channelId>`.
- Respostas em thread podem criar sufixos de sessão de thread (`:thread:<threadTs>`) quando aplicável.
- O padrão de `channels.slack.thread.historyScope` é `thread`; `thread.inheritParent` é `false`.
- `channels.slack.historyLimit` (ou `channels.slack.accounts.*.historyLimit`) controla quantas mensagens recentes de canal/grupo são incluídas no prompt.

Controles de encadeamento de respostas:

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- fallback legado para chats diretos: `channels.slack.dm.replyToMode`

Tags manuais de resposta são suportadas:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Observação: `replyToMode="off"` desativa o encadeamento implícito de respostas. Tags explícitas `[[reply_to_*]]` ainda são respeitadas.

## Mídia, fragmentação e entrega

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Anexos de arquivos do Slack são baixados a partir de URLs privadas hospedadas pelo Slack (fluxo de requisição autenticado por token) e gravados no armazenamento de mídia quando a obtenção é bem-sucedida e os limites de tamanho permitem.


    ```
    Uploads de mídia são limitados por `channels.slack.mediaMaxMb` (padrão 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - blocos de texto usam `channels.slack.textChunkLimit` (padrão 4000)
    - `channels.slack.chunkMode="newline"` habilita divisão priorizando parágrafos
    - envios de arquivos usam as APIs de upload do Slack e podem incluir respostas em threads (`thread_ts`)
    - o limite de mídia de saída segue `channels.slack.mediaMaxMb` quando configurado; caso contrário, os envios do canal usam os padrões por tipo MIME do pipeline de mídia
  
</Accordion>

  <Accordion title="Delivery targets">
    Alvos explícitos preferidos:

    ```
    `im:write` (abrir DMs via `conversations.open` para DMs de usuário)
    [https://docs.slack.dev/reference/methods/conversations.open](https://docs.slack.dev/reference/methods/conversations.open)
    ```

  
</Accordion>
</AccordionGroup>

## Ações e controles

Ações de ferramentas do Slack podem ser controladas com `channels.slack.actions.*`:

Grupos de ações disponíveis na integração atual do Slack:

| Grupo de ação | Padrão  |
| ------------- | ------- |
| messages      | enabled |
| reactions     | enabled |
| pins          | enabled |
| memberInfo    | enabled |
| emojiList     | enabled |

## Eventos e comportamento operacional

- Edições/exclusões de mensagens e transmissões em threads são mapeadas para eventos do sistema.
- Eventos de adição/remoção de reações são mapeados para eventos do sistema.
- Eventos de entrada/saída de membros, canal criado/renomeado e adição/remoção de fixações são mapeados para eventos do sistema.
- `channel_id_changed` pode migrar chaves de configuração do canal quando `configWrites` está habilitado.
- Metadados de tópico/finalidade do canal são tratados como contexto não confiável e podem ser injetados no contexto de roteamento.

## Reagir + listar reações

`ackReaction` envia um emoji de confirmação enquanto o OpenClaw está processando uma mensagem recebida.

Ordem de resolução:

- `ou`channels.slack.channels.<name>.ackReaction\`
- `channels.slack.ackReaction`
- `replyToMode`
- fallback para emoji de identidade do agente (`agents.list[].identity.emoji`, caso contrário "👀")

Notas

- O Slack espera shortcodes (por exemplo, `"eyes"`).
- Use `""` para desativar a reação para um canal ou conta.

## Checklist de manifesto e escopos

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    Se você configurar `channels.slack.userToken`, os escopos típicos de leitura são:


    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Solução de problemas

<AccordionGroup>
  <Accordion title="No replies in channels">
    Verifique, em ordem:

    ```
    `users`: allowlist opcional de usuários por canal.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Verifique:

    ```
    `allowlist` exige que os canais estejam listados em `channels.slack.channels`.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Valide os tokens do bot + do app e a habilitação do Socket Mode nas configurações do app do Slack.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Valide:

    ```
    - signing secret
    - caminho do webhook
    - URLs de requisição do Slack (Events + Interactivity + Slash Commands)
    - `webhookPath` exclusivo por conta HTTP
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Verifique se você pretendia:

    ```
    - modo de comando nativo (`channels.slack.commands.native: true`) com slash commands correspondentes registrados no Slack
    - ou modo de slash command único (`channels.slack.slashCommand.enabled: true`)
    
    Verifique também `commands.useAccessGroups` e as allowlists de canal/usuário.
    ```

  
</Accordion>
</AccordionGroup>

## Pontos de referência de configuração

Referência principal:

- `users:read` (consulta de usuário)
  [https://docs.slack.dev/reference/methods/users.info](https://docs.slack.dev/reference/methods/users.info)

  Campos de Slack de alto impacto:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - Acesso por DM: `dm.enabled`, `dmPolicy`, `allowFrom` (legado: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: permitir/negar o canal quando `groupPolicy="allowlist"`.
  - encadeamento/histórico: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - entrega: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - ops/recursos: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## Relacionados

- Fixar/desafixar/listar
- `channel`: canais padrão (públicos/privados)
- Para fluxo de triagem: [/channels/troubleshooting](/channels/troubleshooting).
- Configuração
- Lista completa de comandos + configuração: [Slash commands](/tools/slash-commands)

