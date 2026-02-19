---
summary: "Plan: CDP를 사용해 Playwright 큐에서 browser act:evaluate를 분리하고, 종단 간 데드라인과 더 안전한 ref 해석을 적용"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP 리팩터"
---

# Browser Evaluate CDP 리팩터 계획

## Context

`act:evaluate`는 사용자가 제공한 JavaScript를 페이지에서 실행합니다. 현재는 Playwright
(`page.evaluate` 또는 `locator.evaluate`)를 통해 실행됩니다. Playwright는 페이지별로 CDP 명령을 직렬화하므로,
중단되었거나 오래 실행되는 evaluate가 페이지 명령 큐를 차단하여 해당 탭의 이후 모든 작업이
"멈춘" 것처럼 보이게 할 수 있습니다.

PR #13498은 실용적인 안전장치(제한된 evaluate, abort 전파, 그리고 최선의
복구)를 추가합니다. 이 문서는 `act:evaluate`를 Playwright로부터 본질적으로
격리하여, evaluate가 중단되더라도 일반적인 Playwright 작업을 막지 못하도록 하는
더 큰 리팩터를 설명합니다.

## Goals

- `act:evaluate`는 동일한 탭에서 이후 브라우저 작업을 영구적으로 차단할 수 없어야 합니다.
- 타임아웃은 호출자가 예산을 신뢰할 수 있도록 종단 간 단일 진실 공급원(single source of truth)이어야 합니다.
- Abort와 타임아웃은 HTTP와 프로세스 내 디스패치 전반에서 동일한 방식으로 처리됩니다.
- evaluate를 위한 요소 타기팅을 지원하되, 모든 것을 Playwright 밖으로 전환하지는 않습니다.
- 기존 호출자와 페이로드에 대한 하위 호환성을 유지합니다.

## Non-goals

- 모든 브라우저 작업(click, type, wait 등)을 교체하는 것 CDP 구현으로.
- PR #13498에서 도입된 기존 안전장치를 제거하는 것(여전히 유용한 폴백으로 유지됨).
- 기존 `browser.evaluateEnabled` 게이트를 넘어서는 새로운 비안전 기능을 도입하는 것.
- evaluate를 위한 프로세스 격리(worker 프로세스/스레드)를 추가하는 것. 이 리팩터 이후에도 여전히 복구하기 어려운
  중단 상태가 발생한다면, 이는 후속 아이디어입니다.

## Current Architecture (왜 멈추는가)

상위 수준에서:

- 호출자는 `act:evaluate`를 브라우저 제어 서비스로 보냅니다.
- 라우트 핸들러는 Playwright를 호출하여 JavaScript를 실행합니다.
- Playwright는 페이지 명령을 직렬화하므로, 완료되지 않는 evaluate는 큐를 차단합니다.
- 큐가 멈추면 이후의 click/type/wait 작업이 해당 탭에서 멈춘 것처럼 보일 수 있습니다.

## 제안된 아키텍처

### 1. 데드라인 전파

단일 예산(budget) 개념을 도입하고 모든 것을 여기에서 파생합니다:

- 호출자가 `timeoutMs`(또는 미래 시점의 데드라인)를 설정합니다.
- 외부 요청 타임아웃, 라우트 핸들러 로직, 페이지 내부 실행 예산은
  모두 동일한 예산을 사용하며, 직렬화 오버헤드를 위해 필요한 경우 소량의 여유 시간을 둡니다.
- Abort는 `AbortSignal`로 전파되어 취소가 전 구간에서 일관되게 처리됩니다.

구현 방향:

- 작은 헬퍼(예: `createBudget({ timeoutMs, signal })`)를 추가하여 다음을 반환하도록 합니다:
  - `signal`: 연결된 AbortSignal
  - `deadlineAtMs`: 절대 데드라인
  - `remainingMs()`: 하위 작업에 사용할 수 있는 남은 예산
- 이 헬퍼를 다음 위치에서 사용합니다:
  - `src/browser/client-fetch.ts` (HTTP 및 프로세스 내 디스패치)
  - `src/node-host/runner.ts` (프록시 경로)
  - 브라우저 액션 구현부 (Playwright 및 CDP)

### 2. 분리된 Evaluate 엔진 (CDP 경로)

Playwright의 페이지별 명령
큐를 공유하지 않는 CDP 기반 evaluate 구현을 추가합니다. 핵심 속성은 evaluate 전송이 별도의 WebSocket 연결과
대상에 연결된 별도의 CDP 세션을 사용한다는 점입니다.

구현 방향:

