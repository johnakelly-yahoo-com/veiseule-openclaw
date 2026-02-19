---
title: "구성 참조"
description: "~/.openclaw/openclaw.json에 대한 전체 필드별 참조"
---

# 구성 참조

`~/.openclaw/openclaw.json`에서 사용 가능한 모든 필드입니다. 작업 중심 개요는 [Configuration](/gateway/configuration)을 참조하세요.

구성 형식은 **JSON5**입니다(주석 + 후행 쉼표 허용). 모든 필드는 선택 사항입니다 — 생략하면 OpenClaw는 안전한 기본값을 사용합니다.

---

## 채널

각 채널은 해당 구성 섹션이 존재하면 자동으로 시작됩니다(`enabled: false`가 아닌 한).

### DM 및 그룹 접근

모든 채널은 DM 정책과 그룹 정책을 지원합니다:

| DM 정책                              | 동작                                                             |
| ---------------------------------- | -------------------------------------------------------------- |
| `pairing` (기본값) | 알 수 없는 발신자에게는 일회성 페어링 코드가 제공되며, 소유자가 승인해야 합니다. |
| `allowlist`                        | `allowFrom`에 포함된 발신자(또는 페어링된 allow 스토어)만 허용 |
| `open`                             | 모든 인바운드 DM 허용 (`allowFrom: ["*"]` 필요)       |
| `disabled`                         | 모든 인바운드 DM 무시                                                  |

| 그룹 정책                                | 동작                                                   |
| ------------------------------------ | ---------------------------------------------------- |
| `allowlist` (기본값) | 설정된 allowlist와 일치하는 그룹만 허용                           |
| `open`                               | 그룹 allowlist를 우회 (멘션 게이팅은 계속 적용됨) |
| `disabled`                           | 모든 그룹/룸 메시지 차단                                       |

<Note>
`channels.defaults.groupPolicy`는 provider의 `groupPolicy`가 설정되지 않은 경우 기본값을 설정합니다.
페어링 코드는 1시간 후 만료됩니다. 대기 중인 DM 페어링 요청은 **채널당 3개**로 제한됩니다.
Slack/Discord에는 특별한 폴백이 있습니다: provider 섹션이 완전히 누락된 경우, 런타임 그룹 정책이 `open`으로 해석될 수 있습니다(시작 시 경고와 함께).
</Note>

### WhatsApp

WhatsApp은 gateway의 웹 채널(Baileys Web)을 통해 실행됩니다. 연결된 세션이 존재하면 자동으로 시작됩니다.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // 파란 체크 표시 (self-chat 모드에서는 false)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
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

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- 아웃바운드 명령은 `default` 계정이 있으면 이를 기본으로 사용하며, 없으면 설정된 계정 id 중 첫 번째(정렬 기준)를 사용합니다.
- 레거시 단일 계정 Baileys 인증 디렉터리는 `openclaw doctor`에 의해 `whatsapp/default`로 마이그레이션됩니다.
- 계정별 오버라이드: `channels.whatsapp.accounts.<id>``.sendReadReceipts`, `channels.whatsapp.accounts.<id>``.dmPolicy`, `channels.whatsapp.accounts.<id>``.allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "답변은 간결하게 유지하세요.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "주제를 벗어나지 마세요.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git 백업" },
        { command: "generate", description: "이미지 생성" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot 토큰: 기본 계정의 경우 `channels.telegram.botToken` 또는 `channels.telegram.tokenFile`, 폴백으로 `TELEGRAM_BOT_TOKEN`을 사용합니다.
- `configWrites: false`는 Telegram에서 시작된 설정 쓰기(슈퍼그룹 ID 마이그레이션, `/config set|unset`)를 차단합니다.
- Telegram 스트림 프리뷰는 `sendMessage` + `editMessageText`를 사용합니다(개인 및 그룹 채팅 모두에서 동작).
- 재시도 정책: [Retry policy](/concepts/retry)를 참조하세요.

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
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
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "짧은 답변만 제공하세요.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- 토큰: `channels.discord.token`, 기본 계정의 경우 폴백으로 `DISCORD_BOT_TOKEN`을 사용합니다.
- 전달 대상에는 `user:<id>`(DM) 또는 `channel:<id>`(guild 채널)을 사용하세요; 숫자 ID만 단독으로 사용하는 것은 허용되지 않습니다.
- Guild 슬러그는 소문자로 작성하며 공백은 `-`로 대체합니다; 채널 키는 슬러그화된 이름을 사용합니다(`#` 제외). 길드 ID 사용을 권장합니다.
- 기본적으로 봇이 작성한 메시지는 무시됩니다. `allowBots: true`로 설정하면 이를 활성화할 수 있습니다(단, 자신의 메시지는 여전히 필터링됨).
- `maxLinesPerMessage`(기본값 17)는 2000자를 넘지 않더라도 긴 메시지를 분할합니다.
- `channels.discord.ui.components.accentColor`는 Discord components v2 컨테이너의 강조 색상을 설정합니다.

**반응 알림 모드:** `off`(없음), `own`(봇의 메시지, 기본값), `all`(모든 메시지), `allowlist`(`guilds.<id>.users`에 있는 사용자로부터의 모든 메시지).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
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

- 서비스 계정 JSON: 인라인(`serviceAccount`) 또는 파일 기반(`serviceAccountFile`).
- 환경 변수 대체값: `GOOGLE_CHAT_SERVICE_ACCOUNT` 또는 `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- 전달 대상에는 `spaces/<spaceId>` 또는 `users/<userId|email>`을 사용하세요.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
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
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
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

- **Socket mode**는 `botToken`과 `appToken`이 모두 필요합니다(기본 계정 환경 변수 대체값으로 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`).
- **HTTP mode**는 `botToken`과 `signingSecret`(루트 또는 계정별)이 필요합니다.
- `configWrites: false`는 Slack에서 시작된 설정 쓰기를 차단합니다.
- 전달 대상에는 `user:<id>`(DM) 또는 `channel:<id>`를 사용하세요.

