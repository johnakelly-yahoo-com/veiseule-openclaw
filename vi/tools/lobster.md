---
title: Lobster
description: Runtime workflow có kiểu cho OpenClaw — các pipeline có thể ghép nối với cổng phê duyệt.---

# Lobster

Lobster là một workflow shell cho phép OpenClaw chạy các chuỗi công cụ nhiều bước như một thao tác duy nhất, có tính quyết định, với các điểm kiểm soát phê duyệt rõ ràng.

## Móc câu

Trợ lý của bạn có thể xây dựng các công cụ để tự quản lý chính nó. Chỉ cần yêu cầu một quy trình làm việc, và 30 phút sau bạn sẽ có một CLI cùng các pipeline chạy như một lệnh duy nhất. Lobster là mảnh ghép còn thiếu: các pipeline xác định, phê duyệt rõ ràng và trạng thái có thể tiếp tục lại.

## Tại sao

Ngày nay, các quy trình làm việc phức tạp đòi hỏi nhiều lần gọi công cụ qua lại. Mỗi lần gọi đều tốn token, và LLM phải điều phối từng bước. Lobster chuyển phần điều phối đó vào một runtime có kiểu dữ liệu rõ ràng:

- **Một lần gọi thay vì nhiều lần**: OpenClaw chạy một lần gọi công cụ Lobster và nhận về kết quả có cấu trúc.
- **Phê duyệt tích hợp sẵn**: Các tác động phụ (gửi email, đăng bình luận) sẽ dừng workflow cho đến khi được phê duyệt rõ ràng.
- **Có thể tiếp tục**: Workflow bị dừng sẽ trả về một token; phê duyệt và tiếp tục mà không cần chạy lại mọi thứ.

## Vì sao dùng DSL thay vì chương trình thuần?

Lobster được thiết kế có chủ đích để nhỏ gọn. Mục tiêu không phải là "một ngôn ngữ mới," mà là một đặc tả pipeline có thể dự đoán được, thân thiện với AI, với cơ chế phê duyệt và resume token ở mức ưu tiên hàng đầu.

- **Phê duyệt/tiếp tục được tích hợp sẵn**: Một chương trình thông thường có thể hỏi ý kiến con người, nhưng không thể _tạm dừng và tiếp tục_ với một token bền vững nếu bạn không tự xây dựng runtime đó.
- **Tính quyết định + khả năng kiểm toán**: Pipeline là dữ liệu, nên dễ ghi log, so sánh diff, chạy lại và rà soát.
- **Bề mặt bị ràng buộc cho AI**: Ngữ pháp nhỏ gọn + truyền JSON giúp giảm các nhánh mã “sáng tạo” và khiến việc kiểm tra hợp lệ trở nên thực tế.
- **Chính sách an toàn được tích hợp**: Timeout, giới hạn đầu ra, kiểm tra sandbox và danh sách cho phép được runtime áp dụng, không phải từng script.
- **Vẫn có thể lập trình**: Mỗi bước có thể gọi bất kỳ CLI hoặc script nào. Nếu bạn muốn JS/TS, hãy tạo các tệp `.lobster` từ mã nguồn.

## Cách hoạt động

OpenClaw khởi chạy `lobster` CLI cục bộ ở **chế độ công cụ** và phân tích một JSON envelope từ stdout.
If the pipeline pauses for approval, the tool returns a `resumeToken` so you can continue later.

## Mẫu: CLI nhỏ + pipe JSON + phê duyệt

Hãy xây dựng các lệnh nhỏ giao tiếp bằng JSON, sau đó xâu chuỗi chúng thành một lần gọi Lobster duy nhất. (Tên lệnh bên dưới chỉ là ví dụ — hãy thay bằng tên của riêng bạn.)

```bash
inbox list --json
inbox categorize --json
inbox apply --json
```

```json
{
  "action": "run",
  "pipeline": "exec --json --shell 'inbox list --json' | exec --stdin json --shell 'inbox categorize --json' | exec --stdin json --shell 'inbox apply --json' | approve --preview-from-stdin --limit 5 --prompt 'Apply changes?'",
  "timeoutMs": 30000
}
```

Nếu pipeline yêu cầu phê duyệt, hãy tiếp tục với token:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

