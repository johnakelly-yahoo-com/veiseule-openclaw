---
summary: "WhatsApp 채널 지원, 접근 제어, 전송 동작 및 운영"
read_when:
  - WhatsApp/웹 채널 동작 또는 인박스 라우팅 작업 시
title: "WhatsApp"
---

# WhatsApp (웹 채널)

상태: Baileys를 통한 WhatsApp Web만 지원합니다. Gateway(게이트웨이)가 세션을 소유합니다.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    기본 DM 정책은 알 수 없는 발신자에 대해 페어링입니다.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    채널 간 진단 및 복구 플레이북.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    전체 채널 설정 패턴 및 예시.
  
</Card>
</CardGroup>

## 구성 빠른 맵

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    특정 계정의 경우:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
다음으로 승인합니다: `openclaw pairing approve whatsapp <code>` (`openclaw pairing list whatsapp`로 목록 확인).
```

    ```
    코드는 1시간 후 만료되며, 대기 요청은 채널당 최대 3개로 제한됩니다.
    ```

  
</Step>
</Steps>

<Note>
개인 WhatsApp을 분리하는 데 매우 유용합니다. WhatsApp Business를 설치하고 OpenClaw 번호를 그곳에 등록하십시오. (채널 메타데이터 및 온보딩 흐름은 해당 설정에 최적화되어 있지만, 개인 번호 설정도 지원됩니다.)
</Note>

## 배포 패턴

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    가장 깔끔한 운영 모드입니다:
  

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    온보딩은 개인 번호 모드를 지원하며, self-chat에 적합한 기본 구성을 작성합니다:
  

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">메시징 플랫폼 채널은 현재 OpenClaw 채널 아키텍처에서 WhatsApp Web 기반(`Baileys`)입니다.

    ```
    내장된 채팅 채널 레지스트리에는 별도의 Twilio WhatsApp 메시징 채널이 없습니다.
    ```

  
</Accordion>
</AccordionGroup>

## 런타임 모델

- Gateway는 WhatsApp 소켓과 재연결 루프를 관리합니다.
- 아웃바운드 전송을 하려면 대상 계정에 대해 활성화된 WhatsApp 리스너가 필요합니다.
- 상태/브로드캐스트 채팅은 무시됩니다.
- 직접 채팅은 DM 세션 규칙(`session.dmScope`; 기본값 `main`은 DM을 에이전트 메인 세션으로 통합함)을 사용합니다.
- 그룹은 `agent:<agentId>:whatsapp:group:<jid>` 세션으로 매핑됩니다.

## 접근 제어 및 활성화

<Tabs>
  <Tab title="DM policy">**다이렉트 메시지 정책**: `channels.whatsapp.dmPolicy`가 다이렉트 채팅 접근을 제어합니다(기본값: `pairing`).

    ```
    `channels.whatsapp.dmPolicy` (다이렉트 메시지 정책: pairing/allowlist/open/disabled).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    그룹 접근은 두 개의 레이어로 구성됩니다:

    ```
    `channels.whatsapp.groups` (그룹 허용 목록 + 멘션 게이팅 기본값; 모두 허용하려면 `"*"` 사용)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    기본적으로 그룹에서 답장하려면 멘션이 필요합니다.

    ```
    멘션 감지는 다음을 포함합니다:
    
    - 봇 아이덴티티에 대한 명시적 WhatsApp 멘션
    - 구성된 멘션 정규식 패턴(`agents.list[].groupChat.mentionPatterns`, 대체값 `messages.groupChat.mentionPatterns`)
    - 암묵적 봇 답장 감지(답장 발신자가 봇 아이덴티티와 일치)
    
    세션 수준 활성화 명령:
    
    - `/activation mention`
    - `/activation always`
    
    `activation`은 전역 설정이 아닌 세션 상태를 업데이트합니다. 이 명령은 소유자(owner)만 사용할 수 있습니다.
    ```

  
</Tab>
</Tabs>

## 개인 번호 및 셀프 채팅 동작

개인 WhatsApp 번호로 실행하는 경우 동일한 번호를 사용하고 `channels.whatsapp.selfChatMode`을 활성화하십시오.

- 셀프 채팅 턴에 대해서는 읽음 확인을 건너뜁니다
- 자기 자신에게 핑을 보내게 되는 mention-JID 자동 트리거 동작을 무시합니다
- 자기 자신 채팅 답장은 설정 시 기본적으로 `[{identity.name}]`를 사용합니다(그렇지 않으면 `[openclaw]`). `messages.responsePrefix`이 설정되지 않은 경우에 해당합니다.

## 메시지 정규화 및 컨텍스트

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    수신된 WhatsApp 메시지는 공통 인바운드 엔벨로프로 래핑됩니다.

    ````
    인용 답장이 존재하는 경우, 컨텍스트는 다음 형식으로 추가됩니다:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    사용 가능한 경우 답장 메타데이터 필드도 채워집니다 (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">미디어만 있는 인바운드 메시지는 플레이스홀더를 사용합니다:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    위치 및 연락처 payload는 라우팅 전에 텍스트 컨텍스트로 정규화됩니다.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    그룹의 경우, 처리되지 않은 메시지는 버퍼링되었다가 봇이 최종적으로 트리거될 때 컨텍스트로 주입될 수 있습니다.

    ```
    최근 _처리되지 않은_ 메시지(기본 50개)가 다음 아래에 삽입됩니다:
    `[Chat messages since your last reply - for context]` (이미 세션에 있는 메시지는 재주입되지 않음)
    ```

  
</Accordion>

  <Accordion title="Read receipts">기본적으로 Gateway는 수신된 WhatsApp 메시지를 수락하면 읽음(파란 체크)으로 표시합니다.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## 전송, 청크 분할 및 미디어

<AccordionGroup>
  <Accordion title="Text chunking">선택적 줄바꿈 청크 처리: `channels.whatsapp.chunkMode="newline"`을 설정하면 길이 기준 청크 처리 전에 빈 줄(문단 경계) 기준으로 분할합니다.
</Accordion>

  <Accordion title="Outbound media behavior">
    - 이미지, 비디오, 오디오(PTT 음성 노트) 및 문서 payload를 지원합니다
    - 음성 노트 호환성을 위해 `audio/ogg`는 `audio/ogg; codecs=opus`로 재작성됩니다
    - 비디오 전송 시 `gifPlayback: true`를 설정하여 애니메이션 GIF 재생을 지원합니다
    - 멀티미디어 답장 payload 전송 시 캡션은 첫 번째 미디어 항목에 적용됩니다
    - 미디어 소스는 HTTP(S), `file://`, 또는 로컬 경로를 사용할 수 있습니다
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - 인바운드 미디어 저장 한도: `channels.whatsapp.mediaMaxMb` (기본값 `50`)
    - 자동 응답에 대한 아웃바운드 미디어 한도: `agents.defaults.mediaMaxMb` (기본값 `5MB`)
    - 이미지는 제한에 맞도록 자동 최적화(리사이즈/품질 조정)됩니다
    - 미디어 전송 실패 시, 응답을 조용히 누락하는 대신 첫 번째 항목 대체로 텍스트 경고를 전송합니다
  
