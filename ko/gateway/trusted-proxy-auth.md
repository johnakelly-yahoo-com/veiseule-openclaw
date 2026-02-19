---
summary: "Gateway 인증을 신뢰할 수 있는 리버스 프록시(Pomerium, Caddy, nginx + OAuth)에 위임하세요"
read_when:
  - identity-aware 프록시 뒤에서 OpenClaw 실행하기
  - OpenClaw 앞단에 Pomerium, Caddy 또는 nginx + OAuth 설정하기
  - 리버스 프록시 구성에서 WebSocket 1008 unauthorized 오류 해결하기
---

# 신뢰된 프록시 인증

> ⚠️ **보안에 민감한 기능입니다.** 이 모드는 인증을 전적으로 리버스 프록시에 위임합니다. 잘못 구성하면 Gateway가 무단 접근에 노출될 수 있습니다. 활성화하기 전에 이 페이지를 주의 깊게 읽으세요.

## 사용해야 하는 경우

`trusted-proxy` 인증 모드는 다음과 같은 경우에 사용하세요:

- OpenClaw를 **identity-aware 프록시**(Pomerium, Caddy + OAuth, nginx + oauth2-proxy, Traefik + forward auth) 뒤에서 실행하는 경우
- 프록시가 모든 인증을 처리하고 헤더를 통해 사용자 식별 정보를 전달하는 경우
- 프록시가 Gateway로 향하는 유일한 경로인 Kubernetes 또는 컨테이너 환경에서 실행하는 경우
- 브라우저가 WS 페이로드에 토큰을 전달할 수 없어 WebSocket `1008 unauthorized` 오류가 발생하는 경우

## 사용하지 말아야 하는 경우

- 프록시가 사용자 인증을 수행하지 않는 경우(TLS 종료기 또는 로드 밸런서에 불과한 경우)
- 프록시를 우회해 Gateway에 접근할 수 있는 경로가 존재하는 경우(방화벽 허점, 내부 네트워크 접근 등)
- 프록시가 전달된 헤더를 올바르게 제거/덮어쓰는지 확실하지 않은 경우
- 개인 단일 사용자 접근만 필요한 경우(더 간단한 설정을 위해 Tailscale Serve + loopback 사용을 고려하세요)

## 작동 방식

1. 리버스 프록시가 사용자 인증을 수행합니다(OAuth, OIDC, SAML 등).
2. 프록시는 인증된 사용자 식별 정보가 포함된 헤더를 추가합니다(예: `x-forwarded-user: nick@example.com`).
3. OpenClaw는 요청이 **신뢰된 프록시 IP**( `gateway.trustedProxies`에 설정됨 )에서 왔는지 확인합니다.
4. OpenClaw는 설정된 헤더에서 사용자 식별 정보를 추출합니다.
5. 모든 확인이 완료되면 요청이 승인됩니다.

## 설정

```json5
{
  gateway: {
    // 네트워크 인터페이스에 바인딩해야 함 (loopback 아님)
    bind: "lan",

    // 중요: 여기에 프록시의 IP만 추가하세요
    trustedProxies: ["10.0.0.1", "172.17.0.1"],

    auth: {
      mode: "trusted-proxy",
      trustedProxy: {
        // 인증된 사용자 식별 정보가 포함된 헤더 (필수)
        userHeader: "x-forwarded-user",

        // 선택 사항: 반드시 존재해야 하는 헤더 (프록시 검증)
        requiredHeaders: ["x-forwarded-proto", "x-forwarded-host"],

        // 선택 사항: 특정 사용자로 제한 (비어 있으면 모두 허용)
        allowUsers: ["nick@example.com", "admin@company.org"],
      },
    },
  },
}
```

### 설정 참조

| 필드                                          | 필수 여부 | 설명                                                                           |
| ------------------------------------------- | ----- | ---------------------------------------------------------------------------- |
| `gateway.trustedProxies`                    | 예     | 신뢰할 프록시 IP 주소 배열입니다. 다른 IP에서 오는 요청은 거부됩니다.   |
| `gateway.auth.mode`                         | 예     | 반드시 `"trusted-proxy"`여야 합니다.                                 |
| `gateway.auth.trustedProxy.userHeader`      | 예     | 인증된 사용자 식별자가 포함된 헤더 이름                                                       |
| `gateway.auth.trustedProxy.requiredHeaders` | 아니요   | 요청을 신뢰하기 위해 반드시 존재해야 하는 추가 헤더                                                |
| `gateway.auth.trustedProxy.allowUsers`      | 아니요   | 허용된 사용자 식별자 목록입니다. 비어 있으면 모든 인증된 사용자를 허용합니다. |

## 프록시 설정 예시

### Pomerium