AI triggers the workflow; Lobster executes the steps. Approval gates keep side effects explicit and auditable.

Ví dụ: ánh xạ các mục đầu vào thành các lần gọi công cụ:

```bash
gog.gmail.search --query 'newer_than:1d' \
  | openclaw.invoke --tool message --action send --each --item-key message --args-json '{"provider":"telegram","to":"..."}'
```

## Các bước LLM chỉ dùng JSON (llm-task)

For workflows that need a **structured LLM step**, enable the optional
`llm-task` plugin tool and call it from Lobster. This keeps the workflow
deterministic while still letting you classify/summarize/draft with a model.

Bật công cụ:

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  },
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

Sử dụng trong một pipeline:

```lobster
openclaw.invoke --tool llm-task --action json --args-json '{
  "prompt": "Given the input email, return intent and draft.",
  "input": { "subject": "Hello", "body": "Can you help?" },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

Xem [LLM Task](/tools/llm-task) để biết chi tiết và các tùy chọn cấu hình.

## Tệp workflow (.lobster)

Lobster can run YAML/JSON workflow files with `name`, `args`, `steps`, `env`, `condition`, and `approval` fields. In OpenClaw tool calls, set `pipeline` to the file path.

```yaml
name: inbox-triage
args:
  tag:
    default: "family"
steps:
  - id: collect
    command: inbox list --json
  - id: categorize
    command: inbox categorize --json
    stdin: $collect.stdout
  - id: approve
    command: inbox apply --approve
    stdin: $categorize.stdout
    approval: required
  - id: execute
    command: inbox apply --execute
    stdin: $categorize.stdout
    condition: $approve.approved
```

Ghi chú:

- `stdin: $step.stdout` và `stdin: $step.json` truyền đầu ra của bước trước.
- `condition` (hoặc `when`) có thể chặn các bước dựa trên `$step.approved`.

## Cài đặt Lobster

Install the Lobster CLI on the **same host** that runs the OpenClaw Gateway (see the [Lobster repo](https://github.com/openclaw/lobster)), and ensure `lobster` is on `PATH`.
If you want to use a custom binary location, pass an **absolute** `lobsterPath` in the tool call.

## Bật công cụ

Lobster là một công cụ plugin **tùy chọn** (không bật theo mặc định).

Khuyến nghị (bổ sung, an toàn):

```json
{
  "tools": {
    "alsoAllow": ["lobster"]
  }
}
```

Hoặc theo từng tác tử:

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": {
          "alsoAllow": ["lobster"]
        }
      }
    ]
  }
}
```

Tránh dùng `tools.allow: ["lobster"]` trừ khi bạn chủ ý chạy ở chế độ danh sách cho phép hạn chế.

Note: allowlists are opt-in for optional plugins. If your allowlist only names
plugin tools (like `lobster`), OpenClaw keeps core tools enabled. To restrict core
tools, include the core tools or groups you want in the allowlist too.

## Ví dụ: phân loại email

Không dùng Lobster:

```
User: "Check my email and draft replies"
→ openclaw calls gmail.list
→ LLM summarizes
→ User: "draft replies to #2 and #5"
→ LLM drafts
→ User: "send #2"
→ openclaw calls gmail.send
(repeat daily, no memory of what was triaged)
```

Dùng Lobster:

```json
{
  "action": "run",
  "pipeline": "email.triage --limit 20",
  "timeoutMs": 30000
}
```

Trả về một phong bì JSON (đã lược bớt):

```json
{
  "ok": true,
  "status": "needs_approval",
  "output": [{ "summary": "5 need replies, 2 need action" }],
  "requiresApproval": {
    "type": "approval_request",
    "prompt": "Send 2 draft replies?",
    "items": [],
    "resumeToken": "..."
  }
}
```

Người dùng phê duyệt → tiếp tục:

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

One workflow. Deterministic. Safe.

## Tham số công cụ

### `run`

Chạy một pipeline ở chế độ tool.

```json
{
  "action": "run",
  "pipeline": "gog.gmail.search --query 'newer_than:1d' | email.triage",
  "cwd": "/path/to/workspace",
  "timeoutMs": 30000,
  "maxStdoutBytes": 512000
}
```

Chạy một tệp workflow với đối số:

```json
{
  "action": "run",
  "pipeline": "/path/to/inbox-triage.lobster",
  "argsJson": "{\"tag\":\"family\"}"
}
```

### `resume`

Tiếp tục một workflow đã bị dừng sau khi được phê duyệt.

```json
{
  "action": "resume",
  "token": "<resumeToken>",
  "approve": true
}
```

### Đầu vào tùy chọn

- `lobsterPath`: Đường dẫn tuyệt đối tới binary Lobster (bỏ qua để dùng `PATH`).
- `cwd`: Thư mục làm việc cho pipeline (mặc định là thư mục làm việc hiện tại của tiến trình).
- `timeoutMs`: Dừng tiến trình con nếu vượt quá thời lượng này (mặc định: 20000).
- `maxStdoutBytes`: Dừng tiến trình con nếu stdout vượt quá kích thước này (mặc định: 512000).
- `argsJson`: Chuỗi JSON truyền vào `lobster run --args-json` (chỉ cho tệp workflow).

## Phong bì đầu ra

Lobster trả về một phong bì JSON với một trong ba trạng thái:

- `ok` → hoàn thành thành công
- `needs_approval` → tạm dừng; cần `requiresApproval.resumeToken` để tiếp tục
- `cancelled` → bị từ chối hoặc hủy một cách rõ ràng

Công cụ hiển thị phong bì ở cả `content` (JSON đẹp) và `details` (đối tượng thô).

## Phê duyệt

Nếu `requiresApproval` xuất hiện, hãy xem prompt và quyết định:

- `approve: true` → tiếp tục và cho phép các tác động phụ
- `approve: false` → hủy và kết thúc workflow

Use `approve --preview-from-stdin --limit N` to attach a JSON preview to approval requests without custom jq/heredoc glue. Resume tokens are now compact: Lobster stores workflow resume state under its state dir and hands back a small token key.

## OpenProse

2. OpenProse kết hợp rất tốt với Lobster: dùng `/prose` để điều phối chuẩn bị đa tác tử, sau đó chạy một pipeline Lobster cho các phê duyệt mang tính quyết định. 3. Nếu một chương trình Prose cần Lobster, hãy cho phép công cụ `lobster` cho các tác tử con thông qua `tools.subagents.tools`. 4. Xem [OpenProse](/prose).

## An toàn

- **Chỉ chạy tiến trình cục bộ** — không có gọi mạng từ chính plugin.
- **Không có bí mật** — Lobster không quản lý OAuth; nó gọi các công cụ OpenClaw làm việc đó.
- **Nhận biết sandbox** — bị vô hiệu hóa khi ngữ cảnh công cụ bị sandbox.
- **Được gia cố** — `lobsterPath` phải là đường dẫn tuyệt đối nếu được chỉ định; timeout và giới hạn đầu ra luôn được áp dụng.

## Xử lý sự cố

- **`lobster subprocess timed out`** → tăng `timeoutMs`, hoặc chia nhỏ pipeline dài.
- **`lobster output exceeded maxStdoutBytes`** → tăng `maxStdoutBytes` hoặc giảm kích thước đầu ra.
- **`lobster returned invalid JSON`** → đảm bảo pipeline chạy ở chế độ tool và chỉ in JSON.
- **`lobster failed (code …)`** → chạy cùng pipeline đó trong terminal để kiểm tra stderr.

## Tìm hiểu thêm

- [Plugins](/tools/plugin)
- [Viết công cụ plugin](/plugins/agent-tools)

## Case study: workflow cộng đồng

5. Một ví dụ công khai: một CLI “second brain” + các pipeline Lobster quản lý ba kho Markdown (cá nhân, đối tác, dùng chung). 6. CLI xuất JSON cho thống kê, danh sách inbox và quét mục cũ; Lobster xâu chuỗi các lệnh đó thành các workflow như `weekly-review`, `inbox-triage`, `memory-consolidation` và `shared-task-sync`, mỗi workflow có các cổng phê duyệt. 7. AI xử lý phần phán đoán (phân loại) khi có thể và quay về các quy tắc mang tính quyết định khi không có.

- Thread: [https://x.com/plattenschieber/status/2014508656335770033](https://x.com/plattenschieber/status/2014508656335770033)
- Repo: [https://github.com/bloomedai/brain-cli](https://github.com/bloomedai/brain-cli)

