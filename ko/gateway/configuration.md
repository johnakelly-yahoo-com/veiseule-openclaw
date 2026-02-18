---
title: "구성"
---

# 구성 🔧

OpenClaw 는 `~/.openclaw/openclaw.json` 에서 선택적 **JSON5** 구성을 읽습니다 (주석 + 후행 콤마 허용).

파일이 없으면 OpenClaw 는 비교적 안전한 기본값 (임베디드 Pi 에이전트 + 발신자별 세션 + 워크스페이스 `~/.openclaw/workspace`) 을 사용합니다. 일반적으로 다음과 같은 경우에만 구성이 필요합니다.

- 봇을 트리거할 수 있는 사용자를 제한 (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom` 등)
- 그룹 허용 목록 + 멘션 동작 제어 (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- 메시지 접두사 사용자 지정 (`messages`)
- 에이전트의 워크스페이스 설정 (`agents.defaults.workspace` 또는 `agents.list[].workspace`)
- 임베디드 에이전트 기본값 (`agents.defaults`) 및 세션 동작 (`session`) 튜닝
- 에이전트별 아이덴티티 설정 (`agents.list[].identity`)

> **구성이 처음이신가요?** 자세한 설명이 포함된 전체 예제는 [Configuration Examples](/gateway/configuration-examples) 가이드를 참고하십시오!

## 엄격한 구성 검증

OpenClaw 는 스키마와 완전히 일치하는 구성만 허용합니다.
알 수 없는 키, 잘못된 타입, 유효하지 않은 값이 있으면 안전을 위해 Gateway(게이트웨이)가 **시작을 거부**합니다.

검증에 실패하면:

- Gateway 가 부팅되지 않습니다.
- 진단 명령만 허용됩니다 (예: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- 정확한 문제를 확인하려면 `openclaw doctor` 를 실행하십시오.
- 마이그레이션/복구를 적용하려면 `openclaw doctor --fix` (또는 `--yes`) 를 실행하십시오.

Doctor 는 명시적으로 `--fix`/`--yes` 에 동의하지 않는 한 변경 사항을 절대 기록하지 않습니다.

## 스키마 + UI 힌트

Gateway 는 UI 편집기를 위해 `config.schema` 를 통해 구성의 JSON Schema 표현을 노출합니다.
Control UI 는 이 스키마로부터 폼을 렌더링하며, 탈출구로 **Raw JSON** 편집기를 제공합니다.

채널 플러그인과 확장은 자체 구성에 대한 스키마 + UI 힌트를 등록할 수 있으므로,  
하드코딩된 폼 없이도 앱 전반에서 스키마 기반 설정을 유지할 수 있습니다.

힌트 (라벨, 그룹화, 민감 필드) 는 스키마와 함께 제공되어,  
클라이언트가 구성 지식을 하드코딩하지 않고도 더 나은 폼을 렌더링할 수 있습니다.

## 적용 + 재시작 (RPC)

`config.apply` 을 사용하여 전체 구성을 검증 + 기록하고 Gateway 를 한 단계로 재시작하십시오.
재시작 센티널을 기록하고, Gateway 가 다시 올라온 후 마지막 활성 세션을 핑합니다.

경고: `config.apply` 는 **전체 구성**을 교체합니다. 일부 키만 변경하려면  
`config.patch` 또는 `openclaw config set` 를 사용하십시오. `~/.openclaw/openclaw.json` 의 백업을 유지하십시오.

매개변수:

- `raw` (string) — 전체 구성을 위한 JSON5 페이로드
- `baseHash` (선택) — `config.get` 의 구성 해시 (구성이 이미 존재할 때 필요)
- `sessionKey` (선택) — 깨우기 핑을 위한 마지막 활성 세션 키
- `note` (선택) — 재시작 센티널에 포함할 메모
- `restartDelayMs` (선택) — 재시작 전 지연 (기본값 2000)

예제 (`gateway call` 사용):

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.apply --params '{
  "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## 부분 업데이트 (RPC)

`config.patch` 을 사용하면 관련 없는 키를 덮어쓰지 않고 기존 구성에 부분 업데이트를 병합할 수 있습니다. JSON merge patch 의미를 적용합니다.

- 객체는 재귀적으로 병합됩니다.
- `null` 는 키를 삭제합니다.
- 배열은 교체됩니다. `config.apply` 와 마찬가지로, 검증 → 기록 → 재시작 센티널 저장 → Gateway 재시작 예약을 수행합니다  
  (`sessionKey` 이 제공되면 선택적으로 깨움).

매개변수:

- `raw` (string) — 변경할 키만 포함한 JSON5 페이로드
- `baseHash` (필수) — `config.get` 의 구성 해시
- `sessionKey` (선택) — 깨우기 핑을 위한 마지막 활성 세션 키
- `note` (선택) — 재시작 센티널에 포함할 메모
- `restartDelayMs` (선택) — 재시작 전 지연 (기본값 2000)

예제:

```bash
openclaw gateway call config.get --params '{}' # capture payload.hash
openclaw gateway call config.patch --params '{
  "raw": "{\\n  channels: { telegram: { groups: { \\"*\\": { requireMention: false } } } }\\n}\\n",
  "baseHash": "<hash-from-config.get>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123",
  "restartDelayMs": 1000
}'
```

## 최소 구성 (권장 시작점)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

다음으로 기본 이미지를 한 번 빌드하십시오:

```bash
scripts/sandbox-setup.sh
```

## 셀프 채팅 모드 (그룹 제어에 권장)

그룹에서 WhatsApp @-멘션에 봇이 응답하지 않도록 하려면 (특정 텍스트 트리거에만 응답):

```json5
{
  agents: {
    defaults: { workspace: "~/.openclaw/workspace" },
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
      },
    ],
  },
  channels: {
    whatsapp: {
      // Allowlist is DMs only; including your own number enables self-chat mode.
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## 구성 포함 (`$include`)

`$include` 지시어를 사용하여 구성을 여러 파일로 분할할 수 있습니다. 이는 다음에 유용합니다.

- 대규모 구성 정리 (예: 클라이언트별 에이전트 정의)
- 환경 간 공통 설정 공유
- 민감한 구성 분리 보관

### 기본 사용법

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },

  // Include a single file (replaces the key's value)
  agents: { $include: "./agents.json5" },

  // Include multiple files (deep-merged in order)
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

```json5
// ~/.openclaw/agents.json5
{
  defaults: { sandbox: { mode: "all", scope: "session" } },
  list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
}
```

### 병합 동작

- **단일 파일**: `$include` 를 포함한 객체를 교체합니다.
- **파일 배열**: 순서대로 깊은 병합을 수행합니다 (뒤의 파일이 앞의 파일을 덮어씀).
- **형제 키 포함**: 포함 이후에 형제 키가 병합됩니다 (포함된 값 덮어씀).
- **형제 키 + 배열/프리미티브**: 지원되지 않습니다 (포함된 콘텐츠는 객체여야 함).

```json5
// Sibling keys override included values
{
  $include: "./base.json5", // { a: 1, b: 2 }
  b: 99, // Result: { a: 1, b: 99 }
}
```

### 중첩 포함

포함된 파일은 자체적으로 `$include` 지시어를 포함할 수 있습니다 (최대 10 단계 깊이).

```json5
// clients/mueller.json5
{
  agents: { $include: "./mueller/agents.json5" },
  broadcast: { $include: "./mueller/broadcast.json5" },
}
```

### 경로 해석

- **상대 경로**: 포함하는 파일을 기준으로 해석됩니다.
- **절대 경로**: 그대로 사용됩니다.
- **상위 디렉토리**: `../` 참조는 예상대로 동작합니다.

```json5
{ "$include": "./sub/config.json5" }      // relative
{ "$include": "/etc/openclaw/base.json5" } // absolute
{ "$include": "../shared/common.json5" }   // parent dir
```

### 오류 처리

- **파일 누락**: 해석된 경로와 함께 명확한 오류를 표시합니다.
- **파싱 오류**: 어떤 포함 파일에서 실패했는지 표시합니다.
- **순환 포함**: 포함 체인과 함께 감지 및 보고됩니다.

### 예제: 다중 클라이언트 법적 설정

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789, auth: { token: "secret" } },

  // Common agent defaults
  agents: {
    defaults: {
      sandbox: { mode: "all", scope: "session" },
    },
    // Merge agent lists from all clients
    list: { $include: ["./clients/mueller/agents.json5", "./clients/schmidt/agents.json5"] },
  },

  // Merge broadcast configs
  broadcast: {
    $include: ["./clients/mueller/broadcast.json5", "./clients/schmidt/broadcast.json5"],
  },

  channels: { whatsapp: { groupPolicy: "allowlist" } },
}
```

```json5
// ~/.openclaw/clients/mueller/agents.json5
[
  { id: "mueller-transcribe", workspace: "~/clients/mueller/transcribe" },
  { id: "mueller-docs", workspace: "~/clients/mueller/docs" },
]
```

```json5
// ~/.openclaw/clients/mueller/broadcast.json5
{
  "120363403215116621@g.us": ["mueller-transcribe", "mueller-docs"],
}
```

## 공통 옵션

### 환경 변수 + `.env`

OpenClaw 는 부모 프로세스 (셸, launchd/systemd, CI 등) 로부터 환경 변수를 읽습니다.

추가로 다음을 로드합니다.

- 현재 작업 디렉토리의 `.env` (존재 시)
- `~/.openclaw/.env` 의 전역 대체 `.env` (일명 `$OPENCLAW_STATE_DIR/.env`)

두 `.env` 파일 모두 기존 환경 변수를 덮어쓰지 않습니다.

구성 내에 인라인 환경 변수를 제공할 수도 있습니다. 이 값은  
프로세스 환경에 키가 없는 경우에만 적용됩니다 (동일한 비덮어쓰기 규칙).

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

전체 우선순위와 소스는 [/environment](/help/environment) 를 참고하십시오.

### `env.shellEnv` (선택)

편의 기능 옵트인: 활성화되어 있고 예상되는 키가 아직 설정되지 않았다면,  
OpenClaw 는 로그인 셸을 실행하여 누락된 예상 키만 가져옵니다 (절대 덮어쓰지 않음).
이는 사실상 셸 프로필을 소싱합니다.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

환경 변수에 해당하는 값:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

### 구성에서 환경 변수 치환

어떤 구성 문자열 값에서도 `${VAR_NAME}` 문법을 사용하여  
환경 변수를 직접 참조할 수 있습니다. 변수는 검증 전에 구성 로드 시점에 치환됩니다.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

**규칙:**

- 대문자 환경 변수 이름만 매칭됩니다: `[A-Z_][A-Z0-9_]*`
- 누락되었거나 비어 있는 환경 변수는 구성 로드 시 오류를 발생시킵니다.
- 리터럴 `${VAR}` 을 출력하려면 `$${VAR}` 로 이스케이프하십시오.
- `$include` 와 함께 동작합니다 (포함된 파일에도 치환 적용).

**인라인 치환:**

```json5
{
  models: {
    providers: {
      custom: {
        baseUrl: "${CUSTOM_API_BASE}/v1", // → "https://api.example.com/v1"
      },
    },
  },
}
```

### 인증 저장소 (OAuth + API 키)

OpenClaw 는 **에이전트별** 인증 프로필 (OAuth + API 키) 을 다음 위치에 저장합니다.

- `<agentDir>/auth-profiles.json` (기본값: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`)

추가 참고: [/concepts/oauth](/concepts/oauth)

레거시 OAuth 가져오기:

- `~/.openclaw/credentials/oauth.json` (또는 `$OPENCLAW_STATE_DIR/credentials/oauth.json`)

임베디드 Pi 에이전트는 다음 위치에 런타임 캐시를 유지합니다.

- `<agentDir>/auth.json` (자동 관리됨; 수동 편집 금지)

레거시 에이전트 디렉토리 (다중 에이전트 이전):

- `~/.openclaw/agent/*` (`openclaw doctor` 가 `~/.openclaw/agents/<defaultAgentId>/agent/*` 로 마이그레이션)

오버라이드:

- OAuth 디렉토리 (레거시 가져오기 전용): `OPENCLAW_OAUTH_DIR`
- 에이전트 디렉토리 (기본 에이전트 루트 재정의): `OPENCLAW_AGENT_DIR` (권장), `PI_CODING_AGENT_DIR` (레거시)

첫 사용 시 OpenClaw 는 `oauth.json` 항목을 `auth-profiles.json` 로 가져옵니다.

### `auth`

인증 프로필을 위한 선택적 메타데이터입니다. 이는 비밀 정보를 저장하지 않으며,  
프로필 ID 를 프로바이더 + 모드 (및 선택적 이메일) 에 매핑하고  
페일오버에 사용되는 프로바이더 회전 순서를 정의합니다.

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

### `agents.list[].identity`

기본값과 UX 에 사용되는 선택적 에이전트별 아이덴티티입니다. 이는 macOS 온보딩 어시스턴트가 기록합니다.

설정된 경우, OpenClaw 는 (명시적으로 설정하지 않았을 때만) 기본값을 파생합니다.

- 활성 에이전트의 `identity.emoji` 에서 `messages.ackReaction` (👀 로 폴백)
- 에이전트의 `identity.name`/`identity.emoji` 에서 `agents.list[].groupChat.mentionPatterns`  
  (Telegram/Slack/Discord/Google Chat/iMessage/WhatsApp 전반에서 “@Samantha” 가 동작)
- `identity.avatar` 는 워크스페이스 상대 이미지 경로 또는 원격 URL/data URL 을 허용합니다. 로컬 파일은 에이전트 워크스페이스 내부에 있어야 합니다.

`identity.avatar` 는 다음을 허용합니다.

- 워크스페이스 상대 경로 (에이전트 워크스페이스 내부에 있어야 함)
- `http(s)` URL
- `data:` URI

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

### `wizard`

CLI 마법사 (`onboard`, `configure`, `doctor`) 가 기록하는 메타데이터입니다.

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

### `logging`

- 기본 로그 파일: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`
- 안정적인 경로가 필요하면 `logging.file` 를 `/tmp/openclaw/openclaw.log` 로 설정하십시오.
- 콘솔 출력은 다음으로 별도 조정할 수 있습니다.
  - `logging.consoleLevel` (기본값 `info`, `--verbose` 시 `debug` 로 상승)
  - `logging.consoleStyle` (`pretty` | `compact` | `json`)
- 도구 요약은 비밀 정보 유출을 방지하기 위해 마스킹할 수 있습니다.
  - `logging.redactSensitive` (`off` | `tools`, 기본값: `tools`)
  - `logging.redactPatterns` (정규식 문자열 배열; 기본값 재정의)

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty",
    redactSensitive: "tools",
    redactPatterns: [
      // Example: override defaults with your own rules.
      "\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1",
      "/\\bsk-[A-Za-z0-9_-]{8,}\\b/gi",
    ],
  },
}
```

