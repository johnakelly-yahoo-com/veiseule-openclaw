---
summary: "Configuración de Hugging Face Inference (autenticación + selección de modelo)"
read_when:
  - Quieres usar Hugging Face Inference con OpenClaw
  - Necesitas la variable de entorno del token de HF o la opción de autenticación por CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) ofrecen chat completions compatibles con OpenAI a través de una única API router. Obtienes acceso a muchos modelos (DeepSeek, Llama y más) con un solo token. OpenClaw utiliza el **endpoint compatible con OpenAI** (solo chat completions); para text-to-image, embeddings o voz, utiliza directamente los [clientes de inferencia de HF](https://huggingface.co/docs/api-inference/quicktour).

- Proveedor: `huggingface`
- Autenticación: `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN` (token de permisos granulares con **Make calls to Inference Providers**)
- API: Compatible con OpenAI (`https://router.huggingface.co/v1`)
- Facturación: Un único token de HF; los [precios](https://huggingface.co/docs/inference-providers/pricing) siguen las tarifas del proveedor con un nivel gratuito.

## Inicio rápido

1. Crea un token de permisos granulares en [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) con el permiso **Make calls to Inference Providers**.
2. Ejecuta el onboarding y elige **Hugging Face** en el menú desplegable de proveedor; luego introduce tu API key cuando se te solicite:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. En el menú desplegable **Default Hugging Face model**, selecciona el modelo que desees (la lista se carga desde la Inference API cuando tienes un token válido; de lo contrario, se muestra una lista integrada). Tu elección se guarda como modelo predeterminado.
4. También puedes establecer o cambiar el modelo predeterminado más adelante en la configuración:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Ejemplo no interactivo

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Esto establecerá `huggingface/deepseek-ai/DeepSeek-R1` como modelo predeterminado.

## Nota sobre el entorno

Si el Gateway se ejecuta como un daemon (launchd/systemd), asegúrate de que `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN`
esté disponible para ese proceso (por ejemplo, en `~/.openclaw/.env` o mediante
`env.shellEnv`).

## Descubrimiento de modelos y menú desplegable de onboarding

OpenClaw descubre modelos llamando **directamente al endpoint de Inference**:

```bash
GET https://router.huggingface.co/v1/models
```

(Opcional: envía `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` o `$HF_TOKEN` para obtener la lista completa; algunos endpoints devuelven un subconjunto sin autenticación). La respuesta tiene formato OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Cuando configuras una clave de API de Hugging Face (mediante el onboarding, `HUGGINGFACE_HUB_TOKEN` o `HF_TOKEN`), OpenClaw utiliza este GET para descubrir los modelos de chat-completion disponibles. Durante el **onboarding interactivo**, después de introducir tu token verás un menú desplegable de **Modelo predeterminado de Hugging Face** rellenado a partir de esa lista (o del catálogo integrado si la solicitud falla). En tiempo de ejecución (por ejemplo, al iniciar el Gateway), cuando hay una clave presente, OpenClaw vuelve a llamar a **GET** `https://router.huggingface.co/v1/models` para actualizar el catálogo. La lista se combina con un catálogo integrado (para metadatos como la ventana de contexto y el coste). Si la solicitud falla o no se ha configurado ninguna clave, solo se utiliza el catálogo integrado.

## Nombres de modelos y opciones editables

- **Nombre desde la API:** El nombre mostrado del modelo se **hidrata desde GET /v1/models** cuando la API devuelve `name`, `title` o `display_name`; de lo contrario, se deriva del id del modelo (por ejemplo, `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Sobrescribir nombre mostrado:** Puedes establecer una etiqueta personalizada por modelo en la configuración para que aparezca como prefieras en la CLI y la UI:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Selección de proveedor / política:** Añade un sufijo al **id del modelo** para elegir cómo el router selecciona el backend:

  - **`:fastest`** — mayor rendimiento (el router decide; la elección de proveedor queda **bloqueada** — no hay selector interactivo de backend).
  - **`:cheapest`** — menor coste por token de salida (el router decide; la elección de proveedor queda **bloqueada**).
  - **`:provider`** — fuerza un backend específico (por ejemplo, `:sambanova`, `:together`).

  Cuando seleccionas **:cheapest** o **:fastest** (por ejemplo, en el menú desplegable de modelo durante el onboarding), el proveedor queda bloqueado: el router decide según el coste o la velocidad y no se muestra ningún paso opcional para “preferir un backend específico”. Puedes añadirlos como entradas independientes en `models.providers.huggingface.models` o establecer `model.primary` con el sufijo. También puedes establecer tu orden predeterminado en [Inference Provider settings](https://hf.co/settings/inference-providers) (sin sufijo = usar ese orden).

- **Fusión de configuración:** Las entradas existentes en `models.providers.huggingface.models` (por ejemplo, en `models.json`) se mantienen cuando se fusiona la configuración. Por lo tanto, cualquier `name`, `alias` u opciones del modelo que configures allí se conservan.

## IDs de modelos y ejemplos de configuración

Las referencias de modelo usan la forma `huggingface/<org>/<model>` (IDs con formato Hub). La lista siguiente proviene de **GET** `https://router.huggingface.co/v1/models`; tu catálogo puede incluir más.

**Ejemplos de IDs (desde el endpoint de inference):**

| Modelo                                 | Ref (prefijo con `huggingface/`) |
| -------------------------------------- | --------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                           |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                         |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                     |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                          |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                    |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                 |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                  |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                               |
| GLM 4.7                | `zai-org/GLM-4.7`                                   |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                              |

Puedes añadir `:fastest`, `:cheapest` o `:provider` (p. ej., `:together`, `:sambanova`) al id del modelo. Configura tu orden predeterminado en [Inference Provider settings](https://hf.co/settings/inference-providers); consulta [Inference Providers](https://huggingface.co/docs/inference-providers) y **GET** `https://router.huggingface.co/v1/models` para la lista completa.

### Ejemplos completos de configuración

**DeepSeek R1 principal con Qwen como respaldo:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**Qwen como predeterminado, con variantes :cheapest y :fastest:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**DeepSeek + Llama + GPT-OSS con alias:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**Forzar un backend específico con :provider:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**Múltiples modelos Qwen y DeepSeek con sufijos de política:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```
