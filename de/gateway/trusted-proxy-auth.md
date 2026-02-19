---
summary: "Delegieren Sie die Gateway-Authentifizierung an einen vertrauenswürdigen Reverse Proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - OpenClaw hinter einem Identity-Aware-Proxy betreiben
  - Einrichten von Pomerium, Caddy oder nginx mit OAuth vor OpenClaw
  - Beheben von WebSocket-`1008 unauthorized`-Fehlern bei Reverse-Proxy-Setups
---

# Trusted Proxy Auth

> ⚠️ **Sicherheitskritische Funktion.** Dieser Modus delegiert die Authentifizierung vollständig an Ihren Reverse Proxy. Eine Fehlkonfiguration kann Ihr Gateway unbefugtem Zugriff aussetzen. Lesen Sie diese Seite sorgfältig durch, bevor Sie dies aktivieren.

## Wann verwenden

Verwenden Sie den Auth-Modus `trusted-proxy`, wenn:

- Sie OpenClaw hinter einem **Identity-Aware-Proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) betreiben
- Ihr Proxy die gesamte Authentifizierung übernimmt und die Benutzeridentität über Header weitergibt
- Sie sich in einer Kubernetes- oder Container-Umgebung befinden, in der der Proxy der einzige Zugangsweg zum Gateway ist
- Sie auf WebSocket-`1008 unauthorized`-Fehler stoßen, weil Browser keine Tokens in WS-Payloads übergeben können

## Wann NICHT verwenden

- Wenn Ihr Proxy Benutzer nicht authentifiziert (nur TLS-Terminierung oder Load Balancer)
- Wenn es irgendeinen Zugangsweg zum Gateway gibt, der den Proxy umgeht (Firewall-Lücken, interner Netzwerkzugang)
- Wenn Sie unsicher sind, ob Ihr Proxy weitergeleitete Header korrekt entfernt/überschreibt
- Wenn Sie nur persönlichen Einzelbenutzerzugang benötigen (erwägen Sie Tailscale Serve + Loopback für eine einfachere Einrichtung)

## Funktionsweise

1. Ihr Reverse Proxy authentifiziert Benutzer (OAuth, OIDC, SAML usw.)
2. Der Proxy fügt einen Header mit der authentifizierten Benutzeridentität hinzu (z. B. `x-forwarded-user: nick@example.com`)
3. OpenClaw prüft, ob die Anfrage von einer **vertrauenswürdigen Proxy-IP** stammt (konfiguriert in `gateway.trustedProxies`)
4. OpenClaw extrahiert die Benutzeridentität aus dem konfigurierten Header
5. Wenn alles korrekt ist, wird die Anfrage autorisiert

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

### Konfigurationsreferenz

| Feld                                        | Erforderlich | Beschreibung                                                                                                                            |
| ------------------------------------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Ja           | Array von Proxy-IP-Adressen, denen vertraut wird. Anfragen von anderen IPs werden abgelehnt.            |
| `gateway.auth.mode`                         | Ja           | Muss `"trusted-proxy"` sein                                                                                                             |
| `gateway.auth.trustedProxy.userHeader`      | Ja           | Header-Name, der die Identität des authentifizierten Benutzers enthält                                                                  |
| `gateway.auth.trustedProxy.requiredHeaders` | Nein         | Zusätzliche Header, die vorhanden sein müssen, damit die Anfrage als vertrauenswürdig gilt                                              |
| `gateway.auth.trustedProxy.allowUsers`      | Nein         | Allowlist von Benutzeridentitäten. Leer bedeutet, dass alle authentifizierten Benutzer zugelassen sind. |

## Beispiele für Proxy-Setup

### Pomerium

Pomerium übergibt die Identität in `x-pomerium-claim-email` (oder anderen Claim-Headern) sowie ein JWT in `x-pomerium-jwt-assertion`.

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

Pomerium-Konfigurationsausschnitt:

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

### Caddy mit OAuth

Caddy mit dem Plugin `caddy-security` kann Benutzer authentifizieren und Identitäts-Header weitergeben.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // Caddy's IP (if on same host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Caddyfile-Ausschnitt:

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

oauth2-proxy authentifiziert Benutzer und übergibt die Identität in `x-auth-request-email`.

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

nginx-Konfigurationsausschnitt:

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

