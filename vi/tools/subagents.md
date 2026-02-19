---
summary: "Sub-agent: tạo các lần chạy tác tử cô lập chạy song song và thông báo kết quả về chat yêu cầu"
read_when:
  - Bạn muốn thực hiện công việc nền/song song thông qua tác tử
  - Bạn đang thay đổi sessions_spawn hoặc chính sách công cụ sub-agent
title: "Sub-Agents"
---

# Sub-agents

Sub-agents là các phiên chạy agent nền được khởi tạo từ một phiên chạy agent hiện có. Chúng chạy trong phiên riêng (`agent:<agentId>:subagent:<uuid>`) và khi hoàn tất sẽ **thông báo** kết quả trở lại kênh chat của người yêu cầu.

## Slash command

Sử dụng `/subagents` để kiểm tra hoặc điều khiển các phiên chạy sub-agent cho **phiên hiện tại**:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

Cách đơn giản nhất để sử dụng sub-agent là yêu cầu agent của bạn một cách tự nhiên:

Mục tiêu chính:

- Song song hóa các tác vụ "nghiên cứu / tác vụ dài / công cụ chậm" mà không chặn phiên chạy chính.
- Giữ sub-agents được cô lập theo mặc định (tách phiên + sandbox tùy chọn).
- Giữ bề mặt công cụ khó bị lạm dụng: sub-agents **không** được cấp session tools theo mặc định.
- Hỗ trợ độ sâu lồng nhau có thể cấu hình cho các mô hình điều phối (orchestrator patterns).

Lưu ý chi phí: mỗi sub-agent có **ngữ cảnh** và mức sử dụng token riêng. Đối với các
tác vụ nặng hoặc lặp lại, hãy đặt model rẻ hơn cho sub-agents và giữ agent chính của bạn trên model chất lượng cao hơn.
Bạn có thể cấu hình điều này qua `agents.defaults.subagents.model` hoặc ghi đè theo từng agent.

## Tool

Sử dụng `sessions_spawn`:

- Khởi động một phiên chạy sub-agent (`deliver: false`, lane toàn cục: `subagent`)
- Sau đó chạy bước announce và đăng phản hồi announce lên kênh chat của người yêu cầu
- Model mặc định: kế thừa từ caller trừ khi bạn đặt `agents.defaults.subagents.model` (hoặc `agents.list[].subagents.model` theo từng agent); nếu chỉ định rõ `sessions_spawn.model` thì giá trị này vẫn được ưu tiên.
- Thinking mặc định: kế thừa từ caller trừ khi bạn đặt `agents.defaults.subagents.thinking` (hoặc `agents.list[].subagents.thinking` theo từng agent); nếu chỉ định rõ `sessions_spawn.thinking` thì giá trị này vẫn được ưu tiên.

Tham số tool:

- `task` (bắt buộc)
- `label?` (tùy chọn)
- `agentId?` (tùy chọn; khởi tạo dưới một agent id khác nếu được phép)
- `model?` (tùy chọn; ghi đè model của sub-agent; các giá trị không hợp lệ sẽ bị bỏ qua và sub-agent sẽ chạy với model mặc định kèm cảnh báo trong kết quả công cụ)
- `thinking?` (tùy chọn; ghi đè mức độ suy luận cho lần chạy của sub-agent)
- `runTimeoutSeconds?` (mặc định `0`; khi được thiết lập, lần chạy của sub-agent sẽ bị hủy sau N giây)
- `cleanup?` (`delete|keep`, mặc định `keep`)

Danh sách cho phép:

- `agents.list[].subagents.allowAgents`: danh sách agent id có thể được nhắm tới qua `agentId` (`["*"]` để cho phép tất cả). Mặc định: chỉ agent gửi yêu cầu.

Khám phá:

- Sử dụng `agents_list` để xem những agent id nào hiện được phép cho `sessions_spawn`.

Tự động lưu trữ:

