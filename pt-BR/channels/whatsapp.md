---
title: "WhatsApp"
---

# WhatsApp (canal web)

Status: apenas WhatsApp Web via Baileys. O Gateway é dono da(s) sessão(ões).

## Configuração rápida (iniciante)

1. Use um **número de telefone separado** se possível (recomendado).
2. Configure o WhatsApp em `~/.openclaw/openclaw.json`.
3. Execute `openclaw channels login` para escanear o QR code (Dispositivos vinculados).
4. Inicie o gateway.

Configuração mínima:

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

## Objetivos

- Múltiplas contas do WhatsApp (multi-account) em um único processo do Gateway.
- Roteamento determinístico: as respostas retornam ao WhatsApp, sem roteamento por modelo.
- O modelo vê contexto suficiente para entender respostas citadas.

## Escritas de configuração

Por padrão, o WhatsApp tem permissão para gravar atualizações de configuração acionadas por `/config set|unset` (requer `commands.config: true`).

Desative com:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Arquitetura (quem é dono do quê)

- **Gateway** é dono do socket do Baileys e do loop do inbox.
- **CLI / app macOS** se comunicam com o gateway; sem uso direto do Baileys.
- **Listener ativo** é necessário para envios de saída; caso contrário, o envio falha imediatamente.

## Obtendo um número de telefone (dois modos)

O WhatsApp exige um número móvel real para verificação. Números VoIP e virtuais geralmente são bloqueados. Existem duas formas suportadas de executar o OpenClaw no WhatsApp:

### Número dedicado (recomendado)

Use um **número de telefone separado** para o OpenClaw. Melhor UX, roteamento limpo, sem peculiaridades de autochat. Configuração ideal: **celular Android reserva/antigo + eSIM**. Deixe-o no Wi‑Fi e na energia, e vincule via QR.

**WhatsApp Business:** Você pode usar o WhatsApp Business no mesmo dispositivo com um número diferente. Ótimo para manter seu WhatsApp pessoal separado — instale o WhatsApp Business e registre o número do OpenClaw ali.

**Exemplo de configuração (número dedicado, allowlist de usuário único):**

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

**Modo de pareamento (opcional):**
Se você quiser pareamento em vez de allowlist, defina `channels.whatsapp.dmPolicy` como `pairing`. Remetentes desconhecidos recebem um código de pareamento; aprove com:
`openclaw pairing approve whatsapp <code>`

### Número pessoal (alternativa)

Alternativa rápida: execute o OpenClaw no **seu próprio número**. Envie mensagens para si mesmo (WhatsApp “Mensagem para você”) para testes, assim você não faz spam para contatos. Espere ler códigos de verificação no seu telefone principal durante a configuração e experimentos. **É necessário habilitar o modo de autochat.**
Quando o assistente pedir seu número pessoal do WhatsApp, informe o telefone a partir do qual você enviará mensagens (o proprietário/remetente), não o número do assistente.

**Exemplo de configuração (número pessoal, autochat):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

As respostas em autochat usam por padrão `[{identity.name}]` quando definido (caso contrário `[openclaw]`)
se `messages.responsePrefix` não estiver definido. Defina explicitamente para personalizar ou desativar
o prefixo (use `""` para removê-lo).

### Dicas para obtenção de número

