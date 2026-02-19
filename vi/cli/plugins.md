---
summary: "Tài liệu tham chiếu CLI cho `openclaw plugins` (liệt kê, cài đặt, bật/tắt, doctor)"
read_when:
  - Bạn muốn cài đặt hoặc quản lý các plugin Gateway chạy trong tiến trình
  - Bạn muốn gỡ lỗi các lỗi tải plugin
title: "plugin"
---

# `openclaw plugins`

Quản lý các plugin/extension của Gateway (được tải trong tiến trình).

Liên quan:

- Hệ thống plugin: [Plugins](/tools/plugin)
- Manifest + schema của plugin: [Plugin manifest](/plugins/manifest)
- Tăng cường bảo mật: [Security](/gateway/security)

## Lệnh

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Các plugin đi kèm được phát hành cùng OpenClaw nhưng ban đầu bị tắt. Dùng `plugins enable` để
kích hoạt chúng.

Tất cả plugin phải kèm theo một tệp `openclaw.plugin.json` với JSON Schema nội tuyến
(`configSchema`, kể cả khi rỗng). Thiếu/không hợp lệ manifest hoặc schema sẽ ngăn plugin được tải
và làm thất bại việc xác thực cấu hình.

### Cài đặt

```bash
openclaw plugins install <path-or-spec>
```

Lưu ý bảo mật: hãy coi việc cài plugin giống như chạy mã. Ưu tiên các phiên bản được ghim.

Thông số npm chỉ **giới hạn ở registry** (tên gói + phiên bản/tag tùy chọn). Các thông số Git/URL/file
sẽ bị từ chối. Việc cài đặt dependency chạy với `--ignore-scripts` để đảm bảo an toàn.

Dùng `--link` để tránh sao chép một thư mục cục bộ (thêm vào `plugins.load.paths`):

Dùng `--link` để tránh sao chép một thư mục cục bộ (thêm vào `plugins.load.paths`):

```bash
openclaw plugins install -l ./my-plugin
```

### Gỡ cài đặt

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall` sẽ xóa bản ghi plugin khỏi `plugins.entries`, `plugins.installs`,
danh sách cho phép plugin và các mục `plugins.load.paths` được liên kết khi áp dụng.
Đối với plugin bộ nhớ đang hoạt động, khe bộ nhớ sẽ được đặt lại về `memory-core`.

Theo mặc định, uninstall cũng xóa thư mục cài đặt plugin trong
thư mục gốc extensions của state dir đang hoạt động (`$OPENCLAW_STATE_DIR/extensions/<id>`). Sử dụng
`--keep-files` để giữ lại các tệp trên đĩa.

`--keep-config` được hỗ trợ như một bí danh đã lỗi thời cho `--keep-files`.

### Cập nhật

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Việc cập nhật chỉ áp dụng cho các plugin được cài từ npm (được theo dõi trong `plugins.installs`).

