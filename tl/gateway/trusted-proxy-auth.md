---
summary: "I-delegate ang gateway authentication sa isang pinagkakatiwalaang reverse proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Pagpapatakbo ng OpenClaw sa likod ng isang identity-aware proxy
  - Pag-set up ng Pomerium, Caddy, o nginx na may OAuth sa harap ng OpenClaw
  - Pag-aayos ng WebSocket 1008 unauthorized na mga error sa mga setup na may reverse proxy
---

# Trusted Proxy Auth

> ⚠️ **Security-sensitive feature.** Ganap na ipinapasa ng mode na ito ang authentication sa iyong reverse proxy. Maaaring magdulot ng hindi awtorisadong access sa iyong Gateway ang maling configuration. Basahing mabuti ang pahinang ito bago i-enable.

## Kailan Gagamitin

Gamitin ang `trusted-proxy` auth mode kapag:

- Pinapatakbo mo ang OpenClaw sa likod ng isang **identity-aware proxy** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Hinahawakan ng iyong proxy ang lahat ng authentication at ipinapasa ang identidad ng user sa pamamagitan ng headers
- Ikaw ay nasa isang Kubernetes o container environment kung saan ang proxy lamang ang daan papunta sa Gateway
- Nakakaranas ka ng WebSocket `1008 unauthorized` na mga error dahil hindi makapagpasa ang mga browser ng tokens sa WS payloads

## Kailan HINDI Gagamitin

- Kung ang iyong proxy ay hindi nag-a-authenticate ng mga user (TLS terminator o load balancer lamang)
- Kung may anumang daan papunta sa Gateway na lumalampas sa proxy (mga butas sa firewall, internal na access sa network)
- Kung hindi ka sigurado kung tama ang pag-strip/pag-overwrite ng iyong proxy sa forwarded headers
- Kung kailangan mo lamang ng personal na single-user access (isaalang-alang ang Tailscale Serve + loopback para sa mas simpleng setup)

## Paano Ito Gumagana

1. Ang iyong reverse proxy ay nag-a-authenticate ng mga user (OAuth, OIDC, SAML, atbp.)
2. Nagdaragdag ang proxy ng header na may identity ng na-authenticate na user (hal., `x-forwarded-user: nick@example.com`)
3. Sinusuri ng OpenClaw na ang request ay nagmula sa isang **pinagkakatiwalaang proxy IP** (na naka-configure sa `gateway.trustedProxies`)
4. Kinukuha ng OpenClaw ang identity ng user mula sa naka-configure na header
5. Kung maayos ang lahat ng pagsusuri, pinapahintulutan ang request

## Configuration

```json5
{
  gateway: {
    // Dapat mag-bind sa network interface (hindi loopback)
    bind: "lan",

    // KRITIKAL: Idagdag lamang dito ang IP(s) ng iyong proxy
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Header na naglalaman ng identity ng na-authenticate na user (kinakailangan)
        userHeader: "x-forwarded-user",

        // Opsyonal: mga header na DAPAT naroroon (beripikasyon ng proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Opsyonal: limitahan sa mga partikular na user (walang laman = payagan lahat)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Sanggunian sa Configuration

| Field                                       | Kinakailangan | Paglalarawan                                                                                                                                          |
| ------------------------------------------- | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Oo            | Array ng mga IP address ng proxy na pagkakatiwalaan. Ang mga request mula sa ibang IP ay tatanggihan.                 |
| `gateway.auth.mode`                         | Oo            | Dapat ay `"trusted-proxy"`                                                                                                                            |
| `gateway.auth.trustedProxy.userHeader`      | Oo            | Pangalan ng header na naglalaman ng identity ng na-authenticate na user                                                                               |
| `gateway.auth.trustedProxy.requiredHeaders` | Hindi         | Mga karagdagang header na dapat naroroon upang mapagkatiwalaan ang request                                                                            |
| `gateway.auth.trustedProxy.allowUsers`      | Hindi         | Allowlist ng mga identity ng user. Kapag walang laman, nangangahulugang payagan ang lahat ng na-authenticate na user. |

## Mga Halimbawa ng Proxy Setup

### Pomerium

Ipinapasa ng Pomerium ang identity sa `x-pomerium-claim-email` (o iba pang claim headers) at isang JWT sa `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP ng Pomerium
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