**반응 알림 모드:** `off`, `own`(기본값), `all`, `allowlist`(`reactionAllowlist`에서 지정).

**스레드 세션 격리:** `thread.historyScope`는 스레드별(기본값) 또는 채널 전체에서 공유로 설정할 수 있습니다. `thread.inheritParent`는 새 스레드에 부모 채널의 대화 기록을 복사합니다.

| 작업 그룹      | 기본값  | 비고               |
| ---------- | ---- | ---------------- |
| reactions  | 활성화됨 | 반응 추가 + 반응 목록 조회 |
| messages   | 활성화됨 | 읽기/전송/수정/삭제      |
| pins       | 활성화됨 | 고정/고정 해제/목록 조회   |
| memberInfo | 활성화됨 | 멤버 정보            |
| emojiList  | 활성화됨 | 커스텀 이모지 목록       |

### Mattermost

Mattermost는 플러그인으로 제공됩니다: `openclaw plugins install @openclaw/mattermost`.

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

채팅 모드: `oncall` (@-멘션 시 응답, 기본값), `onmessage` (모든 메시지), `onchar` (트리거 접두사로 시작하는 메시지).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**리액션 알림 모드:** `off`, `own` (기본값), `all`, `allowlist` (`reactionAllowlist` 기준).

### iMessage

OpenClaw는 `imsg rpc`를 실행합니다 (stdio를 통한 JSON-RPC). 데몬이나 포트가 필요하지 않습니다.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Messages DB에 대한 전체 디스크 접근 권한이 필요합니다.
- `chat_id:<id>` 대상을 사용하는 것을 권장합니다. 채팅 목록을 확인하려면 `imsg chats --limit 20`을 사용하세요.
- `cliPath`는 SSH 래퍼를 가리킬 수 있으며, 첨부 파일을 SCP로 가져오려면 `remoteHost`를 설정하세요.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### 다중 계정 (모든 채널)

채널별로 여러 계정을 실행할 수 있습니다 (각 계정은 고유한 `accountId`를 가짐):

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

- `accountId`를 생략하면 `default`가 사용됩니다 (CLI + 라우팅).
- Env 토큰은 **default** 계정에만 적용됩니다.
- 기본 채널 설정은 계정별로 재정의하지 않는 한 모든 계정에 적용됩니다.
- `bindings[].match.accountId`를 사용하여 각 계정을 서로 다른 에이전트로 라우팅하세요.

### 그룹 채팅 멘션 게이팅

그룹 메시지는 기본적으로 **멘션 필요**로 설정됩니다 (메타데이터 멘션 또는 정규식 패턴). WhatsApp, Telegram, Discord, Google Chat 및 iMessage 그룹 채팅에 적용됩니다.

**멘션 유형:**

- **메타데이터 멘션**: 플랫폼의 기본 @-멘션. WhatsApp 셀프 채팅 모드에서는 무시됩니다.
- **텍스트 패턴**: `agents.list[].groupChat.mentionPatterns`에 정의된 정규식 패턴. 항상 확인됩니다.
- 멘션 감지가 가능한 경우(기본 멘션 또는 하나 이상의 패턴이 있을 때)에만 멘션 게이팅이 적용됩니다.

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

