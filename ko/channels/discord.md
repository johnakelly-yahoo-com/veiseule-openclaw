---
summary: "Discord 봇 지원 상태, 기능, 구성"
read_when:
  - Discord 채널 기능 작업 중일 때
title: "Discord"
---

# Discord (Bot API)

상태: 공식 Discord 봇 게이트웨이를 통해 다이렉트 메시지와 길드 텍스트 채널을 지원할 준비가 되어 있습니다.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM은 기본적으로 페어링 모드로 설정됩니다.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    기본 명령 동작 및 명령 카탈로그.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    채널 간 진단 및 복구 흐름.
  
</Card>
</CardGroup>

## 빠른 설정 (초보자)

<Steps>
  <Step title="Create a Discord bot and enable intents">Discord Developer Portal에서 애플리케이션을 생성하고 봇을 추가한 다음, 다음을 활성화하세요:

    ```
    Discord 앱 설정에서 **Message Content Intent** 를 활성화합니다 (허용 목록 또는 이름 조회를 사용할 계획이라면 **Server Members Intent** 도 함께 활성화합니다).
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    기본 계정을 위한 Env 대체 값:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">
    메시지 권한을 부여하여 봇을 서버에 초대하세요.
  

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
`openclaw pairing approve discord <code>` 로 승인합니다.
```

    ```
    페어링 코드는 1시간 후 만료됩니다.
    ```

  
</Step>
</Steps>

<Note>
토큰 해석은 계정 인식(account-aware) 방식으로 이루어집니다. Config의 토큰 값이 환경 변수 대체값(env fallback)보다 우선합니다. `DISCORD_BOT_TOKEN`은 기본 계정에만 사용됩니다.
</Note>

## 런타임 모델

- Gateway가 Discord 연결을 관리합니다.
- `[[reply_to_current]]` — 트리거된 Discord 메시지에 답글합니다.
- 에이전트는 다음과 같은 액션으로 `discord` 를 호출할 수 있습니다:
- 선택적 길드 규칙: 길드 id (권장) 또는 슬러그를 키로 하는 `channels.discord.guilds` 을 설정하고 채널별 규칙을 지정합니다.
- `dm.groupEnabled`: 그룹 다이렉트 메시지 활성화 (기본값 `false`).
- 네이티브 명령은 공유된 `main` 세션이 아니라 격리된 세션 키 (`agent:<agentId>:discord:slash:<userId>`) 를 사용합니다.

## 접근 제어 및 라우팅

<Tabs>
  <Tab title="DM policy">이전의 ‘누구나 허용’ 동작을 유지하려면 `channels.discord.dm.policy="open"` 와 `channels.discord.dm.allowFrom=["*"]` 을 설정합니다.

    ```
    강제 허용 목록을 사용하려면 `channels.discord.dm.policy="allowlist"` 을 설정하고 `channels.discord.dm.allowFrom` 에 발신자를 나열합니다.
    ```

  
</Tab>

  <Tab title="Guild policy">동작은 `channels.discord.replyToMode` 로 제어됩니다:

    ```
    길드/채널 허용 목록이 채널/사용자를 거부함.
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

    ```
    `DISCORD_BOT_TOKEN` 만 설정하고 `channels.discord` 섹션을 생성하지 않으면, 런타임은
    `groupPolicy` 를 `open` 로 기본 설정합니다.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    길드 메시지는 기본적으로 멘션이 있을 때만 처리됩니다.

멘션 감지에는 다음이 포함됩니다:

- 명시적인 봇 멘션
- 설정된 멘션 패턴 (`agents.list[].groupChat.mentionPatterns`, 대체값 `messages.groupChat.mentionPatterns`)
- 지원되는 경우 봇에 대한 암묵적 답장 동작

`requireMention`은 길드/채널별로 설정됩니다 (`channels.discord.guilds...`).

그룹 DM:

- 기본값: 무시됨 (`dm.groupEnabled=false`)
- 선택적 허용 목록: `dm.groupChannels` (채널 ID 또는 슬러그)

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

### `bindings[].match.roles`를 사용하여 Discord 길드 멤버를 역할 ID에 따라 서로 다른 에이전트로 라우팅합니다.

역할 기반 바인딩은 역할 ID만 허용하며, peer 또는 parent-peer 바인딩 이후, guild-only 바인딩 이전에 평가됩니다. 바인딩에 다른 match 필드(예: `peer` + `guildId` + `roles`)가 함께 설정된 경우, 구성된 모든 필드가 일치해야 합니다. {
bindings: [
{
agentId: "opus",
match: {
channel: "discord",
guildId: "123456789012345678",
roles: ["111111111111111111"],
},
},
{
agentId: "sonnet",
match: {
channel: "discord",
guildId: "123456789012345678",
},
},
],
}