### `channels.whatsapp.dmPolicy`

WhatsApp 다이렉트 메시지 (DM) 처리 방식을 제어합니다.

- `"pairing"` (기본값): 알 수 없는 발신자는 페어링 코드를 받으며, 소유자가 승인해야 합니다.
- `"allowlist"`: `channels.whatsapp.allowFrom` (또는 페어링 허용 저장소) 에 있는 발신자만 허용
- `"open"`: 모든 수신 DM 허용 (**`channels.whatsapp.allowFrom` 에 `"*"` 포함 필요**)
- `"disabled"`: 모든 수신 DM 무시

페어링 코드는 1 시간 후 만료됩니다. 봇은 새 요청이 생성될 때만 페어링 코드를 전송합니다. 대기 중인 DM 페어링 요청은 기본적으로 **채널당 3 개**로 제한됩니다.

페어링 승인:

- `openclaw pairing list whatsapp`
- `openclaw pairing approve whatsapp <code>`

### `channels.whatsapp.allowFrom`

WhatsApp 자동 응답을 트리거할 수 있는 E.164 전화번호 허용 목록 (**DM 전용**).
비어 있고 `channels.whatsapp.dmPolicy="pairing"` 인 경우, 알 수 없는 발신자는 페어링 코드를 받습니다.
그룹의 경우 `channels.whatsapp.groupPolicy` + `channels.whatsapp.groupAllowFrom` 를 사용하십시오.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000, // optional outbound chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      mediaMaxMb: 50, // optional inbound media cap (MB)
    },
  },
}
```

### `channels.whatsapp.sendReadReceipts`

수신 WhatsApp 메시지를 읽음 처리 (파란 체크) 할지 여부를 제어합니다. 기본값: `true`.

셀프 채팅 모드에서는 활성화되어 있어도 항상 읽음 확인을 건너뜁니다.

계정별 재정의: `channels.whatsapp.accounts.<id>.sendReadReceipts`.

```json5
{
  channels: {
    whatsapp: { sendReadReceipts: false },
  },
}
```

### `channels.whatsapp.accounts` (다중 계정)

하나의 게이트웨이에서 여러 WhatsApp 계정을 실행합니다.

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {}, // optional; keeps the default id stable
        personal: {},
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

참고:

- 발신 명령은 `default` 계정이 있으면 이를 기본으로 사용하고, 없으면 정렬된 첫 번째 계정 id 를 사용합니다.
- 레거시 단일 계정 Baileys 인증 디렉토리는 `openclaw doctor` 가 `whatsapp/default` 로 마이그레이션합니다.

### `channels.telegram.accounts` / `channels.discord.accounts` / `channels.googlechat.accounts` / `channels.slack.accounts` / `channels.mattermost.accounts` / `channels.signal.accounts` / `channels.imessage.accounts`

채널당 여러 계정을 실행합니다 (각 계정은 자체 `accountId` 및 선택적 `name` 을 가짐).

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

참고:

- `default` 는 `accountId` 이 생략되었을 때 사용됩니다 (CLI + 라우팅).
- 환경 변수 토큰은 **기본 계정**에만 적용됩니다.
- 기본 채널 설정 (그룹 정책, 멘션 게이팅 등) 은 계정별로 재정의되지 않는 한 모든 계정에 적용됩니다. 계정별로 오버라이드되지 않는 한 모든 계정에 적용됩니다.
- 각 계정을 서로 다른 agents.defaults 로 라우팅하려면 `bindings[].match.accountId` 를 사용하십시오.

### 그룹 채팅 멘션 게이팅 (`agents.list[].groupChat` + `messages.groupChat`)

그룹 메시지는 기본적으로 **멘션 필요** (메타데이터 멘션 또는 정규식 패턴) 입니다. WhatsApp, Telegram, Discord, Google Chat, iMessage 그룹 채팅에 적용됩니다.

**멘션 유형:**

- **메타데이터 멘션**: 플랫폼 네이티브 @-멘션 (예: WhatsApp 탭 멘션). WhatsApp 셀프 채팅 모드에서는 무시됩니다 (`channels.whatsapp.allowFrom` 참고).
- **텍스트 패턴**: `agents.list[].groupChat.mentionPatterns` 에 정의된 정규식 패턴. 셀프 채팅 모드와 관계없이 항상 검사됩니다.
- 멘션 게이팅은 멘션 감지가 가능한 경우에만 적용됩니다 (네이티브 멘션 또는 최소 하나의 `mentionPattern`).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` 는 그룹 히스토리 컨텍스트의 전역 기본값을 설정합니다. 채널은 `channels.<channel>.historyLimit` (또는 다중 계정의 경우 `channels.<channel>.accounts.*.historyLimit`) 으로 재정의할 수 있습니다. 히스토리 래핑을 비활성화하려면 `0` 을 설정하십시오.

#### DM 히스토리 제한

DM 대화는 에이전트가 관리하는 세션 기반 히스토리를 사용합니다. DM 세션당 유지되는 사용자 턴 수를 제한할 수 있습니다.

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30, // limit DM sessions to 30 user turns
      dms: {
        "123456789": { historyLimit: 50 }, // per-user override (user ID)
      },
    },
  },
}
```

해결 순서:

1. DM 별 재정의: `channels.<provider>.dms[userId].historyLimit`
2. 프로바이더 기본값: `channels.<provider>.dmHistoryLimit`
3. 제한 없음 (모든 히스토리 유지)

지원 프로바이더: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

에이전트별 재정의 (설정 시 우선 적용, `[]` 보다 우선):

```json5
{
  agents: {
    list: [
      { id: "work", groupChat: { mentionPatterns: ["@workbot", "\\+15555550123"] } },
      { id: "personal", groupChat: { mentionPatterns: ["@homebot", "\\+15555550999"] } },
    ],
  },
}
```

멘션 게이팅 기본값은 채널별로 존재합니다 (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.imessage.groups`, `channels.discord.guilds`). `*.groups` 이 설정되면 그룹 허용 목록 역할도 수행하며, 모든 그룹을 허용하려면 `"*"` 를 포함하십시오.

네이티브 @-멘션을 무시하고 **특정 텍스트 트리거에만** 응답하려면:

```json5
{
  channels: {
    whatsapp: {
      // Include your own number to enable self-chat mode (ignore native @-mentions).
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          // Only these text patterns will trigger responses
          mentionPatterns: ["reisponde", "@openclaw"],
        },
      },
    ],
  },
}
```

### 그룹 정책 (채널별)

`channels.*.groupPolicy` 를 사용하여 그룹/룸 메시지를 아예 수락할지 여부를 제어합니다.

```json5
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["tg:123456789", "@alice"],
    },
    signal: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: {
          channels: { help: { allow: true } },
        },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
  },
}
```

참고:

- `"open"`: 그룹이 허용 목록을 우회합니다. 멘션 게이팅은 여전히 적용됩니다.
- `"disabled"`: 모든 그룹/룸 메시지를 차단합니다.
- `"allowlist"`: 구성된 허용 목록과 일치하는 그룹/룸만 허용합니다.
- `channels.defaults.groupPolicy` 는 프로바이더의 `groupPolicy` 이 설정되지 않았을 때 기본값을 설정합니다.
- WhatsApp/Telegram/Signal/iMessage/Microsoft Teams 는 `groupAllowFrom` 을 사용합니다 (폴백: 명시적 `allowFrom`).
- Discord/Slack 은 채널 허용 목록 (`channels.discord.guilds.*.channels`, `channels.slack.channels`) 을 사용합니다.
- 그룹 DM (Discord/Slack) 은 여전히 `dm.groupEnabled` + `dm.groupChannels` 에 의해 제어됩니다.
- 기본값은 `groupPolicy: "allowlist"`입니다(`channels.defaults.groupPolicy`로 오버라이드되지 않는 한). 허용 목록이 구성되지 않으면 그룹 메시지는 차단됩니다.

### 멀티 에이전트 라우팅(`agents.list` + `bindings`)

하나의 Gateway 안에서 여러 개의 격리된 에이전트(분리된 워크스페이스, `agentDir`, 세션)를 실행합니다.
수신 메시지는 바인딩을 통해 에이전트로 라우팅됩니다.

- gateway/configuration.md
  - `id`: 안정적인 에이전트 ID (필수).
  - `default`: 선택 사항; 여러 개가 설정된 경우 첫 번째가 적용되며 경고가 기록됩니다.
    아무것도 설정되지 않은 경우, 목록의 **첫 번째 항목**이 기본 에이전트가 됩니다.
  - `name`: 에이전트의 표시 이름.
  - `workspace`: 기본값 `~/.openclaw/workspace-<agentId>` (`main`의 경우 `agents.defaults.workspace`로 대체).
  - `agentDir`: 기본값 `~/.openclaw/agents/<agentId>/agent`.
  - `model`: 에이전트별 기본 모델로, 해당 에이전트에 대해 `agents.defaults.model`을 재정의합니다.
    - 문자열 형식: `"provider/model"`, `agents.defaults.model.primary`만 재정의합니다.
    - 객체 형식: `{ primary, fallbacks }` (`fallbacks`는 `agents.defaults.model.fallbacks`를 재정의하며, `[]`는 해당 에이전트에 대해 전역 폴백을 비활성화).
  - `identity`: 에이전트별 이름/테마/이모지 (멘션 패턴 + 확인 반응에 사용).
  - `groupChat`: 에이전트별 멘션 게이팅 (`mentionPatterns`).
  - `sandbox`: 에이전트별 샌드박스 설정 (`agents.defaults.sandbox`를 재정의).
    - `mode`: `"off"` | `"non-main"` | `"all"`
    - `workspaceAccess`: `"none"` | `"ro"` | `"rw"`
    - `scope`: `"session"` | `"agent"` | `"shared"`
    - `workspaceRoot`: 사용자 지정 샌드박스 워크스페이스 루트
    - `docker`: 에이전트별 Docker 재정의 (예: `image`, `network`, `env`, `setupCommand`, 제한; `scope: "shared"`일 때는 무시됨)
    - `browser`: 에이전트별 샌드박스 브라우저 재정의 (`scope: "shared"`일 때는 무시됨)
    - `prune`: 에이전트별 샌드박스 정리(pruning) 재정의 (`scope: "shared"`일 때는 무시됨)
  - `subagents`: 에이전트별 하위 에이전트 기본값.
    - `allowAgents`: 이 에이전트에서 `sessions_spawn`을 허용할 에이전트 ID의 허용 목록 (`["*"]` = 모두 허용; 기본값: 동일 에이전트만)
  - `tools`: 에이전트별 도구 제한 (샌드박스 도구 정책 이전에 적용).
    - `profile`: 기본 도구 프로필 (허용/차단 이전에 적용)
    - `allow`: 허용된 도구 이름 배열
    - `deny`: 거부된 도구 이름 배열 (거부가 우선)
- `agents.defaults`: 공유 에이전트 기본값 (모델, 워크스페이스, 샌드박스 등).
- `bindings[]`: 수신 메시지를 `agentId`로 라우팅합니다.
  - `match.channel` (필수)
  - `match.accountId` (선택 사항; `*` = 모든 계정; 생략 = 기본 계정)
  - `match.peer` (선택 사항; `{ kind: direct|group|channel, id }`)
  - `match.guildId` / `match.teamId` (선택 사항; 채널별)

결정적 매칭 순서:

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (정확 일치, peer/guild/team 없음)
5. `match.accountId: "*"` (채널 전체, peer/guild/team 없음)
6. 기본 에이전트 (`agents.list[].default`, 그렇지 않으면 첫 번째 목록 항목, 그 외에는 `"main"`)

각 매칭 단계 내에서는 `bindings`에서 처음으로 일치하는 항목이 적용됩니다.

#### 에이전트별 액세스 프로필(다중 에이전트)

각 에이전트는 자체 샌드박스 + 도구 정책을 가질 수 있습니다. 이를 사용해 하나의 게이트웨이에서 접근 수준을 혼합할 수 있습니다:

- **전체 접근** (개인 에이전트)
- **읽기 전용** 도구 + 워크스페이스
- **파일시스템 접근 없음** (메시징/세션 도구만)

우선순위와 추가 예제는 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools)를 참조하세요.

전체 접근 (샌드박스 없음):

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

읽기 전용 도구 + 읽기 전용 워크스페이스:

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "ro",
        },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

파일시스템 접근 없음 (메시징/세션 도구 활성화):

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: {
          mode: "all",
          scope: "agent",
          workspaceAccess: "none",
        },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

