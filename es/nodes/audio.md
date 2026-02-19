---
summary: "Cómo se descargan, transcriben y se inyectan en las respuestas los audios/notas de voz entrantes"
read_when:
  - Cambio de la transcripción de audio o del manejo de medios
title: "Audio y notas de voz"
---

# Audio / Notas de voz — 2026-01-17

## Qué funciona

- **Comprensión de medios (audio)**: Si la comprensión de audio está habilitada (o se detecta automáticamente), OpenClaw:
  1. Localiza el primer adjunto de audio (ruta local o URL) y lo descarga si es necesario.
  2. Aplica `maxBytes` antes de enviar a cada entrada de modelo.
  3. Ejecuta la primera entrada de modelo elegible en orden (proveedor o CLI).
  4. Si falla o se omite (tamaño/tiempo de espera), intenta la siguiente entrada.
  5. En caso de éxito, reemplaza `Body` con un bloque `[Audio]` y establece `{{Transcript}}`.
- **Análisis de comandos**: Cuando la transcripción tiene éxito, `CommandBody`/`RawBody` se establecen con la transcripción para que los comandos con barra sigan funcionando.
- **Registro detallado**: En `--verbose`, registramos cuándo se ejecuta la transcripción y cuándo reemplaza el cuerpo.

## Detección automática (predeterminada)

Si **no configura modelos** y `tools.media.audio.enabled` **no** está establecido en `false`,
OpenClaw detecta automáticamente en este orden y se detiene en la primera opción que funcione:

1. **CLIs locales** (si están instaladas)
   - `sherpa-onnx-offline` (requiere `SHERPA_ONNX_MODEL_DIR` con encoder/decoder/joiner/tokens)
   - `whisper-cli` (de `whisper-cpp`; usa `WHISPER_CPP_MODEL` o el modelo tiny incluido)
   - `whisper` (CLI de Python; descarga modelos automáticamente)
2. **CLI de Gemini** (`gemini`) usando `read_many_files`
3. **Claves de proveedor** (OpenAI → Groq → Deepgram → Google)

Para desactivar la detección automática, establezca `tools.media.audio.enabled: false`.
Para personalizar, establezca `tools.media.audio.models`.
Nota: La detección de binarios es de mejor esfuerzo en macOS/Linux/Windows; asegúrese de que la CLI esté en `PATH` (expandimos `~`), o establezca un modelo de CLI explícito con una ruta completa al comando.

## Ejemplos de configuración

### Proveedor + respaldo por CLI (OpenAI + Whisper CLI)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
            timeoutSeconds: 45,
          },
        ],
      },
    },
  },
}
```

### Solo proveedor con control por alcance

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        scope: {
          default: "allow",
          rules: [{ action: "deny", match: { chatType: "group" } }],
        },
        models: [{ provider: "openai", model: "gpt-4o-mini-transcribe" }],
      },
    },
  },
}
```

### Solo proveedor (Deepgram)

```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }],
      },
    },
  },
}
```

## Notas y límites

- La autenticación del proveedor sigue el orden estándar de autenticación del modelo (perfiles de autenticación, variables de entorno, `models.providers.*.apiKey`).
- Deepgram toma `DEEPGRAM_API_KEY` cuando se usa `provider: "deepgram"`.
- Detalles de configuración de Deepgram: [Deepgram (transcripción de audio)](/providers/deepgram).
- Los proveedores de audio pueden sobrescribir `baseUrl`, `headers` y `providerOptions` mediante `tools.media.audio`.
- El límite de tamaño predeterminado es 20MB (`tools.media.audio.maxBytes`). El audio que exceda el tamaño se omite para ese modelo y se intenta la siguiente entrada.
- El `maxChars` predeterminado para audio está **sin establecer** (transcripción completa). Establezca `tools.media.audio.maxChars` o `maxChars` por entrada para recortar la salida.
- El valor predeterminado automático de OpenAI es `gpt-4o-mini-transcribe`; establezca `model: "gpt-4o-transcribe"` para mayor precisión.
- Use `tools.media.audio.attachments` para procesar múltiples notas de voz (`mode: "all"` + `maxAttachments`).
- La transcripción está disponible para las plantillas como `{{Transcript}}`.
- La salida stdout de la CLI está limitada (5MB); mantenga la salida de la CLI concisa.

## Detección de menciones en grupos

Cuando se establece `requireMention: true` para un chat grupal, OpenClaw ahora transcribe el audio **antes** de comprobar las menciones. Esto permite procesar las notas de voz incluso cuando contienen menciones.

**Cómo funciona:**

1. Si un mensaje de voz no tiene texto y el grupo requiere menciones, OpenClaw realiza una transcripción "preflight".
2. La transcripción se revisa en busca de patrones de mención (p. ej., `@BotName`, disparadores con emoji).
3. Si se encuentra una mención, el mensaje continúa por el flujo completo de respuesta.
4. La transcripción se utiliza para detectar menciones, permitiendo que las notas de voz superen el filtro de menciones.

**Comportamiento alternativo:**

- Si la transcripción falla durante el preflight (tiempo de espera, error de API, etc.), el mensaje se procesa basándose únicamente en la detección de menciones en el texto.
- Esto garantiza que los mensajes mixtos (texto + audio) nunca se descarten incorrectamente.

**Ejemplo:** Un usuario envía una nota de voz diciendo "Hey @Claude, what's the weather?" en un grupo de Telegram con `requireMention: true`. La nota de voz se transcribe, se detecta la mención y el agente responde.

## Gotchas

- Las reglas de alcance usan “primera coincidencia gana”. `chatType` se normaliza a `direct`, `group` o `room`.
- Asegúrese de que su CLI termine con código 0 e imprima texto plano; el JSON debe ajustarse mediante `jq -r .text`.
- Mantenga tiempos de espera razonables (`timeoutSeconds`, predeterminado 60s) para evitar bloquear la cola de respuestas.
- La transcripción preflight solo procesa el **primer** archivo de audio adjunto para la detección de menciones. El audio adicional se procesa durante la fase principal de comprensión de medios.