```json5
Developer Portal 설정
```

## - Message Content Intent
- Server Members Intent (권장)Presence intent는 선택 사항이며, presence 업데이트를 수신하려는 경우에만 필요합니다. 봇의 presence를 설정(`setPresence`)하는 것은 멤버의 presence 업데이트 활성화와는 무관합니다.

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">**Bot** → **Privileged Gateway Intents** 에서 다음을 활성화합니다:

    ```
    - scopes: `bot`, `applications.commands`
    
    일반적인 기본 권한:
    
    - 채널 보기 (View Channels)
    - 메시지 보내기 (Send Messages)
    - 메시지 기록 읽기 (Read Message History)
    - 링크 포함 (Embed Links)
    - 파일 첨부 (Attach Files)
    - 반응 추가 (선택 사항)
    
    명시적으로 필요한 경우가 아니라면 `Administrator` 권한은 피하세요.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">앱에서: **OAuth2** → **URL Generator**

    ```
    
        Discord Developer Mode를 활성화한 후 다음을 복사하세요:
    ```

  
</Accordion>

  <Accordion title="Copy IDs">- 서버 ID
- 채널 ID
- 사용자 ID

신뢰할 수 있는 감사 및 프로브를 위해 OpenClaw 설정에서는 숫자 ID 사용을 권장합니다.

    ```
      
</Accordion>
    ```

  
</Accordion>
</AccordionGroup>

## 기능 세부 사항

- 선택적 네이티브 명령어: `commands.native` 는 기본값이 `"auto"` 입니다 (Discord/Telegram 는 켜짐, Slack 은 꺼짐).
- 설정의 `channels.discord.execApprovals.enabled: true`.
- `channels.discord.commands.native: true|false|"auto"` 으로 재정의할 수 있으며, `false` 은 이전에 등록된 명령을 제거합니다.
- 네이티브 명령은 다이렉트 메시지/길드 메시지와 동일한 허용 목록을 따릅니다 (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, 채널별 규칙).
- 슬래시 명령은 허용 목록에 없는 사용자에게도 Discord UI 에 표시될 수 있지만, 실행 시 OpenClaw 가 허용 목록을 강제하며 “권한 없음” 으로 응답합니다.

전체 승인 및 명령 흐름은 [Exec 승인](/tools/exec-approvals) 과 [슬래시 명령어](/tools/slash-commands) 를 참고하십시오.

## ```
Discord는 에이전트 출력에서 답장 태그를 지원합니다:
```

<AccordionGroup>
  <Accordion title="Reply tags and native replies">- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

`channels.discord.replyToMode`로 제어됩니다:

- `off` (기본값)
- `first`
- `all`

참고: `off`는 암묵적인 답장 스레딩을 비활성화합니다. 명시적인 `[[reply_to_*]]` 태그는 여전히 적용됩니다.

메시지 ID는 컨텍스트/히스토리에 포함되어 에이전트가 특정 메시지를 대상으로 지정할 수 있습니다.

    ```
    
        길드 히스토리 컨텍스트:
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">- `channels.discord.historyLimit` 기본값 `20`
- 대체값: `messages.groupChat.historyLimit`
- `0`은 비활성화

DM 히스토리 제어:

- `channels.discord.dmHistoryLimit`
- `channels.discord.dms["<user_id>"].historyLimit`

스레드 동작:

- Discord 스레드는 채널 세션으로 라우팅됩니다
- 상위 스레드 메타데이터를 상위 세션 연결에 사용할 수 있습니다
- 스레드 전용 항목이 없는 경우, 스레드 설정은 상위 채널 설정을 상속합니다

