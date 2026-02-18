---
title: "서브 에이전트"
---

# 서브 에이전트

서브 에이전트는 기존 에이전트 실행에서 생성되는 백그라운드 에이전트 실행입니다. 각 서브 에이전트는 자체 세션(`agent:<agentId>:subagent:<uuid>`)에서 실행되며, 완료되면 요청자 채팅 채널로 결과를 **announce(알림)** 합니다.

## 슬래시 명령어

**현재 세션**의 서브 에이전트 실행을 확인하거나 제어하려면 `/subagents`를 사용하세요:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info`는 실행 메타데이터(상태, 타임스탬프, 세션 id, 트랜스크립트 경로, cleanup)를 표시합니다.

주요 목적:

- 메인 실행을 차단하지 않고 "리서치 / 장기 작업 / 느린 도구" 작업을 병렬화
- 기본적으로 서브 에이전트를 격리 유지 (세션 분리 + 선택적 샌드박싱)
- 도구 오남용 방지: 서브 에이전트는 기본적으로 세션 도구를 받지 않음
- 오케스트레이터 패턴을 위한 중첩 깊이 설정 지원

비용 참고: 각 서브 에이전트는 **자신만의** 컨텍스트와 토큰 사용량을 가집니다. 무겁거나 반복적인 작업의 경우, 서브 에이전트에는 더 저렴한 모델을 설정하고 메인 에이전트는 더 고품질 모델을 유지하세요.  
`agents.defaults.subagents.model` 또는 에이전트별 오버라이드로 설정할 수 있습니다.

## 도구

`sessions_spawn` 사용:

- 서브 에이전트 실행 시작 (`deliver: false`, 글로벌 레인: `subagent`)
- 이후 announce 단계를 실행하고 announce 응답을 요청자 채팅 채널에 게시
- 기본 모델: 호출자를 상속 (`agents.defaults.subagents.model` 또는 `agents.list[].subagents.model` 설정 시 해당 값 사용); 명시적 `sessions_spawn.model`이 항상 우선
- 기본 thinking: 호출자를 상속 (`agents.defaults.subagents.thinking` 또는 `agents.list[].subagents.thinking` 설정 시 해당 값 사용); 명시적 `sessions_spawn.thinking`이 항상 우선

도구 파라미터:

- `task` (필수)
- `label?` (선택)
- `agentId?` (선택; 허용된 경우 다른 에이전트 id 하위에서 생성)
- `model?` (선택; 서브 에이전트 모델 재정의; 유효하지 않은 값은 건너뛰며 기본 모델로 실행되고 도구 결과에 경고 표시)
- `thinking?` (선택; 서브 에이전트 실행의 thinking 수준 재정의)
- `runTimeoutSeconds?` (기본값 `0`; 설정 시 N초 후 서브 에이전트 실행 중단)
- `cleanup?` (`delete|keep`, 기본값 `keep`)

허용 목록:

- `agents.list[].subagents.allowAgents`: `agentId`로 지정 가능한 에이전트 id 목록 (`["*"]`는 모든 에이전트 허용). 기본값: 요청자 에이전트만 허용.

검색:

- `agents_list`를 사용하여 현재 `sessions_spawn`에 허용된 에이전트 id 확인.

자동 아카이브:

- 서브 에이전트 세션은 `agents.defaults.subagents.archiveAfterMinutes` 이후 자동 아카이브됩니다 (기본값: 60).
- 아카이브는 `sessions.delete`를 사용하며 트랜스크립트 파일을 `*.deleted.<timestamp>`로 이름 변경합니다 (같은 폴더).
- `cleanup: "delete"`는 announce 직후 즉시 아카이브합니다 (트랜스크립트는 이름 변경으로 유지).
- 자동 아카이브는 best-effort 방식이며, 게이트웨이 재시작 시 대기 중 타이머는 유실됩니다.
- `runTimeoutSeconds`는 자동 아카이브를 수행하지 않고 실행만 중단합니다. 세션은 자동 아카이브까지 유지됩니다.
- 자동 아카이브는 depth-1 및 depth-2 세션에 동일하게 적용됩니다.

## 중첩 서브 에이전트

기본적으로 서브 에이전트는 자신의 서브 에이전트를 생성할 수 없습니다 (`maxSpawnDepth: 1`).  
`maxSpawnDepth: 2`로 설정하면 한 단계 중첩을 허용하여 **오케스트레이터 패턴**(main → orchestrator sub-agent → worker sub-sub-agents)을 구현할 수 있습니다.

### 활성화 방법

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // 서브 에이전트가 자식 생성 허용 (기본값: 1)
        maxChildrenPerAgent: 5, // 에이전트 세션당 최대 활성 자식 수 (기본값: 5)
        maxConcurrent: 8, // 글로벌 동시 실행 레인 제한 (기본값: 8)
      },
    },
  },
}
```

### 깊이 레벨

| Depth | Session key shape                            | Role                                          | Can spawn?                   |
| ----- | -------------------------------------------- | --------------------------------------------- | ---------------------------- |
| 0     | `agent:<id>:main`                            | 메인 에이전트                                 | 항상 가능                     |
| 1     | `agent:<id>:subagent:<uuid>`                 | 서브 에이전트 (depth 2 허용 시 오케스트레이터) | `maxSpawnDepth >= 2`인 경우만 |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | 서브-서브 에이전트 (리프 워커)                | 불가                          |

### Announce 체인

결과는 체인을 따라 상위로 전달됩니다:

1. Depth-2 워커 완료 → 부모(depth-1 오케스트레이터)에게 announce
2. Depth-1 오케스트레이터가 announce 수신 후 결과 종합 → 완료 → 메인에 announce
3. 메인 에이전트가 announce 수신 후 사용자에게 전달

각 레벨은 자신의 직접 자식으로부터의 announce만 볼 수 있습니다.

### 깊이별 도구 정책

- **Depth 1 (오케스트레이터, `maxSpawnDepth >= 2`일 때)**: 자식 관리를 위해 `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` 사용 가능. 기타 세션/시스템 도구는 여전히 거부됨.
- **Depth 1 (리프, `maxSpawnDepth == 1`일 때)**: 세션 도구 없음 (현재 기본 동작).
- **Depth 2 (리프 워커)**: 세션 도구 없음 — depth 2에서는 `sessions_spawn`이 항상 거부됨. 추가 자식 생성 불가.

### 에이전트별 생성 제한

각 에이전트 세션(모든 depth)은 동시에 최대 `maxChildrenPerAgent`(기본값: 5)의 활성 자식을 가질 수 있습니다. 단일 오케스트레이터의 과도한 fan-out을 방지합니다.

### 연쇄 중지 (Cascade stop)

Depth-1 오케스트레이터를 중지하면 모든 depth-2 자식도 자동 중지됩니다:

- 메인 채팅에서 `/stop` → 모든 depth-1 에이전트 중지 및 해당 depth-2 자식까지 연쇄 중지
- `/subagents kill <id>` → 특정 서브 에이전트 및 해당 자식 중지
- `/subagents kill all` → 요청자의 모든 서브 에이전트 및 자식 중지

## 인증

서브 에이전트 인증은 세션 유형이 아니라 **에이전트 id** 기준으로 해결됩니다:

- 서브 에이전트 세션 키: `agent:<agentId>:subagent:<uuid>`
- 인증 스토어는 해당 에이전트의 `agentDir`에서 로드
- 메인 에이전트의 인증 프로필은 **폴백**으로 병합되며, 충돌 시 에이전트 프로필이 우선

참고: 병합은 가산적이므로 메인 프로필은 항상 폴백으로 사용 가능합니다. 에이전트별 완전 격리 인증은 아직 지원되지 않습니다.

## Announce

서브 에이전트는 announce 단계를 통해 결과를 보고합니다:

- announce 단계는 요청자 세션이 아닌 서브 에이전트 세션 내부에서 실행됩니다.
- 서브 에이전트가 정확히 `ANNOUNCE_SKIP`을 반환하면 아무것도 게시되지 않습니다.
- 그렇지 않으면 announce 응답은 후속 `agent` 호출(`deliver=true`)을 통해 요청자 채팅 채널에 게시됩니다.
- 가능한 경우 스레드/토픽 라우팅을 유지합니다 (Slack threads, Telegram topics, Matrix threads).
- announce 메시지는 안정적인 템플릿으로 정규화됩니다:
  - `Status:` 실행 결과(`success`, `error`, `timeout`, `unknown`) 기반
  - `Result:` announce 단계의 요약 내용 (없으면 `(not available)`)
  - `Notes:` 오류 세부 정보 및 기타 유용한 컨텍스트
- `Status`는 모델 출력이 아니라 런타임 결과 신호에서 파생됩니다.

Announce 페이로드에는 항상 통계 라인이 포함됩니다 (래핑된 경우에도):

- Runtime (예: `runtime 5m12s`)
- Token usage (input/output/total)
- 모델 가격이 설정된 경우 예상 비용 (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, 트랜스크립트 경로 (메인 에이전트가 `sessions_history`로 조회하거나 파일을 직접 확인 가능)

## 도구 정책 (서브 에이전트 도구)

기본적으로 서브 에이전트는 **세션 도구와 시스템 도구를 제외한 모든 도구**를 사용할 수 있습니다:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2`인 경우, depth-1 오케스트레이터 서브 에이전트는 자식 관리를 위해 추가로 `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`를 받습니다.

설정을 통한 오버라이드:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny가 항상 우선
        deny: ["gateway", "cron"],
        // allow가 설정되면 allow-only 모드 (deny가 여전히 우선)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## 동시성

서브 에이전트는 전용 in-process 큐 레인을 사용합니다:

- 레인 이름: `subagent`
- 동시 실행 수: `agents.defaults.subagents.maxConcurrent` (기본값 `8`)

## 중지

- 요청자 채팅에서 `/stop`을 보내면 요청자 세션이 중단되고, 해당 세션에서 생성된 모든 활성 서브 에이전트 실행이 중첩 자식까지 연쇄 중지됩니다.
- `/subagents kill <id>`는 특정 서브 에이전트를 중지하고 해당 자식까지 연쇄 중지합니다.

## 제한 사항

- 서브 에이전트 announce는 **best-effort**입니다. 게이트웨이가 재시작되면 대기 중인 "announce back" 작업은 유실됩니다.
- 서브 에이전트는 동일한 게이트웨이 프로세스 리소스를 공유합니다; `maxConcurrent`를 안전 장치로 사용하세요.
- `sessions_spawn`은 항상 non-blocking입니다: 즉시 `{ status: "accepted", runId, childSessionKey }`를 반환합니다.
- 서브 에이전트 컨텍스트에는 `AGENTS.md` + `TOOLS.md`만 주입됩니다 (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`는 포함되지 않음).
- 최대 중첩 깊이는 5입니다 (`maxSpawnDepth` 범위: 1–5). 대부분의 사용 사례에는 depth 2가 권장됩니다.
- `maxChildrenPerAgent`는 세션당 활성 자식 수를 제한합니다 (기본값: 5, 범위: 1–20).