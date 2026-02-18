---
title: "Khởi tạo tác tử"
sidebarTitle: "Khởi tạo ban đầu"
---

# Khởi tạo tác tử

Khởi tạo ban đầu là nghi thức **chạy lần đầu** để chuẩn bị không gian làm việc của một tác nhân và
collects identity details. It happens after onboarding, when the agent starts
for the first time.

## Bootstrapping làm gì

Ở lần chạy tác tử đầu tiên, OpenClaw khởi tạo không gian làm việc (mặc định
`~/.openclaw/workspace`):

- Gieo mầm `AGENTS.md`, `BOOTSTRAP.md`, `IDENTITY.md`, `USER.md`.
- Chạy một nghi thức hỏi & đáp ngắn (mỗi lần một câu hỏi).
- Ghi danh tính + tùy chọn vào `IDENTITY.md`, `USER.md`, `SOUL.md`.
- Xóa `BOOTSTRAP.md` khi hoàn tất để đảm bảo chỉ chạy một lần.

## Nơi nó chạy

Khởi tạo ban đầu luôn chạy trên **gateway host**. Nếu ứng dụng macOS kết nối đến
a remote Gateway, the workspace and bootstrapping files live on that remote
machine.

<Note>
Khi Gateway chạy trên một máy khác, hãy chỉnh sửa các tệp không gian làm việc trên
máy chủ gateway (ví dụ: `user@gateway-host:~/.openclaw/workspace`).
</Note>

## Tài liệu liên quan

- Onboarding ứng dụng macOS: [Onboarding](/start/onboarding)
- Bố cục không gian làm việc: [Agent workspace](/concepts/agent-workspace)


