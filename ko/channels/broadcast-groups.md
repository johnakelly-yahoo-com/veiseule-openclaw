---
status: experimental
title: "브로드캐스트 그룹"
---

# 브로드캐스트 그룹

**상태:** 실험적  
**버전:** 2026.1.9 에 추가됨

## 개요

브로드캐스트 그룹을 사용하면 여러 에이전트가 동일한 메시지를 동시에 처리하고 응답할 수 있습니다. 이를 통해 하나의 전화번호만으로 단일 WhatsApp 그룹 또는 다이렉트 메시지(DM)에서 함께 작업하는 전문화된 에이전트 팀을 구성할 수 있습니다.

현재 범위: **WhatsApp 전용** (웹 채널).

브로드캐스트 그룹은 채널 허용 목록과 그룹 활성화 규칙 이후에 평가됩니다. WhatsApp 그룹의 경우, 이는 OpenClaw 가 일반적으로 응답하는 시점(예: 그룹 설정에 따라 멘션 시)에 브로드캐스트가 발생함을 의미합니다.

## 사용 사례

### 1. 전문화된 에이전트 팀

원자적이고 집중된 책임을 가진 여러 에이전트를 배포합니다:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

각 에이전트는 동일한 메시지를 처리하고 자신의 전문 관점을 제공합니다.

### 2. 다국어 지원

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. 품질 보증 워크플로

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. 작업 자동화

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## 구성

### 기본 설정

최상위에 `broadcast` 섹션을 추가합니다(`bindings` 옆). 키는 WhatsApp 피어 ID 입니다:

