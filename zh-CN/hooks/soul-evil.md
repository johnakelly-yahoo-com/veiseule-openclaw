---
title: "SOUL 恶意钩子"
---

# SOUL 恶意钩子

SOUL 恶意钩子会在执行期间将**注入的** `SOUL.md` 内容替换为 `SOUL_EVIL.md`
a purge window or by random chance. It does **not** modify files on disk.

## 工作原理

When `agent:bootstrap` runs, the hook can replace the `SOUL.md` content in memory
before the system prompt is assembled. 23. 如果 `SOUL_EVIL.md` 缺失或为空，
OpenClaw 会记录一条警告并继续使用正常的 `SOUL.md`。

子代理运行时**不会**在其引导文件中包含 `SOUL.md`，因此该钩子
has no effect on sub-agents.

## 启用

```bash
openclaw hooks enable soul-evil
```

然后设置配置：

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

在代理工作区根目录（与 `SOUL.md` 同级）创建 `SOUL_EVIL.md`。

## 选项

- `file` (string): alternate SOUL filename (default: `SOUL_EVIL.md`)
- `chance` (number 0–1): random chance per run to use `SOUL_EVIL.md`
- `purge.at` (HH:mm): daily purge start (24-hour clock)
- `purge.duration` (duration): window length (e.g. `30s`, `10m`, `1h`)

**Precedence:** purge window wins over chance.

**Timezone:** uses `agents.defaults.userTimezone` when set; otherwise host timezone.

## Notes

- No files are written or modified on disk.
- If `SOUL.md` is not in the bootstrap list, the hook does nothing.

## See Also

- [Hooks](/automation/hooks)


