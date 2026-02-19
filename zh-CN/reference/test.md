---
summary: "如何在本地运行测试（vitest）以及何时使用 force/coverage 模式"
read_when:
  - 运行或修复测试
title: "测试"
---

# 测试

- 完整测试套件（测试集、实时测试、Docker）：[测试](/help/testing)

- `pnpm test:force`：终止任何占用默认控制端口的遗留 Gateway 网关进程，然后使用隔离的 Gateway 网关端口运行完整的 Vitest 套件，这样服务器测试不会与正在运行的实例冲突。当之前的 Gateway 网关运行占用了端口 18789 时使用此命令。 14. 当先前的网关运行导致端口 18789 被占用时使用此命令。

- `pnpm test:coverage`：使用 V8 覆盖率运行单元测试套件（通过 `vitest.unit.config.ts`）。 16. 全局阈值为行/分支/函数/语句 70%。 17. 覆盖率排除了集成较重的入口点（CLI 接线、gateway/telegram 桥接、webchat 静态服务器），以便将目标集中在可进行单元测试的逻辑上。

- 在 Node 24+ 上运行 `pnpm test`：OpenClaw 会自动禁用 Vitest 的 `vmForks` 并改用 `forks`，以避免 `ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`。 你可以通过 `OPENCLAW_TEST_VM_FORKS=0|1` 强制指定该行为。

- `pnpm test:e2e`：运行 Gateway 网关端到端冒烟测试（多实例 WS/HTTP/节点配对）。 在 `vitest.e2e.config.ts` 中默认使用 `vmForks` + 自适应 workers；可通过 `OPENCLAW_E2E_WORKERS=<n>` 调整，并设置 `OPENCLAW_E2E_VERBOSE=1` 以启用详细日志。

- `pnpm test:live`：运行提供商实时测试（minimax/zai）。需要 API 密钥和 `LIVE=1`（或提供商特定的 `*_LIVE_TEST=1`）才能取消跳过。 20. 需要 API 密钥以及 `LIVE=1`（或特定提供方的 `*_LIVE_TEST=1`）才能取消跳过。

## 模型延迟基准测试（本地密钥）

脚本：[`scripts/bench-model.ts`](https://github.com/openclaw/openclaw/blob/main/scripts/bench-model.ts)

用法：

- `source ~/.profile && pnpm tsx scripts/bench-model.ts --runs 10`
- 可选环境变量：`MINIMAX_API_KEY`、`MINIMAX_BASE_URL`、`MINIMAX_MODEL`、`ANTHROPIC_API_KEY`
- 默认提示词："Reply with a single word: ok. No punctuation or extra text."

上次运行（2025-12-31，20 次）：

- minimax 中位数 1279ms（最小 1114，最大 2431）
- opus 中位数 2454ms（最小 1224，最大 3170）

## 新手引导 E2E（Docker）

Docker 是可选的；这仅用于容器化的新手引导冒烟测试。

在干净的 Linux 容器中完整的冷启动流程：

```bash
scripts/e2e/onboard-docker.sh
```

此脚本通过伪终端驱动交互式向导，验证配置/工作区/会话文件，然后启动 Gateway 网关并运行 `openclaw health`。

## QR 导入冒烟测试（Docker）

确保 `qrcode-terminal` 在 Docker 中的 Node 22+ 下加载：

```bash
pnpm test:docker:qr
```
