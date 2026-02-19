---
summary: "Gateway 서비스의 런북, 수명 주기 및 운영 가이드"
read_when:
  - Gateway 프로세스를 실행하거나 디버깅할 때
title: "Gateway 런북"
---

# Gateway 서비스 런북

Gateway 서비스의 day-1 시작 및 day-2 운영에는 이 페이지를 사용하세요.

<CardGroup cols={2}>
  <Card title="Deep troubleshooting" icon="siren" href="/gateway/troubleshooting">
    정확한 명령 단계와 로그 시그니처를 포함한 증상 우선 진단.
  
</Card>
  <Card title="Configuration" icon="sliders" href="/gateway/configuration">
    작업 중심 설정 가이드 + 전체 구성 참조.
  
</Card>
</CardGroup>

## 5분 로컬 시작

<Steps>
  <Step title="Start the Gateway">

```bash
openclaw gateway --port 18789
# for full debug/trace logs in stdio:
openclaw gateway --port 18789 --verbose
# if the port is busy, terminate listeners then start:
openclaw gateway --force
# dev loop (auto-reload on TS changes):
pnpm gateway:watch
```

  
</Step>

  <Step title="Verify service health">

```bash
openclaw --profile main gateway install
openclaw --profile rescue gateway install
```

정상 기준: `Runtime: running` 및 `RPC probe: ok`.

  
</Step>

  <Step title="Validate channel readiness">

```bash
openclaw channels status --probe
```

  
</Step>
</Steps>

<Note>
Gateway 구성 다시 로드는 활성 구성 파일 경로(프로필/상태 기본값에서 확인되거나, 설정된 경우 `OPENCLAW_CONFIG_PATH`)를 감시합니다.
기본 모드: `gateway.reload.mode="hybrid"` (안전한 변경은 핫 적용, 중요 변경은 재시작).
</Note>

## 런타임 모델

- 라우팅, 제어 플레인 및 채널 연결을 위한 항상 실행되는 단일 프로세스.
- 단일 포트 멀티플렉스입니다.
  - WebSocket 제어/RPC
  - OpenResponses (HTTP): [`/v1/responses`](/gateway/openresponses-http-api).
  - 제어 UI 및 훅
- 기본 바인드 모드: `loopback`.
- Gateway 인증은 기본적으로 필요합니다: `gateway.auth.token` (또는 `OPENCLAW_GATEWAY_TOKEN`) 또는 `gateway.auth.password`를 설정하십시오.

### 포트 및 바인드 우선순위

| 설정         | 결정 순서                                                                                                       |
| ---------- | ----------------------------------------------------------------------------------------------------------- |
| Gateway 포트 | 포트 우선순위: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > 기본값 `18789`. |
| 바인드 모드     | CLI/override → `gateway.bind` → `loopback`                                                                  |

### 핫 리로드 모드

| `gateway.reload.mode="off"`로 비활성화할 수 있습니다. | 동작                      |
| ---------------------------------------------------------- | ----------------------- |
| `off`                                                      | 구성 다시 로드 없음             |
| `hot`                                                      | 핫 적용이 안전한 변경만 적용        |
| `restart`                                                  | 다시 로드가 필요한 변경 시 재시작     |
| `hybrid` (default)                      | 안전한 경우 핫 적용, 필요한 경우 재시작 |

## 운영자 명령 세트

```bash
openclaw gateway status
openclaw gateway install
openclaw gateway stop
openclaw gateway restart
openclaw logs --follow
```

## 원격 접근

선호: Tailscale/VPN.
대안: SSH 터널.

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

이후 클라이언트는 터널을 통해 `ws://127.0.0.1:18789`에 연결합니다.

<Warning>
원격 사용은 동일한 SSH/Tailscale 터널을 사용하며, gateway 토큰이 구성된 경우 클라이언트는 `connect` 동안 이를 포함합니다.
</Warning>

참고: [Remote Gateway](/gateway/remote), [Authentication](/gateway/authentication), [Tailscale](/gateway/tailscale).

## 감독 및 서비스 수명 주기

프로덕션 수준의 안정성을 위해 supervised 실행을 사용하세요.

<Tabs>
  <Tab title="macOS (launchd)">

```bash
재시작하려면 `openclaw gateway restart` (또는 `launchctl kickstart -k gui/$UID/bot.molt.gateway`)를 사용하십시오.
```

