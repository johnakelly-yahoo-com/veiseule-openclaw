---
summary: "Chạy OpenClaw trong container Podman rootless"
read_when:
  - Bạn muốn một gateway chạy trong container với Podman thay vì Docker
title: "Podman"
---

# Podman

Chạy OpenClaw gateway trong container Podman **rootless**. Sử dụng cùng image như Docker (build từ repo [Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)).

## Yêu cầu

- Podman (rootless)
- Cần quyền sudo cho thiết lập một lần (tạo user, build image)

## Bắt đầu nhanh

**1. Thiết lập một lần** (từ thư mục gốc của repo; tạo user, build image, cài đặt script khởi chạy):

```bash
./setup-podman.sh
```

Lệnh này cũng tạo một file `~openclaw/.openclaw/openclaw.json` tối thiểu (đặt `gateway.mode="local"`) để gateway có thể khởi động mà không cần chạy wizard.

Mặc định container **không** được cài đặt như một dịch vụ systemd; bạn sẽ khởi động thủ công (xem bên dưới). Đối với môi trường production với tự động khởi động và khởi động lại, hãy cài đặt dưới dạng dịch vụ người dùng systemd Quadlet thay thế:

```bash
./setup-podman.sh --quadlet
```

(Hoặc đặt `OPENCLAW_PODMAN_QUADLET=1`; sử dụng `--container` để chỉ cài đặt container và script khởi chạy.)

**2. Khởi động gateway** (thủ công, để kiểm tra nhanh):

```bash
./scripts/run-openclaw-podman.sh launch
```

**3. Wizard khởi tạo** (ví dụ: để thêm channel hoặc provider):

```bash
./scripts/run-openclaw-podman.sh launch setup
```

Sau đó mở `http://127.0.0.1:18789/` và sử dụng token trong `~openclaw/.openclaw/.env` (hoặc giá trị được in ra bởi quá trình setup).

## Systemd (Quadlet, tùy chọn)

Nếu bạn đã chạy `./setup-podman.sh --quadlet` (hoặc `OPENCLAW_PODMAN_QUADLET=1`), một unit [Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html) sẽ được cài đặt để gateway chạy như một dịch vụ systemd ở cấp người dùng cho user openclaw. Dịch vụ sẽ được bật và khởi động ở cuối quá trình thiết lập.

- **Khởi động:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
- **Dừng:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
- **Trạng thái:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
- **Log:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

File quadlet nằm tại `~openclaw/.config/containers/systemd/openclaw.container`. Để thay đổi cổng hoặc biến môi trường, chỉnh sửa file đó (hoặc file `.env` mà nó sử dụng), sau đó chạy `sudo systemctl --machine openclaw@ --user daemon-reload` và khởi động lại dịch vụ. Khi khởi động, dịch vụ sẽ tự động chạy nếu lingering được bật cho openclaw (quá trình thiết lập sẽ thực hiện việc này khi loginctl khả dụng).

Để thêm quadlet **sau** khi thiết lập ban đầu không sử dụng nó, chạy lại: `./setup-podman.sh --quadlet`.

## Người dùng openclaw (không đăng nhập)

`setup-podman.sh` tạo một người dùng hệ thống chuyên dụng `openclaw`:

- **Shell:** `nologin` — không cho phép đăng nhập tương tác; giảm bề mặt tấn công.

- **Home:** ví dụ `/home/openclaw` — chứa `~/.openclaw` (cấu hình, workspace) và script khởi chạy `run-openclaw-podman.sh`.

- **Rootless Podman:** Người dùng phải có phạm vi **subuid** và **subgid**. Nhiều bản phân phối tự động gán các phạm vi này khi người dùng được tạo. Nếu quá trình thiết lập hiển thị cảnh báo, hãy thêm các dòng vào `/etc/subuid` và `/etc/subgid`:

  ```text
  openclaw:100000:65536
  ```

  Sau đó khởi động gateway với người dùng đó (ví dụ từ cron hoặc systemd):

  ```bash
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
  sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
  ```

- **Config:** Chỉ `openclaw` và root có thể truy cập `/home/openclaw/.openclaw`. Để chỉnh sửa cấu hình: sử dụng Control UI khi gateway đang chạy, hoặc `sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`.

## Môi trường và cấu hình