- 그룹 채팅: 그룹 JID (예: `120363403215116621@g.us`)
- DM: E.164 전화번호 (예: `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**결과:** OpenClaw 가 이 채팅에서 응답해야 할 때, 세 개의 에이전트를 모두 실행합니다.

### 처리 전략

에이전트가 메시지를 처리하는 방식을 제어합니다:

#### 병렬 (기본값)

모든 에이전트가 동시에 처리합니다:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### 순차

에이전트가 순서대로 처리합니다(이전 에이전트가 완료될 때까지 대기):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### 전체 예제

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## 동작 방식

### 메시지 흐름

1. **수신 메시지**가 WhatsApp 그룹에 도착
2. **브로드캐스트 확인**: 시스템이 피어 ID 가 `broadcast` 에 있는지 확인
3. **브로드캐스트 목록에 있는 경우**:
   - 나열된 모든 에이전트가 메시지를 처리
   - 각 에이전트는 고유한 세션 키와 분리된 컨텍스트를 가짐
   - 에이전트는 병렬(기본값) 또는 순차로 처리
4. **브로드캐스트 목록에 없는 경우**:
   - 일반 라우팅 적용(첫 번째로 일치하는 바인딩)

참고: 브로드캐스트 그룹은 채널 허용 목록이나 그룹 활성화 규칙(멘션/명령 등)을 우회하지 않습니다. 메시지가 처리 대상이 될 때 _어떤 에이전트가 실행되는지_만 변경합니다.

### 세션 격리

브로드캐스트 그룹의 각 에이전트는 다음을 완전히 분리하여 유지합니다:

- **세션 키** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **대화 기록** (에이전트는 다른 에이전트의 메시지를 보지 않음)
- **워크스페이스** (구성된 경우 분리된 샌드박스)
- **도구 접근** (서로 다른 허용/차단 목록)
- **메모리/컨텍스트** (분리된 IDENTITY.md, SOUL.md 등)
- **그룹 컨텍스트 버퍼** (컨텍스트에 사용되는 최근 그룹 메시지)는 피어별로 공유되므로, 트리거 시 모든 브로드캐스트 에이전트가 동일한 컨텍스트를 봅니다

이를 통해 각 에이전트는 다음을 가질 수 있습니다:

- 서로 다른 성격
- 서로 다른 도구 접근(예: 읽기 전용 vs 읽기-쓰기)
- 서로 다른 모델(예: opus vs sonnet)
- 서로 다른 설치된 Skills

### 예시: 격리된 세션

그룹 `120363403215116621@g.us` 에 에이전트 `["alfred", "baerbel"]` 가 있는 경우:

**Alfred 의 컨텍스트:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel 의 컨텍스트:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## 모범 사례

### 1. 에이전트의 역할을 명확히 유지

각 에이전트를 단일하고 명확한 책임으로 설계합니다:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **좋음:** 각 에이전트가 하나의 역할만 수행  
❌ **나쁨:** 하나의 범용 "dev-helper" 에이전트

### 2. 설명적인 이름 사용

각 에이전트가 무엇을 하는지 명확히 합니다:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. 서로 다른 도구 접근 구성

에이전트에 필요한 도구만 제공합니다:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. 성능 모니터링

에이전트가 많을 경우 다음을 고려하십시오:

- 속도를 위해 `"strategy": "parallel"` (기본값) 사용
- 브로드캐스트 그룹을 5–10 개 에이전트로 제한
- 단순한 에이전트에는 더 빠른 모델 사용

### 5. 장애를 우아하게 처리

에이전트는 독립적으로 실패합니다. 한 에이전트의 오류가 다른 에이전트를 차단하지 않습니다:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## 호환성

### 프로바이더

브로드캐스트 그룹은 현재 다음과 함께 작동합니다:

- ✅ WhatsApp (구현됨)
- 🚧 Telegram (계획됨)
- 🚧 Discord (계획됨)
- 🚧 Slack (계획됨)

### 라우팅

브로드캐스트 그룹은 기존 라우팅과 함께 작동합니다:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: alfred 만 응답 (일반 라우팅)
- `GROUP_B`: agent1 **그리고** agent2 가 응답 (브로드캐스트)

**우선순위:** `broadcast` 이 `bindings` 보다 우선합니다.

## 문제 해결

### 에이전트가 응답하지 않는 경우

**확인 사항:**

1. `agents.list` 에 에이전트 ID 가 존재하는지
2. 피어 ID 형식이 올바른지 (예: `120363403215116621@g.us`)
3. 에이전트가 차단 목록에 있지 않은지

**디버그:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### 하나의 에이전트만 응답하는 경우

**원인:** 피어 ID 가 `bindings` 에는 있지만 `broadcast` 에는 없을 수 있습니다.

**해결:** 브로드캐스트 설정에 추가하거나 바인딩에서 제거합니다.

### 성능 문제

**여러 에이전트로 느린 경우:**

- 그룹당 에이전트 수를 줄이기
- 가벼운 모델 사용(opus 대신 sonnet)
- 샌드박스 시작 시간 확인

## 예제

### 예제 1: 코드 리뷰 팀

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**사용자 전송:** 코드 스니펫  
**응답:**

- code-formatter: "들여쓰기를 수정하고 타입 힌트를 추가했습니다"
- security-scanner: "⚠️ 12 번째 줄에 SQL 인젝션 취약점이 있습니다"
- test-coverage: "커버리지가 45% 이며, 오류 케이스에 대한 테스트가 누락되었습니다"
- docs-checker: "함수 `process_data` 에 대한 docstring 이 누락되었습니다"

### 예제 2: 다국어 지원

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## API 참조

### 설정 스키마

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### 필드

- `strategy` (선택 사항): 에이전트를 처리하는 방법
  - `"parallel"` (기본값): 모든 에이전트를 동시에 처리
  - `"sequential"`: 배열 순서대로 에이전트 처리
- `[peerId]`: WhatsApp 그룹 JID, E.164 번호 또는 기타 피어 ID
  - 값: 메시지를 처리해야 하는 에이전트 ID 배열

## 제한 사항

1. **최대 에이전트 수:** 하드 제한은 없지만 10 개 이상은 느릴 수 있습니다
2. **공유 컨텍스트:** 에이전트는 서로의 응답을 보지 않습니다(설계상)
3. **메시지 순서:** 병렬 응답은 어떤 순서로든 도착할 수 있습니다
4. **요율 제한:** 모든 에이전트가 WhatsApp 요율 제한에 포함됩니다

## 향후 개선 사항

계획된 기능:

- [ ] 공유 컨텍스트 모드(에이전트가 서로의 응답을 확인)
- [ ] 에이전트 조정(에이전트 간 신호 전달)
- [ ] 동적 에이전트 선택(메시지 내용에 따라 에이전트 선택)
- [ ] 에이전트 우선순위(일부 에이전트가 먼저 응답)

## 참고 항목

- [멀티 에이전트 구성](/tools/multi-agent-sandbox-tools)
- [라우팅 구성](/channels/channel-routing)
- [세션 관리](/concepts/sessions)
