---
summary: "Plan: aislar browser act:evaluate de la cola de Playwright usando CDP, con plazos de extremo a extremo y una resolución de referencias más segura"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor Plan

## Contexto

`act:evaluate` ejecuta JavaScript proporcionado por el usuario en la página. Actualmente se ejecuta a través de Playwright
(`page.evaluate` o `locator.evaluate`). Playwright serializa los comandos CDP por página, por lo que un
evaluate bloqueado o de larga duración puede bloquear la cola de comandos de la página y hacer que cada acción posterior
en esa pestaña parezca "bloqueada".

El PR #13498 añade una red de seguridad pragmática (evaluate con límite, propagación de abort y recuperación
en el mejor de los casos). Este documento describe una refactorización más amplia que hace que `act:evaluate` esté inherentemente
aislado de Playwright para que un evaluate bloqueado no pueda bloquear las operaciones normales de Playwright.

## Objetivos

- `act:evaluate` no puede bloquear permanentemente acciones posteriores del navegador en la misma pestaña.
- Los timeouts son una única fuente de verdad de extremo a extremo para que quien llama pueda confiar en un presupuesto.
- Abort y timeout se tratan de la misma manera en HTTP y en el despacho en proceso.
- Se admite la selección de elementos para evaluate sin dejar de usar Playwright para todo.
- Mantener compatibilidad hacia atrás para los llamadores y payloads existentes.

## No objetivos

- Reemplazar todas las acciones del navegador (click, type, wait, etc.) con implementaciones CDP.
- Eliminar la red de seguridad existente introducida en el PR #13498 (sigue siendo un fallback útil).
- Introducir nuevas capacidades inseguras más allá del control existente `browser.evaluateEnabled`.
- Añadir aislamiento de procesos (proceso/hilo worker) para evaluate. Si después de esta refactorización seguimos viendo estados bloqueados difíciles de recuperar,
  esa es una idea para una fase posterior.

## Arquitectura actual (Por qué se bloquea)

A alto nivel:

- Los llamadores envían `act:evaluate` al servicio de control del navegador.
- El controlador de la ruta llama a Playwright para ejecutar el JavaScript.
- Playwright serializa los comandos de la página, por lo que un evaluate que nunca termina bloquea la cola.
- Una cola atascada significa que las operaciones posteriores de click/type/wait en la pestaña pueden parecer bloqueadas.

## Arquitectura propuesta

### 1. Propagación de plazos

Introduce un concepto único de presupuesto y deriva todo a partir de él:

- El llamador establece `timeoutMs` (o un plazo en el futuro).
- El timeout de la solicitud externa, la lógica del manejador de rutas y el presupuesto de ejecución dentro de la página
  utilizan el mismo presupuesto, con un pequeño margen adicional cuando sea necesario para la sobrecarga de serialización.
- La cancelación se propaga como un `AbortSignal` en todas partes para que sea coherente.

Dirección de implementación:

- Añadir un pequeño helper (por ejemplo `createBudget({ timeoutMs, signal })`) que devuelva:
  - `signal`: el AbortSignal vinculado
  - `deadlineAtMs`: plazo absoluto
  - `remainingMs()`: presupuesto restante para operaciones hijas
- Usar este helper en:
  - `src/browser/client-fetch.ts` (HTTP y despacho en proceso)
  - `src/node-host/runner.ts` (ruta de proxy)
  - implementaciones de acciones del navegador (Playwright y CDP)

### 2. Motor de Evaluate separado (ruta CDP)

Añadir una implementación de evaluate basada en CDP que no comparta la cola de comandos por página de Playwright. La propiedad clave es que el transporte de evaluate es una conexión WebSocket independiente
y una sesión CDP separada adjunta al target.

Dirección de implementación:

- Nuevo módulo, por ejemplo `src/browser/cdp-evaluate.ts`, que:
  - Se conecta al endpoint CDP configurado (socket a nivel de navegador).
  - Usa `Target.attachToTarget({ targetId, flatten: true })` para obtener un `sessionId`.
  - Ejecuta ya sea:
    - `Runtime.evaluate` para evaluate a nivel de página, o
    - `DOM.resolveNode` más `Runtime.callFunctionOn` para evaluate a nivel de elemento.
  - En caso de timeout o cancelación:
    - Envía `Runtime.terminateExecution` en modo best-effort para la sesión.
    - Cierra el WebSocket y devuelve un error claro.

