---
summary: "Telegram 봇 지원 상태, 기능 및 구성"
read_when:
  - Telegram 기능 또는 웹훅을 작업할 때
title: "Telegram"
---

# Telegram (Bot API)

상태: grammY 를 통한 봇 다이렉트 메시지 + 그룹에 대해 프로덕션 준비 완료. 기본은 롱 폴링이며, 웹훅은 선택 사항입니다.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Telegram의 기본 DM 정책은 페어링입니다.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    채널 간 진단 및 복구 플레이북.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    전체 채널 구성 패턴 및 예시.
  
</Card>
</CardGroup>

## 빠른 설정 (초보자)

<Steps>
  <Step title="Create the bot token in BotFather">Telegram 을 열고 **@BotFather** 와 대화합니다 ([직접 링크](https://t.me/BotFather)). 핸들이 정확히 `@BotFather` 인지 확인합니다.

    ```
    `/newbot` 는 봇을 생성하고 토큰을 반환합니다 (비밀로 유지하십시오).
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    환경 변수 옵션: `TELEGRAM_BOT_TOKEN=...` (기본 계정에서 작동).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    페어링 코드는 1시간 후 만료됩니다.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    봇을 그룹에 추가한 다음, 접근 모델에 맞게 `channels.telegram.groups`와 `groupPolicy`를 설정하세요.
  
</Step>
</Steps>

<Note>
토큰 확인 순서는 계정 인식 방식입니다. 실제로는 config 값이 env 대체값보다 우선하며, `TELEGRAM_BOT_TOKEN`은 기본 계정에만 적용됩니다.
</Note>

## Telegram 측 설정

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">@BotFather → "Group Privacy" 에서 프라이버시 설정이 **OFF** 인지 확인

    ```
    **참고:** 프라이버시 모드를 전환하면, 변경 사항을 적용하려면 Telegram 이 각 그룹에서 봇을 제거한 후 다시 추가할 것을 요구합니다.
    ```

  
</Accordion>

  <Accordion title="Group permissions">관리자 상태는 그룹 내 Telegram UI 에서 설정합니다.

    ```
    관리자 봇은 항상 모든 그룹 메시지를 수신하므로, 완전한 가시성이 필요하다면 관리자를 사용하십시오.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — 봇을 그룹에 추가하는 것을 허용/차단합니다.
    ```

  
</Accordion>
</AccordionGroup>

## 접근 제어 및 활성화

<Tabs>
  <Tab title="DM policy">`channels.telegram.replyToMode` 로 제어됩니다:

    ```
    - `pairing` (기본값)
    - `allowlist`
    - `open` (`allowFrom`에 `"*"` 포함 필요)
    - `disabled`
    
    `channels.telegram.allowFrom`은 숫자형 Telegram 사용자 ID를 허용합니다. `telegram:` / `tg:` 접두사는 허용되며 정규화됩니다.
    온보딩 마법사는 `@username` 입력을 받아 숫자 ID로 변환합니다.
    업그레이드 후 config에 `@username` allowlist 항목이 포함되어 있다면, `openclaw doctor --fix`를 실행해 이를 변환하세요(최선 시도 방식; Telegram 봇 토큰 필요).
    
    ### Telegram 사용자 ID 찾기
    
    더 안전한 방법(서드파티 봇 불필요):
    
    1. 봇에게 DM을 보냅니다.
    2. `openclaw logs --follow`를 실행합니다.
    3. `from.id`를 확인합니다.
    
    공식 Bot API 방법:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    **프라이버시 참고:** `@userinfobot` 는 서드파티 봇입니다.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">서로 독립적인 두 가지 제어가 있습니다:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">그룹 응답은 기본적으로 언급이 필요합니다 (기본 @멘션 또는 `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    멘션은 다음에서 발생할 수 있습니다:
    
    - 기본 `@botusername` 멘션, 또는
    - 다음의 멘션 패턴:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    세션 단위 명령 토글:
    
    - `/activation always`
    - `/activation mention`
    
    이 명령은 세션 상태만 업데이트합니다. 영구 적용하려면 config를 사용하세요.
    
    영구 config 예시:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    그룹의 메시지를 Telegram 에서 `@userinfobot` 또는 `@getidsbot` 로 전달하면 채팅 ID(예: `-1001234567890` 와 같은 음수)를 확인할 수 있습니다.
    ```

  
</Tab>
</Tabs>

## 런타임 동작

- Gateway 가 소유하는 Telegram Bot API 채널입니다.
- 결정적 라우팅: 응답은 항상 Telegram 으로 돌아가며, 모델은 채널을 선택하지 않습니다.
- 수신 메시지는 응답 컨텍스트와 미디어 플레이스홀더를 포함한 공유 채널 엔벨로프로 정규화됩니다.
- 그룹 세션은 그룹 ID별로 격리됩니다. 포럼 토픽은 \`channels.telegram.groups.<groupId> .topics.<topicId>
- 응답이 토픽에 유지되도록 `message_thread_id` 로 타이핑 표시와 응답을 전송합니다.
- 롱 폴링은 grammY 러너를 사용하며 채팅별 시퀀싱을 적용합니다. 전체 동시성은 `agents.defaults.maxConcurrent` 로 제한됩니다.
- Telegram Bot API 는 읽음 확인을 지원하지 않으므로 `sendReadReceipts` 옵션은 없습니다.

## 기능 참조

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw 는 `sendMessageDraft` 를 사용하여 Telegram 다이렉트 메시지에서 부분 응답 스트리밍을 지원합니다.

    ```
    요구 사항:
    
    - `channels.telegram.streamMode`가 `"off"`가 아님 (기본값: `"partial"`)
    
    모드:
    
    - `off`: 라이브 미리보기 없음
    - `partial`: 부분 텍스트로 빈번한 미리보기 업데이트
    - `block`: `channels.telegram.draftChunk`를 사용한 청크 단위 미리보기 업데이트
    
    `streamMode: "block"`일 때 `draftChunk` 기본값:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars`는 `channels.telegram.textChunkLimit`에 의해 제한됩니다.
    
    이 기능은 1:1 채팅과 그룹/토픽에서 모두 동작합니다.
    
    텍스트 전용 응답의 경우, OpenClaw는 동일한 미리보기 메시지를 유지하고 최종 단계에서 해당 메시지를 직접 수정합니다(두 번째 메시지 없음).
    
    복합 응답(예: 미디어 페이로드)의 경우, OpenClaw는 일반적인 최종 전송 방식으로 대체한 뒤 미리보기 메시지를 정리합니다.
    
    `streamMode`는 블록 스트리밍과는 별개입니다. Telegram에 대해 블록 스트리밍이 명시적으로 활성화된 경우, OpenClaw는 이중 스트리밍을 방지하기 위해 미리보기 스트림을 건너뜁니다.
    
    Telegram 전용 추론 스트림:
    
    - `/reasoning stream`은 생성 중 추론 내용을 라이브 미리보기에 전송합니다
    - 최종 답변은 추론 텍스트 없이 전송됩니다
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">발신 Telegram 텍스트는 `parse_mode: "HTML"` (Telegram 이 지원하는 태그 하위 집합)을 사용합니다.

    ```
    - Markdown 유사 텍스트는 Telegram에서 안전한 HTML로 렌더링됩니다.
    - 원시 모델 HTML은 Telegram 파싱 실패를 줄이기 위해 이스케이프 처리됩니다.
    - Telegram이 파싱된 HTML을 거부하면, OpenClaw는 일반 텍스트로 재시도합니다.
    
    링크 미리보기는 기본적으로 활성화되어 있으며, `channels.telegram.linkPreview: false`로 비활성화할 수 있습니다.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Some commands can be handled by plugins/skills without being registered in Telegram’s command menu.

    ```
    설정을 통해 메뉴에 사용자 정의 명령어를 추가할 수 있습니다:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    규칙:
    
    - 이름은 정규화됩니다(앞의 `/` 제거, 소문자 변환)
    - 유효 패턴: `a-z`, `0-9`, `_`, 길이 `1..32`
    - 사용자 정의 명령은 기본 명령을 덮어쓸 수 없습니다
    - 충돌/중복 항목은 건너뛰고 로그에 기록됩니다
    
    참고:
    
    - 사용자 정의 명령은 메뉴 항목일 뿐이며, 동작이 자동 구현되지는 않습니다
    - Telegram 메뉴에 표시되지 않더라도 plugin/skill 명령은 직접 입력하면 동작할 수 있습니다
    
    기본 명령이 비활성화되면 내장 명령이 제거됩니다. 사용자 정의/plugin 명령은 구성되어 있다면 여전히 등록될 수 있습니다.
    
    일반적인 설정 실패:
    
    - `setMyCommands failed`는 일반적으로 `api.telegram.org`로의 아웃바운드 DNS/HTTPS가 차단되었음을 의미합니다.
    
    ### 기기 페어링 명령 (`device-pair` plugin)
    
    `device-pair` plugin이 설치된 경우:
    
    1. `/pair`로 설정 코드를 생성합니다
    2. iOS 앱에 코드를 붙여넣습니다
    3. `/pair approve`로 가장 최근의 대기 중 요청을 승인합니다
    
    자세한 내용: [페어링](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    인라인 키보드 범위를 구성하세요:


```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    계정별 구성:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `allowlist` — 다이렉트 메시지 + 그룹, 단 `allowFrom`/`groupAllowFrom` 에 의해 허용된 발신자만 가능(제어 명령과 동일한 규칙)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    사용자가 버튼을 클릭하면 콜백 데이터가 다음 형식의 메시지로 에이전트에 전달됩니다:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram 도구 작업에는 다음이 포함됩니다:


    ```
    - `sendMessage` (`to`, `content`, 선택적 `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    채널 메시지 작업은 간편한 별칭을 제공합니다(`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    게이팅 제어:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (기본값: 비활성화)
    
    리액션 제거 동작: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram 은 태그를 통한 선택적 스레드 응답을 지원합니다:

    ```
    - `[[reply_to_current]]`는 트리거한 메시지에 답장합니다
    - `[[reply_to:<id>]]`는 특정 Telegram 메시지 ID에 답장합니다
    
    `channels.telegram.replyToMode`는 처리 방식을 제어합니다:
    
    - `off` (기본값)
    - `first`
    - `all`
    
    참고: `off`는 암시적 답장 스레딩을 비활성화합니다. 명시적인 `[[reply_to_*]]` 태그는 여전히 적용됩니다.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">토픽 (포럼 슈퍼그룹)

    ```
    각 토픽이 분리되도록 Telegram 그룹 세션 키에 `:topic:<threadId>` 를 추가합니다.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">
    ### 오디오 메시지


    ```
    Telegram은 음성 메시지와 오디오 파일을 구분합니다.
    
    - 기본값: 오디오 파일 방식
    - 에이전트 응답에 `[[audio_as_voice]]` 태그를 포함하면 음성 메시지로 강제 전송
    
    메시지 작업 예시:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    Video messages (video vs video note)
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    비디오 노트는 캡션을 지원하지 않습니다. 제공된 메시지 텍스트는 별도로 전송됩니다.
    
    ### 스티커
    
    수신 스티커 처리:
    
    - 정적 WEBP: 다운로드 및 처리됨 (placeholder `<media:sticker>`)
    - 애니메이션 TGS: 건너뜀
    - 비디오 WEBM: 건너뜀
    
    스티커 컨텍스트 필드:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    스티커 캐시 파일:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    스티커는 (가능한 경우) 한 번만 설명되고 캐시되어 반복적인 비전 호출을 줄입니다.
    
    스티커 작업 활성화:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    스티커 수신
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    스티커 캐시
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Telegram API 로부터 `message_reaction` 업데이트를 수신

    ```
    활성화되면, OpenClaw는 다음과 같은 시스템 이벤트를 큐에 추가합니다:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    구성:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (기본값: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (기본값: `minimal`)
    
    참고:
    
    - `own`은 봇이 보낸 메시지에 대한 사용자 리액션만을 의미합니다(전송 메시지 캐시 기반, 최선 시도).
    - Telegram은 리액션 업데이트에 스레드 ID를 제공하지 않습니다.
      - 일반 그룹은 그룹 채팅 세션으로 라우팅됩니다
      - 포럼 그룹은 정확한 원본 토픽이 아닌 그룹 일반 토픽 세션(`:topic:1`)으로 라우팅됩니다
    
    폴링/웹훅의 `allowed_updates`에는 `message_reaction`이 자동으로 포함됩니다.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction`은 OpenClaw가 수신 메시지를 처리하는 동안 확인 이모지를 전송합니다.


    ```
    확인 순서:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - 에이전트 아이덴티티 이모지 대체값 (`agents.list[].identity.emoji`, 없으면 "👀")
    
    참고:
    
    - Telegram은 유니코드 이모지(예: "👀")를 기대합니다.
    - 채널 또는 계정에서 리액션을 비활성화하려면 `""`를 사용하세요.
    
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">
    채널 config 쓰기는 기본적으로 활성화되어 있습니다(`configWrites !== false`).


    ```
    그룹이 슈퍼그룹으로 업그레이드되고 Telegram 이 `migrate_to_chat_id` 을 발생시키는 경우(채팅 ID 변경). OpenClaw 는 `channels.telegram.groups` 을 자동으로 마이그레이션할 수 있습니다.
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">
    기본값: 롱 폴링.


    ```
    웹훅 모드: `channels.telegram.webhookUrl` 및 `channels.telegram.webhookSecret` 설정(선택적으로 `channels.telegram.webhookPath`).
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    발신 텍스트는 `channels.telegram.textChunkLimit` 으로 분할됩니다 (기본 4000).
    선택적 줄바꿈 분할: 길이 분할 전에 빈 줄(문단 경계)에서 분할하려면 `channels.telegram.chunkMode="newline"` 를 설정하십시오.
    미디어 다운로드/업로드는 `channels.telegram.mediaMaxMb` 로 제한됩니다 (기본 5).
    Telegram Bot API 요청은 `channels.telegram.timeoutSeconds` 후 타임아웃됩니다 (grammY 기준 기본 500).
    그룹 히스토리 컨텍스트는 `channels.telegram.historyLimit` (또는 `channels.telegram.accounts.*.historyLimit`)를 사용하며, `messages.groupChat.historyLimit` 로 폴백합니다. 비활성화하려면 `0` 를 설정하십시오 (기본 50).
    사용자별 오버라이드: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>다이렉트 메시지 히스토리는 `channels.telegram.dmHistoryLimit` (사용자 턴 수)로 제한할 수 있습니다.

    ```
    대상은 채팅 ID (`123456789`) 또는 사용자 이름 (`@name`)을 사용할 수 있습니다.
    ```

```bash
예시: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## 문제 해결

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    `channels.telegram.groups.*.requireMention=false` 를 설정한 경우 Telegram Bot API 의 **프라이버시 모드**를 비활성화해야 합니다.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - `channels.telegram.groups`가 존재하는 경우, 해당 그룹이 목록에 포함되어야 합니다(또는 `"*"` 포함)
    - 봇이 그룹에 속해 있는지 확인하세요
    - 건너뜀 사유는 `openclaw logs --follow`로 로그를 검토하세요
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    로그에 `setMyCommands failed` 가 표시되면 일반적으로 `api.telegram.org` 로의 아웃바운드 HTTPS/DNS 가 차단된 것입니다.
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + 사용자 정의 fetch/proxy 조합은 AbortSignal 타입 불일치 시 즉시 중단 동작을 유발할 수 있습니다.
    - 일부 호스트는 `api.telegram.org`를 IPv6로 먼저 해석합니다. IPv6 아웃바운드가 제대로 동작하지 않으면 Telegram API 실패가 간헐적으로 발생할 수 있습니다.
    - DNS 응답을 확인하세요:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

추가 도움말: [채널 문제 해결](/channels/troubleshooting).

## 구성 참조 (Telegram)

기본 참조:

- `channels.telegram.enabled`: 채널 시작 활성화/비활성화.

- `channels.telegram.botToken`: 봇 토큰(BotFather).

- `channels.telegram.tokenFile`: 파일 경로에서 토큰 읽기.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (기본값: 페어링).

- `channels.telegram.allowFrom`: 다이렉트 메시지 허용 목록(ID/사용자 이름). `open` 은 `"*"` 가 필요합니다. `openclaw doctor --fix`는 레거시 `@username` 항목을 ID로 변환할 수 있습니다.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (기본값: 허용 목록).

- `channels.telegram.groupAllowFrom`: 그룹 발신자 허용 목록(ID/사용자 이름). `openclaw doctor --fix`는 레거시 `@username` 항목을 ID로 변환할 수 있습니다.

- `channels.telegram.groups`: 그룹별 기본값 + 허용 목록(`"*"` 로 전역 기본값 사용).
  - `channels.telegram.groups.<id>.groupPolicy`: groupPolicy (`open | allowlist | disabled`) 에 대한 그룹별 오버라이드.
  - `channels.telegram.groups.<id>.requireMention`: 언급 게이팅 기본값.
  - `channels.telegram.groups.<id>.skills`: 스킬 필터(생략 = 모든 스킬, 빈 값 = 없음).
  - `channels.telegram.groups.<id>.allowFrom`: 그룹별 발신자 허용 목록 오버라이드.
  - `channels.telegram.groups.<id>.systemPrompt`: 그룹에 대한 추가 시스템 프롬프트.
  - `channels.telegram.groups.<id>.enabled`: `false` 일 때 그룹 비활성화.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: 토픽별 오버라이드(그룹과 동일한 필드).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: groupPolicy (`open | allowlist | disabled`) 에 대한 토픽별 오버라이드.
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: 토픽별 언급 게이팅 오버라이드.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (기본값: 허용 목록).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: 계정별 오버라이드.

- `channels.telegram.replyToMode`: `off | first | all` (기본값: `first`).

- `channels.telegram.textChunkLimit`: 발신 청크 크기(문자).

- `channels.telegram.chunkMode`: `length` (기본값) 또는 길이 분할 전에 빈 줄(문단 경계)에서 분할하는 `newline`.

- `channels.telegram.linkPreview`: 발신 메시지의 링크 미리보기 토글(기본값: true).

- `channels.telegram.streamMode`: `off | partial | block` (초안 스트리밍).

- `channels.telegram.mediaMaxMb`: 수신/발신 미디어 한도(MB).

- `channels.telegram.retry`: 아웃바운드 Telegram API 호출에 대한 재시도 정책(시도 횟수, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: Node autoSelectFamily 오버라이드(true=활성화, false=비활성화). Node 22 에서는 Happy Eyeballs 타임아웃을 피하기 위해 기본적으로 비활성화됩니다.

- `channels.telegram.proxy`: Bot API 호출을 위한 프록시 URL (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: 웹훅 모드 활성화(`channels.telegram.webhookSecret` 필요).

- `channels.telegram.webhookSecret`: 웹훅 시크릿(웹훅 URL 이 설정된 경우 필수).

- `channels.telegram.webhookPath`: 로컬 웹훅 경로(기본값 `/telegram-webhook`).

- 로컬 리스너는 `0.0.0.0:8787` 에 바인딩되며 기본적으로 `POST /telegram-webhook` 를 제공합니다.

- `channels.telegram.actions.reactions`: Telegram 도구 반응 게이팅.

- `channels.telegram.actions.sendMessage`: Telegram 도구 메시지 전송 게이팅.

- `channels.telegram.actions.deleteMessage`: Telegram 도구 메시지 삭제 게이팅.

- `channels.telegram.actions.sticker`: Telegram 스티커 액션(전송 및 검색) 게이팅(기본값: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — 어떤 반응이 시스템 이벤트를 트리거하는지 제어(설정되지 않으면 기본값: `own`).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — 에이전트의 반응 기능 제어(설정되지 않으면 기본값: `minimal`).

- [Configuration reference - Telegram](/gateway/configuration-reference#telegram)

Telegram 전용 고신호(high-signal) 필드:

- startup/auth: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- access control: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- command/menu: `commands.native`, `customCommands`
- threading/replies: `replyToMode`
- 선택 사항(`streamMode: "block"` 에만 해당):
- formatting/delivery: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- media/network: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- webhook: `webhookUrl`, `webhookSecret`, `webhookPath`, `webhookHost`
- actions/capabilities: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- 반응 알림
- writes/history: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## 관련 항목

- 자세한 내용: [페어링](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- 설정 문제 해결 (명령어)