</Accordion>
</AccordionGroup>

## 확인(ACK) 리액션

`channels.whatsapp.ackReaction` (메시지 수신 시 자동 리액션: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

참고:

- 수신 확인 리액션 (수신 즉시 자동 반응)
- 실패는 로그에 기록되지만 일반적인 답장 전송을 차단하지는 않습니다
- 그룹 모드 `mentions`는 멘션으로 트리거된 턴에 반응하며, 그룹 활성화 `always`는 이 검사를 우회합니다
- WhatsApp은 `messages.ackReaction`을 무시하므로, 대신 `channels.whatsapp.ackReaction`을 사용하십시오.

## 다중 계정 및 자격 증명

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.messagePrefix` (인바운드 접두어; 계정별: `channels.whatsapp.accounts.<accountId>
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">로그아웃: `openclaw channels logout` (또는 `--account <id>`)는 WhatsApp 인증 상태를 삭제합니다(공유 `oauth.json`는 유지).<accountId>자격 증명은 `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`에 저장됩니다.
</Accordion>

  <Accordion title="Logout behavior">
    `openclaw channels logout --channel whatsapp [--account `<id>]`는 해당 계정의 WhatsApp 인증 상태를 초기화합니다.

    ```
    레거시 인증 디렉터리에서는 `oauth.json`은 유지되고 Baileys 인증 파일은 제거됩니다.
    ```

  
</Accordion>
</AccordionGroup>

## 도구, 액션 및 설정 쓰기

- 에이전트 도구 지원에는 WhatsApp 리액션 액션(`react`)이 포함됩니다.
- 동작:
  - \`channels.whatsapp.accounts.<accountId>
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## 문제 해결 (빠른)

<AccordionGroup>
  <Accordion title="Not linked (QR required)">증상: `channels status`에 `linked: false`가 표시되거나 'Not linked' 경고가 나타납니다.

    ````
    수정 방법:
    
    ```bash
    openclaw channels login --channel whatsapp
    openclaw channels status
    ```
    ````

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    증상: 연결된 계정이 반복적으로 연결 해제되거나 재연결을 시도합니다.

    ```
    해결: `openclaw doctor` (또는 Gateway 재시작). 지속되면 `channels login`로 재연결하고 `openclaw logs --follow`를 점검하십시오.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">
    대상 계정에 활성화된 gateway 리스너가 없으면 아웃바운드 전송은 즉시 실패합니다.

    ```
    gateway가 실행 중이며 계정이 연결되어 있는지 확인하세요.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    다음 순서로 확인하세요:

    ```
    그룹 정책: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (기본값 `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway 런타임은 Node를 사용해야 합니다. WhatsApp(Baileys)과 Telegram은 Bun에서 신뢰성이 낮습니다. **Node**로 Gateway를 실행하십시오.
  
</Accordion>
</AccordionGroup>

## 구성 참조 포인터

기본 참조:

- [Configuration reference - WhatsApp](/gateway/configuration-reference#whatsapp)

주요 WhatsApp 필드:

- access: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- delivery: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- 멀티 계정 로그인: `openclaw channels login --account <id>` (`<id>` = `accountId`)..enabled`, `accounts.<id>.authDir\`, 계정 수준 재정의
- operations: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- 세션 동작: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## 관련 항목

- [페어링](/channels/pairing)
- [채널 라우팅](/channels/channel-routing)
- 문제 해결 가이드: [Gateway 문제 해결](/gateway/troubleshooting).