예시: WhatsApp 계정 두 개 → 에이전트 두 개:

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
  channels: {
    whatsapp: {
      accounts: {
        personal: {},
        biz: {},
      },
    },
  },
}
```

### `tools.agentToAgent` (선택 사항)

에이전트 간 메시징은 옵트인 방식입니다:

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `messages.queue`

에이전트 실행이 이미 활성화된 상태에서 수신 메시지가 어떻게 동작하는지 제어합니다.

```json5
{
  messages: {
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog (steer+backlog ok) | interrupt (queue=steer legacy)
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
        discord: "collect",
        imessage: "collect",
        webchat: "collect",
      },
    },
  },
}
```

### `messages.inbound`

**같은 발신자**로부터의 빠른 연속 수신 메시지를 디바운스하여, 연달아 온 여러 메시지가 하나의 에이전트 턴이 되도록 합니다. 디바운싱은 채널 + 대화 단위로 범위가
지정되며, 응답 스레딩/ID 를 위해 가장 최근 메시지를 사용합니다.

```json5
{
  messages: {
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500,
      },
    },
  },
}
```

노트:

- **텍스트 전용** 메시지 묶음을 디바운스하며, 미디어/첨부 파일은 즉시 플러시됩니다.
- 제어 명령(예: `/queue`, `/new`)은 디바운싱을 우회하여 단독으로 유지됩니다.

### `commands` (채팅 명령 처리)

커넥터 전반에서 채팅 명령을 어떻게 활성화할지 제어합니다.

```json5
{
  commands: {
    native: "auto", // register native commands when supported (auto)
    text: true, // parse slash commands in chat messages
    bash: false, // allow ! (alias: /bash) (host-only; requires tools.elevated allowlists)
    bashForegroundMs: 2000, // bash foreground window (0 backgrounds immediately)
    config: false, // allow /config (writes to disk)
    debug: false, // allow /debug (runtime-only overrides)
    restart: false, // allow /restart + gateway restart tool
    useAccessGroups: true, // enforce access-group allowlists/policies for commands
  },
}
```

노트:

- 텍스트 명령은 **단독** 메시지로 보내야 하며, 선행 `/`를 사용해야 합니다(일반 텍스트 별칭 불가).
- `commands.text: false`는 채팅 메시지에서 명령 파싱을 비활성화합니다.
- `commands.native: "auto"`(기본값)는 Discord/Telegram의 네이티브 명령을 켜고 Slack은 끈 채로 둡니다. 지원되지 않는 채널은 텍스트 전용으로 유지됩니다.
- `commands.native: true|false`로 전체를 강제 설정하거나, `channels.discord.commands.native`, `channels.telegram.commands.native`, `channels.slack.commands.native`(bool 또는 `"auto"`)로 채널별 재정의가 가능합니다. `false`는 시작 시 Discord/Telegram에 이전에 등록된 명령을 제거합니다. Slack 명령은 Slack 앱에서 관리됩니다.
- `channels.telegram.customCommands`는 Telegram 봇 메뉴 항목을 추가합니다. 이름은 정규화되며, 네이티브 명령과의 충돌은 무시됩니다.
- `commands.bash: true`는 `! <cmd>`로 호스트 셸 명령을 실행할 수 있게 합니다(` /bash <cmd>`도 별칭으로 동작). `tools.elevated.enabled`가 필요하며, `tools.elevated.allowFrom.<channel>`에서 발신자를 허용 목록에 추가해야 합니다. `commands.bashForegroundMs`는 백그라운드로 전환되기 전 bash가 대기하는 시간을 제어합니다.\` 아래에 위치합니다.
- bash 작업이 실행 중인 동안에는 새로운 `! <cmd>` 요청이 거부됩니다(한 번에 하나만). While a bash job is running, new `! <cmd>` requests are rejected (one at a time).
- `commands.config: true` enables `/config` (reads/writes `openclaw.json`).
- `channels.<provider>.configWrites` gates config mutations initiated by that channel (default: true). This applies to `/config set|unset` plus provider-specific auto-migrations (Telegram supergroup ID changes, Slack channel ID changes).
- `commands.debug: true` enables `/debug` (runtime-only overrides).
- `commands.restart: true` enables `/restart` and the gateway tool restart action.
- `commands.useAccessGroups: false` allows commands to bypass access-group allowlists/policies.
- 슬래시 명령과 지시어는 **권한이 있는 발신자**에게만 적용됩니다. Authorization is derived from
  channel allowlists/pairing plus `commands.useAccessGroups`.

### `web` (WhatsApp web channel runtime)

WhatsApp runs through the gateway’s web channel (Baileys Web). It starts automatically when a linked session exists.
Set `web.enabled: false` to keep it off by default.

```json5
{
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

### `channels.telegram` (bot transport)

OpenClaw starts Telegram only when a `channels.telegram` config section exists. The bot token is resolved from `channels.telegram.botToken` (or `channels.telegram.tokenFile`), with `TELEGRAM_BOT_TOKEN` as a fallback for the default account.
Set `channels.telegram.enabled: false` to disable automatic startup.
Multi-account support lives under `channels.telegram.accounts` (see the multi-account section above). Env tokens only apply to the default account.
Set `channels.telegram.configWrites: false` to block Telegram-initiated config writes (including supergroup ID migrations and `/config set|unset`).

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["tg:123456789"], // optional; "open" requires ["*"]
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Keep answers brief.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Stay on topic.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
      historyLimit: 50, // include last N group messages as context (0 disables)
      replyToMode: "first", // off | first | all
      linkPreview: true, // toggle outbound link previews
      streamMode: "partial", // off | partial | block (draft streaming; separate from block streaming)
      draftChunk: {
        // optional; only for streamMode=block
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true }, // tool action gates (false disables)
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: {
        // transport overrides
        autoSelectFamily: false,
      },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook", // requires webhookSecret
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

Draft streaming notes:

- Uses Telegram `sendMessageDraft` (draft bubble, not a real message).
- Requires **private chat topics** (message_thread_id in DMs; bot has topics enabled).
- `/reasoning stream` streams reasoning into the draft, then sends the final answer.
  Retry policy defaults and behavior are documented in [Retry policy](/concepts/retry).

### `channels.discord` (bot transport)

Configure the Discord bot by setting the bot token and optional gating:
Multi-account support lives under `channels.discord.accounts` (see the multi-account section above). Env tokens only apply to the default account.

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8, // clamp inbound media size
      allowBots: false, // allow bot-authored messages
      actions: {
        // tool action gates (false disables)
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dm: {
        enabled: true, // disable all DMs when false
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["1234567890", "steipete"], // optional DM allowlist ("open" requires ["*"])
        groupEnabled: false, // enable group DMs
        groupChannels: ["openclaw-dm"], // optional group DM allowlist
      },
      guilds: {
        "123456789012345678": {
          // guild id (preferred) or slug
          slug: "friends-of-openclaw",
          requireMention: false, // per-guild default
          reactionNotifications: "own", // off | own | all | allowlist
          users: ["987654321098765432"], // optional per-guild user allowlist
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Short answers only.",
            },
          },
        },
      },
      historyLimit: 20, // include last N guild messages as context
      textChunkLimit: 2000, // optional outbound text chunk size (chars)
      chunkMode: "length", // optional chunking mode (length | newline)
      maxLinesPerMessage: 17, // soft max lines per message (Discord UI clipping)
      retry: {
        // outbound retry policy
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

OpenClaw starts Discord only when a `channels.discord` config section exists. The token is resolved from `channels.discord.token`, with `DISCORD_BOT_TOKEN` as a fallback for the default account (unless `channels.discord.enabled` is `false`). Use `user:<id>` (DM) or `channel:<id>` (guild channel) when specifying delivery targets for cron/CLI commands; bare numeric IDs are ambiguous and rejected.
Guild slugs are lowercase with spaces replaced by `-`; channel keys use the slugged channel name (no leading `#`). Prefer guild ids as keys to avoid rename ambiguity.
Bot-authored messages are ignored by default. Enable with `channels.discord.allowBots` (own messages are still filtered to prevent self-reply loops).
Reaction notification modes:

- `off`: 반응 이벤트 없음.
- `own`: 봇 자신의 메시지에 대한 반응 (기본값).
- `all`: 모든 메시지의 모든 반응.
- `allowlist`: `guilds.<id>.users` 의 반응만 모든 메시지에 적용 (빈 목록은 비활성화).
  Outbound text is chunked by `channels.discord.textChunkLimit` (default 2000). Set `channels.discord.chunkMode="newline"` to split on blank lines (paragraph boundaries) before length chunking. Discord 클라이언트는 매우 긴 메시지를 잘라서 표시할 수 있으므로, `channels.discord.maxLinesPerMessage`(기본값 17)는 2000자 미만이더라도 긴 여러 줄 응답을 분할합니다.
  재시도 정책의 기본값과 동작은 [Retry policy](/concepts/retry)에 문서화되어 있습니다.

### `channels.googlechat` (Chat API 웹훅)

Google Chat은 앱 수준 인증(서비스 계정)을 사용하는 HTTP 웹훅으로 동작합니다.
다중 계정 지원은 `channels.googlechat.accounts` 아래에 있습니다(위의 다중 계정 섹션 참조). 환경 변수는 기본 계정에만 적용됩니다.

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 선택 사항; 멘션 감지 향상
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["users/1234567890"], // 선택 사항; "open"에는 ["*"] 필요
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

노트:

- 서비스 계정 JSON은 인라인(`serviceAccount`) 또는 파일 기반(`serviceAccountFile`)일 수 있습니다.
- 기본 계정에 대한 환경 변수 대체값: `GOOGLE_CHAT_SERVICE_ACCOUNT` 또는 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- `audienceType`과 `audience`는 Chat 앱의 웹훅 인증 설정과 일치해야 합니다.
- 전달 대상을 설정할 때 `spaces/<spaceId>` 또는 `users/<userId|email>`을 사용하세요.

### `channels.slack` (소켓 모드)

Slack은 소켓 모드로 실행되며 봇 토큰과 앱 토큰이 모두 필요합니다:

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["U123", "U456", "*"], // 선택 사항; "open"에는 ["*"] 필요
        groupEnabled: false,
        groupChannels: ["G123"],
      },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Short answers only.",
        },
      },
      historyLimit: 50, // 마지막 N개의 채널/그룹 메시지를 컨텍스트로 포함 (0은 비활성화)
      allowBots: false,
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

다중 계정 지원은 `channels.slack.accounts` 아래에 있습니다(위의 다중 계정 섹션 참조). 환경 변수 토큰은 기본 계정에만 적용됩니다.

OpenClaw는 공급자가 활성화되어 있고 두 토큰이 모두 설정되면(SLACK_BOT_TOKEN + SLACK_APP_TOKEN 또는 설정을 통해) Slack을 시작합니다. 크론/CLI 명령의 전달 대상을 지정할 때 DM은 `user:<id>`, 채널은 `channel:<id>`를 사용하세요.
Slack에서 시작된 설정 쓰기(채널 ID 마이그레이션 및 `/config set|unset` 포함)를 차단하려면 `channels.slack.configWrites: false`로 설정하세요.

봇이 작성한 메시지는 기본적으로 무시됩니다. `channels.slack.allowBots` 또는 `channels.slack.channels.<id>`로 활성화하세요..allowBots\`.

반응 알림 모드:

- `off`: 반응 이벤트 없음.
- `own`: 봇 자신의 메시지에 대한 반응 (기본값).
- `all`: 모든 메시지의 모든 반응.
- `allowlist`: 모든 메시지에서 `channels.slack.reactionAllowlist`에 있는 사용자들의 반응만 허용(빈 목록은 비활성화).

스레드 세션 격리:

- `channels.slack.thread.historyScope`는 스레드 기록이 스레드별(`thread`, 기본값)인지 채널 전체(`channel`)에서 공유되는지를 제어합니다.
- `channels.slack.thread.inheritParent`는 새 스레드 세션이 상위 채널의 대화를 상속할지 여부를 제어합니다(기본값: false).

Slack 액션 그룹(`slack` 도구 액션을 제어):

| 작업 그룹      | 기본값     | 참고 자료         |
| ---------- | ------- | ------------- |
| reactions  | enabled | 반응 추가 + 목록    |
| messages   | enabled | 읽기/전송/편집/삭제   |
| pins       | enabled | 고정/해제/목록      |
| memberInfo | enabled | 멤버 정보         |
| emojiList  | enabled | 사용자 지정 이모지 목록 |

### `channels.mattermost` (봇 토큰)

Mattermost 는 플러그인으로 제공되며 코어 설치에 번들로 포함되지 않습니다.
먼저 설치하세요: `openclaw plugins install @openclaw/mattermost`(또는 git 체크아웃에서 `./extensions/mattermost`).

Mattermost는 봇 토큰과 서버의 기본 URL이 필요합니다:

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

OpenClaw는 계정이 구성되고(봇 토큰 + 기본 URL) 활성화되면 Mattermost를 시작합니다. 토큰과 기본 URL은 기본 계정의 경우 `channels.mattermost.botToken` + `channels.mattermost.baseUrl` 또는 `MATTERMOST_BOT_TOKEN` + `MATTERMOST_URL`에서 확인됩니다(`channels.mattermost.enabled`가 `false`가 아닌 경우).

채팅 모드:

- `oncall`(기본값): @멘션된 경우에만 채널 메시지에 응답합니다.
- `onmessage`: 모든 채널 메시지에 응답합니다.
- `onchar`: 메시지가 트리거 접두사(`channels.mattermost.oncharPrefixes`, 기본값 `[">", "!"]`)로 시작할 때 응답합니다.

접근 제어:

- 기본 DM: `channels.mattermost.dmPolicy="pairing"`(알 수 없는 발신자는 페어링 코드를 받음).
- 공개 다이렉트 메시지: `channels.mattermost.dmPolicy="open"` 와 `channels.mattermost.allowFrom=["*"]` 를 함께 사용합니다.
- 기본값은 `groupPolicy: "allowlist"` 입니다 (`channels.defaults.groupPolicy` 로 재정의되지 않는 한). 허용 목록이 구성되지 않으면 그룹 메시지는 차단됩니다.

다중 계정 지원은 `channels.mattermost.accounts` 아래에 있습니다(위의 다중 계정 섹션 참조). 환경 변수는 기본 계정에만 적용됩니다.
전송 대상을 지정할 때 `channel:<id>` 또는 `user:<id>`(또는 `@username`)를 사용하세요. 접두사 없는 id는 채널 id로 처리됩니다.

### `channels.signal` (signal-cli)

Signal 반응은 시스템 이벤트를 발생시킬 수 있습니다(공유 반응 툴링):

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50, // 최근 N개의 그룹 메시지를 컨텍스트로 포함 (0은 비활성화)
    },
  },
}
```

반응 알림 모드:

- `off`: 반응 이벤트 없음.
- `own`: 봇 자신의 메시지에 대한 반응 (기본값).
- `all`: 모든 메시지의 모든 반응.
- `allowlist`: 모든 메시지에 대해 `channels.signal.reactionAllowlist`에 포함된 발신자의 반응만 허용합니다(빈 목록은 비활성화).

### `channels.imessage` (imsg CLI)

OpenClaw는 `imsg rpc`(stdio 상의 JSON-RPC)를 실행합니다. 데몬이나 포트가 필요하지 않습니다.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host", // SSH 래퍼 사용 시 원격 첨부파일을 위한 SCP
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50, // 최근 N개의 그룹 메시지를 컨텍스트로 포함 (0은 비활성화)
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

다중 계정 지원은 `channels.imessage.accounts` 아래에 있습니다(위의 다중 계정 섹션 참조).

노트:

- Messages DB에 대한 전체 디스크 접근 권한이 필요합니다.
- 첫 전송 시 Messages 자동화 권한을 요청합니다.
- `chat_id:<id>` 대상을 사용하는 것을 권장합니다. 채팅 목록을 보려면 `imsg chats --limit 20`을 사용하세요.
- `channels.imessage.cliPath`는 래퍼 스크립트(예: `imsg rpc`를 실행하는 다른 Mac으로의 `ssh`)를 가리킬 수 있습니다. 비밀번호 프롬프트를 피하려면 SSH 키를 사용하세요.
- 원격 SSH 래퍼의 경우, `includeAttachments`가 활성화되면 SCP로 첨부파일을 가져오기 위해 `channels.imessage.remoteHost`를 설정하세요.

래퍼 예제:

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

### `agents.defaults.workspace`

에이전트가 파일 작업에 사용하는 **단일 전역 워크스페이스 디렉터리**를 설정합니다.

기본값: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

`agents.defaults.sandbox`가 활성화된 경우, 메인 세션이 아닌 세션은 `agents.defaults.sandbox.workspaceRoot` 아래의 스코프별 워크스페이스로 이를 재정의할 수 있습니다.

### `agents.defaults.repoRoot`

시스템 프롬프트의 Runtime 줄에 표시할 선택적 리포지토리 루트입니다. 설정되지 않은 경우, OpenClaw는 워크스페이스(및 현재 작업 디렉터리)에서 위로 탐색하며 `.git` 디렉터리를 감지하려고 시도합니다. 사용하려면 경로가 존재해야 합니다.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

