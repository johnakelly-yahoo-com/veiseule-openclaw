---
summary: "OpenClaw에서 NVIDIA의 OpenAI 호환 API 사용하기"
read_when:
  - OpenClaw에서 NVIDIA 모델을 사용하려는 경우
  - NVIDIA_API_KEY를 설정해야 합니다
title: "NVIDIA"
---

# NVIDIA

NVIDIA는 Nemotron 및 NeMo 모델을 위해 `https://integrate.api.nvidia.com/v1`에서 OpenAI 호환 API를 제공합니다. [NVIDIA NGC](https://catalog.ngc.nvidia.com/)에서 API 키로 인증하세요.

## CLI 설정

키를 한 번 export한 다음, 온보딩을 실행하고 NVIDIA 모델을 설정하세요:

```bash
export NVIDIA_API_KEY="nvapi-..."
openclaw onboard --auth-choice skip
openclaw models set nvidia/nvidia/llama-3.1-nemotron-70b-instruct
```

여전히 `--token`을 전달하는 경우, 해당 값이 셸 히스토리와 `ps` 출력에 남는다는 점을 기억하세요. 가능하면 환경 변수를 사용하는 것이 좋습니다.

## 설정 스니펫

```json5
{
  env: { NVIDIA_API_KEY: "nvapi-..." },
  models: {
    providers: {
      nvidia: {
        baseUrl: "https://integrate.api.nvidia.com/v1",
        api: "openai-completions",
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "nvidia/nvidia/llama-3.1-nemotron-70b-instruct" },
    },
  },
}
```

## 모델 ID

- `nvidia/llama-3.1-nemotron-70b-instruct` (기본값)
- `meta/llama-3.3-70b-instruct`
- `nvidia/mistral-nemo-minitron-8b-8k-instruct`

## 참고 사항

- OpenAI 호환 `/v1` 엔드포인트를 사용하며, NVIDIA NGC의 API 키를 사용하세요.
- `NVIDIA_API_KEY`가 설정되면 Provider가 자동으로 활성화되며, 정적 기본값(131,072토큰 컨텍스트 윈도우, 최대 4,096토큰)을 사용합니다.