Notas:

- Esto sigue ejecutando JavaScript en la página, por lo que la terminación puede tener efectos secundarios. La ventaja
  es que no bloquea la cola de Playwright y se puede cancelar en la capa de transporte eliminando la sesión CDP.

### 3. Historia de Ref (Segmentación de elementos sin una reescritura completa)

La parte difícil es la segmentación de elementos. CDP necesita un manejador del DOM o `backendDOMNodeId`, mientras que
hoy en día la mayoría de las acciones del navegador utilizan locators de Playwright basados en refs de snapshots.

Enfoque recomendado: mantener las refs existentes, pero adjuntar un id opcional resolvible por CDP.

#### 3.1 Extender la información de referencias almacenadas

Extender los metadatos de la referencia de rol almacenada para incluir opcionalmente un id de CDP:

- Actual: `{ role, name, nth }`
- Propuesto: `{ role, name, nth, backendDOMNodeId?: number }`

Esto mantiene todas las acciones existentes basadas en Playwright funcionando y permite que CDP evaluate acepte
el mismo valor de `ref` cuando `backendDOMNodeId` esté disponible.

#### 3.2 Completar backendDOMNodeId en el momento del snapshot

Al generar un snapshot de roles:

1. Generar el mapa de referencias de roles existente como hoy (role, name, nth).
2. Obtener el árbol AX a través de CDP (`Accessibility.getFullAXTree`) y calcular un mapa paralelo de
   `(role, name, nth) -> backendDOMNodeId` usando las mismas reglas de manejo de duplicados.
3. Fusionar el id de vuelta en la información de referencia almacenada para la pestaña actual.

Si el mapeo falla para una referencia, dejar `backendDOMNodeId` como undefined. Esto hace que la funcionalidad sea
de mejor esfuerzo y segura de desplegar.

#### 3.3 Comportamiento de evaluate con Ref

En `act:evaluate`:

- Si `ref` está presente y tiene `backendDOMNodeId`, ejecutar element evaluate mediante CDP.
- Si `ref` está presente pero no tiene `backendDOMNodeId`, volver a la ruta de Playwright (con
  la red de seguridad).

Vía de escape opcional:

- Extender la estructura de la solicitud para aceptar `backendDOMNodeId` directamente para usuarios avanzados (y
  para depuración), manteniendo `ref` como la interfaz principal.

### 4. Mantener una ruta de recuperación de último recurso

Incluso con CDP evaluate, existen otras formas de bloquear una pestaña o una conexión. Mantener los
mecanismos de recuperación existentes (terminar la ejecución + desconectar Playwright) como último recurso
para:

- usuarios heredados
- entornos donde la conexión CDP esté bloqueada
- casos límite inesperados de Playwright

## Plan de implementación (una sola iteración)

### Entregables

- Un motor de evaluate basado en CDP que se ejecute fuera de la cola de comandos por página de Playwright.
- Un único presupuesto integral de timeout/abort utilizado de forma consistente por los llamadores y los handlers.
- Metadatos de ref que puedan incluir opcionalmente `backendDOMNodeId` para element evaluate.
- `act:evaluate` prioriza el motor CDP cuando es posible y recurre a Playwright cuando no lo es.
- Pruebas que demuestren que un evaluate bloqueado no impide acciones posteriores.
- Logs/métricas que hagan visibles los fallos y los fallbacks.

### Lista de verificación de implementación

1. Añadir un helper compartido de "budget" para vincular `timeoutMs` + `AbortSignal` ascendente en:
   - un único `AbortSignal`
   - una fecha límite absoluta
   - un helper `remainingMs()` para operaciones posteriores
2. Actualizar todas las rutas de llamada para usar ese helper de modo que `timeoutMs` signifique lo mismo en todas partes:
   - `src/browser/client-fetch.ts` (HTTP y dispatch en proceso)
   - `src/node-host/runner.ts` (ruta del proxy de node)
   - Wrappers de CLI que llaman a `/act` (añadir `--timeout-ms` a `browser evaluate`)