`messages.groupChat.historyLimit`는 전역 기본값을 설정합니다. 채널은 `channels.<channel>`로 재정의할 수 있습니다.historyLimit`(또는 계정별).`0\`으로 설정하면 비활성화됩니다.

#### DM 기록 제한

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

해결 순서: DM별 재정의 → 제공자 기본값 → 제한 없음 (모두 유지).

지원 대상: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### 셀프 채팅 모드

`allowFrom`에 자신의 번호를 포함하면 셀프 채팅 모드가 활성화됩니다 (기본 @-멘션은 무시하고 텍스트 패턴에만 응답):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### 명령어 (채팅 명령 처리)

```json5
{
  commands: {
    native: "auto", // register native commands when supported
    text: true, // parse /commands in chat messages
    bash: false, // allow ! (alias: /bash)
    bashForegroundMs: 2000,
    config: false, // allow /config
    debug: false, // allow /debug
    restart: false, // allow /restart + gateway restart tool
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- 텍스트 명령어는 반드시 앞에 `/`가 붙은 **단독** 메시지여야 합니다.
- `native: "auto"`는 Discord/Telegram에서는 네이티브 명령어를 활성화하고, Slack에서는 비활성화합니다.
- 채널별로 재정의: `channels.discord.commands.native` (bool 또는 `"auto"`). `false`로 설정하면 이전에 등록된 명령어를 삭제합니다.
- `channels.telegram.customCommands`는 Telegram 봇 메뉴 항목을 추가합니다.
- `bash: true`는 `!`를 활성화합니다. `<cmd>`를 호스트 셸에서 실행합니다. `tools.elevated.enabled`가 필요하며, 발신자가 `tools.elevated.allowFrom.<channel>`에 포함되어 있어야 합니다.\`.
- `config: true`는 `/config`를 활성화합니다 (`openclaw.json` 읽기/쓰기).
- `channels.<provider>`.configWrites\`는 채널별로 설정 변경을 제어합니다 (기본값: true).
- `allowFrom`은 제공자별로 설정됩니다. 설정되면 이는 **유일한** 인증 소스가 됩니다 (채널 허용 목록/페어링 및 `useAccessGroups`는 무시됨).
- `useAccessGroups: false`로 설정하면 `allowFrom`이 설정되지 않은 경우 명령어가 액세스 그룹 정책을 우회할 수 있습니다.

</Accordion>

---

## 에이전트 기본값

### `agents.defaults.workspace`

기본값: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

시스템 프롬프트의 Runtime 줄에 표시되는 선택적 리포지토리 루트입니다. 설정되지 않은 경우, OpenClaw는 workspace에서 위로 탐색하여 자동으로 감지합니다.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

workspace 부트스트랩 파일(`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`)의 자동 생성을 비활성화합니다.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

잘리기 전 각 workspace 부트스트랩 파일의 최대 문자 수입니다. 기본값: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

모든 workspace 부트스트랩 파일에 걸쳐 주입되는 총 최대 문자 수입니다. 기본값: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

시스템 프롬프트 컨텍스트용 시간대(메시지 타임스탬프가 아님). 호스트 시간대로 대체됩니다.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

시스템 프롬프트에서 사용할 시간 형식. 기본값: `auto` (OS 설정을 따름).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

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
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: 형식은 `provider/model` (예: `anthropic/claude-opus-4-6`). provider를 생략하면 OpenClaw는 `anthropic`으로 간주합니다(사용 중단 예정).
- `models`: `/model`에 대한 구성된 모델 카탈로그 및 허용 목록입니다. 각 항목에는 `alias`(단축어)와 `params`(provider별 설정: `temperature`, `maxTokens`)를 포함할 수 있습니다.
- `imageModel`: 기본 모델이 이미지 입력을 지원하지 않는 경우에만 사용됩니다.
- `maxConcurrent`: 세션 전체에서 동시에 실행할 수 있는 최대 에이전트 수(각 세션은 여전히 직렬 실행). 기본값: 1.

**내장 alias 단축어** (`agents.defaults.models`에 모델이 있을 때만 적용됨):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

사용자가 구성한 alias는 항상 기본값보다 우선합니다.

Z.AI GLM-4.x 모델은 `--thinking off`를 설정하거나 `agents.defaults.models["zai/<model>"].params.thinking`을 직접 정의하지 않는 한 자동으로 thinking 모드를 활성화합니다.

### `agents.defaults.cliBackends`

텍스트 전용 대체 실행(도구 호출 없음)을 위한 선택적 CLI 백엔드. API 제공자가 실패할 때 백업으로 유용합니다.

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

- CLI 백엔드는 텍스트 우선이며, 도구는 항상 비활성화됩니다.
- `sessionArg`가 설정된 경우 세션이 지원됩니다.
- `imageArg`가 파일 경로를 허용하는 경우 이미지 전달이 지원됩니다.

### `agents.defaults.heartbeat`

주기적으로 하트비트를 실행합니다.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: 기간 문자열(ms/s/m/h). 기본값: `30m`.
- 에이전트별 설정: `agents.list[].heartbeat`를 설정합니다. 어떤 에이전트든 `heartbeat`를 정의하면, **해당 에이전트들만** 하트비트를 실행합니다.
- 하트비트는 전체 에이전트 턴을 실행합니다 — 간격이 짧을수록 더 많은 토큰을 소모합니다.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
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

- `mode`: `default` 또는 `safeguard` (긴 히스토리를 위한 청크 단위 요약). [Compaction](/concepts/compaction)을 참고하세요.
- `memoryFlush`: 자동 컴팩션 전에 지속 가능한 메모리를 저장하기 위해 실행되는 무음(agentic) 턴입니다. 워크스페이스가 읽기 전용인 경우 건너뜁니다.

### `agents.defaults.contextPruning`

LLM에 전송하기 전에 메모리 내 컨텍스트에서 **오래된 도구 결과**를 정리합니다. 디스크에 저장된 세션 히스토리는 **수정하지 않습니다**.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // duration (ms/s/m/h), default unit: minutes
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"`은 정리(pruning) 패스를 활성화합니다.
- `ttl`은 마지막 캐시 접근 이후 정리를 다시 실행할 수 있는 시점을 제어합니다.
- 정리는 먼저 과도하게 큰 도구 결과를 소프트 트림한 뒤, 필요할 경우 더 오래된 도구 결과를 하드 클리어합니다.

**Soft-trim**은 앞부분과 뒷부분을 유지하고 가운데에 `...`를 삽입합니다.

**Hard-clear**는 전체 도구 결과를 플레이스홀더로 대체합니다.

참고:

- 이미지 블록은 절대 잘리거나 삭제되지 않습니다.
- 비율은 정확한 토큰 수가 아닌 문자 수 기준(대략적)입니다.
- `keepLastAssistants`보다 assistant 메시지 수가 적으면 정리를 건너뜁니다.

</Accordion>

동작 세부 사항은 [Session Pruning](/concepts/session-pruning)을 참고하세요.

### 블록 스트리밍

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (use minMs/maxMs)
    },
  },
}
```

- Telegram이 아닌 채널에서는 블록 응답을 활성화하려면 `*.blockStreaming: true`를 명시적으로 설정해야 합니다.
- 채널별 재정의: `channels.<channel>``.blockStreamingCoalesce` (및 계정별 변형). Signal/Slack/Discord/Google Chat의 기본값은 `minChars: 1500`입니다.
- `humanDelay`: 블록 응답 사이에 무작위 지연을 추가합니다. `natural` = 800–2500ms. 에이전트별 재정의: `agents.list[].humanDelay`.

동작 및 청크 분할 세부 사항은 [Streaming](/concepts/streaming)을 참고하세요.

### 타이핑 표시기

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- 기본값: 직접 채팅/멘션에는 `instant`, 멘션이 없는 그룹 채팅에는 `message`.
- 세션별 재정의: `session.typingMode`, `session.typingIntervalSeconds`.

[Typing Indicators](/concepts/typing-indicators)를 참고하세요.

### `agents.defaults.sandbox`

내장 에이전트를 위한 선택적 **Docker 샌드박싱**. 전체 가이드는 [Sandboxing](/gateway/sandboxing)을 참고하세요.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
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
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
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

<Accordion title="Sandbox details">

**워크스페이스 접근:**

- `none`: `~/.openclaw/sandboxes` 아래에 범위별 샌드박스 워크스페이스 생성
- `ro`: 샌드박스 워크스페이스는 `/workspace`, 에이전트 워크스페이스는 `/agent`에 읽기 전용으로 마운트
- `rw`: 에이전트 워크스페이스를 `/workspace`에 읽기/쓰기 가능으로 마운트

**범위:**

- `session`: 세션별 컨테이너 + 워크스페이스
- `agent`: 에이전트별 하나의 컨테이너 + 워크스페이스 (기본값)
- `shared`: 컨테이너와 워크스페이스를 공유 (세션 간 격리 없음)

\*\*`setupCommand`\*\*는 컨테이너 생성 후 한 번 실행됩니다 (`sh -lc`를 통해). 네트워크 아웃바운드, 쓰기 가능한 루트, root 사용자 권한이 필요합니다.

**컨테이너의 기본값은 `network: "none"`** — 에이전트에 아웃바운드 접근이 필요하면 `"bridge"`로 설정하세요.

**인바운드 첨부 파일**은 활성 워크스페이스의 `media/inbound/*`에 저장됩니다.

\*\*`docker.binds`\*\*는 추가 호스트 디렉터리를 마운트합니다. 전역 및 에이전트별 binds는 병합됩니다.

**샌드박스 브라우저** (`sandbox.browser.enabled`): 컨테이너에서 실행되는 Chromium + CDP. noVNC URL이 시스템 프롬프트에 주입됩니다. 메인 설정에서 `browser.enabled`를 요구하지 않습니다.

- `allowHostControl: false` (기본값)는 샌드박스 세션이 호스트 브라우저를 대상으로 제어하는 것을 차단합니다.
- `sandbox.browser.binds`는 추가 호스트 디렉터리를 샌드박스 브라우저 컨테이너에만 마운트합니다. 설정된 경우(`[]` 포함), 브라우저 컨테이너에 대해 `docker.binds`를 대체합니다.

</Accordion>

이미지 빌드:

```bash
scripts/sandbox-setup.sh           # main sandbox image
scripts/sandbox-browser-setup.sh   # optional browser image
```

### `agents.list` (에이전트별 재정의)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // or { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: 안정적인 에이전트 id (필수).
- `default`: 여러 개가 설정된 경우 첫 번째가 적용됩니다 (경고 로그 기록). 설정된 항목이 없으면 목록의 첫 번째 항목이 기본값이 됩니다.
- `model`: 문자열 형식은 `primary`만 재정의합니다. 객체 형식 `{ primary, fallbacks }`는 둘 다 재정의합니다 (`[]`는 전역 fallbacks 비활성화).
- `identity.avatar`: 워크스페이스 기준 경로, `http(s)` URL 또는 `data:` URI.
- `identity`는 기본값을 파생합니다: `ackReaction`은 `emoji`에서, `mentionPatterns`는 `name`/`emoji`에서 생성됩니다.
- `subagents.allowAgents`: `sessions_spawn`에 허용되는 에이전트 id의 allowlist (`["*"]` = 모든 에이전트, 기본값: 동일 에이전트만).

---

## 멀티 에이전트 라우팅

하나의 Gateway에서 여러 개의 격리된 에이전트를 실행합니다. [Multi-Agent](/concepts/multi-agent)를 참고하세요.

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

### 바인딩 match 필드

- `match.channel` (필수)
- `match.accountId` (선택 사항; `*` = 모든 계정; 생략 시 = 기본 계정)
- `match.peer` (선택 사항; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (선택 사항; 채널별)

**결정적 매칭 순서:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (정확히 일치, peer/guild/team 없음)
5. `match.accountId: "*"` (채널 전체)
6. 기본 agent

각 단계 내에서는 먼저 일치하는 `bindings` 항목이 적용됩니다.

### Agent별 접근 프로필

<Accordion title="Full access (no sandbox)">

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

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
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

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
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

</Accordion>

우선순위에 대한 자세한 내용은 [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools)를 참조하세요.

---

## 세션

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // legacy (runtime always uses "main")
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: DM을 그룹화하는 방식입니다.
  - `main`: 모든 DM이 메인 세션을 공유합니다.
  - `per-peer`: 채널 전반에서 발신자 id 기준으로 분리합니다.
  - `per-channel-peer`: 채널 + 발신자별로 분리합니다 (멀티 사용자 인박스에 권장).
  - `per-account-channel-peer`: 계정 + 채널 + 발신자별로 분리합니다 (멀티 계정에 권장).
- **`identityLinks`**: 채널 간 세션 공유를 위해 표준 id를 provider 접두사가 붙은 peer와 매핑합니다.
- **`reset`**: 기본 리셋 정책입니다. `daily`는 로컬 시간 `atHour`에 리셋되며, `idle`은 `idleMinutes` 이후 리셋됩니다. 둘 다 설정된 경우, 먼저 만료되는 설정이 적용됩니다.
- **`resetByType`**: 유형별 재정의 설정 (`direct`, `group`, `thread`). 레거시 `dm`은 `direct`의 별칭으로 허용됩니다.
- **`mainKey`**: 레거시 필드입니다. 현재 런타임은 메인 direct-chat 버킷에 항상 `"main"`을 사용합니다.
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`, 레거시 `dm` 별칭 포함), `keyPrefix`, 또는 `rawKeyPrefix`로 매칭합니다. 첫 번째 deny가 우선 적용됩니다.
- **`maintenance`**: `warn`은 세션 제거 시 활성 세션에 경고를 표시하며, `enforce`는 정리(pruning) 및 로테이션을 적용합니다.

</Accordion>

---

## 메시지

```json5
{
  messages: {
    responsePrefix: "🦞", // or "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 disables
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### 응답 접두사

채널/계정별 오버라이드: `channels.<channel>``.responsePrefix`, `channels.<channel>``.accounts.<id>``.responsePrefix`.

해결 순서 (가장 구체적인 설정이 우선): account → channel → global. `""`로 설정하면 비활성화되며 상속이 중단됩니다. `"auto"`는 `[{identity.name}]`을(를) 생성합니다.

**템플릿 변수:**

| 변수                | 설명               | 예시                                |
| ----------------- | ---------------- | --------------------------------- |
| `{model}`         | 짧은 모델 이름         | `claude-opus-4-6`                 |
| `{modelFull}`     | 전체 모델 식별자        | `anthropic/claude-opus-4-6`       |
| `{provider}`      | 제공자 이름           | `anthropic`                       |
| `{thinkingLevel}` | 현재 thinking 레벨   | `high`, `low`, `off`              |
| `{identity.name}` | 에이전트 identity 이름 | (`"auto"`와 동일) |

변수는 대소문자를 구분하지 않습니다. `{think}`는 `{thinkingLevel}`의 별칭입니다.

### Ack 반응

- 기본값은 활성 에이전트의 `identity.emoji`이며, 없으면 `"👀"`입니다. `""`로 설정하면 비활성화됩니다.
- 채널별 오버라이드: `channels.<channel>``.ackReaction`, `channels.<channel>``.accounts.<id>``.ackReaction`.
- 해결 순서: account → channel → `messages.ackReaction` → identity 기본값.
- 범위: `group-mentions` (기본값), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: 응답 후 ack를 제거합니다 (Slack/Discord/Telegram/Google Chat 전용).

### 인바운드 디바운스

동일한 발신자가 빠르게 보낸 텍스트 전용 메시지를 하나의 에이전트 턴으로 묶습니다. 미디어/첨부 파일은 즉시 플러시됩니다. 제어 명령은 디바운싱을 우회합니다.

### TTS (텍스트 음성 변환)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
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

- `auto`는 자동 TTS를 제어합니다. `/tts off|always|inbound|tagged`는 세션별로 재정의합니다.
- `summaryModel`은 자동 요약을 위해 `agents.defaults.model.primary`를 재정의합니다.
- API 키는 `ELEVENLABS_API_KEY`/`XI_API_KEY` 및 `OPENAI_API_KEY`를 대체로 사용합니다.

---

## Talk

Talk 모드의 기본값 (macOS/iOS/Android).

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

- Voice ID는 `ELEVENLABS_VOICE_ID` 또는 `SAG_VOICE_ID`를 대체로 사용합니다.
- `apiKey`는 `ELEVENLABS_API_KEY`를 대체로 사용합니다.
- `voiceAliases`를 사용하면 Talk 지시어에서 친숙한 이름을 사용할 수 있습니다.

---

## 도구

### 도구 프로필

`tools.profile`은 `tools.allow`/`tools.deny` 이전에 기본 허용 목록을 설정합니다:

| 프로필         | 포함 항목                                                                                     |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | `session_status`만                                                                         |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | 제한 없음 (설정하지 않은 경우와 동일)                                                 |

### 도구 그룹

| 그룹                 | 도구                                                                                       |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash`는 `exec`의 별칭으로 허용됨)                          |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | 모든 내장 도구 (provider 플러그인 제외)                                           |

### `tools.allow` / `tools.deny`

전역 도구 허용/차단 정책 (deny가 우선 적용됨). 대소문자를 구분하지 않으며, `*` 와일드카드를 지원합니다. Docker 샌드박스가 비활성화된 경우에도 적용됩니다.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

특정 provider 또는 모델에 대해 도구를 추가로 제한합니다. 적용 순서: 기본 프로필 → provider 프로필 → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

상승된(호스트) exec 접근을 제어합니다:

```json5
{
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

- 에이전트별 재정의(`agents.list[].tools.elevated`)는 추가 제한만 가능합니다.
- `/elevated on|off|ask|full`은 세션별로 상태를 저장하며, 인라인 지시어는 단일 메시지에만 적용됩니다.
- 상승된 `exec`는 호스트에서 실행되며, 샌드박싱을 우회합니다.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // 또는 BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

수신 미디어 이해 기능(이미지/오디오/비디오)을 구성합니다:

```json5
{
  tools: {
    media: {
      concurrency: 2,
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

<Accordion title="Media model entry fields">

**Provider 항목** (`type: "provider"` 또는 생략):

- `provider`: API provider id (`openai`, `anthropic`, `google`/`gemini`, `groq` 등)
- `model`: 모델 id 재정의
- `profile` / `preferredProfile`: 인증 프로필 선택

**CLI 항목** (`type: "cli"`):

- `command`: 실행할 실행 파일
- `args`: 템플릿 인자 (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}` 등 지원)

**공통 필드:**

- `capabilities`: 선택적 목록 (`image`, `audio`, `video`). 기본값: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: 항목별 재정의 옵션.
- 실패 시 다음 항목으로 자동 대체됩니다.

Provider 인증은 표준 순서를 따릅니다: 인증 프로필 → 환경 변수 → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

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

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: 생성된 하위 에이전트에 대한 기본 모델. 생략하면 하위 에이전트는 호출자의 모델을 상속합니다.
- 하위 에이전트별 도구 정책: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## 사용자 정의 Provider 및 기본 URL

OpenClaw는 pi-coding-agent 모델 카탈로그를 사용합니다. config 또는 `~/.openclaw/agents/<agentId>/agent/models.json`의 `models.providers`를 통해 사용자 정의 Provider를 추가하세요.

```json5
{
  models: {
    mode: "merge", // merge (기본값) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- 사용자 정의 인증이 필요한 경우 `authHeader: true`와 `headers`를 사용하세요.
- `OPENCLAW_AGENT_DIR`(또는 `PI_CODING_AGENT_DIR`)로 에이전트 설정 루트 경로를 재정의하세요.

### Provider 예시

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

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

Cerebras에는 `cerebras/zai-glm-4.7`을 사용하고, Z.AI 직접 연결에는 `zai/glm-4.7`을 사용하세요.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

`OPENCODE_API_KEY`(또는 `OPENCODE_ZEN_API_KEY`)를 설정하세요. 단축 명령: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

`ZAI_API_KEY`를 설정하세요. `z.ai/*` 및 `z-ai/*`는 허용되는 별칭입니다. 단축 명령: `openclaw onboard --auth-choice zai-api-key`.

- 일반 엔드포인트: `https://api.z.ai/api/paas/v4`
- 코딩 엔드포인트(기본값): `https://api.z.ai/api/coding/paas/v4`
- 일반 엔드포인트를 사용하려면 base URL을 재정의한 사용자 정의 Provider를 정의하세요.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

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

중국 엔드포인트의 경우: `baseUrl: "https://api.moonshot.cn/v1"` 또는 `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

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

Anthropic 호환, 내장 Provider. 단축 명령: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

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

Base URL에는 `/v1`을 포함하지 마세요(Anthropic 클라이언트가 자동으로 추가합니다). 단축 명령: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
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

`MINIMAX_API_KEY`를 설정합니다. 단축 명령: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models)을 참고하세요. 요약: 고성능 하드웨어에서 LM Studio Responses API를 통해 MiniMax M2.1을 실행하고, 장애 대비용으로 호스팅 모델을 함께 유지하세요.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: 번들된 skills에만 적용되는 선택적 허용 목록입니다(관리형/워크스페이스 skills에는 영향 없음).
- `entries.<skillKey>.enabled: false`로 설정하면 번들되었거나 설치된 경우에도 해당 skill을 비활성화합니다.
- `entries.<skillKey>.apiKey`: 기본 env 변수를 선언하는 skills를 위한 편의 설정입니다.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions`, 그리고 `plugins.load.paths`에서 로드됩니다.
- **구성 변경 사항은 gateway 재시작이 필요합니다.**
- `allow`: 선택적 허용 목록(목록에 있는 plugins만 로드됨). `deny`가 우선 적용됩니다.

[Plugins](/tools/plugin)을 참고하세요.

---

## Browser

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false`로 설정하면 `act:evaluate` 및 `wait --fn`이 비활성화됩니다.
- 원격 프로필은 attach 전용입니다(시작/중지/재설정 비활성화).
- 자동 감지 순서: 기본 브라우저가 Chromium 기반인 경우 → Chrome → Brave → Edge → Chromium → Chrome Canary.
- 제어 서비스: 루프백 전용(`gateway.port`에서 파생된 포트, 기본값 `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: 네이티브 앱 UI 크롬의 강조 색상(예: Talk Mode 말풍선 색조 등).
- `assistant`: Control UI의 식별 정보 재정의. 활성 에이전트 식별 정보로 대체됩니다.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local`(gateway 실행) 또는 `remote`(원격 gateway에 연결). `local`이 아니면 Gateway는 시작되지 않습니다.
- `port`: WS + HTTP에 사용되는 단일 멀티플렉스 포트. 우선순위: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback`(기본값), `lan`(`0.0.0.0`), `tailnet`(Tailscale IP 전용) 또는 `custom`.
- **Auth**: 기본적으로 필수입니다. 루프백이 아닌 바인딩에는 공유 token/password가 필요합니다. 온보딩 마법사는 기본적으로 token을 생성합니다.
- `auth.mode: "trusted-proxy"`: 인증을 ID 인식 리버스 프록시에 위임하고 `gateway.trustedProxies`의 ID 헤더를 신뢰합니다([Trusted Proxy Auth](/gateway/trusted-proxy-auth) 참고).
- `auth.allowTailscale`: `true`로 설정하면 Tailscale Serve ID 헤더가 인증을 충족합니다(`tailscale whois`로 검증됨). `tailscale.mode = "serve"`일 때 기본값은 `true`입니다.
- `auth.rateLimit`: 선택적 인증 실패 제한 기능입니다. 클라이언트 IP별 및 인증 범위별로 적용됩니다(shared-secret과 device-token은 각각 독립적으로 추적됨). 차단된 요청은 `429` + `Retry-After`를 반환합니다.
  - `auth.rateLimit.exemptLoopback`의 기본값은 `true`입니다. 로컬호스트 트래픽에도 속도 제한을 의도적으로 적용하려면(테스트 환경 또는 엄격한 프록시 배포 시) `false`로 설정하세요.
- `tailscale.mode`: `serve`(tailnet 전용, loopback 바인딩) 또는 `funnel`(공개, 인증 필요).
- `remote.transport`: `ssh`(기본값) 또는 `direct`(ws/wss). `direct`를 사용할 경우 `remote.url`은 반드시 `ws://` 또는 `wss://`여야 합니다.
- `gateway.remote.token`은 원격 CLI 호출 전용이며, 로컬 gateway 인증을 활성화하지 않습니다.
- `trustedProxies`: TLS를 종료하는 리버스 프록시 IP입니다. 직접 제어하는 프록시만 나열하세요.
- `gateway.tools.deny`: HTTP `POST /tools/invoke`에서 차단할 추가 도구 이름(기본 deny 목록에 추가).
- `gateway.tools.allow`: 기본 HTTP deny 목록에서 특정 도구 이름을 제거합니다.

</Accordion>

### OpenAI 호환 엔드포인트

- Chat Completions: 기본적으로 비활성화되어 있습니다. `gateway.http.endpoints.chatCompletions.enabled: true`로 활성화하세요.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Responses URL 입력 보안 강화:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### 멀티 인스턴스 격리

하나의 호스트에서 고유한 포트와 상태 디렉터리를 사용해 여러 gateway를 실행하세요:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

편의 플래그: `--dev`(`~/.openclaw-dev` + 포트 `19001` 사용), `--profile <name>`(`~/.openclaw-<name>` 사용).

[Multiple Gateways](/gateway/multiple-gateways)를 참고하세요.

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
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

인증: `Authorization: Bearer <token>` 또는 `x-openclaw-token: <token>`.

**엔드포인트:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - 요청 페이로드의 `sessionKey`는 `hooks.allowRequestSessionKey=true`일 때만 허용됩니다(기본값: `false`).
- `POST /hooks/<name>` → `hooks.mappings`를 통해 처리됩니다.

<Accordion title="Mapping details">

- `match.path`는 `/hooks` 이후의 하위 경로와 일치합니다(예: `/hooks/gmail` → `gmail`).
- `match.source`는 일반 경로에서 페이로드 필드와 일치합니다.
- `{{messages[0].subject}}`와 같은 템플릿은 페이로드에서 값을 읽습니다.
- `transform`은 hook action을 반환하는 JS/TS 모듈을 가리킬 수 있습니다.
  - `transform.module`은 상대 경로여야 하며 `hooks.transformsDir` 내부에 있어야 합니다(절대 경로와 상위 경로 탐색은 거부됨).
- `agentId`는 특정 에이전트로 라우팅합니다. 알 수 없는 ID는 기본값으로 대체됩니다.
- `allowedAgentIds`: 명시적 라우팅을 제한합니다(`*` 또는 생략 = 모두 허용, `[]` = 모두 거부).
- `defaultSessionKey`: 명시적인 `sessionKey` 없이 hook 에이전트를 실행할 때 사용할 선택적 고정 세션 키입니다.
- `allowRequestSessionKey`: `/hooks/agent` 호출자가 `sessionKey`를 설정하도록 허용합니다(기본값: `false`).
- `allowedSessionKeyPrefixes`: 명시적 `sessionKey` 값(요청 + 매핑)에 대한 선택적 접두사 허용 목록입니다. 예: `["hook:"]`.
- `deliver: true`는 최종 응답을 채널로 전송합니다. `channel`의 기본값은 `last`입니다.
- `model`은 이 hook 실행에 사용할 LLM을 재정의합니다(모델 카탈로그가 설정된 경우 허용된 모델이어야 함).

</Accordion>

### Gmail 통합

```json5
{
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
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- 구성된 경우 Gateway는 부팅 시 `gog gmail watch serve`를 자동으로 시작합니다. 비활성화하려면 `OPENCLAW_SKIP_GMAIL_WATCHER=1`을 설정하세요.
- Gateway와 함께 별도의 `gog gmail watch serve`를 실행하지 마세요.

---

## Canvas 호스트

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // 또는 OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway 포트에서 HTTP를 통해 에이전트가 편집 가능한 HTML/CSS/JS 및 A2UI를 제공합니다:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- 로컬 전용: `gateway.bind: "loopback"`(기본값)을 유지하세요.
- 루프백이 아닌 바인딩의 경우: canvas 경로는 다른 Gateway HTTP 엔드포인트와 동일하게 Gateway 인증(token/password/trusted-proxy)을 요구합니다.
- Node WebView는 일반적으로 인증 헤더를 전송하지 않습니다. 노드가 페어링 및 연결되면, Gateway는 비공개 IP 대체 방식을 허용하여 URL에 비밀 정보를 노출하지 않고도 노드가 canvas/A2UI를 로드할 수 있게 합니다.
- 제공되는 HTML에 라이브 리로드 클라이언트를 삽입합니다.
- 비어 있는 경우 기본 `index.html`을 자동으로 생성합니다.
- `/__openclaw__/a2ui/` 경로에서 A2UI도 제공합니다.
- 변경 사항을 적용하려면 gateway를 재시작해야 합니다.
- 대규모 디렉터리 또는 `EMFILE` 오류가 발생하는 경우 라이브 리로드를 비활성화하세요.

---

## Discovery

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal`(기본값): TXT 레코드에서 `cliPath` + `sshPort`를 제외합니다.
- `full`: `cliPath` + `sshPort`를 포함합니다.
- 호스트 이름의 기본값은 `openclaw`입니다. `OPENCLAW_MDNS_HOSTNAME`으로 재정의할 수 있습니다.

### 와이드 영역 (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` 아래에 유니캐스트 DNS-SD 영역을 작성합니다. 네트워크 간 Discovery를 위해 DNS 서버(CoreDNS 권장) + Tailscale split DNS와 함께 구성하세요.

설정: `openclaw dns setup --apply`.

---

## 환경

### `env` (인라인 환경 변수)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- 인라인 env 변수는 프로세스 env에 해당 키가 없을 때만 적용됩니다.
- `.env` 파일: CWD `.env` + `~/.openclaw/.env` (기존 변수를 덮어쓰지 않음).
- `shellEnv`: 로그인 셸 프로필에서 필요한 키가 없을 경우 이를 가져옵니다.
- 전체 우선순위는 [Environment](/help/environment)를 참고하세요.

### Env 변수 치환

`${VAR_NAME}`을 사용해 모든 config 문자열에서 env 변수를 참조할 수 있습니다:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- 대문자 이름만 매칭됩니다: `[A-Z_][A-Z0-9_]*`.
- 누락되었거나 비어 있는 변수는 config 로드 시 오류를 발생시킵니다.
- 리터럴 `${VAR}`가 필요하면 `$${VAR}`로 이스케이프하세요.
- `$include`와 함께 사용할 수 있습니다.

---

## Auth 저장소

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

- 에이전트별 auth 프로필은 `<agentDir>/auth-profiles.json`에 저장됩니다.
- 레거시 OAuth는 `~/.openclaw/credentials/oauth.json`에서 가져옵니다.
- [OAuth](/concepts/oauth)를 참고하세요.

---

## 로깅

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- 기본 로그 파일: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- 고정 경로를 사용하려면 `logging.file`을 설정하세요.
- `--verbose` 사용 시 `consoleLevel`은 `debug`로 상승합니다.

---

## Wizard

CLI wizard(`onboard`, `configure`, `doctor`)가 기록하는 메타데이터:

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

---

## Identity

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

macOS 온보딩 도우미에 의해 작성됩니다. 기본값을 다음과 같이 도출합니다:

- `identity.emoji`에서 `messages.ackReaction`을 설정합니다 (없으면 👀로 대체)
- `identity.name`/`identity.emoji`에서 `mentionPatterns`를 생성합니다.
- `avatar`는 다음을 허용합니다: 워크스페이스 상대 경로, `http(s)` URL 또는 `data:` URI

---

## Bridge (레거시, 제거됨)

현재 빌드에는 더 이상 TCP bridge가 포함되지 않습니다. 노드는 Gateway WebSocket을 통해 연결됩니다. `bridge.*` 키는 더 이상 config 스키마의 일부가 아닙니다 (제거할 때까지 유효성 검사 실패; `openclaw doctor --fix`로 알 수 없는 키를 제거할 수 있음).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: 완료된 cron 세션을 정리하기 전에 보관하는 기간입니다. 기본값: `24h`.

[Cron Jobs](/automation/cron-jobs)를 참조하세요.

---

## 미디어 모델 템플릿 변수

`tools.media.*.models[].args`에서 확장되는 템플릿 플레이스홀더:

| 변수                 | 설명                                                             |
| ------------------ | -------------------------------------------------------------- |
| `{{Body}}`         | 전체 수신 메시지 본문                                                   |
| `{{RawBody}}`      | 원본 본문 (히스토리/발신자 래퍼 없음)                      |
| `{{BodyStripped}}` | 그룹 멘션이 제거된 본문                                                  |
| `{{From}}`         | 발신자 식별자                                                        |
| `{{To}}`           | 수신 대상 식별자                                                      |
| `{{MessageSid}}`   | 채널 메시지 ID                                                      |
| `{{SessionId}}`    | 현재 세션 UUID                                                     |
| `{{IsNewSession}}` | 새 세션이 생성된 경우 "true"                                            |
| `{{MediaUrl}}`     | 수신 미디어 의사 URL                                                  |
| `{{MediaPath}}`    | 로컬 미디어 경로                                                      |
| `{{MediaType}}`    | 미디어 유형 (image/audio/document/…)             |
| `{{Transcript}}`   | 오디오 전사 내용                                                      |
| `{{Prompt}}`       | CLI 항목에 대해 확인된 미디어 프롬프트                                        |
| `{{MaxChars}}`     | CLI 항목에 대해 확인된 최대 출력 문자 수                                      |
| `{{ChatType}}`     | "direct" 또는 "group"                                            |
| `{{GroupSubject}}` | 그룹 제목 (가능한 범위 내)                            |
| `{{GroupMembers}}` | 그룹 구성원 미리보기 (가능한 범위 내)                      |
| `{{SenderName}}`   | 발신자 표시 이름 (가능한 범위 내에서)                      |
| `{{SenderE164}}`   | 발신자 전화번호 (가능한 범위 내에서)                       |
| `{{Provider}}`     | Provider 힌트 (whatsapp, telegram, discord 등) |

---

## Config에 포함 (`$include`)

Config를 여러 파일로 분할:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**병합 동작:**

- 단일 파일: 해당 객체를 완전히 대체합니다.
- 파일 배열: 순서대로 깊은 병합(deep merge)됩니다 (뒤의 항목이 앞의 항목을 덮어씀).
- 형제 키: include 이후에 병합됩니다 (포함된 값을 덮어씀).
- 중첩 include: 최대 10단계까지 지원합니다.
- 경로: 상대 경로(포함하는 파일 기준), 절대 경로, 또는 `../` 상위 경로 참조.
- 오류: 누락된 파일, 파싱 오류, 순환 include에 대해 명확한 메시지를 제공합니다.

---

_관련: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_

