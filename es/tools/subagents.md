---
title: "Subagentes"
---

# Sub-agentes

Los sub-agentes son ejecuciones de agentes en segundo plano generadas desde una ejecución de agente existente. Se ejecutan en su propia sesión (`agent:<agentId>:subagent:<uuid>`) y, cuando finalizan, **anuncian** su resultado de vuelta al canal de chat solicitante.

## Comando slash

Usa `/subagents` para inspeccionar o controlar ejecuciones de subagentes para la **sesión actual**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` muestra metadatos de la ejecución (estado, marcas de tiempo, id de sesión, ruta de la transcripción, limpieza).

Objetivos principales:

- Paralelizar trabajo de “investigación / tarea larga / herramienta lenta” sin bloquear la ejecución principal.
- Mantener los sub-agentes aislados por defecto (separación de sesión + sandboxing opcional).
- Mantener la superficie de herramientas difícil de usar incorrectamente: los sub-agentes **no** obtienen herramientas de sesión por defecto.
- Soportar profundidad de anidamiento configurable para patrones de orquestador.

Nota de costo: cada sub-agente tiene su **propio** contexto y uso de tokens. Para tareas pesadas o repetitivas, configura un modelo más económico para los sub-agentes y mantén tu agente principal en un modelo de mayor calidad. Puedes configurarlo mediante `agents.defaults.subagents.model` o anulaciones por agente.

## Herramienta

Usa `sessions_spawn`:

- Inicia una ejecución de sub-agente (`deliver: false`, carril global: `subagent`)
- Luego ejecuta un paso de anuncio y publica la respuesta de anuncio en el canal de chat solicitante
- Modelo predeterminado: hereda del llamador a menos que configures `agents.defaults.subagents.model` (o `agents.list[].subagents.model` por agente); un `sessions_spawn.model` explícito siempre tiene prioridad.
- Thinking predeterminado: hereda del llamador a menos que configures `agents.defaults.subagents.thinking` (o `agents.list[].subagents.thinking` por agente); un `sessions_spawn.thinking` explícito siempre tiene prioridad.

Parámetros de la herramienta:

- `task` (requerido)
- `label?` (opcional)
- `agentId?` (opcional; generar bajo otro id de agente si está permitido)
- `model?` (opcional; anula el modelo del sub-agente; valores inválidos se omiten y el sub-agente se ejecuta con el modelo predeterminado con una advertencia en el resultado de la herramienta)
- `thinking?` (opcional; anula el nivel de thinking para la ejecución del sub-agente)
- `runTimeoutSeconds?` (predeterminado `0`; cuando se establece, la ejecución del sub-agente se aborta después de N segundos)
- `cleanup?` (`delete|keep`, predeterminado `keep`)

Lista de permitidos:

- `agents.list[].subagents.allowAgents`: lista de ids de agente que pueden ser objetivo mediante `agentId` (`["*"]` para permitir cualquiera). Predeterminado: solo el agente solicitante.

Descubrimiento:

- Usa `agents_list` para ver qué ids de agente están permitidos actualmente para `sessions_spawn`.

Auto-archivo:

- Las sesiones de sub-agentes se archivan automáticamente después de `agents.defaults.subagents.archiveAfterMinutes` (predeterminado: 60).
- El archivado usa `sessions.delete` y renombra la transcripción a `*.deleted.<timestamp>` (misma carpeta).
- `cleanup: "delete"` archiva inmediatamente después del anuncio (aún conserva la transcripción mediante el renombrado).
- El auto-archivo es best-effort; los temporizadores pendientes se pierden si la puerta de enlace se reinicia.
- `runTimeoutSeconds` **no** auto-archiva; solo detiene la ejecución. La sesión permanece hasta el auto-archivo.
- El auto-archivo aplica por igual a sesiones de profundidad 1 y 2.

## Sub-agentes anidados

Por defecto, los sub-agentes no pueden generar sus propios sub-agentes (`maxSpawnDepth: 1`). Puedes habilitar un nivel de anidamiento estableciendo `maxSpawnDepth: 2`, lo que permite el **patrón de orquestador**: principal → sub-agente orquestador → sub-sub-agentes trabajadores.

### Cómo habilitar

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // permitir que los sub-agentes generen hijos (predeterminado: 1)
        maxChildrenPerAgent: 5, // máximo de hijos activos por sesión de agente (predeterminado: 5)
        maxConcurrent: 8, // límite global de concurrencia del carril (predeterminado: 8)
      },
    },
  },
}
```