- **eSIM local** da operadora do seu país (mais confiável)
  - Áustria: [hot.at](https://www.hot.at)
  - Reino Unido: [giffgaff](https://www.giffgaff.com) — SIM grátis, sem contrato
- **SIM pré-pago** — barato, só precisa receber um SMS para verificação

**Evite:** TextNow, Google Voice, a maioria dos serviços de “SMS grátis” — o WhatsApp bloqueia esses agressivamente.

**Dica:** O número só precisa receber um SMS de verificação. Depois disso, as sessões do WhatsApp Web persistem via `creds.json`.

## Por que não Twilio?

- As primeiras versões do OpenClaw suportavam a integração do WhatsApp Business da Twilio.
- Números do WhatsApp Business não são adequados para um assistente pessoal.
- A Meta impõe uma janela de resposta de 24 horas; se você não respondeu nas últimas 24 horas, o número comercial não pode iniciar novas mensagens.
- Uso de alto volume ou “conversas intensas” aciona bloqueios agressivos, porque contas comerciais não são feitas para enviar dezenas de mensagens de assistente pessoal.
- Resultado: entrega pouco confiável e bloqueios frequentes, então o suporte foi removido.

## Login + credenciais

- Comando de login: `openclaw channels login` (QR via Dispositivos vinculados).
- Login multi-account: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Conta padrão (quando `--account` é omitido): `default` se presente; caso contrário, o primeiro id de conta configurado (ordenado).
- Credenciais armazenadas em `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Cópia de backup em `creds.json.bak` (restaurada em caso de corrupção).
- Compatibilidade legada: instalações antigas armazenavam arquivos do Baileys diretamente em `~/.openclaw/credentials/`.
- Logout: `openclaw channels logout` (ou `--account <id>`) apaga o estado de autenticação do WhatsApp (mas mantém o `oauth.json` compartilhado).
- Socket deslogado => erro instruindo a relincar.

## Fluxo de entrada (DM + grupo)

- Eventos do WhatsApp vêm de `messages.upsert` (Baileys).
- Listeners do inbox são removidos no desligamento para evitar acúmulo de handlers de eventos em testes/reinícios.
- Chats de status/broadcast são ignorados.
- Chats diretos usam E.164; grupos usam JID de grupo.
- **Política de DM**: `channels.whatsapp.dmPolicy` controla o acesso a chats diretos (padrão: `pairing`).
  - Pareamento: remetentes desconhecidos recebem um código de pareamento (aprovação via `openclaw pairing approve whatsapp <code>`; códigos expiram após 1 hora).
  - Aberto: requer `channels.whatsapp.allowFrom` incluir `"*"`.
  - Seu número do WhatsApp vinculado é implicitamente confiável, então mensagens próprias ignoram as verificações de `channels.whatsapp.dmPolicy` e `channels.whatsapp.allowFrom`.

### Modo número pessoal (alternativa)

Se você executar o OpenClaw no **seu número pessoal do WhatsApp**, habilite `channels.whatsapp.selfChatMode` (veja o exemplo acima).

Comportamento:

- DMs de saída nunca acionam respostas de pareamento (evita spam a contatos).
- Remetentes desconhecidos de entrada ainda seguem `channels.whatsapp.dmPolicy`.
- O modo de autochat (allowFrom inclui seu número) evita recibos de leitura automáticos e ignora JIDs de menção.
- Recibos de leitura são enviados para DMs que não são autochat.

## Recibos de leitura

Por padrão, o gateway marca mensagens recebidas do WhatsApp como lidas (tiques azuis) assim que são aceitas.

Desativar globalmente:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Desativar por conta:

```json5
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

Notas:

- O modo de autochat sempre ignora recibos de leitura.

## FAQ do WhatsApp: envio de mensagens + pareamento

**O OpenClaw envia mensagens para contatos aleatórios quando eu vinculo o WhatsApp?**  
Não. A política padrão de DM é **pareamento**, então remetentes desconhecidos recebem apenas um código de pareamento e sua mensagem **não é processada**. O OpenClaw só responde a chats que ele recebe ou a envios que você aciona explicitamente (agente/CLI).

**Como funciona o pareamento no WhatsApp?**  
Pareamento é um bloqueio de DM para remetentes desconhecidos:

- A primeira DM de um novo remetente retorna um código curto (a mensagem não é processada).
- Aprove com: `openclaw pairing approve whatsapp <code>` (liste com `openclaw pairing list whatsapp`).
- Os códigos expiram após 1 hora; solicitações pendentes são limitadas a 3 por canal.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. O controle de acesso a DM (`dmPolicy`/`allowFrom`) é global por conta do WhatsApp. Veja [Roteamento Multi‑Agente](/concepts/multi-agent).

**Por que você pede meu número de telefone no assistente?**  
O assistente o usa para definir sua **allowlist/proprietário** para que suas próprias DMs sejam permitidas. Não é usado para envio automático. Se você executar no seu número pessoal do WhatsApp, use esse mesmo número e habilite `channels.whatsapp.selfChatMode`.

## Normalização de mensagens (o que o modelo vê)

- `Body` é o corpo da mensagem atual com envelope.

- O contexto de resposta citada é **sempre anexado**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Metadados de resposta também são definidos:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = corpo citado ou placeholder de mídia
  - `ReplyToSender` = E.164 quando conhecido

- Mensagens recebidas apenas com mídia usam placeholders:
  - `<media:image|video|audio|document|sticker>`

## Grupos

- Grupos mapeiam para sessões `agent:<agentId>:whatsapp:group:<jid>`.
- Política de grupos: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (padrão `allowlist`).
- Modos de ativação:
  - `mention` (padrão): requer @menção ou correspondência por regex.
  - `always`: sempre aciona.
- `/activation mention|always` é apenas para o proprietário e deve ser enviado como mensagem independente.
- Proprietário = `channels.whatsapp.allowFrom` (ou E.164 próprio se não definido).
- **Injeção de histórico** (apenas pendentes):
  - Mensagens recentes _não processadas_ (padrão 50) inseridas em:
    `[Chat messages since your last reply - for context]` (mensagens já na sessão não são reinjetadas)
  - Mensagem atual em:
    `[Current message - respond to this]`
  - Sufixo do remetente anexado: `[from: Name (+E164)]`
- Metadados de grupo em cache por 5 min (assunto + participantes).

## Entrega de respostas (encadeamento)

- O WhatsApp Web envia mensagens padrão (sem encadeamento de resposta citada no gateway atual).
- Tags de resposta são ignoradas neste canal.

## Reações de confirmação (auto-reagir ao receber)

O WhatsApp pode enviar automaticamente reações com emoji às mensagens recebidas imediatamente ao recebê-las, antes de o bot gerar uma resposta. Isso fornece feedback instantâneo aos usuários de que a mensagem foi recebida.

**Configuração:**

```json
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

**Opções:**

- `emoji` (string): Emoji a usar para confirmação (ex.: "👀", "✅", "📨"). Vazio ou omitido = recurso desativado.
- `direct` (boolean, padrão: `true`): Enviar reações em chats diretos/DM.
- `group` (string, padrão: `"mentions"`): Comportamento em grupos:
  - `"always"`: Reagir a todas as mensagens do grupo (mesmo sem @menção)
  - `"mentions"`: Reagir apenas quando o bot for @mencionado
  - `"never"`: Nunca reagir em grupos

**Sobrescrita por conta:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Notas de comportamento:**

- As reações são enviadas **imediatamente** ao receber a mensagem, antes de indicadores de digitação ou respostas do bot.
- Em grupos com `requireMention: false` (ativação: sempre), `group: "mentions"` reagirá a todas as mensagens (não apenas @menções).
- Fire-and-forget: falhas de reação são registradas em log, mas não impedem o bot de responder.
- O JID do participante é incluído automaticamente para reações em grupo.
- O WhatsApp ignora `messages.ackReaction`; use `channels.whatsapp.ackReaction` em vez disso.

## Ferramenta do agente (reações)

- Ferramenta: `whatsapp` com a ação `react` (`chatJid`, `messageId`, `emoji`, `remove` opcional).
- Opcional: `participant` (remetente do grupo), `fromMe` (reagir à sua própria mensagem), `accountId` (multi-account).
- Semântica de remoção de reação: veja [/tools/reactions](/tools/reactions).
- Controle da ferramenta: `channels.whatsapp.actions.reactions` (padrão: habilitado).

## Limites

- Texto de saída é fragmentado em `channels.whatsapp.textChunkLimit` (padrão 4000).
- Fragmentação opcional por nova linha: defina `channels.whatsapp.chunkMode="newline"` para dividir em linhas em branco (limites de parágrafo) antes da fragmentação por comprimento.
- Salvamentos de mídia de entrada são limitados por `channels.whatsapp.mediaMaxMb` (padrão 50 MB).
- Itens de mídia de saída são limitados por `agents.defaults.mediaMaxMb` (padrão 5 MB).

## Envio de saída (texto + mídia)

- Usa listener web ativo; erro se o gateway não estiver em execução.
- Fragmentação de texto: máx. 4k por mensagem (configurável via `channels.whatsapp.textChunkLimit`, opcional `channels.whatsapp.chunkMode`).
- Mídia:
  - Imagem/vídeo/áudio/documento suportados.
  - Áudio enviado como PTT; `audio/ogg` => `audio/ogg; codecs=opus`.
  - Legenda apenas no primeiro item de mídia.
  - Busca de mídia suporta HTTP(S) e caminhos locais.
  - GIFs animados: o WhatsApp espera MP4 com `gifPlayback: true` para loop inline.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: parâmetros `send` incluem `gifPlayback: true`

## Notas de voz (áudio PTT)

O WhatsApp envia áudio como **notas de voz** (bolha PTT).

- Melhores resultados: OGG/Opus. O OpenClaw reescreve `audio/ogg` para `audio/ogg; codecs=opus`.
- `[[audio_as_voice]]` é ignorado para WhatsApp (o áudio já é enviado como nota de voz).

## Limites de mídia + otimização

- Limite padrão de saída: 5 MB (por item de mídia).
- Sobrescrita: `agents.defaults.mediaMaxMb`.
- Imagens são otimizadas automaticamente para JPEG abaixo do limite (redimensionamento + varredura de qualidade).
- Mídia acima do limite => erro; resposta de mídia recua para aviso em texto.

## Heartbeats

- **Heartbeat do Gateway** registra a saúde da conexão (`web.heartbeatSeconds`, padrão 60s).
- **Heartbeat do agente** pode ser configurado por agente (`agents.list[].heartbeat`) ou globalmente
  via `agents.defaults.heartbeat` (fallback quando não há entradas por agente).
  - Usa o prompt de heartbeat configurado (padrão: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + comportamento de pular `HEARTBEAT_OK`.
  - A entrega usa por padrão o último canal utilizado (ou alvo configurado).

## Comportamento de reconexão

- Política de backoff: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Se maxAttempts for atingido, o monitoramento web para (degradado).
- Deslogado => parar e exigir novo vínculo.

## Mapa rápido de configuração

- `channels.whatsapp.dmPolicy` (política de DM: pairing/allowlist/open/disabled).
- `channels.whatsapp.selfChatMode` (configuração no mesmo telefone; bot usa seu número pessoal do WhatsApp).
- `channels.whatsapp.allowFrom` (allowlist de DM). O WhatsApp usa números E.164 (sem nomes de usuário).
- `channels.whatsapp.mediaMaxMb` (limite de salvamento de mídia de entrada).
- `channels.whatsapp.ackReaction` (auto-reação ao receber mensagem: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (configurações por conta + `authDir` opcional).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (limite de mídia de entrada por conta).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (sobrescrita de reação de confirmação por conta).
- `channels.whatsapp.groupAllowFrom` (allowlist de remetentes de grupo).
- `channels.whatsapp.groupPolicy` (política de grupo).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (contexto de histórico de grupo; `0` desativa).
- `channels.whatsapp.dmHistoryLimit` (limite de histórico de DM em turnos do usuário). Sobrescritas por usuário: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (allowlist de grupo + padrões de bloqueio por menção; use `"*"` para permitir todos)
- `channels.whatsapp.actions.reactions` (bloquear reações de ferramenta do WhatsApp).
- `agents.list[].groupChat.mentionPatterns` (ou `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (prefixo de entrada; por conta: `channels.whatsapp.accounts.<accountId>.messagePrefix`; obsoleto: `messages.messagePrefix`)
- `messages.responsePrefix` (prefixo de saída)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (sobrescrita opcional)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (sobrescritas por agente)
- `session.*` (escopo, idle, store, mainKey)
- `web.enabled` (desativar inicialização do canal quando false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Logs + solução de problemas

- Subsistemas: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Arquivo de log: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (configurável).
- Guia de solução de problemas: [Solução de problemas do Gateway](/gateway/troubleshooting).

## Solução de problemas (rápida)

**Não vinculado / login por QR necessário**

- Sintoma: `channels status` mostra `linked: false` ou avisa “Not linked”.
- Correção: execute `openclaw channels login` no host do Gateway e escaneie o QR (WhatsApp → Configurações → Dispositivos vinculados).

**Vinculado, mas desconectado / loop de reconexão**

- Sintoma: `channels status` mostra `running, disconnected` ou avisa “Linked but disconnected”.
- Correção: `openclaw doctor` (ou reinicie o gateway). Se persistir, relinque via `channels login` e inspecione `openclaw logs --follow`.

**Runtime Bun**

- Bun **não é recomendado**. WhatsApp (Baileys) e Telegram são pouco confiáveis no Bun.
  Execute o gateway com **Node**. (Veja a nota de runtime em Primeiros passos.)
