---
owner: "openclaw"
status: "complete"
last_updated: "2026-01-05"
title: "Endurecimiento de Cron Add"
---

# Endurecimiento de Cron Add y alineación de esquemas

## Contexto

Los registros recientes del Gateway muestran fallas repetidas `cron.add` con parámetros inválidos (faltan `sessionTarget`, `wakeMode`, `payload`, y `schedule` malformado). Esto indica que al menos un cliente (probablemente la ruta de llamada de la herramienta del agente) está enviando cargas de trabajo envueltas o parcialmente especificadas. Por separado, existe una divergencia entre los enums del proveedor de cron en TypeScript, el esquema del Gateway, las banderas de la CLI y los tipos de formularios de la UI, además de una discrepancia de la UI para `cron.status` (espera `jobCount` mientras el Gateway devuelve `jobs`).

## Objetivos

- Detener el spam de INVALID_REQUEST `cron.add` normalizando cargas envueltas comunes e infiriendo campos `kind` faltantes.
- Alinear las listas de proveedores de cron entre el esquema del Gateway, los tipos de cron, la documentación de la CLI y los formularios de la UI.
- Hacer explícito el esquema de la herramienta de cron del agente para que el LLM produzca cargas de trabajo correctas.
- Corregir la visualización del conteo de trabajos del estado de cron en la UI de Control.
- Agregar pruebas para cubrir la normalización y el comportamiento de la herramienta.

## No objetivos

- Cambiar la semántica de programación de cron o el comportamiento de ejecución de trabajos.
- Agregar nuevos tipos de programación o análisis de expresiones cron.
- Reformar la UI/UX de cron más allá de las correcciones de campos necesarias.

## Hallazgos (brechas actuales)

- `CronPayloadSchema` en el Gateway excluye `signal` + `imessage`, mientras que los tipos de TS los incluyen.
- CronStatus de la UI de Control espera `jobCount`, pero el Gateway devuelve `jobs`.
- El esquema de la herramienta de cron del agente permite objetos `job` arbitrarios, lo que habilita entradas malformadas.
- El Gateway valida estrictamente `cron.add` sin normalización, por lo que las cargas envueltas fallan.

## Qué cambió

- `cron.add` y `cron.update` ahora normalizan formas comunes de envoltura e infieren campos `kind` faltantes.
- El esquema de la herramienta de cron del agente coincide con el esquema del Gateway, lo que reduce cargas inválidas.
- Los enums de proveedores están alineados entre el Gateway, la CLI, la UI y el selector de macOS.
- La UI de Control usa el campo de conteo `jobs` del Gateway para el estado.

## Comportamiento actual

- **Normalización:** las cargas `data`/`job` envueltas se desempaquetan; `schedule.kind` y `payload.kind` se infieren cuando es seguro.
- **Valores predeterminados:** se aplican valores predeterminados seguros para `wakeMode` y `sessionTarget` cuando faltan.
- **Proveedores:** Discord/Slack/Signal/iMessage ahora se muestran de forma consistente en la CLI y la UI.

Vea [Cron jobs](/automation/cron-jobs) para la forma normalizada y ejemplos.

## Verificación

- Observe los registros del Gateway para una reducción de errores INVALID_REQUEST `cron.add`.
- Confirme que el estado de cron en la UI de Control muestre el conteo de trabajos después de actualizar.

## Seguimientos opcionales

- Prueba manual de la UI de Control: agregar un trabajo de cron por proveedor y verificar el conteo de trabajos del estado.

## Preguntas abiertas

- ¿Debería `cron.add` aceptar `state` explícito de los clientes (actualmente no permitido por el esquema)?
- ¿Deberíamos permitir `webchat` como proveedor de entrega explícito (actualmente filtrado en la resolución de entrega)?

