---
summary: "Subagentes: creación de ejecuciones de agentes aisladas que anuncian resultados de vuelta al chat solicitante"
read_when:
  - Desea trabajo en segundo plano/paralelo mediante el agente
  - Está cambiando sessions_spawn o la política de herramientas de subagentes
title: "Subagentes"
---

# Subagentes

Sub-agents let you run background tasks without blocking the main conversation. When you spawn a sub-agent, it runs in its own isolated session, does its work, and announces the result back to the chat when finished.

## Comando

Usa el comando slash `/subagents` para inspeccionar y controlar ejecuciones de subagentes para la sesión actual:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info` muestra los metadatos de ejecución (estado, marcas de tiempo, id de sesión, ruta de la transcripción, limpieza).

Objetivos principales:

- Paralelizar el trabajo de "research / long task / slow tool" sin bloquear la ejecución principal.
- Mantener los subagentes aislados por defecto (separación de sesión + sandboxing opcional).
- Mantener la superficie de herramientas difícil de usar incorrectamente: los subagentes **no** reciben las herramientas de sesión por defecto.
- Admitir una profundidad de anidamiento configurable para patrones de orquestador.

Each sub-agent has its **own** context and token usage. Para tareas pesadas o repetitivas,
establece un modelo más económico para los subagentes y mantén tu agente principal en un modelo de mayor calidad.
In a multi-agent setup, you can set sub-agent defaults per agent:

## #> [limit] [tools]\`

Explicit `model` parameter in the `sessions_spawn` call

- Global default: `agents.defaults.subagents.thinking`
- Luego ejecuta un paso de anuncio y publica la respuesta de anuncio en el canal de chat del solicitante
- Model: target agent’s normal model selection (unless `subagents.model` is set)
- Per-agent config: `agents.list[].subagents.thinking`

Parámetros de la herramienta:

- `task`
- `etiqueta`
- Spawn under a different agent id (must be allowed)
- Invalid model values are silently skipped — the sub-agent runs on the next valid default with a warning in the tool result.
- Thinking: no sub-agent override (unless `subagents.thinking` is set)
- Abort the sub-agent after N seconds
- `"delete"` \\

Lista de permitidos:

- Per-agent config: `agents.list[].subagents.model` Predeterminado: solo el agente solicitante.

Descubrimiento:

- <Tip>
Usa la herramienta `agents_list` para descubrir qué IDs de agente están permitidos actualmente para `sessions_spawn`.
</Tip>

Auto-Archive

- Sub-agent sessions are automatically archived after a configurable period:
- Archive renames the transcript to `*.deleted.` (misma carpeta).
- `"delete"` archives immediately after announce
- Auto-archive timers are best-effort; pending timers are lost if the gateway restarts.
- `runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
- El archivado automático se aplica por igual a las sesiones de profundidad 1 y 2.

## Stopping Sub-Agents

- **No nested spawning:** Sub-agents cannot spawn their own sub-agents. Puedes habilitar un nivel de anidamiento estableciendo `maxSpawnDepth: 2`, lo que permite el **patrón de orquestador**: principal → subagente orquestador → sub-subagentes trabajadores.

### Cómo habilitar

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

### Niveles de profundidad

| Profundidad | Forma de la clave de sesión                                                                                                                                                                                                                                                                                                                                                                                                  | Rol                                                                        | ¿Puede crear subagentes?     |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ---------------------------- |
| 0           | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                    | Per-Agent Overrides                                                        | Siempre                      |
| 1           | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                             | Subagente (orquestador cuando se permite profundidad 2) | Solo si `maxSpawnDepth >= 2` |
| 2           | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // or ["\*"] to allow any&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Gestión de Subagentes (`/subagents`)                    | Nunca                        |

### Cadena de anuncios

Los resultados fluyen de vuelta hacia arriba en la cadena:

1. El worker de profundidad 2 finaliza → anuncia a su padre (orquestador de profundidad 1)
2. El orquestador de profundidad 1 recibe el anuncio, sintetiza los resultados, finaliza → anuncia al principal
3. El agente principal recibe el anuncio y lo entrega al usuario

Cada nivel solo ve los anuncios de sus hijos directos.

### Política de herramientas

