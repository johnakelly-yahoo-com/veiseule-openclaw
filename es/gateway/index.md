---
summary: "Manual operativo del servicio Gateway, su ciclo de vida y operaciones"
read_when:
  - Al ejecutar o depurar el proceso del gateway
title: "Manual operativo del Gateway"
---

# Manual operativo del servicio Gateway

Utiliza esta página para el arranque del día 1 y las operaciones del día 2 del servicio Gateway.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    Diagnósticos orientados a síntomas con secuencias de comandos exactas y firmas de registros.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    Guía de configuración orientada a tareas + referencia completa de configuración.
  
</Card>
</CardGroup>

## Inicio local en 5 minutos

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

Línea base saludable: `Runtime: running` y `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
La recarga de configuración del Gateway supervisa la ruta activa del archivo de configuración (resuelta desde los valores predeterminados del perfil/estado, o `OPENCLAW_CONFIG_PATH` cuando está establecido).
Modo predeterminado: `gateway.reload.mode="hybrid"` (aplica en caliente cambios seguros, reinicia en cambios críticos).
</Note>

## Modelo de ejecución

- Un proceso siempre activo para el enrutamiento, el plano de control y las conexiones de canales.
- Multiplexado de puerto único.
  - Control/RPC por WebSocket
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - UI de control y hooks
- Modo de enlace predeterminado: `loopback`.
- La autenticación del Gateway es requerida por defecto: configure `gateway.auth.token` (o `OPENCLAW_GATEWAY_TOKEN`) o `gateway.auth.password`.

### Precedencia de puerto y enlace

| Configuración      | Orden de resolución                                                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------- |
| Puerto del Gateway | Precedencia de puertos: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > predeterminado `18789`. |
| Modo de enlace     | CLI/override → `gateway.bind` → `loopback`                                                                                            |

### Modos de recarga en caliente

| Deshabilite con `gateway.reload.mode="off"`. | Comportamiento de keepalive                                           |
| ------------------------------------------------------------ | --------------------------------------------------------------------- |
| `off`                                                        | Sin recarga de configuración                                          |
| `hot`                                                        | Aplicar solo cambios seguros en caliente                              |
| `restart`                                                    | Reiniciar en cambios que requieren recarga                            |
| `hybrid` (predeterminado)                 | Aplicar en caliente cuando sea seguro, reiniciar cuando sea necesario |

## Conjunto de comandos del operador

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## Acceso remoto

Tailscale/VPN es preferido; de lo contrario, túnel SSH:
Alternativa: túnel SSH.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

Luego los clientes se conectan a `ws://127.0.0.1:18789` a través del túnel.

<Warning>
El uso remoto pasa por el mismo túnel SSH/Tailscale; si se configura un token del gateway, el cliente lo incluye durante `connect`.
</Warning>

Ver: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## Supervisión y ciclo de vida del servicio

Utiliza ejecuciones supervisadas para una fiabilidad similar a producción.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
Para reiniciar, use `openclaw gateway restart` (o `launchctl kickstart -k gui/$UID/bot.molt.gateway`).
```

Las etiquetas de LaunchAgent son `ai.openclaw.gateway` (predeterminado) o `ai.openclaw.<profile>` al ejecutar un perfil con nombre. `openclaw doctor` audita y repara la desviación de configuración del servicio.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

Habilite lingering (requerido para que el servicio de usuario sobreviva a cierre de sesión/inactividad):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

Usa una unidad del sistema para hosts multiusuario/siempre activos.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## Múltiples gateways (mismo host)

Por lo general es innecesario: un Gateway puede servir múltiples canales de mensajería y agentes.
Use múltiples Gateways solo para redundancia o aislamiento estricto (ej.: bot de rescate).

Lista de verificación por instancia:

- `gateway.port` único
- `OPENCLAW_CONFIG_PATH` único
- `OPENCLAW_STATE_DIR` único
- `agents.defaults.workspace` único

Ejemplo:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

Vea [Múltiples gateways](/gateway/multiple-gateways).

### Ruta rápida del perfil de desarrollo

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

Los valores predeterminados incluyen estado/configuración aislados y el puerto base del gateway `19001`.

## Protocolo (vista del operador)

- El primer frame del cliente debe ser `connect`.
- El Gateway devuelve una instantánea `hello-ok` (`presence`, `health`, `stateVersion`, `uptimeMs`, límites/política).
- Solicitudes: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- Eventos comunes: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Las ejecuciones del agente son de dos etapas:

1. Acuse de recibo inmediato de aceptación (`status:"accepted"`)
2. Las respuestas `agent` son de dos etapas: primero un ack `res` `{runId,status:"accepted"}`, luego un `res` `{runId,status:"ok"|"error",summary}` final tras finalizar la ejecución; la salida en streaming llega como `event:"agent"`.

Documentación completa: [Protocolo del Gateway](/gateway/protocol) y [Protocolo Bridge (heredado)](/gateway/bridge-protocol).

## Verificaciones operativas

### Vitalidad

- Abre WS y envía `connect`.
- Espera una respuesta `hello-ok` con la instantánea.

### Preparación

```bash
`openclaw gateway health|status` — solicitar salud/estado sobre el WS del Gateway.
```

### Recuperación de brechas

Los eventos no se reproducen. En brechas de secuencia, actualiza el estado (`health`, `system-presence`) antes de continuar.

## Firmas de fallo comunes

| Firma                                                          | Problema probable                                        |
| -------------------------------------------------------------- | -------------------------------------------------------- |
| `refusing to bind gateway ... without auth`                    | Vinculación no loopback sin token/contraseña             |
| `another gateway instance is already listening` / `EADDRINUSE` | Conflicto de puerto                                      |
| `Gateway start blocked: set gateway.mode=local`                | Configuración establecida en modo remoto                 |
| `unauthorized` during connect                                  | Desajuste de autenticación entre el cliente y el Gateway |

Para escaleras de diagnóstico completas, usa [Gateway Troubleshooting](/gateway/troubleshooting).

## Garantías de seguridad

- Los clientes del protocolo Gateway fallan rápidamente cuando el Gateway no está disponible (sin retroceso implícito a canal directo).
- Los primeros frames no-connect o JSON malformado se rechazan y el socket se cierra.
- Apagado ordenado: emite el evento `shutdown` antes de cerrar; los clientes deben manejar cierre + reconexión.

---

Relacionado:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)

