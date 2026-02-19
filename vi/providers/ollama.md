---
summary: "Chạy OpenClaw với Ollama (runtime LLM cục bộ)"
read_when:
  - Bạn muốn chạy OpenClaw với các mô hình cục bộ thông qua Ollama
  - Bạn cần hướng dẫn thiết lập và cấu hình Ollama
title: "Ollama"
---

# Ollama

Ollama is a local LLM runtime that makes it easy to run open-source models on your machine. OpenClaw tích hợp với API gốc của Ollama (`/api/chat`), hỗ trợ streaming và tool calling, và có thể **tự động phát hiện các mô hình hỗ trợ tool** khi bạn chọn sử dụng `OLLAMA_API_KEY` (hoặc hồ sơ xác thực) và không định nghĩa mục `models.providers.ollama` một cách tường minh.

## Khởi động nhanh

1. Cài đặt Ollama: [https://ollama.ai](https://ollama.ai)

2. Tải một mô hình:

```bash
ollama pull gpt-oss:20b
# or
ollama pull llama3.3
# or
ollama pull qwen2.5-coder:32b
# or
ollama pull deepseek-r1:32b
```

3. Bật Ollama cho OpenClaw (bất kỳ giá trị nào cũng được; Ollama không yêu cầu khóa thật):

```bash
# Set environment variable
export OLLAMA_API_KEY="ollama-local"

# Or configure in your config file
openclaw config set models.providers.ollama.apiKey "ollama-local"
```

4. Sử dụng các mô hình Ollama:

```json5
{
  agents: {
    defaults: {
      model: { primary: "ollama/gpt-oss:20b" },
    },
  },
}
```

## Khám phá mô hình (nhà cung cấp ngầm định)

Khi bạn đặt `OLLAMA_API_KEY` (hoặc một auth profile) và **không** định nghĩa `models.providers.ollama`, OpenClaw sẽ khám phá các mô hình từ instance Ollama cục bộ tại `http://127.0.0.1:11434`:

- Truy vấn `/api/tags` và `/api/show`
- Chỉ giữ lại các mô hình báo cáo khả năng `tools`
- Đánh dấu `reasoning` khi mô hình báo cáo `thinking`
- Đọc `contextWindow` từ `model_info["<arch>.context_length"]` khi có
- Đặt `maxTokens` bằng 10× cửa sổ ngữ cảnh
- Đặt mọi chi phí thành `0`

Cách này tránh việc phải khai báo mô hình thủ công trong khi vẫn giữ danh mục phù hợp với khả năng của Ollama.

Để xem các mô hình hiện có:

```bash
ollama list
openclaw models list
```

Để thêm một mô hình mới, chỉ cần tải nó bằng Ollama:

```bash
ollama pull mistral
```

Mô hình mới sẽ được tự động khám phá và sẵn sàng sử dụng.

Nếu bạn đặt `models.providers.ollama` một cách tường minh, quá trình tự động khám phá sẽ bị bỏ qua và bạn phải định nghĩa mô hình thủ công (xem bên dưới).

## Cấu hình

### Thiết lập cơ bản (khám phá ngầm định)

Cách đơn giản nhất để bật Ollama là thông qua biến môi trường:

```bash
export OLLAMA_API_KEY="ollama-local"
```

### Thiết lập tường minh (mô hình thủ công)

Sử dụng cấu hình tường minh khi:

- Ollama chạy trên máy chủ/cổng khác.
- Bạn muốn ép buộc cửa sổ ngữ cảnh hoặc danh sách mô hình cụ thể.
- Bạn muốn bao gồm các mô hình không báo cáo hỗ trợ tool.

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

Nếu `OLLAMA_API_KEY` được đặt, bạn có thể bỏ qua `apiKey` trong mục provider và OpenClaw sẽ tự điền để kiểm tra khả dụng.

### URL cơ sở tùy chỉnh (cấu hình tường minh)

Nếu Ollama đang chạy trên một máy chủ hoặc cổng khác (cấu hình tường minh sẽ vô hiệu hóa tự động khám phá, vì vậy hãy định nghĩa mô hình thủ công):

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

### Chọn mô hình

Sau khi cấu hình, tất cả các mô hình Ollama của bạn đều khả dụng:

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

## Nâng cao

### Mô hình suy luận

OpenClaw đánh dấu các mô hình có khả năng suy luận khi Ollama báo cáo `thinking` trong `/api/show`:

```bash
ollama pull deepseek-r1:32b
```

### Chi phí mô hình

Ollama miễn phí và chạy cục bộ, vì vậy mọi chi phí mô hình đều được đặt là $0.

### Cấu hình streaming

Tích hợp Ollama của OpenClaw sử dụng **API gốc của Ollama** (`/api/chat`) theo mặc định, hỗ trợ đầy đủ streaming và tool calling đồng thời. Không cần cấu hình đặc biệt.

#### Chế độ tương thích OpenAI (Legacy)

Nếu bạn cần sử dụng endpoint tương thích OpenAI thay thế (ví dụ: phía sau proxy chỉ hỗ trợ định dạng OpenAI), hãy đặt `api: "openai-completions"` một cách tường minh:

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

Lưu ý: Endpoint tương thích OpenAI có thể không hỗ trợ streaming và tool calling đồng thời. Bạn có thể cần tắt streaming với `params: { streaming: false }` trong cấu hình model.

### Tắt streaming cho các nhà cung cấp khác

For auto-discovered models, OpenClaw uses the context window reported by Ollama when available, otherwise it defaults to `8192`. You can override `contextWindow` and `maxTokens` in explicit provider config.

## Xử lý sự cố

### Cửa sổ ngữ cảnh

Đảm bảo Ollama đang chạy và bạn đã đặt `OLLAMA_API_KEY` (hoặc một auth profile), đồng thời bạn **không** định nghĩa mục `models.providers.ollama` một cách tường minh:

```bash
ollama serve
```

Và đảm bảo API có thể truy cập được:

```bash
curl http://localhost:11434/api/tags
```

### Không có mô hình khả dụng

OpenClaw only auto-discovers models that report tool support. If your model isn't listed, either:

- Tải một mô hình có khả năng dùng tool, hoặc
- Định nghĩa mô hình một cách tường minh trong `models.providers.ollama`.

Để thêm mô hình:

```bash
ollama list  # See what's installed
ollama pull gpt-oss:20b  # Pull a tool-capable model
ollama pull llama3.3     # Or another model
```

### Kết nối bị từ chối

Để thêm mô hình:

```bash
# Check if Ollama is running
ps aux | grep ollama

# Or restart Ollama
ollama serve
```

## Kết nối bị từ chối

- [Model Providers](/concepts/model-providers) - Tổng quan về tất cả các nhà cung cấp
- [Model Selection](/concepts/models) - Cách chọn mô hình
- [Configuration](/gateway/configuration) - Tham chiếu cấu hình đầy đủ
