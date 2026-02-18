---
title: "Ngữ cảnh"
---

# Ngữ cảnh

Nó bị ràng buộc bởi **cửa sổ ngữ cảnh** của mô hình (giới hạn token). `/status` → xem nhanh “cửa sổ của tôi đang đầy tới mức nào?” + cài đặt phiên.

Mô hình tư duy cho người mới bắt đầu:

- **System prompt** (do OpenClaw xây dựng): quy tắc, công cụ, danh sách Skills, thời gian/thời gian chạy và các tệp workspace được chèn.
- **Lịch sử hội thoại**: các tin nhắn của bạn + tin nhắn của trợ lý trong phiên này.
- **Lời gọi/kết quả công cụ + tệp đính kèm**: đầu ra lệnh, đọc tệp, hình ảnh/âm thanh, v.v.

Ngữ cảnh _không giống_ “bộ nhớ”: bộ nhớ có thể được lưu trên đĩa và tải lại sau; ngữ cảnh là những gì nằm trong cửa sổ hiện tại của mô hình.

## Khởi động nhanh (kiểm tra ngữ cảnh)

- `/status` → xem nhanh “cửa sổ của tôi đầy đến mức nào?” + cài đặt phiên.
- `/context list` → những gì được chèn + kích thước ước tính (theo từng tệp + tổng).
- `/context detail` → phân tích sâu hơn: kích thước theo từng tệp, theo từng schema công cụ, theo từng mục skill, và kích thước system prompt.
- `/usage tokens` → thêm chân trang mức sử dụng theo từng phản hồi vào các câu trả lời bình thường.
- `/compact` → tóm tắt lịch sử cũ thành một mục gọn để giải phóng không gian cửa sổ.

Xem thêm: [Slash commands](/tools/slash-commands), [Mức dùng & chi phí token](/reference/token-use), [Nén](/concepts/compaction).

## Ví dụ đầu ra

Giá trị thay đổi theo mô hình, nhà cung cấp, chính sách công cụ và những gì có trong workspace của bạn.

### `/context list`

```
🧠 Context breakdown
Workspace: <workspaceDir>
Bootstrap max/file: 20,000 chars
Sandbox: mode=non-main sandboxed=false
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars (~5,976 tok))

Injected workspace files:
- AGENTS.md: OK | raw 1,742 chars (~436 tok) | injected 1,742 chars (~436 tok)
- SOUL.md: OK | raw 912 chars (~228 tok) | injected 912 chars (~228 tok)
- TOOLS.md: TRUNCATED | raw 54,210 chars (~13,553 tok) | injected 20,962 chars (~5,241 tok)
- IDENTITY.md: OK | raw 211 chars (~53 tok) | injected 211 chars (~53 tok)
- USER.md: OK | raw 388 chars (~97 tok) | injected 388 chars (~97 tok)
- HEARTBEAT.md: MISSING | raw 0 | injected 0
- BOOTSTRAP.md: OK | raw 0 chars (~0 tok) | injected 0 chars (~0 tok)

Skills list (system prompt text): 2,184 chars (~546 tok) (12 skills)
Tools: read, edit, write, exec, process, browser, message, sessions_send, …
Tool list (system prompt text): 1,032 chars (~258 tok)
Tool schemas (JSON): 31,988 chars (~7,997 tok) (counts toward context; not shown as text)
Tools: (same as above)

Session tokens (cached): 14,250 total / ctx=32,000
```

### `/context detail`

```
🧠 Context breakdown (detailed)
…
Top skills (prompt entry size):
- frontend-design: 412 chars (~103 tok)
- oracle: 401 chars (~101 tok)
… (+10 more skills)

Top tools (schema size):
- browser: 9,812 chars (~2,453 tok)
- exec: 6,240 chars (~1,560 tok)
… (+N more tools)
```

## Những gì được tính vào cửa sổ ngữ cảnh

Mọi thứ mà mô hình nhận được đều được tính, bao gồm:

- System prompt (tất cả các phần).
- Lịch sử hội thoại.
- Lời gọi công cụ + kết quả công cụ.
- Tệp đính kèm/bản chép (hình ảnh/âm thanh/tệp).
- Tóm tắt nén và các tạo phẩm cắt tỉa.
- “Wrapper” của nhà cung cấp hoặc header ẩn (không hiển thị, nhưng vẫn được tính).

## Cách OpenClaw xây dựng system prompt

Nó bao gồm: It includes:

- Danh sách công cụ + mô tả ngắn.
- Danh sách Skills (chỉ metadata; xem bên dưới).
- Vị trí workspace.
- Thời gian (UTC + thời gian người dùng đã chuyển đổi nếu được cấu hình).
- Metadata thời gian chạy (host/OS/mô hình/suy nghĩ).
- Các tệp bootstrap workspace được chèn dưới **Project Context**.

Phân tích đầy đủ: [System Prompt](/concepts/system-prompt).

## Các tệp workspace được chèn (Project Context)

Theo mặc định, OpenClaw chèn một tập tệp workspace cố định (nếu có):

- `AGENTS.md`
- `SOUL.md`
- `TOOLS.md`
- `IDENTITY.md`
- `USER.md`
- `HEARTBEAT.md`
- `BOOTSTRAP.md` (chỉ lần chạy đầu tiên)

Large files are truncated per-file using `agents.defaults.bootstrapMaxChars` (default `20000` chars). 1. `/context` hiển thị kích thước **raw vs injected** và liệu có xảy ra cắt bớt hay không.

## Skills: những gì được chèn vs tải theo nhu cầu

2. System prompt bao gồm một **danh sách kỹ năng** gọn nhẹ (tên + mô tả + vị trí). 3. Danh sách này có chi phí overhead thực sự.

4. Hướng dẫn kỹ năng _không_ được bao gồm theo mặc định. 5. Mô hình được kỳ vọng sẽ `đọc` `SKILL.md` của kỹ năng **chỉ khi cần**.

## Công cụ: có hai loại chi phí

Công cụ ảnh hưởng đến ngữ cảnh theo hai cách:

1. **Văn bản danh sách công cụ** trong system prompt (những gì bạn thấy là “Tooling”).
2. **Tool schemas** (JSON). 6. Những thứ này được gửi tới mô hình để nó có thể gọi công cụ. 7. Chúng được tính vào context ngay cả khi bạn không thấy chúng dưới dạng văn bản thuần.

`/context detail` phân tích các schema công cụ lớn nhất để bạn thấy yếu tố nào chiếm ưu thế.

## Lệnh, chỉ thị và “phím tắt nội tuyến”

Slash commands are handled by the Gateway. 8. Có một vài hành vi khác nhau:

- **Lệnh độc lập**: một tin nhắn chỉ chứa `/...` sẽ chạy như một lệnh.
- **Chỉ thị**: `/think`, `/verbose`, `/reasoning`, `/elevated`, `/model`, `/queue` được loại bỏ trước khi mô hình nhìn thấy tin nhắn.
  - Tin nhắn chỉ có chỉ thị sẽ lưu cài đặt phiên.
  - Chỉ thị nội tuyến trong một tin nhắn bình thường hoạt động như gợi ý theo từng tin nhắn.
- **Phím tắt nội tuyến** (chỉ người gửi trong danh sách cho phép): một số token `/...` nhất định bên trong tin nhắn bình thường có thể chạy ngay (ví dụ: “hey /status”), và được loại bỏ trước khi mô hình nhìn thấy phần văn bản còn lại.

Chi tiết: [Slash commands](/tools/slash-commands).

## Phiên, nén và cắt tỉa (những gì được lưu)

Những gì được lưu giữa các tin nhắn phụ thuộc vào cơ chế:

- **Lịch sử bình thường** được lưu trong bản chép phiên cho đến khi bị nén/cắt tỉa theo chính sách.
- **Nén** lưu một bản tóm tắt vào bản chép và giữ nguyên các tin nhắn gần đây.
- **Cắt tỉa** loại bỏ kết quả công cụ cũ khỏi prompt _trong bộ nhớ_ cho một lần chạy, nhưng không ghi lại bản chép.

Tài liệu: [Session](/concepts/session), [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning).

## `/context` thực sự báo cáo gì

`/context` ưu tiên báo cáo system prompt **được xây dựng cho lần chạy** mới nhất khi có sẵn:

- `System prompt (run)` = được chụp từ lần chạy nhúng (có khả năng dùng công cụ) gần nhất và được lưu trong kho phiên.
- `System prompt (estimate)` = được tính động khi không có báo cáo lần chạy (hoặc khi chạy qua backend CLI không tạo báo cáo).

Dù theo cách nào, nó báo cáo kích thước và các yếu tố đóng góp lớn nhất; nó **không** đổ toàn bộ system prompt hay các schema công cụ.


