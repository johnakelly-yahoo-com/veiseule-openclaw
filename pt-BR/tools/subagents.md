---
summary: "Subagentes: criação de execuções de agentes isoladas que anunciam resultados de volta ao chat solicitante"
read_when:
  - Você quer trabalho em segundo plano/paralelo via o agente
  - Você está alterando sessions_spawn ou a política de ferramentas de subagentes
title: "Subagentes"
---

# Subagentes

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

## Comando

Use o comando de barra `/subagents` para inspecionar e controlar execuções de subagentes para a sessão atual:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` mostra metadados da execução (status, timestamps, id da sessão, caminho da transcrição, limpeza).

Objetivos principais:

- Paralelizar trabalhos de "pesquisa / tarefa longa / ferramenta lenta" sem bloquear a execução principal.
- Manter subagentes isolados por padrão (separação de sessão + sandbox opcional).
- Manter a superfície de ferramentas difícil de usar incorretamente: subagentes **não** recebem ferramentas de sessão por padrão.
- Oferecer suporte a profundidade de aninhamento configurável para padrões de orquestrador.

Each sub-agent has its **own** context and token usage. Para tarefas pesadas ou repetitivas, defina um modelo mais econômico para subagentes e mantenha seu agente principal em um modelo de maior qualidade.
In a multi-agent setup, you can set sub-agent defaults per agent:

## #> [limit] [tools]\`

Parâmetro `model` explícito na chamada `sessions_spawn`

- Inicia a execução de um subagente (`deliver: false`, canal global: `subagent`)
- Em seguida, executa uma etapa de anúncio e publica a resposta do anúncio no canal de chat do solicitante
- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Configuração por agente: `agents.list[].subagents.thinking`

Parâmetros da ferramenta:

- `task`
- `label?` (opcional)
- Criar sob um ID de agente diferente (deve ser permitido)
- Valores de modelo inválidos são ignorados silenciosamente — o subagente é executado com o próximo padrão válido, com um aviso no resultado da ferramenta.
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Automatically aborts the sub-agent run after the specified time
- `"delete"` \\

Lista de permissões:

- `agents.list[].subagents.allowAgents`: lista de ids de agentes que podem ser direcionados via `agentId` (`["*"]` para permitir qualquer um). Padrão: apenas o agente solicitante.

Descoberta:

- Use a ferramenta `agents_list` para descobrir quais IDs de agente estão atualmente permitidos para `sessions_spawn`.

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted.` (mesma pasta).
- `"delete"` arquiva imediatamente após o anúncio
- Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- O arquivamento automático se aplica igualmente a sessões de profundidade 1 e profundidade 2.

## Stopping Sub-Agents

- **No nested spawning:** Sub-agents cannot spawn their own sub-agents. Você pode habilitar um nível de aninhamento definindo `maxSpawnDepth: 2`, o que permite o **padrão de orquestrador**: principal → subagente orquestrador → sub-subagentes trabalhadores.

### Como habilitar

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Níveis de profundidade

| Profundidade | Formato da chave da sessão                                                                                                                                                                                                                                                                                                                                                                                                                | Função                                                                        | Pode gerar subagentes?          |
| ------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- | ------------------------------- |
| 0            | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                                 | _(agente do chamador)_                                     | Sempre                          |
| 1            | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                                          | Subagente (orquestrador quando profundidade 2 é permitida) | Somente se `maxSpawnDepth >= 2` |
| 2            | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // ou ["\*"] para permitir qualquer um&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Gerenciando Subagentes (`/subagents`)                      | Nunca                           |

### Cadeia de anúncio

Os resultados fluem de volta pela cadeia:

1. Trabalhador de profundidade 2 finaliza → anuncia ao seu pai (orquestrador de profundidade 1)
2. O orquestrador de profundidade 1 recebe o announce, sintetiza os resultados, finaliza → anuncia para o principal
3. O agente principal recebe o announce e entrega ao usuário

