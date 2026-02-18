---
summary: "Sub-agent: tạo các lần chạy tác tử cô lập chạy song song và thông báo kết quả về chat yêu cầu"
read_when:
  - Bạn muốn thực hiện công việc nền/song song thông qua tác tử
  - Bạn đang thay đổi sessions_spawn hoặc chính sách công cụ sub-agent
title: "Tác tử phụ"
---

# Sub-Agents

Sub-agent cho phép bạn chạy các tác vụ nền mà không chặn cuộc hội thoại chính. Khi bạn tạo một sub-agent, nó chạy trong phiên cô lập riêng, thực hiện công việc của mình và thông báo kết quả trở lại cuộc trò chuyện khi hoàn tất.

**Các trường hợp sử dụng:**

- Nghiên cứu một chủ đề trong khi agent chính tiếp tục trả lời câu hỏi
- Chạy song song nhiều tác vụ dài (thu thập dữ liệu web, phân tích mã, xử lý tệp)
- Ủy quyền nhiệm vụ cho các agent chuyên biệt trong thiết lập đa agent

## Khởi động nhanh

Cách đơn giản nhất để sử dụng sub-agent là yêu cầu agent của bạn một cách tự nhiên:

> "Tạo một sub-agent để nghiên cứu ghi chú phát hành Node.js mới nhất"

Agent sẽ gọi công cụ `sessions_spawn` ở phía sau. Khi sub-agent hoàn tất, nó sẽ thông báo các phát hiện của mình trở lại cuộc trò chuyện của bạn.

Bạn cũng có thể chỉ định rõ các tùy chọn:

> "Tạo một sub-agent để phân tích nhật ký máy chủ từ hôm nay. Sử dụng gpt-5.2 và đặt thời gian chờ 5 phút."

## Cách hoạt động

<Steps>
  <Step title="Main agent spawns">
    Agent chính gọi `sessions_spawn` với mô tả nhiệm vụ. Lời gọi là **không chặn** — agent chính nhận lại `{ status: "accepted", runId, childSessionKey }` ngay lập tức.
  </Step>
  <Step title="Sub-agent runs in the background">Một phiên cô lập mới được tạo (`agent:<agentId>:subagent:<uuid>`) trên làn hàng đợi `subagent` chuyên dụng.</Step>
  <Step title="Result is announced">
    Khi sub-agent hoàn tất, nó sẽ thông báo các phát hiện của mình trở lại cuộc trò chuyện của người yêu cầu. Agent chính đăng một bản tóm tắt bằng ngôn ngữ tự nhiên.
  </Step>
  <Step title="Session is archived">
    Phiên sub-agent được tự động lưu trữ sau 60 phút (có thể cấu hình). Bản ghi hội thoại được bảo tồn.
  </Step>
</Steps>

