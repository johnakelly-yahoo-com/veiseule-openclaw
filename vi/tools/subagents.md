---
title: "Tác tử phụ"
---

# Sub-agents

Sub-agents là các lần chạy agent nền được tạo ra từ một lần chạy agent hiện có. Chúng chạy trong phiên riêng (`agent:<agentId>:subagent:<uuid>`) và khi hoàn tất sẽ **thông báo** kết quả trở lại kênh chat của bên yêu cầu.

## Slash command

Sử dụng `/subagents` để kiểm tra hoặc điều khiển các lần chạy sub-agent cho **phiên hiện tại**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info` hiển thị metadata của lần chạy (trạng thái, mốc thời gian, session id, đường dẫn transcript, cleanup).

Mục tiêu chính:

- Song song hóa các tác vụ "nghiên cứu / công việc dài / tool chậm" mà không chặn lần chạy chính.
- Giữ sub-agent được cô lập theo mặc định (tách phiên + sandbox tùy chọn).
- Giữ bề mặt công cụ khó bị lạm dụng: sub-agent **không** nhận session tools theo mặc định.
- Hỗ trợ độ sâu lồng nhau có thể cấu hình cho các mẫu điều phối (orchestrator).

Lưu ý về chi phí: mỗi sub-agent có **ngữ cảnh** và mức sử dụng token **riêng**. Với các tác vụ nặng hoặc lặp lại, hãy đặt model rẻ hơn cho sub-agent và giữ agent chính ở model chất lượng cao hơn.
Bạn có thể cấu hình qua `agents.defaults.subagents.model` hoặc ghi đè theo từng agent.

## Tool

Sử dụng `sessions_spawn`:

- Bắt đầu một lần chạy sub-agent (`deliver: false`, global lane: `subagent`)
- Sau đó chạy bước announce và đăng phản hồi announce lên kênh chat của bên yêu cầu
- Model mặc định: kế thừa từ bên gọi trừ khi bạn đặt `agents.defaults.subagents.model` (hoặc `agents.list[].subagents.model`); nếu chỉ định rõ `sessions_spawn.model` thì sẽ được ưu tiên.
- Thinking mặc định: kế thừa từ bên gọi trừ khi bạn đặt `agents.defaults.subagents.thinking` (hoặc `agents.list[].subagents.thinking`); nếu chỉ định rõ `sessions_spawn.thinking` thì sẽ được ưu tiên.

Tham số tool:

- `task` (bắt buộc)
- `label?` (tùy chọn)
- `agentId?` (tùy chọn; spawn dưới agent id khác nếu được phép)
- `model?` (tùy chọn; ghi đè model cho sub-agent; giá trị không hợp lệ sẽ bị bỏ qua và sub-agent chạy với model mặc định kèm cảnh báo trong kết quả tool)
- `thinking?` (tùy chọn; ghi đè mức thinking cho lần chạy sub-agent)
- `runTimeoutSeconds?` (mặc định `0`; nếu đặt, lần chạy sub-agent sẽ bị hủy sau N giây)
- `cleanup?` (`delete|keep`, mặc định `keep`)

Allowlist:

- `agents.list[].subagents.allowAgents`: danh sách agent id có thể được nhắm tới qua `agentId` (`["*"]` để cho phép tất cả). Mặc định: chỉ agent bên yêu cầu.

Discovery:

- Sử dụng `agents_list` để xem những agent id nào hiện được phép cho `sessions_spawn`.

Auto-archive:

- Phiên sub-agent tự động được lưu trữ sau `agents.defaults.subagents.archiveAfterMinutes` (mặc định: 60).
- Lưu trữ sử dụng `sessions.delete` và đổi tên transcript thành `*.deleted.<timestamp>` (cùng thư mục).
- `cleanup: "delete"` sẽ lưu trữ ngay sau announce (vẫn giữ transcript thông qua đổi tên).
- Auto-archive là cơ chế best-effort; các bộ hẹn giờ đang chờ sẽ bị mất nếu gateway khởi động lại.
- `runTimeoutSeconds` **không** tự động lưu trữ; nó chỉ dừng lần chạy. Phiên vẫn tồn tại cho đến khi auto-archive.
- Auto-archive áp dụng như nhau cho phiên depth-1 và depth-2.

## Nested Sub-Agents

Theo mặc định, sub-agent không thể tự spawn sub-agent khác (`maxSpawnDepth: 1`). Bạn có thể bật một cấp lồng nhau bằng cách đặt `maxSpawnDepth: 2`, cho phép **mẫu orchestrator**: main → orchestrator sub-agent → worker sub-sub-agents.

### How to enable

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // cho phép sub-agent spawn con (mặc định: 1)
        maxChildrenPerAgent: 5, // số lượng con đang hoạt động tối đa mỗi phiên agent (mặc định: 5)
        maxConcurrent: 8, // giới hạn đồng thời toàn cục của lane (mặc định: 8)
      },
    },
  },
}
```

### Depth levels

