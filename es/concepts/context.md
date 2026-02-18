---
title: "Contexto"
---

# Contexto

El “contexto” es **todo lo que OpenClaw envía al modelo para una ejecución**. Está limitado por la **ventana de contexto** del modelo (límite de tokens).

Modelo mental para principiantes:

- **System prompt** (construido por OpenClaw): reglas, herramientas, lista de Skills, tiempo/entorno de ejecución y archivos del espacio de trabajo inyectados.
- **Historial de la conversación**: sus mensajes + los mensajes del asistente para esta sesión.
- **Llamadas/resultados de herramientas + adjuntos**: salida de comandos, lecturas de archivos, imágenes/audio, etc.

El contexto _no es lo mismo_ que la “memoria”: la memoria puede almacenarse en disco y recargarse más tarde; el contexto es lo que está dentro de la ventana actual del modelo.

## Inicio rápido (inspeccionar el contexto)

- `/status` → vista rápida de “¿qué tan llena está mi ventana?” + configuración de la sesión.
- `/context list` → qué se inyecta + tamaños aproximados (por archivo + totales).
- `/context detail` → desglose más profundo: tamaños por archivo, por esquema de herramienta, por entrada de Skill y tamaño del system prompt.
- `/usage tokens` → añade un pie de uso por respuesta a las respuestas normales.
- `/compact` → resume el historial antiguo en una entrada compacta para liberar espacio de la ventana.

Vea también: [Slash commands](/tools/slash-commands), [Uso de tokens y costos](/reference/token-use), [Compactación](/concepts/compaction).

## Ejemplo de salida

Los valores varían según el modelo, el proveedor, la política de herramientas y lo que haya en su espacio de trabajo.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Qué cuenta para la ventana de contexto

Todo lo que recibe el modelo cuenta, incluyendo:

- System prompt (todas las secciones).
- Historial de la conversación.
- Llamadas a herramientas + resultados de herramientas.
- Adjuntos/transcripciones (imágenes/audio/archivos).
- Resúmenes de compactación y artefactos de poda.
- “Envolturas” del proveedor o encabezados ocultos (no visibles, pero cuentan).

## Cómo OpenClaw construye el system prompt

El system prompt es **propiedad de OpenClaw** y se reconstruye en cada ejecución. Incluye:

- Lista de herramientas + descripciones breves.
- Lista de Skills (solo metadatos; ver más abajo).
- Ubicación del espacio de trabajo.
- Hora (UTC + hora del usuario convertida si está configurada).
- Metadatos de ejecución (host/SO/modelo/razonamiento).
- Archivos de arranque del espacio de trabajo inyectados bajo **Project Context**.

Desglose completo: [System Prompt](/concepts/system-prompt).

## Archivos del espacio de trabajo inyectados (Project Context)

De forma predeterminada, OpenClaw inyecta un conjunto fijo de archivos del espacio de trabajo (si existen):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (solo en la primera ejecución)

Los archivos grandes se truncan por archivo usando `agents.defaults.bootstrapMaxChars` (valor predeterminado: `20000` caracteres). `/context` muestra los tamaños **sin procesar vs inyectados** y si ocurrió truncamiento.

## Skills: qué se inyecta vs lo que se carga bajo demanda

El system prompt incluye una **lista compacta de Skills** (nombre + descripción + ubicación). Esta lista tiene una sobrecarga real.

Las instrucciones de las Skills _no_ se incluyen de forma predeterminada. Se espera que el modelo `read` el `SKILL.md` de la Skill **solo cuando sea necesario**.

## Herramientas: hay dos costos

Las herramientas afectan al contexto de dos maneras:

1. **Texto de la lista de herramientas** en el system prompt (lo que usted ve como “Tooling”).
2. **Esquemas de herramientas** (JSON). Estos se envían al modelo para que pueda llamar a las herramientas. Cuentan para el contexto aunque no los vea como texto plano.

`/context detail` desglosa los esquemas de herramientas más grandes para que pueda ver qué domina.

## Comandos, directivas y “atajos en línea”

Los slash commands los maneja el Gateway. Hay algunos comportamientos distintos:

- **Comandos independientes**: un mensaje que es solo `/...` se ejecuta como comando.
- **Directivas**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` se eliminan antes de que el modelo vea el mensaje.
  - Los mensajes que solo contienen directivas mantienen la configuración de la sesión.
  - Las directivas en línea dentro de un mensaje normal actúan como pistas por mensaje.
- **Atajos en línea** (solo remitentes en la lista de permitidos): ciertos tokens `/...` dentro de un mensaje normal pueden ejecutarse de inmediato (ejemplo: “hey /status”) y se eliminan antes de que el modelo vea el texto restante.

Detalles: [Slash commands](/tools/slash-commands).

## Sesiones, compactación y poda (qué persiste)

Lo que persiste entre mensajes depende del mecanismo:

- **Historial normal** persiste en la transcripción de la sesión hasta que la política lo compacte o pode.
- **Compactación** persiste un resumen en la transcripción y mantiene intactos los mensajes recientes.
- **Poda** elimina resultados antiguos de herramientas del prompt _en memoria_ para una ejecución, pero no reescribe la transcripción.

Documentación: [Session](/concepts/session), [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning).

## Qué `/context` realmente informa

`/context` prefiere el informe más reciente del system prompt **construido en la ejecución** cuando está disponible:

- `System prompt (run)` = capturado de la última ejecución incrustada (con capacidad de herramientas) y persistido en el almacén de sesiones.
- `System prompt (estimate)` = calculado al vuelo cuando no existe un informe de ejecución (o cuando se ejecuta mediante un backend de CLI que no genera el informe).

En cualquier caso, informa tamaños y los principales contribuyentes; **no** vuelca el system prompt completo ni los esquemas de herramientas.
