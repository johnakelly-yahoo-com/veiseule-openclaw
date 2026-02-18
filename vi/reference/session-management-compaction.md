---
title: "Chuyên sâu quản lý phiên"
---

# Quản lý phiên & Nén (Chuyên sâu)

Tài liệu này giải thích cách OpenClaw quản lý phiên từ đầu đến cuối:

- **Định tuyến phiên** (cách thông điệp đến được ánh xạ tới một `sessionKey`)
- **Kho phiên** (`sessions.json`) và những gì nó theo dõi
- **Lưu trữ bản ghi hội thoại** (`*.jsonl`) và cấu trúc của nó
- **Vệ sinh bản ghi hội thoại** (các chỉnh sửa theo nhà cung cấp trước khi chạy)
- **Giới hạn ngữ cảnh** (cửa sổ ngữ cảnh so với token được theo dõi)
- **Nén** (nén thủ công + tự động) và nơi gắn công việc trước khi nén
- **Dọn dẹp im lặng** (ví dụ: ghi bộ nhớ không nên tạo đầu ra hiển thị cho người dùng)

Nếu bạn muốn xem tổng quan ở mức cao trước, hãy bắt đầu với:

- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## Nguồn sự thật: Gateway

OpenClaw được thiết kế xoay quanh một **tiến trình Gateway** duy nhất nắm quyền sở hữu trạng thái phiên.

- Các UI (ứng dụng macOS, Control UI web, TUI) nên truy vấn Gateway để lấy danh sách phiên và số lượng token.
- Ở chế độ từ xa, các tệp phiên nằm trên máy chủ từ xa; “kiểm tra các tệp trên Mac cục bộ” sẽ không phản ánh những gì Gateway đang dùng.

---

## Hai lớp lưu trữ

OpenClaw lưu trữ phiên ở hai lớp:

1. **Kho phiên (`sessions.json`)**
   - Ánh xạ khóa/giá trị: `sessionKey -> SessionEntry`
   - Nhỏ, có thể thay đổi, an toàn để chỉnh sửa (hoặc xóa mục)
   - Theo dõi metadata của phiên (session id hiện tại, hoạt động gần nhất, các toggle, bộ đếm token, v.v.)

2. **Bản ghi hội thoại (`<sessionId>.jsonl`)**
   - Bản ghi chỉ-ghi-nối với cấu trúc cây (các mục có `id` + `parentId`)
   - Lưu trữ cuộc trò chuyện thực tế + lời gọi công cụ + tóm tắt nén
   - Dùng để tái dựng ngữ cảnh mô hình cho các lượt sau

---

## Vị trí trên đĩa

Theo từng tác tử, trên máy chủ gateway:

- Kho: `~/.openclaw/agents/<agentId>/sessions/sessions.json`
- Bản ghi hội thoại: `~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Phiên theo chủ đề Telegram: `.../<sessionId>-topic-<threadId>.jsonl`

OpenClaw phân giải các đường dẫn này thông qua `src/config/sessions.ts`.

---

## Khóa phiên (`sessionKey`)

Một `sessionKey` xác định _bạn đang ở “ngăn” hội thoại nào_ (định tuyến + cô lập).

Các mẫu phổ biến:

- Chat chính/trực tiếp (theo từng tác tử): `agent:<agentId>:<mainKey>` (mặc định `main`)
- Nhóm: `agent:<agentId>:<channel>:group:<id>`
- Phòng/kênh (Discord/Slack): `agent:<agentId>:<channel>:channel:<id>` hoặc `...:room:<id>`
- Cron: `cron:<job.id>`
- Webhook: `hook:<uuid>` (trừ khi bị ghi đè)

Các quy tắc chuẩn được ghi tại [/concepts/session](/concepts/session).

---

## Session id (`sessionId`)

Mỗi `sessionKey` trỏ tới một `sessionId` hiện tại (tệp bản ghi hội thoại tiếp tục cuộc trò chuyện).

Quy tắc kinh nghiệm:

- **Reset** (`/new`, `/reset`) tạo một `sessionId` mới cho `sessionKey` đó.
- **Reset hằng ngày** (mặc định 4:00 sáng theo giờ địa phương trên máy chủ gateway) tạo một `sessionId` mới ở thông điệp kế tiếp sau mốc reset.
- **Hết hạn khi nhàn rỗi** (`session.reset.idleMinutes` hoặc legacy `session.idleMinutes`) tạo một `sessionId` mới khi một tin nhắn đến sau cửa sổ nhàn rỗi. Khi cả daily + idle đều được cấu hình, cái nào hết hạn trước sẽ được ưu tiên.

Chi tiết triển khai: quyết định diễn ra trong `initSessionState()` ở `src/auto-reply/reply/session.ts`.

---

## Lược đồ kho phiên (`sessions.json`)

Kiểu giá trị của kho là `SessionEntry` trong `src/config/sessions.ts`.

Các trường chính (không đầy đủ):

- `sessionId`: id bản ghi hội thoại hiện tại (tên tệp được suy ra từ đây trừ khi đặt `sessionFile`)
- `updatedAt`: dấu thời gian hoạt động gần nhất
- `sessionFile`: ghi đè đường dẫn bản ghi hội thoại tường minh (tùy chọn)
- `chatType`: `direct | group | room` (giúp UI và chính sách gửi)
- `provider`, `subject`, `room`, `space`, `displayName`: metadata cho gắn nhãn nhóm/kênh
- Toggle:
  - `thinkingLevel`, `verboseLevel`, `reasoningLevel`, `elevatedLevel`
  - `sendPolicy` (ghi đè theo phiên)
- Chọn mô hình:
  - `providerOverride`, `modelOverride`, `authProfileOverride`
- Bộ đếm token (nỗ lực tốt nhất / phụ thuộc nhà cung cấp):
  - `inputTokens`, `outputTokens`, `totalTokens`, `contextTokens`
- `compactionCount`: số lần tự động nén đã hoàn tất cho khóa phiên này
- `memoryFlushAt`: dấu thời gian của lần xả bộ nhớ trước khi nén gần nhất
- `memoryFlushCompactionCount`: số lần nén tại thời điểm lần xả cuối chạy

Kho an toàn để chỉnh sửa, nhưng Gateway là thẩm quyền: nó có thể ghi lại hoặc tái tạo các mục khi phiên chạy.

---

## Cấu trúc bản ghi hội thoại (`*.jsonl`)

Bản ghi hội thoại được quản lý bởi `@mariozechner/pi-coding-agent`’s `SessionManager`.

Tệp ở định dạng JSONL:

- Dòng đầu: header phiên (`type: "session"`, bao gồm `id`, `cwd`, `timestamp`, tùy chọn `parentSession`)
- Sau đó: các mục phiên với `id` + `parentId` (cây)

Các loại mục đáng chú ý:

- `message`: thông điệp user/assistant/toolResult
- `custom_message`: thông điệp do extension chèn _có_ đi vào ngữ cảnh mô hình (có thể ẩn khỏi UI)
- `custom`: trạng thái extension _không_ đi vào ngữ cảnh mô hình
- `compaction`: tóm tắt nén được lưu với `firstKeptEntryId` và `tokensBefore`
- `branch_summary`: tóm tắt được lưu khi điều hướng một nhánh cây

OpenClaw cố ý **không** “chỉnh sửa” bản ghi; Gateway dùng `SessionManager` để đọc/ghi chúng.

---

## Cửa sổ ngữ cảnh vs token được theo dõi

Hai khái niệm khác nhau đều quan trọng:

1. **Cửa sổ ngữ cảnh của mô hình**: giới hạn cứng theo từng mô hình (token nhìn thấy bởi mô hình)
2. **Bộ đếm trong kho phiên**: thống kê cuộn được ghi vào `sessions.json` (dùng cho /status và dashboard)

Nếu bạn đang tinh chỉnh giới hạn:

- Cửa sổ ngữ cảnh đến từ danh mục mô hình (và có thể ghi đè qua cấu hình).
- `contextTokens` trong kho là giá trị ước lượng/báo cáo lúc chạy; đừng coi đó là bảo đảm nghiêm ngặt.

Xem thêm [/token-use](/reference/token-use).

---

## Nén: là gì

Nén tóm tắt cuộc trò chuyện cũ thành một mục `compaction` được lưu trong bản ghi và giữ nguyên các thông điệp gần đây.

Sau khi nén, các lượt sau sẽ thấy:

- Tóm tắt nén
- Các thông điệp sau `firstKeptEntryId`

32. Khóa phiên sai? 33. Bắt đầu với [/concepts/session](/concepts/session-pruning) và xác nhận `sessionKey` trong `/status`.

---

## Khi nào tự động nén diễn ra (runtime Pi)

Trong tác tử Pi nhúng, tự động nén kích hoạt trong hai trường hợp:

1. **Khôi phục tràn**: mô hình trả lỗi tràn ngữ cảnh → nén → thử lại.
2. **Bảo trì theo ngưỡng**: sau một lượt thành công, khi:

`contextTokens > contextWindow - reserveTokens`

Trong đó:

- `contextWindow` là cửa sổ ngữ cảnh của mô hình
- `reserveTokens` là phần đệm dành cho prompt + đầu ra mô hình kế tiếp

Đây là ngữ nghĩa runtime của Pi (OpenClaw tiêu thụ sự kiện, nhưng Pi quyết định khi nào nén).

---

## Thiết lập nén (`reserveTokens`, `keepRecentTokens`)

Thiết lập nén của Pi nằm trong cài đặt Pi:

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000,
  },
}
```

