---
title: "Subagentes"
---

# Subagentes

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

**Use cases:**

- Research a topic while the main agent continues answering questions
- Run multiple long tasks in parallel (web scraping, code analysis, file processing)
- Delegate tasks to specialized agents in a multi-agent setup

## Inicio Rapido

The simplest way to use sub-agents is to ask your agent naturally:

> "Spawn a sub-agent to research the latest Node.js release notes"

The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.

You can also be explicit about options:

> "Spawn a sub-agent to analyze the server logs from today. Use gpt-5.2 and set a 5-minute timeout."

## Como funciona

<Steps>
  <Step title="Main agent spawns">
    The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
  </Step>
  <Step title="Sub-agent runs in the background">
    A new isolated session is created (`agent:<agentId>:subagent:<uuid>`) on the dedicated `subagent` queue lane.
  </Step>
  <Step title="Result is announced">
    When the sub-agent finishes, it announces its findings back to the requester chat. The main agent posts a natural-language summary.
  </Step>
  <Step title="Session is archived">
    The sub-agent session is auto-archived after 60 minutes (configurable). Transcripts are preserved.
  </Step>
</Steps>

<Tip>
Each sub-agent has its **own** context and token usage. Set a cheaper model for sub-agents to save costs — see [Setting a Default Model](#setting-a-default-model) below.
</Tip>

## Configuração

Sub-agents work out of the box with no configuration. Padrões:

- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Max concurrent: 8
- Auto-archive: after 60 minutes

### Setting a Default Model

Use a cheaper model for sub-agents to save on token costs:

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

### Setting a Default Thinking Level

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### Per-Agent Overrides

In a multi-agent setup, you can set sub-agent defaults per agent:

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### Concorrência

Control how many sub-agents can run at the same time:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

Sub-agents use a dedicated queue lane (`subagent`) separate from the main agent queue, so sub-agent runs don't block inbound replies.

### Auto-Archive

Sub-agent sessions are automatically archived after a configurable period:

```json5
{
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>
Archive renames the transcript to `*.deleted.<timestamp>` (same folder) — transcripts are preserved, not deleted. Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
</Note>

## The `sessions_spawn` Tool

This is the tool the agent calls to create sub-agents.

### Parâmetros

| Parameter           | Tipo                     | Default                                   | Description                                                                                                |
| ------------------- | ------------------------ | ----------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| `task`              | string                   | _(required)_           | What the sub-agent should do                                                                               |
| `rótulo`            | string                   | —                                         | Short label for identification                                                                             |
| `agentId`           | string                   | _(agente do chamador)_ | Criar sob um ID de agente diferente (deve ser permitido)                                |
| `modelo`            | string                   | _(opcional)_           | Substituir o modelo para este subagente                                                                    |
| `thinking`          | string                   | _(opcional)_           | Substituir o nível de raciocínio (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds` | número                   | `0` (sem limite)       | Abortar o subagente após N segundos                                                                        |
| `limpeza`           | `"delete"` \\| `"keep"` | `"keep"`                                  | `"delete"` arquiva imediatamente após o anúncio                                                            |

### Ordem de Resolução do Modelo

O modelo do subagente é resolvido nesta ordem (a primeira correspondência vence):

1. Parâmetro `model` explícito na chamada `sessions_spawn`
2. Configuração por agente: `agents.list[].subagents.model`
3. Padrão global: `agents.defaults.subagents.model`
4. Resolução normal de modelo do agente de destino para essa nova sessão

O nível de raciocínio é resolvido nesta ordem:

1. Parâmetro `thinking` explícito na chamada `sessions_spawn`
2. Configuração por agente: `agents.list[].subagents.thinking`
3. Padrão global: `agents.defaults.subagents.thinking`
4. Caso contrário, nenhuma substituição de raciocínio específica de subagente é aplicada

<Note>
Valores de modelo inválidos são ignorados silenciosamente — o subagente é executado com o próximo padrão válido, com um aviso no resultado da ferramenta.
</Note>

### Criação Entre Agentes

Por padrão, subagentes só podem ser criados sob seu próprio ID de agente. Para permitir que um agente crie subagentes sob outros IDs de agente:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // ou ["*"] para permitir qualquer um
        },
      },
    ],
  },
}
```

<Tip>
Use a ferramenta `agents_list` para descobrir quais IDs de agente estão atualmente permitidos para `sessions_spawn`.
</Tip>

## Gerenciando Subagentes (`/subagents`)

Use o comando de barra `/subagents` para inspecionar e controlar execuções de subagentes para a sessão atual:

| Comando                                    | Descrição                                                                        |
| ------------------------------------------ | -------------------------------------------------------------------------------- |
| `/subagents list`                          | Listar todas as execuções de subagentes (ativas e concluídas) |
| `/subagents stop <id\\|#\\|all>`         | Parar um subagente em execução                                                   |
| `/subagents log <id\\|#> [limit] [tools]` | Visualizar a transcrição do subagente                                            |
| `/subagents info <id\\|#>`                | Mostrar metadados detalhados da execução                                         |
| `/subagents send <id\\|#> <message>`      | Enviar uma mensagem para um subagente em execução                                |