- 새 모듈(예: `src/browser/cdp-evaluate.ts`)을 추가하여 다음을 수행합니다:
  - 설정된 CDP 엔드포인트(브라우저 수준 소켓)에 연결합니다.
  - `Target.attachToTarget({ targetId, flatten: true })`를 사용해 `sessionId`를 가져옵니다.
  - 다음 중 하나를 실행합니다:
    - 페이지 수준 evaluate의 경우 `Runtime.evaluate`, 또는
    - 요소 evaluate의 경우 `DOM.resolveNode`와 `Runtime.callFunctionOn` 조합.
  - 타임아웃 또는 abort 발생 시:
    - 해당 세션에 대해 `Runtime.terminateExecution`을 최선의 노력으로 전송합니다.
    - WebSocket을 닫고 명확한 오류를 반환합니다.

참고:

- 이는 여전히 페이지에서 JavaScript를 실행하므로, 종료 시 부작용이 발생할 수 있습니다. 장점은
  Playwright 큐를 막지 않으며, CDP 세션을 종료하여 전송 계층에서 취소할 수 있다는 점입니다.

### 3. Ref 스토리 (전체 재작성 없이 요소 타겟팅)

어려운 부분은 요소 타겟팅입니다. CDP는 DOM 핸들이나 `backendDOMNodeId`가 필요하지만,
현재 대부분의 브라우저 액션은 스냅샷의 ref를 기반으로 한 Playwright locator를 사용합니다.

권장 접근 방식: 기존 ref를 유지하되, 선택적으로 CDP에서 확인 가능한 id를 함께 연결합니다.

#### 3.1 저장된 Ref 정보 확장

저장된 role ref 메타데이터를 선택적으로 CDP id를 포함할 수 있도록 확장합니다:

- 현재: `{ role, name, nth }`
- 제안: `{ role, name, nth, backendDOMNodeId?: number }`

이렇게 하면 기존의 모든 Playwright 기반 동작을 그대로 유지하면서, `backendDOMNodeId`를 사용할 수 있는 경우 CDP evaluate가 동일한 `ref` 값을 사용할 수 있습니다.

#### 3.2 스냅샷 시점에 backendDOMNodeId 채우기

role 스냅샷을 생성할 때:

1. 현재와 동일하게 기존 role ref 맵(role, name, nth)을 생성합니다.
2. CDP를 통해 AX 트리(`Accessibility.getFullAXTree`)를 가져오고, 동일한 중복 처리 규칙을 사용하여 `(role, name, nth) -> backendDOMNodeId`의 병렬 맵을 계산합니다.
3. 해당 id를 현재 탭에 저장된 ref 정보에 다시 병합합니다.

ref에 대한 매핑에 실패하면 `backendDOMNodeId`를 undefined로 둡니다. 이렇게 하면 이 기능은 최선의 노력(best-effort) 방식으로 안전하게 배포할 수 있습니다.

#### 3.3 Ref를 사용한 Evaluate 동작

`act:evaluate`에서:

- `ref`가 존재하고 `backendDOMNodeId`를 가지고 있으면 CDP를 통해 element evaluate를 실행합니다.
- `ref`는 존재하지만 `backendDOMNodeId`가 없으면 Playwright 경로로 폴백합니다(안전 장치 포함).

선택적 이스케이프 해치:

- 고급 호출자(및 디버깅 목적)를 위해 `backendDOMNodeId`를 직접 받을 수 있도록 요청 형태를 확장하되, `ref`를 기본 인터페이스로 유지합니다.

### 4. 최후 수단 복구 경로 유지

CDP evaluate를 사용하더라도 탭이나 연결이 멈출 수 있는 다른 원인들이 존재합니다. 다음과 같은 경우를 대비해 기존 복구 메커니즘(실행 종료 + Playwright 연결 해제)을 최후 수단으로 유지합니다:

- 레거시 호출자
- CDP attach가 차단된 환경
- 예상치 못한 Playwright 엣지 케이스

## 구현 계획 (단일 반복)

### 산출물

- Playwright의 페이지별 명령 큐 외부에서 실행되는 CDP 기반 evaluate 엔진.
- 호출자와 핸들러 전반에서 일관되게 사용되는 단일 end-to-end timeout/abort 예산.
- element evaluate를 위해 선택적으로 `backendDOMNodeId`를 포함할 수 있는 ref 메타데이터.
- `act:evaluate`는 가능하면 CDP 엔진을 우선 사용하고, 불가능한 경우 Playwright로 폴백합니다.
- evaluate가 멈추더라도 이후 동작이 중단되지 않음을 증명하는 테스트.
- 실패 및 폴백이 가시적으로 드러나는 로그/메트릭.

### 구현 체크리스트

1. `timeoutMs` + 상위 `AbortSignal`을 다음과 연결하는 공통 "budget" 헬퍼 추가:
   - 단일 `AbortSignal`
   - 절대 마감 시간(deadline)
   - 하위 작업을 위한 `remainingMs()` 헬퍼