Cada nível vê apenas announces de seus filhos diretos.

### Tool Policy

- **Profundidade 1 (orquestrador, quando `maxSpawnDepth >= 2`)**: Recebe `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` para poder gerenciar seus filhos. Outras ferramentas de sessão/sistema permanecem negadas.
- **Profundidade 1 (folha, quando `maxSpawnDepth == 1`)**: Sem ferramentas de sessão (comportamento padrão atual).
- **Profundidade 2 (worker folha)**: Sem ferramentas de sessão — `sessions_spawn` é sempre negado na profundidade 2. Não pode gerar novos filhos.

### Per-Agent Overrides

Cada sessão de agente (em qualquer profundidade) pode ter no máximo `maxChildrenPerAgent` (padrão: 5) filhos ativos ao mesmo tempo. Isso evita uma expansão descontrolada a partir de um único orquestrador.

### Parada em cascata

Parar um orquestrador de profundidade 1 interrompe automaticamente todos os seus filhos de profundidade 2:

- `/stop` no chat principal interrompe todos os agentes de profundidade 1 e propaga para seus filhos de profundidade 2.
- `) on the dedicated `subagent\` queue lane.
- `/subagents kill all` interrompe todos os subagentes do solicitante e propaga em cascata.

## Autenticação

A autenticação de subagentes é resolvida por **id do agente**, não pelo tipo de sessão:

- A chave de sessão do subagente é `agent:<agentId>:subagent:<uuid>`.
- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)

The merge is additive — main profiles are always available as fallbacks
Fully isolated auth per sub-agent is not currently supported.

## Announce Status

Os subagentes reportam de volta por meio de uma etapa de announce:

- A etapa de announce é executada dentro da sessão do subagente (não na sessão do solicitante).
- If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted. This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
- When the sub-agent finishes, it announces its findings back to the requester chat.
- As respostas de anúncio preservam o roteamento de thread/tópico quando disponível (threads do Slack, tópicos do Telegram, threads do Matrix).
- As mensagens de announce são normalizadas para um modelo estável:
  - `Status:` derivado do resultado da execução (`success`, `error`, `timeout` ou `unknown`).
  - `Result:` o conteúdo resumido da etapa de announce (ou `(not available)` se estiver ausente).
  - `Notes:` detalhes de erro e outros contextos úteis.
- The announce message includes a status derived from the runtime outcome (not from model output):

Each announce includes a stats line with:

- Tempo de execução (por exemplo, `runtime 5m12s`)
- Uso de tokens (entrada/saída/total)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` e caminho do transcript (para que o agente principal possa buscar o histórico via `sessions_history` ou inspecionar o arquivo no disco)

## Customizing Sub-Agent Tools

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

- Parâmetro `thinking` explícito na chamada `sessions_spawn`
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

Quando `maxSpawnDepth >= 2`, subagentes orquestradores de profundidade 1 recebem adicionalmente `sessions_spawn`, `subagents`, `sessions_list` e `sessions_history` para poder gerenciar seus filhos.

Substituição via configuração:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

## Parâmetros

Subagentes usam uma fila dedicada em processo:

- :subagent:
- {
  agents: {
  defaults: {
  subagents: {
  maxConcurrent: 4, // default: 8
  },
  },
  },
  }

## Interrompendo

- The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.
- `/subagents stop <id>`

## Limitações

- O announce do subagente é **best-effort**. - **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- - **Recursos compartilhados:** Subagentes compartilham o processo do gateway; use `maxConcurrent` como válvula de segurança.
- The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
- O contexto do subagente injeta apenas `AGENTS.md` + `TOOLS.md` (sem `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ou `BOOTSTRAP.md`).
- A profundidade máxima de aninhamento é 5 (intervalo de `maxSpawnDepth`: 1–5). Profundidade 2 é recomendada para a maioria dos casos de uso.
- `maxChildrenPerAgent` limita filhos ativos por sessão (padrão: 5, intervalo: 1–20).