워크스페이스 부트스트랩 파일(`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`)의 자동 생성을 비활성화합니다.

워크스페이스 파일이 리포지토리에서 제공되는 사전 시드된 배포에 사용하세요.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

잘리기 전에 시스템 프롬프트에 주입되는 각 워크스페이스 부트스트랩 파일의 최대 문자 수입니다. 기본값: `20000`.

파일이 이 제한을 초과하면, OpenClaw는 경고를 기록하고 마커와 함께 앞/뒤를 잘라 주입합니다.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.userTimezone`

사용자의 시간대를 **시스템 프롬프트 컨텍스트**에 설정합니다(메시지 봉투의 타임스탬프에는 적용되지 않음). 설정되지 않은 경우, OpenClaw는 실행 시 호스트의 시간대를 사용합니다.

```json5
환경 변수 + `.env`
```

### `agents.defaults.timeFormat`

시스템 프롬프트의 현재 날짜 및 시간 섹션에 표시되는 **시간 형식**을 제어합니다.
기본값: `auto`(OS 설정).

```json5
2026-02-08T09:25:33Z
```

### `메시지`

수신/발신 접두사와 선택적 확인(ack) 반응을 제어합니다.
큐잉, 세션, 스트리밍 컨텍스트에 대해서는 [Messages](/concepts/messages)를 참조하세요.

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions",
    removeAckAfterReply: false,
  },
}
```

`responsePrefix`는 이미 존재하지 않는 한 채널 전반에 걸쳐 **모든 발신 답변**(도구 요약, 블록 스트리밍, 최종 답변)에 적용됩니다.

재정의는 채널별 및 계정별로 구성할 수 있습니다:

- `channels.<channel>.responsePrefix`
- `channels.<channel>.accounts.<id>.responsePrefix`

해결 순서 (가장 구체적인 항목이 우선):

1. `channels.<channel>.accounts.<id>.responsePrefix`
2. `channels.<channel>.responsePrefix`
3. `messages.responsePrefix`

의미:

- `undefined`는 다음 단계로 그대로 전달됩니다.
- `""`는 접두사를 명시적으로 비활성화하고 연쇄 적용을 중단합니다.
- `"auto"`는 라우팅된 에이전트에 대해 `[{identity.name}]`를 도출합니다.

재정의는 확장을 포함한 모든 채널과 모든 발신 답변 유형에 적용됩니다.

`messages.responsePrefix`가 설정되지 않은 경우 기본적으로 접두사는 적용되지 않습니다. WhatsApp 자기 자신과의 채팅 답변은 예외입니다: 설정된 경우 기본값은 `[{identity.name}]`이며, 그렇지 않으면 `[openclaw]`를 사용하여 같은 휴대전화 내 대화가 읽기 쉽도록 합니다.
설정 시 라우팅된 에이전트에 대해 `[{identity.name}]`를 도출하려면 `"auto"`로 설정하세요.

#### 템플릿 변수

`responsePrefix` 문자열에는 동적으로 해석되는 템플릿 변수를 포함할 수 있습니다:

| 변수                              | 설명         | 예제                                   |
| ------------------------------- | ---------- | ------------------------------------ |
| {model}                         | 짧은 모델 이름   | `claude-opus-4-6`, `gpt-4o`          |
| {modelFull}                     | 전체 모델 식별자  | `anthropic/claude-opus-4-6`          |
| {provider}                      | 제공자 이름     | `anthropic`, `openai`                |
| {thinkingLevel}                 | 현재 사고 수준   | `high`, `low`, `off`                 |
| {identity.name} | 에이전트 식별 이름 | (`"auto"` 모드와 동일) |

변수는 대소문자를 구분하지 않습니다 (`{MODEL}` = `{model}`). `{think}`는 `{thinkingLevel}`의 별칭입니다.
해결되지 않은 변수는 리터럴 텍스트로 그대로 유지됩니다.

```json5
{
  messages: {
    responsePrefix: "[{model} | think:{thinkingLevel}]",
  },
}
```

예시 출력: `[claude-opus-4-6 | think:high] Here's my response...`

WhatsApp 수신 접두사는 `channels.whatsapp.messagePrefix`를 통해 구성됩니다(사용 중단됨:
`messages.messagePrefix`). 기본값은 **변경되지 않습니다**: `channels.whatsapp.allowFrom`이 비어 있으면 `"[openclaw]"`, 그렇지 않으면 `""`(접두사 없음)입니다. `"[openclaw]"`를 사용할 때 라우팅된 에이전트에 `identity.name`이 설정되어 있으면 OpenClaw는 대신 `[{identity.name}]`를 사용합니다.

`ackReaction`은 반응을 지원하는 채널(Slack/Discord/Telegram/Google Chat)에서 수신 메시지를 확인하기 위해 최선의 노력(best-effort)으로 이모지 반응을 전송합니다. 기본값은 설정되어 있을 경우 활성 에이전트의 `identity.emoji`이며, 그렇지 않으면 `"👀"`입니다. `""`로 설정하면 비활성화됩니다.

`ackReactionScope`는 반응이 언제 실행되는지를 제어합니다:

- `group-mentions` (기본값): 그룹/룸에서 멘션이 필요하고 **봇이 멘션된 경우에만**
- `group-all`: 모든 그룹/룸 메시지
- `direct`: 다이렉트 메시지 전용
- `all`: 모든 메시지

`removeAckAfterReply`는 응답이 전송된 후 봇의 확인(ack) 반응을 제거합니다
(Slack/Discord/Telegram/Google Chat 전용). 기본값: `false`.

#### `messages.tts`

발신 응답에 대해 텍스트 음성 변환(text-to-speech)을 활성화합니다. 켜져 있으면 OpenClaw가 ElevenLabs 또는 OpenAI를 사용해 오디오를 생성하고
응답에 첨부합니다. Telegram은 Opus 음성 노트를 사용하며,
다른 채널은 MP3 오디오를 전송합니다.

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all (include tool/block replies)
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true,
      },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

노트:

- `messages.tts.auto`는 자동 TTS를 제어합니다(`off`, `always`, `inbound`, `tagged`).
- `/tts off|always|inbound|tagged`는 세션별 자동 모드를 설정합니다(설정보다 우선).
- `messages.tts.enabled`는 레거시이며, doctor가 이를 `messages.tts.auto`로 마이그레이션합니다.
- `prefsPath`는 로컬 오버라이드(provider/limit/summarize)를 저장합니다.
- `maxTextLength`는 TTS 입력의 하드 제한이며, 요약은 이에 맞게 잘립니다.
- `summaryModel`은 자동 요약을 위해 `agents.defaults.model.primary`를 재정의합니다.
  - `provider/model` 또는 `agents.defaults.models`의 별칭(alias)을 허용합니다.
- `modelOverrides`는 `[[tts:...]]` 태그와 같은 모델 기반 오버라이드를 활성화합니다(기본값: 켜짐).
- `/tts limit` 및 `/tts summary`는 사용자별 요약 설정을 제어합니다.
- `apiKey` 값은 `ELEVENLABS_API_KEY`/`XI_API_KEY` 및 `OPENAI_API_KEY`로 폴백됩니다.
- `elevenlabs.baseUrl`은 ElevenLabs API 기본 URL을 재정의합니다.
- `elevenlabs.voiceSettings`는 `stability`/`similarityBoost`/`style`(0..1),
  `useSpeakerBoost`, 그리고 `speed`(0.5..2.0)를 지원합니다.

### `talk`

Talk 모드의 기본값(macOS/iOS/Android). Voice ID가 설정되지 않은 경우 `ELEVENLABS_VOICE_ID` 또는 `SAG_VOICE_ID`로 폴백됩니다.
`apiKey`가 설정되지 않은 경우 `ELEVENLABS_API_KEY`(또는 게이트웨이의 셸 프로필)로 폴백됩니다.
`voiceAliases`를 사용하면 Talk 지시문에서 친숙한 이름을 사용할 수 있습니다(예: `"voice":"Clawd"`).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

### `agents.defaults`

임베디드 에이전트 런타임(model/thinking/verbose/timeouts)을 제어합니다.
`agents.defaults.models`는 구성된 모델 카탈로그를 정의하며(`/model`의 허용 목록 역할도 함).
`agents.defaults.model.primary`는 기본 모델을 설정하고, `agents.defaults.model.fallbacks`는 전역 페일오버입니다.
`agents.defaults.imageModel`은 선택 사항이며 **기본 모델에 이미지 입력이 없는 경우에만 사용됩니다**.
각 `agents.defaults.models` 항목에는 다음이 포함될 수 있습니다:

- `alias`(선택 사항, 예: `/opus`와 같은 모델 단축키).
- `params`(선택 사항: 모델 요청으로 그대로 전달되는 공급자별 API 파라미터).

`params`는 스트리밍 실행(임베디드 에이전트 + 컴팩션)에도 적용됩니다. 현재 지원되는 키: `temperature`, `maxTokens`. 이는 호출 시 옵션과 병합되며, 호출자가 제공한 값이 우선합니다. `temperature`는 고급 조정 옵션입니다—모델의 기본값을 알고 있으며 변경이 필요한 경우가 아니라면 설정하지 마세요.

Example:

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-5-20250929": {
          params: { temperature: 0.6 },
        },
        "openai/gpt-5.2": {
          params: { maxTokens: 8192 },
        },
      },
    },
  },
}
```

Z.AI GLM-4.x 모델은 다음 중 하나를 하지 않는 한 자동으로 사고(thinking) 모드를 활성화합니다:

- `--thinking off`를 설정하거나,
- `agents.defaults.models["zai/<model>"].params.thinking`을 직접 정의합니다.

OpenClaw에는 몇 가지 내장 별칭(aliase) 단축 표기도 함께 제공됩니다. 기본값은 해당 모델이 이미 `agents.defaults.models`에 존재할 때만 적용됩니다:

- `opus` -> `anthropic/claude-opus-4-6`
- `sonnet` -> `anthropic/claude-sonnet-4-5`
- `gpt` -> `openai/gpt-5.2`
- `gpt-mini` -> `openai/gpt-5-mini`
- `gemini` -> `google/gemini-3-pro-preview`
- `gemini-flash` -> `google/gemini-3-flash-preview`

동일한 별칭 이름(대소문자 무시)을 직접 구성하면, 사용자 값이 우선합니다(기본값은 절대 덮어쓰지 않음).

예시: Opus 4.6을 기본으로 하고 MiniMax M2.1을 폴백으로 사용(호스팅된 MiniMax):

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
    },
  },
}
```

MiniMax 인증: `MINIMAX_API_KEY`(환경 변수)를 설정하거나 `models.providers.minimax`를 구성하세요.

#### `agents.defaults.cliBackends` (CLI 폴백)

텍스트 전용 폴백 실행(도구 호출 없음)을 위한 선택적 CLI 백엔드입니다. API 제공자가 실패했을 때의 백업 경로로 유용합니다. 파일 경로를 받는 `imageArg`를 구성하면 이미지 패스스루가 지원됩니다.

노트:

- CLI 백엔드는 **텍스트 우선**이며, 도구는 항상 비활성화됩니다.
- `sessionArg`가 설정되면 세션이 지원되며, 세션 ID는 백엔드별로 유지됩니다.
- `claude-cli`의 경우 기본값이 미리 연결되어 있습니다. PATH가 최소인 경우(launchd/systemd) 명령 경로를 재정의하세요.

Example:

```json5
{
  agents: {
    defaults: {
      cliBackends: {
        "claude-cli": {
          command: "/opt/homebrew/bin/claude",
        },
        "my-cli": {
          command: "my-cli",
          args: ["--json"],
          output: "json",
          modelArg: "--model",
          sessionArg: "--session",
          sessionMode: "existing",
          systemPromptArg: "--system",
          systemPromptWhen: "first",
          imageArg: "--image",
          imageMode: "repeat",
        },
      },
    },
  },
}
```

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "Opus" },
        "anthropic/claude-sonnet-4-1": { alias: "Sonnet" },
        "openrouter/deepseek/deepseek-r1:free": {},
        "zai/glm-4.7": {
          alias: "GLM",
          params: {
            thinking: {
              type: "enabled",
              clear_thinking: false,
            },
          },
        },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: [
          "openrouter/deepseek/deepseek-r1:free",
          "openrouter/meta-llama/llama-3.3-70b-instruct:free",
        ],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      heartbeat: {
        every: "30m",
        target: "last",
      },
      maxConcurrent: 3,
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
      exec: {
        backgroundMs: 10000,
        timeoutSec: 1800,
        cleanupMs: 1800000,
      },
      contextTokens: 200000,
    },
  },
}
```

#### `agents.defaults.contextPruning` (도구 결과 가지치기)

`agents.defaults.contextPruning`은 LLM으로 요청을 보내기 직전에 인메모리 컨텍스트에서 **오래된 도구 결과**를 제거합니다.
이는 디스크에 저장된 세션 기록을 수정하지 **않습니다**(`*.jsonl`은 완전하게 유지됨).

이는 시간이 지남에 따라 큰 도구 출력이 누적되는 수다스러운 에이전트의 토큰 사용량을 줄이기 위한 것입니다.

상위 수준 개요:

- 사용자/어시스턴트 메시지는 절대 건드리지 않습니다.
- 마지막 `keepLastAssistants`개의 어시스턴트 메시지를 보호합니다(그 이후의 도구 결과는 가지치기되지 않음).
- 부트스트랩 프리픽스를 보호합니다(첫 번째 사용자 메시지 이전의 내용은 가지치기되지 않음).
- 모드:
  - `adaptive`: 추정된 컨텍스트 비율이 `softTrimRatio`를 초과하면 과도하게 큰 도구 결과를 소프트 트림합니다(앞/뒤 유지).
    그런 다음 추정된 컨텍스트 비율이 `hardClearRatio`를 초과하고 **그리고**
    정리 가능한 도구 결과의 분량이 충분할 때(`minPrunableToolChars`)
    가장 오래된 적격 도구 결과를 하드 클리어합니다.
  - `aggressive`: 비율 검사 없이 컷오프 이전의 적격 도구 결과를 항상 `hardClear.placeholder`로 대체합니다.

소프트 vs 하드 프루닝(LLM에 전송되는 컨텍스트에서 무엇이 바뀌는지):

- **소프트 트림**: _과도하게 큰_ 도구 결과에만 적용됩니다. 처음 + 끝을 유지하고 가운데에 `...`를 삽입합니다.
  - 이전: `toolResult("…very long output…")`
  - 이후: `toolResult("HEAD…\n...\n…TAIL\n\n[Tool result trimmed: …]")`
- **하드 클리어**: 전체 도구 결과를 플레이스홀더로 대체합니다.
  - 이전: `toolResult("…very long output…")`
  - 이후: `toolResult("[Old tool result content cleared]")`

참고 / 현재 제한 사항:

- 현재 **이미지 블록을 포함한 도구 결과는 건너뜁니다**(트림/클리어되지 않음).
- 추정된 “컨텍스트 비율”은 정확한 토큰이 아닌 **문자 수**를 기준으로 합니다(근사치).
- 세션에 아직 `keepLastAssistants` 이상의 어시스턴트 메시지가 없으면 프루닝을 건너뜁니다.
- `aggressive` 모드에서는 `hardClear.enabled`가 무시됩니다(적격 도구 결과는 항상 `hardClear.placeholder`로 대체됨).

기본값(adaptive):

```json5
{
  agents: { defaults: { contextPruning: { mode: "adaptive" } } },
}
```

비활성화하려면:

```json5
{
  agents: { defaults: { contextPruning: { mode: "off" } } },
}
```

기본값(`mode`가 `"adaptive"` 또는 `"aggressive"`일 때):

- `keepLastAssistants`: `3`
- `softTrimRatio`: `0.3` (adaptive 전용)
- `hardClearRatio`: `0.5` (adaptive 전용)
- `minPrunableToolChars`: `50000` (adaptive 전용)
- `softTrim`: `{ maxChars: 4000, headChars: 1500, tailChars: 1500 }` (adaptive 전용)
- `hardClear`: `{ enabled: true, placeholder: "[Old tool result content cleared]" }`

예시(aggressive, 최소):

```json5
{
  agents: { defaults: { contextPruning: { mode: "aggressive" } } },
}
```

예시(adaptive 튜닝):

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "adaptive",
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        // 선택 사항: 특정 도구로 프루닝 제한(deny 우선; "*" 와일드카드 지원)
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

동작 세부 사항은 [/concepts/session-pruning](/concepts/session-pruning)을 참조하세요.

#### `agents.defaults.compaction` (헤드룸 예약 + 메모리 플러시)

`agents.defaults.compaction.mode`는 컴팩션 요약 전략을 선택합니다. 기본값은 `default`이며, 매우 긴 히스토리에 대해 청크 요약을 활성화하려면 `safeguard`로 설정하세요. [/concepts/compaction](/concepts/compaction)을 참고하십시오.

`agents.defaults.compaction.reserveTokensFloor`는 Pi 컴팩션을 위한 최소 `reserveTokens`
값을 강제합니다(기본값: `20000`). 비활성화하려면 `0`으로 설정하세요.

`agents.defaults.compaction.memoryFlush`는 자동 컴팩션 전에 **무음** 에이전트 턴을 실행하여
모델이 내구성 있는 메모리를 디스크에 저장하도록 지시합니다(예:
`memory/YYYY-MM-DD.md`). 세션 토큰 추정치가 컴팩션 한도 아래의
소프트 임계값을 초과하면 트리거됩니다.

레거시 기본값:

- `memoryFlush.enabled`: `true`
- `memoryFlush.softThresholdTokens`: `4000`
- `memoryFlush.prompt` / `memoryFlush.systemPrompt`: built-in defaults with `NO_REPLY`
- Note: memory flush is skipped when the session workspace is read-only
  (`agents.defaults.sandbox.workspaceAccess: "ro"` or `"none"`).

Example (tuned):

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard",
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

Block streaming:

- `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (기본값 꺼짐).

