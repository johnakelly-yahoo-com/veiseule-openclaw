---
summary: "Déléguer l’authentification du Gateway à un reverse proxy de confiance (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Exécuter OpenClaw derrière un proxy sensible à l’identité
  - Configurer Pomerium, Caddy ou nginx avec OAuth devant OpenClaw
  - Résoudre les erreurs WebSocket `1008 unauthorized` avec des configurations de reverse proxy
---

# Authentification par proxy de confiance

> ⚠️ **Fonctionnalité sensible du point de vue de la sécurité.** Ce mode délègue entièrement l’authentification à votre reverse proxy. Une mauvaise configuration peut exposer votre Gateway à des accès non autorisés. Lisez attentivement cette page avant d’activer cette option.

## Quand l’utiliser

Utilisez le mode d’authentification `trusted-proxy` lorsque :

- Vous exécutez OpenClaw derrière un **proxy sensible à l’identité** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Votre proxy gère toute l’authentification et transmet l’identité utilisateur via des en-têtes
- Vous êtes dans un environnement Kubernetes ou conteneurisé où le proxy est le seul point d’accès au Gateway
- Vous rencontrez des erreurs WebSocket `1008 unauthorized` car les navigateurs ne peuvent pas transmettre de jetons dans les charges utiles WS

## Quand ne PAS l’utiliser

- Si votre proxy n’authentifie pas les utilisateurs (simple terminateur TLS ou équilibreur de charge)
- S’il existe un chemin vers le Gateway qui contourne le proxy (failles de pare-feu, accès réseau interne)
- Si vous n’êtes pas certain que votre proxy supprime/remplace correctement les en-têtes transférés
- Si vous avez seulement besoin d’un accès personnel mono-utilisateur (envisagez Tailscale Serve + loopback pour une configuration plus simple)

## Comment ça fonctionne

1. Votre reverse proxy authentifie les utilisateurs (OAuth, OIDC, SAML, etc.)
2. Le proxy ajoute un en-tête contenant l’identité de l’utilisateur authentifié (ex. : `x-forwarded-user: nick@example.com`)
3. OpenClaw vérifie que la requête provient d’une **IP de proxy de confiance** (configurée dans `gateway.trustedProxies`)
4. OpenClaw extrait l’identité utilisateur à partir de l’en-tête configuré
5. Si tout est conforme, la requête est autorisée

## Configuration

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

### Référence de configuration

| Champ                                       | Obligatoire | Description                                                                                                                                             |
| ------------------------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Oui         | Liste des adresses IP des proxys à considérer comme fiables. Les requêtes provenant d’autres adresses IP sont rejetées. |
| `gateway.auth.mode`                         | Oui         | Doit être "trusted-proxy"                                                                                                                               |
| `gateway.auth.trustedProxy.userHeader`      | Oui         | Nom de l’en-tête contenant l’identité de l’utilisateur authentifié                                                                                      |
| `gateway.auth.trustedProxy.requiredHeaders` | Non         | En-têtes supplémentaires qui doivent être présents pour que la requête soit considérée comme fiable                                                     |
| `gateway.auth.trustedProxy.allowUsers`      | Non         | Liste blanche des identités d’utilisateurs. Vide signifie autoriser tous les utilisateurs authentifiés.                 |

## Exemples de configuration de proxy

### Pomerium

Pomerium transmet l’identité dans `x-pomerium-claim-email` (ou d’autres en-têtes de revendication) ainsi qu’un JWT dans `x-pomerium-jwt-assertion`.

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

Extrait de configuration Pomerium :

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

### Caddy avec OAuth

Caddy avec le plugin `caddy-security` peut authentifier les utilisateurs et transmettre les en-têtes d’identité.

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

Extrait de Caddyfile :

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

oauth2-proxy authentifie les utilisateurs et transmet l’identité dans `x-auth-request-email`.

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

Extrait de configuration nginx :

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

### Traefik avec Forward Auth

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

## Liste de vérification de sécurité

Avant d’activer l’authentification trusted-proxy, vérifiez :

- [ ] **Le proxy est l’unique point d’accès** : Le port du Gateway est protégé par un pare-feu et inaccessible à tout autre service que votre proxy
- [ ] **trustedProxies est minimal** : Uniquement les adresses IP réelles de votre proxy, pas des sous-réseaux entiers
- [ ] **Le proxy supprime les en-têtes** : Votre proxy écrase (et n’ajoute pas) les en-têtes `x-forwarded-*` provenant des clients
- [ ] **Terminaison TLS** : Votre proxy gère TLS ; les utilisateurs se connectent via HTTPS
- [ ] **allowUsers est défini** (recommandé) : Limitez l’accès aux utilisateurs connus plutôt que d’autoriser toute personne authentifiée

## Audit de sécurité

`openclaw security audit` signalera l’authentification trusted-proxy avec un niveau de sévérité **critique**. C’est intentionnel — cela vous rappelle que vous déléguez la sécurité à la configuration de votre proxy.

L’audit vérifie :

- Configuration `trustedProxies` manquante
- Configuration `userHeader` manquante
- `allowUsers` vide (autorise tout utilisateur authentifié)

## Dépannage

### "trusted_proxy_untrusted_source"

La requête ne provient pas d’une IP présente dans `gateway.trustedProxies`. Vérifiez :

- L’IP du proxy est-elle correcte ? (Les IP des conteneurs Docker peuvent changer)
- Y a-t-il un équilibreur de charge devant votre proxy ?
- Utilisez `docker inspect` ou `kubectl get pods -o wide` pour trouver les IP réelles

### "trusted_proxy_user_missing"

L’en-tête utilisateur était vide ou manquant. Vérifiez :

- Votre proxy est-il configuré pour transmettre les en-têtes d’identité ?
- Le nom de l’en-tête est-il correct ? (insensible à la casse, mais l’orthographe compte)
- L’utilisateur est-il réellement authentifié au niveau du proxy ?

### "trusted_proxy_missing_header_\*"

Un en-tête requis était absent. Vérifiez :

- La configuration de votre proxy pour ces en-têtes spécifiques
- Si des en-têtes sont supprimés quelque part dans la chaîne

### "trusted_proxy_user_not_allowed"

L’utilisateur est authentifié mais ne figure pas dans `allowUsers`. Ajoutez-le ou supprimez la liste d’autorisation.

### WebSocket échoue toujours

Assurez-vous que votre proxy :

- Prend en charge les mises à niveau WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Transmet les en-têtes d’identité lors des requêtes de mise à niveau WebSocket (pas seulement HTTP)
- N’a pas de chemin d’authentification distinct pour les connexions WebSocket

## Migration depuis l’authentification par token

Si vous passez de l’authentification par token à trusted-proxy :

1. Configurez votre proxy pour authentifier les utilisateurs et transmettre les en-têtes
2. Testez la configuration du proxy indépendamment (curl avec en-têtes)
3. Mettez à jour la configuration OpenClaw avec l’authentification trusted-proxy
4. Redémarrez le Gateway
5. Tester les connexions WebSocket depuis l’interface de contrôle
6. Exécutez `openclaw security audit` et examinez les résultats

## Connexe

- [Security](/gateway/security) — guide complet de sécurité
- [Configuration](/gateway/configuration) — référence de configuration
- [Remote Access](/gateway/remote) — autres modèles d’accès à distance
- [Tailscale](/gateway/tailscale) — alternative plus simple pour un accès limité au tailnet
