---
summary: "上下文窗口 + 压缩：OpenClaw 如何将会话保持在模型限制内"
read_when:
  - 你想了解自动压缩和 /compact
  - 你正在调试长会话触及上下文限制的问题
title: "Compaction"
---

# 上下文窗口与压缩

Every model has a **context window** (max tokens it can see). Long-running chats accumulate messages and tool results; once the window is tight, OpenClaw **compacts** older history to stay within limits.

## 什么是压缩

Compaction **summarizes older conversation** into a compact summary entry and keeps recent messages intact. The summary is stored in the session history, so future requests use:

- 压缩摘要
- 压缩点之后的近期消息

压缩会**持久化**到会话的 JSONL 历史记录中。

## 配置

在你的 `openclaw.json` 中使用 `agents.defaults.compaction` 设置来配置压缩行为（模式、目标 token 数等）。

## 自动压缩（默认开启）

当会话接近或超过模型的上下文窗口时，OpenClaw 会触发自动压缩，并可能使用压缩后的上下文重试原始请求。

你会看到：

- 详细模式下显示 `🧹 Auto-compaction complete`
- `/status` 显示 `🧹 Compactions: <count>`

在压缩之前，OpenClaw 可以运行一次**静默记忆刷写**轮次，将持久化笔记写入磁盘。详情及配置请参阅[记忆](/concepts/memory)。 9. 详情和配置请参阅 [Memory](/concepts/memory)。

## 手动压缩

使用 `/compact`（可选附带指令）强制执行一次压缩：

```
/compact Focus on decisions and open questions
```

## 上下文窗口来源

14. 上下文窗口是模型特定的。 上下文窗口因模型而异。OpenClaw 使用已配置提供商目录中的模型定义来确定限制。

## 16. 压缩 vs 修剪

- **压缩**：总结并**持久化**到 JSONL 中。
- **会话修剪**：仅裁剪旧的**工具结果**，**在内存中**按请求进行。

19. 修剪详情请参阅 [/concepts/session-pruning](/concepts/session-pruning)。

## 提示

- 当会话感觉过时或上下文臃肿时，使用 `/compact`。
- 大型工具输出已被截断；修剪可以进一步减少工具结果的堆积。
- 如果你需要全新开始，`/new` 或 `/reset` 会启动一个新的会话 ID。

