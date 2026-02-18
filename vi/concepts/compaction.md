---
title: "Nén"
---

# Cửa sổ ngữ cảnh & Nén

Các cuộc trò chuyện dài tích lũy thông điệp và kết quả công cụ; khi cửa sổ trở nên chật, OpenClaw **nén (compacts)** lịch sử cũ để nằm trong giới hạn. Long-running chats accumulate messages and tool results; once the window is tight, OpenClaw **compacts** older history to stay within limits.

## Nén là gì

Nén (compaction) **tóm tắt các cuộc trò chuyện cũ hơn** thành một mục tóm tắt ngắn gọn và giữ nguyên các tin nhắn gần đây. Bản tóm tắt được lưu trong lịch sử phiên, vì vậy các yêu cầu sau này sẽ sử dụng:

- Bản tóm tắt nén
- Các tin nhắn gần đây sau điểm nén

Nén được **lưu bền vững** trong lịch sử JSONL của phiên.

## Cấu hình

Xem [Cấu hình & chế độ nén](/concepts/compaction) cho các thiết lập `agents.defaults.compaction`.

## Tự động nén (bật mặc định)

Khi một phiên tiến gần hoặc vượt quá cửa sổ ngữ cảnh của mô hình, OpenClaw kích hoạt tự động nén và có thể thử lại yêu cầu ban đầu bằng ngữ cảnh đã được nén.

Bạn sẽ thấy:

- `🧹 Auto-compaction complete` ở chế độ verbose
- `/status` hiển thị `🧹 Compactions: <count>`

Xem [Memory](/concepts/memory) để biết chi tiết và cấu hình. See [Memory](/concepts/memory) for details and config.

## Nén thủ công

Dùng `/compact` (tùy chọn kèm hướng dẫn) để buộc chạy một lượt nén:

```
/compact Focus on decisions and open questions
```

## Nguồn cửa sổ ngữ cảnh

Cửa sổ ngữ cảnh phụ thuộc vào từng mô hình. OpenClaw sử dụng định nghĩa mô hình từ danh mục nhà cung cấp đã cấu hình để xác định các giới hạn.

## Nén vs cắt tỉa

- **Nén**: tóm tắt và **lưu bền vững** vào JSONL.
- **Cắt tỉa phiên**: chỉ cắt bớt **kết quả công cụ** cũ, **trong bộ nhớ**, theo từng yêu cầu.

Xem [/concepts/session-pruning](/concepts/session-pruning) để biết chi tiết về cắt tỉa.

## Mẹo

- Dùng `/compact` khi phiên có cảm giác ì trệ hoặc ngữ cảnh bị phình to.
- Các đầu ra công cụ lớn đã được cắt ngắn sẵn; cắt tỉa có thể tiếp tục giảm sự tích tụ của kết quả công cụ.
- Nếu bạn cần bắt đầu lại từ đầu, `/new` hoặc `/reset` sẽ tạo một id phiên mới.


