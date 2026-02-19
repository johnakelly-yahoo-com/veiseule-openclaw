---
summary: "OpenClaw 온보딩 옵션 및 흐름 개요"
read_when:
  - 온보딩 경로 선택
  - 새 환경 설정
title: "온보딩 개요"
sidebarTitle: "온보딩 개요"
---

# 온보딩 개요

OpenClaw는 Gateway 실행 위치와 provider 구성 방식에 따라 여러 온보딩 경로를 지원합니다.

## 온보딩 경로를 선택하세요

- macOS, Linux, Windows(WSL2 사용)를 위한 **CLI 마법사**.
- Apple silicon 또는 Intel Mac에서 가이드형 첫 실행을 위한 **macOS 앱**.

## CLI 온보딩 마법사

터미널에서 마법사를 실행하세요:

```bash
openclaw onboard
```

Gateway, 워크스페이스, 채널 및 Skills를 완전히 제어하려면 CLI 마법사를 사용하세요. 문서:

- [온보딩 마법사 (CLI)](/start/wizard)
- [`openclaw onboard` command](/cli/onboard)

## macOS 앱 온보딩

macOS에서 완전히 가이드된 설정을 원할 경우 OpenClaw 앱을 사용하세요. 문서:

- [Onboarding (macOS App)](/start/onboarding)

## Custom Provider

표시되지 않은 엔드포인트가 필요하거나, 표준 OpenAI 또는 Anthropic API를 노출하는 호스팅 제공자를 사용하는 경우 CLI 마법사에서 **Custom Provider**를 선택하세요. 다음 항목을 입력하라는 안내가 표시됩니다:

- OpenAI 호환, Anthropic 호환 또는 **Unknown**(자동 감지) 중에서 선택하세요.
- 기본 URL과 API 키(제공자가 요구하는 경우)를 입력하세요.
- 모델 ID와 선택적 별칭을 제공하세요.
- 여러 사용자 지정 엔드포인트가 공존할 수 있도록 Endpoint ID를 선택하세요.

자세한 단계는 위의 CLI 온보딩 문서를 따르세요.

