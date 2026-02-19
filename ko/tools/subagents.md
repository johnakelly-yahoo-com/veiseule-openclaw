---
summary: "서브 에이전트: 요청자 채널로 결과를 알리는 격리된 에이전트 실행을 스폰합니다"
read_when:
  - 에이전트를 통한 백그라운드/병렬 작업이 필요할 때
  - sessions_spawn 또는 서브 에이전트 도구 정책을 변경할 때
title: "서브 에이전트"
---

# 서브 에이전트

서브 에이전트를 사용하면 메인 대화를 차단하지 않고 백그라운드 작업을 실행할 수 있습니다. 서브 에이전트를 생성하면 자체적으로 격리된 세션에서 실행되어 작업을 수행한 후, 완료되면 결과를 채팅에 알립니다.

## 명령

현재 세션의 하위 에이전트 실행을 검사하고 제어하려면 `/subagents` 슬래시 명령을 사용하세요:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info`는 실행 메타데이터(상태, 타임스탬프, 세션 id, 트랜스크립트 경로, 정리 여부)를 표시합니다.

주요 목표:

- "research / long task / slow tool" 작업을 병렬화하여 메인 실행을 차단하지 않습니다.
- 기본적으로 하위 에이전트를 격리 상태로 유지합니다(세션 분리 + 선택적 샌드박싱).
- 도구 표면을 오용하기 어렵게 유지합니다: 하위 에이전트는 기본적으로 세션 도구에 접근할 수 없습니다.
- 오케스트레이터 패턴을 위해 구성 가능한 중첩 깊이를 지원합니다.

각 서브 에이전트는 **자신만의** 컨텍스트와 토큰 사용량을 가집니다. 무겁거나 반복적인 작업의 경우 하위 에이전트에는 더 저렴한 모델을 설정하고, 메인 에이전트는 더 높은 품질의 모델을 유지하세요.
에이전트별 설정: `agents.list[].subagents.model`

## #> [limit] [tools]\`

`sessions_spawn` 호출의 명시적 `model` 파라미터

- 하위 에이전트 실행을 시작합니다 (`deliver: false`, 전역 레인: `subagent`)
- 그다음 announce 단계를 실행하고 announce 응답을 요청자 채팅 채널에 게시합니다.
- 전역 기본값: `agents.defaults.subagents.model`
- 에이전트별 설정: `agents.list[].subagents.thinking`

도구 매개변수:

- `task`
- `label`
- 다른 에이전트 ID 하위에서 생성 (허용되어야 함)
- 모델: 대상 에이전트의 기본 모델 선택 (`subagents.model`이 설정되지 않은 경우)
- 사고(thinking): 서브 에이전트에 대한 오버라이드 없음 (`subagents.thinking`이 설정되지 않은 경우)
- `runTimeoutSeconds?` (기본값 `0`; 설정 시 N초 후 하위 에이전트 실행이 중단됨)
- `"delete"` \\

허용 목록:

- `agents.list[].subagents.allowAgents`: `agentId`를 통해 대상으로 지정할 수 있는 에이전트 id 목록(`"[*]"`가 아니라 `["*"]`를 사용하여 모든 에이전트 허용). 기본값: 요청자 에이전트만.

검색:

- `sessions_spawn`에 대해 현재 허용된 에이전트 ID를 확인하려면 `agents_list` 도구를 사용하세요.
</Tip>

자동 아카이브

- 서브 에이전트 세션은 설정 가능한 기간 이후 자동으로 아카이브됩니다:
- 아카이브 시 대화 기록의 이름이 \`\*.deleted.
  (같은 폴더).
- `"delete"`는 알림 후 즉시 보관 처리
- 자동 아카이브 타이머는 최선의 노력(best-effort) 방식이며, 게이트웨이가 재시작되면 대기 중인 타이머는 유실됩니다.
- `runTimeoutSeconds`는 세션을 자동으로 아카이브하지 **않습니다**. 세션은 일반적인 아카이브 타이머가 실행될 때까지 유지됩니다.
- 자동 보관은 depth-1 및 depth-2 세션에 동일하게 적용됩니다.

## 서브 에이전트 중지

기본적으로 하위 에이전트는 자신의 하위 에이전트를 생성할 수 없습니다(`maxSpawnDepth: 1`). `maxSpawnDepth: 2`로 설정하면 한 단계의 중첩을 활성화할 수 있으며, 이는 **orchestrator pattern**(main → orchestrator 하위 에이전트 → worker 하위-하위 에이전트)을 허용합니다.

### 방법

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

### 깊이 수준

| 깊이                        | 세션 키 형태                                                                                                                                                                                                  | 역할                                                       | 생성 가능?                     |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- | -------------------------- |
| 3. | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;model: "minimax/MiniMax-M2.1",&#xA;},&#xA;},&#xA;},&#xA;}           | 에이전트별 오버라이드                                              | 항상                         |
| 1. | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;thinking: "low",&#xA;},&#xA;},&#xA;},&#xA;}                                         | 하위 에이전트(depth 2가 허용된 경우 orchestrator) | `maxSpawnDepth >= 2`인 경우에만 |
| 2. | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;archiveAfterMinutes: 120, // default: 60&#xA;},&#xA;},&#xA;},&#xA;} | 하위 에이전트 관리 (`/subagents`)             | 절대 불가                      |

### 문자열

결과는 체인을 따라 상위로 전달됩니다:

1. Depth-2 worker가 완료 → 상위(depth-1 orchestrator)에게 announce
2. Depth-1 orchestrator가 announce를 수신하고 결과를 종합한 뒤 완료 → main에 announce
3. 메인 에이전트가 announce를 수신하여 사용자에게 전달합니다.

각 레벨은 자신의 직계 하위에서 올라온 announce만 볼 수 있습니다.

### 도구 정책

- **Depth 1 (orchestrator, when `maxSpawnDepth >= 2`)**: `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`를 받아 자신의 하위 에이전트를 관리할 수 있습니다. 기타 session/system 도구는 계속 거부됩니다.
- **Depth 1 (leaf, when `maxSpawnDepth == 1`)**: session 도구 없음 (현재 기본 동작).
- **Depth 2 (leaf worker)**: session 도구 없음 — `sessions_spawn`는 depth 2에서 항상 거부됩니다. 추가 하위 에이전트를 생성할 수 없습니다.

### 에이전트별 spawn 제한

각 에이전트 세션(모든 depth에서)은 동시에 최대 `maxChildrenPerAgent`(기본값: 5)개의 활성 하위 에이전트만 가질 수 있습니다. 이를 통해 단일 orchestrator에서 무분별하게 fan-out이 발생하는 것을 방지합니다.

### 연쇄 중지

depth-1 orchestrator를 중지하면 해당 depth-2 하위 에이전트도 자동으로 모두 중지됩니다:

- 메인 채팅에서 `/stop`을 실행하면 모든 depth-1 에이전트가 중지되며, 그들의 depth-2 하위 에이전트까지 연쇄적으로 중지됩니다.
- `/subagents stop <id>`
- `/subagents kill all`은 요청자의 모든 하위 에이전트를 중지하며, 연쇄적으로 적용됩니다.

## 인증

서브 에이전트 인증은 세션 유형이 아니라 **에이전트 id** 로 해결됩니다:

- 하위 에이전트 세션 키는 `agent:<agentId>:subagent:<uuid>`입니다.
- 인증 스토어는 대상 에이전트의 `agentDir`에서 로드됩니다
- 메인 에이전트의 인증 프로필은 **폴백**으로 병합됩니다(충돌 시 에이전트 프로필이 우선합니다).

병합은 가산적입니다 — 메인 프로필은 항상 폴백으로 사용 가능합니다. <Note>
현재 서브 에이전트별로 완전히 격리된 인증은 지원되지 않습니다.
</Note>

## Announce 상태

하위 에이전트는 announce 단계를 통해 결과를 보고합니다:

- announce 단계는 요청자 세션이 아닌 하위 에이전트 세션 내부에서 실행됩니다.
- 하위 에이전트가 정확히 `ANNOUNCE_SKIP`로 응답하면 아무 것도 게시되지 않습니다.
- 그 외의 경우 announce 응답은 후속 `agent` 호출(`deliver=true`)을 통해 요청자 채팅 채널에 게시됩니다.
- 알림 응답은 가능한 경우 스레드/토픽 라우팅을 유지합니다(Slack 스레드, Telegram 토픽, Matrix 스레드).
- announce 메시지는 안정적인 템플릿으로 정규화됩니다:
  - `Status:` 실행 결과(`success`, `error`, `timeout`, 또는 `unknown`)에서 파생됩니다.
  - `Result:` announce 단계의 요약 내용(없을 경우 `(not available)`).
  - `Notes:` 오류 세부 정보 및 기타 유용한 컨텍스트.
- announce 메시지에는 모델 출력이 아닌 실행 결과에서 파생된 상태가 포함됩니다:

각 announce에는 다음이 포함된 통계 라인이 있습니다:

- 실행 시간 (예: `runtime 5m12s`)
- 토큰 사용량(입력/출력/총합)
- 예상 비용 (`models.providers.*.models[].cost`를 통해 모델 가격이 구성된 경우)
- `sessionKey`, `sessionId`, 및 transcript 경로 (메인 에이전트가 `sessions_history`로 기록을 조회하거나 디스크의 파일을 직접 확인할 수 있도록)

## 서브 에이전트 도구 커스터마이징

기본적으로 서브 에이전트는 **안전하지 않거나 백그라운드 작업에 불필요한** 일부 도구를 제외한 **모든 도구**를 사용할 수 있습니다:

- `sessions_spawn` 호출의 명시적 `thinking` 파라미터
- `runTimeoutSeconds`
- `sessions_send`
- `sessions_spawn` 도구

`maxSpawnDepth >= 2`인 경우, depth-1 orchestrator 하위 에이전트는 자신의 하위 에이전트를 관리할 수 있도록 `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history`를 추가로 받습니다.

config를 통해 재정의:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // 설정된 경우 deny가 여전히 우선
      },
    },
  },
}
```