Você pode referenciar subagentes pelo índice da lista (`1`, `2`), prefixo do ID da execução, chave completa da sessão ou `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">
    ```
    /subagents list
    ```

    ````
    ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">
    ```
    /subagents info 1
    ```

    ````
    ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">
    ```
    /subagents log 1 10
    ```

    ````
    Mostra as últimas 10 mensagens da transcrição do subagente. Adicione `tools` para incluir mensagens de chamadas de ferramentas:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">
    ```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Envia uma mensagem para a sessão do subagente em execução e aguarda até 30 segundos por uma resposta.
    ```

  </Accordion>
</AccordionGroup>

## Anúncio (Como os Resultados Retornam)

Quando um subagente termina, ele passa por uma etapa de **anúncio**:

1. The sub-agent's final reply is captured
2. A summary message is sent to the main agent's session with the result, status, and stats
3. The main agent posts a natural-language summary to your chat

As respostas de anúncio preservam o roteamento de thread/tópico quando disponível (threads do Slack, tópicos do Telegram, threads do Matrix).

### Announce Stats

Each announce includes a stats line with:

- Runtime duration
- Uso de tokens (entrada/saída/total)
- Estimated cost (when model pricing is configured via `models.providers.*.models[].cost`)
- Session key, session id, and transcript path

### Announce Status

The announce message includes a status derived from the runtime outcome (not from model output):

- **successful completion** (`ok`) — task completed normally
- **error** — task failed (error details in notes)
- **timeout** — task exceeded `runTimeoutSeconds`
- **unknown** — status could not be determined

<Tip>
If no user-facing announcement is needed, the main-agent summarize step can return `NO_REPLY` and nothing is posted.
This is different from `ANNOUNCE_SKIP`, which is used in agent-to-agent announce flow (`sessions_send`).
</Tip>

## Tool Policy

By default, sub-agents get **all tools except** a set of denied tools that are unsafe or unnecessary for background tasks:

<AccordionGroup>
  <Accordion title="Default denied tools">
    | Denied tool | Reason |
    |-------------|--------|
    | `sessions_list` | Session management — main agent orchestrates |
    | `sessions_history` | Session management — main agent orchestrates |
    | `sessions_send` | Session management — main agent orchestrates |
    | `sessions_spawn` | No nested fan-out (sub-agents cannot spawn sub-agents) |
    | `gateway` | System admin — dangerous from sub-agent |
    | `agents_list` | System admin |
    | `whatsapp_login` | Interactive setup — not a task |
    | `session_status` | Status/scheduling — main agent coordinates |
    | `cron` | Status/scheduling — main agent coordinates |
    | `memory_search` | Pass relevant info in spawn prompt instead |
    | `memory_get` | Pass relevant info in spawn prompt instead |
  </Accordion>
</AccordionGroup>

### Customizing Sub-Agent Tools

You can further restrict sub-agent tools:

```json5
{
  tools: {
    subagents: {
      tools: {
        // deny always wins over allow
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

To restrict sub-agents to **only** specific tools:

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

<Note>
Custom deny entries are **added to** the default deny list. If `allow` is set, only those tools are available (the default deny list still applies on top).
</Note>

## Autenticação

A autenticação de subagentes é resolvida por **id do agente**, não pelo tipo de sessão:

- The auth store is loaded from the target agent's `agentDir`
- The main agent's auth profiles are merged in as a **fallback** (agent profiles win on conflicts)
- The merge is additive — main profiles are always available as fallbacks

<Note>
Fully isolated auth per sub-agent is not currently supported.
</Note>

## Context and System Prompt

Sub-agents receive a reduced system prompt compared to the main agent:

- **Included:** Tooling, Workspace, Runtime sections, plus `AGENTS.md` and `TOOLS.md`
- **Not included:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

The sub-agent also receives a task-focused system prompt that instructs it to stay focused on the assigned task, complete it, and not act as the main agent.

## Stopping Sub-Agents

| Method                 | Effect                                                                    |
| ---------------------- | ------------------------------------------------------------------------- |
| `/stop` in the chat    | Aborts the main session **and** all active sub-agent runs spawned from it |
| `/subagents stop <id>` | Stops a specific sub-agent without affecting the main session             |
| `runTimeoutSeconds`    | Automatically aborts the sub-agent run after the specified time           |

<Note>
`runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
</Note>

## Full Configuration Example

<Accordion title="Complete sub-agent configuration">
```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops can spawn sub-agents under "main"
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agents can't use the browser
      },
    },
  },
}
```
</Accordion>

## Limitações

<Warning>
- **Best-effort announce:** If the gateway restarts, pending announce work is lost.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Recursos compartilhados:** Subagentes compartilham o processo do gateway; use `maxConcurrent` como válvula de segurança.
- **Arquivamento automático é melhor esforço:** Temporizadores de arquivamento pendentes são perdidos na reinicialização do gateway.
</Warning>

## Veja também

- [Session Tools](/concepts/session-tool) — detalhes sobre `sessions_spawn` e outras ferramentas de sessão
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — restrições de ferramentas por agente e sandboxing
- [Configuration](/gateway/configuration) — referência de `agents.defaults.subagents`
- [Queue](/concepts/queue) — como funciona a lane `subagent`