- **Profundidad 1 (orquestador, cuando `maxSpawnDepth >= 2`)**: Recibe `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` para poder gestionar a sus hijos. Otras herramientas de sesión/sistema permanecen denegadas.
- **Profundidad 1 (hoja, cuando `maxSpawnDepth == 1`)**: Sin herramientas de sesión (comportamiento predeterminado actual).
- **Profundidad 2 (worker hoja)**: Sin herramientas de sesión — `sessions_spawn` siempre está denegado en profundidad 2. No puede generar más hijos.

### Cross-Agent Spawning

Cada sesión de agente (en cualquier profundidad) puede tener como máximo `maxChildrenPerAgent` (por defecto: 5) hijos activos al mismo tiempo. Esto evita una expansión descontrolada desde un único orquestador.

### Detención en cascada

Detener un orquestador de profundidad 1 detiene automáticamente a todos sus hijos de profundidad 2:

- `/stop` en el chat principal detiene todos los agentes de profundidad 1 y se propaga en cascada a sus hijos de profundidad 2.
- `) on the dedicated `subagent\` queue lane.
- `/subagents kill all` detiene todos los subagentes del solicitante y se propaga en cascada.

## Autenticación

La autenticación de subagentes se resuelve por **id de agente**, no por tipo de sesión:

- La clave de sesión del subagente es `agent:<agentId>:subagent:<uuid>`.
- El almacén de autenticación se carga desde el `agentDir` del agente objetivo
- Los perfiles de autenticación del agente principal se combinan como **respaldo** (los perfiles del agente ganan en caso de conflicto)

Nota: la fusión es aditiva, por lo que los perfiles principales siempre están disponibles como alternativa.
Fully isolated auth per sub-agent is not currently supported.

## Anuncio

Los subagentes informan mediante un paso de anuncio:

- El paso de anuncio se ejecuta dentro de la sesión del subagente (no en la sesión del solicitante).
- Si el subagente responde exactamente `ANNOUNCE_SKIP`, no se publica nada.
- When the sub-agent finishes, it announces its findings back to the requester chat.
- Las respuestas de anuncio conservan el enrutamiento de hilo/tema cuando está disponible (hilos de Slack, temas de Telegram, hilos de Matrix).
- Los mensajes de anuncio se normalizan a una plantilla estable:
  - `Status:` derivado del resultado de la ejecución (`success`, `error`, `timeout` o `unknown`).
  - `Result:` el contenido resumido del paso de anuncio (o `(not available)` si falta).
  - `Notes:` detalles del error y otro contexto útil.
- El mensaje de anuncio incluye un estado derivado del resultado de la ejecución (no de la salida del modelo):

Las cargas útiles de anuncio incluyen una línea de estadísticas al final (incluso cuando están envueltas):

- Tiempo de ejecución (por ejemplo, `runtime 5m12s`)
- Uso de tokens (entrada/salida/total)
- Costo estimado (cuando el precio del modelo está configurado vía `models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` y la ruta de la transcripción (para que el agente principal pueda obtener el historial mediante `sessions_history` o inspeccionar el archivo en disco)

## Personalizar herramientas de subagentes

Para restringir a los subagentes **solo** a herramientas específicas:

- Explicit `thinking` parameter in the `sessions_spawn` call
- `runTimeoutSeconds`
- `sessions_send`
- The `sessions_spawn` Tool

Cuando `maxSpawnDepth >= 2`, los subagentes orquestadores de profundidad 1 reciben adicionalmente `sessions_spawn`, `subagents`, `sessions_list` y `sessions_history` para poder gestionar a sus hijos.

Sobrescribir mediante configuración:

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

## Concurrencia

Los subagentes utilizan un carril de cola dedicado en el propio proceso:

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

## Detención

- The agent will call the `sessions_spawn` tool behind the scenes. When the sub-agent finishes, it announces its findings back into your chat.
- `/subagents stop <id>`

## Limitaciones

- El anuncio del subagente es **best-effort**. Si el gateway se reinicia, el trabajo pendiente de "announce back" se pierde.
- - **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- The main agent calls `sessions_spawn` with a task description. The call is **non-blocking** — the main agent gets back `{ status: "accepted", runId, childSessionKey }` immediately.
- El contexto del subagente solo inyecta `AGENTS.md` + `TOOLS.md` (no `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ni `BOOTSTRAP.md`).
- La profundidad máxima de anidamiento es 5 (rango de `maxSpawnDepth`: 1–5). Se recomienda la profundidad 2 para la mayoría de los casos de uso.
- `maxChildrenPerAgent` limita los hijos activos por sesión (por defecto: 5, rango: 1–20).