2. `timeoutMs`가 어디서나 동일한 의미를 갖도록 모든 호출 경로를 해당 헬퍼를 사용하도록 업데이트:
   - `src/browser/client-fetch.ts` (HTTP 및 인프로세스 디스패치)
   - `src/node-host/runner.ts` (node proxy 경로)
   - `/act`를 호출하는 CLI 래퍼 (`browser evaluate`에 `--timeout-ms` 추가)
3. `src/browser/cdp-evaluate.ts` 구현:
   - 브라우저 레벨 CDP 소켓에 연결
   - `sessionId`를 얻기 위해 `Target.attachToTarget` 호출
   - 페이지 evaluate를 위해 `Runtime.evaluate` 실행
   - 요소 evaluate를 위해 `DOM.resolveNode` + `Runtime.callFunctionOn` 실행
   - 타임아웃/abort 시: 가능한 한 `Runtime.terminateExecution`을 호출한 후 소켓 종료
4. 저장된 role ref 메타데이터에 선택적으로 `backendDOMNodeId` 포함하도록 확장:
   - Playwright 동작을 위한 기존 `{ role, name, nth }` 동작 유지
   - CDP 요소 타겟팅을 위해 `backendDOMNodeId?: number` 추가
5. 스냅샷 생성 시 `backendDOMNodeId` 채우기 (best-effort):
   - CDP를 통해 AX 트리 가져오기 (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` 매핑을 계산하여 저장된 ref 맵에 병합
   - 매핑이 모호하거나 없으면 id를 undefined로 둠
6. `act:evaluate` 라우팅 업데이트:
   - `ref`가 없으면: 항상 CDP evaluate 사용
   - `ref`가 `backendDOMNodeId`로 해석되면: CDP 요소 evaluate 사용
   - 그 외의 경우: Playwright evaluate로 폴백 (여전히 제한 시간 내에서 abort 가능)
7. 기존 "last resort" 복구 경로는 기본 경로가 아닌 폴백으로 유지합니다.
8. 테스트 추가:
   - 멈춘 evaluate가 예산 시간 내에 타임아웃되고 다음 click/type이 성공함
   - abort가 evaluate를 취소(클라이언트 연결 종료 또는 타임아웃)하고 이후 동작을 차단하지 않음
   - 매핑 실패 시 Playwright로 깔끔하게 폴백
9. 관측 가능성 추가:
   - evaluate 소요 시간 및 타임아웃 카운터
   - terminateExecution 사용량
   - 폴백 비율 (CDP -> Playwright) 및 사유

### 승인 기준

- 의도적으로 멈춘 `act:evaluate`가 호출자 예산 시간 내에 반환되며 이후 동작을 위해
  탭이 멈추지 않는다.
- `timeoutMs`는 CLI, agent tool, node proxy, in-process 호출 전반에서 일관되게 동작한다.
- `ref`가 `backendDOMNodeId`로 매핑될 수 있으면 요소 evaluate는 CDP를 사용하고, 그렇지 않은 경우에도
  폴백 경로는 여전히 제한 시간 내에서 복구 가능하다.

## 테스트 계획

- 단위 테스트:
  - role ref와 AX 트리 노드 간 `(role, name, nth)` 매칭 로직.
  - 예산 헬퍼 동작 (headroom, 남은 시간 계산).
- 통합 테스트:
  - CDP evaluate 타임아웃이 예산 시간 내에 반환되며 다음 동작을 차단하지 않음.
  - Abort가 evaluate를 취소하고 가능한 범위 내에서 termination을 트리거함.
- 계약 테스트:
  - `BrowserActRequest`와 `BrowserActResponse`가 계속 호환되도록 하세요.

## 위험 요소 및 대응 방안

- 매핑은 완벽하지 않습니다:
  - 대응: 가능한 한 최선의 매핑을 수행하고, Playwright evaluate로 폴백하며, 디버그 도구를 추가합니다.
- `Runtime.terminateExecution`에는 부작용이 있습니다:
  - 대응: 타임아웃/중단 시에만 사용하고, 해당 동작을 오류 메시지에 문서화합니다.
- 추가 오버헤드:
  - 대응: 스냅샷이 요청된 경우에만 AX 트리를 가져오고, 타깃별로 캐시하며,
    CDP 세션을 짧게 유지합니다.
- 확장 프로그램 릴레이 제한 사항:
  - 대응: 페이지별 소켓을 사용할 수 없는 경우 브라우저 수준의 attach API를 사용하고,
    현재 Playwright 경로를 폴백으로 유지합니다.

## 열린 질문

- 새 엔진을 `playwright`, `cdp`, 또는 `auto`로 설정 가능하게 해야 할까요?
- 고급 사용자를 위해 새로운 "nodeRef" 형식을 노출해야 할까요, 아니면 `ref`만 유지할까요?
- 프레임 스냅샷과 셀렉터 범위 스냅샷은 AX 매핑에 어떻게 반영되어야 할까요?
