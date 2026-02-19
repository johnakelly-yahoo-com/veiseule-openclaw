---
read_when:
  - 온보딩 마법사 실행 또는 구성 시
  - 새 머신 설정 시
sidebarTitle: Wizard (CLI)
summary: "CLI 온보딩 마법사: Gateway, 워크스페이스, 채널, Skills의 대화형 설정"
title: 온보딩 마법사 (CLI)
x-i18n:
  generated_at: "2026-02-08T17:15:18Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 9a650d46044a930aa4aaec30b35f1273ca3969bf676ab67bf4e1575b5c46db4c
  source_path: start/wizard.md
  workflow: 15
---

# 온보딩 마법사 (CLI)

CLI 온보딩 마법사는 macOS, Linux, Windows(WSL2 경유)에서 OpenClaw를 설정할 때 권장되는 경로입니다. 로컬 Gateway 또는 원격 Gateway 연결과 함께 워크스페이스 기본 설정, 채널, Skills를 구성합니다.

```bash
openclaw onboard
```

<Info>
최대한 빠르게 첫 채팅을 시작하는 방법: Control UI를 엽니다(채널 설정 불필요). `openclaw dashboard`를 실행하면 브라우저에서 채팅할 수 있습니다. 문서: [Dashboard](/web/dashboard).
</Info>

## 빠른 시작 vs 상세 설정

마법사는 **빠른 시작**(기본 설정) 또는 **상세 설정**(완전한 제어) 중 하나를 선택하여 시작합니다.

<Tabs>
  <Tab title="クイックスタート（デフォルト設定）">
    - loopback 상의 로컬 Gateway
    - 기존 워크스페이스 또는 기본 워크스페이스
    - Gateway 포트 `18789`
    - Gateway 인증 토큰 자동 생성(loopback에서도 생성됨)
    - Tailscale 공개 비활성화
    - Telegram 및 WhatsApp DM은 기본적으로 허용 목록 방식(전화번호 입력을 요청받을 수 있음)
  
</Tab>
  <Tab title="詳細設定（完全な制御）">
    - 모드, 워크스페이스, Gateway, 채널, 데몬, Skills에 대한 전체 프롬프트 흐름 표시
  
</Tab>
</Tabs>

## CLI 온보딩 세부 정보

<Columns>
  <Card title="CLIリファレンス" href="/start/wizard-cli-reference">
    로컬 및 원격 흐름에 대한 전체 설명, 인증 및 모델 매트릭스, 설정 출력, 마법사 RPC, signal-cli 동작.
  
</Card>
  <Card title="自動化とスクリプト" href="/start/wizard-cli-automation">
    비대화형 온보딩 레시피와 자동화된 `agents add` 예시.
  
</Card>
</Columns>

## 자주 사용하는 후속 명령어

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json`은 비대화형 모드를 의미하지 않습니다. 스크립트에서는 `--non-interactive`를 사용하세요.
</Note>

<Tip>
권장: 에이전트가 `web_search`를 사용할 수 있도록 Brave Search API 키를 설정하세요(`web_fetch`는 키 없이 작동). 가장 쉬운 방법: `openclaw configure --section web`을 실행하면 `tools.web.search.apiKey`가 저장됩니다. 문서: [Web 도구](/tools/web).
</Tip>

## 관련 문서

- CLI 명령어 참조: [`openclaw onboard`](/cli/onboard)
- macOS 앱 온보딩: [온보딩](/start/onboarding)
- 에이전트 첫 실행 가이드: [에이전트 부트스트래핑](/start/bootstrapping)