OpenClaw.app은 Node 기반 gateway 릴레이를 번들링하고, 사용자별 LaunchAgent를
`bot.molt.gateway` (또는 `bot.molt.<profile> `; 레거시 `com.openclaw.*` 레이블도 정상적으로 언로드됨) 라벨로 설치할 수 있습니다.`(이름이 지정된 프로필).`openclaw doctor\`는 서비스 구성 드리프트를 점검하고 복구합니다.

  
</Tab>

  <Tab title="Linux (systemd user)">

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

linger 활성화(로그아웃/유휴 후에도 사용자 서비스 유지에 필요):

```bash
sudo loginctl enable-linger youruser
```

  
</Tab>

  <Tab title="Linux (system service)">

다중 사용자/항상 실행되는 호스트의 경우 시스템 유닛을 사용하세요.

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

  
</Tab>
</Tabs>

## 다중 Gateway (동일 호스트)

기본적으로 호스트당 하나의 Gateway를 가정합니다.
여러 프로필을 실행하는 경우 포트/상태를 격리하고 올바른 인스턴스를 대상으로 하십시오.

인스턴스별 체크리스트:

- 고유한 `gateway.port`
- 고유한 `OPENCLAW_CONFIG_PATH`
- 고유한 `OPENCLAW_STATE_DIR`
- 고유한 `agents.defaults.workspace`

예시:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

[Multiple gateways](/gateway/multiple-gateways)를 참고하십시오.

### Dev 프로필 (`--dev`)

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
# then target the dev instance:
openclaw --dev status
openclaw --dev health
```

기본값에는 격리된 상태/구성과 기본 gateway 포트 `19001`이 포함됩니다.

## 프로토콜 (운영자 관점)

- 클라이언트는 재연결해야 합니다.
- Gateway는 `hello-ok` 스냅샷(`presence`, `health`, `stateVersion`, `uptimeMs`, limits/policy)을 반환합니다.
- 요청: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
- 일반 이벤트: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`.

Agent 실행은 두 단계로 이루어집니다:

1. 즉시 수락 확인 응답(`status:"accepted"`)
2. `agent` 응답은 2단계입니다: 먼저 `res` ack `{runId,status:"accepted"}`, 이후 실행이 끝나면 최종 `res` `{runId,status:"ok"|"error",summary}`; 스트리밍 출력은 `event:"agent"`로 도착합니다.

전체 문서: [Gateway protocol](/gateway/protocol) 및 [Bridge protocol (legacy)](/gateway/bridge-protocol).

## 운영 점검

### 활성 상태 확인

- WS를 열고 `connect`를 전송합니다.
- 스냅샷이 포함된 `hello-ok` 응답을 예상하세요.

### 준비 상태

```bash
`openclaw gateway health|status` — Gateway WS를 통해 상태/헬스를 요청합니다.
```

### 갭 복구

이벤트는 재생되지 않습니다. 시퀀스에 갭이 발생하면 계속하기 전에 상태(`health`, `system-presence`)를 새로 고치세요.

## 일반적인 실패 시그니처

| 시그니처                                                           | 가능한 문제                       |
| -------------------------------------------------------------- | ---------------------------- |
| `refusing to bind gateway ... without auth`                    | 토큰/비밀번호 없이 non-loopback에 바인딩 |
| `another gateway instance is already listening` / `EADDRINUSE` | 포트 충돌                        |
| `Gateway start blocked: set gateway.mode=local`                | 구성이 원격 모드로 설정됨               |
| `unauthorized` during connect                                  | 클라이언트와 Gateway 간 인증 불일치      |

전체 진단 단계는 [Gateway Troubleshooting](/gateway/troubleshooting)을 참고하세요.

## 안전 보장

- Gateway가 다운되면 전송은 즉시 실패합니다.
- 유효하지 않거나 connect가 아닌 첫 프레임은 거부되고 연결이 종료됩니다.
- 정상 종료: 닫기 전에 `shutdown` 이벤트를 발행합니다.

---

관련 항목:

- [Troubleshooting](/gateway/troubleshooting)
- [Background Process](/gateway/background-process)
- [Configuration](/gateway/configuration)
- [Health](/gateway/health)
- [Doctor](/gateway/doctor)
- [Authentication](/gateway/authentication)