Pomerium은 `x-pomerium-claim-email`(또는 기타 claim 헤더)에 사용자 식별 정보를 전달하고, `x-pomerium-jwt-assertion`에 JWT를 전달합니다.

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

Pomerium 설정 스니펫:

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

### OAuth와 함께 사용하는 Caddy

`caddy-security` 플러그인을 사용하는 Caddy는 사용자를 인증하고 사용자 식별 헤더를 전달할 수 있습니다.

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

Caddyfile 스니펫:

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

oauth2-proxy는 사용자를 인증하고 `x-auth-request-email`에 사용자 식별 정보를 전달합니다.

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

nginx 설정 스니펫:

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

### Forward Auth와 함께 사용하는 Traefik

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

## 보안 체크리스트

trusted-proxy 인증을 활성화하기 전에 다음을 확인하세요:

- [ ] **프록시가 유일한 경로인지 확인**: Gateway 포트가 프록시를 제외한 모든 접근으로부터 방화벽으로 차단되어 있어야 합니다.
- [ ] **trustedProxies를 최소화**: 전체 서브넷이 아니라 실제 프록시 IP만 포함해야 합니다.
- [ ] **프록시가 헤더를 제거하는지 확인**: 프록시가 클라이언트로부터 받은 `x-forwarded-*` 헤더를 추가(append)하지 않고 덮어써야(overwrite) 합니다.
- [ ] **TLS 종료**: 프록시가 TLS를 처리하며, 사용자는 HTTPS를 통해 연결해야 합니다.
- [ ] **allowUsers 설정** (권장): 인증된 누구나 허용하지 말고, 알려진 사용자로 제한하세요.

## 보안 감사

`openclaw security audit`는 trusted-proxy 인증에 대해 **critical** 심각도의 경고를 표시합니다. 이는 의도된 동작으로, 보안을 프록시 설정에 위임하고 있음을 상기시키기 위한 것입니다.

감사 항목은 다음을 확인합니다:

- `trustedProxies` 구성 누락
- `userHeader` 구성 누락
- 비어 있는 `allowUsers` (인증된 모든 사용자 허용)

## 문제 해결

### "trusted_proxy_untrusted_source"

요청이 `gateway.trustedProxies`에 포함된 IP에서 오지 않았습니다. 확인 사항:

- 프록시 IP가 올바른가요? (Docker 컨테이너 IP는 변경될 수 있음)
- 프록시 앞에 로드 밸런서가 있나요?
- 실제 IP를 확인하려면 `docker inspect` 또는 `kubectl get pods -o wide`를 사용하세요

### "trusted_proxy_user_missing"

사용자 헤더가 비어 있거나 누락되었습니다. 확인 사항:

- 프록시가 인증 헤더를 전달하도록 구성되어 있나요?
- 헤더 이름이 올바른가요? (대소문자는 구분하지 않지만, 철자는 정확해야 합니다)
- 사용자가 실제로 프록시에서 인증되었나요?

### "trusted_proxy_missing_header_\*"

필수 헤더가 존재하지 않습니다. 확인 사항:

- 해당 특정 헤더에 대한 프록시 구성
- 전달 과정 어딘가에서 헤더가 제거되고 있지는 않은지 여부

### "trusted_proxy_user_not_allowed"

사용자는 인증되었지만 `allowUsers`에 포함되어 있지 않습니다. 사용자를 추가하거나 allowlist를 제거하세요.

### WebSocket이 여전히 실패하는 경우

프록시가 다음을 충족하는지 확인하세요:

- WebSocket 업그레이드를 지원 (`Upgrade: websocket`, `Connection: upgrade`)
- WebSocket 업그레이드 요청 시(HTTP뿐만 아니라) 인증 헤더를 전달
- WebSocket 연결에 대해 별도의 인증 경로가 없음

## 토큰 인증에서 마이그레이션

토큰 인증에서 trusted-proxy로 전환하는 경우:

1. 프록시가 사용자를 인증하고 헤더를 전달하도록 구성하세요
2. 프록시 설정을 독립적으로 테스트하세요 (헤더와 함께 curl 사용)
3. OpenClaw 구성에서 trusted-proxy 인증으로 업데이트하세요
4. Gateway를 재시작하세요
5. Control UI에서 WebSocket 연결을 테스트하세요
6. `openclaw security audit`를 실행하고 결과를 검토하세요

## 관련 항목

- [Security](/gateway/security) — 전체 보안 가이드
- [Configuration](/gateway/configuration) — 설정 레퍼런스
- [Remote Access](/gateway/remote) — 기타 원격 액세스 패턴
- [Tailscale](/gateway/tailscale) — tailnet 전용 액세스를 위한 더 간단한 대안