## 동시성

하위 에이전트는 전용 인-프로세스 큐 레인을 사용합니다:

- ```
  :subagent:
  ```
- 전역 기본값: `agents.defaults.subagents.thinking`

## 중지

- 에이전트는 내부적으로 `sessions_spawn` 도구를 호출합니다. 서브 에이전트가 작업을 완료하면, 그 결과를 다시 이 채팅에 알려줍니다.
- ```
  `) 전용 `subagent` 큐 레인에서.
  ```

## 제한 사항

- 하위 에이전트 announce는 **best-effort** 방식입니다. gateway가 재시작되면 대기 중인 "announce back" 작업은 손실됩니다.
- 하위 에이전트는 동일한 gateway 프로세스 리소스를 공유하므로, `maxConcurrent`를 안전 장치로 간주하세요.
- 메인 에이전트는 작업 설명과 함께 `sessions_spawn`를 호출합니다. 이 호출은 **비차단(non-blocking)** 방식입니다 — 메인 에이전트는 즉시 `{ status: "accepted", runId, childSessionKey }`를 반환받습니다.
- 하위 에이전트 컨텍스트에는 `AGENTS.md` + `TOOLS.md`만 주입됩니다 (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`는 제외).
- 최대 중첩 깊이는 5입니다 (`maxSpawnDepth` 범위: 1–5). 대부분의 사용 사례에는 depth 2를 권장합니다.
- `maxChildrenPerAgent`는 세션당 활성 하위 에이전트 수를 제한합니다 (기본값: 5, 범위: 1–20).

