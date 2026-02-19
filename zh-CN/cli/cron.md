---
summary: "`openclaw cron` 的 CLI 参考（调度和运行后台作业）"
read_when:
  - 你需要定时作业和唤醒功能
  - 你正在调试 cron 执行和日志
title: "cron"
---

# `openclaw cron`

管理 Gateway 网关调度器的 cron 作业。

相关内容：

- Cron 作业：[Cron 作业](/automation/cron-jobs)

提示：运行 `openclaw cron --help` 查看完整的命令集。

说明：隔离式 `cron add` 任务默认使用 `--announce` 投递摘要。使用 `--no-deliver` 仅内部运行。
`--deliver` 仍作为 `--announce` 的弃用别名保留。 Use `--no-deliver` to keep
output internal. `--deliver` remains as a deprecated alias for `--announce`.

说明：一次性（`--at`）任务成功后默认删除。使用 `--keep-after-run` 保留。 Use `--keep-after-run` to keep them.

Note: recurring jobs now use exponential retry backoff after consecutive errors (30s → 1m → 5m → 15m → 60m), then return to normal schedule after the next successful run.

## 常见编辑

更新投递设置而不更改消息：

```bash
openclaw cron edit <job-id> --announce --channel telegram --to "123456789"
```

为隔离的作业禁用投递：

```bash
openclaw cron edit <job-id> --no-deliver
```

Announce to a specific channel:

```bash
openclaw cron edit <job-id> --announce --channel slack --to "channel:C1234567890"
```

