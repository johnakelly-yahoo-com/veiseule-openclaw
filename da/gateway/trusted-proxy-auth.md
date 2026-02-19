---
summary: "Overlad gateway-godkendelse til en betroet reverse proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Kørsel af OpenClaw bag en identitetsbevidst proxy
  - Opsætning af Pomerium, Caddy eller nginx med OAuth foran OpenClaw
  - Løsning af WebSocket 1008 unauthorized-fejl med reverse proxy-opsætninger
---

# Trusted Proxy Auth

> ⚠️ **Sikkerhedskritisk funktion.** Denne tilstand overdrager godkendelse fuldstændigt til din reverse proxy. Fejlkonfiguration kan eksponere din Gateway for uautoriseret adgang. Læs denne side grundigt, før du aktiverer.

## Hvornår skal det bruges

Brug `trusted-proxy` auth-tilstand når:

- Du kører OpenClaw bag en **identitetsbevidst proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Din proxy håndterer al godkendelse og videresender brugeridentitet via headers
- Du er i et Kubernetes- eller container-miljø, hvor proxyen er den eneste adgangsvej til Gateway
- Du får WebSocket `1008 unauthorized`-fejl, fordi browsere ikke kan sende tokens i WS-payloads

## Hvornår det IKKE skal bruges

- Hvis din proxy ikke godkender brugere (kun fungerer som TLS-terminering eller load balancer)
- Hvis der findes nogen adgangsvej til Gateway, som omgår proxyen (huller i firewall, intern netværksadgang)
- Hvis du er usikker på, om din proxy korrekt fjerner/overskriver videresendte headers
- Hvis du kun har brug for personlig enkeltbrugeradgang (overvej Tailscale Serve + loopback for en enklere opsætning)

## Sådan fungerer det

1. Din reverse proxy godkender brugere (OAuth, OIDC, SAML osv.)
2. Proxyen tilføjer en header med den godkendte brugeridentitet (f.eks. `x-forwarded-user: nick@example.com`)
3. OpenClaw kontrollerer, at anmodningen kom fra en **betroet proxy-IP** (konfigureret i `gateway.trustedProxies`)
4. OpenClaw udtrækker brugeridentiteten fra den konfigurerede header
5. Hvis alt er i orden, godkendes anmodningen

## Konfiguration

```json5
{
  gateway: {
    // Skal binde til netværksinterface (ikke loopback)
    bind: "lan",

    // KRITISK: Tilføj kun din proxy's IP-adresse(r) her
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header der indeholder godkendt brugeridentitet (påkrævet)
        userHeader: "x-forwarded-user",

        // Valgfrit: headers der SKAL være til stede (proxy-verifikation)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Valgfrit: begræns til specifikke brugere (tom = tillad alle)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Konfigurationsreference

| Felt                                        | Påkrævet | Beskrivelse                                                                                                                    |
| ------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `gateway.trustedProxies`                    | Ja       | Array af proxy-IP-adresser, der skal have tillid. Forespørgsler fra andre IP-adresser afvises. |
| `gateway.auth.mode`                         | Ja       | Skal være `"trusted-proxy"`                                                                                                    |
| `gateway.auth.trustedProxy.userHeader`      | Ja       | Headernavn, der indeholder den godkendte brugers identitet                                                                     |
| `gateway.auth.trustedProxy.requiredHeaders` | Nej      | Yderligere headers, som skal være til stede, for at forespørgslen betragtes som betroet                                        |
| `gateway.auth.trustedProxy.allowUsers`      | Nej      | Allowliste over brugeridentiteter. Tom betyder, at alle godkendte brugere tillades.            |

## Eksempler på proxy-opsætning

### Pomerium

Pomerium sender identitet i `x-pomerium-claim-email` (eller andre claim-headers) og en JWT i `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // Pomerium's IP
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

Pomerium-konfigurationsudsnit:

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

Caddy med `caddy-security`-pluginet kan godkende brugere og sende identitets-headers videre.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (hvis på samme vært)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile-udsnit:

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

oauth2-proxy godkender brugere og sender identitet i `x-auth-request-email`.

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

nginx-konfigurationsudsnit:

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
    trustedProxies: ["172.17.0.1"], // Traefik container IP
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Sikkerhedstjekliste

Før du aktiverer trusted-proxy-godkendelse, skal du bekræfte:

- [ ] **Proxy er den eneste adgangsvej**: Gateway-porten er beskyttet af firewall mod alt andet end din proxy
- [ ] **trustedProxies er minimal**: Kun dine faktiske proxy-IP’er, ikke hele undernet
- [ ] **Proxy fjerner headers**: Din proxy overskriver (ikke tilføjer) `x-forwarded-*`-headers fra klienter
- [ ] **TLS-terminering**: Din proxy håndterer TLS; brugere forbinder via HTTPS
- [ ] **allowUsers er sat** (anbefalet): Begræns til kendte brugere frem for at tillade alle godkendte

## Sikkerhedsrevision

`openclaw security audit` vil markere trusted-proxy-godkendelse med en **kritisk** alvorlighedsgrad. Dette er tilsigtet — det er en påmindelse om, at du delegerer sikkerheden til din proxy-opsætning.

Revisionen kontrollerer for:

- Manglende `trustedProxies`-konfiguration
- Manglende `userHeader`-konfiguration
- Tom `allowUsers` (tillader enhver godkendt bruger)

## Fejlfinding

### "trusted_proxy_untrusted_source"

Anmodningen kom ikke fra en IP i `gateway.trustedProxies`. Tjek:

- Er proxy-IP’en korrekt? (Docker-container-IP’er kan ændre sig)
- Er der en load balancer foran din proxy?
- Brug `docker inspect` eller `kubectl get pods -o wide` for at finde de faktiske IP’er

### "trusted_proxy_user_missing"

Bruger-headeren var tom eller manglede. Tjek:

- Er din proxy konfigureret til at videresende identitets-headers?
- Er header-navnet korrekt? (ikke følsom over for store/små bogstaver, men stavning er vigtig)
- Er brugeren faktisk godkendt i proxyen?

### "trusted_proxy_missing_header_\*"

En påkrævet header var ikke til stede. Tjek:

- Din proxy-konfiguration for de specifikke headers
- Om headers bliver fjernet et sted i kæden

### "trusted_proxy_user_not_allowed"

Brugeren er godkendt, men findes ikke i `allowUsers`. Tilføj dem enten, eller fjern allowlisten.

### WebSocket fejler stadig

Sørg for, at din proxy:

- Understøtter WebSocket-opgraderinger (`Upgrade: websocket`, `Connection: upgrade`)
- Videresender identitets-headers ved WebSocket-opgraderingsanmodninger (ikke kun HTTP)
- Ikke har en separat godkendelsessti for WebSocket-forbindelser

## Migrering fra Token-godkendelse

Hvis du skifter fra token-godkendelse til trusted-proxy:

1. Konfigurer din proxy til at godkende brugere og videresende headers
2. Test proxy-opsætningen uafhængigt (curl med headers)
3. Opdater OpenClaw-konfigurationen med trusted-proxy-godkendelse
4. Genstart Gateway
5. Test WebSocket-forbindelser fra Control UI
6. Kør `openclaw security audit` og gennemgå resultaterne

## Relateret

- [Security](/gateway/security) — komplet sikkerhedsvejledning
- [Configuration](/gateway/configuration) — konfigurationsreference
- [Remote Access](/gateway/remote) — andre mønstre for fjernadgang
- [Tailscale](/gateway/tailscale) — enklere alternativ til kun tailnet-adgang
