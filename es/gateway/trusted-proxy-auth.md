---
summary: "Delegar la autenticación del gateway a un proxy inverso de confianza (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Ejecutar OpenClaw detrás de un proxy con reconocimiento de identidad
  - Configurar Pomerium, Caddy o nginx con OAuth delante de OpenClaw
  - Solucionar errores WebSocket 1008 unauthorized en configuraciones con proxy inverso
---

# Autenticación de proxy de confianza

> ⚠️ **Función sensible desde el punto de vista de la seguridad.** Este modo delega completamente la autenticación a tu proxy inverso. Una mala configuración puede exponer tu Gateway a accesos no autorizados. Lee esta página atentamente antes de habilitarlo.

## Cuándo usarlo

Usa el modo de autenticación `trusted-proxy` cuando:

- Ejecutas OpenClaw detrás de un **proxy con reconocimiento de identidad** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Tu proxy gestiona toda la autenticación y transmite la identidad del usuario mediante encabezados
- Estás en un entorno Kubernetes o de contenedores donde el proxy es la única vía de acceso al Gateway
- Recibes errores WebSocket `1008 unauthorized` porque los navegadores no pueden enviar tokens en cargas útiles WS

## Cuándo NO usarlo

- Si tu proxy no autentica usuarios (solo actúa como terminador TLS o balanceador de carga)
- Si existe alguna ruta hacia el Gateway que omita el proxy (reglas de firewall abiertas, acceso desde la red interna)
- Si no estás seguro de que tu proxy elimina/sobrescribe correctamente los encabezados reenviados
- Si solo necesitas acceso personal para un único usuario (considera Tailscale Serve + loopback para una configuración más sencilla)

## Cómo funciona

1. Tu proxy inverso autentica a los usuarios (OAuth, OIDC, SAML, etc.)
2. El proxy añade un encabezado con la identidad del usuario autenticado (p. ej., `x-forwarded-user: nick@example.com`)
3. OpenClaw comprueba que la solicitud provenga de una **IP de proxy de confianza** (configurada en `gateway.trustedProxies`)
4. OpenClaw extrae la identidad del usuario del encabezado configurado
5. Si todo es correcto, la solicitud queda autorizada

## Configuración

```json5
{
  gateway: {
    // Debe enlazarse a la interfaz de red (no loopback)
    bind: "lan",

    // CRÍTICO: Añade aquí solo la(s) IP(s) de tu proxy
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Encabezado que contiene la identidad del usuario autenticado (obligatorio)
        userHeader: "x-forwarded-user",

        // Opcional: encabezados que DEBEN estar presentes (verificación del proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Opcional: restringir a usuarios específicos (vacío = permitir todos)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Referencia de configuración

| Campo                                       | Obligatorio | Descripción                                                                                                                             |
| ------------------------------------------- | ----------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Sí          | Lista de direcciones IP del proxy en las que confiar. Las solicitudes desde otras IP serán rechazadas.  |
| `gateway.auth.mode`                         | Sí          | Debe ser `"trusted-proxy"`                                                                                                              |
| `gateway.auth.trustedProxy.userHeader`      | Sí          | Nombre del encabezado que contiene la identidad del usuario autenticado                                                                 |
| `gateway.auth.trustedProxy.requiredHeaders` | No          | Encabezados adicionales que deben estar presentes para que la solicitud sea considerada confiable                                       |
| `gateway.auth.trustedProxy.allowUsers`      | No          | Lista de identidades de usuario permitidas. Vacío significa permitir a todos los usuarios autenticados. |

## Ejemplos de configuración de proxy

### Pomerium

Pomerium pasa la identidad en `x-pomerium-claim-email` (u otros encabezados de claim) y un JWT en `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de Pomerium
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-pomerium-claim-email",
        requiredHeaders: ["x-pomerium-jwt-assertion"],
      },
    },
  },
}
```

Fragmento de configuración de Pomerium:

```yaml
routes:
  - from: https://openclaw.example.com
    to: http://openclaw-gateway:18789
    policy:
      - allow:
          or:
            - email:
                is: nick@example.com
    pass_identity_headers: true
