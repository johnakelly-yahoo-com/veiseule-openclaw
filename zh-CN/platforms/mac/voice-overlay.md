---
summary: "唤醒词与按键说话重叠时的语音浮层生命周期"
read_when:
  - 调整语音浮层行为
title: "语音浮层"
---

# 语音浮层生命周期（macOS）

Audience: macOS app contributors. Goal: keep the voice overlay predictable when wake-word and push-to-talk overlap.

## 当前意图

- If the overlay is already visible from wake-word and the user presses the hotkey, the hotkey session _adopts_ the existing text instead of resetting it. The overlay stays up while the hotkey is held. When the user releases: send if there is trimmed text, otherwise dismiss.
- 单独使用唤醒词时仍在静音后自动发送；按键说话在松开时立即发送。

## Implemented (Dec 9, 2025)

- Overlay sessions now carry a token per capture (wake-word or push-to-talk). Partial/final/send/dismiss/level updates are dropped when the token doesn’t match, avoiding stale callbacks.
- 按键说话会接管任何可见的浮层文本作为前缀（因此在唤醒浮层显示时按下热键会保留文本并追加新语音）。它最多等待 1.5 秒获取最终转录结果，然后回退到当前文本。 It waits up to 1.5s for a final transcript before falling back to the current text.
- 提示音/浮层日志以 `info` 级别输出，分类为 `voicewake.overlay`、`voicewake.ptt` 和 `voicewake.chime`（会话开始、部分、最终、发送、关闭、提示音原因）。

## 后续步骤

1. **VoiceSessionCoordinator（actor）**
   - 同一时间只拥有一个 `VoiceSession`。
   - API（基于令牌）：`beginWakeCapture`、`beginPushToTalk`、`updatePartial`、`endCapture`、`cancel`、`applyCooldown`。
   - Drops callbacks that carry stale tokens (prevents old recognizers from reopening the overlay).
2. **VoiceSession（模型）**
   - 字段：`token`、`source`（wakeWord|pushToTalk）、已提交/临时文本、提示音标志、计时器（自动发送、空闲）、`overlayMode`（display|editing|sending）、冷却截止时间。
3. **浮层绑定**
   - `VoiceSessionPublisher`（`ObservableObject`）将活跃会话镜像到 SwiftUI。
   - `VoiceWakeOverlayView` 仅通过 publisher 渲染；绝不直接修改全局单例。
   - 浮层用户操作（`sendNow`、`dismiss`、`edit`）携带会话令牌回调到 coordinator。
4. **统一发送路径**
   - `endCapture` 时：如果去除空白后文本为空 → 关闭；否则 `performSend(session:)`（播放一次发送提示音、转发、关闭）。
   - 按键说话：无延迟；唤醒词：可选自动发送延迟。
   - 按键说话结束后对唤醒运行时施加短暂冷却，防止唤醒词立即重新触发。
5. **日志**
   - Coordinator 在子系统 `bot.molt`、分类 `voicewake.overlay` 和 `voicewake.chime` 下输出 `.info` 级别日志。
   - 关键事件：`session_started`、`adopted_by_push_to_talk`、`partial`、`finalized`、`send`、`dismiss`、`cancel`、`cooldown`。

## 调试清单

- 复现浮层粘滞问题时流式查看日志：

  ```bash
  sudo log stream --predicate 'subsystem == "bot.molt" AND category CONTAINS "voicewake"' --level info --style compact
  ```

- 验证只有一个活跃会话令牌；过时回调应被 coordinator 丢弃。

- 确保按键说话松开时始终使用活跃令牌调用 `endCapture`；如果文本为空，预期 `dismiss` 且不播放提示音或发送。

## 迁移步骤（建议）

1. 添加 `VoiceSessionCoordinator`、`VoiceSession` 和 `VoiceSessionPublisher`。
2. 重构 `VoiceWakeRuntime`，使其创建/更新/结束会话，而非直接操作 `VoiceWakeOverlayController`。
3. 重构 `VoicePushToTalk`，使其接管现有会话并在松开时调用 `endCapture`；施加运行时冷却。
4. 将 `VoiceWakeOverlayController` 连接到 publisher；移除来自 runtime/PTT 的直接调用。
5. 添加会话接管、冷却和空文本关闭的集成测试。
