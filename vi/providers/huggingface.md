---
summary: "Thiết lập Hugging Face Inference (xác thực + chọn model)"
read_when:
  - Bạn muốn sử dụng Hugging Face Inference với OpenClaw
  - Bạn cần biến môi trường HF token hoặc lựa chọn xác thực CLI
title: "Hugging Face (Inference)"
---

# Hugging Face (Inference)

[Hugging Face Inference Providers](https://huggingface.co/docs/inference-providers) cung cấp chat completions tương thích OpenAI thông qua một router API duy nhất. Bạn có quyền truy cập vào nhiều model (DeepSeek, Llama và hơn thế nữa) chỉ với một token. OpenClaw sử dụng **endpoint tương thích OpenAI** (chỉ chat completions); đối với text-to-image, embeddings hoặc speech, hãy sử dụng trực tiếp [HF inference clients](https://huggingface.co/docs/api-inference/quicktour).

- Provider: `huggingface`
- Auth: `HUGGINGFACE_HUB_TOKEN` hoặc `HF_TOKEN` (fine-grained token với quyền **Make calls to Inference Providers**)
- API: Tương thích OpenAI (`https://router.huggingface.co/v1`)
- Thanh toán: Một token HF duy nhất; [pricing](https://huggingface.co/docs/inference-providers/pricing) theo mức giá của nhà cung cấp với gói miễn phí.

## Bắt đầu nhanh

1. Tạo một fine-grained token tại [Hugging Face → Settings → Tokens](https://huggingface.co/settings/tokens/new?ownUserPermissions=inference.serverless.write&tokenType=fineGrained) với quyền **Make calls to Inference Providers**.
2. Chạy onboarding và chọn **Hugging Face** trong menu thả xuống provider, sau đó nhập API key khi được yêu cầu:

```bash
openclaw onboard --auth-choice huggingface-api-key
```

3. Trong menu thả xuống **Default Hugging Face model**, chọn model bạn muốn (danh sách được tải từ Inference API khi bạn có token hợp lệ; nếu không, danh sách tích hợp sẵn sẽ được hiển thị). Lựa chọn của bạn sẽ được lưu làm model mặc định.
4. Bạn cũng có thể đặt hoặc thay đổi model mặc định sau trong cấu hình:

```json5
{
  agents: {
    defaults: {
      model: { primary: "huggingface/deepseek-ai/DeepSeek-R1" },
    },
  },
}
```

## Ví dụ không tương tác

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice huggingface-api-key \
  --huggingface-api-key "$HF_TOKEN"
```

Thao tác này sẽ đặt `huggingface/deepseek-ai/DeepSeek-R1` làm model mặc định.

## Lưu ý về môi trường

Nếu Gateway chạy dưới dạng daemon (launchd/systemd), hãy đảm bảo `HUGGINGFACE_HUB_TOKEN` hoặc `HF_TOKEN`
khả dụng cho tiến trình đó (ví dụ: trong `~/.openclaw/.env` hoặc thông qua
`env.shellEnv`).

## Dropdown khám phá và khởi tạo model

OpenClaw phát hiện model bằng cách gọi **Inference endpoint trực tiếp**:

```bash
GET https://router.huggingface.co/v1/models
```

(Tùy chọn: gửi `Authorization: Bearer $HUGGINGFACE_HUB_TOKEN` hoặc `$HF_TOKEN` để nhận danh sách đầy đủ; một số endpoint chỉ trả về danh sách rút gọn nếu không có xác thực.) Phản hồi có định dạng kiểu OpenAI `{ "object": "list", "data": [ { "id": "Qwen/Qwen3-8B", "owned_by": "Qwen", ... }, ... ] }`.

Khi bạn cấu hình Hugging Face API key (thông qua onboarding, `HUGGINGFACE_HUB_TOKEN`, hoặc `HF_TOKEN`), OpenClaw sẽ dùng lệnh GET này để phát hiện các model chat-completion khả dụng. Trong quá trình **onboarding tương tác**, sau khi nhập token, bạn sẽ thấy dropdown **Default Hugging Face model** được điền dữ liệu từ danh sách đó (hoặc từ danh mục tích hợp sẵn nếu yêu cầu thất bại). Khi chạy thực tế (ví dụ: khi Gateway khởi động), nếu có key, OpenClaw sẽ снова gọi **GET** `https://router.huggingface.co/v1/models` để làm mới danh mục. Danh sách này được hợp nhất với danh mục tích hợp sẵn (để bổ sung metadata như context window và chi phí). Nếu yêu cầu thất bại hoặc không thiết lập key, chỉ danh mục tích hợp sẵn được sử dụng.

## Tên model và các tùy chọn có thể chỉnh sửa

- **Tên từ API:** Tên hiển thị của model được **lấy từ GET /v1/models** khi API trả về `name`, `title` hoặc `display_name`; nếu không, nó sẽ được suy ra từ id của model (ví dụ: `deepseek-ai/DeepSeek-R1` → “DeepSeek R1”).
- **Ghi đè tên hiển thị:** Bạn có thể đặt nhãn tùy chỉnh cho từng model trong cấu hình để hiển thị theo ý muốn trong CLI và UI:

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

- **Lựa chọn provider / policy:** Thêm hậu tố vào **model id** để chọn cách router chọn backend:

  - **`:fastest`** — thông lượng cao nhất (router tự chọn; lựa chọn provider bị **khóa** — không có trình chọn backend tương tác).
  - **`:cheapest`** — chi phí thấp nhất trên mỗi output token (router tự chọn; lựa chọn provider bị **khóa**).
  - **`:provider`** — ép buộc một backend cụ thể (ví dụ: `:sambanova`, `:together`).

  Khi bạn chọn **:cheapest** hoặc **:fastest** (ví dụ: trong dropdown chọn model khi onboarding), provider sẽ bị khóa: router quyết định theo chi phí hoặc tốc độ và sẽ không hiển thị bước tùy chọn “ưu tiên backend cụ thể”. Bạn có thể thêm các mục này như các entry riêng biệt trong `models.providers.huggingface.models` hoặc đặt `model.primary` kèm hậu tố. Bạn cũng có thể thiết lập thứ tự mặc định trong [Inference Provider settings](https://hf.co/settings/inference-providers) (không có hậu tố = sử dụng thứ tự đó).

- **Hợp nhất cấu hình:** Các entry hiện có trong `models.providers.huggingface.models` (ví dụ: trong `models.json`) sẽ được giữ nguyên khi cấu hình được hợp nhất. Vì vậy, mọi `name`, `alias` hoặc tùy chọn model tùy chỉnh bạn đặt ở đó sẽ được сохранены.

## ID model và ví dụ cấu hình

Tham chiếu model sử dụng dạng `huggingface/<org>/<model>` (ID theo kiểu Hub). Danh sách dưới đây được lấy từ **GET** `https://router.huggingface.co/v1/models`; danh mục của bạn có thể bao gồm nhiều hơn.

**Ví dụ ID (từ inference endpoint):**

| Model                                  | Ref (thêm tiền tố `huggingface/`) |
| -------------------------------------- | ---------------------------------------------------- |
| DeepSeek R1                            | `deepseek-ai/DeepSeek-R1`                            |
| DeepSeek V3.2          | `deepseek-ai/DeepSeek-V3.2`                          |
| Qwen3 8B                               | `Qwen/Qwen3-8B`                                      |
| Qwen2.5 7B Instruct    | `Qwen/Qwen2.5-7B-Instruct`                           |
| Qwen3 32B                              | `Qwen/Qwen3-32B`                                     |
| Llama 3.3 70B Instruct | `meta-llama/Llama-3.3-70B-Instruct`                  |
| Llama 3.1 8B Instruct  | `meta-llama/Llama-3.1-8B-Instruct`                   |
| GPT-OSS 120B                           | `openai/gpt-oss-120b`                                |
| GLM 4.7                | `zai-org/GLM-4.7`                                    |
| Kimi K2.5              | `moonshotai/Kimi-K2.5`                               |

Bạn có thể thêm `:fastest`, `:cheapest` hoặc `:provider` (ví dụ: `:together`, `:sambanova`) vào id của model. Đặt thứ tự mặc định của bạn trong [Inference Provider settings](https://hf.co/settings/inference-providers); xem [Inference Providers](https://huggingface.co/docs/inference-providers) và **GET** `https://router.huggingface.co/v1/models` để xem danh sách đầy đủ.

### Ví dụ cấu hình đầy đủ

**DeepSeek R1 chính với Qwen dự phòng:**

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

**Qwen làm mặc định, với các biến thể :cheapest và :fastest:**

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

**DeepSeek + Llama + GPT-OSS với alias:**

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

**Buộc sử dụng một backend cụ thể với :provider:**

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

**Nhiều model Qwen và DeepSeek với hậu tố chính sách:**

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

