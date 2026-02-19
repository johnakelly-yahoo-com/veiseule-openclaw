---
summary: "Resumen de las opciones y flujos de incorporación de OpenClaw"
read_when:
  - Elegir una ruta de incorporación
  - Configurar un nuevo entorno
title: "Resumen de incorporación"
sidebarTitle: "Resumen de incorporación"
---

# Resumen de incorporación

OpenClaw admite múltiples rutas de incorporación según dónde se ejecute el Gateway
y cómo prefieras configurar los proveedores.

## Elige tu ruta de incorporación

- **Asistente CLI** para macOS, Linux y Windows (mediante WSL2).
- **Aplicación macOS** para una primera ejecución guiada en Macs con Apple silicon o Intel.

## Asistente de incorporación CLI

Ejecuta el asistente en una terminal:

```bash
openclaw onboard
```

Utiliza el asistente CLI cuando quieras tener control total del Gateway, el espacio de trabajo,
los canales y las skills. Documentación:

- [Onboarding Wizard (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## Incorporación mediante la aplicación macOS

Usa la aplicación OpenClaw cuando quieras una configuración completamente guiada en macOS. Documentación:

- [Onboarding (macOS App)](/start/onboarding)

## Proveedor personalizado

Si necesitas un endpoint que no esté en la lista, incluidos proveedores alojados que expongan APIs estándar de OpenAI o Anthropic, elige **Custom Provider** en el asistente de la CLI. Se te pedirá:

- Elegir compatible con OpenAI, compatible con Anthropic o **Unknown** (detección automática).
- Introducir una URL base y una clave API (si el proveedor la requiere).
- Proporcionar un ID de modelo y un alias opcional.
- Elegir un Endpoint ID para que puedan coexistir varios endpoints personalizados.

Para ver los pasos detallados, sigue la documentación de onboarding de la CLI anterior.
