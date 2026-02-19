---
summary: "Deleguj uwierzytelnianie Gateway do zaufanego odwrotnego proxy (Pomerium, Caddy, nginx + OAuth)"
read_when:
  - Uruchamianie OpenClaw za proxy świadomym tożsamości
  - Konfigurowanie Pomerium, Caddy lub nginx z OAuth przed OpenClaw
  - Naprawa błędów WebSocket `1008 unauthorized` w konfiguracjach z odwrotnym proxy
---

# Trusted Proxy Auth

> ⚠️ **Funkcja wrażliwa na bezpieczeństwo.** Ten tryb w pełni deleguje uwierzytelnianie do Twojego odwrotnego proxy. Błędna konfiguracja może narazić Twój Gateway na nieautoryzowany dostęp. Przed włączeniem przeczytaj uważnie tę stronę.

## Kiedy używać

Użyj trybu uwierzytelniania `trusted-proxy`, gdy:

- Uruchamiasz OpenClaw za **proxy świadomym tożsamości** (Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth)
- Twoje proxy obsługuje całe uwierzytelnianie i przekazuje tożsamość użytkownika przez nagłówki
- Działasz w środowisku Kubernetes lub kontenerowym, gdzie proxy jest jedyną drogą do Gateway
- Napotykasz błędy WebSocket `1008 unauthorized`, ponieważ przeglądarki nie mogą przekazywać tokenów w payloadach WS

## Kiedy NIE używać

- Jeśli Twoje proxy nie uwierzytelnia użytkowników (jest tylko terminatorem TLS lub load balancerem)
- Jeśli istnieje jakakolwiek ścieżka do Gateway omijająca proxy (dziury w firewallu, dostęp z sieci wewnętrznej)
- Jeśli nie masz pewności, czy Twoje proxy prawidłowo usuwa/nadpisuje przekazywane nagłówki
- Jeśli potrzebujesz jedynie osobistego dostępu dla jednego użytkownika (rozważ Tailscale Serve + loopback dla prostszej konfiguracji)

## Jak to działa

1. Twoje odwrotne proxy uwierzytelnia użytkowników (OAuth, OIDC, SAML itp.)
2. Proxy dodaje nagłówek z tożsamością uwierzytelnionego użytkownika (np. `x-forwarded-user: nick@example.com`)
3. OpenClaw sprawdza, czy żądanie pochodzi z **zaufanego adresu IP proxy** (skonfigurowanego w `gateway.trustedProxies`)
4. OpenClaw wyodrębnia tożsamość użytkownika z skonfigurowanego nagłówka
5. Jeśli wszystko się zgadza, żądanie zostaje autoryzowane

## Konfiguracja

```json5
{
  gateway: {
    // Musi być powiązane z interfejsem sieciowym (nie loopback)
    bind: "lan",

    // KRYTYCZNE: Dodaj tutaj tylko adres(y) IP swojego proxy
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // Nagłówek zawierający tożsamość uwierzytelnionego użytkownika (wymagany)
        userHeader: "x-forwarded-user",

        // Opcjonalne: nagłówki, które MUSZĄ być obecne (weryfikacja proxy)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // Opcjonalne: ograniczenie do określonych użytkowników (puste = zezwól wszystkim)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### Opis konfiguracji

| Pole                                        | Wymagane | Opis                                                                                                                                         |
| ------------------------------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | Tak      | Tablica adresów IP proxy, którym można ufać. Żądania z innych adresów IP są odrzucane.                       |
| `gateway.auth.mode`                         | Tak      | Musi mieć wartość `"trusted-proxy"`                                                                                                          |
| `gateway.auth.trustedProxy.userHeader`      | Tak      | Nazwa nagłówka zawierającego tożsamość uwierzytelnionego użytkownika                                                                         |
| `gateway.auth.trustedProxy.requiredHeaders` | Nie      | Dodatkowe nagłówki, które muszą być obecne, aby żądanie zostało uznane za zaufane                                                            |
| `gateway.auth.trustedProxy.allowUsers`      | Nie      | Lista dozwolonych tożsamości użytkowników. Puste oznacza zezwolenie wszystkim uwierzytelnionym użytkownikom. |

## Przykłady konfiguracji proxy

### Pomerium

Pomerium przekazuje tożsamość w `x-pomerium-claim-email` (lub innych nagłówkach z deklaracjami) oraz JWT w `x-pomerium-jwt-assertion`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP Pomerium
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

Fragment konfiguracji Pomerium:

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

### Caddy z OAuth

Caddy z wtyczką `caddy-security` może uwierzytelniać użytkowników i przekazywać nagłówki tożsamości.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["127.0.0.1"], // IP Caddy (jeśli na tym samym hoście)
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-forwarded-user",
      },
    },
  },
}
```

