---
summary: "Regras de gerenciamento de sessão, chaves e persistência para chats"
read_when:
  - Modificando o tratamento ou armazenamento de sessões
title: "Gerenciamento de Sessões"
---

# Gerenciamento de Sessões

O OpenClaw trata **uma sessão de chat direto por agente** como primária. Chats diretos colapsam para `agent:<agentId>:<mainKey>` (padrão `main`), enquanto chats em grupo/canal recebem suas próprias chaves. `session.mainKey` é respeitado.

Use `session.dmScope` para controlar como **mensagens diretas** são agrupadas:

- `main` (padrão): todos os DMs compartilham a sessão principal para continuidade.
- `per-peer`: isola por id do remetente entre canais.
- `per-channel-peer`: isola por canal + remetente (recomendado para caixas de entrada multiusuário).
- `per-account-channel-peer`: isola por conta + canal + remetente (recomendado para caixas de entrada multicontra).
  Use `session.identityLinks` para mapear ids de pares prefixados pelo provedor para uma identidade canônica, de modo que a mesma pessoa compartilhe uma sessão de DM entre canais ao usar `per-peer`, `per-channel-peer` ou `per-account-channel-peer`.

## Modo DM seguro (recomendado para configurações multiusuário)

> **Aviso de Segurança:** Se seu agente pode receber DMs de **várias pessoas**, você deve considerar fortemente habilitar o modo DM seguro. Sem ele, todos os usuários compartilham o mesmo contexto de conversa, o que pode vazar informações privadas entre usuários.

**Exemplo do problema com as configurações padrão:**

- Alice (`<SENDER_A>`) envia uma mensagem ao seu agente sobre um assunto privado (por exemplo, uma consulta médica)
- Bob (`<SENDER_B>`) envia uma mensagem ao seu agente perguntando "Sobre o que estávamos falando?"
- Como ambos os DMs compartilham a mesma sessão, o modelo pode responder ao Bob usando o contexto anterior da Alice.

