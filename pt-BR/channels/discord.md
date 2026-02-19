---
summary: "Status de suporte do bot do Discord, capacidades e configuração"
read_when:
  - Trabalhando em recursos do canal Discord
title: "Discord"
---

# Discord (API de Bot)

Status: pronto para DMs e canais de texto de guilda via o gateway oficial de bot do Discord.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    DMs do Discord usam o modo de pareamento por padrão.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Comportamento de comandos nativos e catálogo de comandos.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnósticos entre canais e fluxo de reparo.
  
</Card>
</CardGroup>

## Configuração rápida (iniciante)

<Steps>
  <Step title="Create a Discord bot and enable intents">Criar o app do Discord + usuário do bot

    ```
    Nas configurações do aplicativo Discord, ative **Message Content Intent** (e **Server Members Intent** se você pretende usar listas de permissões ou consultas por nome).
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Fallback de variável de ambiente para a conta padrão:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    Convide o bot para o seu servidor com permissões de mensagem.
  

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Os códigos de pareamento expiram após 1 hora.
    ```

  
</Step>
</Steps>

<Note>
A resolução de token é sensível à conta. Valores de token na configuração têm prioridade sobre o fallback de env. `DISCORD_BOT_TOKEN` é usado apenas para a conta padrão.
</Note>

## Modelo de runtime

- O Gateway é responsável pela conexão com o Discord.
- Manter o roteamento determinístico: as respostas sempre retornam ao canal de onde chegaram.
- O agente pode chamar `discord` com ações como:
- Chats diretos são consolidados na sessão principal do agente (padrão `agent:main:main`); canais de guilda permanecem isolados como `agent:<agentId>:discord:channel:<channelId>` (nomes de exibição usam `discord:<guildSlug>#<channelSlug>`).
- DMs em grupo são ignoradas por padrão; ative via `channels.discord.dm.groupEnabled` e, opcionalmente, restrinja por `channels.discord.dm.groupChannels`.
- Comandos nativos usam chaves de sessão isoladas (`agent:<agentId>:discord:slash:<userId>`) em vez da sessão compartilhada `main`.

## Controle de acesso e roteamento

<Tabs>
  <Tab title="DM policy">Para ignorar todas as DMs: defina `channels.discord.dm.enabled=false` ou `channels.discord.dm.policy="disabled"`.

    ```
    Para manter o comportamento antigo de “aberto a qualquer um”: defina `channels.discord.dm.policy="open"` e `channels.discord.dm.allowFrom=["*"]`.
    ```

  
</Tab>

  <Tab title="Guild policy">O comportamento é controlado por `channels.discord.replyToMode`:

    ```
    Comandos nativos respeitam as mesmas listas de permissões que DMs/mensagens de guilda (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, regras por canal).
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Se você definir apenas `DISCORD_BOT_TOKEN` e nunca criar uma seção `channels.discord`, o runtime
        define `groupPolicy` como `open`.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Mensagens de guild são, por padrão, restritas a menções.


    ```
    A detecção de menção inclui:
    
    - menção explícita ao bot
    - padrões de menção configurados (`agents.list[].groupChat.mentionPatterns`, fallback `messages.groupChat.mentionPatterns`)
    - comportamento implícito de resposta ao bot em casos suportados
    
    `requireMention` é configurado por guild/canal (`channels.discord.guilds...`).
    
    Group DMs:
    
    - padrão: ignoradas (`dm.groupEnabled=false`)
    - allowlist opcional via `dm.groupChannels` (IDs ou slugs de canais)
    ```

  
</Tab>
</Tabs>

### Roteamento de agente baseado em função

Use `bindings[].match.roles` para rotear membros de guild do Discord para diferentes agentes por ID de função. Bindings baseados em função aceitam apenas IDs de função e são avaliados após bindings de peer ou parent-peer e antes de bindings apenas de guild. Se um binding também definir outros campos de correspondência (por exemplo, `peer` + `guildId` + `roles`), todos os campos configurados devem corresponder.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Configuração do Developer Portal

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Crie um bot do Discord e copie o token do bot.
    ```

  
</Accordion>

  <Accordion title="Privileged intents">Em **Bot** → **Privileged Gateway Intents**, ative:

    ```
    - Message Content Intent
    - Server Members Intent (recomendado)
    
    Presence intent é opcional e só é necessário se você quiser receber atualizações de presença. Definir a presença do bot (`setPresence`) não exige habilitar atualizações de presença para membros.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">No seu app: **OAuth2** → **URL Generator**

    ```
    - scopes: `bot`, `applications.commands`
    
    Permissões básicas típicas:
    
    - View Channels
    - Send Messages
    - Read Message History
    - Embed Links
    - Attach Files
    - Add Reactions (opcional)
    
    Evite `Administrator`, a menos que seja explicitamente necessário.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Ative o Modo Desenvolvedor do Discord e, em seguida, copie:

    ```
    - ID do servidor
    - ID do canal
    - ID do usuário
    
    Prefira IDs numéricos na configuração do OpenClaw para auditorias e verificações mais confiáveis.
    ```

  