채널 주제는 **신뢰되지 않은(untrusted)** 컨텍스트로 주입됩니다 (system prompt로는 사용되지 않음).

    ```
    
        `ackReaction`은 OpenClaw가 수신 메시지를 처리하는 동안 확인용 이모지를 전송합니다.
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications` 을 사용합니다:

    ```
    `guilds.<id> .reactionNotifications`: 반응 시스템 이벤트 모드 (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">해석 순서:

- `channels.discord.accounts.<accountId>.ackReaction`
- `channels.discord.ackReaction`
- `messages.ackReaction`
- 에이전트 identity 이모지 대체값 (`agents.list[].identity.emoji`, 없으면 "👀")

참고:

- Discord는 유니코드 이모지 또는 커스텀 이모지 이름을 허용합니다.
- 채널 또는 계정에서 반응을 비활성화하려면 `""`를 사용하세요.

    ```
    
        채널에서 시작된 설정 쓰기는 기본적으로 활성화되어 있습니다.
    ```

  
</Accordion>

  <Accordion title="Config writes">이는 `/config set|unset` 흐름에 영향을 줍니다 (명령 기능이 활성화된 경우).

비활성화:

    ```
    
        `channels.discord.proxy`를 사용하여 Discord gateway WebSocket 트래픽을 HTTP(S) 프록시를 통해 라우팅합니다.
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">계정별 재정의:

```json5
PK 조회가 실패하면 (예: 토큰이 없는 비공개 시스템),
  프록시 메시지는 봇 메시지로 취급되어 `channels.discord.allowBots=true` 가 없는 한 드롭됩니다.
```

    ```
    {
      channels: {
        discord: {
          accounts: {
            primary: {
              proxy: "http://proxy.example:8080",
            },
          },
        },
      },
    }
    ```

```json5

    PluralKit 해석을 활성화하여 프록시된 메시지를 시스템 멤버 identity에 매핑합니다:
```

  
</Accordion>

  <Accordion title="PluralKit support">참고:

- 허용 목록에는 `pk:<memberId>`를 사용할 수 있습니다
- 멤버 표시 이름은 이름/슬러그로 매칭됩니다
- 조회는 원본 메시지 ID를 사용하며 시간 창 제한을 받습니다
- 조회에 실패하면, 프록시된 메시지는 봇 메시지로 처리되어 `allowBots=true`가 아닌 한 삭제됩니다

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    
        status 또는 activity 필드를 설정한 경우에만 Presence 업데이트가 적용됩니다.
    ```

  
</Accordion>

  <Accordion title="Presence configuration">Status만 설정하는 예시:

    ```
    Activity 예시 (custom status가 기본 activity 유형입니다):
    ```

```json5
또는 설정: `channels.discord.token: "..."`.
```

    ```
    {
      channels: {
        discord: {
          activity: "Focus time",
          activityType: 4,
        },
      },
    }
    ```

```json5
Streaming 예시:
```

    ```
    Streaming example:
    ```

```json5
OpenClaw 를 `channels.discord.token` 으로 설정합니다 (또는 폴백으로 `DISCORD_BOT_TOKEN`).
```

    ```
    Activity type map:
    
    - 0: Playing
    - 1: Streaming (requires `activityUrl`)
    - 2: Listening
    - 3: Watching
    - 4: Custom (활동 텍스트를 상태 메시지로 사용; 이모지는 선택 사항)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">    Discord는 DM에서 버튼 기반 exec 승인 기능을 지원하며, 선택적으로 원래 채널에 승인 프롬프트를 게시할 수 있습니다.

    ```
    {
      channels: {
        discord: {
          enabled: true,
          token: "abc.123",
          groupPolicy: "allowlist",
          guilds: {
            "*": {
              channels: {
                general: { allow: true },
              },
            },
          },
          mediaMaxMb: 8,
          actions: {
            reactions: true,
            stickers: true,
            emojiUploads: true,
            stickerUploads: true,
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
            channels: true,
            voiceStatus: true,
            events: true,
            moderation: false,
            presence: false,
          },
          replyToMode: "off",
          dm: {
            enabled: true,
            policy: "pairing", // pairing | allowlist | open | disabled
            allowFrom: ["123456789012345678", "steipete"],
            groupEnabled: false,
            groupChannels: ["openclaw-dm"],
          },
          guilds: {
            "*": { requireMention: true },
            "123456789012345678": {
              slug: "friends-of-openclaw",
              requireMention: false,
              reactionNotifications: "own",
              users: ["987654321098765432", "steipete"],
              channels: {
                general: { allow: true },
                help: {
                  allow: true,
                  requireMention: true,
                  users: ["987654321098765432"],
                  skills: ["search", "docs"],
                  systemPrompt: "Keep answers short.",
                },
              },
            },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## 도구 액션

Discord 메시지 액션에는 메시징, 채널 관리, 모더레이션, 프레즌스, 메타데이터 액션이 포함됩니다.

Core examples:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- 반응 + 반응 목록 + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Action 게이트는 `channels.discord.actions.*` 아래에 있습니다.

Default gate behavior:

| 액션 그룹                                                                                                         | 기본값     |
| ------------------------------------------------------------------------------------------------------------- | ------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled |
| 역할                                                                                                            | 비활성화    |
| 모더레이션                                                                                                         | 비활성화    |
| presence                                                                                                      | 비활성화    |

## Components v2 UI

OpenClaw는 exec 승인 및 컨텍스트 간 마커를 위해 Discord components v2를 사용합니다. Discord 메시지 액션은 사용자 정의 UI를 위해 `components`도 사용할 수 있습니다(고급; Carbon component 인스턴스 필요). 기존 `embeds`도 계속 사용할 수 있지만 권장되지는 않습니다.

- `channels.discord.ui.components.accentColor`는 Discord 컴포넌트 컨테이너에서 사용되는 강조 색상(hex)을 설정합니다.
- 각 계정별로 `channels.discord.accounts.<id> .ui.components.accentColor`에서 설정합니다.
- components v2가 존재하면 `embeds`는 무시됩니다.

Example:

```json5
채널 (예: `#help`) → **Copy Channel ID**
```

## messages

Discord 음성 메시지는 파형 미리보기를 표시하며 OGG/Opus 오디오와 메타데이터가 필요합니다. OpenClaw는 파형을 자동으로 생성하지만, 오디오 파일을 검사하고 변환하려면 gateway 호스트에 `ffmpeg` 및 `ffprobe`가 설치되어 있어야 합니다.

기능 및 제한

- **로컬 파일 경로**를 제공하세요(URL은 허용되지 않음).
- 텍스트 콘텐츠는 생략하세요(Discord는 동일한 payload에 텍스트와 음성 메시지를 함께 허용하지 않음).
- 모든 오디오 형식이 허용되며, 필요한 경우 OpenClaw가 OGG/Opus로 변환합니다.

Example:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## 문제 해결

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **“Used disallowed intents”**: Developer Portal 에서 **Message Content Intent** (그리고 대개 **Server Members Intent**) 를 활성화한 뒤 Gateway(게이트웨이) 를 재시작하십시오.
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `groupPolicy`: 길드 채널 처리 제어 (`open|disabled|allowlist`).`allowlist` 는 채널 허용 목록이 필요합니다.
    ```

