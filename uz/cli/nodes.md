---
title: "`openclaw nodes`"
---

# Juftlangan tugunlarni (qurilmalarni) boshqarish va tugun imkoniyatlarini chaqirish.

Bog‘liq:

Tugunlar sharhi: [Nodes](/nodes)

- Tugunlar haqida umumiy ma’lumot: [Nodes](/nodes)
- Kamera: [Camera nodes](/nodes/camera)
- Tasvirlar: [Image nodes](/nodes/images)

Umumiy sozlamalar:

- `--url`, `--token`, `--timeout`, `--json`

## Umumiy buyruqlar

```bash
openclaw nodes list
openclaw nodes list --connected
openclaw nodes list --last-connected 24h
openclaw nodes pending
openclaw nodes approve <requestId>
openclaw nodes status
openclaw nodes status --connected
openclaw nodes status --last-connected 24h
```

`nodes list` kutilayotgan/juftlangan jadvallarni chiqaradi. Juftlangan qatorlar eng so‘nggi ulanish vaqtini (Last Connect) o‘z ichiga oladi.
Use `--connected` to only show currently-connected nodes. Use `--last-connected <duration>` to
filter to nodes that connected within a duration (e.g. `24h`, `7d`).

## Chaqirish / ishga tushirish

```bash
openclaw nodes invoke --node <id|name|ip> --command <command> --params <json>
openclaw nodes run --node <id|name|ip> <command...>
openclaw nodes run --raw "git status"
openclaw nodes run --agent main --node <id|name|ip> --raw "git status"
```

Chaqirish bayroqlari:

- `--params <json>`: JSON obyekt satri (standart `{}`).
- `--invoke-timeout <ms>`: node invoke timeout (default `15000`).
- `--idempotency-key <key>`: optional idempotency key.

### Exec-style defaults

`nodes run` mirrors the model’s exec behavior (defaults + approvals):

- Reads `tools.exec.*` (plus `agents.list[].tools.exec.*` overrides).
- Uses exec approvals (`exec.approval.request`) before invoking `system.run`.
- `--node` can be omitted when `tools.exec.node` is set.
- Requires a node that advertises `system.run` (macOS companion app or headless node host).

Flags:

- `--cwd <path>`: working directory.
- `--env <key=val>`: env override (repeatable).
- `--command-timeout <ms>`: command timeout.
- `--invoke-timeout <ms>`: node invoke timeout (default `30000`).
- `--needs-screen-recording`: require screen recording permission.
- `--raw <command>`: run a shell string (`/bin/sh -lc` or `cmd.exe /c`).
- `--agent <id>`: agent-scoped approvals/allowlists (defaults to configured agent).
- `--ask <off|on-miss|always>`, `--security <deny|allowlist|full>`: overrides.

