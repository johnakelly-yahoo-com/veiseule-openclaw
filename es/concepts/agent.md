---
summary: "Runtime del agente (pi-mono integrado), contrato del espacio de trabajo y arranque de sesión"
read_when:
  - Al cambiar el runtime del agente, el arranque del espacio de trabajo o el comportamiento de la sesión
title: "Runtime del agente"
---

# Runtime del agente 🤖

OpenClaw ejecuta un único runtime de agente integrado derivado de **pi-mono**.

## Espacio de trabajo (obligatorio)

OpenClaw utiliza un único directorio de espacio de trabajo del agente (`agents.defaults.workspace`) como el **único** directorio de trabajo (`cwd`) del agente para herramientas y contexto.

Recomendado: use `openclaw setup` para crear `~/.openclaw/openclaw.json` si falta e inicializar los archivos del espacio de trabajo.

Diseño completo del espacio de trabajo + guía de copias de seguridad: [Espacio de trabajo del agente](/concepts/agent-workspace)

Si `agents.defaults.sandbox` está habilitado, las sesiones que no son principales pueden sobrescribir esto con
espacios de trabajo por sesión bajo `agents.defaults.sandbox.workspaceRoot` (ver
[Configuración del Gateway](/gateway/configuration)).

## Archivos de arranque (inyectados)

Dentro de `agents.defaults.workspace`, OpenClaw espera estos archivos editables por el usuario:

- `AGENTS.md` — instrucciones operativas + “memoria”
- `SOUL.md` — persona, límites, tono
- `TOOLS.md` — notas de herramientas mantenidas por el usuario (p. ej., `imsg`, `sag`, convenciones)
- `BOOTSTRAP.md` — ritual de primera ejecución de una sola vez (se elimina tras completarse)
- `IDENTITY.md` — nombre/vibra/emoji del agente
- `USER.md` — perfil del usuario + forma de tratamiento preferida

En el primer turno de una sesión nueva, OpenClaw inyecta el contenido de estos archivos directamente en el contexto del agente.

Los archivos en blanco se omiten. Los archivos grandes se recortan y se truncan con un marcador para mantener los prompts livianos (lea el archivo para ver el contenido completo).

Si falta un archivo, OpenClaw inyecta una sola línea de marcador de “archivo faltante” (y `openclaw setup` creará una plantilla predeterminada segura).

`BOOTSTRAP.md` solo se crea para un **espacio de trabajo completamente nuevo** (sin otros archivos de arranque presentes). Si lo elimina después de completar el ritual, no debería recrearse en reinicios posteriores.

Para deshabilitar por completo la creación de archivos de arranque (para espacios de trabajo presembrados), establezca:

```json5
{ agent: { skipBootstrap: true } }
```

## Herramientas integradas

Las herramientas principales (leer/ejecutar/editar/escribir y herramientas del sistema relacionadas) siempre están disponibles,
sujetas a la política de herramientas. `apply_patch` es opcional y está controlado por
`tools.exec.applyPatch`. `TOOLS.md` **no** controla qué herramientas existen; es
orientación sobre cómo _usted_ quiere que se usen.

## Habilidades

OpenClaw carga Skills desde tres ubicaciones (el espacio de trabajo gana en conflictos de nombre):

- Incluidas (entregadas con la instalación)
- Gestionadas/locales: `~/.openclaw/skills`
- Espacio de trabajo: `<workspace>/skills`

Las Skills pueden estar controladas por config/env (ver `skills` en [Configuración del Gateway](/gateway/configuration)).

## Integración de pi-mono

OpenClaw reutiliza partes del código base de pi-mono (modelos/herramientas), pero **la gestión de sesiones, el descubrimiento y el cableado de herramientas pertenecen a OpenClaw**.

- No hay runtime de agente de pi-coding.
- No se consultan configuraciones de `~/.pi/agent` ni `<workspace>/.pi`.

## Sesiones

Las transcripciones de sesiones se almacenan como JSONL en:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

El ID de sesión es estable y lo elige OpenClaw.
Las carpetas de sesiones heredadas de Pi/Tau **no** se leen.

## Dirección durante el streaming

Cuando el modo de cola es `steer`, los mensajes entrantes se inyectan en la ejecución actual.
La cola se verifica **después de cada llamada a herramienta**; si hay un mensaje en cola,
las llamadas a herramientas restantes del mensaje actual del asistente se omiten (resultados de herramienta con error
"Skipped due to queued user message."), luego el mensaje del usuario en cola
se inyecta antes de la siguiente respuesta del asistente.

Cuando el modo de cola es `followup` o `collect`, los mensajes entrantes se retienen hasta que
finaliza el turno actual, y luego comienza un nuevo turno del agente con las cargas en cola. Vea
[Cola](/concepts/queue) para el modo y el comportamiento de debounce/cap.

El streaming por bloques envía los bloques del asistente completados tan pronto como finalizan; está
**desactivado por defecto** (`agents.defaults.blockStreamingDefault: "off"`).
Ajuste el límite mediante `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; por defecto text_end).
Controle la fragmentación suave de bloques con `agents.defaults.blockStreamingChunk` (por defecto
800–1200 caracteres; prefiere saltos de párrafo, luego saltos de línea; las oraciones al final).
Una los fragmentos transmitidos con `agents.defaults.blockStreamingCoalesce` para reducir
spam de una sola línea (fusión basada en inactividad antes del envío). Los canales que no son Telegram requieren
`*.blockStreaming: true` explícito para habilitar respuestas por bloques.
Los resúmenes verbosos de herramientas se emiten al inicio de la herramienta (sin debounce); la UI de control
transmite la salida de herramientas mediante eventos del agente cuando está disponible.
Más detalles: [Streaming + fragmentación](/concepts/streaming).

## Referencias de modelos

Las referencias de modelos en la configuración (por ejemplo `agents.defaults.model` y `agents.defaults.models`) se analizan dividiendo por el **primer** `/`.

- Use `provider/model` al configurar modelos.
- Si el ID del modelo contiene `/` (estilo OpenRouter), incluya el prefijo del proveedor (ejemplo: `openrouter/moonshotai/kimi-k2`).
- Si omite el proveedor, OpenClaw trata la entrada como un alias o como un modelo para el **proveedor predeterminado** (solo funciona cuando no hay `/` en el ID del modelo).

## Configuración (mínima)

Como mínimo, establezca:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (muy recomendado)

---

_Siguiente: [Chats grupales](/channels/group-messages)_ 🦞
