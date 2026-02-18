---
title: "구성"
---

# 구성

OpenClaw 는 `~/.openclaw/openclaw.json` 에서 선택적 <Tooltip tip="JSON5는 주석과 후행 콤마를 지원합니다">**JSON5**</Tooltip> 구성을 읽습니다.

파일이 없으면 OpenClaw 는 안전한 기본값을 사용합니다. 다음과 같은 경우에 구성을 추가합니다:

- 채널을 연결하고 누가 봇에 메시지를 보낼 수 있는지 제어
- 모델, 도구, 샌드박스, 자동화(cron, hooks) 설정
- 세션, 미디어, 네트워킹 또는 UI 튜닝

사용 가능한 모든 필드는 [전체 레퍼런스](/gateway/configuration-reference)를 참고하세요.

<Tip>
**구성이 처음이신가요?** `openclaw onboard`로 대화형 설정을 시작하거나, 완전한 복사-붙여넣기 예제가 포함된 [Configuration Examples](/gateway/configuration-examples) 가이드를 확인하세요.
</Tip>

## Minimal config

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Editing config

<Tabs>
  <Tab title="대화형 마법사">
    ```bash
    openclaw onboard       # 전체 설정 마법사
    openclaw configure     # 구성 마법사
    ```
  </Tab>
  <Tab title="CLI (한 줄 명령)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  </Tab>
  <Tab title="Control UI">
    [http://127.0.0.1:18789](http://127.0.0.1:18789) 를 열고 **Config** 탭을 사용하세요.  
    Control UI는 구성 스키마로부터 폼을 렌더링하며, 필요 시 **Raw JSON** 편집기를 사용할 수 있습니다.
  </Tab>
  <Tab title="직접 편집">
    `~/.openclaw/openclaw.json` 파일을 직접 수정하세요. Gateway는 파일을 감시하며 변경 사항을 자동으로 적용합니다 (자세한 내용은 [hot reload](#config-hot-reload) 참고).
  </Tab>
</Tabs>

## Strict validation

<Warning>
OpenClaw 는 스키마와 완전히 일치하는 구성만 허용합니다. 알 수 없는 키, 잘못된 타입, 유효하지 않은 값이 있으면 Gateway 는 **시작을 거부**합니다. 루트 레벨에서 허용되는 유일한 예외는 `$schema` (string)이며, 이는 에디터가 JSON Schema 메타데이터를 연결할 수 있도록 합니다.
</Warning>

검증에 실패하면:

- Gateway 가 부팅되지 않습니다
- 진단 명령만 동작합니다 (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- 정확한 문제를 확인하려면 `openclaw doctor` 를 실행하세요
- `openclaw doctor --fix` (또는 `--yes`) 로 자동 복구를 적용하세요

## Common tasks

<AccordionGroup>
  <Accordion title="채널 설정 (WhatsApp, Telegram, Discord 등)">
    각 채널은 `channels.<provider>` 아래에 자체 구성 섹션을 가집니다. 설정 단계는 각 채널 문서를 참고하세요:

    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`

    모든 채널은 동일한 DM 정책 패턴을 공유합니다:

    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```

  </Accordion>

  <Accordion title="모델 선택 및 구성">
    기본 모델과 선택적 폴백을 설정합니다:

    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```

    - `agents.defaults.models` 는 모델 카탈로그를 정의하며 `/model` 의 허용 목록 역할을 합니다.
    - 모델 참조는 `provider/model` 형식을 사용합니다 (예: `anthropic/claude-opus-4-6`).
    - 채팅에서 모델을 전환하려면 [Models CLI](/concepts/models), 인증 회전 및 폴백 동작은 [Model Failover](/concepts/model-failover)를 참고하세요.
    - 커스텀/셀프 호스팅 프로바이더는 레퍼런스의 [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls)를 참고하세요.

  </Accordion>

  <Accordion title="누가 봇에 메시지를 보낼 수 있는지 제어">
    DM 접근은 채널별 `dmPolicy` 로 제어됩니다:

    - `"pairing"` (기본값): 알 수 없는 발신자는 1회용 페어링 코드를 받아 승인 필요
    - `"allowlist"`: `allowFrom` (또는 페어링 허용 저장소)에 있는 발신자만 허용
    - `"open"`: 모든 수신 DM 허용 (`allowFrom: ["*"]` 필요)
    - `"disabled"`: 모든 DM 무시

    그룹의 경우 `groupPolicy` + `groupAllowFrom` 또는 채널별 허용 목록을 사용하세요.

    자세한 내용은 [전체 레퍼런스](/gateway/configuration-reference#dm-and-group-access)를 참고하세요.

  </Accordion>

  <Accordion title="그룹 채팅 멘션 게이팅 설정">
    그룹 메시지는 기본적으로 **멘션 필요** 입니다. 에이전트별로 패턴을 구성할 수 있습니다:

    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

    - **메타데이터 멘션**: 플랫폼 네이티브 @-멘션 (WhatsApp 탭 멘션, Telegram @bot 등)
    - **텍스트 패턴**: `mentionPatterns` 의 정규식 패턴
    - 채널별 오버라이드 및 셀프 채팅 모드는 [전체 레퍼런스](/gateway/configuration-reference#group-chat-mention-gating)를 참고하세요.

  </Accordion>

  <Accordion title="세션 및 리셋 구성">
    세션은 대화 연속성과 격리를 제어합니다:

    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // 다중 사용자에 권장
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```

    - `dmScope`: `main` (공유) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - 범위, 아이덴티티 링크, 전송 정책은 [Session Management](/concepts/session)를 참고하세요.
    - 모든 필드는 [전체 레퍼런스](/gateway/configuration-reference#session)를 참고하세요.

  </Accordion>

  <Accordion title="샌드박스 활성화">
    에이전트 세션을 격리된 Docker 컨테이너에서 실행합니다:

    ```json5
    {
      agents: {
        defaults: {
          sandbox: {
            mode: "non-main",  // off | non-main | all
            scope: "agent",    // session | agent | shared
          },
        },
      },
    }
    ```

    먼저 이미지를 빌드하세요: `scripts/sandbox-setup.sh`

    전체 가이드는 [Sandboxing](/gateway/sandboxing), 모든 옵션은 [전체 레퍼런스](/gateway/configuration-reference#sandbox)를 참고하세요.

  </Accordion>

  <Accordion title="Heartbeat 설정 (주기적 체크인)">
    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    - `every`: 기간 문자열 (`30m`, `2h`). `0m` 으로 설정하면 비활성화.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - 전체 가이드는 [Heartbeat](/gateway/heartbeat)를 참고하세요.

  </Accordion>

  <Accordion title="Cron 작업 구성">
    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    기능 개요 및 CLI 예제는 [Cron jobs](/automation/cron-jobs)를 참고하세요.

  </Accordion>

  <Accordion title="웹훅 설정 (hooks)">
    Gateway 에 HTTP 웹훅 엔드포인트를 활성화합니다:

    ```json5
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        defaultSessionKey: "hook:ingress",
        allowRequestSessionKey: false,
        allowedSessionKeyPrefixes: ["hook:"],
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            agentId: "main",
            deliver: true,
          },
        ],
      },
    }
    ```

    모든 매핑 옵션과 Gmail 통합은 [전체 레퍼런스](/gateway/configuration-reference#hooks)를 참고하세요.

  </Accordion>

  <Accordion title="멀티 에이전트 라우팅 구성">
    분리된 워크스페이스와 세션을 가진 여러 에이전트를 실행합니다:

    ```json5
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
    }
    ```

    바인딩 규칙과 에이전트별 접근 프로필은 [Multi-Agent](/concepts/multi-agent) 및 [전체 레퍼런스](/gateway/configuration-reference#multi-agent-routing)를 참고하세요.

  </Accordion>

  <Accordion title="구성을 여러 파일로 분리 ($include)">
    `$include` 를 사용하여 큰 구성을 정리할 수 있습니다:

    ```json5
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
      agents: { $include: "./agents.json5" },
      broadcast: {
        $include: ["./clients/a.json5", "./clients/b.json5"],
      },
    }
    ```

    - **단일 파일**: 포함된 객체를 대체
    - **파일 배열**: 순서대로 deep-merge (나중 것이 우선)
    - **형제 키**: include 이후 병합 (포함 값 덮어씀)
    - **중첩 include**: 최대 10단계 지원
    - **상대 경로**: 포함 파일 기준으로 해석
    - **오류 처리**: 누락 파일, 파싱 오류, 순환 포함에 대해 명확한 오류 제공

  </Accordion>
</AccordionGroup>

## Config hot reload

Gateway 는 `~/.openclaw/openclaw.json` 파일을 감시하며 대부분의 설정 변경을 자동으로 적용합니다 — 일반적으로 수동 재시작이 필요하지 않습니다.

### Reload modes

| Mode                   | Behavior                                                                                |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **`hybrid`** (기본값) | 안전한 변경은 즉시 적용하고, 중요한 변경은 자동으로 재시작합니다.                      |
| **`hot`**              | 안전한 변경만 즉시 적용합니다. 재시작이 필요하면 경고만 표시합니다.                    |
| **`restart`**          | 모든 구성 변경 시 Gateway 를 재시작합니다.                                             |
| **`off`**              | 파일 감시를 비활성화합니다. 변경 사항은 다음 수동 재시작 시 적용됩니다.                |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### 무엇이 즉시 적용되고 무엇이 재시작이 필요한가

대부분의 필드는 다운타임 없이 즉시 적용됩니다. `hybrid` 모드에서는 재시작이 필요한 변경도 자동 처리됩니다.

| Category            | Fields                                                               | Restart needed? |
| ------------------- | -------------------------------------------------------------------- | --------------- |
| Channels            | `channels.*`, `web` (WhatsApp) — 모든 내장 및 확장 채널             | No              |
| Agent & models      | `agent`, `agents`, `models`, `routing`                               | No              |
| Automation          | `hooks`, `cron`, `agent.heartbeat`                                   | No              |
| Sessions & messages | `session`, `messages`                                                | No              |
| Tools & media       | `tools`, `browser`, `skills`, `audio`, `talk`                        | No              |
| UI & misc           | `ui`, `logging`, `identity`, `bindings`                              | No              |
| Gateway server      | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                 | **Yes**         |
| Infrastructure      | `discovery`, `canvasHost`, `plugins`                                 | **Yes**         |

<Note>
`gateway.reload` 와 `gateway.remote` 는 예외입니다 — 변경해도 재시작을 트리거하지 않습니다.
</Note>

## Config RPC (programmatic updates)

<AccordionGroup>
  <Accordion title="config.apply (전체 교체)">
    전체 구성을 검증 + 기록하고 Gateway 를 한 단계로 재시작합니다.

    <Warning>
    `config.apply` 는 **전체 구성**을 교체합니다. 일부만 변경하려면 `config.patch` 또는 `openclaw config set` 을 사용하세요.
    </Warning>

    Params:

    - `raw` (string) — 전체 구성을 위한 JSON5 페이로드
    - `baseHash` (optional) — `config.get` 의 구성 해시 (구성이 존재하는 경우 필요)
    - `sessionKey` (optional) — 재시작 후 깨우기 ping 을 위한 세션 키
    - `note` (optional) — 재시작 센티널 메모
    - `restartDelayMs` (optional) — 재시작 전 지연 (기본값 2000)

    ```bash
    openclaw gateway call config.get --params '{}'  # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```

  </Accordion>

  <Accordion title="config.patch (부분 업데이트)">
    기존 구성에 부분 업데이트를 병합합니다 (JSON merge patch 의미론):

    - 객체는 재귀적으로 병합
    - `null` 은 키 삭제
    - 배열은 교체

    Params:

    - `raw` (string) — 변경할 키만 포함한 JSON5
    - `baseHash` (required) — `config.get` 의 구성 해시
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` 와 동일

    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```

  </Accordion>
</AccordionGroup>

## Environment variables

OpenClaw 는 부모 프로세스의 환경 변수와 함께 다음을 읽습니다:

- 현재 작업 디렉터리의 `.env` (존재 시)
- `~/.openclaw/.env` (전역 폴백)

두 파일 모두 기존 환경 변수를 덮어쓰지 않습니다. 구성에서 인라인 환경 변수도 설정할 수 있습니다:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (선택 사항)">
  활성화되어 있고 필요한 키가 설정되지 않은 경우, OpenClaw 는 로그인 셸을 실행하여 누락된 키만 가져옵니다:

```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Env var equivalent: `OPENCLAW_LOAD_SHELL_ENV=1`
</Accordion>

<Accordion title="구성 값에서 환경 변수 치환">
  어떤 구성 문자열 값에서도 `${VAR_NAME}` 형식으로 환경 변수를 참조할 수 있습니다:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Rules:

- 대문자 이름만 매칭: `[A-Z_][A-Z0-9_]*`
- 누락/빈 변수는 로드 시 오류 발생
- 리터럴 출력은 `$${VAR}` 로 이스케이프
- `$include` 파일 내부에서도 동작
- 인라인 치환: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

전체 우선순위와 소스는 [Environment](/help/environment)를 참고하세요.

## Full reference

모든 필드에 대한 전체 문서는 **[Configuration Reference](/gateway/configuration-reference)** 를 참고하세요.

---

_관련 문서: [Configuration Examples](/gateway/configuration-examples) · [Configuration Reference](/gateway/configuration-reference) · [Doctor](/gateway/doctor)_
