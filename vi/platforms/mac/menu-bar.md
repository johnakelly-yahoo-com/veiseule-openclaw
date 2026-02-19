---
summary: "Logic trạng thái menu bar và những gì hiển thị cho người dùng"
read_when:
  - Điều chỉnh UI menu mac hoặc logic trạng thái
title: "Thanh menu"
---

# Logic Trạng Thái Menu Bar

## Những gì được hiển thị

- Chúng tôi hiển thị trạng thái làm việc hiện tại của tác tử trong biểu tượng menu bar và ở hàng trạng thái đầu tiên của menu.
- Trạng thái sức khỏe bị ẩn khi đang có công việc; nó sẽ quay lại khi tất cả các phiên đều ở trạng thái nhàn rỗi.
- Khối “Nodes” trong menu chỉ liệt kê **thiết bị** (các node đã ghép cặp qua `node.list`), không phải các mục client/presence.
- Một mục “Usage” xuất hiện dưới Context khi có sẵn ảnh chụp mức sử dụng của nhà cung cấp.

## Mô hình trạng thái

- Phiên: các sự kiện đến với `runId` (theo từng lần chạy) cùng với `sessionKey` trong payload. Session “main” có khóa là `main`; nếu không có, chúng tôi fallback sang session được cập nhật gần nhất.
- Ưu tiên: main luôn thắng. Nếu main đang hoạt động, trạng thái của nó được hiển thị ngay lập tức. Nếu main đang nhàn rỗi, phiên không‑main hoạt động gần đây nhất sẽ được hiển thị. Chúng tôi không chuyển qua lại giữa chừng khi đang hoạt động; chỉ chuyển khi session hiện tại idle hoặc main trở nên hoạt động.
- Loại hoạt động:
  - `job`: thực thi lệnh mức cao (`state: started|streaming|done|error`).
  - `tool`: `phase: start|result` với `toolName` và `meta/args`.

## Enum IconState (Swift)

- `idle`
- `workingMain(ActivityKind)`
- `workingOther(ActivityKind)`
- `overridden(ActivityKind)` (ghi đè debug)

### ActivityKind → biểu tượng

- `exec` → 💻
- `read` → 📄
- `write` → ✍️
- `edit` → 📝
- `attach` → 📎
- default → 🛠️

### Ánh xạ hiển thị

- `idle`: critter bình thường.
- `workingMain`: huy hiệu có glyph, màu đầy đủ, hoạt ảnh chân “đang làm việc”.
- `workingOther`: huy hiệu có glyph, màu dịu, không chạy.
- `overridden`: dùng glyph/màu đã chọn bất kể hoạt động.

## Văn bản hàng trạng thái (menu)

- Khi đang có công việc: `<Session role> · <activity label>`
  - Ví dụ: `Main · exec: pnpm test`, `Other · read: apps/macos/Sources/OpenClaw/AppState.swift`.
- Khi nhàn rỗi: quay về tóm tắt sức khỏe.

## Thu nhận sự kiện

- Nguồn: các sự kiện kênh điều khiển `agent` (`ControlChannel.handleAgentEvent`).
- Trường được phân tích:
  - `stream: "job"` với `data.state` cho bắt đầu/kết thúc.
  - `stream: "tool"` với `data.phase`, `name`, tùy chọn `meta`/`args`.
- Nhãn:
  - `exec`: dòng đầu của `args.command`.
  - `read`/`write`: đường dẫn rút gọn.
  - `edit`: đường dẫn cộng với loại thay đổi suy luận từ `meta`/số lượng diff.
  - fallback: tên công cụ.

## Ghi đè debug

- Settings ▸ Debug ▸ bộ chọn “Icon override”:
  - `System (auto)` (mặc định)
  - `Working: main` (theo loại công cụ)
  - `Working: other` (theo loại công cụ)
  - `Idle`
- Lưu qua `@AppStorage("iconOverride")`; ánh xạ tới `IconState.overridden`.

## Danh sách kiểm tra kiểm thử

- Kích hoạt job của phiên chính: xác minh biểu tượng chuyển ngay và hàng trạng thái hiển thị nhãn của phiên chính.
- Kích hoạt job của phiên không‑chính khi phiên chính nhàn rỗi: biểu tượng/trạng thái hiển thị phiên không‑chính; giữ ổn định cho đến khi hoàn tất.
- Bắt đầu phiên chính khi phiên khác đang hoạt động: biểu tượng chuyển sang phiên chính ngay lập tức.
- Các đợt công cụ nhanh: đảm bảo huy hiệu không nhấp nháy (TTL ân hạn trên kết quả công cụ).
- Hàng sức khỏe xuất hiện lại khi tất cả các phiên đều nhàn rỗi.

