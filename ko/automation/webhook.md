---
summary: "웨이크 및 격리된 에이전트 실행을 위한 Webhook 인그레스"
read_when:
  - Webhook 엔드포인트 추가 또는 변경 시
  - 외부 시스템을 OpenClaw 에 연결할 때
title: "Webhooks"
---

# Webhooks

Gateway(게이트웨이)는 외부 트리거를 위한 소규모 HTTP Webhook 엔드포인트를 노출할 수 있습니다.

## 활성화

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
  },
}
```

참고:

- `hooks.token` 는 `hooks.enabled=true` 인 경우 필요합니다.
- `hooks.path` 의 기본값은 `/hooks` 입니다.

## 인증

모든 요청에는 hook 토큰이 포함되어야 합니다. 헤더 사용을 권장합니다:

- `Authorization: Bearer <token>` (권장)
- `x-openclaw-token: <token>`
- 쿼리 문자열 토큰은 허용되지 않습니다 (`?token=...`은 `400` 반환).

## 엔드포인트

### `POST /hooks/wake`

페이로드:

```json
{ "text": "System line", "mode": "now" }
```

- `text` **필수** (string): 이벤트에 대한 설명 (예: "New email received").
- `mode` 선택 (`now` | `next-heartbeat`): 즉시 하트비트를 트리거할지 여부 (기본값 `now`) 또는 다음 주기적 확인을 기다릴지 여부.

효과:

- **메인** 세션에 시스템 이벤트를 큐에 추가합니다
- `mode=now` 인 경우, 즉시 하트비트를 트리거합니다

### `POST /hooks/agent`

페이로드:

```json
{
  "message": "Run this",
  "name": "Email",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **필수** (string): 에이전트가 처리할 프롬프트 또는 메시지.
- `name` 선택 (string): 훅의 사람이 읽을 수 있는 이름 (예: "GitHub"). 세션 요약에서 접두사로 사용됩니다.
- `agentId` 선택 사항 (string): 이 hook을 특정 에이전트로 라우팅합니다. 알 수 없는 ID는 기본 에이전트로 대체됩니다. 설정된 경우, 해당 hook은 확인된 에이전트의 워크스페이스 및 구성으로 실행됩니다.
- `sessionKey` 선택 (string): 에이전트 세션을 식별하는 데 사용되는 키. 기본적으로 `hooks.allowRequestSessionKey=true`가 아니면 이 필드는 허용되지 않습니다.
- `wakeMode` 선택 (`now` | `next-heartbeat`): 즉시 하트비트를 트리거할지 여부 (기본값 `now`) 또는 다음 주기적 확인을 기다릴지 여부.
- `deliver` 선택 (boolean): `true` 인 경우, 에이전트의 응답이 메시징 채널로 전송됩니다. 기본값은 `true` 입니다. 하트비트 확인만 포함된 응답은 자동으로 건너뜁니다.
- `channel` 선택 (string): 전송을 위한 메시징 채널. 다음 중 하나: `last`, `whatsapp`, `telegram`, `discord`, `slack`, `mattermost` (plugin), `signal`, `imessage`, `msteams`. 기본값은 `last` 입니다.
- `to` 선택 (string): 채널의 수신자 식별자 (예: WhatsApp/Signal 의 전화번호, Telegram 의 채팅 ID, Discord/Slack/Mattermost (plugin) 의 채널 ID, MS Teams 의 대화 ID). 기본값은 메인 세션의 마지막 수신자입니다.
- `model` 선택 (string): 모델 재정의 (예: `anthropic/claude-3-5-sonnet` 또는 별칭). 제한이 있는 경우 허용된 모델 목록에 포함되어야 합니다.
- `thinking` 선택 (string): 사고 수준 재정의 (예: `low`, `medium`, `high`).
- `timeoutSeconds` 선택 (number): 에이전트 실행의 최대 지속 시간 (초).

효과:

- **격리된** 에이전트 턴을 실행합니다 (자체 세션 키)
- 항상 **메인** 세션에 요약을 게시합니다
- `wakeMode=now` 인 경우, 즉시 하트비트를 트리거합니다

## 세션 키 정책 (호환성에 영향이 있는 변경)

`/hooks/agent` 페이로드의 `sessionKey` 재정의는 기본적으로 비활성화됩니다.

- 권장: 고정된 `hooks.defaultSessionKey`를 설정하고 요청 오버라이드는 비활성화하세요.
- 선택 사항: 필요한 경우에만 요청 오버라이드를 허용하고, 접두사를 제한하세요.

권장 설정:

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
  },
}
```

호환 설정(레거시 동작):

```json5
{
  hooks: {
    enabled: true,
    token: "${OPENCLAW_HOOKS_TOKEN}",
    allowRequestSessionKey: true,
    allowedSessionKeyPrefixes: ["hook:"], // strongly recommended
  },
}
```

### `POST /hooks/<name>` (매핑됨)

커스텀 훅 이름은 `hooks.mappings` 를 통해 해석됩니다 (구성 참조). 매핑은
임의의 페이로드를 `wake` 또는 `agent` 작업으로 변환할 수 있으며,
선택적으로 템플릿 또는 코드 변환을 사용할 수 있습니다.

매핑 옵션 (요약):

- `hooks.presets: ["gmail"]` 는 내장 Gmail 매핑을 활성화합니다.
- `hooks.mappings` 를 사용하면 구성에서 `match`, `action`, 그리고 템플릿을 정의할 수 있습니다.
- `hooks.transformsDir` + `transform.module` 는 커스텀 로직을 위한 JS/TS 모듈을 로드합니다.
  - `hooks.transformsDir`(설정한 경우)는 OpenClaw 설정 디렉터리(일반적으로 `~/.openclaw/hooks/transforms`) 아래의 transforms 루트 내에 있어야 합니다.
  - `transform.module`은 유효한 transforms 디렉터리 내에서 확인(resolve)되어야 합니다(경로 이탈/탈출 경로는 거부됨).
- `match.source` 를 사용하여 범용 인제스트 엔드포인트 (페이로드 기반 라우팅) 를 유지합니다.
- TS 변환은 TS 로더 (예: `bun` 또는 `tsx`) 또는 런타임에 사전 컴파일된 `.js` 가 필요합니다.
- `deliver: true` + `channel`/`to` 를 매핑에 설정하여 채팅 표면으로 응답을 라우팅합니다
  (`channel` 의 기본값은 `last` 이며 WhatsApp 으로 폴백됩니다).
- `agentId`는 훅을 특정 에이전트로 라우팅합니다. 알 수 없는 ID는 기본 에이전트로 대체됩니다.
- `hooks.allowedAgentIds`는 명시적인 `agentId` 라우팅을 제한합니다. 이를 생략하거나(`*` 포함) 모든 에이전트를 허용할 수 있습니다. `[]`로 설정하면 명시적인 `agentId` 라우팅을 거부합니다.
- `hooks.defaultSessionKey`는 명시적인 키가 제공되지 않은 경우 훅 에이전트 실행의 기본 세션을 설정합니다.
- `hooks.allowRequestSessionKey`는 `/hooks/agent` 페이로드에서 `sessionKey` 설정을 허용할지 여부를 제어합니다(기본값: `false`).
- `hooks.allowedSessionKeyPrefixes`는 요청 페이로드 및 매핑에서 명시적인 `sessionKey` 값의 접두사를 선택적으로 제한합니다.
- `allowUnsafeExternalContent: true` 는 해당 훅에 대해 외부 콘텐츠 안전 래퍼를 비활성화합니다
  (위험함; 신뢰된 내부 소스에 대해서만 사용하십시오).
- `openclaw webhooks gmail setup` 는 `openclaw webhooks gmail run` 를 위한 `hooks.gmail` 구성을 작성합니다.
  전체 Gmail 감시 흐름은 [Gmail Pub/Sub](/automation/gmail-pubsub) 를 참고하십시오.

## 응답

- `/hooks/wake` 에 대해 `200`
- `/hooks/agent` 에 대해 `202` (비동기 실행 시작됨)
- 인증 실패 시 `401`
- 동일한 클라이언트에서 반복적인 인증 실패 후 `429` 반환(`Retry-After` 확인)
- 유효하지 않은 페이로드 시 `400`
- 과도하게 큰 페이로드 시 `413`

## 예제

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"New email received","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","wakeMode":"next-heartbeat"}'
```

