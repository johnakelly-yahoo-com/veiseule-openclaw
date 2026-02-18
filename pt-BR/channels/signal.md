---
title: "Signal"
---

# Signal (signal-cli)

Status: integração externa via CLI. O Gateway se comunica com `signal-cli` por HTTP JSON-RPC + SSE.

## Início rápido (iniciante)

1. Use um **número de Signal separado** para o bot (recomendado).
2. Instale `signal-cli` (Java necessário).
3. Vincule o dispositivo do bot e inicie o daemon:
   - `signal-cli link -n "OpenClaw"`
4. Configure o OpenClaw e inicie o gateway.

Configuração mínima:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

## O que é

- Canal do Signal via `signal-cli` (não é libsignal embutido).
- Roteamento determinístico: as respostas sempre retornam ao Signal.
- DMs compartilham a sessão principal do agente; grupos são isolados (`agent:<agentId>:signal:group:<groupId>`).

## Escritas de configuração

Por padrão, o Signal pode escrever atualizações de configuração disparadas por `/config set|unset` (requer `commands.config: true`).

Desative com:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## O modelo de números (importante)

- O gateway se conecta a um **dispositivo do Signal** (a conta `signal-cli`).
- Se você executar o bot na **sua conta pessoal do Signal**, ele ignorará suas próprias mensagens (proteção contra loop).
- Para “eu envio mensagem ao bot e ele responde”, use um **número de bot separado**.

## Configuração (caminho rápido)

1. Instale `signal-cli` (Java necessário).
2. Vincule uma conta de bot:
   - `signal-cli link -n "OpenClaw"` e então escaneie o QR no Signal.
3. Configure o Signal e inicie o gateway.

Exemplo:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Suporte a múltiplas contas: use `channels.signal.accounts` com configuração por conta e `name` opcional. Veja [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) para o padrão compartilhado.

## Modo de daemon externo (httpUrl)

Se você quiser gerenciar o `signal-cli` por conta própria (inicializações frias lentas da JVM, init de container ou CPUs compartilhadas), execute o daemon separadamente e aponte o OpenClaw para ele:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Isso ignora o auto-spawn e a espera de inicialização dentro do OpenClaw. Para inicializações lentas ao usar auto-spawn, defina `channels.signal.startupTimeoutMs`.

## Controle de acesso (DMs + grupos)

DMs:

- Padrão: `channels.signal.dmPolicy = "pairing"`.
- Remetentes desconhecidos recebem um código de pareamento; as mensagens são ignoradas até aprovação (códigos expiram após 1 hora).
- Aprovar via:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Pareamento é a troca de token padrão para DMs do Signal. Detalhes: [Pareamento](/channels/pairing)
- Remetentes somente por UUID (de `sourceUuid`) são armazenados como `uuid:<id>` em `channels.signal.allowFrom`.

Grupos:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` controla quem pode acionar em grupos quando `allowlist` está definido.

## Como funciona (comportamento)

- `signal-cli` roda como um daemon; o gateway lê eventos via SSE.
- Mensagens de entrada são normalizadas no envelope de canal compartilhado.
- As respostas sempre retornam ao mesmo número ou grupo.

## Mídia + limites

- Texto de saída é dividido em blocos de `channels.signal.textChunkLimit` (padrão 4000).
- Divisão opcional por nova linha: defina `channels.signal.chunkMode="newline"` para dividir em linhas em branco (limites de parágrafo) antes da divisão por comprimento.
- Anexos suportados (base64 obtido de `signal-cli`).
- Limite padrão de mídia: `channels.signal.mediaMaxMb` (padrão 8).
- Use `channels.signal.ignoreAttachments` para pular o download de mídia.
- O contexto do histórico de grupos usa `channels.signal.historyLimit` (ou `channels.signal.accounts.*.historyLimit`), com fallback para `messages.groupChat.historyLimit`. Defina `0` para desativar (padrão 50).

## Digitação + recibos de leitura

- **Indicadores de digitação**: o OpenClaw envia sinais de digitação via `signal-cli sendTyping` e os renova enquanto uma resposta está em execução.
- **Recibos de leitura**: quando `channels.signal.sendReadReceipts` é true, o OpenClaw encaminha recibos de leitura para DMs permitidas.
- O signal-cli não expõe recibos de leitura para grupos.

## Reações (ferramenta de mensagem)

- Use `message action=react` com `channel=signal`.
- Alvos: remetente E.164 ou UUID (use `uuid:<id>` da saída de pareamento; UUID simples também funciona).
- `messageId` é o timestamp do Signal da mensagem à qual você está reagindo.
- Reações em grupos exigem `targetAuthor` ou `targetAuthorUuid`.

Exemplos:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Configuração:

- `channels.signal.actions.reactions`: habilitar/desabilitar ações de reação (padrão true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` desativa reações do agente (a ferramenta de mensagem `react` gerará erro).
  - `minimal`/`extensive` habilita reações do agente e define o nível de orientação.
