---
summary: "imsg를 통한 레거시 iMessage 지원 (stdio 상의 JSON-RPC). 새로운 설정에서는 BlueBubbles 사용을 권장합니다."
read_when:
  - iMessage 지원 설정
  - iMessage 송수신 디버깅
title: "iMessage"
---

# iMessage (레거시: imsg)

<Warning>
**권장:** 새로운 iMessage 설정에는 [BlueBubbles](/channels/bluebubbles)를 사용하십시오.

`imsg` 채널은 레거시 외부 CLI 통합이며, 향후 릴리스에서 제거될 수 있습니다. 
</Warning>

상태: 레거시 외부 CLI 통합. Gateway(게이트웨이)는 `imsg rpc` (stdio 상의 JSON-RPC)를 스폰합니다.

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    새로운 설정에 권장되는 iMessage 경로입니다.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage DM은 기본적으로 페어링 모드를 사용합니다.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    iMessage 전체 필드 참조입니다.
  
</Card>
</CardGroup>

## 빠른 설정 (초보자)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="OpenClaw 구성">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="gateway 시작">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="첫 번째 DM 페어링 승인 (기본 dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            페어링 요청은 1시간 후 만료됩니다.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">`channels.imessage.cliPath` 는 stdin/stdout 을 프록시하는 어떤 명령이든 가리킬 수 있습니다(예: 다른 Mac 으로 SSH 한 뒤 `imsg rpc` 를 실행하는 래퍼 스크립트).

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    첨부 파일이 활성화된 경우 권장 설정:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    `remoteHost` 가 설정되지 않은 경우, OpenClaw 는 래퍼 스크립트의 SSH 명령을 파싱하여 자동 감지를 시도합니다.
    ```

  
</Tab>
</Tabs>

## 요구 사항 및 권한 (macOS)

- Messages 에 로그인된 macOS.
- OpenClaw + `imsg` 에 대한 전체 디스크 접근 권한(Messages DB 접근).
- Messages.app을 통해 메시지를 전송하려면 Automation 권한이 필요합니다.

<Tip>
권한은 프로세스 컨텍스트별로 부여됩니다. gateway가 헤드리스(LaunchAgent/SSH)로 실행되는 경우, 동일한 컨텍스트에서 프롬프트를 트리거하기 위해 일회성 인터랙티브 명령을 실행하세요:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## 액세스 제어 및 라우팅

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy`는 다이렉트 메시지를 제어합니다:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (기본값: 허용 목록).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`allowlist` 가 설정된 경우, `channels.imessage.groupAllowFrom` 이 그룹에서 트리거할 수 있는 사용자를 제어합니다.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DM은 직접 라우팅을 사용하고, 그룹은 그룹 라우팅을 사용합니다.
    - 기본값 `session.dmScope=main`에서는 iMessage DM이 에이전트 메인 세션으로 통합됩니다.
    - 그룹 세션은 격리됩니다 (`agent:<agentId>그룹:<chat_id>`).
    - 답장은 원래의 채널/대상 메타데이터를 사용해 iMessage로 다시 라우팅됩니다.

    ```
    여러 참여자가 있는 스레드가 `is_group=false` 와 함께 도착하더라도, `channels.imessage.groups` 를 사용하여 `chat_id` 으로 여전히 분리할 수 있습니다(아래 '그룹 유사 스레드' 참고).
    ```

  
</Tab>
</Tabs>

## 배포 패턴

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">봇이 **별도의 iMessage 아이덴티티**로 전송하도록 하여 개인 Messages 를 깔끔하게 유지하려면, 전용 Apple ID 와 전용 macOS 사용자를 사용하십시오.

    ```
    다른 Mac 에서 iMessage 를 사용하려면, `channels.imessage.cliPath` 을 SSH 를 통해 원격 macOS 호스트에서 `imsg` 를 실행하는 래퍼로 설정하십시오. OpenClaw 는 stdio 만 필요합니다.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    일반적인 토폴로지:


    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    `ssh bot@mac-mini.tailnet-1234.ts.net` 가 프롬프트 없이 동작하도록 SSH 키를 사용하십시오.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">macOS 에서 `imsg` 로 구동되는 iMessage 채널입니다.

    ```
    각 계정은 `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb`, 히스토리 설정과 같은 필드를 재정의할 수 있습니다.
    ```

  
</Accordion>
</AccordionGroup>

## 미디어, 청크 처리 및 전송 대상

<AccordionGroup>
  <Accordion title="Attachments and media">미디어 업로드는 `channels.imessage.mediaMaxMb` 로 제한됩니다(기본값 16).
</Accordion>

  <Accordion title="Outbound chunking">`channels.imessage.chunkMode`: 길이 기준 청크 처리 전에 빈 줄(문단 경계)에서 분할하기 위한 `length` (기본값) 또는 `newline`.
</Accordion>

  <Accordion title="Addressing formats">
    권장되는 명시적 대상:


    ```
    - `chat_id:123` (안정적인 라우팅을 위해 권장)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    핸들 대상도 지원됩니다:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## 설정 쓰기

기본적으로 iMessage 는 `/config set|unset` 에 의해 트리거되는 설정 업데이트 쓰기를 허용합니다(`commands.config: true` 필요).

비활성화하려면:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## 문제 해결

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    바이너리 및 RPC 지원을 확인하세요:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    probe에서 RPC unsupported가 보고되면 `imsg`를 업데이트하세요.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">체크리스트:

    ```
    `channels.imessage.dmPolicy`: `pairing | allowlist | open | disabled` (기본값: 페어링).
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">참고:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">승인 방법:

    ```
    `channels.imessage.accounts.bot.cliPath` 을 봇 사용자로 `imsg` 를 실행하는 SSH 래퍼로 지정합니다.
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">
    동일한 사용자/세션 컨텍스트의 대화형 GUI 터미널에서 다시 실행하고 프롬프트를 승인하세요:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    Gateway(게이트웨이)를 시작하고 macOS 프롬프트(자동화 + 전체 디스크 접근)를 승인합니다.
    ```

  
</Accordion>
</AccordionGroup>

## 구성 참조 포인터

- iMessage 를 구성하고 Gateway(게이트웨이)를 시작합니다.
- 전체 설정: [Configuration](/gateway/configuration)
- 자세한 내용: [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)

