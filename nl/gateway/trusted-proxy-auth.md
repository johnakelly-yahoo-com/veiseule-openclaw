---
summary: "Delegeer gateway-authenticatie aan een vertrouwde reverse proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - OpenClaw draaien achter een identity-aware proxy
  - Pomerium, Caddy of nginx met OAuth instellen vóór OpenClaw
  - WebSocket 1008 unauthorized-fouten oplossen met reverse proxy-configuraties
---

# Trusted Proxy Auth

> ⚠️ **Beveiligingsgevoelige functie.** Deze modus delegeert authenticatie volledig aan je reverse proxy. Een verkeerde configuratie kan je Gateway blootstellen aan ongeautoriseerde toegang. Lees deze pagina zorgvuldig voordat je dit inschakelt.

## Wanneer te gebruiken

Gebruik de `trusted-proxy`-authmodus wanneer:

- Je OpenClaw achter een **identity-aware proxy** draait (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Je proxy alle authenticatie afhandelt en de gebruikersidentiteit via headers doorgeeft
- Je je in een Kubernetes- of containeromgeving bevindt waar de proxy de enige route naar de Gateway is
- Je WebSocket `1008 unauthorized`-fouten krijgt omdat browsers geen tokens in WS-payloads kunnen doorgeven

## Wanneer NIET gebruiken

- Als je proxy geen gebruikers authenticeert (alleen een TLS-terminator of load balancer)
- Als er een pad naar de Gateway bestaat dat de proxy omzeilt (firewall-openingen, interne netwerktoegang)
- Als je niet zeker weet of je proxy forwarded headers correct verwijdert/overschrijft
- Als je alleen persoonlijke single-user-toegang nodig hebt (overweeg Tailscale Serve + loopback voor een eenvoudigere configuratie)

## Hoe het werkt

1. Je reverse proxy authenticeert gebruikers (OAuth, OIDC, SAML, enz.)
2. De proxy voegt een header toe met de geauthenticeerde gebruikersidentiteit (bijv. `x-forwarded-user: nick@example.com`)
3. OpenClaw controleert of het verzoek afkomstig is van een **vertrouwd proxy-IP** (geconfigureerd in `gateway.trustedProxies`)
4. OpenClaw haalt de gebruikersidentiteit uit de geconfigureerde header
5. Als alles klopt, wordt het verzoek geautoriseerd

## Configuratie

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

### Configuratiereferentie

| Veld                                        | Vereist | Beschrijving                                                                                                                                       |
| ------------------------------------------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Ja      | Array van proxy-IP-adressen die vertrouwd worden. Verzoeken van andere IP-adressen worden geweigerd.               |
| `gateway.auth.mode`                         | Ja      | Moet `"trusted-proxy"` zijn                                                                                                                        |
| `gateway.auth.trustedProxy.userHeader`      | Ja      | Headernaam die de geauthenticeerde gebruikersidentiteit bevat                                                                                      |
| `gateway.auth.trustedProxy.requiredHeaders` | Nee     | Aanvullende headers die aanwezig moeten zijn zodat het verzoek wordt vertrouwd                                                                     |
| `gateway.auth.trustedProxy.allowUsers`      | Nee     | Toegestane lijst van gebruikersidentiteiten. Leeg betekent dat alle geauthenticeerde gebruikers worden toegestaan. |

## Voorbeelden van proxyconfiguratie

### Pomerium

Pomerium geeft identiteit door in `x-pomerium-claim-email` (of andere claim-headers) en een JWT in `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP van Pomerium
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

Pomerium-configuratiefragment:

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

### Caddy met OAuth

Caddy met de `caddy-security`-plugin kan gebruikers authenticeren en identiteit-headers doorgeven.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP van Caddy (indien op dezelfde host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile-fragment:

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

oauth2-proxy authenticeert gebruikers en geeft identiteit door in `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP van nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

nginx-configuratiefragment:

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

### Traefik met Forward Auth

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["172.17.0.1"], // IP van de Traefik-container
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

## Beveiligingschecklist

Voordat je trusted-proxy-auth inschakelt, controleer het volgende:

- [ ] **Proxy is het enige pad**: De Gateway-poort is via een firewall afgeschermd voor alles behalve je proxy
- [ ] **trustedProxies is minimaal**: Alleen de daadwerkelijke proxy-IP’s, niet volledige subnetten
- [ ] **Proxy verwijdert headers**: Je proxy overschrijft (voegt niet toe aan) `x-forwarded-*`-headers van clients
- [ ] **TLS-terminatie**: Je proxy verwerkt TLS; gebruikers verbinden via HTTPS
- [ ] **allowUsers is ingesteld** (aanbevolen): Beperk tot bekende gebruikers in plaats van iedereen toe te staan die is geauthenticeerd

## Beveiligingsaudit

`openclaw security audit` markeert trusted-proxy-auth met een bevinding met **kritieke** ernst. Dit is opzettelijk — het is een herinnering dat je de beveiliging delegeert aan je proxyconfiguratie.

De audit controleert op:

- Ontbrekende `trustedProxies`-configuratie
- Ontbrekende `userHeader`-configuratie
- Lege `allowUsers` (staat elke geauthenticeerde gebruiker toe)

## Probleemoplossing

### "trusted_proxy_untrusted_source"

Het verzoek is niet afkomstig van een IP-adres in `gateway.trustedProxies`. Controleer:

- Is het proxy-IP correct? (Docker container-IP's kunnen veranderen)
- Staat er een load balancer voor je proxy?
- Gebruik `docker inspect` of `kubectl get pods -o wide` om de daadwerkelijke IP's te vinden

### "trusted_proxy_user_missing"

De gebruikersheader was leeg of ontbrak. Controleer:

- Is je proxy geconfigureerd om identity-headers door te geven?
- Is de headernaam correct? (niet hoofdlettergevoelig, maar de spelling is belangrijk)
- Is de gebruiker daadwerkelijk geauthenticeerd bij de proxy?

### "trusted_proxy_missing_header_\*"

Een vereiste header ontbrak. Controleer:

- Je proxyconfiguratie voor die specifieke headers
- Of headers ergens in de keten worden verwijderd

### "trusted_proxy_user_not_allowed"

De gebruiker is geauthenticeerd maar staat niet in `allowUsers`. Voeg de gebruiker toe of verwijder de allowlist.

### WebSocket faalt nog steeds

Zorg ervoor dat je proxy:

- WebSocket-upgrades ondersteunt (`Upgrade: websocket`, `Connection: upgrade`)
- De identity-headers doorgeeft bij WebSocket-upgradeverzoeken (niet alleen HTTP)
- Geen apart authenticatiepad heeft voor WebSocket-verbindingen

## Migratie van Token-authenticatie

Als je overstapt van token-auth naar trusted-proxy:

1. Configureer je proxy om gebruikers te authenticeren en headers door te geven
2. Test de proxyconfiguratie afzonderlijk (curl met headers)
3. Werk de OpenClaw-configuratie bij met trusted-proxy-auth
4. Herstart de Gateway
5. Test WebSocket-verbindingen vanuit de Control UI
6. Voer `openclaw security audit` uit en bekijk de bevindingen

## Gerelateerd

- [Security](/gateway/security) — volledige beveiligingsgids
- [Configuration](/gateway/configuration) — configuratiereferentie
- [Remote Access](/gateway/remote) — andere patronen voor externe toegang
- [Tailscale](/gateway/tailscale) — eenvoudiger alternatief voor alleen tailnet-toegang

