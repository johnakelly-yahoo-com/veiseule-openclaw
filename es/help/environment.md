---
title: "Variables de entorno"
---

# Variables de entorno

OpenClaw obtiene variables de entorno de múltiples fuentes. La regla es **nunca sobrescribir valores existentes**.

## Precedencia (más alta → más baja)

1. **Entorno del proceso** (lo que el proceso del Gateway ya tiene del shell/daemon padre).
2. **`.env` en el directorio de trabajo actual** (valor predeterminado de dotenv; no sobrescribe).
3. **`.env` global** en `~/.openclaw/.env` (también conocido como `$OPENCLAW_STATE_DIR/.env`; no sobrescribe).
4. **Bloque `env` de configuración** en `~/.openclaw/openclaw.json` (se aplica solo si falta).
5. **Importación opcional del shell de inicio de sesión** (`env.shellEnv.enabled` o `OPENCLAW_LOAD_SHELL_ENV=1`), aplicada solo para claves esperadas faltantes.

Si el archivo de configuración falta por completo, el paso 4 se omite; la importación del shell aún se ejecuta si está habilitada.

## Bloque de configuración `env`

Dos formas equivalentes de establecer variables env en línea (ambas no son anuladas):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Importación de variables de entorno del shell

`env.shellEnv` ejecuta su shell de inicio de sesión e importa solo las claves esperadas **faltantes**:

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

Equivalentes de var Env:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Sustitución de var Env en configuración

Puede referenciar variables de entorno directamente en valores de cadena de la configuración usando la sintaxis `${VAR_NAME}`:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Consulte [Configuración: Sustitución de variables de entorno](/gateway/configuration#env-var-substitution-in-config) para conocer todos los detalles.

## Variables de entorno relacionadas con rutas

| Variable               | Propósito                                                                                                                                                                                                                           |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | Anula el directorio principal utilizado para toda la resolución interna de rutas (`~/.openclaw/`, agent dirs, sessions, credentials). Useful when running OpenClaw as a dedicated service user. |
| `OPENCLAW_STATE_DIR`   | Anula el directorio de estado (predeterminado `~/.openclaw`).                                                                                                                                            |
| `OPENCLAW_CONFIG_PATH` | Anula la ruta del archivo de configuración (por defecto `~/.openclaw/openclaw.json`).                                                                                                            |

### `OPENCLAW_HOME`

Cuando se establece, `OPENCLAW_HOME` reemplaza el directorio home del sistema (`$HOME` / `os.homedir()`) para toda la resolución interna de rutas. Esto permite un aislamiento completo del sistema de archivos para cuentas de servicio sin interfaz gráfica.

**Precedencia:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**Ejemplo** (LaunchDaemon de macOS):

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` también puede establecerse como una ruta con tilde (por ejemplo `~/svc`), que se expande usando `$HOME` antes de su uso.

## Relacionado

- [Configuración del Gateway](/gateway/configuration)
- [Preguntas frecuentes: variables de entorno y carga de .env](/help/faq#env-vars-and-env-loading)
- [Descripción general de modelos](/concepts/models)