### 다른 모델 사용

해당 실행에 대해 모델을 재정의하려면 에이전트 페이로드 (또는 매핑) 에 `model` 를 추가하십시오:

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-openclaw-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"Summarize inbox","name":"Email","model":"openai/gpt-5.2-mini"}'
```

`agents.defaults.models` 를 강제하는 경우, 재정의 모델이 해당 목록에 포함되어 있는지 확인하십시오.

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"Hello","snippet":"Hi"}]}'
```

## 보안

- 훅 엔드포인트는 loopback, tailnet, 또는 신뢰된 리버스 프록시 뒤에 두십시오.
- 전용 훅 토큰을 사용하고, 게이트웨이 인증 토큰을 재사용하지 마십시오.
- 무차별 대입 공격을 지연시키기 위해 반복적인 인증 실패는 클라이언트 주소별로 속도 제한이 적용됩니다.
- 멀티 에이전트 라우팅을 사용하는 경우, 명시적인 `agentId` 선택을 제한하도록 `hooks.allowedAgentIds`를 설정하세요.
- 호출자가 세션을 선택해야 하는 경우가 아니라면 `hooks.allowRequestSessionKey=false`를 유지하세요.
- 요청 `sessionKey`를 활성화하는 경우, `hooks.allowedSessionKeyPrefixes`(예: `["hook:"]`)를 제한하세요.
- Webhook 로그에 민감한 원시 페이로드를 포함하지 마십시오.
- 훅 페이로드는 기본적으로 신뢰되지 않은 것으로 처리되며 안전 경계로 래핑됩니다.
  특정 훅에 대해 이를 비활성화해야 하는 경우, 해당 훅의 매핑에서 `allowUnsafeExternalContent: true` 를 설정하십시오
  (위험함).
