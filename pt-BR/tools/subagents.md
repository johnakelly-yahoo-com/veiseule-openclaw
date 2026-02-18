---
title: "Subagentes"
---

# Subagentes

Subagentes são execuções de agente em segundo plano iniciadas a partir de uma execução existente. Eles rodam em sua própria sessão (`agent:<agentId>:subagent:<uuid>`) e, quando finalizam, **anunciam** seu resultado de volta ao canal de chat solicitante.

## Comando de barra

Use `/subagents` para inspecionar ou controlar execuções de subagentes para a **sessão atual**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` mostra metadados da execução (status, timestamps, session id, caminho da transcrição, limpeza).

Objetivos principais:

- Paralelizar trabalhos de “pesquisa / tarefa longa / ferramenta lenta” sem bloquear a execução principal.
- Manter subagentes isolados por padrão (separação de sessão + sandbox opcional).
- Manter a superfície de ferramentas difícil de usar incorretamente: subagentes **não** recebem ferramentas de sessão por padrão.
- Suportar profundidade de aninhamento configurável para padrões de orquestração.

Observação de custo: cada subagente tem seu **próprio** contexto e uso de tokens. Para tarefas pesadas ou repetitivas, defina um modelo mais barato para subagentes e mantenha seu agente principal em um modelo de maior qualidade. Você pode configurar isso via `agents.defaults.subagents.model` ou substituições por agente.

## Ferramenta

Use `sessions_spawn`:

- Inicia uma execução de subagente (`deliver: false`, lane global: `subagent`)
- Em seguida executa uma etapa de anúncio e publica a resposta de anúncio no canal de chat solicitante
- Modelo padrão: herda do chamador, a menos que você defina `agents.defaults.subagents.model` (ou `agents.list[].subagents.model`); um `sessions_spawn.model` explícito ainda prevalece.
- Thinking padrão: herda do chamador, a menos que você defina `agents.defaults.subagents.thinking` (ou `agents.list[].subagents.thinking`); um `sessions_spawn.thinking` explícito ainda prevalece.

Parâmetros da ferramenta:

- `task` (obrigatório)
- `label?` (opcional)
- `agentId?` (opcional; criar sob outro id de agente se permitido)
- `model?` (opcional; substitui o modelo do subagente; valores inválidos são ignorados e o subagente roda no modelo padrão com um aviso no resultado da ferramenta)
- `thinking?` (opcional; substitui o nível de raciocínio para a execução do subagente)
- `runTimeoutSeconds?` (padrão `0`; quando definido, a execução do subagente é abortada após N segundos)
- `cleanup?` (`delete|keep`, padrão `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: lista de ids de agente que podem ser alvo via `agentId` (`["*"]` para permitir qualquer um). Padrão: apenas o agente solicitante.

Descoberta:

- Use `agents_list` para ver quais ids de agente estão atualmente permitidos para `sessions_spawn`.

Arquivamento automático:

- Sessões de subagentes são arquivadas automaticamente após `agents.defaults.subagents.archiveAfterMinutes` (padrão: 60).
- O arquivamento usa `sessions.delete` e renomeia a transcrição para `*.deleted.<timestamp>` (mesma pasta).
- `cleanup: "delete"` arquiva imediatamente após o anúncio (ainda mantém a transcrição via rename).
- O arquivamento automático é melhor esforço; temporizadores pendentes são perdidos se o gateway reiniciar.
- `runTimeoutSeconds` **não** arquiva automaticamente; apenas interrompe a execução. A sessão permanece até o arquivamento automático.
- O arquivamento automático se aplica igualmente a sessões de profundidade 1 e 2.

## Subagentes Aninhados

Por padrão, subagentes não podem criar seus próprios subagentes (`maxSpawnDepth: 1`). Você pode habilitar um nível de aninhamento definindo `maxSpawnDepth: 2`, o que permite o **padrão de orquestrador**: principal → subagente orquestrador → sub-subagentes trabalhadores.

### Como habilitar

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // permitir que subagentes criem filhos (padrão: 1)
        maxChildrenPerAgent: 5, // máximo de filhos ativos por sessão de agente (padrão: 5)
        maxConcurrent: 8, // limite global de concorrência da lane (padrão: 8)
      },
    },
  },
}
```

### Níveis de profundidade

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Agente principal                              | Sempre                       |
| 1     | `agent:<id>:subagent:<uuid>`                 | Subagente (orquestrador quando depth 2 permitido) | Apenas se `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-subagente (worker final)                  | Nunca                        |

### Cadeia de anúncio

Os resultados fluem de volta pela cadeia:

1. Worker de profundidade 2 finaliza → anuncia ao seu pai (orquestrador de profundidade 1)
2. Orquestrador de profundidade 1 recebe o anúncio, sintetiza os resultados, finaliza → anuncia ao principal
3. O agente principal recebe o anúncio e entrega ao usuário

