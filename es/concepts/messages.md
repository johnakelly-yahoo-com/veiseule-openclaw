---
summary: "Flujo de mensajes, sesiones, colas y visibilidad del razonamiento"
read_when:
  - Explicar cómo los mensajes entrantes se convierten en respuestas
  - Aclarar sesiones, modos de cola o comportamiento de streaming
  - Documentar la visibilidad del razonamiento y las implicaciones de uso
title: "Mensajes"
---

# Mensajes

Esta página reúne cómo OpenClaw maneja los mensajes entrantes, las sesiones, la cola,
el streaming y la visibilidad del razonamiento.

## Flujo de mensajes (alto nivel)

```
Inbound message
  -> routing/bindings -> session key
  -> queue (if a run is active)
  -> agent run (streaming + tools)
  -> outbound replies (channel limits + chunking)
```

Los controles clave viven en la configuración:

- `messages.*` para prefijos, colas y comportamiento de grupos.
- `agents.defaults.*` para streaming por bloques y valores predeterminados de fragmentación.
- Anulaciones por canal (`channels.whatsapp.*`, `channels.telegram.*`, etc.) para límites y conmutadores de streaming.

Consulte [Configuration](/gateway/configuration) para el esquema completo.

## Dedupe entrante

Los canales pueden volver a entregar el mismo mensaje después de reconexiones. OpenClaw mantiene una
caché de corta duración indexada por canal/cuenta/par/sesión/id de mensaje para que las entregas
duplicadas no desencadenen otra ejecución del agente.

## Debouncing entrante

Mensajes consecutivos rápidos del **mismo remitente** pueden agruparse en un solo turno del agente mediante `messages.inbound`. El debouncing tiene alcance por canal + conversación
y utiliza el mensaje más reciente para el encadenamiento/IDs de la respuesta.

Configuración (valor predeterminado global + anulaciones por canal):

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

Notas:

- El debouncing se aplica a mensajes **solo de texto**; los medios/adjuntos se envían de inmediato.
- Los comandos de control omiten el debouncing para que permanezcan independientes.

## Sesiones y dispositivos

Las sesiones son propiedad del gateway, no de los clientes.

- Los chats directos se colapsan en la clave de sesión principal del agente.
- Los grupos/canales obtienen sus propias claves de sesión.
- El almacén de sesiones y las transcripciones viven en el host del Gateway.

Varios dispositivos/canales pueden mapearse a la misma sesión, pero el historial no se sincroniza por completo de vuelta a cada cliente. Recomendación: use un dispositivo principal para conversaciones largas para evitar contextos divergentes. La UI de Control y la TUI siempre muestran la transcripción de la sesión respaldada por el gateway, por lo que son la fuente de verdad.

Detalles: [Session management](/concepts/session).

## Cuerpos entrantes y contexto del historial

OpenClaw separa el **cuerpo del prompt** del **cuerpo del comando**:

- `Body`: texto del prompt enviado al agente. Esto puede incluir envolturas del canal y
  envolturas opcionales de historial.
- `CommandBody`: texto de usuario sin procesar para el análisis de directivas/comandos.
- `RawBody`: alias heredado de `CommandBody` (se mantiene por compatibilidad).

Cuando un canal proporciona historial, utiliza una envoltura compartida:

- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

Para **chats no directos** (grupos/canales/salas), el **cuerpo del mensaje actual** se antepone con la
etiqueta del remitente (el mismo estilo usado para las entradas de historial). Esto mantiene consistentes los mensajes en tiempo real y los mensajes en cola/historial en el prompt del agente.

Los buffers de historial son **solo pendientes**: incluyen mensajes de grupo que _no_
desencadenaron una ejecución (por ejemplo, mensajes con mención obligatoria) y **excluyen** mensajes
ya presentes en la transcripción de la sesión.

La eliminación de directivas solo se aplica a la sección del **mensaje actual** para que el historial
permanezca intacto. Los canales que envuelven el historial deben establecer `CommandBody` (o
`RawBody`) con el texto original del mensaje y mantener `Body` como el prompt combinado.
Los buffers de historial son configurables mediante `messages.groupChat.historyLimit` (valor predeterminado global)
y anulaciones por canal como `channels.slack.historyLimit` o
`channels.telegram.accounts.<id>.historyLimit` (establezca `0` para deshabilitar).

## Cola y seguimientos

Si ya hay una ejecución activa, los mensajes entrantes pueden ponerse en cola, dirigirse a la
ejecución actual o recopilarse para un turno de seguimiento.

- Configure mediante `messages.queue` (y `messages.queue.byChannel`).
- Modos: `interrupt`, `steer`, `followup`, `collect`, además de variantes con backlog.

Detalles: [Queueing](/concepts/queue).

## Streaming, fragmentación y agrupación

El streaming por bloques envía respuestas parciales a medida que el modelo produce bloques de texto.
La fragmentación respeta los límites de texto del canal y evita dividir código con cercas.

Configuraciones clave:

- `agents.defaults.blockStreamingDefault` (`on|off`, desactivado de forma predeterminada)
- `agents.defaults.blockStreamingBreak` (`text_end|message_end`)
- `agents.defaults.blockStreamingChunk` (`minChars|maxChars|breakPreference`)
- `agents.defaults.blockStreamingCoalesce` (agrupación basada en inactividad)
- `agents.defaults.humanDelay` (pausa de tipo humano entre respuestas por bloques)
- Anulaciones por canal: `*.blockStreaming` y `*.blockStreamingCoalesce` (los canales que no son Telegram requieren `*.blockStreaming: true` explícito)

Detalles: [Streaming + chunking](/concepts/streaming).

## Visibilidad del razonamiento y tokens

OpenClaw puede exponer u ocultar el razonamiento del modelo:

- `/reasoning on|off|stream` controla la visibilidad.
- El contenido de razonamiento aún cuenta para el uso de tokens cuando lo produce el modelo.
- Telegram admite streaming de razonamiento en la burbuja de borrador.

Detalles: [Thinking + reasoning directives](/tools/thinking) y [Token use](/reference/token-use).

## Prefijos, encadenamiento y respuestas

El formato de los mensajes salientes se centraliza en `messages`:

- `messages.responsePrefix`, `channels.<channel>.responsePrefix` y `channels.<channel>.accounts.<id>.responsePrefix` (cascada de prefijos salientes), además de `channels.whatsapp.messagePrefix` (prefijo entrante de WhatsApp)
- Encadenamiento de respuestas mediante `replyToMode` y valores predeterminados por canal

Detalles: [Configuration](/gateway/configuration#messages) y la documentación de los canales.