**A correção:** Defina `dmScope` para isolar sessões por usuário:

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    // Secure DM mode: isolate DM context per channel + sender.
    dmScope: "per-channel-peer",
  },
}
```

**Quando habilitar isso:**

- Você tem aprovações de pareamento para mais de um remetente
- Você usa uma lista de permissões de DM com várias entradas
- Você define `dmPolicy: "open"`
- Vários números de telefone ou contas podem enviar mensagens ao seu agente

Notas:

- O padrão é `dmScope: "main"` para continuidade (todos os DMs compartilham a sessão principal). Isso é adequado para configurações de usuário único.
- Para caixas de entrada multicontra no mesmo canal, prefira `per-account-channel-peer`.
- Se a mesma pessoa entrar em contato com você por vários canais, use `session.identityLinks` para colapsar as sessões de DM em uma identidade canônica.
- Você pode verificar suas configurações de DM com `openclaw security audit` (veja [security](/cli/security)).

## O Gateway é a fonte da verdade

Todo o estado da sessão é **de propriedade do gateway** (o OpenClaw “mestre”). Clientes de UI (app macOS, WebChat etc.) devem consultar o gateway para listas de sessões e contagens de tokens em vez de ler arquivos locais.

- Em **modo remoto**, o armazenamento de sessões que importa fica no host do Gateway remoto, não no seu Mac.
- As contagens de tokens exibidas nas UIs vêm dos campos do armazenamento do gateway (`inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`). Os clientes não analisam transcrições JSONL para “ajustar” totais.

## Onde o estado vive

- No **host do Gateway**:
  - Arquivo de armazenamento: `~/.openclaw/agents/<agentId>/sessions/sessions.json` (por agente).
- Transcrições: `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` (sessões de tópicos do Telegram usam `.../<SessionId>-topic-<threadId>.jsonl`).
- O armazenamento é um mapa `sessionKey -> { sessionId, updatedAt, ... }`. Excluir entradas é seguro; elas são recriadas sob demanda.
- Entradas de grupo podem incluir `displayName`, `channel`, `subject`, `room` e `space` para rotular sessões nas UIs.
- Entradas de sessão incluem metadados `origin` (rótulo + dicas de roteamento) para que as UIs possam explicar de onde veio uma sessão.
- O OpenClaw **não** lê pastas de sessão legadas do Pi/Tau.

## Poda de sessões

O OpenClaw remove **resultados antigos de ferramentas** do contexto em memória imediatamente antes das chamadas ao LLM por padrão.
Isso **não** reescreve o histórico JSONL.
Isso **não** reescreve o histórico JSONL. Veja [/concepts/session-pruning](/concepts/session-pruning).

## Liberação de memória antes da compactação

Quando uma sessão se aproxima da compactação automática, o OpenClaw pode executar uma **liberação silenciosa de memória**
que lembra o modelo de gravar notas duráveis em disco. Isso só é executado quando
o workspace é gravável. Veja [Memory](/concepts/memory) e
[Compaction](/concepts/compaction).

## Mapeamento de transportes → chaves de sessão

- Chats diretos seguem `session.dmScope` (padrão `main`).
  - `main`: `agent:<agentId>:<mainKey>` (continuidade entre dispositivos/canais).
    - Vários números de telefone e canais podem mapear para a mesma chave principal do agente; eles atuam como transportes para uma única conversa.
  - `per-peer`: `agent:<agentId>:dm:<peerId>`.
  - `per-channel-peer`: `agent:<agentId>:<channel>:dm:<peerId>`.
  - `per-account-channel-peer`: `agent:<agentId>:<channel>:<accountId>:dm:<peerId>` (accountId tem padrão `default`).
  - Se `session.identityLinks` corresponder a um id de par prefixado pelo provedor (por exemplo, `telegram:123`), a chave canônica substitui `<peerId>` para que a mesma pessoa compartilhe uma sessão entre canais.
- Chats em grupo isolam o estado: `agent:<agentId>:<channel>:group:<id>` (salas/canais usam `agent:<agentId>:<channel>:channel:<id>`).
  - Tópicos de fórum do Telegram acrescentam `:topic:<threadId>` ao id do grupo para isolamento.
  - Chaves legadas `group:<id>` ainda são reconhecidas para migração.
- Contextos de entrada ainda podem usar `group:<id>`; o canal é inferido a partir de `Provider` e normalizado para a forma canônica `agent:<agentId>:<channel>:group:<id>`.
- Outras fontes:
  - Jobs de cron: `cron:<job.id>`
  - Webhooks: `hook:<uuid>` (a menos que explicitamente definido pelo hook)
  - Execuções de nó: `node-<nodeId>`

## Ciclo de vida

- Política de redefinição: sessões são reutilizadas até expirarem, e a expiração é avaliada na próxima mensagem de entrada.
- Redefinição diária: padrão **4:00 AM no horário local do host do Gateway**. Uma sessão fica obsoleta quando sua última atualização é anterior ao horário da redefinição diária mais recente.
- Redefinição por inatividade (opcional): `idleMinutes` adiciona uma janela deslizante de inatividade. Quando redefinições diária e por inatividade estão configuradas, **a que expirar primeiro** força uma nova sessão.
- Legado apenas por inatividade: se você definir `session.idleMinutes` sem nenhuma configuração `session.reset`/`resetByType`, o OpenClaw permanece no modo apenas por inatividade para compatibilidade retroativa.
- Substituições por tipo (opcional): `resetByType` permite substituir a política para sessões `direct`, `group` e `thread` (thread = threads do Slack/Discord, tópicos do Telegram, threads do Matrix quando fornecidos pelo conector).
- Substituições por canal (opcional): `resetByChannel` substitui a política de redefinição para um canal (aplica-se a todos os tipos de sessão para esse canal e tem precedência sobre `reset`/`resetByType`).
- Gatilhos de redefinição: `/new` ou `/reset` exatos (mais quaisquer extras em `resetTriggers`) iniciam um novo id de sessão e encaminham o restante da mensagem. `/new <model>` aceita um alias de modelo, `provider/model` ou nome do provedor (correspondência aproximada) para definir o novo modelo da sessão. Se `/new` ou `/reset` for enviado sozinho, o OpenClaw executa um curto turno de saudação “hello” para confirmar a redefinição.
- Redefinição manual: exclua chaves específicas do armazenamento ou remova a transcrição JSONL; a próxima mensagem as recria.
- Jobs de cron isolados sempre geram um novo `sessionId` por execução (sem reutilização por inatividade).

## Política de envio (opcional)

Bloqueia a entrega para tipos específicos de sessão sem listar ids individuais.

```json5
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
      ],
      default: "allow",
    },
  },
}
```

Substituição em tempo de execução (somente proprietário):

- `/send on` → permitir para esta sessão
- `/send off` → negar para esta sessão
- `/send inherit` → limpar substituição e usar regras de configuração
  Envie como mensagens independentes para que sejam registradas.

## Configuração (exemplo opcional de renomeação)

```json5
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // keep group keys separate
    dmScope: "main", // DM continuity (set per-channel-peer/per-account-channel-peer for shared inboxes)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // Defaults: mode=daily, atHour=4 (gateway host local time).
      // If you also set idleMinutes, whichever expires first wins.
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## Inspeção

