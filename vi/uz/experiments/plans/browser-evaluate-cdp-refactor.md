---
summary: "Kế hoạch: tách browser act:evaluate khỏi hàng đợi Playwright bằng CDP, với deadline end-to-end và xử lý tham chiếu ref an toàn hơn"
owner: "openclaw"
status: "bản nháp"
last_updated: "2026-02-10"
title: "Tái cấu trúc Browser Evaluate CDP"
---

# Kế hoạch Tái cấu trúc Browser Evaluate CDP

## Bối cảnh

`act:evaluate` thực thi JavaScript do người dùng cung cấp trong trang. Hiện tại nó chạy thông qua Playwright
(`page.evaluate` hoặc `locator.evaluate`). Playwright tuần tự hóa các lệnh CDP theo từng trang, vì vậy một evaluate
bị treo hoặc chạy quá lâu có thể chặn hàng đợi lệnh của trang và khiến mọi hành động sau đó
trên tab đó trông như bị "treo".

PR #13498 bổ sung một cơ chế an toàn thực dụng (giới hạn thời gian evaluate, truyền abort, và
phục hồi ở mức tối đa có thể). Tài liệu này mô tả một đợt tái cấu trúc lớn hơn nhằm làm cho `act:evaluate` được
cô lập vốn có khỏi Playwright, để một evaluate bị treo không thể làm kẹt các thao tác Playwright thông thường.

## Mục tiêu

- `act:evaluate` không thể chặn vĩnh viễn các hành động trình duyệt sau đó trên cùng một tab.
- Timeout là nguồn sự thật duy nhất xuyên suốt từ đầu đến cuối để bên gọi có thể tin tưởng vào ngân sách thời gian.
- Abort và timeout được xử lý theo cùng một cách giữa HTTP và điều phối trong cùng tiến trình.
- Hỗ trợ nhắm mục tiêu phần tử cho evaluate mà không cần chuyển toàn bộ khỏi Playwright.
- Duy trì khả năng tương thích ngược cho các bên gọi và payload hiện có.

## Không thuộc phạm vi

- Thay thế tất cả các hành động trình duyệt (click, type, wait, v.v.) bằng các triển khai CDP.
- Loại bỏ cơ chế an toàn hiện có được giới thiệu trong PR #13498 (nó vẫn là một phương án dự phòng hữu ích).
- Giới thiệu các khả năng không an toàn mới vượt ra ngoài cổng `browser.evaluateEnabled` hiện có.
- Thêm cô lập tiến trình (worker process/thread) cho evaluate. Nếu sau đợt tái cấu trúc này chúng ta vẫn gặp các trạng thái bị treo
  khó phục hồi, đó sẽ là một ý tưởng tiếp theo.

## Kiến trúc Hiện tại (Vì sao bị treo)

Ở mức độ tổng quan:

- Các bên gọi gửi `act:evaluate` tới dịch vụ điều khiển trình duyệt.
- Trình xử lý route gọi vào Playwright để thực thi JavaScript.
- Playwright tuần tự hóa các lệnh trang, vì vậy một evaluate không bao giờ kết thúc sẽ chặn hàng đợi.
- Hàng đợi bị kẹt có nghĩa là các thao tác click/type/wait sau đó trên tab có thể trông như bị treo.

## Kiến trúc Đề xuất

### 1. Truyền Deadline

Giới thiệu một khái niệm ngân sách thời gian duy nhất và suy ra mọi thứ từ đó:

- Bên gọi đặt `timeoutMs` (hoặc một deadline trong tương lai).
- Timeout của request bên ngoài, logic của route handler và ngân sách thực thi bên trong trang
  đều sử dụng cùng một ngân sách, với một khoảng dư nhỏ khi cần cho overhead tuần tự hóa.
- Abort được truyền dưới dạng `AbortSignal` ở mọi nơi để việc hủy bỏ nhất quán.

Định hướng triển khai:

- Thêm một hàm trợ giúp nhỏ (ví dụ `createBudget({ timeoutMs, signal })`) trả về:
  - `signal`: AbortSignal được liên kết
  - `deadlineAtMs`: thời điểm hết hạn tuyệt đối
  - `remainingMs()`: thời gian ngân sách còn lại cho các tác vụ con
- Sử dụng hàm trợ giúp này trong:
  - `src/browser/client-fetch.ts` (HTTP và điều phối trong cùng tiến trình)
  - `src/node-host/runner.ts` (đường dẫn proxy)
  - các triển khai browser action (Playwright và CDP)

### 2. Tách riêng Evaluate Engine (Đường dẫn CDP)

Thêm một triển khai evaluate dựa trên CDP không dùng chung hàng đợi lệnh theo từng trang của Playwright. Thuộc tính chính là kênh truyền evaluate sử dụng một kết nối WebSocket riêng
và một phiên CDP riêng được gắn vào target.

