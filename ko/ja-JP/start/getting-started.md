---
read_when:
  - 처음부터 시작하는 초기 설정
  - 작동하는 채팅으로 가는 가장 빠른 방법을 알고 싶습니다
summary: OpenClaw를 설치하고 몇 분 만에 첫 번째 채팅을 실행해 보세요.
title: 시작하기
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# 시작하기

목표: 처음부터 최소한의 설정으로 첫 번째 작동하는 채팅을 실행합니다.

<Info>
가장 빠른 채팅 방법: Control UI를 엽니다(채널 설정 불필요). `openclaw dashboard`를 실행하여 브라우저에서 채팅하거나,
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gateway 호스트</Tooltip>에서 `http://127.0.0.1:18789/`를 엽니다.
문서: [Dashboard](/web/dashboard) 및 [Control UI](/web/control-ui).
</Info>

## 사전 요구 사항

- Node 22 이상

<Tip>
확실하지 않은 경우 `node --version`으로 Node 버전을 확인하세요.
</Tip>

## 빠른 설정 (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    기타 설치 방법 및 요구 사항: [설치](/install).
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    마법사는 인증, Gateway 설정 및 선택적 채널을 구성합니다.
    자세한 내용은 [온보딩 마법사](/start/wizard)를 참조하세요.
    ```

  
</Step>
  <Step title="Gatewayを確認">
    서비스를 설치한 경우, 이미 실행 중이어야 합니다:

    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Control UI가 로드되면 Gateway를 사용할 준비가 된 것입니다.
</Check>

## 선택적 확인 및 추가 기능

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    빠른 테스트나 문제 해결에 유용합니다.


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    구성된 채널이 필요합니다.


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## 더 알아보기

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    전체 CLI 마법사 참조 및 고급 옵션.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    macOS 앱의 최초 실행 흐름.
   
</Card>
</Columns>

## 완료 후 상태

- 실행 중인 Gateway
- 구성된 인증
- Control UI 접근 또는 연결된 채널

## 다음 단계

- DM 보안 및 승인: [페어링](/channels/pairing)
- 추가 채널 연결: [채널](/channels)
- 고급 워크플로 및 소스에서 빌드: [설정](/start/setup)
