---
title: "Các mức Thinking"
---

# Các mức Thinking (chỉ thị /think)

## Chức năng

- Chỉ thị nội tuyến trong bất kỳ nội dung gửi vào nào: `/t <level>`, `/think:<level>`, hoặc `/thinking <level>`.
- Các mức (bí danh): `off | minimal | low | medium | high | xhigh` (chỉ dành cho mô hình GPT-5.2 + Codex)
  - minimal → “suy nghĩ”
  - low → “suy nghĩ kỹ”
  - medium → “suy nghĩ kỹ hơn”
  - high → “ultrathink” (ngân sách tối đa)
  - xhigh → “ultrathink+” (chỉ dành cho mô hình GPT-5.2 + Codex)
  - `x-high`, `x_high`, `extra-high`, `extra high`, và `extra_high` ánh xạ tới `xhigh`.
  - `highest`, `max` ánh xạ tới `high`.
- Ghi chú theo nhà cung cấp:
  - Z.AI (`zai/*`) chỉ hỗ trợ chế độ suy nghĩ nhị phân (`on`/`off`). Bất kỳ mức nào khác `off` đều được xem là `on` (ánh xạ thành `low`).

## Thứ tự phân giải

1. Chỉ thị nội tuyến trên tin nhắn (chỉ áp dụng cho tin nhắn đó).
2. Ghi đè theo phiên (đặt bằng cách gửi một tin nhắn chỉ chứa chỉ thị).
3. Mặc định toàn cục (`agents.defaults.thinkingDefault` trong cấu hình).
4. Dự phòng: low cho các mô hình có khả năng suy luận; tắt cho các mô hình khác.

## Thiết lập mặc định theo phiên

- Gửi một tin nhắn **chỉ** gồm chỉ thị (cho phép khoảng trắng), ví dụ: `/think:medium` hoặc `/t high`.
- Thiết lập này được giữ cho phiên hiện tại (mặc định theo từng người gửi); được xóa bởi `/think:off` hoặc khi phiên bị reset do nhàn rỗi.
- Phản hồi xác nhận được gửi (`Thinking level set to high.` / `Thinking disabled.`). Nếu mức không hợp lệ (ví dụ: `/thinking big`), lệnh sẽ bị từ chối kèm gợi ý và trạng thái phiên sẽ giữ nguyên.
- Gửi `/think` (hoặc `/think:`) không kèm đối số để xem mức thinking hiện tại.

## Áp dụng theo tác tử

- **Pi nhúng**: mức đã phân giải được truyền vào runtime tác tử Pi trong tiến trình.

## Chỉ thị verbose (/verbose hoặc /v)

- Các mức: `on` (tối thiểu) | `full` | `off` (mặc định).
- Tin nhắn chỉ chứa chỉ thị sẽ bật/tắt verbose theo phiên và phản hồi `Verbose logging enabled.` / `Verbose logging disabled.`; mức không hợp lệ trả về gợi ý mà không thay đổi trạng thái.
- `/verbose off` lưu một ghi đè theo phiên rõ ràng; xóa bằng UI Sessions bằng cách chọn `inherit`.
- Chỉ thị nội tuyến chỉ ảnh hưởng đến tin nhắn đó; mặc định theo phiên/toàn cục áp dụng cho các trường hợp khác.
- Gửi `/verbose` (hoặc `/verbose:`) không kèm đối số để xem mức verbose hiện tại.
- Khi verbose được bật, các agent xuất kết quả công cụ có cấu trúc (Pi, các agent JSON khác) sẽ gửi lại mỗi lần gọi công cụ như một thông điệp chỉ chứa metadata riêng biệt, có tiền tố `<emoji> <tool-name>: <arg>` khi có sẵn (đường dẫn/lệnh). Các tóm tắt công cụ này được gửi ngay khi mỗi công cụ bắt đầu (các bong bóng riêng biệt), không phải dưới dạng delta streaming.
- Khi verbose là `full`, đầu ra của công cụ cũng được chuyển tiếp sau khi hoàn tất (bong bóng riêng biệt, được cắt ngắn ở độ dài an toàn). Nếu bạn chuyển `/verbose on|full|off` trong khi một tiến trình đang chạy, các bong bóng công cụ tiếp theo sẽ tuân theo thiết lập mới.

## Hiển thị suy luận (/reasoning)

- Các mức: `on|off|stream`.
- Tin nhắn chỉ chứa chỉ thị sẽ bật/tắt việc hiển thị các khối thinking trong phản hồi.
- Khi được bật, suy luận được gửi như một **tin nhắn riêng** với tiền tố `Reasoning:`.
- `stream` (chỉ Telegram): stream suy luận vào bong bóng nháp Telegram trong khi phản hồi đang tạo, sau đó gửi câu trả lời cuối cùng không kèm suy luận.
- Bí danh: `/reason`.
- Gửi `/reasoning` (hoặc `/reasoning:`) không kèm đối số để xem mức suy luận hiện tại.

## Liên quan

- Tài liệu về Elevated mode nằm tại [Elevated mode](/tools/elevated).

## Heartbeat

- Nội dung probe Heartbeat là lời nhắc heartbeat đã cấu hình (mặc định: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`). Các chỉ thị nội tuyến trong thông điệp heartbeat được áp dụng như bình thường (nhưng tránh thay đổi thiết lập mặc định của phiên từ heartbeat).
- Mặc định, Heartbeat chỉ gửi payload cuối cùng. Để gửi thêm thông điệp `Reasoning:` riêng biệt (khi có), đặt `agents.defaults.heartbeat.includeReasoning: true` hoặc theo từng agent `agents.list[].heartbeat.includeReasoning: true`.

## Giao diện web chat

- Bộ chọn thinking trên web chat phản chiếu mức đã lưu của phiên từ kho phiên/cấu hình đầu vào khi trang tải.
- Chọn một mức khác chỉ áp dụng cho tin nhắn kế tiếp (`thinkingOnce`); sau khi gửi, bộ chọn sẽ quay lại mức đã lưu của phiên.
- Để thay đổi mặc định theo phiên, gửi một chỉ thị `/think:<level>` (như trước); bộ chọn sẽ phản ánh sau lần tải lại tiếp theo.


