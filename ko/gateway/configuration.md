---
summary: "Configuration 개요: 일반적인 작업, 빠른 설정, 전체 참조 링크"
read_when:
  - OpenClaw를 처음 설정하기
  - 일반적인 configuration 패턴 찾기
  - 특정 config 섹션으로 이동하기
title: "구성"
---

# 구성 🔧

OpenClaw 는 `~/.openclaw/openclaw.json` 에서 선택적 **JSON5** 구성을 읽습니다 (주석 + 후행 콤마 허용).

파일이 없으면 OpenClaw 는 비교적 안전한 기본값 (임베디드 Pi 에이전트 + 발신자별 세션 + 워크스페이스 `~/.openclaw/workspace`) 을 사용합니다. 일반적으로 다음과 같은 경우에만 구성이 필요합니다.

- 채널을 연결하고 봇에 메시지를 보낼 수 있는 사용자를 제어하기
- 모델, 도구, 샌드박싱 또는 자동화(cron, hooks) 설정하기
- 세션, 미디어, 네트워킹 또는 UI 조정하기

사용 가능한 모든 필드는 [full reference](/gateway/configuration-reference)를 참고하세요.

<Tip>
`openclaw --profile <name> …` → `~/.openclaw-<name>`를 사용 (포트는 설정/환경 변수/플래그로 지정)
</Tip>