```bash
먼저 `openclaw doctor` 과 `openclaw channels status --probe` 를 실행합니다 (조치 가능한 경고 + 빠른 감사).
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">    Common causes:

    ```
    **어떤 채널도 허용하지 않으려면** `channels.discord.groupPolicy: "disabled"` 을 설정하십시오 (또는 빈 허용 목록 유지).
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**권한 감사** (`channels status --probe`) 는 숫자 채널 ID 만 검사합니다.

    ```
    slug 키를 사용하는 경우 런타임 매칭은 동작할 수 있지만, probe는 권한을 완전히 검증할 수 없습니다.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **다이렉트 메시지가 동작하지 않음**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"`, 또는 아직 승인되지 않았습니다 (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">봇이 작성한 메시지는 기본적으로 무시됩니다.

    ```
    경고: 다른 봇에 대한 응답을 허용하는 경우 (`channels.discord.allowBots=true`), `requireMention`, `channels.discord.guilds.*.channels.<id> .users` 허용 목록 및/또는 `AGENTS.md` 과 `SOUL.md` 의 가드레일을 사용하여 봇 간 응답 루프를 방지하십시오.
    ```

  
</Accordion>
</AccordionGroup>

## Configuration reference pointers

Primary reference:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

High-signal Discord fields:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- `guilds.<id> .channels.<channel> .allow`: `groupPolicy="allowlist"` 인 경우 채널 허용/거부.
- 명령에 대한 접근 그룹 검사를 우회하려면 `commands.useAccessGroups: false` 을 사용합니다.
- `dmHistoryLimit`: 사용자 턴 기준 다이렉트 메시지 히스토리 제한. 사용자별 재정의: `dms["<user_id>"].historyLimit`.
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## 보안 및 운영

- 봇 토큰은 비밀로 취급하세요(관리 환경에서는 `DISCORD_BOT_TOKEN` 사용 권장).
- 최소 권한의 Discord 권한만 부여하세요.
- 명령 배포/상태가 오래된 경우 gateway를 재시작하고 `openclaw channels status --probe`로 다시 확인하세요.

## 관련

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- [Troubleshooting](/channels/troubleshooting)
- 전체 명령 목록 및 설정: [슬래시 명령어](/tools/slash-commands)