Cada nível vê apenas anúncios de seus filhos diretos.

### Política de ferramentas por profundidade

- **Depth 1 (orquestrador, quando `maxSpawnDepth >= 2`)**: Recebe `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` para poder gerenciar seus filhos. Outras ferramentas de sessão/sistema permanecem negadas.
- **Depth 1 (leaf, quando `maxSpawnDepth == 1`)**: Sem ferramentas de sessão (comportamento padrão atual).
- **Depth 2 (worker final)**: Sem ferramentas de sessão — `sessions_spawn` é sempre negada na profundidade 2. Não pode criar mais filhos.

### Limite de criação por agente

Cada sessão de agente (em qualquer profundidade) pode ter no máximo `maxChildrenPerAgent` (padrão: 5) filhos ativos ao mesmo tempo. Isso evita fan-out descontrolado a partir de um único orquestrador.

### Parada em cascata

Parar um orquestrador de profundidade 1 automaticamente para todos os seus filhos de profundidade 2:

- `/stop` no chat principal para todos os agentes de profundidade 1 e propaga para seus filhos de profundidade 2.
- `/subagents kill <id>` para um subagente específico e propaga para seus filhos.
- `/subagents kill all` para todos os subagentes do solicitante e propaga.

## Autenticação

A autenticação do subagente é resolvida por **id do agente**, não pelo tipo de sessão:

- A chave de sessão do subagente é `agent:<agentId>:subagent:<uuid>`.
- O auth store é carregado do `agentDir` desse agente.
- Os perfis de autenticação do agente principal são mesclados como **fallback**; perfis do agente sobrescrevem os do principal em caso de conflito.

Observação: a mesclagem é aditiva, então os perfis do principal estão sempre disponíveis como fallback. Autenticação totalmente isolada por agente ainda não é suportada.

## Anúncio

Subagentes reportam de volta por meio de uma etapa de anúncio:

- A etapa de anúncio roda dentro da sessão do subagente (não na sessão do solicitante).
- Se o subagente responder exatamente `ANNOUNCE_SKIP`, nada é publicado.
- Caso contrário, a resposta de anúncio é publicada no canal de chat solicitante via uma chamada `agent` subsequente (`deliver=true`).
- Respostas de anúncio preservam o roteamento de thread/tópico quando disponível (threads do Slack, tópicos do Telegram, threads do Matrix).
- Mensagens de anúncio são normalizadas em um template estável:
  - `Status:` derivado do resultado da execução (`success`, `error`, `timeout` ou `unknown`).
  - `Result:` o conteúdo resumido da etapa de anúncio (ou `(not available)` se ausente).
  - `Notes:` detalhes de erro e outros contextos úteis.
- `Status` não é inferido da saída do modelo; vem dos sinais de resultado em runtime.

Payloads de anúncio incluem uma linha de estatísticas ao final (mesmo quando encapsulados):

- Runtime (ex.: `runtime 5m12s`)
- Uso de tokens (input/output/total)
- Custo estimado quando o pricing do modelo está configurado (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` e caminho da transcrição (para que o agente principal possa buscar histórico via `sessions_history` ou inspecionar o arquivo em disco)

## Política de Ferramentas (ferramentas de subagente)

Por padrão, subagentes recebem **todas as ferramentas exceto ferramentas de sessão** e ferramentas de sistema:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Quando `maxSpawnDepth >= 2`, subagentes orquestradores de depth 1 recebem adicionalmente `sessions_spawn`, `subagents`, `sessions_list` e `sessions_history` para gerenciar seus filhos.

Substitua via configuração:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // if allow is set, it becomes allow-only (deny still wins)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concorrência

Subagentes usam uma lane de fila dedicada no processo:

- Nome da lane: `subagent`
- Concorrência: `agents.defaults.subagents.maxConcurrent` (padrão `8`)

## Interrupção

- Enviar `/stop` no chat solicitante aborta a sessão solicitante e interrompe quaisquer execuções de subagentes ativas iniciadas a partir dela, propagando para filhos aninhados.
- `/subagents kill <id>` interrompe um subagente específico e propaga para seus filhos.

## Limitações

- O anúncio de subagente é **best-effort**. Se o gateway reiniciar, trabalhos pendentes de “announce back” são perdidos.
- Subagentes ainda compartilham os mesmos recursos do processo do gateway; trate `maxConcurrent` como uma válvula de segurança.
- `sessions_spawn` é sempre não bloqueante: retorna `{ status: "accepted", runId, childSessionKey }` imediatamente.
- O contexto do subagente injeta apenas `AGENTS.md` + `TOOLS.md` (sem `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ou `BOOTSTRAP.md`).
- A profundidade máxima de aninhamento é 5 (intervalo de `maxSpawnDepth`: 1–5). Profundidade 2 é recomendada para a maioria dos casos de uso.
- `maxChildrenPerAgent` limita filhos ativos por sessão (padrão: 5, intervalo: 1–20).