</Accordion>
</AccordionGroup>

## Comandos nativos e autenticação de comandos

- Comandos nativos opcionais: `commands.native` tem padrão `"auto"` (ativado para Discord/Telegram, desativado para Slack).
- Ou config: `channels.discord.token: "..."`.
- Substitua com `channels.discord.commands.native: true|false|"auto"`; `false` limpa comandos registrados anteriormente.
- A autenticação de comandos nativos usa as mesmas allowlists/políticas do Discord que o tratamento normal de mensagens.
- Slash commands ainda podem ser visíveis na UI do Discord para usuários que não estão na lista de permissões; o OpenClaw aplica as listas na execução e responde “não autorizado”.

Veja [Slash commands](/tools/slash-commands) para o catálogo de comandos e comportamento.

## Detalhes do recurso

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    O Discord oferece suporte a tags de resposta na saída do agente:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    Controlado por `channels.discord.replyToMode`:
    
    - `off` (padrão)
    - `first`
    - `all`
    
    Observação: `off` desativa o encadeamento implícito de respostas. Tags explícitas `[[reply_to_*]]` ainda são respeitadas.
    
    IDs de mensagens são expostos no contexto/histórico para que os agentes possam direcionar mensagens específicas.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Contexto de histórico da guild:

    ```
    - `channels.discord.historyLimit` padrão `20`
    - fallback: `messages.groupChat.historyLimit`
    - `0` desativa
    
    Controles de histórico de DM:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    Comportamento de threads:
    
    - threads do Discord são roteadas como sessões de canal
    - metadados da thread pai podem ser usados para vinculação à sessão pai
    - a configuração da thread herda a configuração do canal pai, a menos que exista uma entrada específica para a thread
    
    Tópicos de canal são injetados como contexto **não confiável** (não como prompt de sistema).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications`:

    ```
    `guilds.<id> .reactionNotifications`: modo de evento do sistema de reações (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` envia um emoji de confirmação enquanto o OpenClaw está processando uma mensagem recebida.

    ```
    Ordem de resolução:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - fallback para emoji da identidade do agente (`agents.list[].identity.emoji`, caso contrário "👀")
    
    Observações:
    
    - O Discord aceita emoji unicode ou nomes de emoji personalizados.
    - Use `""` para desativar a reação em um canal ou conta.
    ```

  
</Accordion>

  <Accordion title="Config writes">
    Gravações de configuração iniciadas pelo canal são habilitadas por padrão.

    ```
    Isso afeta os fluxos `/config set|unset` (quando os recursos de comando estão habilitados).
    
    Desativar:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Roteie o tráfego WebSocket do gateway do Discord por meio de um proxy HTTP(S) com `channels.discord.proxy`.

```json5
No canal do seu servidor, envie: `@Krill hello` (ou o nome do seu bot).
```

    ```
    Substituição por conta:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">`pluralkit`: resolver mensagens proxied do PluralKit para que membros do sistema apareçam como remetentes distintos.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Observações:
    
    - allowlists podem usar `pk:<memberId>`
    - nomes de exibição de membros são correspondidos por nome/slug
    - buscas usam o ID original da mensagem e são limitadas por janela de tempo
    - se a busca falhar, mensagens proxy são tratadas como mensagens de bot e descartadas, a menos que `allowBots=true`
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Atualizações de presença são aplicadas apenas quando você define um campo de status ou atividade.

    ```
    Exemplo apenas de status:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Exemplo de atividade (status personalizado é o tipo de atividade padrão):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Exemplo de streaming:
    ```

```json5
Execute o gateway; ele inicia automaticamente o canal do Discord quando um token está disponível (config primeiro, fallback por env) e `channels.discord.enabled` não é `false`.
```

    ```
    Mapa de tipos de atividade:
    
    - 0: Jogando
    - 1: Transmitindo (requer `activityUrl`)
    - 2: Ouvindo
    - 3: Assistindo
    - 4: Personalizado (usa o texto da atividade como estado do status; emoji é opcional)
    - 5: Competindo
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    O Discord oferece suporte a aprovações de exec baseadas em botões em DMs e pode opcionalmente publicar solicitações de aprovação no canal de origem.

    ```
    Caminho de configuração:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, padrão: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    Quando `target` é `channel` ou `both`, a solicitação de aprovação fica visível no canal. Apenas aprovadores configurados podem usar os botões; outros usuários recebem uma negação efêmera. As solicitações de aprovação incluem o texto do comando, portanto habilite a entrega no canal apenas em canais confiáveis. Se o ID do canal não puder ser derivado da chave da sessão, o OpenClaw retorna para entrega por DM.
    
    Se as aprovações falharem com IDs de aprovação desconhecidos, verifique a lista de aprovadores e se o recurso está habilitado.
    
    Documentação relacionada: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Ações de ferramentas