| Depth | Session key shape                            | Vai trò                                       | Có thể spawn?               |
| ----- | -------------------------------------------- | --------------------------------------------- | --------------------------- |
| 0     | `agent:<id>:main`                            | Main agent                                    | Luôn luôn                   |
| 1     | `agent:<id>:subagent:<uuid>`                 | Sub-agent (orchestrator khi cho phép depth 2) | Chỉ nếu `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (worker lá)                     | Không bao giờ               |

### Announce chain

Kết quả được truyền ngược lên theo chuỗi:

1. Worker depth-2 hoàn tất → announce tới parent của nó (orchestrator depth-1)
2. Orchestrator depth-1 nhận announce, tổng hợp kết quả, hoàn tất → announce tới main
3. Main agent nhận announce và gửi tới người dùng

Mỗi cấp chỉ thấy announce từ các con trực tiếp của mình.

### Tool policy by depth

- **Depth 1 (orchestrator, khi `maxSpawnDepth >= 2`)**: Nhận `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` để có thể quản lý các agent con. Các session/system tools khác vẫn bị từ chối.
- **Depth 1 (leaf, khi `maxSpawnDepth == 1`)**: Không có session tools (hành vi mặc định hiện tại).
- **Depth 2 (leaf worker)**: Không có session tools — `sessions_spawn` luôn bị từ chối ở depth 2. Không thể spawn thêm con.

### Per-agent spawn limit

Mỗi phiên agent (ở bất kỳ depth nào) có thể có tối đa `maxChildrenPerAgent` (mặc định: 5) agent con đang hoạt động cùng lúc. Điều này ngăn chặn việc fan-out mất kiểm soát từ một orchestrator.

### Cascade stop

Dừng một orchestrator depth-1 sẽ tự động dừng tất cả các con depth-2 của nó:

- `/stop` trong chat chính sẽ dừng tất cả agent depth-1 và cascade tới các con depth-2 của chúng.
- `/subagents kill <id>` dừng một sub-agent cụ thể và cascade tới các con của nó.
- `/subagents kill all` dừng tất cả sub-agent của bên yêu cầu và cascade.

## Authentication

Xác thực sub-agent được phân giải theo **agent id**, không theo loại phiên:

- Session key của sub-agent là `agent:<agentId>:subagent:<uuid>`.
- Auth store được tải từ `agentDir` của agent đó.
- Các profile auth của main agent được gộp vào như một **fallback**; profile của agent sẽ ghi đè profile main khi có xung đột.

Lưu ý: việc gộp là additive, vì vậy profile của main luôn khả dụng như fallback. Cô lập xác thực hoàn toàn theo từng agent hiện chưa được hỗ trợ.

## Announce

Sub-agent báo cáo lại thông qua một bước announce:

- Bước announce chạy bên trong phiên sub-agent (không phải phiên của bên yêu cầu).
- Nếu sub-agent trả lời chính xác `ANNOUNCE_SKIP`, sẽ không có gì được đăng.
- Ngược lại, phản hồi announce sẽ được đăng lên kênh chat của bên yêu cầu thông qua một lần gọi `agent` tiếp theo (`deliver=true`).
- Phản hồi announce giữ nguyên định tuyến thread/topic khi có (Slack threads, Telegram topics, Matrix threads).
- Thông điệp announce được chuẩn hóa theo một mẫu ổn định:
  - `Status:` được suy ra từ kết quả lần chạy (`success`, `error`, `timeout`, hoặc `unknown`).
  - `Result:` nội dung tóm tắt từ bước announce (hoặc `(not available)` nếu thiếu).
  - `Notes:` chi tiết lỗi và các ngữ cảnh hữu ích khác.
- `Status` không được suy ra từ đầu ra của model; nó đến từ tín hiệu kết quả runtime.

Payload announce bao gồm một dòng thống kê ở cuối (kể cả khi được bọc):

- Thời lượng chạy (ví dụ: `runtime 5m12s`)
- Mức sử dụng token (input/output/total)
- Chi phí ước tính khi giá model được cấu hình (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` và đường dẫn transcript (để main agent có thể lấy lịch sử qua `sessions_history` hoặc kiểm tra file trên đĩa)

## Tool Policy (sub-agent tools)

Theo mặc định, sub-agent nhận **tất cả tool ngoại trừ session tools** và system tools:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Khi `maxSpawnDepth >= 2`, sub-agent depth-1 (orchestrator) bổ sung nhận `sessions_spawn`, `subagents`, `sessions_list`, và `sessions_history` để có thể quản lý các agent con.

Ghi đè qua cấu hình:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny wins
        deny: ["gateway", "cron"],
        // nếu allow được đặt, nó trở thành allow-only (deny vẫn thắng)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Concurrency

Sub-agent sử dụng một lane hàng đợi chuyên dụng trong cùng tiến trình:

- Tên lane: `subagent`
- Độ đồng thời: `agents.defaults.subagents.maxConcurrent` (mặc định `8`)

## Stopping

- Gửi `/stop` trong chat của bên yêu cầu sẽ hủy phiên của bên yêu cầu và dừng mọi lần chạy sub-agent đang hoạt động được spawn từ đó, bao gồm cascade tới các con lồng nhau.
- `/subagents kill <id>` dừng một sub-agent cụ thể và cascade tới các con của nó.

## Limitations

- Announce của sub-agent là **best-effort**. Nếu gateway khởi động lại, các tác vụ "announce back" đang chờ sẽ bị mất.
- Sub-agent vẫn chia sẻ tài nguyên tiến trình gateway; hãy coi `maxConcurrent` như một van an toàn.
- `sessions_spawn` luôn không chặn: nó trả về `{ status: "accepted", runId, childSessionKey }` ngay lập tức.
- Ngữ cảnh sub-agent chỉ inject `AGENTS.md` + `TOOLS.md` (không có `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, hoặc `BOOTSTRAP.md`).
- Độ sâu lồng tối đa là 5 (`maxSpawnDepth` phạm vi: 1–5). Depth 2 được khuyến nghị cho hầu hết các trường hợp sử dụng.
- `maxChildrenPerAgent` giới hạn số agent con đang hoạt động mỗi phiên (mặc định: 5, phạm vi: 1–20).