```

### Caddy con OAuth

Caddy con el plugin `caddy-security` puede autenticar usuarios y pasar encabezados de identidad.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP de Caddy (si está en el mismo host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Fragmento de Caddyfile:

```
openclaw.example.com {
    authenticate with oauth2_provider
    authorize with policy1

    reverse_proxy openclaw:18789 {
        header_up X-Forwarded-User {http.auth.user.email}
    }
}
```

### nginx + oauth2-proxy

oauth2-proxy autentica a los usuarios y pasa la identidad en `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP de nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

Fragmento de configuración de nginx:

```nginx
location / {
    auth_request /oauth2/auth;
    auth_request_set $user $upstream_http_x_auth_request_email;

    proxy_pass http://openclaw:18789;
    proxy_set_header X-Auth-Request-Email $user;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

### Traefik con Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP del contenedor de Traefik
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Lista de verificación de seguridad

Antes de habilitar la autenticación trusted-proxy, verifica:

- [ ] **El proxy es la única vía de acceso**: El puerto del Gateway está protegido por firewall frente a todo excepto tu proxy
- [ ] **trustedProxies es mínimo**: Solo las IP reales de tu proxy, no subredes completas
- [ ] **El proxy elimina encabezados**: Tu proxy sobrescribe (no añade) los encabezados `x-forwarded-*` provenientes de los clientes
- [ ] **Terminación TLS**: Tu proxy gestiona TLS; los usuarios se conectan mediante HTTPS
- [ ] **allowUsers está configurado** (recomendado): Restringe a usuarios conocidos en lugar de permitir a cualquiera autenticado

## Auditoría de seguridad

`openclaw security audit` marcará la autenticación trusted-proxy con una gravedad **crítica**. Esto es intencional — es un recordatorio de que estás delegando la seguridad a la configuración de tu proxy.

La auditoría comprueba:

- Falta la configuración de `trustedProxies`
- Falta la configuración de `userHeader`
- `allowUsers` vacío (permite cualquier usuario autenticado)

## Solución de problemas

### "trusted_proxy_untrusted_source"

La solicitud no provino de una IP en `gateway.trustedProxies`. Comprueba:

- ¿Es correcta la IP del proxy? (Las IP de los contenedores Docker pueden cambiar)
- ¿Hay un balanceador de carga delante de tu proxy?
- Usa `docker inspect` o `kubectl get pods -o wide` para encontrar las IP reales

### "trusted_proxy_user_missing"

El encabezado de usuario estaba vacío o faltaba. Comprueba:

- ¿Está tu proxy configurado para pasar los encabezados de identidad?
- ¿Es correcto el nombre del encabezado? (no distingue entre mayúsculas y minúsculas, pero la ortografía importa)
- ¿Está el usuario realmente autenticado en el proxy?

### "trusted_proxy_missing_header_\*"

Faltaba un encabezado requerido. Comprueba:

- La configuración de tu proxy para esos encabezados específicos
- Si los encabezados están siendo eliminados en algún punto de la cadena

### "trusted_proxy_user_not_allowed"

El usuario está autenticado pero no está en `allowUsers`. Añádelo o elimina la lista de permitidos.

### WebSocket sigue fallando

Asegúrate de que tu proxy:

- Sea compatible con actualizaciones de WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Pase los encabezados de identidad en las solicitudes de actualización de WebSocket (no solo HTTP)
- No tenga una ruta de autenticación separada para conexiones WebSocket

## Migración desde autenticación por token

Si estás pasando de autenticación por token a trusted-proxy:

1. Configura tu proxy para autenticar usuarios y pasar encabezados
2. Prueba la configuración del proxy de forma independiente (curl con encabezados)
3. Actualiza la configuración de OpenClaw con autenticación trusted-proxy
4. Reinicia el Gateway
5. Prueba las conexiones WebSocket desde la Control UI
6. Ejecuta `openclaw security audit` y revisa los resultados

## Relacionado

- [Security](/gateway/security) — guía completa de seguridad
- [Configuration](/gateway/configuration) — referencia de configuración
- [Remote Access](/gateway/remote) — otros patrones de acceso remoto
- [Tailscale](/gateway/tailscale) — alternativa más sencilla para acceso solo dentro de la tailnet
