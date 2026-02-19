---
summary: "Status de suporte do bot do Telegram, capacidades e configuração"
read_when:
  - Trabalhando em recursos ou webhooks do Telegram
title: "Telegram"
---

# Telegram (Bot API)

Status: pronto para produção para DMs de bots + grupos via grammY. Long-polling por padrão; webhook opcional.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">O pareamento é a troca de token padrão usada para DMs do Telegram.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Diagnósticos entre canais e guias de correção.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Padrões completos de configuração de canal e exemplos.
  
</Card>
</CardGroup>

## Configuração rápida (iniciante)

<Steps>
  <Step title="Create the bot token in BotFather">Abra o Telegram e converse com **@BotFather** ([link direto](https://t.me/BotFather)). Confirme que o handle é exatamente `@BotFather`.

    ```
    Execute `/newbot`, depois siga as instruções (nome + nome de usuário terminando em `bot`).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Opção via env: `TELEGRAM_BOT_TOKEN=...` (funciona para a conta padrão).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Remetentes desconhecidos recebem um código de pareamento; mensagens são ignoradas até a aprovação (códigos expiram após 1 hora).
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Adicione o bot ao seu grupo e, em seguida, defina `channels.telegram.groups` e `groupPolicy` para corresponder ao seu modelo de acesso.
  
</Step>
</Steps>

<Note>
A ordem de resolução de token é sensível à conta. Na prática, os valores de configuração prevalecem sobre o fallback de variáveis de ambiente, e `TELEGRAM_BOT_TOKEN` se aplica apenas à conta padrão.
</Note>

## Configurações do lado do Telegram

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Bots do Telegram usam **Modo de Privacidade** por padrão, o que limita quais mensagens de grupos eles recebem.

    ```
    **Nota:** Ao alternar o modo de privacidade, o Telegram exige remover e re-adicionar o bot
    a cada grupo para que a alteração tenha efeito.
    ```

  
</Accordion>

  <Accordion title="Group permissions">O status de admin é definido dentro do grupo (UI do Telegram).

    ```
    Adicionar o bot como **admin** do grupo (bots admin recebem todas as mensagens).
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — permitir/negar adicionar o bot a grupos.
    ```

  
</Accordion>
</AccordionGroup>

## Controle de acesso e ativação

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` controla o acesso por mensagem direta:


    ```
    - `pairing` (padrão)
    - `allowlist`
    - `open` (exige que `allowFrom` inclua `"*"`)
    - `disabled`
    
    `channels.telegram.allowFrom` aceita IDs numéricos de usuários do Telegram. Prefixos `telegram:` / `tg:` são aceitos e normalizados.
    O assistente de onboarding aceita entrada `@username` e a resolve para IDs numéricos.
    Se você atualizou e sua configuração contém entradas `@username` na allowlist, execute `openclaw doctor --fix` para resolvê-las (melhor esforço; requer um token de bot do Telegram).
    
    ### Encontrando seu ID de usuário do Telegram
    
    Mais seguro (sem bot de terceiros):
    
    1. Envie uma DM para o seu bot.
    2. Execute `openclaw logs --follow`.
    3. Leia `from.id`.
    
    Método oficial da Bot API:
    
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    **Nota de privacidade:** `@userinfobot` é um bot de terceiros.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">Dois controles independentes:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Respostas em grupos exigem menção por padrão (menção nativa @ ou `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    Menção pode vir de:
    
    - menção nativa `@botusername`, ou
    - padrões de menção em:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Alternâncias de comando em nível de sessão:
    
    - `/activation always`
    - `/activation mention`
    
    Esses comandos atualizam apenas o estado da sessão. Use a configuração para persistência.
    
    Exemplo de configuração persistente:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Encaminhe qualquer mensagem do grupo para `@userinfobot` ou `@getidsbot` no Telegram para ver o ID do chat (número negativo como `-1001234567890`).
    ```

  
</Tab>
</Tabs>

## Comportamento em tempo de execução

- Um canal da API de Bots do Telegram pertencente ao Gateway.
- Roteamento determinístico: as respostas voltam para o Telegram; o modelo nunca escolhe canais.
- Mensagens de entrada são normalizadas no envelope de canal compartilhado com contexto de resposta e placeholders de mídia.
- Sessões de grupo são isoladas por ID do grupo. Anexa `:topic:<threadId>` à chave de sessão do grupo do Telegram para que cada tópico fique isolado.
- Envia indicadores de digitação e respostas com `message_thread_id` para que as respostas permaneçam no tópico.
- Long-polling usa o runner do grammY com sequenciamento por chat; a concorrência geral é limitada por `agents.defaults.maxConcurrent`. A concorrência geral do runner sink usa `agents.defaults.maxConcurrent`.
- A API de Bots do Telegram não suporta recibos de leitura; não há opção `sendReadReceipts`.

## Referência de recursos

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">O OpenClaw pode transmitir respostas parciais em DMs do Telegram usando `sendMessageDraft`.

    ```
    Requisito:
    
    - `channels.telegram.streamMode` não é `"off"` (padrão: `"partial"`)
    
    Modos:
    
    - `off`: sem pré-visualização ao vivo
    - `partial`: atualizações frequentes de pré-visualização a partir de texto parcial
    - `block`: atualizações de pré-visualização em blocos usando `channels.telegram.draftChunk`
    
    Padrões de `draftChunk` para `streamMode: "block"`:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars` é limitado por `channels.telegram.textChunkLimit`.
    
    Funciona em chats diretos e em grupos/tópicos.
    
    Para respostas somente em texto, o OpenClaw mantém a mesma mensagem de pré-visualização e realiza uma edição final no local (sem segunda mensagem).
    
    Para respostas complexas (por exemplo, cargas de mídia), o OpenClaw volta ao envio final normal e depois remove a mensagem de pré-visualização.
    
    `streamMode` é separado do streaming em blocos. Quando o streaming em blocos é explicitamente habilitado para o Telegram, o OpenClaw ignora o stream de pré-visualização para evitar streaming duplo.
    
    Stream de raciocínio somente no Telegram:
    
    - `/reasoning stream` envia o raciocínio para a pré-visualização ao vivo durante a geração
    - a resposta final é enviada sem o texto de raciocínio
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Texto de saída do Telegram usa `parse_mode: "HTML"` (subconjunto de tags suportadas pelo Telegram).

    ```
    - Texto no estilo Markdown é renderizado para HTML seguro para o Telegram.
    - HTML bruto do modelo é escapado para reduzir falhas de parsing no Telegram.
    - Se o Telegram rejeitar o HTML parseado, o OpenClaw tenta novamente como texto simples.
    
    Pré-visualizações de link são habilitadas por padrão e podem ser desativadas com `channels.telegram.linkPreview: false`.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    O OpenClaw registra comandos nativos (como `/status`, `/reset`, `/model`) no menu de bots do Telegram na inicialização. Você pode adicionar comandos personalizados ao menu via config:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Regras:
    
    - nomes são normalizados (remove `/` inicial, minúsculas)
    - padrão válido: `a-z`, `0-9`, `_`, comprimento `1..32`
    - comandos personalizados não podem sobrescrever comandos nativos
    - conflitos/duplicatas são ignorados e registrados em log
    
    Observações:
    
    - comandos personalizados são apenas entradas de menu; não implementam comportamento automaticamente
    - comandos de plugin/skill ainda podem funcionar quando digitados, mesmo que não apareçam no menu do Telegram
    
    Se os comandos nativos estiverem desativados, os comandos embutidos são removidos. Comandos personalizados/de plugin ainda podem ser registrados se configurados.
    
    Falha comum na configuração:
    
    - `setMyCommands failed` geralmente significa que o DNS/HTTPS de saída para `api.telegram.org` está bloqueado.
    
    ### Comandos de pareamento de dispositivo (plugin `device-pair`)
    
    Quando o plugin `device-pair` está instalado:
    
    1. `/pair` gera o código de configuração
    2. cole o código no app iOS
    3. `/pair approve` aprova a solicitação pendente mais recente
    
    Mais detalhes: [Pairing](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Configure o escopo do teclado inline:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Substituição por conta:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = nenhuma mensagem de grupo é aceita
      O padrão é `groupPolicy: "allowlist"` (bloqueado a menos que você adicione `groupAllowFrom`).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Quando um usuário clica em um botão, os dados de callback são enviados de volta ao agente como uma mensagem no formato:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    As ações de ferramenta do Telegram incluem:


    ```
    Ferramenta: `telegram` com ação `react` (`chatId`, `messageId`, `emoji`).
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">O Telegram suporta respostas encadeadas opcionais via tags:

    ```
    - `[[reply_to_current]]` responde à mensagem que disparou
    - `[[reply_to:<id>]]` responde a um ID específico de mensagem do Telegram
    
    `channels.telegram.replyToMode` controla o tratamento:
    
    - `off` (padrão)
    - `first`
    - `all`
    
    Observação: `off` desativa o encadeamento implícito de respostas. Tags explícitas `[[reply_to_*]]` ainda são respeitadas.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Tópicos (supergrupos de fórum)

    ```
    **Grupos de fórum:** Reações em grupos de fórum incluem `message_thread_id` e usam chaves de sessão como `agent:main:telegram:group:{chatId}:topic:{threadId}`.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### Mensagens de áudio


    ```
    `[[audio_as_voice]]` — enviar áudio como nota de voz em vez de arquivo.
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Notas de vídeo não suportam legendas; o texto fornecido é enviado separadamente.
    
    ### Stickers
    
    Tratamento de stickers recebidos:
    
    - WEBP estático: baixado e processado (placeholder `<media:sticker>`)
    - TGS animado: ignorado
    - WEBM em vídeo: ignorado
    
    Campos de contexto do sticker:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Arquivo de cache de stickers:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Stickers são descritos uma vez (quando possível) e armazenados em cache para reduzir chamadas repetidas de visão.
    
    Habilitar ações de sticker:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Enviando stickers
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Cache de adesivo
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Recebe a atualização `message_reaction` da API do Telegram

    ```
    Quando habilitado, o OpenClaw enfileira eventos de sistema como:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Configuração:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (padrão: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (padrão: `minimal`)
    
    Observações:
    
    - `own` significa reações de usuários a mensagens enviadas pelo bot apenas (melhor esforço via cache de mensagens enviadas).
    - O Telegram não fornece IDs de tópico nas atualizações de reação.
      - grupos não fórum são roteados para a sessão de chat do grupo
      - grupos de fórum são roteados para a sessão do tópico geral do grupo (`:topic:1`), não para o tópico exato de origem
    
    `allowed_updates` para polling/webhook inclui `message_reaction` automaticamente.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction` envia um emoji de confirmação enquanto o OpenClaw está processando uma mensagem recebida.


    ```
    Ordem de resolução:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - fallback para emoji da identidade do agente (`agents.list[].identity.emoji`, senão "👀")
    
    Observações:
    
    - O Telegram espera emoji unicode (por exemplo "👀").
    - Use `""` para desativar a reação para um canal ou conta.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    Escritas de configuração de canal são habilitadas por padrão (`configWrites !== false`).


    ```
    Você executa `/config set` ou `/config unset` em um chat do Telegram (requer `commands.config: true`).
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">Padrão: long polling.

    ```
    Modo webhook: defina `channels.telegram.webhookUrl` e `channels.telegram.webhookSecret` (opcionalmente `channels.telegram.webhookPath`).
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Texto de saída é dividido em `channels.telegram.textChunkLimit` (padrão 4000).
    Divisão opcional por nova linha: defina `channels.telegram.chunkMode="newline"` para dividir em linhas em branco (limites de parágrafo) antes da divisão por comprimento.
    Downloads/uploads de mídia são limitados por `channels.telegram.mediaMaxMb` (padrão 5).
    - `channels.telegram.timeoutSeconds` substitui o timeout do cliente da API do Telegram (se não definido, aplica-se o padrão do grammY).
    Contexto do histórico do grupo usa `channels.telegram.historyLimit` (ou `channels.telegram.accounts.*.historyLimit`), com fallback para `messages.groupChat.historyLimit`. Defina `0` para desabilitar (padrão 50).
    O histórico de DM pode ser limitado com `channels.telegram.dmHistoryLimit` (turnos do usuário). Substituições por usuário: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>Chamadas de saída à API do Telegram repetem em erros transitórios/rede/429 com backoff exponencial e jitter. Configure via `channels.telegram.retry`.

    ```
    O destino de envio via CLI pode ser ID numérico do chat ou username:
    ```

```bash
Exemplo: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Solução de problemas

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    Se você definiu `channels.telegram.groups.*.requireMention=false`, o **modo de privacidade** da API de Bots do Telegram deve estar desabilitado.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - quando `channels.telegram.groups` existir, o grupo deve estar listado (ou incluir `"*"`)
    - verifique se o bot é membro do grupo
    - revise os logs: `openclaw logs --follow` para motivos de ignorar
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - autorize a identidade do remetente (emparelhamento e/ou `allowFrom` numérico)
    - a autorização de comandos ainda se aplica mesmo quando a política do grupo é `open`
    - `setMyCommands failed` geralmente indica problemas de alcance DNS/HTTPS para `api.telegram.org`
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + fetch/proxy customizado pode acionar comportamento de abortamento imediato se os tipos de AbortSignal não corresponderem.
    - Alguns hosts resolvem `api.telegram.org` para IPv6 primeiro; saída IPv6 com falha pode causar falhas intermitentes na API do Telegram.
    - Valide as respostas DNS:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Mais ajuda: [Solução de problemas de canais](/channels/troubleshooting).

## Referência de configuração (Telegram)

Referência principal:

- `channels.telegram.enabled`: habilitar/desabilitar a inicialização do canal.

- `channels.telegram.botToken`: token do bot (BotFather).

- `channels.telegram.tokenFile`: ler token de um caminho de arquivo.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (padrão: pareamento).

- `channels.telegram.allowFrom`: lista de permissões de DM (ids/usernames). `open` requer `"*"`. `openclaw doctor --fix` pode resolver entradas legadas `@username` para IDs.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (padrão: lista de permissões).

- `channels.telegram.groupAllowFrom`: lista de permissões de remetentes de grupo (ids/usernames). `openclaw doctor --fix` pode resolver entradas legadas `@username` para IDs.

- `channels.telegram.groups`: padrões por grupo + lista de permissões (use `"*"` para padrões globais).
  - `channels.telegram.groups.<id>.groupPolicy`: substituição por grupo para groupPolicy (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: padrão de gating por menção.
  - `channels.telegram.groups.<id>.skills`: filtro de skills (omitir = todas as skills, vazio = nenhuma).
  - `channels.telegram.groups.<id>.allowFrom`: substituição da lista de permissões de remetentes por grupo.
  - `channels.telegram.groups.<id>.systemPrompt`: prompt de sistema extra para o grupo.
  - `channels.telegram.groups.<id>.enabled`: desabilitar o grupo quando `false`.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: substituições por tópico (mesmos campos do grupo).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: substituição por tópico para groupPolicy (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: substituição de gating por menção por tópico.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (padrão: lista de permissões).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: substituição por conta.

- `channels.telegram.replyToMode`: `off | first | all` (padrão: `first`).

- `channels.telegram.textChunkLimit`: tamanho de divisão de saída (caracteres).

- `channels.telegram.chunkMode`: `length` (padrão) ou `newline` para dividir em linhas em branco (limites de parágrafo) antes da divisão por comprimento.

- `channels.telegram.linkPreview`: alternar prévias de links para mensagens de saída (padrão: true).

- `channels.telegram.streamMode`: `off | partial | block` (streaming de rascunho).

- `channels.telegram.mediaMaxMb`: limite de mídia de entrada/saída (MB).

- `channels.telegram.retry`: política de tentativas para chamadas de saída da API do Telegram (tentativas, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: substituir autoSelectFamily do Node (true=habilitar, false=desabilitar). Padrão desabilitado no Node 22 para evitar timeouts do Happy Eyeballs.

- `channels.telegram.proxy`: URL de proxy para chamadas da API de Bots (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: habilitar modo webhook (requer `channels.telegram.webhookSecret`).

- `channels.telegram.webhookSecret`: segredo do webhook (obrigatório quando webhookUrl está definido).

- `channels.telegram.webhookPath`: caminho local do webhook (padrão `/telegram-webhook`).

- O listener local se liga a `0.0.0.0:8787` e serve `POST /telegram-webhook` por padrão.

- `channels.telegram.actions.reactions`: gatear reações da ferramenta do Telegram.

- `channels.telegram.actions.sendMessage`: gatear envios de mensagens da ferramenta do Telegram.

- `channels.telegram.actions.deleteMessage`: gatear exclusões de mensagens da ferramenta do Telegram.

- `channels.telegram.actions.sticker`: gatear ações de figurinhas do Telegram — enviar e buscar (padrão: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — controlar quais reações disparam eventos de sistema (padrão: `own` quando não definido).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — controlar a capacidade de reação do agente (padrão: `minimal` quando não definido).

- Configuração completa: [Configuração](/gateway/configuration)

Campos específicos do Telegram de alto sinal:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- Comandos exigem autorização mesmo em grupos com `groupPolicy: "open"`
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- Opcional (apenas para `streamMode: "block"`):
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- No modo webhook, reações são incluídas no webhook `allowed_updates`
- Gateamento de ferramentas: `channels.telegram.actions.reactions`, `channels.telegram.actions.sendMessage`, `channels.telegram.actions.deleteMessage` (padrão: habilitado) e `channels.telegram.actions.sticker` (padrão: desabilitado).
- Notificações de reações
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## Relacionado

- Detalhes: [Pareamento](/channels/pairing)
- [Roteamento de canal](/channels/channel-routing)
- Solução de problemas de configuração (comandos)