As ações de mensagem do Discord incluem mensagens, administração de canais, moderação, presença e ações de metadados.

Exemplos principais:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- Reagir + listar reações + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Os controles de ação ficam em `channels.discord.actions.*`.

Comportamento padrão dos controles:

| Grupo de ações                                                                                                | Padrão     |
| ------------------------------------------------------------------------------------------------------------- | ---------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled    |
| roleInfo                                                                                                      | desativado |
| moderação                                                                                                     | desativado |
| presença                                                                                                      | desativado |

## UI Components v2

O OpenClaw usa componentes v2 do Discord para aprovações de exec e marcadores entre contextos. As ações de mensagem do Discord também podem aceitar `components` para UI personalizada (avançado; requer instâncias de componente Carbon), enquanto os `embeds` legados continuam disponíveis, mas não são recomendados.

- `channels.discord.ui.components.accentColor` define a cor de destaque usada pelos contêineres de componentes do Discord (hex).
- Defina por conta com `channels.discord.accounts.<id>.ui.components.accentColor`.
- `embeds` são ignorados quando componentes v2 estão presentes.

Exemplo:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

## messages

Mensagens de voz no Discord exibem uma prévia de forma de onda e exigem áudio OGG/Opus além de metadados. O OpenClaw gera a forma de onda automaticamente, mas precisa que `ffmpeg` e `ffprobe` estejam disponíveis no host do gateway para inspecionar e converter arquivos de áudio.

Requisitos e restrições:

- Forneça um **caminho de arquivo local** (URLs são rejeitadas).
- Omita o conteúdo de texto (o Discord não permite texto + mensagem de voz na mesma carga).
- Qualquer formato de áudio é aceito; o OpenClaw converte para OGG/Opus quando necessário.

Exemplo:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Solução de problemas

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    - habilite Message Content Intent
    - habilite Server Members Intent quando depender da resolução de usuário/membro
    - reinicie o gateway após alterar intents
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`: controla o tratamento de canais de guilda (`open|disabled|allowlist`); `allowlist` requer listas de permissões de canal.
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">    Causas comuns:

    ```
    Para permitir **nenhum canal**, defina `channels.discord.groupPolicy: "disabled"` (ou mantenha uma lista vazia).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**Auditorias de permissões** (`channels status --probe`) verificam apenas IDs numéricos de canal.

    ```
    Se você usar chaves slug, a correspondência em tempo de execução ainda pode funcionar, mas o probe não consegue verificar completamente as permissões.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DMs não funcionam**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, ou você ainda não foi aprovado (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">Mensagens escritas pelo bot são ignoradas por padrão; defina `channels.discord.allowBots=true` para permiti-las (as próprias mensagens continuam filtradas).

    ```
    Aviso: se você permitir respostas a outros bots (`channels.discord.allowBots=true`), evite loops de bot-para-bot com listas de permissões `requireMention`, `channels.discord.guilds.*.channels.<id> .users` e/ou limpe os guardrails em `AGENTS.md` e `SOUL.md`.
    ```

  
</Accordion>
</AccordionGroup>

## Pontos de referência da configuração

Referência principal:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Campos importantes do Discord:

- `presence` (status/atividade do bot, padrão `false`)
- `guilds.<id> .channels.<channel> .allow`: permitir/negar o canal quando `groupPolicy="allowlist"`.
- Use `commands.useAccessGroups: false` para ignorar verificações de grupo de acesso para comandos.
- resposta/histórico: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- entrega: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- mídia/retry: `mediaMaxMb`, `retry`
- ações: `actions.*`
- presença: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- recursos: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Segurança e operações

- Trate tokens de bot como segredos (`DISCORD_BOT_TOKEN` é preferível em ambientes supervisionados).
- Conceda as permissões mínimas necessárias no Discord.
- Se o deploy/estado dos comandos estiver desatualizado, reinicie o gateway e verifique novamente com `openclaw channels status --probe`.

## Relacionado

- [Pareamento](/channels/pairing)
- [Roteamento de canal](/channels/channel-routing)
- [Solução de problemas](/channels/troubleshooting)
- Lista completa de comandos + config: [Slash commands](/tools/slash-commands)