<Tip>
Mỗi sub-agent có **ngữ cảnh** và mức sử dụng token **riêng** của mình. Đặt một mô hình rẻ hơn cho sub-agent để tiết kiệm chi phí — xem [Setting a Default Model](#setting-a-default-model) bên dưới.
</Tip>

## Cấu hình

Sub-agent hoạt động ngay lập tức mà không cần cấu hình. Mặc định:

- Mô hình: lựa chọn mô hình thông thường của agent mục tiêu (trừ khi `subagents.model` được đặt)
- Tư duy: không có ghi đè cho sub-agent (trừ khi `subagents.thinking` được đặt)
- Số lượng đồng thời tối đa: 8
- Tự động lưu trữ: sau 60 phút

### Thiết lập Mô hình Mặc định

Sử dụng một mô hình rẻ hơn cho sub-agent để tiết kiệm chi phí token:

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
      },
    },
  },
}
```

### Thiết lập Mức độ Tư duy Mặc định

```json5
{
  agents: {
    defaults: {
      subagents: {
        thinking: "low",
      },
    },
  },
}
```

### Ghi đè theo từng agent

Trong thiết lập đa agent, bạn có thể đặt mặc định sub-agent cho từng agent:

```json5
{
  agents: {
    list: [
      {
        id: "researcher",
        subagents: {
          model: "anthropic/claude-sonnet-4",
        },
      },
      {
        id: "assistant",
        subagents: {
          model: "minimax/MiniMax-M2.1",
        },
      },
    ],
  },
}
```

### Đồng thời

Kiểm soát số lượng sub-agent có thể chạy đồng thời:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

Sub-agent sử dụng một làn hàng đợi riêng (`subagent`) tách biệt với hàng đợi agent chính, vì vậy các lần chạy sub-agent không chặn các phản hồi đến.

### Tự động lưu trữ

Các phiên sub-agent được tự động lưu trữ sau một khoảng thời gian có thể cấu hình:

```json5
{
  agents: {
    defaults: {
      subagents: {
        archiveAfterMinutes: 120, // default: 60
      },
    },
  },
}
```

<Note>
Việc lưu trữ sẽ đổi tên bản ghi hội thoại thành `*.deleted.<timestamp>` (cùng thư mục) — các bản ghi hội thoại được bảo toàn, không bị xóa. Bộ hẹn giờ tự động lưu trữ là theo cơ chế best-effort; các bộ hẹn giờ đang chờ sẽ bị mất nếu gateway khởi động lại.
</Note>

## Công cụ `sessions_spawn`

Đây là công cụ mà agent gọi để tạo sub-agent.

### Tham số

| Tham số             | Type                 | Default                                  | Description                                                                                                      |
| ------------------- | -------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `task`              | string               | _(bắt buộc)_          | Sub-agent nên thực hiện điều gì                                                                                  |
| `label`             | string               | —                                        | Nhãn ngắn để nhận diện                                                                                           |
| `agentId`           | string               | _(agent của bên gọi)_ | Tạo dưới một agent id khác (phải được cho phép)                                               |
| `mô hình`           | string               | _(tùy chọn)_          | Ghi đè model cho sub-agent này                                                                                   |
| `thinking`          | string               | _(tùy chọn)_          | Ghi đè mức độ suy nghĩ (`off`, `low`, `medium`, `high`, v.v.) |
| `runTimeoutSeconds` | số                   | `0` (không giới hạn)  | Hủy sub-agent sau N giây                                                                                         |
| `dọn dẹp`           | "delete" \\| "keep" | "keep"                                   | "delete" lưu trữ ngay sau khi thông báo                                                                          |

### Thứ tự phân giải model

Model của sub-agent được phân giải theo thứ tự sau (khớp đầu tiên được dùng):

1. Tham số `model` được chỉ định rõ trong lời gọi `sessions_spawn`
2. Cấu hình theo agent: `agents.list[].subagents.model`
3. Mặc định toàn cục: `agents.defaults.subagents.model`
4. Cơ chế phân giải model thông thường của agent mục tiêu cho phiên mới đó

Mức độ suy nghĩ được phân giải theo thứ tự sau:

1. Tham số `thinking` được chỉ định rõ trong lời gọi `sessions_spawn`
2. Cấu hình theo agent: `agents.list[].subagents.thinking`
3. Mặc định toàn cục: `agents.defaults.subagents.thinking`
4. Nếu không, sẽ không áp dụng ghi đè mức độ suy nghĩ riêng cho sub-agent

<Note>
Các giá trị model không hợp lệ sẽ bị bỏ qua một cách im lặng — sub-agent sẽ chạy với giá trị mặc định hợp lệ tiếp theo và có cảnh báo trong kết quả công cụ.</Note>

### Tạo sub-agent chéo giữa các agent

Theo mặc định, các sub-agent chỉ có thể được spawn dưới chính agent id của chúng. Để cho phép một agent spawn sub-agent dưới các agent id khác:

```json5
{
  agents: {
    list: [
      {
        id: "orchestrator",
        subagents: {
          allowAgents: ["researcher", "coder"], // or ["*"] to allow any
        },
      },
    ],
  },
}
```

<Tip>Sử dụng công cụ `agents_list` để khám phá những agent id nào hiện đang được phép cho `sessions_spawn`.</Tip>

## Quản lý Sub-Agent (`/subagents`)

Sử dụng lệnh slash `/subagents` để kiểm tra và điều khiển các lần chạy sub-agent cho phiên hiện tại:

| Lệnh                                       | Mô tả                                                                                      |
| ------------------------------------------ | ------------------------------------------------------------------------------------------ |
| `/subagents list`                          | Liệt kê tất cả các lần chạy sub-agent (đang hoạt động và đã hoàn thành) |
| `/subagents stop <id\\|#\\|all>`         | Dừng một sub-agent đang chạy                                                               |
| `/subagents log <id\\|#> [limit] [tools]` | Xem transcript của sub-agent                                                               |
| `/subagents info <id\\|#>`                | Hiển thị metadata chi tiết của lần chạy                                                    |
| `/subagents send <id\\|#> <message>`      | Gửi một tin nhắn tới sub-agent đang chạy                                                   |