## 최소 구성 (권장 시작점)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Config 편집하기

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # 전체 설정 마법사
openclaw configure     # config 마법사
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    Open [http://127.0.0.1:18789](http://127.0.0.1:18789)을 열고 **Config** 탭을 사용하세요.
    Control UI 는 이 스키마로부터 폼을 렌더링하며, 탈출구로 **Raw JSON** 편집기를 제공합니다.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (또는 `OPENCLAW_CONFIG_PATH`) Gateway는 파일을 감시하고 변경 사항을 자동으로 적용합니다 (자세한 내용은 [hot reload](#config-hot-reload) 참고).
  
</Tab>
</Tabs>

## 엄격한 구성 검증

<Warning>
OpenClaw 는 스키마와 완전히 일치하는 구성만 허용합니다. 알 수 없는 키, 잘못된 타입, 유효하지 않은 값이 있으면 안전을 위해 Gateway(게이트웨이)가 **시작을 거부**합니다. 루트 레벨에서의 유일한 예외는 `$schema`(문자열)이며, 이를 통해 에디터가 JSON Schema 메타데이터를 연결할 수 있습니다.
</Warning>

검증에 실패하면:

- Gateway 가 부팅되지 않습니다.
- 진단 명령만 허용됩니다 (예: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- 정확한 문제를 확인하려면 `openclaw doctor` 를 실행하십시오.
- 마이그레이션/복구를 적용하려면 `openclaw doctor --fix` (또는 `--yes`) 를 실행하십시오.

## 공통 옵션

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">각 채널은 `channels.` 아래에 고유한 config 섹션을 가집니다.<provider>` 아래에 위치합니다. 설정 단계는 해당 채널 전용 페이지를 참고하세요:

    ```
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

  
</Accordion>

  <Accordion title="Choose and configure models">기본 모델과 선택적 대체(fallback) 모델을 설정하세요:

    ```
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

  
</Accordion>

  <Accordion title="Control who can message the bot">DM 접근은 채널별로 `dmPolicy`를 통해 제어됩니다:

    ```
    - `"pairing"` (기본값): 알 수 없는 발신자는 승인을 위한 1회용 페어링 코드를 받습니다
    - `"allowlist"`: `allowFrom`(또는 페어링된 허용 저장소)에 있는 발신자만 허용
    - `"open"`: 모든 수신 DM 허용 (`allowFrom: ["*"]` 필요)
    - `"disabled"`: 모든 DM 무시
    
    그룹의 경우 `groupPolicy` + `groupAllowFrom` 또는 채널별 allowlist를 사용하세요.
    
    채널별 세부 사항은 [full reference](/gateway/configuration-reference#dm-and-group-access)를 참고하세요.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    그룹 메시지는 기본적으로 **멘션 필요** (메타데이터 멘션 또는 정규식 패턴) 입니다. `agents.defaults.subagents`는 하위 에이전트 기본값을 구성합니다:

    ```
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

  
</Accordion>

  <Accordion title="Configure sessions and resets">세션은 대화의 연속성과 격리를 제어합니다:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // 다중 사용자 환경에 권장
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (공유) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - 범위 지정, ID 연결, 전송 정책은 [Session Management](/concepts/session)를 참고하세요.
    - 모든 필드는 [full reference](/gateway/configuration-reference#session)를 참고하세요.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">에이전트 세션을 격리된 Docker 컨테이너에서 실행:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">```json5
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

    ```
    - `every`: 기간 문자열 (`30m`, `2h`). 비활성화하려면 `0m`으로 설정하세요.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - 전체 가이드는 [Heartbeat](/gateway/heartbeat)를 참고하세요.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">게이트웨이 HTTP 서버에 간단한 HTTP 웹훅 엔드포인트를 활성화합니다.

    ```
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

  
</Accordion>

  <Accordion title="Configure multi-agent routing">하나의 Gateway 안에서 여러 개의 격리된 에이전트(분리된 워크스페이스, `agentDir`, 세션)를 실행합니다.

    ```
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

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">`$include` 지시어를 사용하여 구성을 여러 파일로 분할할 수 있습니다. 이는 다음에 유용합니다.

    ```
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

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

### 리로드 모드

| 모드:                   | 동작 방식                                                                                                                                                          |
| ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (기본값) | 안전한 변경 사항은 즉시 핫 적용됩니다. 중요한 변경 사항은 자동으로 재시작됩니다.                                                                                 |
| …)                                    | `hot`: only apply hot-safe changes; log when a restart is required. 재시작이 필요한 경우 경고 로그를 남깁니다 — 재시작은 사용자가 수행합니다. |
| **`restart`**                         | `restart`: restart the Gateway on any config change.                                                                           |
| `{{Prompt}}`                          | 파일 감시를 비활성화합니다. 변경 사항은 다음 수동 재시작 시 적용됩니다.                                                                                      |

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

### 어떤 항목이 핫 적용되고 어떤 항목이 재시작이 필요한가

대부분의 필드는 다운타임 없이 핫 적용됩니다. `hybrid` 모드에서는 재시작이 필요한 변경 사항도 자동으로 처리됩니다.

| 카테고리                                 | 필드:                                                     | 재시작 필요?                     |
| ------------------------------------ | ----------------------------------------------------------------------- | --------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — 모든 내장 및 확장 채널       | 아니요                         |
| 에이전트 및 모델                            | `agent`, `agents`, `models`, `routing`                                  | 아니요                         |
| 자동화                                  | `hooks`, `cron`, `agent.heartbeat`                                      | 아니요                         |
| messages                             | `messages.queue`                                                        | 아니요                         |
| 도구 및 미디어                             | `tools`, `browser`, `skills`, `audio`, `talk`                           | 아니요                         |
| 스키마 + UI 힌트                          | `ui`, `logging`, `identity`, `bindings`                                 | 아니요                         |
| Gateway 서버                           | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP) | **인라인 치환:** |
| 인프라                                  | `discovery`, `canvasHost`, `plugins`                                    | **멘션 유형:**  |

<Note>
`gateway.reload` 및 `gateway.remote`는 예외입니다 — 이를 변경해도 **재시작이 트리거되지 않습니다**.
</Note>

## 부분 업데이트 (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">`config.apply` 을 사용하여 전체 구성을 검증 + 기록하고 Gateway 를 한 단계로 재시작하십시오.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">`config.patch` 을 사용하면 관련 없는 키를 덮어쓰지 않고 기존 구성에 부분 업데이트를 병합할 수 있습니다. JSON merge patch 의미를 적용합니다.

    ````
    - 객체는 재귀적으로 병합됩니다
    - `null`은 키를 삭제합니다
    - 배열은 교체됩니다
    
    Params:
    
    - `raw` (string) — 변경할 키만 포함한 JSON5
    - `baseHash` (required) — `config.get`에서 가져온 config 해시
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply`와 동일
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## 구성에서 환경 변수 치환

OpenClaw는 부모 프로세스와 다음 위치에서 env 변수를 읽습니다:

- 현재 작업 디렉토리의 `.env` (존재 시)
- `~/.openclaw/.env` 의 전역 대체 `.env` (일명 `$OPENCLAW_STATE_DIR/.env`)

두 `.env` 파일 모두 기존 환경 변수를 덮어쓰지 않습니다. config에서 인라인 env 변수도 설정할 수 있습니다:

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

<Accordion title="Shell env import (optional)">편의 기능 옵트인: 활성화되어 있고 예상되는 키가 아직 설정되지 않았다면,  
OpenClaw 는 로그인 셸을 실행하여 누락된 예상 키만 가져옵니다 (절대 덮어쓰지 않음).

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

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">어떤 구성 문자열 값에서도 `${VAR_NAME}` 문법을 사용하여  
환경 변수를 직접 참조할 수 있습니다.

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
- 누락되었거나 비어 있는 변수는 로드 시점에 오류를 발생시킵니다.
- 리터럴 `${VAR}` 을 출력하려면 `$${VAR}` 로 이스케이프하십시오.
- `$include` 와 함께 동작합니다 (포함된 파일에도 치환 적용).
- Inline substitution: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

전체 우선순위와 소스는 [/environment](/help/environment) 를 참고하십시오.

## 전체 참조

**구성이 처음이신가요?** 자세한 설명이 포함된 전체 예제는 [Configuration Examples](/gateway/configuration-examples) 가이드를 참고하십시오!

---

프로바이더별 개요 + 예제: [/concepts/model-providers](/concepts/model-providers).