- Channel overrides: `*.blockStreaming` (and per-account variants) to force block streaming on/off.
  Non-Telegram channels require an explicit `*.blockStreaming: true` to enable block replies.

- `agents.defaults.blockStreamingBreak`: `"text_end"` or `"message_end"` (default: text_end).

- `agents.defaults.blockStreamingChunk`: soft chunking for streamed blocks. Defaults to
  800–1200 chars, prefers paragraph breaks (`\n\n`), then newlines, then sentences.
  Example:

  ```json5
  {
    agents: { defaults: { blockStreamingChunk: { minChars: 800, maxChars: 1200 } } },
  }
  ```

- `agents.defaults.blockStreamingCoalesce`: merge streamed blocks before sending.
  Defaults to `{ idleMs: 1000 }` and inherits `minChars` from `blockStreamingChunk`
  with `maxChars` capped to the channel text limit. Signal/Slack/Discord/Google Chat default
  to `minChars: 1500` unless overridden.
  Channel overrides: `channels.whatsapp.blockStreamingCoalesce`, `channels.telegram.blockStreamingCoalesce`,
  `channels.discord.blockStreamingCoalesce`, `channels.slack.blockStreamingCoalesce`, `channels.mattermost.blockStreamingCoalesce`,
  `channels.signal.blockStreamingCoalesce`, `channels.imessage.blockStreamingCoalesce`, `channels.msteams.blockStreamingCoalesce`,
  `channels.googlechat.blockStreamingCoalesce`
  (and per-account variants).

- `agents.defaults.humanDelay`: randomized pause between **block replies** after the first.
  Modes: `off` (default), `natural` (800–2500ms), `custom` (use `minMs`/`maxMs`).
  Per-agent override: `agents.list[].humanDelay`.
  Example:

  ```json5
  {
    agents: { defaults: { humanDelay: { mode: "natural" } } },
  }
  ```

  See [/concepts/streaming](/concepts/streaming) for behavior + chunking details.

Typing indicators:

- `agents.defaults.typingMode`: `"never" | "instant" | "thinking" | "message"`. Defaults to
  `instant` for direct chats / mentions and `message` for unmentioned group chats.
- `session.typingMode`: per-session override for the mode.
- `agents.defaults.typingIntervalSeconds`: how often the typing signal is refreshed (default: 6s).
- `session.typingIntervalSeconds`: per-session override for the refresh interval.
  See [/concepts/typing-indicators](/concepts/typing-indicators) for behavior details.

`agents.defaults.model.primary` should be set as `provider/model` (e.g. `anthropic/claude-opus-4-6`).
Aliases come from `agents.defaults.models.*.alias` (e.g. `Opus`).
If you omit the provider, OpenClaw currently assumes `anthropic` as a temporary
deprecation fallback.
Z.AI models are available as `zai/<model>` (e.g. `zai/glm-4.7`) and require
`ZAI_API_KEY` (or legacy `Z_AI_API_KEY`) in the environment.

`agents.defaults.heartbeat` configures periodic heartbeat runs:

- `every`: duration string (`ms`, `s`, `m`, `h`); default unit minutes. Default:
  `30m`. Set `0m` to disable.
- `model`: optional override model for heartbeat runs (`provider/model`).
- `includeReasoning`: when `true`, heartbeats will also deliver the separate `Reasoning:` message when available (same shape as `/reasoning on`). Default: `false`.
- `session`: optional session key to control which session the heartbeat runs in. Default: `main`.
- `to`: 선택적 수신자 오버라이드(채널별 ID, 예: WhatsApp의 경우 E.164, Telegram의 경우 채팅 ID).
- `target`: 선택적 전송 채널(`last`, `whatsapp`, `telegram`, `discord`, `slack`, `msteams`, `signal`, `imessage`, `none`). 기본값: `last`.
- `prompt`: 하트비트 본문에 대한 선택적 오버라이드(기본값: 존재한다면 `Read HEARTBEAT.md if it exists (workspace context).`). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). 오버라이드는 그대로 전송됩니다. 파일을 계속 읽고 싶다면 `Read HEARTBEAT.md\` 줄을 포함하세요.
- `ackMaxChars`: 전달 전에 `HEARTBEAT_OK` 이후 허용되는 최대 문자 수(기본값: 300).

에이전트별 Heartbeat:

- 특정 에이전트에 대해 하트비트 설정을 활성화하거나 오버라이드하려면 `agents.list[].heartbeat`를 설정하세요.
- 어떤 에이전트 항목이든 `heartbeat`를 정의하면 **그 에이전트들만** 하트비트를 실행합니다; 기본값은 해당 에이전트들을 위한 공통 기준선이 됩니다.

Heartbeat 는 전체 에이전트 턴을 실행합니다. 짧은 간격은 더 많은 토큰을 소모합니다; `every`에 유의하고, `HEARTBEAT.md`를 아주 작게 유지하고, 그리고/또는 더 저렴한 `model`을 선택하세요.

`tools.exec`는 백그라운드 exec 기본값을 구성합니다:

- `backgroundMs`: 자동 백그라운드 전환 전까지의 시간(ms, 기본값 10000)
- `timeoutSec`: 이 실행 시간(초) 이후 자동 종료(기본값 1800)
- `cleanupMs`: 완료된 세션을 메모리에 유지하는 시간(ms, 기본값 1800000)
- `notifyOnExit`: 백그라운드 exec 종료 시 시스템 이벤트를 큐에 넣고 하트비트를 요청합니다(기본값 true)
- `applyPatch.enabled`: 실험적 `apply_patch` 활성화(OpenAI/OpenAI Codex 전용; 기본값 false)
- `applyPatch.allowModels`: 모델 ID의 선택적 허용 목록(예: `gpt-5.2` 또는 `openai/gpt-5.2`)
  참고: `applyPatch`는 `tools.exec` 하위에만 있습니다.

`tools.web`는 웹 검색 + 가져오기 도구를 구성합니다:

- `tools.web.search.enabled` (기본값: 키가 존재할 때 true)
- `tools.web.search.apiKey` (권장: `openclaw configure --section web`을 통해 설정하거나 `BRAVE_API_KEY` 환경 변수를 사용)
- `tools.web.search.maxResults` (1–10, 기본값 5)
- `tools.web.search.timeoutSeconds` (기본값 30)
- `tools.web.search.cacheTtlMinutes` (기본값 15)
- `tools.web.fetch.enabled` (기본값 true)
- `tools.web.fetch.maxChars` (기본값 50000)
- `tools.web.fetch.maxCharsCap` (기본값 50000; 구성/도구 호출의 maxChars를 상한 처리)
- `tools.web.fetch.timeoutSeconds` (기본값 30)
- `tools.web.fetch.cacheTtlMinutes` (기본값 15)
- `tools.web.fetch.userAgent` (선택적 오버라이드)
- `tools.web.fetch.readability` (기본값 true; 비활성화하면 기본 HTML 정리만 사용)
- `tools.web.fetch.firecrawl.enabled` (API 키가 설정되어 있을 때 기본값 true)
- `tools.web.fetch.firecrawl.apiKey` (선택 사항; 기본값 `FIRECRAWL_API_KEY`)
- `tools.web.fetch.firecrawl.baseUrl` (기본값 [https://api.firecrawl.dev](https://api.firecrawl.dev))
- `tools.web.fetch.firecrawl.onlyMainContent` (기본값 true)
- `tools.web.fetch.firecrawl.maxAgeMs` (선택)
- `tools.web.fetch.firecrawl.timeoutSeconds` (선택)

`tools.media`는 인바운드 미디어 이해(이미지/오디오/비디오)를 구성합니다:

- `tools.media.models`: 공유 모델 목록(기능 태그 지정; 기능별 목록 이후에 사용).
- `tools.media.concurrency`: 최대 동시 기능 실행 수(기본값 2).
- `tools.media.image` / `tools.media.audio` / `tools.media.video`:
  - `enabled`: 옵트아웃 스위치(모델이 구성되어 있을 때 기본값 true).
  - `prompt`: 선택적 프롬프트 오버라이드(이미지/비디오는 `maxChars` 힌트를 자동으로 추가).
  - `maxChars`: 최대 출력 문자 수(이미지/비디오 기본값 500; 오디오는 미설정).
  - `maxBytes`: 전송할 최대 미디어 크기(기본값: 이미지 10MB, 오디오 20MB, 비디오 50MB).
  - `timeoutSeconds`: 요청 타임아웃(기본값: 이미지 60초, 오디오 60초, 비디오 120초).
  - `language`: 선택적 오디오 힌트.
  - `attachments`: 첨부 정책(`mode`, `maxAttachments`, `prefer`).
  - `scope`: `match.channel`, `match.chatType`, 또는 `match.keyPrefix`를 사용하는 선택적 게이팅(첫 번째 일치가 우선).
  - `models`: 정렬된 모델 항목 목록; 실패하거나 미디어가 너무 크면 다음 항목으로 폴백됩니다.
- 각 `models[]` 항목:
  - 1. 제공자 항목 (`type: "provider"` 또는 생략):
    - 2. `provider`: API 제공자 ID (`openai`, `anthropic`, `google`/`gemini`, `groq` 등).
    - 3. `model`: 모델 ID 오버라이드 (이미지에는 필수; 오디오 제공자의 기본값은 `gpt-4o-mini-transcribe`/`whisper-large-v3-turbo`, 비디오의 기본값은 `gemini-3-flash-preview`).
    - 4. `profile` / `preferredProfile`: 인증 프로필 선택.
  - 5. CLI 항목 (`type: "cli"`):
    - 6. `command`: 실행할 실행 파일.
    - 7. `args`: 템플릿 인자 (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` 등 지원).
  - 8. `capabilities`: 공유 항목을 제한하기 위한 선택적 목록 (`image`, `audio`, `video`). 9. 생략 시 기본값: `openai`/`anthropic`/`minimax` → 이미지, `google` → 이미지+오디오+비디오, `groq` → 오디오.
  - 10. `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`는 항목별로 오버라이드할 수 있습니다.

11. 구성된 모델이 없으면 (또는 `enabled: false`인 경우) 이해(understanding)는 건너뛰며, 모델은 여전히 원본 첨부 파일을 받습니다.

12. 제공자 인증은 표준 모델 인증 순서를 따릅니다(인증 프로필, `OPENAI_API_KEY`/`GROQ_API_KEY`/`GEMINI_API_KEY` 같은 환경 변수, 또는 `models.providers.*.apiKey`).

Example:

```json5
13. {
  tools: {
    media: {
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

14. `agents.defaults.subagents`는 하위 에이전트 기본값을 구성합니다:

- 15. `model`: 생성된 하위 에이전트의 기본 모델(문자열 또는 `{ primary, fallbacks }`). 16. 생략 시, 하위 에이전트는 에이전트별 또는 호출별로 오버라이드되지 않는 한 호출자의 모델을 상속합니다.
- 17. `maxConcurrent`: 동시 실행 가능한 하위 에이전트 최대 수(기본값 1).
- 18. `archiveAfterMinutes`: N분 후 하위 에이전트 세션을 자동 아카이브(기본값 60; 비활성화하려면 `0` 설정).
- 19. 하위 에이전트별 도구 정책: `tools.subagents.tools.allow` / `tools.subagents.tools.deny` (deny가 우선).

20. `tools.profile`은 `tools.allow`/`tools.deny` 이전에 적용되는 **기본 도구 허용 목록**을 설정합니다:

- `minimal`: `session_status` 만
- `coding`: `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`
- `messaging`: `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status`
- `full`: 제한 없음 (미설정과 동일)

에이전트별 재정의: `agents.list[].tools.profile`.

예시 (기본은 메시징 전용, Slack + Discord 도구도 허용):

```json5
{
  tools: {
    profile: "messaging",
    allow: ["slack", "discord"],
  },
}
```

예시 (코딩 프로필이지만 exec/process 는 전역 차단):

```json5
{
  tools: {
    profile: "coding",
    deny: ["group:runtime"],
  },
}
```

21. `tools.byProvider`를 사용하면 특정 제공자(또는 단일 `provider/model`)에 대해 도구를 **추가로 제한**할 수 있습니다.
    에이전트별 재정의: `agents.list[].tools.byProvider`.

22. 적용 순서: 기본 프로필 → 제공자 프로필 → 허용/거부 정책.
23. 제공자 키는 `provider`(예: `google-antigravity`) 또는 `provider/model`
    (예: `openai/gpt-5.2`)을 허용합니다.

예시 (전역 코딩 프로필은 유지하되, Google Antigravity 에는 최소 도구만):

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
    },
  },
}
```

