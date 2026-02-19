---
summary: "`openclaw onboard`에 대한 CLI 레퍼런스(대화형 온보딩 마법사)"
read_when:
  - Gateway(게이트웨이), 워크스페이스, 인증, 채널, Skills에 대한 안내식 설정이 필요할 때
title: "온보드"
---

# `openclaw onboard`

대화형 온보딩 마법사(로컬 또는 원격 Gateway(게이트웨이) 설정).

## 관련 가이드

- CLI 온보딩 허브: [Onboarding Wizard (CLI)](/start/wizard)
- 온보딩 개요: [Onboarding Overview](/start/onboarding-overview)
- CLI 자동화: [CLI Automation](/start/wizard-cli-automation)
- CLI 온보딩 레퍼런스: [CLI Onboarding Reference](/start/wizard-cli-reference)
- macOS 온보딩: [Onboarding (macOS App)](/start/onboarding)

## 예제

```bash
openclaw onboard
openclaw onboard --flow quickstart
openclaw onboard --flow manual
openclaw onboard --mode remote --remote-url ws://gateway-host:18789
```

비대화형 사용자 지정 provider:

```bash
openclaw onboard --non-interactive \
  --auth-choice custom-api-key \
  --custom-base-url "https://llm.example.com/v1" \
  --custom-model-id "foo-large" \
  --custom-api-key "$CUSTOM_API_KEY" \
  --custom-compatibility openai
```

`--custom-api-key`는 비대화형 모드에서 선택 사항입니다. 생략하면 온보딩에서 `CUSTOM_API_KEY`를 확인합니다.

비대화형 Z.AI 엔드포인트 선택:

참고: `--auth-choice zai-api-key`는 이제 키에 가장 적합한 Z.AI 엔드포인트를 자동 감지합니다 (`zai/glm-5`가 포함된 일반 API를 우선 사용).
GLM Coding Plan 엔드포인트를 특정해서 사용하려면 `zai-coding-global` 또는 `zai-coding-cn`을 선택하세요.

```bash
# 프롬프트 없는 엔드포인트 선택
openclaw onboard --non-interactive \
  --auth-choice zai-coding-global \
  --zai-api-key "$ZAI_API_KEY"

# 기타 Z.AI 엔드포인트 선택:
# --auth-choice zai-coding-cn
# --auth-choice zai-global
# --auth-choice zai-cn
```

플로우 참고 사항:

- `quickstart`: 최소한의 프롬프트, Gateway(게이트웨이) 토큰을 자동 생성합니다.
- `manual`: 포트/바인드/인증에 대한 전체 프롬프트(`advanced`의 별칭).
- 가장 빠른 첫 채팅: `openclaw dashboard` (Control UI, 채널 설정 없음).
- Custom Provider: 나열되지 않은 호스팅 provider를 포함하여, OpenAI 또는 Anthropic 호환 엔드포인트에 연결합니다. Unknown을 사용하여 자동 감지하세요.

## 일반적인 후속 명령

```bash
openclaw configure
openclaw agents add <name>
```

<Note>
`--json`는 비대화형 모드를 의미하지 않습니다. 스크립트에는 `--non-interactive`를 사용하십시오.
</Note>