### Niveles de profundidad

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | Agente principal                              | Siempre                      |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agente (orquestador cuando se permite depth 2) | Solo si `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agente (trabajador hoja)              | Nunca                        |

### Cadena de anuncio

Los resultados fluyen de regreso por la cadena:

1. El trabajador de profundidad 2 finaliza → anuncia a su padre (orquestador de profundidad 1)
2. El orquestador de profundidad 1 recibe el anuncio, sintetiza resultados, finaliza → anuncia al principal
3. El agente principal recibe el anuncio y lo entrega al usuario

Cada nivel solo ve anuncios de sus hijos directos.

### Política de herramientas por profundidad

- **Depth 1 (orquestador, cuando `maxSpawnDepth >= 2`)**: Obtiene `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` para poder gestionar a sus hijos. Otras herramientas de sesión/sistema permanecen denegadas.
- **Depth 1 (hoja, cuando `maxSpawnDepth == 1`)**: Sin herramientas de sesión (comportamiento predeterminado actual).
- **Depth 2 (trabajador hoja)**: Sin herramientas de sesión — `sessions_spawn` siempre está denegado en profundidad 2. No puede generar más hijos.

### Límite de generación por agente

Cada sesión de agente (en cualquier profundidad) puede tener como máximo `maxChildrenPerAgent` (predeterminado: 5) hijos activos al mismo tiempo. Esto evita una expansión descontrolada desde un único orquestador.

### Detención en cascada

Detener un orquestador de profundidad 1 detiene automáticamente todos sus hijos de profundidad 2:

- `/stop` en el chat principal detiene todos los agentes de profundidad 1 y se propaga a sus hijos de profundidad 2.
- `/subagents kill <id>` detiene un sub-agente específico y se propaga a sus hijos.
- `/subagents kill all` detiene todos los sub-agentes del solicitante y se propaga.

## Autenticación

La autenticación de sub-agentes se resuelve por **id de agente**, no por tipo de sesión:

- La clave de sesión del sub-agente es `agent:<agentId>:subagent:<uuid>`.
- El almacén de autenticación se carga desde el `agentDir` de ese agente.
- Los perfiles de autenticación del agente principal se combinan como **respaldo**; los perfiles del agente prevalecen en caso de conflicto.

Nota: la combinación es aditiva, por lo que los perfiles del agente principal siempre están disponibles como respaldo. El aislamiento completo de autenticación por agente aún no está soportado.

## Anuncio

Los sub-agentes informan de vuelta mediante un paso de anuncio:

- El paso de anuncio se ejecuta dentro de la sesión del sub-agente (no en la sesión del solicitante).
- Si el sub-agente responde exactamente `ANNOUNCE_SKIP`, no se publica nada.
- De lo contrario, la respuesta de anuncio se publica en el canal de chat solicitante mediante una llamada de seguimiento `agent` (`deliver=true`).
- Las respuestas de anuncio conservan el enrutamiento de hilo/tema cuando está disponible (hilos de Slack, temas de Telegram, hilos de Matrix).
- Los mensajes de anuncio se normalizan a una plantilla estable:
  - `Status:` derivado del resultado de la ejecución (`success`, `error`, `timeout` o `unknown`).
  - `Result:` el contenido resumido del paso de anuncio (o `(not available)` si falta).
  - `Notes:` detalles del error y otro contexto útil.
- `Status` no se infiere de la salida del modelo; proviene de señales del resultado en tiempo de ejecución.

Las cargas de anuncio incluyen una línea de estadísticas al final (incluso cuando están envueltas):

- Tiempo de ejecución (por ejemplo, `runtime 5m12s`)
- Uso de tokens (entrada/salida/total)
- Costo estimado cuando el precio del modelo está configurado (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` y ruta de la transcripción (para que el agente principal pueda obtener el historial mediante `sessions_history` o inspeccionar el archivo en disco)

## Política de herramientas (herramientas de sub-agente)

Por defecto, los sub-agentes obtienen **todas las herramientas excepto** herramientas de sesión y herramientas del sistema:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Cuando `maxSpawnDepth >= 2`, los sub-agentes orquestadores de profundidad 1 reciben adicionalmente `sessions_spawn`, `subagents`, `sessions_list` y `sessions_history` para poder gestionar a sus hijos.

Anular mediante configuración:

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

## Concurrencia

Los sub-agentes usan un carril de cola dedicado en proceso:

- Nombre del carril: `subagent`
- Concurrencia: `agents.defaults.subagents.maxConcurrent` (predeterminado `8`)

## Detención

- Enviar `/stop` en el chat del solicitante aborta la sesión del solicitante y detiene cualquier ejecución activa de sub-agentes generada desde ella, propagándose a hijos anidados.
- `/subagents kill <id>` detiene un sub-agente específico y se propaga a sus hijos.

## Limitaciones

- El anuncio de sub-agente es **best-effort**. Si la puerta de enlace se reinicia, el trabajo pendiente de “anunciar de vuelta” se pierde.
- Los sub-agentes aún comparten los recursos del mismo proceso de la puerta de enlace; trata `maxConcurrent` como una válvula de seguridad.
- `sessions_spawn` siempre es no bloqueante: devuelve `{ status: "accepted", runId, childSessionKey }` inmediatamente.
- El contexto del sub-agente solo inyecta `AGENTS.md` + `TOOLS.md` (no `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` ni `BOOTSTRAP.md`).
- La profundidad máxima de anidamiento es 5 (`maxSpawnDepth` rango: 1–5). Se recomienda profundidad 2 para la mayoría de los casos de uso.
- `maxChildrenPerAgent` limita los hijos activos por sesión (predeterminado: 5, rango: 1–20).
