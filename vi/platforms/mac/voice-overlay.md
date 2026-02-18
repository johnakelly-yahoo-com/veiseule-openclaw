---
title: "Lớp phủ giọng nói"
---

# Vòng đời Lớp phủ Giọng nói (macOS)

Đối tượng: các cộng tác viên ứng dụng macOS. Mục tiêu: giữ cho lớp phủ giọng nói hoạt động ổn định khi wake-word và push-to-talk chồng lấn.

## Ý định hiện tại

- Nếu lớp phủ đã hiển thị do wake-word và người dùng nhấn phím nóng, phiên phím nóng sẽ _adopts_ (tiếp nhận) nội dung văn bản hiện có thay vì đặt lại. Lớp phủ vẫn hiển thị trong khi phím nóng được giữ. Khi người dùng thả phím: gửi nếu có văn bản sau khi đã loại bỏ khoảng trắng thừa, nếu không thì đóng lớp phủ.
- Chỉ dùng từ đánh thức vẫn tự động gửi khi im lặng; nhấn‑để‑nói gửi ngay khi thả.

## Đã triển khai (9 Thg 12, 2025)

- Các phiên lớp phủ giờ đây mang theo một token cho mỗi lần ghi âm (wake-word hoặc push-to-talk). Các cập nhật partial/final/send/dismiss/level sẽ bị loại bỏ nếu token không khớp, nhằm tránh các callback cũ không còn hợp lệ.
- Push-to-talk sẽ sử dụng mọi văn bản đang hiển thị trên lớp phủ làm tiền tố (vì vậy khi nhấn phím nóng trong lúc lớp phủ wake đang mở, văn bản hiện có sẽ được giữ lại và nối thêm lời nói mới). Hệ thống chờ tối đa 1,5 giây để nhận bản ghi cuối cùng trước khi quay về sử dụng văn bản hiện tại.
- Ghi log chuông/lớp phủ được phát tại `info` trong các danh mục `voicewake.overlay`, `voicewake.ptt` và `voicewake.chime` (bắt đầu phiên, partial, final, send, dismiss, lý do chuông).

## Bước tiếp theo

1. **VoiceSessionCoordinator (actor)**
   - Sở hữu đúng một `VoiceSession` tại một thời điểm.
   - API (dựa trên token): `beginWakeCapture`, `beginPushToTalk`, `updatePartial`, `endCapture`, `cancel`, `applyCooldown`.
   - Bỏ các callback mang token cũ (ngăn các bộ nhận dạng cũ mở lại lớp phủ).
2. **VoiceSession (model)**
   - Trường: `token`, `source` (wakeWord|pushToTalk), văn bản committed/volatile, cờ chuông, bộ hẹn giờ (tự động gửi, nhàn rỗi), `overlayMode` (display|editing|sending), thời hạn cooldown.
3. **Liên kết lớp phủ**
   - `VoiceSessionPublisher` (`ObservableObject`) phản chiếu phiên đang hoạt động vào SwiftUI.
   - `VoiceWakeOverlayView` chỉ render thông qua publisher; không bao giờ thay đổi trực tiếp các singleton toàn cục.
   - Các hành động người dùng trên lớp phủ (`sendNow`, `dismiss`, `edit`) gọi ngược vào coordinator với token của phiên.
4. **Luồng gửi hợp nhất**
   - Khi `endCapture`: nếu văn bản đã cắt gọn rỗng → đóng; nếu không thì `performSend(session:)` (phát chuông gửi một lần, chuyển tiếp, đóng).
   - Nhấn‑để‑nói: không trì hoãn; từ đánh thức: có thể trì hoãn để tự động gửi.
   - Áp dụng một cooldown ngắn cho runtime từ đánh thức sau khi nhấn‑để‑nói kết thúc để từ đánh thức không kích hoạt lại ngay.
5. **Ghi log**
   - Coordinator phát log `.info` trong subsystem `bot.molt`, các danh mục `voicewake.overlay` và `voicewake.chime`.
   - Các sự kiện chính: `session_started`, `adopted_by_push_to_talk`, `partial`, `finalized`, `send`, `dismiss`, `cancel`, `cooldown`.

## Danh sách kiểm tra gỡ lỗi

- Stream log trong khi tái hiện lớp phủ bị “dính”:

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- Xác minh chỉ có một token phiên đang hoạt động; các callback cũ phải bị coordinator bỏ.

- Đảm bảo việc thả nhấn‑để‑nói luôn gọi `endCapture` với token đang hoạt động; nếu văn bản rỗng, kỳ vọng `dismiss` mà không có chuông hoặc gửi.

## Các bước di chuyển (đề xuất)

1. Thêm `VoiceSessionCoordinator`, `VoiceSession` và `VoiceSessionPublisher`.
2. Tái cấu trúc `VoiceWakeRuntime` để tạo/cập nhật/kết thúc phiên thay vì chạm trực tiếp vào `VoiceWakeOverlayController`.
3. Tái cấu trúc `VoicePushToTalk` để kế thừa các phiên hiện có và gọi `endCapture` khi thả; áp dụng cooldown runtime.
4. Kết nối `VoiceWakeOverlayController` với publisher; loại bỏ các lời gọi trực tiếp từ runtime/PTT.
5. Thêm các bài kiểm thử tích hợp cho việc kế thừa phiên, cooldown và đóng khi văn bản rỗng.