Snippet ng config ng Pomerium:

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

### Caddy na may OAuth

Ang Caddy na may `caddy-security` plugin ay maaaring mag-authenticate ng mga user at magpasa ng identity headers.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP ng Caddy (kung nasa parehong host)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Snippet ng Caddyfile:

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

Ang oauth2-proxy ay nag-a-authenticate ng mga user at ipinapasa ang pagkakakilanlan sa `x-auth-request-email`.

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

nginx config snippet:

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

### Traefik na may Forward Auth

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

## Checklist sa Seguridad

Bago paganahin ang trusted-proxy auth, tiyaking:

- [ ] **Proxy lamang ang daan**: Ang Gateway port ay naka-firewall laban sa lahat maliban sa iyong proxy
- [ ] **Minimal ang trustedProxies**: Tanging ang aktwal mong proxy IPs lamang, hindi buong subnets
- [ ] **Inaalis ng Proxy ang headers**: Pinapalitan (hindi dinaragdagan) ng iyong proxy ang `x-forwarded-*` headers mula sa mga client
- [ ] **TLS termination**: Ang iyong proxy ang humahawak ng TLS; kumokonekta ang mga user gamit ang HTTPS
- [ ] **Naka-set ang allowUsers** (inirerekomenda): Limitahan sa mga kilalang user sa halip na payagan ang sinumang authenticated

## Security Audit

Magfa-flag ang `openclaw security audit` ng trusted-proxy auth bilang **critical** na isyu sa severity. Sinadya ito — paalala ito na ipinagkakatiwala mo ang seguridad sa setup ng iyong proxy.

Sinusuri ng audit ang mga sumusunod:

- Nawawalang `trustedProxies` configuration
- Nawawalang `userHeader` configuration
- Walang laman ang `allowUsers` (pinapayagan ang sinumang authenticated na user)

## Troubleshooting

### "trusted_proxy_untrusted_source"

Ang request ay hindi nagmula sa isang IP na nasa `gateway.trustedProxies`. Suriin:

- Tama ba ang proxy IP? (Maaaring magbago ang Docker container IPs)
- May load balancer ba sa harap ng iyong proxy?
- Gamitin ang `docker inspect` o `kubectl get pods -o wide` upang makita ang aktwal na mga IP

### "trusted_proxy_user_missing"

Walang laman o nawawala ang user header. Suriin:

- Naka-configure ba ang iyong proxy na ipasa ang identity headers?
- Tama ba ang pangalan ng header? (hindi case-sensitive, pero mahalaga ang tamang spelling)
- Authenticated ba talaga ang user sa proxy?

### "trusted_proxy_missing_header_\*"

Walang isang kinakailangang header. Suriin:

- Ang configuration ng iyong proxy para sa mga partikular na header na iyon
- Kung may mga header na natatanggal sa alinmang bahagi ng chain

### "trusted_proxy_user_not_allowed"

Na-authenticate ang user ngunit wala sa `allowUsers`. Idagdag sila o alisin ang allowlist.

### Patuloy Pa Ring Nabibigo ang WebSocket

Tiyaking ang iyong proxy:

- Sumusuporta sa WebSocket upgrades (`Upgrade: websocket`, `Connection: upgrade`)
- Ipinapasa ang identity headers sa mga kahilingan ng WebSocket upgrade (hindi lang HTTP)
- Walang hiwalay na auth path para sa mga koneksyong WebSocket

## Paglipat mula sa Token Auth

Kung lilipat ka mula sa token auth patungong trusted-proxy:

1. I-configure ang iyong proxy para i-authenticate ang mga user at ipasa ang mga header
2. Subukan ang setup ng proxy nang hiwalay (curl na may mga header)
3. I-update ang OpenClaw config gamit ang trusted-proxy auth
4. I-restart ang Gateway
5. Subukan ang mga koneksyong WebSocket mula sa Control UI
6. Patakbuhin ang `openclaw security audit` at suriin ang mga resulta

## Kaugnay

- [Security](/gateway/security) — kumpletong gabay sa seguridad
- [Configuration](/gateway/configuration) — sanggunian ng config
- [Remote Access](/gateway/remote) — iba pang mga pattern ng remote access
- [Tailscale](/gateway/tailscale) — mas simpleng alternatibo para sa tailnet-only access