- Các phiên sub-agent sẽ được tự động lưu trữ sau `agents.defaults.subagents.archiveAfterMinutes` (mặc định: 60).
- Việc lưu trữ sử dụng `sessions.delete` và đổi tên bản ghi thành `*.deleted.<timestamp>` (cùng thư mục).
- `cleanup: "delete"` sẽ lưu trữ ngay sau khi thông báo (vẫn giữ lại bản ghi thông qua đổi tên).
- Tự động lưu trữ hoạt động theo cơ chế best-effort; các bộ hẹn giờ đang chờ sẽ bị mất nếu gateway khởi động lại.
- `runTimeoutSeconds` **không** tự động lưu trữ; nó chỉ dừng lần chạy. Phiên sẽ vẫn tồn tại cho đến khi được tự động lưu trữ.
- Tự động lưu trữ áp dụng như nhau cho các phiên độ sâu 1 và độ sâu 2.

## Sub-Agent lồng nhau

Mặc định, sub-agent không thể tạo sub-agent riêng của mình (`maxSpawnDepth: 1`). Bạn có thể bật một cấp lồng nhau bằng cách đặt `maxSpawnDepth: 2`, cho phép **mô hình điều phối (orchestrator pattern)**: main → orchestrator sub-agent → worker sub-sub-agents.

### Cách bật

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

| Độ sâu | Dạng khóa phiên                              | Vai trò                                                           | Có thể tạo thêm không?       |
| ------ | -------------------------------------------- | ----------------------------------------------------------------- | ---------------------------- |
| 0      | `agent:&lt;id&gt;:main`                            | Agent chính                                                       | Luôn luôn                    |
| 1      | `agent:&lt;id&gt;:subagent:<uuid>`                 | Sub-agent (orchestrator khi cho phép độ sâu 2) | Chỉ khi `maxSpawnDepth >= 2` |
| 2      | `agent:&lt;id&gt;:subagent:<uuid>:subagent:<uuid>` | Sub-sub-agent (worker lá)                      | Không bao giờ                |

### Chuỗi announce

Sub-agent sử dụng một làn hàng đợi riêng (`subagent`) tách biệt với hàng đợi agent chính, vì vậy các lần chạy sub-agent không chặn các phản hồi đến.

1. Worker ở độ sâu 2 hoàn tất → gửi announce tới tiến trình cha (orchestrator độ sâu 1)
2. Orchestrator độ sâu 1 nhận announce, tổng hợp kết quả, hoàn tất → gửi announce tới main
3. Agent main nhận announce và gửi kết quả tới người dùng

Các phiên sub-agent được tự động lưu trữ sau một khoảng thời gian có thể cấu hình:

### Chính sách công cụ theo độ sâu

- **Độ sâu 1 (orchestrator, khi `maxSpawnDepth >= 2`)**: Được cấp `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` để có thể quản lý các tiến trình con. Các công cụ session/system khác vẫn bị từ chối.
- **Độ sâu 1 (leaf, khi `maxSpawnDepth == 1`)**: Không có công cụ session (hành vi mặc định hiện tại).
- **Độ sâu 2 (leaf worker)**: Không có công cụ session — `sessions_spawn` luôn bị từ chối ở độ sâu 2. Không thể tạo thêm tiến trình con.

### Công cụ `sessions_spawn`

Mỗi phiên agent (ở bất kỳ độ sâu nào) có thể có tối đa `maxChildrenPerAgent` (mặc định: 5) tiến trình con đang hoạt động cùng lúc. Điều này ngăn chặn việc mở rộng mất kiểm soát từ một orchestrator duy nhất.

### Tham số

Dừng một orchestrator độ sâu 1 sẽ tự động dừng tất cả các tiến trình con độ sâu 2 của nó:

- `/stop` trong chat chính sẽ dừng tất cả agent độ sâu 1 và lan truyền tới các tiến trình con độ sâu 2 của chúng.
- `/subagents kill <id>` dừng một sub-agent cụ thể và lan truyền tới các tiến trình con của nó.
- `/subagents kill all` dừng tất cả sub-agent của người yêu cầu và lan truyền tiếp.

## Xác thực

Xác thực sub-agent được phân giải theo **agent id**, không theo loại phiên:

- Khóa phiên sub-agent là `agent:<agentId>:subagent:<uuid>`.
- Auth store được tải từ `agentDir` của agent đó.
- Các hồ sơ xác thực của agent chính được gộp vào như một **fallback**; hồ sơ của agent sẽ ghi đè hồ sơ chính nếu có xung đột.

Lưu ý: việc gộp là cộng dồn, vì vậy các hồ sơ chính luôn khả dụng như fallback. Hiện chưa hỗ trợ cơ chế xác thực tách biệt hoàn toàn cho từng agent.

## Announce

Sub-agent báo cáo lại thông qua một bước announce:

- Bước announce chạy bên trong phiên sub-agent (không phải phiên của người yêu cầu).
- Nếu sub-agent trả lời chính xác `ANNOUNCE_SKIP`, sẽ không có nội dung nào được đăng.
- Ngược lại, nội dung announce sẽ được đăng lên kênh chat của người yêu cầu thông qua một lệnh gọi `agent` tiếp theo (`deliver=true`).
- Phản hồi announce giữ nguyên định tuyến luồng/chủ đề khi có (Slack threads, Telegram topics, Matrix threads).
- Thông điệp announce được chuẩn hóa theo một mẫu ổn định:
  - `Status:` được suy ra từ kết quả chạy (`success`, `error`, `timeout`, hoặc `unknown`).
  - `Result:` nội dung tóm tắt từ bước announce (hoặc `(not available)` nếu thiếu).
  - `Notes:` chi tiết lỗi và các ngữ cảnh hữu ích khác.
- `Status` không được suy đoán từ đầu ra của model; nó đến từ các tín hiệu kết quả khi chạy.

Payload announce bao gồm một dòng thống kê ở cuối (kể cả khi được bao bọc):

- Thời gian chạy (ví dụ: `runtime 5m12s`)
- Mức sử dụng token (đầu vào/đầu ra/tổng)
- Chi phí ước tính khi đã cấu hình giá model (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId`, và đường dẫn transcript (để agent chính có thể lấy lịch sử qua `sessions_history` hoặc kiểm tra file trên đĩa)

## Quản lý Sub-Agent (`/subagents`)

Sử dụng lệnh slash `/subagents` để kiểm tra và điều khiển các lần chạy sub-agent cho phiên hiện tại:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

Bạn có thể tham chiếu sub-agent bằng chỉ số trong danh sách (`1`, `2`), tiền tố run id, session key đầy đủ, hoặc `last`.

Ghi đè thông qua cấu hình:

`````json5
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
`````

## Đồng thời

Các sub-agent sử dụng một hàng đợi nội bộ chuyên dụng trong cùng tiến trình:

- Tên lane: `subagent`
- Độ đồng thời: `agents.defaults.subagents.maxConcurrent` (mặc định `8`)

## Dừng

- Gửi `/stop` trong cuộc trò chuyện của requester sẽ hủy phiên requester và dừng mọi tiến trình sub-agent đang chạy được tạo từ đó, bao gồm cả các tiến trình con lồng nhau.
- `/subagents kill <id>` sẽ dừng một sub-agent cụ thể và dừng cả các tiến trình con của nó.

## Hạn chế

- Thông báo của sub-agent là **best-effort**. Nếu gateway khởi động lại, các tác vụ "announce back" đang chờ sẽ bị mất.
- Sub-agent vẫn chia sẻ cùng tài nguyên của tiến trình gateway; hãy xem `maxConcurrent` như một van an toàn.
- `sessions_spawn` luôn không chặn: nó trả về `{ status: "accepted", runId, childSessionKey }` ngay lập tức.
- Ngữ cảnh của sub-agent chỉ chèn `AGENTS.md` + `TOOLS.md` (không có `SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, hoặc `BOOTSTRAP.md`).
- Độ sâu lồng tối đa là 5 (`maxSpawnDepth` phạm vi: 1–5). Độ sâu 2 được khuyến nghị cho hầu hết các trường hợp sử dụng.
- `maxChildrenPerAgent` giới hạn số tiến trình con đang hoạt động trên mỗi phiên (mặc định: 5, phạm vi: 1–20).