### Traefik mit Forward Auth

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

## Sicherheits-Checkliste

Bevor Sie die trusted-proxy-Authentifizierung aktivieren, prüfen Sie:

- [ ] **Proxy ist der einzige Zugriffspfad**: Der Gateway-Port ist durch eine Firewall geschützt und nur über Ihren Proxy erreichbar
- [ ] **trustedProxies ist minimal**: Nur die tatsächlichen Proxy-IP-Adressen, keine gesamten Subnetze
- [ ] **Proxy entfernt Header**: Ihr Proxy überschreibt (und hängt nicht an) `x-forwarded-*`-Header von Clients
- [ ] **TLS-Terminierung**: Ihr Proxy übernimmt TLS; Benutzer verbinden sich über HTTPS
- [ ] **allowUsers ist gesetzt** (empfohlen): Beschränken Sie den Zugriff auf bekannte Benutzer, anstatt allen authentifizierten Benutzern Zugriff zu gewähren

## Sicherheitsaudit

`openclaw security audit` meldet Trusted-Proxy-Authentifizierung mit einem **critical**-Schweregrad. Dies ist beabsichtigt — es erinnert Sie daran, dass Sie die Sicherheit an Ihre Proxy-Konfiguration delegieren.

Das Audit prüft auf:

- Fehlende `trustedProxies`-Konfiguration
- Fehlende `userHeader`-Konfiguration
- Leere `allowUsers` (erlaubt jedem authentifizierten Benutzer den Zugriff)

## Fehlerbehebung

### "trusted_proxy_untrusted_source"

Die Anfrage stammt nicht von einer IP in `gateway.trustedProxies`. Überprüfen Sie:

- Ist die Proxy-IP korrekt? (Docker-Container-IPs können sich ändern)
- Befindet sich ein Load Balancer vor Ihrem Proxy?
- Verwenden Sie `docker inspect` oder `kubectl get pods -o wide`, um die tatsächlichen IPs zu ermitteln

### "trusted_proxy_user_missing"

Der User-Header war leer oder fehlte. Überprüfen Sie:

- Ist Ihr Proxy so konfiguriert, dass er Identity-Header weiterleitet?
- Ist der Header-Name korrekt? (Groß-/Kleinschreibung wird nicht berücksichtigt, aber die Schreibweise muss stimmen)
- Ist der Benutzer am Proxy tatsächlich authentifiziert?

### "trusted_proxy_missing_header_\*"

Ein erforderlicher Header war nicht vorhanden. Überprüfen Sie:

- Ihre Proxy-Konfiguration für diese spezifischen Header
- Ob Header irgendwo in der Kette entfernt werden

### "trusted_proxy_user_not_allowed"

Der Benutzer ist authentifiziert, befindet sich jedoch nicht in `allowUsers`. Fügen Sie ihn entweder hinzu oder entfernen Sie die Allowlist.

### WebSocket schlägt weiterhin fehl

Stellen Sie sicher, dass Ihr Proxy:

- WebSocket-Upgrades unterstützt (`Upgrade: websocket`, `Connection: upgrade`)
- Die Identity-Header bei WebSocket-Upgrade-Anfragen weiterleitet (nicht nur bei HTTP)
- Keinen separaten Authentifizierungspfad für WebSocket-Verbindungen hat

## Migration von Token-Authentifizierung

Wenn Sie von Token-Authentifizierung zu Trusted-Proxy wechseln:

1. Konfigurieren Sie Ihren Proxy so, dass er Benutzer authentifiziert und Header weiterleitet
2. Testen Sie die Proxy-Konfiguration unabhängig (curl mit Headern)
3. Aktualisieren Sie die OpenClaw-Konfiguration mit Trusted-Proxy-Authentifizierung
4. Gateway neu starten
5. WebSocket-Verbindungen aus der Control UI testen
6. `openclaw security audit` ausführen und Ergebnisse prüfen

## Verwandt

- [Security](/gateway/security) — vollständiger Sicherheitsleitfaden
- [Configuration](/gateway/configuration) — Konfigurationsreferenz
- [Remote Access](/gateway/remote) — weitere Remote-Zugriffsmuster
- [Tailscale](/gateway/tailscale) — einfachere Alternative für reinen Tailnet-Zugriff
