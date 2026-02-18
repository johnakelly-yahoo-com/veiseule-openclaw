---
title: "Subagentes"
---

# Subagentes

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

**Use cases:**

- Research a topic while the main agent continues answering questions
- Run multiple long tasks in parallel (web scraping, code analysis, file processing)
- Delegate tasks to specialized agents in a multi-agent setup

## Inicio rápido

The simplest way to use sub-agents is to ask your agent naturally:

> "Spawn a sub-agent to research the latest Node.js release notes"

The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.

You can also be explicit about options:

> "Spawn a sub-agent to analyze the server logs from today. Use gpt-5.2 and set a 5-minute timeout."

## Cómo funciona

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

## Configuración

Sub-agents work out of the box with no configuration. Valores predeterminados:

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

### Concurrencia

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

### Parámetros

| Parameter           | Tipo                     | Predeterminado                        | Descripción                                                                                       |
| ------------------- | ------------------------ | ------------------------------------- | ------------------------------------------------------------------------------------------------- |
| `task`              | string                   | _(required)_       | What the sub-agent should do                                                                      |
| `etiqueta`          | string                   | —                                     | Short label for identification                                                                    |
| `agentId`           | string                   | _(caller's agent)_ | Spawn under a different agent id (must be allowed)                             |
| `modelo`            | string                   | _(optional)_       | Override the model for this sub-agent                                                             |
| `thinking`          | string                   | _(optional)_       | Override thinking level (`off`, `low`, `medium`, `high`, etc.) |
| `runTimeoutSeconds` | number                   | `0` (no limit)     | Abort the sub-agent after N seconds                                                               |
| `limpieza`          | `"delete"` \\| `"keep"` | `"keep"`                              | `"delete"` archives immediately after announce                                                    |

### Model Resolution Order

The sub-agent model is resolved in this order (first match wins):

1. Explicit `model` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.model`
3. Global default: `agents.defaults.subagents.model`
4. Target agent’s normal model resolution for that new session

Thinking level is resolved in this order:

1. Explicit `thinking` parameter in the `sessions_spawn` call
2. Per-agent config: `agents.list[].subagents.thinking`
3. Global default: `agents.defaults.subagents.thinking`
4. Otherwise no sub-agent-specific thinking override is applied

<Note>
Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
</Note>

### Cross-Agent Spawning

By default, sub-agents can only spawn under their own agent id. Para permitir que un agente genere subagentes bajo otros IDs de agente:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>Usa la herramienta `agents_list` para descubrir qué IDs de agente están permitidos actualmente para `sessions_spawn`.</Tip>

## Gestión de Subagentes (`/subagents`)

Usa el comando slash `/subagents` para inspeccionar y controlar ejecuciones de subagentes para la sesión actual:

| Comando                                    | Descripción                                                                           |
| ------------------------------------------ | ------------------------------------------------------------------------------------- |
| `/subagents list`                          | Listar todas las ejecuciones de subagentes (activas y completadas) |
| `/subagents stop <id\\|#\\|all>`         | Detener un subagente en ejecución                                                     |
| `/subagents log <id\\|#> [limit] [tools]` | Ver la transcripción del subagente                                                    |
| `/subagents info <id\\|#>`                | Mostrar metadatos detallados de la ejecución                                          |
| `/subagents send <id\\|#> <message>`      | Enviar un mensaje a un subagente en ejecución                                         |

Puedes referenciar subagentes por índice de la lista (`1`, `2`), prefijo del ID de ejecución, clave de sesión completa o `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">```
/subagents list
```

    ````
    ```
    🧭 Subagentes (sesión actual)
    Activos: 1 · Completados: 2
    1) ✅ · registros de investigación · 2m31s · ejecución a1b2c3d4 · agente:main:subagent:...
    2) ✅ · comprobar dependencias · 45s · ejecución e5f6g7h8 · agente:main:subagent:...
    3) 🔄 · desplegar staging · 1m12s · ejecución i9j0k1l2 · agente:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Solicitud de detención para desplegar staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">```
/subagents info 1
```

    ````
    ```
    ℹ️ Información del subagente
    Estado: ✅
    Etiqueta: registros de investigación
    Tarea: Investigar los últimos registros de errores del servidor y resumir los hallazgos
    Ejecución: a1b2c3d4-...
    Sesión: agent:main:subagent:...
    Tiempo de ejecución: 2m31s
    Limpieza: mantener
    Resultado: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">```