- **Token:** Được lưu trong `~openclaw/.openclaw/.env` dưới dạng `OPENCLAW_GATEWAY_TOKEN`. `setup-podman.sh` và `run-openclaw-podman.sh` sẽ tạo token nếu chưa có (sử dụng `openssl`, `python3` hoặc `od`).
- **Tùy chọn:** Trong tệp `.env` đó, bạn có thể đặt các khóa nhà cung cấp (ví dụ `GROQ_API_KEY`, `OLLAMA_API_KEY`) và các biến môi trường OpenClaw khác.
- **Cổng host:** Theo mặc định, script ánh xạ `18789` (gateway) và `18790` (bridge). Ghi đè ánh xạ cổng **host** bằng `OPENCLAW_PODMAN_GATEWAY_HOST_PORT` và `OPENCLAW_PODMAN_BRIDGE_HOST_PORT` khi khởi chạy.
- **Đường dẫn:** Cấu hình và workspace trên host mặc định là `~openclaw/.openclaw` và `~openclaw/.openclaw/workspace`. Ghi đè các đường dẫn host được script khởi chạy sử dụng bằng `OPENCLAW_CONFIG_DIR` và `OPENCLAW_WORKSPACE_DIR`.

## Các lệnh hữu ích

- **Logs:** Với quadlet: `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`. Với script: `sudo -u openclaw podman logs -f openclaw`
- **Dừng:** Với quadlet: `sudo systemctl --machine openclaw@ --user stop openclaw.service`. Với script: `sudo -u openclaw podman stop openclaw`
- **Khởi động lại:** Với quadlet: `sudo systemctl --machine openclaw@ --user start openclaw.service`. Với script: chạy lại script khởi chạy hoặc `podman start openclaw`
- **Xóa container:** `sudo -u openclaw podman rm -f openclaw` — cấu hình và workspace trên host vẫn được giữ nguyên

## Khắc phục sự cố

- **Permission denied (EACCES) trên config hoặc auth-profiles:** Container mặc định sử dụng `--userns=keep-id` và chạy với cùng uid/gid như người dùng host đang chạy script. Đảm bảo `OPENCLAW_CONFIG_DIR` và `OPENCLAW_WORKSPACE_DIR` trên host thuộc sở hữu của người dùng đó.
- **Gateway không khởi động (thiếu `gateway.mode=local`):** Đảm bảo `~openclaw/.openclaw/openclaw.json` tồn tại và đặt `gateway.mode="local"`. `setup-podman.sh` sẽ tạo tệp này nếu chưa có.
- **Rootless Podman lỗi với người dùng openclaw:** Kiểm tra `/etc/subuid` và `/etc/subgid` có chứa một dòng cho `openclaw` (ví dụ `openclaw:100000:65536`). Thêm nếu thiếu và khởi động lại.
- **Tên container đã được sử dụng:** Script khởi chạy dùng `podman run --replace`, vì vậy container hiện có sẽ được thay thế khi bạn khởi động lại. Để dọn dẹp thủ công: `podman rm -f openclaw`.
- **Không tìm thấy script khi chạy với openclaw:** Đảm bảo đã chạy `setup-podman.sh` để `run-openclaw-podman.sh` được sao chép vào thư mục home của openclaw (ví dụ `/home/openclaw/run-openclaw-podman.sh`).
- **Không tìm thấy dịch vụ Quadlet hoặc không khởi động được:** Chạy `sudo systemctl --machine openclaw@ --user daemon-reload` sau khi chỉnh sửa tệp `.container`. Quadlet yêu cầu cgroups v2: `podman info --format '{{.Host.CgroupsVersion}}'` nên hiển thị `2`.

## Tùy chọn: chạy dưới người dùng của riêng bạn

Để chạy gateway bằng người dùng thông thường của bạn (không dùng người dùng openclaw riêng biệt): build image, tạo `~/.openclaw/.env` với `OPENCLAW_GATEWAY_TOKEN`, và chạy container với `--userns=keep-id` cùng các mount tới `~/.openclaw` của bạn. Script khởi chạy được thiết kế cho luồng openclaw-user; với thiết lập một người dùng, bạn có thể thay vào đó chạy thủ công lệnh `podman run` trong script, trỏ cấu hình và workspace về thư mục home của bạn. Khuyến nghị cho hầu hết người dùng: sử dụng `setup-podman.sh` và chạy dưới người dùng openclaw để cấu hình và tiến trình được tách biệt.

