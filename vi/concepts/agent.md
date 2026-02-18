---
title: "Runtime tác tử"
---

# Runtime tác tử 🤖

OpenClaw chạy một runtime tác tử nhúng duy nhất, bắt nguồn từ **pi-mono**.

## Workspace (bắt buộc)

OpenClaw sử dụng một thư mục workspace tác tử duy nhất (`agents.defaults.workspace`) làm thư mục làm việc **duy nhất** (`cwd`) của tác tử cho công cụ và ngữ cảnh.

Khuyến nghị: dùng `openclaw setup` để tạo `~/.openclaw/openclaw.json` nếu thiếu và khởi tạo các tệp workspace.

Bố cục workspace đầy đủ + hướng dẫn sao lưu: [Agent workspace](/concepts/agent-workspace)

Nếu `agents.defaults.sandbox` được bật, các phiên không phải chính có thể ghi đè bằng
workspace theo từng phiên dưới `agents.defaults.sandbox.workspaceRoot` (xem
[Cấu hình Gateway](/gateway/configuration)).

## Tệp bootstrap (được chèn)

Bên trong `agents.defaults.workspace`, OpenClaw mong đợi các tệp có thể chỉnh sửa bởi người dùng sau:

- `AGENTS.md` — hướng dẫn vận hành + “bộ nhớ”
- `SOUL.md` — persona, ranh giới, giọng điệu
- `TOOLS.md` — ghi chú công cụ do người dùng duy trì (ví dụ `imsg`, `sag`, quy ước)
- `BOOTSTRAP.md` — nghi thức chạy lần đầu một lần (bị xóa sau khi hoàn tất)
- `IDENTITY.md` — tên/vibe/emoji của tác tử
- `USER.md` — hồ sơ người dùng + cách xưng hô ưa thích

Ở lượt đầu của một phiên mới, OpenClaw chèn trực tiếp nội dung của các tệp này vào ngữ cảnh tác tử.

`BOOTSTRAP.md` chỉ được tạo cho **workspace hoàn toàn mới** (không có tệp bootstrap nào khác). Large files are trimmed and truncated with a marker so prompts stay lean (read the file for full content).

Nếu một tệp bị thiếu, OpenClaw chèn một dòng đánh dấu “missing file” duy nhất (và `openclaw setup` sẽ tạo một mẫu mặc định an toàn).

ID phiên là ổn định và do OpenClaw chọn. If you delete it after completing the ritual, it should not be recreated on later restarts.

Để tắt hoàn toàn việc tạo tệp bootstrap (cho workspace đã được seed sẵn), đặt:

```json5
{ agent: { skipBootstrap: true } }
```

## Công cụ tích hợp sẵn

Các công cụ cốt lõi (read/exec/edit/write và các công cụ hệ thống liên quan) luôn sẵn sàng,
subject to tool policy. `apply_patch` is optional and gated by
`tools.exec.applyPatch`. `TOOLS.md` does **not** control which tools exist; it’s
guidance for how _you_ want them used.

## Kỹ năng

OpenClaw tải Skills từ ba vị trí (workspace thắng khi trùng tên):

- Bundled (đi kèm bản cài đặt)
- Được quản lý/cục bộ: `~/.openclaw/skills`
- Không gian làm việc: `<workspace>/skills`

Skills có thể bị kiểm soát bởi config/env (xem `skills` trong [Cấu hình Gateway](/gateway/configuration)).

## Tích hợp pi-mono

OpenClaw tái sử dụng một số phần của codebase pi-mono (mô hình/công cụ), nhưng **quản lý phiên, khám phá và wiring công cụ thuộc OpenClaw**.

- Không có runtime tác tử pi-coding.
- Không tham chiếu các thiết lập `~/.pi/agent` hoặc `<workspace>/.pi`.

## Phiên

Bản ghi phiên được lưu dưới dạng JSONL tại:

- `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl`

Khi chế độ hàng đợi là `steer`, các thông điệp đến sẽ được chèn vào lần chạy hiện tại.
Legacy Pi/Tau session folders are **not** read.

## Điều hướng khi streaming

Khi chế độ hàng đợi là `followup` hoặc `collect`, các thông điệp đến sẽ được giữ lại cho đến khi
lượt hiện tại kết thúc, sau đó một lượt agent mới bắt đầu với các payload đã xếp hàng.
The queue is checked **after each tool call**; if a queued message is present,
remaining tool calls from the current assistant message are skipped (error tool
results with "Skipped due to queued user message."), then the queued user
message is injected before the next assistant response.

Xem
[Queue](/concepts/queue) để biết hành vi theo chế độ + debounce/cap. Điều khiển việc chia khối stream mềm bằng `agents.defaults.blockStreamingChunk` (mặc định
800–1200 ký tự; ưu tiên ngắt đoạn, sau đó là xuống dòng; câu là lựa chọn cuối).

Block streaming sends completed assistant blocks as soon as they finish; it is
**off by default** (`agents.defaults.blockStreamingDefault: "off"`).
Tune the boundary via `agents.defaults.blockStreamingBreak` (`text_end` vs `message_end`; defaults to text_end).
Gộp các khối stream bằng `agents.defaults.blockStreamingCoalesce` để giảm spam dòng đơn (gộp dựa trên trạng thái rảnh trước khi gửi).
Các kênh không phải Telegram yêu cầu
`*.blockStreaming: true` một cách tường minh để bật phản hồi dạng khối. Non-Telegram channels require
explicit `*.blockStreaming: true` to enable block replies.
Chi tiết thêm: [Streaming + chunking](/concepts/streaming).
Mỗi mô hình đều có **cửa sổ ngữ cảnh** (số token tối đa mà nó có thể nhìn thấy).

## Tham chiếu mô hình

Các tham chiếu mô hình trong config (ví dụ `agents.defaults.model` và `agents.defaults.models`) được phân tích bằng cách tách theo `/` **đầu tiên**.

- Dùng `provider/model` khi cấu hình mô hình.
- Nếu ID mô hình tự nó chứa `/` (kiểu OpenRouter), hãy bao gồm tiền tố nhà cung cấp (ví dụ: `openrouter/moonshotai/kimi-k2`).
- Nếu bạn bỏ qua nhà cung cấp, OpenClaw coi đầu vào là một alias hoặc một mô hình cho **nhà cung cấp mặc định** (chỉ hoạt động khi không có `/` trong ID mô hình).

## Cấu hình (tối thiểu)

Ít nhất, hãy đặt:

- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom` (khuyến nghị mạnh)

---

_Tiếp theo: [Group Chats](/channels/group-messages)_ 🦞


