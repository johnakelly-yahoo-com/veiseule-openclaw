---
summary: "Socket 또는 HTTP 웹훅 모드를 위한 Slack 설정"
read_when:
  - Slack 설정 또는 Slack socket/HTTP 모드 디버깅 시
title: "Slack"
---

# Slack

상태: Slack 앱 통합을 통해 DM + 채널에서 프로덕션 사용 가능. HTTP 모드 (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DM은 기본적으로 페어링 모드로 설정됩니다.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    네이티브 명령 동작 및 명령 카탈로그.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    크로스 채널 진단 및 복구 플레이북.
  
</Card>
</CardGroup>

## 빠른 설정 (초보자)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Slack 앱 설정에서:

        ```
        **App Token** (`xapp-...`) 과 **Bot Token** (`xoxb-...`) 을 생성합니다.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            환경 변수 폴백(기본 계정에만 적용):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Subscribe app events">
          다음에 대해 봇 이벤트를 구독하세요:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          또한 DM을 위해 App Home **Messages Tab**을 활성화하세요.
        
</Step>
      
        <Step title="Start gateway">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
        Gateway(게이트웨이) 가 HTTPS 를 통해 Slack 에서 접근 가능한 경우 (일반적인 서버 배포) HTTP 웹훅 모드를 사용합니다. HTTP 모드는 Events API + Interactivity + Slash Commands 를 공유 요청 URL 로 사용합니다.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      다중 계정 HTTP 모드: `channels.slack.accounts.<id> .mode = "http"` 를 설정하고 각 계정마다 고유한 `webhookPath` 를 제공하여 각 Slack 앱이 자체 URL 을 가리키도록 합니다.

  
</Tab>
</Tabs>

## 토큰 모델

- Socket Mode에는 `botToken` + `appToken`이 필요합니다.
- HTTP 모드에는 `botToken` + `signingSecret`이 필요합니다.
- 설정된 토큰이 환경 변수 폴백보다 우선합니다.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` 환경 변수 폴백은 기본 계정에만 적용됩니다.
- userTokenReadOnly 를 명시적으로 설정한 예시 (사용자 토큰 쓰기 허용):
- 선택 사항: 발신 메시지가 활성 에이전트 아이덴티티(사용자 지정 `username` 및 아이콘)를 사용하도록 하려면 `chat:write.customize`를 추가하세요. `icon_emoji`는 `:emoji_name:` 형식을 사용합니다.

<Tip>
작업/디렉터리 읽기의 경우, 설정되어 있다면 사용자 토큰을 우선 사용할 수 있습니다. 사용자 토큰이 있으면 읽기는 사용자 토큰을 우선하며, 쓰기는 명시적으로 옵트인하지 않는 한 봇 토큰을 사용합니다. `userTokenReadOnly: false` 를 설정하더라도, 봇 토큰이 사용 가능하면 쓰기에는 봇 토큰이 계속 우선됩니다.
</Tip>

## 접근 제어 및 라우팅

<Tabs>
  <Tab title="DM policy">누구나 허용하려면 `channels.slack.dm.policy="open"` 와 `channels.slack.dm.allowFrom=["*"]` 를 설정하십시오.

    ```
    `allowlist` 는 채널이 `channels.slack.channels` 에 나열되어 있을 것을 요구합니다.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` 는 채널 처리를 제어합니다 (`open|disabled|allowlist`).

    ```
    연결되었지만 채널 응답이 없음: `groupPolicy` 에 의해 채널이 차단되었거나 `channels.slack.channels` 허용 목록에 없음.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    채널 메시지는 기본적으로 멘션이 있을 때만 처리됩니다.
  

    ```
    경고: 다른 봇에 대한 답글을 허용하는 경우 (`channels.slack.allowBots=true` 또는 `channels.slack.channels.<id> .allowBots=true`), `requireMention`, `channels.slack.channels.<id> .users` 허용 목록 및/또는 `AGENTS.md` 와 `SOUL.md` 의 명확한 가드레일로 봇 간 무한 답글 루프를 방지하십시오.
    ```

  
</Tab>
</Tabs>

## 명령 및 슬래시 동작

- Slack 슬래시 명령은 Slack 앱에서 관리되며 자동으로 제거되지 않습니다.
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- 네이티브 명령이 활성화된 경우, Slack에서 일치하는 슬래시 명령(`/<command>` 이름)을 등록하세요.
- 네이티브 명령을 활성화하는 경우, 노출하려는 각 명령마다 하나의 `slash_commands` 항목을 추가합니다 (`/help` 목록과 일치). `channels.slack.commands.native` 로 재정의할 수 있습니다.

기본 슬래시 명령 설정:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

슬래시 세션은 분리된 키를 사용합니다:

- Slash 명령은 `agent:<agentId>:slack:slash:<userId>` 세션을 사용합니다 (접두사는 `channels.slack.slashCommand.sessionPrefix` 로 구성 가능).

그리고 명령 실행은 대상 대화 세션(`CommandTargetSessionKey`)을 기준으로 계속 라우팅됩니다.

## 스레딩, 세션 및 답장 태그

- DM은 `direct`로, 채널은 `channel`로, MPIM은 `group`으로 라우팅됩니다.
- 기본 `session.dmScope=main` 설정에서는 Slack DM이 에이전트의 메인 세션으로 통합됩니다.
- 채널은 `agent:<agentId>:slack:channel:<channelId>` 세션에 매핑됩니다.
- 스레드 답장은 해당하는 경우 스레드 세션 접미사(`:thread:<threadTs>`)를 생성할 수 있습니다.
- `channels.slack.thread.historyScope`의 기본값은 `thread`이며, `thread.inheritParent`의 기본값은 `false`입니다.
- `channels.slack.thread.initialHistoryLimit`은 새 스레드 세션이 시작될 때 기존 스레드 메시지를 몇 개까지 가져올지 제어합니다(기본값 `20`; 비활성화하려면 `0`으로 설정).

답글 스레딩

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- 직접 채팅에 대한 레거시 폴백: `channels.slack.dm.replyToMode`

수동 답장 태그가 지원됩니다:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]` — 특정 메시지 ID 에 답글.

참고: `replyToMode="off"`는 암시적 답글 스레딩을 비활성화합니다. 명시적인 `[[reply_to_*]]` 태그는 계속 적용됩니다.

## 미디어, 청크 분할 및 전송

<AccordionGroup>
  <Accordion title="Inbound attachments">첨부 파일은 허용되고 크기 제한 내인 경우 미디어 저장소로 다운로드됩니다.

    ```
    미디어 업로드는 `channels.slack.mediaMaxMb` 로 제한됩니다 (기본값 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - 텍스트 청크는 `channels.slack.textChunkLimit`(기본값 4000)을 사용합니다
    - `channels.slack.chunkMode="newline"`은 문단 우선 분할을 활성화합니다
    - 파일 전송은 Slack 업로드 API를 사용하며 스레드 답글(`thread_ts`)을 포함할 수 있습니다
    - 발신 미디어 용량 제한은 설정된 경우 `channels.slack.mediaMaxMb`를 따르며, 그렇지 않으면 채널 전송은 미디어 파이프라인의 MIME 유형 기본값을 따릅니다
  
</Accordion>

  <Accordion title="Delivery targets">
    권장되는 명시적 대상:

    ```
    - DM의 경우 `user:<id>`
    - 채널의 경우 `channel:<id>`
    
    사용자 대상으로 전송할 때 Slack DM은 Slack conversation API를 통해 열립니다.
    ```

  
</Accordion>
</AccordionGroup>

## 작업 및 게이트

Slack 도구 작업은 `channels.slack.actions.*` 로 게이팅할 수 있습니다:

현재 Slack 도구에서 사용 가능한 작업 그룹:

| 작업 그룹      | 기본값     |
| ---------- | ------- |
| messages   | enabled |
| reactions  | enabled |
| pins       | enabled |
| memberInfo | enabled |
| emojiList  | enabled |

## 동작

- 메시지 수정/삭제/스레드 브로드캐스트는 시스템 이벤트로 매핑됩니다.
- 리액션 추가/제거 이벤트는 시스템 이벤트로 매핑됩니다.
- 멤버 참여/퇴장, 채널 생성/이름 변경, 고정 추가/제거 이벤트는 시스템 이벤트로 매핑됩니다.
- `channel_id_changed`는 `configWrites`가 활성화된 경우 채널 구성 키를 마이그레이션할 수 있습니다.
- 채널 주제/목적 메타데이터는 신뢰할 수 없는 컨텍스트로 처리되며 라우팅 컨텍스트에 주입될 수 있습니다.

## Ack 리액션

`ackReaction`은 OpenClaw가 수신 메시지를 처리하는 동안 확인용 이모지를 전송합니다.

해결 순서:

- 다중 계정의 경우 `channels.slack.accounts.<id>.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- 에이전트 식별 이모지 대체값 (`agents.list[].identity.emoji`, 없으면 "👀")

참고

- Slack은 쇼트코드(예: `"eyes"`)를 기대합니다.
- 채널 또는 계정에서 리액션을 비활성화하려면 `""`를 사용하세요.

## 매니페스트 및 권한 범위 체크리스트

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    `channels.slack.userToken`을 구성하는 경우, 일반적인 읽기 권한 범위는 다음과 같습니다:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## 문제 해결

<AccordionGroup>
  <Accordion title="No replies in channels">
    다음을 순서대로 확인하세요:

    ```
    `users`: 선택적 채널별 사용자 허용 목록.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    확인:

    ```
    **채널을 전혀 허용하지 않으려면**, `channels.slack.groupPolicy: "disabled"` 를 설정하십시오 (또는 빈 허용 목록 유지).
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Slack 앱 설정에서 봇 및 앱 토큰과 Socket Mode 활성화를 검증하세요.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    검증:

    ```
    - signing secret
    - webhook 경로
    - Slack Request URL(Events + Interactivity + Slash Commands)
    - HTTP 계정별 고유한 `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    다음 의도에 해당하는지 확인하세요:

    ```
    Slash Commands → `channels.slack.slashCommand` 를 사용하는 경우 `/openclaw` 를 생성합니다. 네이티브 명령을 활성화하는 경우, 내장 명령마다 하나의 슬래시 명령을 추가합니다 (`/help` 와 동일한 이름). Slack 에서는 기본적으로 네이티브가 꺼져 있으므로 `channels.slack.commands.native: true` 를 설정해야 합니다 (전역 `commands.native` 의 기본값은 `"auto"` 로 Slack 을 끕니다).
    ```

  
</Accordion>
</AccordionGroup>

## 구성 참조 포인터

우선순위:

- [https://api.slack.com/apps](https://api.slack.com/apps) 에서 Slack 앱을 생성합니다 (From scratch).

  중요한 Slack 필드:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM 접근: `dm.enabled`, `dmPolicy`, `allowFrom` (레거시: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: `groupPolicy="allowlist"` 일 때 채널 허용/차단.
  - 스레딩/히스토리: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - 전송: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - 운영/기능: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## 관련

- [페어링](/channels/pairing)
- `channel`: 일반 채널 (공개/비공개)
- 트리아지 흐름: [/channels/troubleshooting](/channels/troubleshooting).
- [구성](/gateway/configuration)
- 전체 명령 목록 + 설정: [Slash commands](/tools/slash-commands)