- `openclaw status` — mostra o caminho do armazenamento e sessões recentes.
- `openclaw sessions --json` — despeja todas as entradas (filtre com `--active <minutes>`).
- `openclaw gateway call sessions.list --params '{}'` — busca sessões do gateway em execução (use `--url`/`--token` para acesso a gateway remoto).
- Envie `/status` como mensagem independente no chat para ver se o agente está acessível, quanto do contexto da sessão está sendo usado, alternâncias atuais de thinking/verbose e quando suas credenciais do WhatsApp web foram atualizadas pela última vez (ajuda a identificar necessidade de relink).
- Envie `/context list` ou `/context detail` para ver o que está no prompt do sistema e nos arquivos de workspace injetados (e os maiores contribuintes de contexto).
- Envie `/stop` como mensagem independente para abortar a execução atual, limpar followups enfileirados para essa sessão e interromper quaisquer execuções de subagentes geradas a partir dela (a resposta inclui a contagem interrompida).
- Envie `/compact` (instruções opcionais) como mensagem independente para resumir contexto antigo e liberar espaço de janela. Veja [/concepts/compaction](/concepts/compaction).
- Transcrições JSONL podem ser abertas diretamente para revisar turnos completos.

## Dicas

- Mantenha a chave primária dedicada a tráfego 1:1; deixe que grupos mantenham suas próprias chaves.
- Ao automatizar a limpeza, exclua chaves individuais em vez de todo o armazenamento para preservar contexto em outros lugares.

## Metadados de origem da sessão

Cada entrada de sessão registra de onde veio (best-effort) em `origin`:

- `label`: rótulo humano (resolvido a partir do rótulo da conversa + assunto do grupo/canal)
- `provider`: id de canal normalizado (incluindo extensões)
- `from`/`to`: ids de roteamento brutos do envelope de entrada
- `accountId`: id da conta do provedor (quando multicontra)
- `threadId`: id de thread/tópico quando o canal oferece suporte
  Os campos de origem são preenchidos para mensagens diretas, canais e grupos. Se um
  conector apenas atualiza o roteamento de entrega (por exemplo, para manter uma sessão
  principal de DM atualizada), ele ainda deve fornecer contexto de entrada para que a
  sessão mantenha seus metadados explicativos. Extensões podem fazer isso enviando `ConversationLabel`,
  `GroupSubject`, `GroupChannel`, `GroupSpace` e `SenderName` no contexto de entrada
  e chamando `recordSessionMetaFromInbound` (ou passando o mesmo contexto
  para `updateLastRoute`).