- Substituições por conta: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Alvos de entrega (CLI/cron)

- DMs: `signal:+15551234567` (ou E.164 simples).
- DMs por UUID: `uuid:<id>` (ou UUID simples).
- Grupos: `signal:group:<groupId>`.
- Nomes de usuário: `username:<name>` (se suportado pela sua conta do Signal).

## Solução de problemas

Execute esta sequência primeiro:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Depois, confirme o estado de pareamento de DMs se necessário:

```bash
openclaw pairing list signal
```

Falhas comuns:

- Daemon acessível, mas sem respostas: verifique as configurações da conta/daemon (`httpUrl`, `account`) e o modo de recebimento.
- DMs ignoradas: o remetente está pendente de aprovação de pareamento.
- Mensagens de grupo ignoradas: o bloqueio por remetente/menção do grupo impede a entrega.

Para o fluxo de triagem: [/channels/troubleshooting](/channels/troubleshooting).

## Referência de configuração (Signal)

Configuração completa: [Configuração](/gateway/configuration)

Opções do provedor:

- `channels.signal.enabled`: habilitar/desabilitar a inicialização do canal.
- `channels.signal.account`: E.164 da conta do bot.
- `channels.signal.cliPath`: caminho para `signal-cli`.
- `channels.signal.httpUrl`: URL completa do daemon (substitui host/porta).
- `channels.signal.httpHost`, `channels.signal.httpPort`: bind do daemon (padrão 127.0.0.1:8080).
- `channels.signal.autoStart`: auto-spawn do daemon (padrão true se `httpUrl` não estiver definido).
- `channels.signal.startupTimeoutMs`: tempo limite de espera de inicialização em ms (limite 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: pular downloads de anexos.
- `channels.signal.ignoreStories`: ignorar stories do daemon.
- `channels.signal.sendReadReceipts`: encaminhar recibos de leitura.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (padrão: pareamento).
- `channels.signal.allowFrom`: lista de permissões de DMs (E.164 ou `uuid:<id>`). `open` requer `"*"`. O Signal não tem nomes de usuário; use IDs de telefone/UUID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (padrão: lista de permissões).
- `channels.signal.groupAllowFrom`: lista de permissões de remetentes em grupos.
- `channels.signal.historyLimit`: máximo de mensagens de grupo a incluir como contexto (0 desativa).
- `channels.signal.dmHistoryLimit`: limite de histórico de DMs em turnos do usuário. Substituições por usuário: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: tamanho do bloco de saída (caracteres).
- `channels.signal.chunkMode`: `length` (padrão) ou `newline` para dividir em linhas em branco (limites de parágrafo) antes da divisão por comprimento.
- `channels.signal.mediaMaxMb`: limite de mídia de entrada/saída (MB).

Opções globais relacionadas:

- `agents.list[].groupChat.mentionPatterns` (o Signal não oferece suporte a menções nativas).
- `messages.groupChat.mentionPatterns` (fallback global).
- `messages.responsePrefix`.