Định hướng triển khai:

- Module mới, ví dụ `src/browser/cdp-evaluate.ts`, thực hiện:
  - Kết nối tới endpoint CDP đã cấu hình (socket cấp trình duyệt).
  - Sử dụng `Target.attachToTarget({ targetId, flatten: true })` để lấy `sessionId`.
  - Chạy một trong hai:
    - `Runtime.evaluate` cho evaluate ở cấp trang, hoặc
    - `DOM.resolveNode` cùng với `Runtime.callFunctionOn` cho evaluate ở cấp phần tử.
  - Khi timeout hoặc abort:
    - Gửi `Runtime.terminateExecution` theo cơ chế best-effort cho phiên.
    - Đóng WebSocket và trả về lỗi rõ ràng.

Lưu ý:

- Việc này vẫn thực thi JavaScript trong trang, vì vậy việc chấm dứt có thể gây ra tác dụng phụ. Lợi ích
  là không làm kẹt hàng đợi Playwright và có thể hủy ở tầng transport
  bằng cách đóng phiên CDP.

### 3. Câu chuyện Ref (Nhắm mục tiêu phần tử mà không cần viết lại toàn bộ)

Phần khó là nhắm mục tiêu phần tử. CDP cần một DOM handle hoặc `backendDOMNodeId`, trong khi
hiện nay hầu hết các browser action sử dụng Playwright locator dựa trên ref từ snapshot.

Cách tiếp cận khuyến nghị: giữ nguyên các ref hiện có, nhưng gắn thêm một id có thể phân giải bởi CDP (tùy chọn).

#### 3.1 Mở rộng thông tin Ref được lưu trữ

Mở rộng metadata role ref được lưu để có thể bao gồm id CDP:

- Hiện tại: `{ role, name, nth }`
- Đề xuất: `{ role, name, nth, backendDOMNodeId?: number }`

Điều này giúp tất cả các action dựa trên Playwright hiện tại tiếp tục hoạt động và cho phép CDP evaluate chấp nhận cùng giá trị `ref` khi `backendDOMNodeId` khả dụng.

#### 3.2 Điền backendDOMNodeId tại thời điểm Snapshot

Khi tạo role snapshot:

1. Tạo bản đồ role ref hiện có như hiện nay (role, name, nth).
2. Lấy cây AX qua CDP (`Accessibility.getFullAXTree`) và tính toán một bản đồ song song
   `(role, name, nth) -> backendDOMNodeId` sử dụng cùng quy tắc xử lý trùng lặp.
3. Gộp id trở lại vào thông tin ref đã lưu cho tab hiện tại.

Nếu việc ánh xạ thất bại cho một ref, hãy để `backendDOMNodeId` là undefined. Điều này khiến tính năng trở thành
best-effort và an toàn để triển khai.

#### 3.3 Đánh giá hành vi với Ref

Trong `act:evaluate`:

- Nếu có `ref` và có `backendDOMNodeId`, chạy evaluate phần tử qua CDP.
- Nếu có `ref` nhưng không có `backendDOMNodeId`, quay về đường Playwright (với
  cơ chế an toàn).

Tùy chọn lối thoát:

- Mở rộng cấu trúc yêu cầu để chấp nhận trực tiếp `backendDOMNodeId` cho callers nâng cao (và
  để gỡ lỗi), đồng thời giữ `ref` là giao diện chính.

### 4. Giữ một cơ chế khôi phục như phương án cuối cùng

Ngay cả với CDP evaluate, vẫn có những cách khác có thể làm treo một tab hoặc kết nối. Giữ các cơ chế khôi phục hiện có (terminate execution + disconnect Playwright) như phương án cuối cùng
cho:

- callers cũ
- các môi trường nơi CDP attach bị chặn
- các trường hợp biên không lường trước của Playwright

## Kế hoạch triển khai (Một vòng lặp)

### Hạng mục bàn giao

- Một engine evaluate dựa trên CDP chạy bên ngoài hàng đợi lệnh theo từng trang của Playwright.
- Một ngân sách timeout/abort end-to-end duy nhất được sử dụng nhất quán bởi callers và handlers.
- Metadata của Ref có thể tùy chọn mang theo `backendDOMNodeId` cho evaluate phần tử.
- `act:evaluate` ưu tiên engine CDP khi có thể và quay về Playwright khi không thể.
- Các bài kiểm thử chứng minh rằng evaluate bị treo không làm kẹt các hành động sau đó.
- Logs/metrics giúp hiển thị rõ lỗi và các lần fallback.

### Danh sách kiểm tra triển khai

1. Thêm một helper "budget" dùng chung để liên kết `timeoutMs` + `AbortSignal` từ upstream thành:
   - một `AbortSignal` duy nhất
   - một deadline tuyệt đối
   - một helper `remainingMs()` cho các thao tác downstream
