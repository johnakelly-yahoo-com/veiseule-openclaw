# AGENTS.md - không gian làm việc dịch tài liệu ja-JP

## Đọc khi

- Duy trì `docs/ja-JP/**`
- Cập nhật pipeline dịch tiếng Nhật (glossary/TM/prompt)
- Xử lý phản hồi hoặc lỗi hồi quy của bản dịch tiếng Nhật

## Pipeline (docs-i18n)

- Tài liệu nguồn: `docs/**/*.md`
- Tài liệu đích: `docs/ja-JP/**/*.md`
- Thuật ngữ (Glossary): `docs/.i18n/glossary.ja-JP.json`
- Bộ nhớ dịch (Translation memory): `docs/.i18n/ja-JP.tm.jsonl`
- Quy tắc prompt: `scripts/docs-i18n/prompt.go`

Các lệnh chạy phổ biến:

```bash
# Bulk (chế độ doc; có thể chạy song song)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc -parallel 6 ../../docs/**/*.md

# Single file
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode doc ../../docs/start/getting-started.md

# Small patches (chế độ segment; sử dụng TM; không song song)
cd scripts/docs-i18n
go run . -docs ../../docs -lang ja-JP -mode segment ../../docs/start/getting-started.md
```

Lưu ý:

- Ưu tiên chế độ `doc` để dịch toàn bộ trang; dùng chế độ `segment` cho các chỉnh sửa nhỏ.
- Nếu một tệp rất lớn bị timeout, hãy chỉnh sửa có mục tiêu hoặc chia nhỏ trang trước khi chạy lại.
- Sau khi dịch, kiểm tra nhanh: các đoạn/khối code không thay đổi, liên kết/anchor không thay đổi, placeholder được giữ nguyên.