24. 예시(제공자/모델별 허용 목록):

```json5
{
  tools: {
    allow: ["group:fs", "group:runtime", "sessions_list"],
    byProvider: {
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

25. `tools.allow` / `tools.deny`는 전역 도구 허용/거부 정책을 구성합니다(deny가 우선).
26. 매칭은 대소문자를 구분하지 않으며 `*` 와일드카드를 지원합니다(`"*"`는 모든 도구를 의미).
27. 이는 Docker 샌드박스가 **꺼져 있어도** 적용됩니다.

28. 예시(브라우저/캔버스 전체 비활성화):

```json5
29. {
  tools: { deny: ["browser", "canvas"] },
}
```

30. 도구 그룹(단축키)은 **전역** 및 **에이전트별** 도구 정책에서 작동합니다:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:web`: `web_search`, `web_fetch`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:openclaw`: 모든 기본 제공 OpenClaw 도구(프로바이더 플러그인은 제외)

31. `tools.elevated`는 상승된(호스트) 실행 접근을 제어합니다:

- 32. `enabled`: 상승 모드 허용(기본값 true).
- 33. `allowFrom`: 채널별 허용 목록(비어 있으면 비활성화).
  - 34. `whatsapp`: E.164 번호.
  - 35. `telegram`: 채팅 ID 또는 사용자 이름.
  - 36. `discord`: 사용자 ID 또는 사용자 이름(생략 시 `channels.discord.dm.allowFrom`로 대체).
  - 37. `signal`: E.164 번호.
  - 38. `imessage`: 핸들이나 채팅 ID.
  - 39. `webchat`: 세션 ID 또는 사용자 이름.

Example:

```json5
40. {
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

에이전트별 오버라이드(추가 제한):

```json5
{
  agents: {
    list: [
      {
        id: "family",
        tools: {
          elevated: { enabled: false },
        },
      },
    ],
  },
}
```

노트:

- `tools.elevated`는 전역 기준선입니다. `agents.list[].tools.elevated`는 추가로 제한만 할 수 있습니다(둘 다 허용해야 함).
- `/elevated on|off|ask|full`은 세션 키별로 상태를 저장하며, 인라인 지시문은 단일 메시지에만 적용됩니다.
- Elevated `exec`는 호스트에서 실행되며 샌드박싱을 우회합니다.
- 도구 정책은 여전히 적용되며, `exec`가 거부되면 elevated는 사용할 수 없습니다.

`agents.defaults.maxConcurrent`는 세션 전반에 걸쳐 병렬로 실행될 수 있는 내장 에이전트 실행의 최대 개수를 설정합니다. 각 세션은 여전히 직렬화됩니다(세션 키당 한 번에 하나의 실행). 기본값: 1.

### `agents.defaults.sandbox`

내장 에이전트를 위한 선택적 **Docker 샌드박싱**. 호스트 시스템에 접근하지 못하도록 메인 세션이 아닌 세션을 대상으로 합니다.

자세한 내용: [Sandboxing](/gateway/sandboxing)

기본값(활성화된 경우):

- 범위: `"agent"` (에이전트당 하나의 컨테이너 + 워크스페이스)
- Debian bookworm-slim 기반 이미지
- 에이전트 워크스페이스 접근: `workspaceAccess: "none"` (기본값)
  - `"none"`: `~/.openclaw/sandboxes` 아래에 범위별 샌드박스 워크스페이스를 사용
- `"ro"`: 샌드박스 워크스페이스를 `/workspace`에 유지하고, 에이전트 워크스페이스를 `/agent`에 읽기 전용으로 마운트(`write`/`edit`/`apply_patch` 비활성화)
  - `"rw"`: 에이전트 워크스페이스를 `/workspace`에 읽기/쓰기 마운트
- 자동 정리: 유휴 > 24시간 또는 수명 > 7일
- 도구 정책: `exec`, `process`, `read`, `write`, `edit`, `apply_patch`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`만 허용(거부 우선)
  - `tools.sandbox.tools`로 설정하고, 에이전트별로 `agents.list[].tools.sandbox.tools`에서 오버라이드
  - 샌드박스 정책에서 지원되는 도구 그룹 약어: `group:runtime`, `group:fs`, `group:sessions`, `group:memory` ([Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#tool-groups-shorthands) 참조)
- 선택적 샌드박스 브라우저(Chromium + CDP, noVNC 관찰자)
- 하드닝 옵션: `network`, `user`, `pidsLimit`, `memory`, `cpus`, `ulimits`, `seccompProfile`, `apparmorProfile`

경고: `scope: "shared"`는 컨테이너와 워크스페이스를 공유함을 의미합니다. 세션 간 격리가 없습니다. 세션별 격리를 위해 `scope: "session"`을 사용하세요.

레거시: `perSession`은 여전히 지원됩니다(`true` → `scope: "session"`, `false` → `scope: "shared"`).

`setupCommand`는 컨테이너가 생성된 후 **한 번만** 실행됩니다(컨테이너 내부에서 `sh -lc`를 통해).
패키지 설치를 위해서는 네트워크 송신, 쓰기 가능한 루트 파일시스템, 그리고 root 사용자를 보장하세요.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared (agent is default)
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          // Per-agent override (multi-agent): agents.list[].sandbox.docker.*
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/var/run/docker.sock:/var/run/docker.sock", "/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          containerPrefix: "openclaw-sbx-browser-",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          allowedControlUrls: ["http://10.0.0.42:18791"],
          allowedControlHosts: ["browser.lab.local", "10.0.0.42"],
          allowedControlPorts: [18791],
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24, // 0 disables idle pruning
          maxAgeDays: 7, // 0 disables max-age pruning
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

다음으로 기본 샌드박스 이미지를 한 번 빌드합니다:

```bash
scripts/sandbox-setup.sh
```

참고: 샌드박스 컨테이너의 기본값은 `network: "none"`입니다. 에이전트에 아웃바운드 접근이 필요하면 `agents.defaults.sandbox.docker.network`를 `"bridge"`(또는 사용자 지정 네트워크)로 설정하세요.

참고: 인바운드 첨부 파일은 활성 워크스페이스의 `media/inbound/*`에 스테이징됩니다. `workspaceAccess: "rw"`인 경우, 이는 파일이 에이전트 워크스페이스에 기록됨을 의미합니다.

참고: `docker.binds`는 추가 호스트 디렉터리를 마운트하며, 전역 및 에이전트별 바인드는 병합됩니다.

선택적 브라우저 이미지를 다음으로 빌드합니다:

```bash
scripts/sandbox-browser-setup.sh
```

`agents.defaults.sandbox.browser.enabled=true`인 경우, 브라우저 도구는 샌드박스된 Chromium 인스턴스(CDP)를 사용합니다. noVNC가 활성화되어 있으면(headless=false일 때 기본값), noVNC URL이 시스템 프롬프트에 주입되어 에이전트가 이를 참조할 수 있습니다.
이는 메인 설정에서 `browser.enabled`를 필요로 하지 않습니다. 샌드박스 제어 URL은 세션별로 주입됩니다.

`agents.defaults.sandbox.browser.allowHostControl`(기본값: false)은 샌드박스된 세션이 브라우저 도구를 통해 **호스트** 브라우저 제어 서버를 명시적으로 대상으로 지정할 수 있게 합니다 (`target: "host"`). 엄격한 샌드박스 격리를 원한다면 이 옵션을 끄세요.

원격 제어를 위한 허용 목록:

- `allowedControlUrls`: `target: "custom"`에 대해 허용되는 정확한 제어 URL.
- `allowedControlHosts`: 허용되는 호스트명(호스트명만, 포트 없음).
- `allowedControlPorts`: 허용되는 포트(기본값: http=80, https=443).
  기본값: 모든 허용 목록이 설정되지 않음(제한 없음). `allowHostControl`의 기본값은 false입니다.

### `models` (커스텀 프로바이더 + 기본 URL)

OpenClaw는 **pi-coding-agent** 모델 카탈로그를 사용합니다. 커스텀 프로바이더를 추가할 수 있습니다 (LiteLLM, 로컬 OpenAI 호환 서버, Anthropic 프록시 등)
`~/.openclaw/agents/<agentId>/agent/models.json`에 작성하거나 OpenClaw 설정의 `models.providers` 아래에 동일한 스키마를 정의하여 추가할 수 있습니다.

프로바이더별 개요 + 예제: [/concepts/model-providers](/concepts/model-providers).

- `models.providers`가 존재하면, OpenClaw는 시작 시 `models.json`을 `~/.openclaw/agents/<agentId>/agent/`에 작성/병합합니다:
- 기본 동작: **병합** (기존 프로바이더를 유지하고 이름 기준으로 덮어씀)

파일 내용을 덮어쓰려면 `models.mode: "replace"`로 설정하세요

```json5
`agents.defaults.model.primary`(프로바이더/모델)를 통해 모델을 선택합니다.
```

### {&#xA;agents: {&#xA;defaults: {&#xA;model: { primary: "custom-proxy/llama-3.1-8b" },&#xA;models: {&#xA;"custom-proxy/llama-3.1-8b": {},&#xA;},&#xA;},&#xA;},&#xA;models: {&#xA;mode: "merge",&#xA;providers: {&#xA;"custom-proxy": {&#xA;baseUrl: "http://localhost:4000/v1",&#xA;apiKey: "LITELLM_KEY",&#xA;api: "openai-completions",&#xA;models: [&#xA;{&#xA;id: "llama-3.1-8b",&#xA;name: "Llama 3.1 8B",&#xA;reasoning: false,&#xA;input: ["text"],&#xA;cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },&#xA;contextWindow: 128000,&#xA;maxTokens: 32000,&#xA;},&#xA;],&#xA;},&#xA;},&#xA;},&#xA;}

OpenCode Zen (멀티 모델 프록시) OpenCode Zen은 모델별 엔드포인트를 제공하는 멀티 모델 게이트웨이입니다.

노트:

- OpenClaw는 pi-ai의 내장 `opencode` 프로바이더를 사용합니다. [https://opencode.ai/auth](https://opencode.ai/auth)에서 `OPENCODE_API_KEY`(또는 `OPENCODE_ZEN_API_KEY`)를 설정하세요.
- 모델 참조는 `opencode/<modelId>` 형식을 사용합니다 (예: `opencode/claude-opus-4-6`).
- `agents.defaults.models`를 통해 허용 목록을 활성화한 경우, 사용할 각 모델을 추가하세요.

```json5
바로가기: `openclaw onboard --auth-choice opencode-zen`.
```

### {&#xA;agents: {&#xA;defaults: {&#xA;model: { primary: "opencode/claude-opus-4-6" },&#xA;models: { "opencode/claude-opus-4-6": { alias: "Opus" } },&#xA;},&#xA;},&#xA;}

Z.AI (GLM-4.7) — 프로바이더 별칭 지원 Z.AI 모델은 내장 `zai` 프로바이더를 통해 사용할 수 있습니다.

환경에 `ZAI_API_KEY`를 설정하고 프로바이더/모델 형식으로 모델을 참조하세요.

```json5
바로가기: `openclaw onboard --auth-choice zai-api-key`.
```

노트:

- {
  agents: {
  defaults: {
  model: { primary: "zai/glm-4.7" },
  models: { "zai/glm-4.7": {} },
  },
  },
  }
- `z.ai/*`와 `z-ai/*`는 허용되는 별칭이며 `zai/*`로 정규화됩니다.
- `ZAI_API_KEY`가 없으면 `zai/*`에 대한 요청은 런타임에 인증 오류로 실패합니다.
- 예시 오류: `No API key found for provider "zai".` Z.AI의 일반 API 엔드포인트는 `https://api.z.ai/api/paas/v4`입니다.
  GLM 코딩 요청은 전용 코딩 엔드포인트 `https://api.z.ai/api/coding/paas/v4`를 사용합니다. 내장 `zai` 프로바이더는 코딩 엔드포인트를 사용합니다.
- 일반 엔드포인트가 필요하다면, 위의 커스텀 프로바이더 섹션을 참고하여 `models.providers`에서 기본 URL을 오버라이드하는 커스텀 프로바이더를 정의하세요.

### Moonshot AI (Kimi)

문서/설정에는 가짜 플레이스홀더를 사용하세요. 실제 API 키를 커밋하지 마세요.

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

노트:

- 환경 변수에 `MOONSHOT_API_KEY`를 설정하거나 `openclaw onboard --auth-choice moonshot-api-key`를 사용하세요.
- 모델 참조: `moonshot/kimi-k2.5`.
- 중국 엔드포인트의 경우 다음 중 하나를 선택하세요:
  - `openclaw onboard --auth-choice moonshot-api-key-cn`를 실행하세요(마법사가 `https://api.moonshot.cn/v1`를 설정합니다), 또는
  - `models.providers.moonshot`에 `baseUrl: "https://api.moonshot.cn/v1"`를 수동으로 설정하세요.

### Kimi Coding

Moonshot AI의 Kimi Coding 엔드포인트(Anthropic 호환, 내장 프로바이더)를 사용하세요:

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

노트:

- 환경 변수에 `KIMI_API_KEY`를 설정하거나 `openclaw onboard --auth-choice kimi-code-api-key`를 사용하세요.
- 모델 참조: `kimi-coding/k2p5`.

### Synthetic (Anthropic 호환)

Synthetic의 Anthropic 호환 엔드포인트를 사용하세요:

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

노트:

- `SYNTHETIC_API_KEY`를 설정하거나 `openclaw onboard --auth-choice synthetic-api-key`를 사용하세요.
- 모델 참조: `synthetic/hf:MiniMaxAI/MiniMax-M2.1`.
- Anthropic 클라이언트가 `/v1`를 자동으로 추가하므로 Base URL에는 `/v1`를 포함하지 않아야 합니다.

### 로컬 모델(LM Studio) — 권장 설정

현재 로컬 가이드는 [/gateway/local-models](/gateway/local-models)를 참고하세요. 요약(TL;DR): 충분한 하드웨어에서 LM Studio Responses API로 MiniMax M2.1을 실행하고, 장애 대비용으로 호스티드 모델은 병합 상태로 유지하세요.

### MiniMax M2.1

LM Studio 없이 MiniMax M2.1을 직접 사용:

```json5
{
  agent: {
    model: { primary: "minimax/MiniMax-M2.1" },
    models: {
      "anthropic/claude-opus-4-6": { alias: "Opus" },
      "minimax/MiniMax-M2.1": { alias: "Minimax" },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            // Pricing: update in models.json if you need exact cost tracking.
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

노트:

- 환경 변수 `MINIMAX_API_KEY`를 설정하거나 `openclaw onboard --auth-choice minimax-api`를 사용하세요.
- 사용 가능한 모델: `MiniMax-M2.1`(기본값).
- 정확한 비용 추적이 필요하면 `models.json`에서 가격을 업데이트하세요.

### Cerebras (GLM 4.6 / 4.7)

Cerebras의 OpenAI 호환 엔드포인트를 사용하세요:

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

노트:

- Cerebras에는 `cerebras/zai-glm-4.7`를 사용하고, Z.AI 직접 연결에는 `zai/glm-4.7`를 사용하세요.
- 환경 또는 설정에서 `CEREBRAS_API_KEY`를 설정하세요.

노트:

- 지원되는 API: `openai-completions`, `openai-responses`, `anthropic-messages`,
  `google-generative-ai`
- 커스텀 인증이 필요한 경우 `authHeader: true` + `headers`를 사용하세요.
- `models.json`을 다른 위치에 저장하려면 `OPENCLAW_AGENT_DIR`(또는 `PI_CODING_AGENT_DIR`)로 에이전트 설정 루트를 오버라이드하세요(기본값: `~/.openclaw/agents/main/agent`).

### `세션`

세션 범위, 리셋 정책, 리셋 트리거, 그리고 세션 스토어가 기록되는 위치를 제어합니다.

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main",
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    // Default is already per-agent under ~/.openclaw/agents/<agentId>/sessions/sessions.json
    // You can override with {agentId} templating:
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    // Direct chats collapse to agent:<agentId>:<mainKey> (default: "main").
    mainKey: "main",
    agentToAgent: {
      // Max ping-pong reply turns between requester/target (0–5).
      maxPingPongTurns: 5,
    },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

필드:

- `mainKey`: 다이렉트 채팅 버킷 키(기본값: `"main"`). `agentId`를 변경하지 않고 기본 DM 스레드를 “이름 변경”하고 싶을 때 유용합니다.
  - 샌드박스 참고: `agents.defaults.sandbox.mode: "non-main"`은 메인 세션을 감지하기 위해 이 키를 사용합니다. `mainKey`와 일치하지 않는 모든 세션 키(그룹/채널)는 샌드박스 처리됩니다.
- `dmScope`: DM 세션을 그룹화하는 방식(기본값: `"main"`).
  - `main`: 모든 DM이 연속성을 위해 메인 세션을 공유합니다.
  - `per-peer`: 채널 전반에서 발신자 ID별로 DM을 분리합니다.
  - `per-channel-peer`: isolate DMs per channel + sender (recommended for multi-user inboxes).
  - `per-account-channel-peer`: isolate DMs per account + channel + sender (recommended for multi-account inboxes).
  - Secure DM mode (recommended): set `session.dmScope: "per-channel-peer"` when multiple people can DM the bot (shared inboxes, multi-person allowlists, or `dmPolicy: "open"`).
- `identityLinks`: map canonical ids to provider-prefixed peers so the same person shares a DM session across channels when using `per-peer`, `per-channel-peer`, or `per-account-channel-peer`.
  - Example: `alice: ["telegram:123456789", "discord:987654321012345678"]`.
- `reset`: primary reset policy. Defaults to daily resets at 4:00 AM local time on the gateway host.
  - `mode`: `daily` or `idle` (default: `daily` when `reset` is present).
  - `atHour`: local hour (0-23) for the daily reset boundary.
  - `idleMinutes`: sliding idle window in minutes. 일일 + 유휴가 모두 구성된 경우 먼저 만료되는 쪽이 우선합니다.
- `resetByType`: per-session overrides for `direct`, `group`, and `thread`. Legacy `dm` key is accepted as an alias for `direct`.
  - If you only set legacy `session.idleMinutes` without any `reset`/`resetByType`, OpenClaw stays in idle-only mode for backward compatibility.
- `heartbeatIdleMinutes`: optional idle override for heartbeat checks (daily reset still applies when enabled).
- `agentToAgent.maxPingPongTurns`: max reply-back turns between requester/target (0–5, default 5).
- `sendPolicy.default`: `allow` or `deny` fallback when no rule matches.
- `sendPolicy.rules[]`: match by `channel`, `chatType` (`direct|group|room`), or `keyPrefix` (e.g. `cron:`). First deny wins; otherwise allow.

### `skills` (skills config)

Controls bundled allowlist, install preferences, extra skill folders, and per-skill
overrides. Applies to **bundled** skills and `~/.openclaw/skills` (workspace skills
still win on name conflicts).

필드:

- `allowBundled`: **번들된** skills 전용 선택적 허용 목록입니다. If set, only those
  bundled skills are eligible (managed/workspace skills unaffected).
- `load.extraDirs`: 스캔할 추가 skill 디렉토리(가장 낮은 우선순위).
- `install.preferBrew`: 가능할 경우 brew 설치 관리자를 선호합니다(기본값: true).
- `install.nodeManager`: node installer preference (`npm` | `pnpm` | `yarn`, default: npm).
- `entries.<skillKey>`: per-skill config overrides.

Skill 별 필드:

- `enabled`: 번들되었거나 설치되어 있더라도 skill 을 비활성화하려면 `false`를 설정합니다.
- `env`: 에이전트 실행 시 주입되는 환경 변수(이미 설정되어 있지 않은 경우에만).
- `apiKey`: optional convenience for skills that declare a primary env var (e.g. `nano-banana-pro` → `GEMINI_API_KEY`).

Example:

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm",
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

### `plugins` (extensions)

Controls plugin discovery, allow/deny, and per-plugin config. Plugins are loaded
from `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, plus any
`plugins.load.paths` entries. **Config changes require a gateway restart.**
See [/plugin](/tools/plugin) for full usage.

필드:

- `enabled`: master toggle for plugin loading (default: true).
- `allow`: optional allowlist of plugin ids; when set, only listed plugins load.
- `deny`: optional denylist of plugin ids (deny wins).
- `load.paths`: extra plugin files or directories to load (absolute or `~`).
- `entries.<pluginId>`: per-plugin overrides.
  - `enabled`: set `false` to disable.
  - `config`: plugin-specific config object (validated by the plugin if provided).

Example:

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: {
          provider: "twilio",
        },
      },
    },
  },
}
```

### `browser` (openclaw-managed browser)

OpenClaw can start a **dedicated, isolated** Chrome/Brave/Edge/Chromium instance for openclaw and expose a small loopback control service.
Profiles can point at a **remote** Chromium-based browser via `profiles.<name>.cdpUrl`. Remote
profiles are attach-only (start/stop/reset are disabled).

`browser.cdpUrl` remains for legacy single-profile configs and as the base
scheme/host for profiles that only set `cdpPort`.

기본값:

- enabled: `true`
- evaluateEnabled: `true` (set `false` to disable `act:evaluate` and `wait --fn`)
- control service: loopback only (port derived from `gateway.port`, default `18791`)
- CDP URL: `http://127.0.0.1:18792` (control service + 1, legacy single-profile)
- profile color: `#FF4500` (lobster-orange)
- Note: the control server is started by the running gateway (OpenClaw.app menubar, or `openclaw gateway`).
- Auto-detect order: default browser if Chromium-based; otherwise Chrome → Brave → Edge → Chromium → Chrome Canary.

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    // cdpUrl: "http://127.0.0.1:18792", // legacy single-profile override
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // Advanced:
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false, // set true when tunneling a remote CDP to localhost
  },
}
```

### `ui` (Appearance)

Optional accent color used by the native apps for UI chrome (e.g. Talk Mode bubble tint).

If unset, clients fall back to a muted light-blue.

```json5
{
  ui: {
    seamColor: "#FF4500", // hex (RRGGBB or #RRGGBB)
    // Optional: Control UI assistant identity override.
    // If unset, the Control UI uses the active agent identity (config or IDENTITY.md).
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, or image URL/data URI
    },
  },
}
```

### `gateway` (Gateway server mode + bind)

Use `gateway.mode` to explicitly declare whether this machine should run the Gateway.

기본값:

- mode: **unset** (treated as “do not auto-start”)
- bind: `loopback`
- port: `18789` (single port for WS + HTTP)

```json5
{
  gateway: {
    mode: "local", // or "remote"
    port: 18789, // WS + HTTP multiplex
    bind: "loopback",
    // controlUi: { enabled: true, basePath: "/openclaw" }
    // auth: { mode: "token", token: "your-token" } // token gates WS + Control UI access
    // tailscale: { mode: "off" | "serve" | "funnel" }
  },
}
```

Control UI base path:

- `gateway.controlUi.basePath` sets the URL prefix where the Control UI is served.
- Examples: `"/ui"`, `"/openclaw"`, `"/apps/openclaw"`.
- Default: root (`/`) (unchanged).
- `gateway.controlUi.root` sets the filesystem root for Control UI assets (default: `dist/control-ui`).
- `gateway.controlUi.allowInsecureAuth` allows token-only auth for the Control UI when
  device identity is omitted (typically over HTTP). Default: `false`. Prefer HTTPS
  (Tailscale Serve) or `127.0.0.1`.
- `gateway.controlUi.dangerouslyDisableDeviceAuth` disables device identity checks for the
  Control UI (token/password only). Default: `false`. Break-glass only.

관련 문서:

- [컨트롤 UI](/web/control-ui)
- [Web overview](/web)
- [Tailscale](/gateway/tailscale)
- [원격 액세스](/gateway/remote)

신뢰된 프록시:

- `gateway.trustedProxies`: list of reverse proxy IPs that terminate TLS in front of the Gateway.
- When a connection comes from one of these IPs, OpenClaw uses `x-forwarded-for` (or `x-real-ip`) to determine the client IP for local pairing checks and HTTP auth/local checks.
- Only list proxies you fully control, and ensure they **overwrite** incoming `x-forwarded-for`.

참고:

- `openclaw gateway` refuses to start unless `gateway.mode` is set to `local` (or you pass the override flag).
- `gateway.port` controls the single multiplexed port used for WebSocket + HTTP (control UI, hooks, A2UI).
- OpenAI Chat Completions endpoint: **disabled by default**; enable with `gateway.http.endpoints.chatCompletions.enabled: true`.
- Precedence: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > default `18789`.
- Gateway auth is required by default (token/password or Tailscale Serve identity). Non-loopback binds require a shared token/password.
- The onboarding wizard generates a gateway token by default (even on loopback).
- `gateway.remote.token` is **only** for remote CLI calls; it does not enable local gateway auth. `gateway.token` is ignored.

Auth and Tailscale:

- `gateway.auth.mode` sets the handshake requirements (`token` or `password`). When unset, token auth is assumed.
- `gateway.auth.token` stores the shared token for token auth (used by the CLI on the same machine).
- When `gateway.auth.mode` is set, only that method is accepted (plus optional Tailscale headers).
- `gateway.auth.password` can be set here, or via `OPENCLAW_GATEWAY_PASSWORD` (recommended).
- `gateway.auth.allowTailscale` allows Tailscale Serve identity headers
  (`tailscale-user-login`) to satisfy auth when the request arrives on loopback
  with `x-forwarded-for`, `x-forwarded-proto`, and `x-forwarded-host`. OpenClaw
  verifies the identity by resolving the `x-forwarded-for` address via
  `tailscale whois` before accepting it. When `true`, Serve requests do not need
  a token/password; set `false` to require explicit credentials. Defaults to
  `true` when `tailscale.mode = "serve"` and auth mode is not `password`.
- `gateway.tailscale.mode: "serve"` uses Tailscale Serve (tailnet only, loopback bind).
- `gateway.tailscale.mode: "funnel"` exposes the dashboard publicly; requires auth.
- `gateway.tailscale.resetOnExit` resets Serve/Funnel config on shutdown.

Remote client defaults (CLI):

- `gateway.remote.url` sets the default Gateway WebSocket URL for CLI calls when `gateway.mode = "remote"`.
- `gateway.remote.transport` selects the macOS remote transport (`ssh` default, `direct` for ws/wss). When `direct`, `gateway.remote.url` must be `ws://` or `wss://`. `ws://host` defaults to port `18789`.
- `gateway.remote.token` supplies the token for remote calls (leave unset for no auth).
- `gateway.remote.password` supplies the password for remote calls (leave unset for no auth).

macOS app behavior:

- OpenClaw.app watches `~/.openclaw/openclaw.json` and switches modes live when `gateway.mode` or `gateway.remote.url` changes.
- If `gateway.mode` is unset but `gateway.remote.url` is set, the macOS app treats it as remote mode.
- When you change connection mode in the macOS app, it writes `gateway.mode` (and `gateway.remote.url` + `gateway.remote.transport` in remote mode) back to the config file.

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Direct transport example (macOS app):

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      transport: "direct",
      url: "wss://gateway.example.ts.net",
      token: "your-token",
    },
  },
}
```

### `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

모드:

- `hybrid` (default): hot-apply safe changes; restart the Gateway for critical changes.
- `hot`: only apply hot-safe changes; log when a restart is required.
- `restart`: restart the Gateway on any config change.
- `off`: disable hot reload.

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

#### 핫 리로드 매트릭스 (파일 + 영향)

감시되는 파일:

- `~/.openclaw/openclaw.json` (또는 `OPENCLAW_CONFIG_PATH`)

핫 적용됨 (전체 게이트웨이 재시작 없음):

- `hooks` (웹훅 인증/경로/매핑) + `hooks.gmail` (Gmail 워처 재시작)
- `browser` (브라우저 제어 서버 재시작)
- `cron` (크론 서비스 재시작 + 동시성 업데이트)
- `agents.defaults.heartbeat` (하트비트 러너 재시작)
- `web` (WhatsApp 웹 채널 재시작)
- `telegram`, `discord`, `signal`, `imessage` (채널 재시작)
- `agent`, `models`, `routing`, `messages`, `session`, `whatsapp`, `logging`, `skills`, `ui`, `talk`, `identity`, `wizard` (동적 읽기)

전체 게이트웨이 재시작 필요:

- `gateway` (포트/바인드/인증/제어 UI/tailscale)
- `bridge` (레거시)
- `디스커버리`
- `canvasHost`
- `플러그인`
- 알 수 없거나 지원되지 않는 설정 경로 (안전을 위해 기본적으로 재시작)

### 멀티 인스턴스 격리

하나의 호스트에서 여러 게이트웨이를 실행하려면(중복성 또는 구조용 봇을 위해), 인스턴스별 상태 + 설정을 격리하고 고유한 포트를 사용하세요:

- `OPENCLAW_CONFIG_PATH` (인스턴스별 설정)
- `OPENCLAW_STATE_DIR` (세션/자격 증명)
- `agents.defaults.workspace` (메모리)
- `gateway.port` (인스턴스별 고유)

35. 편의 플래그 (CLI):

- 36. `openclaw --dev …` → `~/.openclaw-dev`를 사용하고 기본값 `19001`에서 포트를 이동
- 37. `openclaw --profile <name> …` → `~/.openclaw-<name>`를 사용 (포트는 설정/환경 변수/플래그로 지정)

38. 파생된 포트 매핑(gateway/browser/canvas)은 [Gateway runbook](/gateway)을 참조하세요.
39. 브라우저/CDP 포트 격리 상세 내용은 [Multiple gateways](/gateway/multiple-gateways)를 참조하세요.

Example:

```bash
40. OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

### `hooks` (게이트웨이 웹훅)

게이트웨이 HTTP 서버에 간단한 HTTP 웹훅 엔드포인트를 활성화합니다.

기본값:

- enabled: `false`
- path: `/hooks`
- maxBodyBytes: `262144` (256 KB)

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        wakeMode: "now",
        name: "Gmail",
        sessionKey: "hook:gmail:{{messages[0].id}}",
        messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
        deliver: true,
        channel: "last",
        model: "openai/gpt-5.2-mini",
      },
    ],
  },
}
```

요청에는 훅 토큰이 포함되어야 합니다:

- `Authorization: Bearer <token>` **또는**
- `x-openclaw-token: <token>`

엔드포인트:

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds?` }\` 반환
- `POST /hooks/<name>` → `hooks.mappings`를 통해 해석됨

13. `/hooks/agent`는 항상 메인 세션에 요약을 게시하며(선택적으로 `wakeMode: "now"`를 통해 즉시 하트비트를 트리거할 수 있음).

14. 매핑 참고 사항:

- 15. `match.path`는 `/hooks` 뒤의 하위 경로와 일치합니다 (예: `/hooks/gmail` → `gmail`).
- 16. `match.source`는 페이로드 필드와 일치합니다 (예: `{ source: "gmail" }`) 따라서 범용 `/hooks/ingest` 경로를 사용할 수 있습니다.
- 17. `{{messages[0].subject}}`와 같은 템플릿은 페이로드에서 값을 읽습니다.
- 18. `transform`은 훅 액션을 반환하는 JS/TS 모듈을 가리킬 수 있습니다.
- 19. `deliver: true`는 최종 응답을 채널로 전송합니다; `channel`의 기본값은 `last`입니다 (WhatsApp으로 폴백).
- 20. 이전 전달 경로가 없는 경우 `channel` + `to`를 명시적으로 설정하세요 (Telegram/Discord/Google Chat/Slack/Signal/iMessage/MS Teams에는 필수).
- 21. `model`은 이 훅 실행에 사용할 LLM을 재정의합니다 (`provider/model` 또는 별칭; `agents.defaults.models`가 설정된 경우 허용 목록에 있어야 함).

22. Gmail 헬퍼 설정 (`openclaw webhooks gmail setup` / `run`에서 사용됨):

```json5
23. {
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },

      // 선택 사항: Gmail 훅 처리를 위해 더 저렴한 모델 사용
      // 인증/레이트리밋/타임아웃 시 agents.defaults.model.fallbacks, 그 다음 primary로 폴백
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      // 선택 사항: Gmail 훅의 기본 thinking 레벨
      thinking: "off",
    },
  },
}
```

24. Gmail 훅에 대한 모델 재정의:

- 25. `hooks.gmail.model`은 Gmail 훅 처리에 사용할 모델을 지정합니다 (기본값: 세션 primary).
- 26. `agents.defaults.models`의 `provider/model` 참조 또는 별칭을 허용합니다.
- 27. 인증/레이트리밋/타임아웃 시 `agents.defaults.model.fallbacks`, 그 다음 `agents.defaults.model.primary`로 폴백합니다.
- 28. `agents.defaults.models`가 설정된 경우, 훅 모델을 허용 목록에 포함하세요.
- 29. 시작 시 구성된 모델이 모델 카탈로그 또는 허용 목록에 없으면 경고합니다.
- 30. `hooks.gmail.thinking`은 Gmail 훅의 기본 thinking 레벨을 설정하며, 훅별 `thinking`에 의해 재정의됩니다.

31. 게이트웨이 자동 시작:

- 32. `hooks.enabled=true`이고 `hooks.gmail.account`가 설정되어 있으면, 게이트웨이는 부팅 시 `gog gmail watch serve`를 시작하고 워치를 자동 갱신합니다.
- 33. 자동 시작을 비활성화하려면 `OPENCLAW_SKIP_GMAIL_WATCHER=1`을 설정하세요 (수동 실행용).
- 34. 게이트웨이와 함께 별도의 `gog gmail watch serve`를 실행하지 마세요; `listen tcp 127.0.0.1:8788: bind: address already in use` 오류로 실패합니다.

35. 참고: `tailscale.mode`가 켜져 있으면, Tailscale이 `/gmail-pubsub`을 올바르게 프록시할 수 있도록 OpenClaw는 기본적으로 `serve.path`를 `/`로 설정합니다 (설정된 경로 접두사를 제거함).
36. 백엔드가 접두사가 붙은 경로를 받아야 하는 경우, `hooks.gmail.tailscale.target`을 전체 URL로 설정하고 `serve.path`를 맞추세요.

### 37. `canvasHost` (LAN/테일넷 Canvas 파일 서버 + 라이브 리로드)

38. 게이트웨이는 HTML/CSS/JS 디렉터리를 HTTP로 제공하여 iOS/Android 노드가 간단히 `canvas.navigate`로 접근할 수 있습니다.

39. 기본 루트: `~/.openclaw/workspace/canvas`  
    기본 포트: `18793` (openclaw 브라우저 CDP 포트 `18792`와의 충돌을 피하기 위해 선택됨)  
    서버는 노드가 접근할 수 있도록 **게이트웨이 바인드 호스트**(LAN 또는 Tailnet)에서 수신합니다.

40. 서버:

- `canvasHost.root` 아래의 파일을 제공합니다
- 제공되는 HTML에 아주 작은 라이브 리로드 클라이언트를 주입합니다
- 디렉터리를 감시하고 `/__openclaw__/ws`의 WebSocket 엔드포인트를 통해 리로드를 브로드캐스트합니다
- 디렉터리가 비어 있을 때 시작용 `index.html`을 자동 생성합니다 (즉시 무언가가 보이도록)
- `/__openclaw__/a2ui/`에서 A2UI도 제공하며 노드에 `canvasHostUrl`로 광고됩니다
  (Canvas/A2UI에 대해 노드가 항상 사용)

디렉터리가 크거나 `EMFILE`에 도달하면 라이브 리로드(및 파일 감시)를 비활성화합니다:

- config: `canvasHost: { liveReload: false }`

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    port: 18793,
    liveReload: true,
  },
}
```

`canvasHost.*` 변경 사항은 게이트웨이 재시작이 필요합니다 (config reload 시 재시작됨).

비활성화하려면:

- config: `canvasHost: { enabled: false }`
- env: `OPENCLAW_SKIP_CANVAS_HOST=1`

### `bridge` (레거시 TCP 브리지, 제거됨)

현재 빌드에는 더 이상 TCP 브리지 리스너가 포함되지 않으며 `bridge.*` 설정 키는 무시됩니다.
노드는 게이트웨이 WebSocket을 통해 연결합니다. 이 섹션은 역사적 참고용으로 유지됩니다.

레거시 동작:

- 게이트웨이는 노드(iOS/Android)를 위해 간단한 TCP 브리지를 노출할 수 있었으며, 일반적으로 포트 `18790`을 사용했습니다.

기본값:

- enabled: `true`
- port: `18790`
- bind: `lan` (`0.0.0.0`에 바인드)

바인드 모드:

- `lan`: `0.0.0.0` (LAN/Wi‑Fi 및 Tailscale을 포함한 모든 인터페이스에서 접근 가능)
- `tailnet`: 머신의 Tailscale IP에만 바인드 (Vienna ⇄ London에 권장)
- `loopback`: `127.0.0.1` (로컬 전용)
- `auto`: tailnet IP가 있으면 우선, 없으면 `lan`

TLS:

- `bridge.tls.enabled`: 브리지 연결에 TLS 활성화 (활성화 시 TLS 전용).
- `bridge.tls.autoGenerate`: 인증서/키가 없을 때 자체 서명 인증서를 생성 (기본값: true).
- `bridge.tls.certPath` / `bridge.tls.keyPath`: 브리지 인증서 + 개인 키의 PEM 경로.
- `bridge.tls.caPath`: 선택적 PEM CA 번들 (커스텀 루트 또는 향후 mTLS).

TLS가 활성화되면 게이트웨이는 노드가 인증서를 고정(pin)할 수 있도록 discovery TXT 레코드에 `bridgeTls=1`과 `bridgeTlsSha256`을 광고합니다. 수동 연결은 아직 지문이 저장되지 않은 경우 최초 신뢰(TOFU)를 사용합니다.
자동 생성 인증서는 PATH에 `openssl`이 필요합니다; 생성에 실패하면 브리지는 시작되지 않습니다.

```json5
{
  bridge: {
    enabled: true,
    port: 18790,
    bind: "tailnet",
    tls: {
      enabled: true,
      // 생략 시 ~/.openclaw/bridge/tls/bridge-{cert,key}.pem 사용
      // certPath: "~/.openclaw/bridge/tls/bridge-cert.pem",
      // keyPath: "~/.openclaw/bridge/tls/bridge-key.pem"
    },
  },
}
```

### `discovery.mdns` (Bonjour / mDNS 브로드캐스트 모드)

LAN mDNS 디스커버리 브로드캐스트(`_openclaw-gw._tcp`)를 제어합니다.

- `minimal` (기본값): TXT 레코드에서 `cliPath` + `sshPort`를 생략
- `full`: TXT 레코드에 `cliPath` + `sshPort` 포함
- `off`: mDNS 브로드캐스트를 완전히 비활성화
- 호스트명: 기본값은 `openclaw` (`openclaw.local`을 광고) `OPENCLAW_MDNS_HOSTNAME`으로 재정의합니다.

```json5
{
  discovery: { mdns: { mode: "minimal" } },
}
```

### `discovery.wideArea` (광역 Bonjour / 유니캐스트 DNS‑SD)

활성화되면 Gateway는 구성된 검색 도메인(예: `openclaw.internal.`)을 사용하여 `~/.openclaw/dns/` 아래에 `_openclaw-gw._tcp`에 대한 유니캐스트 DNS‑SD 존을 작성합니다.

iOS/Android가 네트워크를 넘어(Vienna ⇄ London) 검색할 수 있도록 다음과 함께 사용하세요:

- 선택한 도메인을 제공하는 게이트웨이 호스트의 DNS 서버 (CoreDNS 권장)
- 클라이언트가 해당 도메인을 게이트웨이 DNS 서버를 통해 해석하도록 하는 Tailscale **분할 DNS**

일회성 설정 도우미 (게이트웨이 호스트):

```bash
openclaw dns setup --apply
```

```json5
{
  discovery: { wideArea: { enabled: true } },
}
```

## 미디어 모델 템플릿 변수

템플릿 자리표시는 `tools.media.*.models[].args` 및 `tools.media.models[].args`(그리고 향후 템플릿이 적용되는 모든 인자 필드)에서 확장됩니다.

| 변수                 | 설명                                                                           |          |         |            |       |        |          |         |         |    |
| ------------------ | ---------------------------------------------------------------------------- | -------- | ------- | ---------- | ----- | ------ | -------- | ------- | ------- | -- |
| `{{Body}}`         | 전체 인바운드 메시지 본문                                                               |          |         |            |       |        |          |         |         |    |
| `{{RawBody}}`      | 원본 인바운드 메시지 본문(히스토리/발신자 래퍼 없음; 명령 파싱에 최적)                 |          |         |            |       |        |          |         |         |    |
| `{{BodyStripped}}` | 그룹 멘션이 제거된 본문(에이전트에 가장 적합한 기본값)                           |          |         |            |       |        |          |         |         |    |
| `{{From}}`         | 발신자 식별자(WhatsApp의 경우 E.164; 채널별로 다를 수 있음) |          |         |            |       |        |          |         |         |    |
| `{{To}}`           | 수신자 식별자                                                                      |          |         |            |       |        |          |         |         |    |
| `{{MessageSid}}`   | 채널 메시지 ID(사용 가능한 경우)                                      |          |         |            |       |        |          |         |         |    |
| `{{SessionId}}`    | 현재 세션 UUID                                                                   |          |         |            |       |        |          |         |         |    |
| `{{IsNewSession}}` | 새 세션이 생성되었을 때 `"true"`                                                       |          |         |            |       |        |          |         |         |    |
| `{{MediaUrl}}`     | 인바운드 미디어 의사-URL(있는 경우)                                    |          |         |            |       |        |          |         |         |    |
| `{{MediaPath}}`    | 로컬 미디어 경로(다운로드된 경우)                                       |          |         |            |       |        |          |         |         |    |
| `{{MediaType}}`    | 미디어 유형(image/audio/document/…)                            |          |         |            |       |        |          |         |         |    |
| `{{Transcript}}`   | Audio transcript (when enabled)                           |          |         |            |       |        |          |         |         |    |
| `{{Prompt}}`       | Resolved media prompt for CLI entries                                        |          |         |            |       |        |          |         |         |    |
| `{{MaxChars}}`     | Resolved max output chars for CLI entries                                    |          |         |            |       |        |          |         |         |    |
| `{{ChatType}}`     | `"direct"` or `"group"`                                                      |          |         |            |       |        |          |         |         |    |
| `{{GroupSubject}}` | Group subject (best effort)                               |          |         |            |       |        |          |         |         |    |
| `{{GroupMembers}}` | Group members preview (best effort)                       |          |         |            |       |        |          |         |         |    |
| `{{SenderName}}`   | Sender display name (best effort)                         |          |         |            |       |        |          |         |         |    |
| `{{SenderE164}}`   | Sender phone number (best effort)                         |          |         |            |       |        |          |         |         |    |
| `{{Provider}}`     | Provider hint (whatsapp                                   | telegram | discord | googlechat | slack | signal | imessage | msteams | webchat | …) |

## Cron (Gateway scheduler)

Cron is a Gateway-owned scheduler for wakeups and scheduled jobs. See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}
```

---

_다음: [Agent Runtime](/concepts/agent)_ 🦞