3. Implementar `src/browser/cdp-evaluate.ts`:
   - conectarse al socket CDP a nivel de navegador
   - `Target.attachToTarget` para obtener un `sessionId`
   - ejecutar `Runtime.evaluate` para la evaluación de página
   - ejecutar `DOM.resolveNode` + `Runtime.callFunctionOn` para la evaluación de elementos
   - en caso de timeout/abort: intentar `Runtime.terminateExecution` y luego cerrar el socket
4. Extender los metadatos de referencia de rol almacenados para incluir opcionalmente `backendDOMNodeId`:
   - mantener el comportamiento existente `{ role, name, nth }` para acciones de Playwright
   - añadir `backendDOMNodeId?: number` para la selección de elementos mediante CDP
5. Rellenar `backendDOMNodeId` durante la creación del snapshot (best-effort):
   - obtener el árbol AX mediante CDP (`Accessibility.getFullAXTree`)
   - calcular `(role, name, nth) -> backendDOMNodeId` y fusionarlo en el mapa de referencias almacenado
   - si el mapeo es ambiguo o falta, dejar el id como undefined
6. Actualizar el enrutamiento de `act:evaluate`:
   - si no hay `ref`: usar siempre evaluación CDP
   - si `ref` se resuelve a un `backendDOMNodeId`: usar evaluación de elemento mediante CDP
   - en caso contrario: volver a Playwright evaluate (también limitado y abortable)
7. Mantener la ruta de recuperación existente de "último recurso" como fallback, no como ruta predeterminada.
8. Añadir pruebas:
   - una evaluación bloqueada supera el tiempo límite dentro del presupuesto y el siguiente click/type tiene éxito
   - abort cancela la evaluación (desconexión del cliente o timeout) y desbloquea acciones posteriores
   - los fallos de mapeo vuelven limpiamente a Playwright
9. Añadir observabilidad:
   - duración de evaluate y contadores de timeout
   - uso de terminateExecution
   - tasa de fallback (CDP -> Playwright) y motivos

### Criterios de aceptación

- Un `act:evaluate` deliberadamente colgado devuelve dentro del presupuesto del llamador y no bloquea la
  pestaña para acciones posteriores.
- `timeoutMs` se comporta de forma coherente en CLI, herramienta del agente, proxy de node y llamadas en proceso.
- Si `ref` puede mapearse a `backendDOMNodeId`, la evaluación de elemento usa CDP; en caso contrario, la
  ruta de fallback sigue estando limitada y es recuperable.

## Plan de pruebas

- Pruebas unitarias:
  - lógica de coincidencia `(role, name, nth)` entre referencias de rol y nodos del árbol AX.
  - comportamiento del helper de presupuesto (margen, cálculo del tiempo restante).
- Pruebas de integración:
  - el timeout de CDP evaluate devuelve dentro del presupuesto y no bloquea la siguiente acción.
  - Abort cancela evaluate y activa terminateExecution como mejor esfuerzo.
- Pruebas de contrato:
  - Asegúrate de que `BrowserActRequest` y `BrowserActResponse` sigan siendo compatibles.

## Riesgos y mitigaciones

- El mapeo es imperfecto:
  - Mitigación: mapeo de mejor esfuerzo, recurrir a Playwright evaluate como alternativa y añadir herramientas de depuración.
- `Runtime.terminateExecution` tiene efectos secundarios:
  - Mitigación: usar solo en caso de timeout/abort y documentar el comportamiento en los errores.
- Sobrecarga adicional:
  - Mitigación: obtener el árbol AX solo cuando se soliciten snapshots, almacenar en caché por target y mantener
    la sesión CDP de corta duración.
- Limitaciones del relay de la extensión:
  - Mitigación: usar APIs de conexión a nivel de navegador cuando no haya sockets por página disponibles y
    mantener la ruta actual de Playwright como alternativa.

## Preguntas abiertas

- ¿Debería el nuevo motor ser configurable como `playwright`, `cdp` o `auto`?
- ¿Queremos exponer un nuevo formato "nodeRef" para usuarios avanzados o mantener solo `ref`?
- ¿Cómo deberían participar los snapshots de frame y los snapshots con alcance de selector en el mapeo AX?