2. Cập nhật tất cả các đường gọi để sử dụng helper đó ώστε `timeoutMs` có cùng ý nghĩa ở mọi nơi:
   - `src/browser/client-fetch.ts` (HTTP và điều phối trong cùng tiến trình)
   - `src/node-host/runner.ts` (đường proxy node)
   - Các wrapper CLI gọi `/act` (thêm `--timeout-ms` vào `browser evaluate`)
3. Triển khai `src/browser/cdp-evaluate.ts`:
   - kết nối tới socket CDP ở cấp trình duyệt
   - `Target.attachToTarget` để lấy `sessionId`
   - chạy `Runtime.evaluate` cho page evaluate
   - chạy `DOM.resolveNode` + `Runtime.callFunctionOn` cho element evaluate
   - khi timeout/abort: cố gắng `Runtime.terminateExecution` rồi đóng socket
4. Mở rộng metadata ref vai trò đã lưu để có thể tùy chọn bao gồm `backendDOMNodeId`:
   - giữ nguyên hành vi `{ role, name, nth }` hiện có cho các hành động Playwright
   - thêm `backendDOMNodeId?: number` để nhắm mục tiêu phần tử CDP
5. Điền `backendDOMNodeId` trong quá trình tạo snapshot (best-effort):
   - lấy cây AX qua CDP (`Accessibility.getFullAXTree`)
   - tính toán ánh xạ `(role, name, nth) -> backendDOMNodeId` và hợp nhất vào ref map đã lưu
   - nếu ánh xạ mơ hồ hoặc thiếu, để id là undefined
6. Cập nhật định tuyến `act:evaluate`:
   - nếu không có `ref`: luôn sử dụng CDP evaluate
   - nếu `ref` phân giải được thành `backendDOMNodeId`: sử dụng CDP element evaluate
   - ngược lại: quay về Playwright evaluate (vẫn có giới hạn và có thể hủy)
7. Giữ đường khôi phục "last resort" hiện có làm phương án dự phòng, không phải đường mặc định.
8. Thêm các bài kiểm thử:
   - evaluate bị treo sẽ hết thời gian trong phạm vi ngân sách và thao tác click/type tiếp theo vẫn thành công
   - abort hủy evaluate (ngắt kết nối client hoặc timeout) và giải phóng cho các hành động tiếp theo
   - lỗi ánh xạ sẽ quay về Playwright một cách gọn gàng
9. Thêm khả năng quan sát:
   - thời lượng evaluate và bộ đếm timeout
   - mức sử dụng terminateExecution
   - tỷ lệ fallback (CDP -> Playwright) và lý do

### Tiêu chí chấp nhận

- `act:evaluate` bị treo có chủ đích sẽ trả về trong phạm vi ngân sách của caller và không làm
  tab bị kẹt cho các hành động sau đó.
- `timeoutMs` hoạt động nhất quán trên CLI, agent tool, node proxy và các lời gọi trong cùng tiến trình.
- Nếu `ref` có thể được ánh xạ sang `backendDOMNodeId`, element evaluate sẽ dùng CDP; nếu không, đường
  fallback vẫn được giới hạn và có thể khôi phục.

## Kế hoạch kiểm thử

- Unit tests:
  - logic khớp `(role, name, nth)` giữa role refs và các nút trong cây AX.
  - hành vi của helper ngân sách (headroom, phép tính thời gian còn lại).
- Integration tests:
  - CDP evaluate timeout trả về trong phạm vi ngân sách và không chặn hành động tiếp theo.
  - Abort hủy evaluate và kích hoạt terminate theo best-effort.
- Contract tests:
  - Đảm bảo `BrowserActRequest` và `BrowserActResponse` vẫn tương thích.

## Rủi ro và biện pháp giảm thiểu

- Ánh xạ không hoàn hảo:
  - Biện pháp: ánh xạ best-effort, fallback sang Playwright evaluate và bổ sung công cụ debug.
- `Runtime.terminateExecution` có tác dụng phụ:
  - Biện pháp: chỉ sử dụng khi timeout/abort và ghi nhận hành vi trong thông báo lỗi.
- Chi phí phát sinh thêm:
  - Biện pháp: chỉ lấy cây AX khi cần snapshot, cache theo từng target và giữ
    phiên CDP tồn tại trong thời gian ngắn.
- Giới hạn của extension relay:
  - Biện pháp: sử dụng API attach cấp trình duyệt khi không có socket theo từng trang,
    và giữ đường Playwright hiện tại làm fallback.

## Câu hỏi mở

- Có nên cho phép cấu hình engine mới dưới dạng `playwright`, `cdp` hoặc `auto` không?
- Chúng ta có nên cung cấp định dạng "nodeRef" mới cho người dùng nâng cao, hay chỉ giữ `ref`?
- Ảnh chụp frame và ảnh chụp theo phạm vi selector nên tham gia vào việc ánh xạ AX như thế nào?