/subagents log 1 10
```

    ````
    Muestra los últimos 10 mensajes de la transcripción del subagente. Añade `tools` para incluir mensajes de llamadas a herramientas:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">```
/subagents send 3 "Also check the staging environment"
```

    ```
    Envía un mensaje a la sesión del subagente en ejecución y espera hasta 30 segundos por una respuesta.
    ```

  </Accordion>
</AccordionGroup>

## Anunciar (Cómo regresan los resultados)

Cuando un subagente finaliza, pasa por un paso de **anuncio**:

1. Se captura la respuesta final del subagente
2. Se envía un mensaje de resumen a la sesión del agente principal con el resultado, estado y estadísticas
3. El agente principal publica un resumen en lenguaje natural en tu chat

Las respuestas de anuncio conservan el enrutamiento de hilo/tema cuando está disponible (hilos de Slack, temas de Telegram, hilos de Matrix).

### Estadísticas del anuncio

Cada anuncio incluye una línea de estadísticas con:

- Duración del tiempo de ejecución
- Uso de tokens (entrada/salida/total)
- Costo estimado (cuando el precio del modelo está configurado vía `models.providers.*.models[].cost`)
- Clave de sesión, ID de sesión y ruta de la transcripción

### Estado del anuncio

El mensaje de anuncio incluye un estado derivado del resultado de la ejecución (no de la salida del modelo):

- **finalización exitosa** (`ok`) — la tarea se completó normalmente
- **error** — la tarea falló (detalles del error en notas)
- **timeout** — la tarea excedió `runTimeoutSeconds`
- **desconocido** — no se pudo determinar el estado

<Tip>
Si no se necesita un anuncio visible para el usuario, el paso de resumen del agente principal puede devolver `NO_REPLY` y no se publica nada.
Esto es diferente de `ANNOUNCE_SKIP`, que se usa en el flujo de anuncio entre agentes (`sessions_send`).
</Tip>

## Política de herramientas

De forma predeterminada, los subagentes obtienen **todas las herramientas excepto** un conjunto de herramientas denegadas que son inseguras o innecesarias para tareas en segundo plano:

<AccordionGroup>
  <Accordion title="Default denied tools">| Herramienta denegada | Motivo |
|-------------|--------|
| `sessions_list` | Gestión de sesiones — el agente principal orquesta |
| `sessions_history` | Gestión de sesiones — el agente principal orquesta |
| `sessions_send` | Gestión de sesiones — el agente principal orquesta |
| `sessions_spawn` | Sin fan-out anidado (los subagentes no pueden generar subagentes) |
| `gateway` | Administración del sistema — peligroso desde un subagente |
| `agents_list` | Administración del sistema |
| `whatsapp_login` | Configuración interactiva — no es una tarea |
| `session_status` | Estado/programación — el agente principal coordina |
| `cron` | Estado/programación — el agente principal coordina |
| `memory_search` | Pasa la información relevante en el prompt de creación |
| `memory_get` | Pasa la información relevante en el prompt de creación |</Accordion>
</AccordionGroup>

### Personalizar herramientas de subagentes

Puedes restringir aún más las herramientas de los subagentes:

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

Para restringir a los subagentes **solo** a herramientas específicas:

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
Las entradas de denegación personalizadas se **añaden a** la lista de denegación predeterminada. Si se establece `allow`, solo esas herramientas están disponibles (la lista de denegación predeterminada aún se aplica encima).
</Note>

## Autenticación

La autenticación de subagentes se resuelve por **id de agente**, no por tipo de sesión:

- El almacén de autenticación se carga desde el `agentDir` del agente objetivo
- Los perfiles de autenticación del agente principal se combinan como **respaldo** (los perfiles del agente ganan en caso de conflicto)
- La combinación es aditiva — los perfiles del agente principal siempre están disponibles como respaldos

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

## Limitaciones

<Warning>
- **Anuncio de mejor esfuerzo:** Si la puerta de enlace se reinicia, el trabajo de anuncio pendiente se pierde.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- **Auto-archive is best-effort:** Pending archive timers are lost on gateway restart.
</Warning>

## Véase también

- [Session Tools](/concepts/session-tool) — details on `sessions_spawn` and other session tools
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — per-agent tool restrictions and sandboxing
- [Configuration](/gateway/configuration) — `agents.defaults.subagents` reference
- [Queue](/concepts/queue) — how the `subagent` lane works
