---
summary: "Ollama (로컬 LLM 런타임)로 OpenClaw 실행"
read_when:
  - Ollama 를 통해 로컬 모델로 OpenClaw 를 실행하려는 경우
  - Ollama 설정 및 구성 가이드가 필요한 경우
title: "Ollama"
---

# Ollama

Ollama 는 머신에서 오픈 소스 모델을 쉽게 실행할 수 있게 해주는 로컬 LLM 런타임입니다. OpenClaw 는 Ollama 의 OpenAI 호환 API 와 통합되며, `OLLAMA_API_KEY` (또는 인증 프로파일)로 옵트인하고 명시적인 `models.providers.ollama` 항목을 정의하지 않으면 **도구 사용이 가능한 모델을 자동으로 검색**할 수 있습니다.

## 빠른 시작

1. Ollama 설치: [https://ollama.ai](https://ollama.ai)

2. 모델 가져오기:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. OpenClaw 에서 Ollama 활성화 (어떤 값이든 동작합니다. Ollama 는 실제 키를 요구하지 않습니다):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Ollama 모델 사용:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## 모델 검색 (암시적 프로바이더)

`OLLAMA_API_KEY` (또는 인증 프로파일)을 설정하고 `models.providers.ollama` 를 정의하지 **않으면**, OpenClaw 는 `http://127.0.0.1:11434` 에서 로컬 Ollama 인스턴스로부터 모델을 검색합니다:

- `/api/tags` 및 `/api/show` 를 조회합니다
- `tools` 기능을 보고하는 모델만 유지합니다
- 모델이 `thinking` 를 보고하면 `reasoning` 로 표시합니다
- 가능한 경우 `model_info["<arch>.context_length"]` 에서 `contextWindow` 를 읽습니다
- 컨텍스트 윈도우의 10× 값으로 `maxTokens` 를 설정합니다
- 모든 비용을 `0` 로 설정합니다

이를 통해 수동 모델 항목을 피하면서도 Ollama 의 기능과 카탈로그를 정렬된 상태로 유지할 수 있습니다.

사용 가능한 모델을 확인하려면:

```bash
ollama list
openclaw models list
```

새 모델을 추가하려면 Ollama 로 가져오기만 하면 됩니다:

```bash
ollama pull mistral
```

새 모델은 자동으로 검색되어 사용 가능해집니다.

`models.providers.ollama` 을 명시적으로 설정하면 자동 검색이 건너뛰어지며, 아래와 같이 모델을 수동으로 정의해야 합니다.

## 구성

### 기본 설정 (암시적 검색)

Ollama 를 활성화하는 가장 간단한 방법은 환경 변수를 사용하는 것입니다:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### 명시적 설정 (수동 모델)

다음과 같은 경우에는 명시적 구성을 사용하십시오:

- Ollama 가 다른 호스트/포트에서 실행되는 경우
- 특정 컨텍스트 윈도우나 모델 목록을 강제하려는 경우
- 도구 지원을 보고하지 않는 모델을 포함하려는 경우

```json5
{
  models: {
    providers: {
      ollama: {
        // Use a host that includes /v1 for OpenAI-compatible APIs
        baseUrl: "http://ollama-host:11434/v1",
        apiKey: "ollama-local",
        api: "openai-completions",
        models: [
          {
            id: "gpt-oss:20b",
            name: "GPT-OSS 20B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 8192,
            maxTokens: 8192 * 10
          }
        ]
      }
    }
  }
}
```

`OLLAMA_API_KEY` 이 설정되어 있으면, 프로바이더 항목에서 `apiKey` 를 생략할 수 있으며 OpenClaw 가 가용성 확인을 위해 이를 채웁니다.

### 사용자 지정 기본 URL (명시적 구성)

Ollama 가 다른 호스트나 포트에서 실행 중인 경우 (명시적 구성은 자동 검색을 비활성화하므로 모델을 수동으로 정의해야 합니다):

```json5
{
  models: {
    providers: {
      ollama: {
        apiKey: "ollama-local",
        baseUrl: "http://ollama-host:11434/v1",
      },
    },
  },
}
```

### 모델 선택

구성이 완료되면 모든 Ollama 모델을 사용할 수 있습니다:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "ollama/gpt-oss:20b",
        fallbacks: ["ollama/llama3.3", "ollama/qwen2.5-coder:32b"],
      },
    },
  },
}
```

## 고급

### 추론 모델

Ollama 가 `/api/show` 에서 `thinking` 를 보고하면 OpenClaw 는 해당 모델을 추론 가능 모델로 표시합니다:

```bash
ollama pull deepseek-r1:32b
```

### 모델 비용

Ollama 는 무료이며 로컬에서 실행되므로 모든 모델 비용은 $0 으로 설정됩니다.

### 스트리밍 구성

OpenClaw의 Ollama 통합은 기본적으로 **네이티브 Ollama API** (`/api/chat`)를 사용하며, 스트리밍과 도구 호출을 동시에 완전히 지원합니다. 특별한 설정은 필요하지 않습니다.

#### 레거시 OpenAI 호환 모드

대신 OpenAI 호환 엔드포인트를 사용해야 하는 경우(예: OpenAI 형식만 지원하는 프록시 뒤에 있는 경우) `api: "openai-completions"`을 명시적으로 설정하세요:

```json5
{
  models: {
    providers: {
      ollama: {
        baseUrl: "http://ollama-host:11434/v1",
        api: "openai-completions",
        apiKey: "ollama-local",
        models: [...]
      }
    }
  }
}
```

참고: OpenAI 호환 엔드포인트는 스트리밍과 도구 호출을 동시에 지원하지 않을 수 있습니다. Ollama 모델에 대해 `streaming: false` 를 명시적으로 설정하십시오([스트리밍 구성](#streaming-configuration) 참고)

### 컨텍스트 윈도우

자동으로 검색된 모델의 경우, OpenClaw 는 가능한 경우 Ollama 가 보고한 컨텍스트 윈도우를 사용하며, 그렇지 않으면 기본값으로 `8192` 를 사용합니다. 명시적 프로바이더 구성에서 `contextWindow` 및 `maxTokens` 를 재정의할 수 있습니다.

## 문제 해결

### Ollama 가 감지되지 않는 경우

Ollama 가 실행 중인지 확인하고, `OLLAMA_API_KEY` (또는 인증 프로파일)을 설정했는지, 그리고 명시적인 `models.providers.ollama` 항목을 정의하지 **않았는지** 확인하십시오:

```bash
ollama serve
```

또한 API 에 접근 가능한지 확인하십시오:

```bash
curl http://localhost:11434/api/tags
```

### 사용 가능한 모델이 없는 경우

OpenClaw 는 도구 지원을 보고하는 모델만 자동으로 검색합니다. 모델이 목록에 없다면 다음 중 하나를 수행하십시오:

- 도구 사용이 가능한 모델을 가져오거나
- `models.providers.ollama` 에서 모델을 명시적으로 정의하십시오.

모델을 추가하려면:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### 연결이 거부되는 경우

Ollama 가 올바른 포트에서 실행 중인지 확인하십시오:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## 참고

- [Model Providers](/concepts/model-providers) - 모든 프로바이더 개요
- [Model Selection](/concepts/models) - 모델 선택 방법
- [Configuration](/gateway/configuration) - 전체 구성 참조