Bạn có thể tham chiếu sub-agent bằng chỉ số trong danh sách (`1`, `2`), tiền tố run id, session key đầy đủ, hoặc `last`.

<AccordionGroup>
  <Accordion title="Example: list and stop a sub-agent">```
    /subagents list
    ```

    ````
    ```
    🧭 Subagents (current session)
    Active: 1 · Done: 2
    1) ✅ · research logs · 2m31s · run a1b2c3d4 · agent:main:subagent:...
    2) ✅ · check deps · 45s · run e5f6g7h8 · agent:main:subagent:...
    3) 🔄 · deploy staging · 1m12s · run i9j0k1l2 · agent:main:subagent:...
    ```
    
    ```
    /subagents stop 3
    ```
    
    ```
    ⚙️ Stop requested for deploy staging.
    ```
    ````

  </Accordion>
  <Accordion title="Example: inspect a sub-agent">```
    /subagents info 1
    ```

    ````
    ```
    ℹ️ Subagent info
    Status: ✅
    Label: research logs
    Task: Research the latest server error logs and summarize findings
    Run: a1b2c3d4-...
    Session: agent:main:subagent:...
    Runtime: 2m31s
    Cleanup: keep
    Outcome: ok
    ```
    ````

  </Accordion>
  <Accordion title="Example: view sub-agent log">```
    /subagents log 1 10
    ```

    ````
    Hiển thị 10 tin nhắn cuối cùng từ transcript của sub-agent. Thêm `tools` để bao gồm các thông điệp gọi công cụ:
    
    ```
    /subagents log 1 10 tools
    ```
    ````

  </Accordion>
  <Accordion title="Example: send a follow-up message">```
    /subagents send 3 "Also check the staging environment"
    ```

    ```
    Gửi một tin nhắn vào phiên của sub-agent đang chạy và chờ tối đa 30 giây để nhận phản hồi.
    ```

  </Accordion>
</AccordionGroup>

## Announce (Cách Kết Quả Được Trả Về)

Khi một sub-agent hoàn thành, nó sẽ trải qua một bước **announce**:

1. Phản hồi cuối cùng của sub-agent được ghi nhận
2. Một thông điệp tóm tắt được gửi tới phiên của main agent với kết quả, trạng thái và thống kê
3. Main agent đăng một bản tóm tắt bằng ngôn ngữ tự nhiên lên cuộc trò chuyện của bạn

Phản hồi announce giữ nguyên định tuyến luồng/chủ đề khi có (Slack threads, Telegram topics, Matrix threads).

### Thống Kê Announce

Mỗi announce bao gồm một dòng thống kê với:

- Thời lượng chạy
- Mức sử dụng token (đầu vào/đầu ra/tổng)
- Chi phí ước tính (khi giá model được cấu hình qua `models.providers.*.models[].cost`)
- Session key, session id và đường dẫn transcript

### Trạng Thái Announce

Thông điệp announce bao gồm một trạng thái được suy ra từ kết quả runtime (không phải từ đầu ra của model):

- **hoàn thành thành công** (`ok`) — tác vụ hoàn tất bình thường
- **lỗi** — tác vụ thất bại (chi tiết lỗi trong notes)
- **hết thời gian** — tác vụ vượt quá `runTimeoutSeconds`
- **không xác định** — không thể xác định trạng thái

<Tip>
Nếu không cần announce hướng tới người dùng, bước summarize của main agent có thể trả về `NO_REPLY` và sẽ không có gì được đăng.
Điều này khác với `ANNOUNCE_SKIP`, được dùng trong luồng announce giữa các agent (`sessions_send`).
</Tip>

## Chính Sách Công Cụ

Theo mặc định, sub-agent nhận **tất cả các công cụ ngoại trừ** một tập công cụ bị từ chối vì không an toàn hoặc không cần thiết cho các tác vụ nền:

<AccordionGroup>
  <Accordion title="Default denied tools">| Công cụ bị từ chối | Lý do |
