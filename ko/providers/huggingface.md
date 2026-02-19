---
summary: "Hugging Face Inference 설정 (인증 + 모델 선택)"
read_when:
  - OpenClaw와 함께 Hugging Face Inference를 사용하려는 경우
  - HF 토큰 환경 변수 또는 CLI 인증 선택이 필요합니다.
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers)는 단일 라우터 API를 통해 OpenAI 호환 채팅 완성 기능을 제공합니다. 하나의 토큰으로 다양한 모델(DeepSeek, Llama 등)에 접근할 수 있습니다. OpenClaw는 **OpenAI 호환 엔드포인트**(채팅 완성 전용)만 사용합니다. 텍스트-이미지, 임베딩 또는 음성 기능은 [HF inference clients](https://huggingface.co/docs/api-inference/quicktour)를 직접 사용하세요.

- Provider: `huggingface`
- 인증: `HUGGINGFACE_HUB_TOKEN` 또는 `HF_TOKEN` (**Make calls to Inference Providers** 권한이 있는 세분화된 토큰)
- API: OpenAI 호환 (`https://router.huggingface.co/v1`)
- 요금: 단일 HF 토큰 사용; [pricing](https://huggingface.co/docs/inference-providers/pricing)은 제공업체 요율을 따르며 무료 티어가 포함됩니다.

## 빠른 시작

1. **Make calls to Inference Providers** 권한이 있는 세분화된 토큰을 [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained)에서 생성하세요.
2. 온보딩을 실행하고 provider 드롭다운에서 **Hugging Face**를 선택한 다음, 메시지가 표시되면 API 키를 입력하세요:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. **Default Hugging Face model** 드롭다운에서 원하는 모델을 선택하세요(유효한 토큰이 있으면 Inference API에서 목록을 불러오고, 그렇지 않으면 기본 제공 목록이 표시됩니다). 선택한 모델은 기본 모델로 저장됩니다.
4. 나중에 config에서 기본 모델을 설정하거나 변경할 수도 있습니다:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## 비대화형 예시

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

이 명령은 `huggingface/deepseek-ai/DeepSeek-R1`를 기본 모델로 설정합니다.

## 환경 참고 사항

Gateway가 데몬(launchd/systemd)으로 실행되는 경우, `HUGGINGFACE_HUB_TOKEN` 또는 `HF_TOKEN`이
해당 프로세스에서 사용 가능하도록 설정해야 합니다(예: `~/.openclaw/.env`에 추가하거나
`env.shellEnv`를 통해 설정).

## 모델 검색 및 온보딩 드롭다운

OpenClaw는 **Inference 엔드포인트를 직접 호출**하여 모델을 검색합니다:

```bash
GET https://router.huggingface.co/v1/models
```

(선택 사항: 전체 목록을 받으려면 `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` 또는 `$HF_TOKEN`을 전송하세요. 일부 엔드포인트는 인증 없이 호출할 경우 부분 목록만 반환합니다.) 응답은 OpenAI 형식의 `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Hugging Face API 키(온보딩, `HUGGINGFACE_HUB_TOKEN`, 또는 `HF_TOKEN`을 통해)를 구성하면 OpenClaw는 이 GET 요청을 사용해 사용 가능한 chat-completion 모델을 검색합니다. **대화형 온보딩** 중에 토큰을 입력하면 해당 목록(또는 요청 실패 시 내장 카탈로그)으로 채워진 **Default Hugging Face model** 드롭다운이 표시됩니다. 런타임(예: Gateway 시작 시)에도 키가 존재하면 OpenClaw는 카탈로그를 새로 고치기 위해 다시 **GET** `https://router.huggingface.co/v1/models`를 호출합니다. 이 목록은 내장 카탈로그(컨텍스트 윈도우 및 비용 등의 메타데이터 포함)와 병합됩니다. 요청이 실패하거나 키가 설정되지 않은 경우에는 내장 카탈로그만 사용됩니다.

## 모델 이름 및 편집 가능한 옵션

- **API에서 가져온 이름:** API가 `name`, `title`, 또는 `display_name`을 반환하면 모델 표시 이름은 **GET /v1/models**에서 가져와 적용됩니다(hydrated). 그렇지 않으면 모델 id에서 파생됩니다(예: `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **표시 이름 재정의:** CLI 및 UI에서 원하는 방식으로 표시되도록 모델별로 config에서 사용자 지정 레이블을 설정할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1 (fast)" },
        "huggingface/deepseek-ai/DeepSeek-R1:cheapest": { alias: "DeepSeek R1 (cheap)" },
      },
    },
  },
}
```

- **Provider / 정책 선택:** **model id**에 접미사를 추가하여 라우터가 백엔드를 선택하는 방식을 지정할 수 있습니다:

  - **`:fastest`** — 최고 처리량(라우터가 선택; provider 선택은 **고정** — 대화형 백엔드 선택기 없음).
  - **`:cheapest`** — 출력 토큰당 최저 비용(라우터가 선택; provider 선택은 **고정**).
  - **`:provider`** — 특정 백엔드를 강제 지정(예: `:sambanova`, `:together`).

  온보딩 모델 드롭다운 등에서 **:cheapest** 또는 **:fastest**를 선택하면 provider가 고정됩니다: 비용 또는 속도 기준으로 라우터가 결정하며, 선택적 “특정 백엔드 선호” 단계는 표시되지 않습니다. 이 항목들을 `models.providers.huggingface.models`에 별도 엔트리로 추가하거나, 접미사를 포함하여 `model.primary`를 설정할 수 있습니다. [Inference Provider settings](https://hf.co/settings/inference-providers)에서 기본 순서를 설정할 수도 있습니다(접미사가 없으면 해당 순서 사용).

- **Config 병합:** config를 병합할 때 `models.providers.huggingface.models`(예: `models.json`)의 기존 항목은 유지됩니다. 따라서 그곳에 설정한 사용자 지정 `name`, `alias`, 또는 모델 옵션은 그대로 보존됩니다.

## 모델 ID 및 구성 예시

모델 참조는 `huggingface/<org>/<model>` 형식(Hub 스타일 ID)을 사용합니다. 아래 목록은 **GET** `https://router.huggingface.co/v1/models`의 결과이며, 사용자 카탈로그에는 더 많은 항목이 포함될 수 있습니다.

**예시 ID(추론 엔드포인트 기준):**

| 모델                                     | Ref (`huggingface/` 접두사 포함) |
| -------------------------------------- | ---------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                      |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                    |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                     |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                               |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`            |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`             |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                          |
| GLM 4.7                | `zai-org/GLM-4.7`                              |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                         |

모델 ID에 `:fastest`, `:cheapest`, 또는 `:provider`(예: `:together`, `:sambanova`)를 추가할 수 있습니다. 기본 순서는 [Inference Provider settings](https://hf.co/settings/inference-providers)에서 설정하세요. 전체 목록은 [Inference Providers](https://huggingface.co/docs/inference-providers) 및 **GET** `https://router.huggingface.co/v1/models`를 참고하세요.

### 전체 구성 예시

**Qwen을 대체(fallback)로 사용하는 기본 DeepSeek R1:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-R1",
        fallbacks: ["huggingface/Qwen/Qwen3-8B"],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1": { alias: "DeepSeek R1" },
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
      },
    },
  },
}
```

**기본값으로 Qwen 사용, `:cheapest` 및 `:fastest` 변형 포함:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen3-8B" },
      models: {
        "huggingface/Qwen/Qwen3-8B": { alias: "Qwen3 8B" },
        "huggingface/Qwen/Qwen3-8B:cheapest": { alias: "Qwen3 8B (cheapest)" },
        "huggingface/Qwen/Qwen3-8B:fastest": { alias: "Qwen3 8B (fastest)" },
      },
    },
  },
}
```

**별칭(alias)과 함께 사용하는 DeepSeek + Llama + GPT-OSS:**

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "huggingface/deepseek-ai/DeepSeek-V3.2",
        fallbacks: [
          "huggingface/meta-llama/Llama-3.3-70B-Instruct",
          "huggingface/openai/gpt-oss-120b",
        ],
      },
      models: {
        "huggingface/deepseek-ai/DeepSeek-V3.2": { alias: "DeepSeek V3.2" },
        "huggingface/meta-llama/Llama-3.3-70B-Instruct": { alias: "Llama 3.3 70B" },
        "huggingface/openai/gpt-oss-120b": { alias: "GPT-OSS 120B" },
      },
    },
  },
}
```

**`:provider`로 특정 백엔드 강제 지정:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1:together" },
      models: {
        "huggingface/deepseek-ai/DeepSeek-R1:together": { alias: "DeepSeek R1 (Together)" },
      },
    },
  },
}
```

**정책 접미사를 사용하는 여러 Qwen 및 DeepSeek 모델:**

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest" },
      models: {
        "huggingface/Qwen/Qwen2.5-7B-Instruct": { alias: "Qwen2.5 7B" },
        "huggingface/Qwen/Qwen2.5-7B-Instruct:cheapest": { alias: "Qwen2.5 7B (cheap)" },
        "huggingface/deepseek-ai/DeepSeek-R1:fastest": { alias: "DeepSeek R1 (fast)" },
        "huggingface/meta-llama/Llama-3.1-8B-Instruct": { alias: "Llama 3.1 8B" },
      },
    },
  },
}
```
