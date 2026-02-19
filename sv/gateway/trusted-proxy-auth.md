---
summary: "Delegera gateway-autentisering till en betrodd reverse proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Köra OpenClaw bakom en identitetsmedveten proxy
  - Konfigurera Pomerium, Caddy eller nginx med OAuth framför OpenClaw
  - Åtgärda WebSocket 1008 unauthorized-fel med reverse proxy-konfigurationer
---

# Betrodd proxy-autentisering

> ⚠️ **Säkerhetskänslig funktion.** Detta läge delegerar autentisering helt till din reverse proxy. Felkonfiguration kan exponera din Gateway för obehörig åtkomst. Läs denna sida noggrant innan du aktiverar.

## När ska det användas

Använd `trusted-proxy`-autentiseringsläge när:

- Du kör OpenClaw bakom en **identitetsmedveten proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Din proxy hanterar all autentisering och skickar vidare användaridentitet via headers
- Du befinner dig i en Kubernetes- eller container-miljö där proxyn är den enda vägen till Gateway
- Du får WebSocket `1008 unauthorized`-fel eftersom webbläsare inte kan skicka tokens i WS-payloads

## När ska det INTE användas

- Om din proxy inte autentiserar användare (bara fungerar som TLS-terminering eller lastbalanserare)
- Om det finns någon väg till Gateway som kringgår proxyn (brandväggshål, intern nätverksåtkomst)
- Om du är osäker på om din proxy korrekt tar bort/ersätter vidarebefordrade headers
- Om du endast behöver personlig åtkomst för en enskild användare (överväg Tailscale Serve + loopback för en enklare konfiguration)

## Hur det fungerar

1. Din reverse proxy autentiserar användare (OAuth, OIDC, SAML, etc.)
2. Proxyn lägger till en header med den autentiserade användarens identitet (t.ex. `x-forwarded-user: nick@example.com`)
3. OpenClaw kontrollerar att begäran kom från en **betrodd proxy-IP** (konfigurerad i `gateway.trustedProxies`)
4. OpenClaw extraherar användaridentiteten från den konfigurerade headern
5. Om allt stämmer auktoriseras begäran

## Konfiguration

```json5
{
  gateway: {
    // Must bind to network interface (not loopback)
    bind: "lan",

    // CRITICAL: Only add your proxy's IP(s) here
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header containing authenticated user identity (required)
        userHeader: "x-forwarded-user",

        // Optional: headers that MUST be present (proxy verification)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Optional: restrict to specific users (empty = allow all)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Konfigurationsreferens

| Fält                                        | Obligatorisk | Beskrivning                                                                                                                     |
| ------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Ja           | Array med proxy-IP-adresser som ska litas på. Begäranden från andra IP-adresser avvisas.        |
| `gateway.auth.mode`                         | Ja           | Måste vara `"trusted-proxy"`                                                                                                    |
| `gateway.auth.trustedProxy.userHeader`      | Ja           | Header-namn som innehåller den autentiserade användarens identitet                                                              |
| `gateway.auth.trustedProxy.requiredHeaders` | Nej          | Ytterligare headers som måste finnas med för att begäran ska anses betrodd                                                      |
| `gateway.auth.trustedProxy.allowUsers`      | Nej          | Tillåtelselista över användaridentiteter. Tom innebär att alla autentiserade användare tillåts. |

## Exempel på proxykonfiguration

### Pomerium

Pomerium skickar identitet i `x-pomerium-claim-email` (eller andra claim-headers) och en JWT i `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomeriums IP
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

Pomerium konfigurationsutdrag:

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

### Caddy med OAuth

Caddy med pluginen `caddy-security` kan autentisera användare och skicka vidare identitetsheaders.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddys IP (om på samma värd)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile-utdrag:

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

oauth2-proxy autentiserar användare och skickar identiteten i `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // nginx/oauth2-proxy IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx konfigurationsutdrag:

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

### Traefik med Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // Traefik-containerns IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Säkerhetschecklista

Innan du aktiverar trusted-proxy-autentisering, verifiera:

- [ ] **Proxy är den enda vägen**: Gateway-porten är brandväggsskyddad från allt utom din proxy
- [ ] **trustedProxies är minimal**: Endast dina faktiska proxy-IP-adresser, inte hela subnät
- [ ] **Proxy rensar headers**: Din proxy skriver över (inte lägger till) `x-forwarded-*`-headers från klienter
- [ ] **TLS-terminering**: Din proxy hanterar TLS; användare ansluter via HTTPS
- [ ] **allowUsers är satt** (rekommenderas): Begränsa till kända användare istället för att tillåta alla autentiserade

## Säkerhetsgranskning

`openclaw security audit` kommer att flagga trusted-proxy-autentisering med en **kritisk** allvarlighetsgrad. Detta är avsiktligt — det är en påminnelse om att du delegerar säkerheten till din proxykonfiguration.

Granskningen kontrollerar följande:

- Saknad `trustedProxies`-konfiguration
- Saknad `userHeader`-konfiguration
- Tom `allowUsers` (tillåter alla autentiserade användare)

## Felsökning

### "trusted_proxy_untrusted_source"

Begäran kom inte från en IP-adress i `gateway.trustedProxies`. Kontrollera:

- Är proxyns IP-adress korrekt? (Docker-containrars IP-adresser kan ändras)
- Finns det en lastbalanserare framför din proxy?
- Använd `docker inspect` eller `kubectl get pods -o wide` för att hitta faktiska IP-adresser

### "trusted_proxy_user_missing"

Användarhuvudet var tomt eller saknades. Kontrollera:

- Är din proxy konfigurerad för att vidarebefordra identitetshuvuden?
- Är header-namnet korrekt? (skiftlägesokänsligt, men stavningen är viktig)
- Är användaren faktiskt autentiserad i proxyn?

### "trusted_proxy_missing_header_\*"

En obligatorisk header saknades. Kontrollera:

- Din proxykonfiguration för dessa specifika headers
- Om headers tas bort någonstans i kedjan

### "trusted_proxy_user_not_allowed"

Användaren är autentiserad men finns inte i `allowUsers`. Lägg antingen till dem eller ta bort allowlistan.

### WebSocket fungerar fortfarande inte

Se till att din proxy:

- Stöder WebSocket-uppgraderingar (`Upgrade: websocket`, `Connection: upgrade`)
- Vidarebefordrar identitetshuvuden vid WebSocket-uppgraderingsförfrågningar (inte bara HTTP)
- Inte har en separat autentiseringsväg för WebSocket-anslutningar

## Migrering från token-autentisering

Om du går från token-autentisering till trusted-proxy:

1. Konfigurera din proxy så att den autentiserar användare och vidarebefordrar headers
2. Testa proxyinställningen oberoende (curl med headers)
3. Uppdatera OpenClaw-konfigurationen med trusted-proxy-autentisering
4. Starta om Gateway
5. Testa WebSocket-anslutningar från Control UI
6. Kör `openclaw security audit` och granska resultaten

## Relaterat

- [Security](/gateway/security) — fullständig säkerhetsguide
- [Configuration](/gateway/configuration) — konfigurationsreferens
- [Remote Access](/gateway/remote) — andra mönster för fjärråtkomst
- [Tailscale](/gateway/tailscale) — enklare alternativ för åtkomst endast via tailnet

