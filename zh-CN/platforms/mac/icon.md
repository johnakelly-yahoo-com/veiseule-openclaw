---
summary: "macOS 上 OpenClaw 菜单栏图标的状态和动画"
read_when:
  - 更改菜单栏图标行为
title: "菜单栏图标"
---

# 菜单栏图标状态

作者：steipete · 更新时间：2025-12-06 · 范围：macOS 应用（`apps/macos`）

- **空闲：** 正常图标动画（眨眼、偶尔摆动）。
- **暂停：** 状态项使用 `appearsDisabled`；无动画。
- **语音触发（大耳朵）：** 语音唤醒检测器在听到唤醒词时调用 `AppState.triggerVoiceEars(ttl: nil)`，在捕获语音期间保持 `earBoostActive=true`。耳朵放大（1.9 倍），显示圆形耳孔以提高可读性，然后在 1 秒静音后通过 `stopVoiceEars()` 恢复。仅由应用内语音管道触发。 Ears scale up (1.9x), get circular ear holes for readability, then drop via `stopVoiceEars()` after 1s of silence. Only fired from the in-app voice pipeline.
- **工作中（代理运行）：** `AppState.isWorking=true` 会驱动一个“尾巴/腿部疾走”的微动画：在任务进行中腿部摆动更快并略有偏移。 Currently toggled around WebChat agent runs; add the same toggle around other long tasks when you wire them.

Wiring points

- 语音唤醒：运行时/测试器在触发时调用 `AppState.triggerVoiceEars(ttl: nil)`，在 1 秒静音后调用 `stopVoiceEars()` 以匹配捕获窗口。
- 智能体活动：在工作区间前后设置 `AppStateStore.shared.setWorking(true/false)`（已在 WebChat 智能体调用中完成）。保持区间简短，并在 `defer` 块中重置以避免动画卡住。 Keep spans short and reset in `defer` blocks to avoid stuck animations.

形状与尺寸

- 基础图标在 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` 中绘制。
- 耳朵缩放默认为 `1.0`；语音增强时设置 `earScale=1.9` 并切换 `earHoles=true`，不改变整体框架（18×18 pt 模板图像渲染到 36×36 px Retina 后备存储）。
- 快速摆动使用最高约 1.0 的腿部摆幅并带有轻微的水平抖动；它与现有的空闲摆动叠加。

行为说明

- 耳朵/工作状态没有外部 CLI/代理切换；保持仅由应用自身信号控制，以避免意外的状态抖动。
- 保持 TTL 较短（&lt;10 秒），以便在任务挂起时图标能快速恢复到基准状态。