|-------------|--------|
| `sessions_list` | Quản lý phiên — main agent điều phối |
| `sessions_history` | Quản lý phiên — main agent điều phối |
| `sessions_send` | Quản lý phiên — main agent điều phối |
| `sessions_spawn` | Không fan-out lồng nhau (sub-agent không thể spawn sub-agent) |
| `gateway` | Quản trị hệ thống — nguy hiểm từ sub-agent |
| `agents_list` | Quản trị hệ thống |
| `whatsapp_login` | Thiết lập tương tác — không phải tác vụ |
| `session_status` | Trạng thái/lập lịch — main agent điều phối |
| `cron` | Trạng thái/lập lịch — main agent điều phối |
| `memory_search` | Thay vào đó hãy truyền thông tin liên quan trong prompt spawn |
| `memory_get` | Thay vào đó hãy truyền thông tin liên quan trong prompt spawn |</Accordion>
</AccordionGroup>

### Tùy Biến Công Cụ Sub-Agent

Bạn có thể hạn chế thêm các công cụ của sub-agent:

```json5
{
  tools: {
    subagents: {
      tools: {
        // deny always wins over allow
        deny: ["browser", "firecrawl"],
      },
    },
  },
}
```

Để giới hạn sub-agent **chỉ** sử dụng các công cụ cụ thể:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // deny still wins if set
      },
    },
  },
}
```

<Note>
Các mục deny tùy chỉnh được **thêm vào** danh sách deny mặc định. Nếu `allow` được thiết lập, chỉ những công cụ đó mới khả dụng (danh sách deny mặc định vẫn được áp dụng thêm).
</Note>

## Xác thực

Xác thực sub-agent được phân giải theo **agent id**, không theo loại phiên:

- Auth store được tải từ `agentDir` của agent mục tiêu.
- Các profile auth của main agent được gộp vào như một **fallback** (profile của agent thắng khi có xung đột).
- The merge is additive — main profiles are always available as fallbacks

<Note>
Fully isolated auth per sub-agent is not currently supported.
</Note>

## Context and System Prompt

Sub-agents receive a reduced system prompt compared to the main agent:

- **Bao gồm:** Các phần Tooling, Workspace, Runtime, cùng với `AGENTS.md` và `TOOLS.md`
- **Không bao gồm:** `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`

Tác nhân phụ cũng nhận được một prompt hệ thống tập trung vào nhiệm vụ, hướng dẫn nó tập trung vào nhiệm vụ được giao, hoàn thành nhiệm vụ đó và không hoạt động như tác nhân chính.

## Stopping Sub-Agents

| Method                        | Effect                                                                    |
| ----------------------------- | ------------------------------------------------------------------------- |
| `/stop` trong cuộc trò chuyện | Aborts the main session **and** all active sub-agent runs spawned from it |
| `/subagents stop <id>`        | Stops a specific sub-agent without affecting the main session             |
| `runTimeoutSeconds`           | Automatically aborts the sub-agent run after the specified time           |

<Note>
`runTimeoutSeconds` does **not** auto-archive the session. The session remains until the normal archive timer fires.
</Note>

## Full Configuration Example

<Accordion title="Complete sub-agent configuration">```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-sonnet-4" },
      subagents: {
        model: "minimax/MiniMax-M2.1",
        thinking: "low",
        maxConcurrent: 4,
        archiveAfterMinutes: 30,
      },
    },
    list: [
      {
        id: "main",
        default: true,
        name: "Personal Assistant",
      },
      {
        id: "ops",
        name: "Ops Agent",
        subagents: {
          model: "anthropic/claude-sonnet-4",
          allowAgents: ["main"], // ops can spawn sub-agents under "main"
        },
      },
    ],
  },
  tools: {
    subagents: {
      tools: {
        deny: ["browser"], // sub-agents can't use the browser
      },
    },
  },
}
```</Accordion>

## Hạn chế

<Warning>
- **Thông báo theo nỗ lực tối đa:** Nếu gateway khởi động lại, các tác vụ thông báo đang chờ sẽ bị mất.
- **No nested spawning:** Sub-agents cannot spawn their own sub-agents.
- **Shared resources:** Sub-agents share the gateway process; use `maxConcurrent` as a safety valve.
- **Tự động lưu trữ là theo nỗ lực tối đa:** Các bộ hẹn giờ lưu trữ đang chờ sẽ bị mất khi gateway khởi động lại.
</Warning>

## Xem thêm

- [Session Tools](/concepts/session-tool) — details on `sessions_spawn` and other session tools
- [Multi-Agent Sandbox and Tools](/tools/multi-agent-sandbox-tools) — per-agent tool restrictions and sandboxing
- [Configuration](/gateway/configuration) — `agents.defaults.subagents` reference
- [Queue](/concepts/queue) — how the `subagent` lane works
