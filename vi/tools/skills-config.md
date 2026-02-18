---
title: "Cấu hình Skills"
---

# Cấu hình Skills

Tất cả cấu hình liên quan đến skills nằm dưới `skills` trong `~/.openclaw/openclaw.json`.

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: "GEMINI_KEY_HERE",
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## Trường

- 50. `allowBundled`: danh sách cho phép tùy chọn chỉ dành cho các kỹ năng **được đóng gói**. When set, only
các kỹ năng đi kèm trong danh sách đủ điều kiện (không ảnh hưởng đến kỹ năng managed/workspace).
- `load.extraDirs`: các thư mục skill bổ sung để quét (độ ưu tiên thấp nhất).
- `load.watch`: theo dõi các thư mục skill và làm mới ảnh chụp skills (mặc định: true).
- `load.watchDebounceMs`: debounce cho các sự kiện watcher của skill tính bằng mili giây (mặc định: 250).
- `install.preferBrew`: ưu tiên trình cài đặt brew khi có sẵn (mặc định: true).
- `install.nodeManager`: tùy chọn trình cài đặt node (`npm` | `pnpm` | `yarn` | `bun`, mặc định: npm).
Điều này chỉ ảnh hưởng đến **việc cài đặt kỹ năng**; môi trường chạy Gateway vẫn phải là Node
  (Bun not recommended for WhatsApp/Telegram).
- `entries.<skillKey>`: ghi đè theo từng kỹ năng.

Các trường theo từng skill:

- `enabled`: đặt `false` để vô hiệu hóa một skill ngay cả khi nó được gói sẵn/cài đặt.
- `env`: biến môi trường được chèn cho lần chạy tác tử (chỉ khi chưa được đặt).
- `apiKey`: tiện ích tùy chọn cho các skills khai báo một biến môi trường chính.

## Ghi chú

- Các khóa dưới `entries` mặc định ánh xạ tới tên kỹ năng. Nếu một kỹ năng định nghĩa
`metadata.openclaw.skillKey`, hãy sử dụng khóa đó thay thế.
- Các thay đổi đối với skills sẽ được áp dụng ở lượt tác tử tiếp theo khi watcher được bật.

### Skills sandboxed + biến môi trường

1. Khi một phiên làm việc được **sandboxed**, các tiến trình kỹ năng sẽ chạy bên trong Docker. 2. Sandbox **không** kế thừa `process.env` của máy chủ.

Hãy dùng một trong các cách sau:

- `agents.defaults.sandbox.docker.env` (hoặc `agents.list[].sandbox.docker.env` theo từng tác tử)
- bake biến môi trường vào image sandbox tùy chỉnh của bạn

3. `env` toàn cục và `skills.entries.<skill>`
4. `.env/apiKey` chỉ áp dụng cho các lần chạy trên **host**.5. OpenClaw sử dụng các thư mục kỹ năng **tương thích [AgentSkills](https://agentskills.io)** để dạy agent cách sử dụng công cụ.

