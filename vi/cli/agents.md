---
title: "agents"
---

# `openclaw agents`

Quản lý các tác tử cô lập (không gian làm việc + xác thực + định tuyến).

Liên quan:

- Định tuyến đa tác tử: [Multi-Agent Routing](/concepts/multi-agent)
- Không gian làm việc của tác tử: [Agent workspace](/concepts/agent-workspace)

## Ví dụ

```bash
openclaw agents list
openclaw agents add work --workspace ~/.openclaw/workspace-work
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
openclaw agents set-identity --agent main --avatar avatars/openclaw.png
openclaw agents delete work
```

## Tệp danh tính

Mỗi không gian làm việc của tác tử có thể bao gồm một `IDENTITY.md` tại thư mục gốc của workspace:

- Đường dẫn ví dụ: `~/.openclaw/workspace/IDENTITY.md`
- `set-identity --from-identity` đọc từ thư mục gốc của workspace (hoặc một `--identity-file` được chỉ định rõ)

Đường dẫn avatar được phân giải tương đối so với thư mục gốc của workspace.

## Đặt danh tính

`set-identity` ghi các trường vào `agents.list[].identity`:

- `name`
- `theme`
- `emoji`
- `avatar` (đường dẫn tương đối theo workspace, URL http(s), hoặc data URI)

Tải từ `IDENTITY.md`:

```bash
openclaw agents set-identity --workspace ~/.openclaw/workspace --from-identity
```

Ghi đè các trường một cách tường minh:

```bash
openclaw agents set-identity --agent main --name "OpenClaw" --emoji "🦞" --avatar avatars/openclaw.png
```

Mẫu cấu hình:

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "OpenClaw",
          theme: "space lobster",
          emoji: "🦞",
          avatar: "avatars/openclaw.png",
        },
      },
    ],
  },
}
```
