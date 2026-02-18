---
title: "Barra de menús"
---

# Lógica del estado de la barra de menús

## Qué se muestra

- Mostramos el estado actual de trabajo del agente en el ícono de la barra de menús y en la primera fila de estado del menú.
- El estado de salud se oculta mientras el trabajo está activo; vuelve cuando todas las sesiones están inactivas.
- El bloque “Nodes” del menú enumera solo **dispositivos** (nodos emparejados vía `node.list`), no entradas de cliente/presencia.
- Aparece una sección “Usage” bajo Context cuando hay instantáneas de uso del proveedor disponibles.

## Modelo de estado

- Sesiones: los eventos llegan con `runId` (por ejecución) más `sessionKey` en la carga útil. La sesión “principal” es la clave `main`; si falta, usamos como respaldo la sesión actualizada más recientemente.
- Prioridad: la principal siempre gana. Si la principal está activa, su estado se muestra de inmediato. Si la principal está inactiva, se muestra la sesión no principal activa más reciente. No alternamos en mitad de la actividad; solo cambiamos cuando la sesión actual pasa a inactiva o la principal se vuelve activa.
- Tipos de actividad:
  - `job`: ejecución de comandos de alto nivel (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` con `toolName` y `meta/args`.

## Enum IconState (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (anulación de depuración)

### ActivityKind → glifo

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- predeterminado → 🛠️

### Mapeo visual

- `idle`: criatura normal.
- `workingMain`: insignia con glifo, tinte completo, animación de patas “trabajando”.
- `workingOther`: insignia con glifo, tinte atenuado, sin correteo.
- `overridden`: usa el glifo/tinte elegido independientemente de la actividad.

## Texto de la fila de estado (menú)

- Mientras el trabajo está activo: `<Session role> · <activity label>`
  - Ejemplos: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Cuando está inactivo: vuelve al resumen de salud.

## Ingesta de eventos

- Origen: eventos `agent` del canal de control (`ControlChannel.handleAgentEvent`).
- Campos analizados:
  - `stream: "job"` con `data.state` para inicio/detención.
  - `stream: "tool"` con `data.phase`, `name`, opcional `meta`/`args`.
- Etiquetas:
  - `exec`: primera línea de `args.command`.
  - `read`/`write`: ruta abreviada.
  - `edit`: ruta más tipo de cambio inferido de `meta`/recuentos de diff.
  - respaldo: nombre de la herramienta.

## Anulación de depuración

- Ajustes ▸ Depuración ▸ selector “Icon override”:
  - `System (auto)` (predeterminado)
  - `Working: main` (por tipo de herramienta)
  - `Working: other` (por tipo de herramienta)
  - `Idle`
- Almacenado vía `@AppStorage("iconOverride")`; mapeado a `IconState.overridden`.

## Lista de verificación de pruebas

- Dispare un trabajo de la sesión principal: verifique que el ícono cambie de inmediato y que la fila de estado muestre la etiqueta principal.
- Dispare un trabajo de una sesión no principal mientras la principal está inactiva: el ícono/estado muestra la no principal; se mantiene estable hasta que finaliza.
- Inicie la principal mientras otra está activa: el ícono cambia a la principal al instante.
- Ráfagas rápidas de herramientas: asegúrese de que la insignia no parpadee (gracia de TTL en resultados de herramientas).
- La fila de salud reaparece cuando todas las sesiones están inactivas.