OpenClaw cũng áp dụng một mức sàn an toàn cho các lần chạy nhúng:

- Nếu `compaction.reserveTokens < reserveTokensFloor`, OpenClaw sẽ nâng lên.
- Mức sàn mặc định là `20000` token.
- Đặt `agents.defaults.compaction.reserveTokensFloor: 0` để tắt mức sàn.
- Nếu đã cao hơn, OpenClaw giữ nguyên.

Lý do: chừa đủ khoảng trống cho “dọn dẹp” nhiều lượt (như ghi bộ nhớ) trước khi nén trở nên không thể tránh khỏi.

Triển khai: `ensurePiCompactionReserveTokens()` trong `src/agents/pi-settings.ts`
(được gọi từ `src/agents/pi-embedded-runner.ts`).

---

## Bề mặt hiển thị cho người dùng

Bạn có thể quan sát nén và trạng thái phiên qua:

- `/status` (trong bất kỳ phiên chat nào)
- `openclaw status` (CLI)
- `openclaw sessions` / `sessions --json`
- Chế độ verbose: `🧹 Auto-compaction complete` + số lần nén

---

## Dọn dẹp im lặng (`NO_REPLY`)

OpenClaw hỗ trợ các lượt “im lặng” cho tác vụ nền nơi người dùng không nên thấy đầu ra trung gian.

Quy ước:

- Assistant bắt đầu đầu ra bằng `NO_REPLY` để báo “không gửi phản hồi cho người dùng”.
- OpenClaw loại bỏ/ẩn điều này ở lớp phân phối.

Kể từ `2026.1.10`, OpenClaw cũng ẩn **streaming nháp/đang gõ** khi một mảnh (chunk) bắt đầu bằng `NO_REPLY`, để các thao tác im lặng không rò rỉ đầu ra từng phần giữa lượt.

---

## “Xả bộ nhớ” trước khi nén (đã triển khai)

Mục tiêu: trước khi tự động nén xảy ra, chạy một lượt tác tử im lặng để ghi trạng thái bền vững
xuống đĩa (ví dụ: `memory/YYYY-MM-DD.md` trong workspace của tác tử) để nén không thể
xóa ngữ cảnh quan trọng.

OpenClaw dùng cách tiếp cận **xả trước ngưỡng**:

1. Theo dõi mức sử dụng ngữ cảnh của phiên.
2. Khi vượt qua “ngưỡng mềm” (thấp hơn ngưỡng nén của Pi), chạy một chỉ thị im lặng
   “ghi bộ nhớ ngay” gửi cho tác tử.
3. Dùng `NO_REPLY` để người dùng không thấy gì.

Cấu hình (`agents.defaults.compaction.memoryFlush`):

- `enabled` (mặc định: `true`)
- `softThresholdTokens` (mặc định: `4000`)
- `prompt` (thông điệp người dùng cho lượt xả)
- `systemPrompt` (system prompt bổ sung được nối cho lượt xả)

Ghi chú:

- Prompt/system prompt mặc định bao gồm gợi ý `NO_REPLY` để ẩn phân phối.
- Lượt xả chạy một lần cho mỗi chu kỳ nén (được theo dõi trong `sessions.json`).
- Lượt xả chỉ chạy cho các phiên Pi nhúng (backend CLI bỏ qua).
- Lượt xả bị bỏ qua khi workspace của phiên là chỉ đọc (`workspaceAccess: "ro"` hoặc `"none"`).
- Xem [Memory](/concepts/memory) để biết bố cục tệp workspace và các mẫu ghi.

Pi cũng cung cấp hook `session_before_compact` trong extension API, nhưng logic xả của OpenClaw hiện nằm phía Gateway.

---

## Danh sách kiểm tra xử lý sự cố

- Session key sai? Bắt đầu với [/concepts/session](/concepts/session) và xác nhận `sessionKey` trong `/status`.
- Lệch giữa store và transcript? 37. Kiểm tra:
- Spam nén? 39. Xác nhận phản hồi bắt đầu bằng `NO_REPLY` (token chính xác) và bạn đang dùng bản build có bao gồm bản sửa ức chế streaming.
  - cửa sổ ngữ cảnh mô hình (quá nhỏ)
  - thiết lập nén (`reserveTokens` quá cao so với cửa sổ mô hình có thể gây nén sớm)
  - phình to tool-result: bật/điều chỉnh cắt tỉa phiên
- 40. "Xin chào, C-3PO! Xác nhận phản hồi bắt đầu bằng `NO_REPLY` (token chính xác) và bạn đang ở bản build bao gồm bản sửa lỗi chặn streaming.
