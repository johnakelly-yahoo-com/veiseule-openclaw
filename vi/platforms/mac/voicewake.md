---
title: "Kích hoạt bằng giọng nói"
---

# Voice Wake & Nhấn-để-nói

## Chế độ

- **Chế độ từ kích hoạt** (mặc định): trình nhận dạng giọng nói luôn bật và chờ các từ kích hoạt (`swabbleTriggerWords`). Khi khớp, hệ thống bắt đầu ghi âm, hiển thị lớp phủ với văn bản tạm thời và tự động gửi sau khi im lặng.
- **Nhấn để nói (giữ phím Option bên phải)**: giữ phím Option bên phải để ghi âm ngay lập tức—không cần từ kích hoạt. Lớp phủ xuất hiện khi giữ phím; thả ra sẽ hoàn tất và chuyển tiếp sau một khoảng trễ ngắn để bạn có thể chỉnh sửa văn bản.

## Hành vi thời gian chạy (từ đánh thức)

- Trình nhận dạng giọng nói chạy trong `VoiceWakeRuntime`.
- Chỉ kích hoạt khi có **khoảng dừng đáng kể** giữa từ kích hoạt và từ tiếp theo (khoảng ~0,55 giây). Lớp phủ/âm báo có thể bắt đầu ngay khi tạm dừng, thậm chí trước khi lệnh bắt đầu.
- Cửa sổ im lặng: 2,0s khi đang nói liên tục, 5,0s nếu chỉ nghe thấy từ kích hoạt.
- Dừng cứng: 120s để ngăn phiên chạy quá lâu.
- Chống dội giữa các phiên: 350ms.
- Overlay được điều khiển qua `VoiceWakeOverlayController` với màu sắc cho trạng thái đã xác nhận/tạm thời.
- Sau khi gửi, trình nhận dạng khởi động lại sạch sẽ để lắng nghe lần kích hoạt tiếp theo.

## Bất biến vòng đời

- Nếu Voice Wake được bật và đã cấp quyền, trình nhận dạng từ đánh thức phải luôn lắng nghe (trừ khi đang có một phiên nhấn-để-nói rõ ràng).
- Trạng thái hiển thị của overlay (bao gồm việc đóng thủ công bằng nút X) không bao giờ được ngăn trình nhận dạng tiếp tục hoạt động.

## Lỗi overlay bị “dính” (trước đây)

Trước đây, nếu overlay bị kẹt hiển thị và bạn đóng thủ công, Voice Wake có thể trông như “chết” vì nỗ lực khởi động lại của runtime có thể bị chặn bởi trạng thái hiển thị của overlay và không có lần khởi động lại tiếp theo được lên lịch.

Gia cố:

- Việc khởi động lại runtime của wake không còn bị chặn bởi trạng thái hiển thị overlay.
- Hoàn tất thao tác đóng overlay sẽ kích hoạt `VoiceWakeRuntime.refresh(...)` qua `VoiceSessionCoordinator`, nên việc đóng bằng nút X luôn tiếp tục lắng nghe.

## Chi tiết nhấn-để-nói

- Phát hiện phím nóng sử dụng một monitor toàn cục `.flagsChanged` cho **Option phải** (`keyCode 61` + `.option`). We only observe events (no swallowing).
- Pipeline thu âm nằm trong `VoicePushToTalk`: bắt đầu Speech ngay, stream các phần tạm thời lên overlay và gọi `VoiceWakeForwarder` khi thả phím.
- Khi nhấn-để-nói bắt đầu, chúng tôi tạm dừng runtime từ đánh thức để tránh xung đột các tap âm thanh; nó sẽ tự động khởi động lại sau khi thả phím.
- Quyền: cần Microphone + Speech; để nhận sự kiện cần phê duyệt Accessibility/Input Monitoring.
- Bàn phím ngoài: một số loại có thể không hiển thị Option phải như mong đợi—hãy cung cấp phím tắt dự phòng nếu người dùng báo bị bỏ sót.

## Cài đặt cho người dùng

- **Voice Wake**: bật runtime từ đánh thức.
- **Giữ Cmd+Fn để nói**: bật chế độ theo dõi nhấn-để-nói. Bị vô hiệu hóa trên macOS < 26.
- Bộ chọn ngôn ngữ & mic, đồng hồ mức âm trực tiếp, bảng từ kích hoạt, trình kiểm thử (chỉ cục bộ; không chuyển tiếp).
- Bộ chọn mic giữ lại lựa chọn cuối cùng nếu thiết bị ngắt kết nối, hiển thị gợi ý đã ngắt và tạm thời chuyển sang mặc định hệ thống cho đến khi thiết bị quay lại.
- **Âm thanh**: phát âm báo khi phát hiện từ kích hoạt và khi gửi; mặc định là âm thanh hệ thống “Glass” của macOS. Bạn có thể chọn bất kỳ tệp nào có thể tải bằng `NSSound` (ví dụ: MP3/WAV/AIFF) cho từng sự kiện hoặc chọn **No Sound**.

## Hành vi chuyển tiếp

- Khi Voice Wake được bật, bản chép lời sẽ được chuyển tiếp tới gateway/tác tử đang hoạt động (cùng chế độ cục bộ vs từ xa như phần còn lại của ứng dụng mac).
- Replies are delivered to the **last-used main provider** (WhatsApp/Telegram/Discord/WebChat). If delivery fails, the error is logged and the run is still visible via WebChat/session logs.

## Payload chuyển tiếp

- `VoiceWakeForwarder.prefixedTranscript(_:)` thêm tiền tố gợi ý máy trước khi gửi. Được dùng chung cho cả chế độ từ kích hoạt và nhấn-để-nói.

## Xác minh nhanh

- Bật nhấn-để-nói, giữ Cmd+Fn, nói, thả: overlay phải hiển thị các phần tạm thời rồi gửi.
- Trong lúc giữ, biểu tượng tai ở thanh menu phải được phóng to (dùng `triggerVoiceEars(ttl:nil)`); thả phím thì trở lại bình thường.