Fragment Caddyfile:

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

oauth2-proxy uwierzytelnia użytkowników i przekazuje tożsamość w `x-auth-request-email`.

```json5
{
  gateway: {
    bind: "lan",
    trustedProxies: ["10.0.0.1"], // IP nginx/oauth2-proxy
    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        userHeader: "x-auth-request-email",
      },
    },
  },
}
```

fragment konfiguracji nginx:

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

### Traefik z Forward Auth

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

## Lista kontrolna bezpieczeństwa

Przed włączeniem uwierzytelniania trusted-proxy sprawdź:

- [ ] **Proxy to jedyna ścieżka**: Port Gateway jest chroniony zaporą i dostępny wyłącznie dla Twojego proxy
- [ ] **trustedProxies jest minimalne**: Tylko rzeczywiste adresy IP Twojego proxy, nie całe podsieci
- [ ] **Proxy usuwa nagłówki**: Twoje proxy nadpisuje (nie dopisuje) nagłówki `x-forwarded-*` od klientów
- [ ] **Zakończenie TLS**: Twoje proxy obsługuje TLS; użytkownicy łączą się przez HTTPS
- [ ] **allowUsers jest ustawione** (zalecane): Ogranicz dostęp do znanych użytkowników zamiast zezwalać wszystkim uwierzytelnionym

## Audyt bezpieczeństwa

`openclaw security audit` oznaczy uwierzytelnianie trusted-proxy jako problem o **krytycznym** poziomie. Jest to celowe — przypomnienie, że delegujesz bezpieczeństwo do konfiguracji swojego proxy.

Audyt sprawdza:

- Brak konfiguracji `trustedProxies`
- Brak konfiguracji `userHeader`
- Puste `allowUsers` (zezwala każdemu uwierzytelnionemu użytkownikowi)

## Rozwiązywanie problemów

### "trusted_proxy_untrusted_source"

Żądanie nie pochodziło z adresu IP znajdującego się w `gateway.trustedProxies`. Sprawdź:

- Czy adres IP proxy jest poprawny? (Adresy IP kontenerów Docker mogą się zmieniać)
- Czy przed Twoim proxy znajduje się load balancer?
- Użyj `docker inspect` lub `kubectl get pods -o wide`, aby znaleźć rzeczywiste adresy IP

### "trusted_proxy_user_missing"

Nagłówek użytkownika był pusty lub nieobecny. Sprawdź:

- Czy Twoje proxy jest skonfigurowane do przekazywania nagłówków tożsamości?
- Czy nazwa nagłówka jest poprawna? (bez rozróżniania wielkości liter, ale pisownia ma znaczenie)
- Czy użytkownik jest faktycznie uwierzytelniony w proxy?

### "trusted_proxy_missing_header_\*"

Wymagany nagłówek nie był obecny. Sprawdź:

- Konfigurację proxy dla tych konkretnych nagłówków
- Czy nagłówki nie są usuwane gdzieś po drodze

### "trusted_proxy_user_not_allowed"

Użytkownik jest uwierzytelniony, ale nie znajduje się w `allowUsers`. Dodaj je albo usuń allowlistę.

### WebSocket nadal nie działa

Upewnij się, że Twój proxy:

- Obsługuje aktualizację do WebSocket (`Upgrade: websocket`, `Connection: upgrade`)
- Przekazuje nagłówki tożsamości przy żądaniach aktualizacji WebSocket (nie tylko HTTP)
- Nie ma osobnej ścieżki uwierzytelniania dla połączeń WebSocket

## Migracja z uwierzytelniania tokenem

Jeśli przechodzisz z uwierzytelniania tokenem na trusted-proxy:

1. Skonfiguruj proxy tak, aby uwierzytelniał użytkowników i przekazywał nagłówki
2. Przetestuj konfigurację proxy niezależnie (curl z nagłówkami)
3. Zaktualizuj konfigurację OpenClaw o uwierzytelnianie trusted-proxy
4. Uruchom ponownie Gateway
5. Przetestuj połączenia WebSocket z Control UI
6. Uruchom `openclaw security audit` i przejrzyj wyniki

## Powiązane

- [Security](/gateway/security) — pełny przewodnik po bezpieczeństwie
- [Configuration](/gateway/configuration) — dokumentacja konfiguracji
- [Remote Access](/gateway/remote) — inne wzorce dostępu zdalnego
- [Tailscale](/gateway/tailscale) — prostsza alternatywa dla dostępu tylko w tailnet
