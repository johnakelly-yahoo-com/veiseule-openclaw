---
title: "Tài liệu tham chiếu Trình hướng dẫn Onboarding"
sidebarTitle: "Tham khảo Trình hướng dẫn"
---

# Tài liệu tham chiếu Trình hướng dẫn Onboarding

Đây là tài liệu tham chiếu đầy đủ cho trình hướng dẫn CLI `openclaw onboard`.
Để có cái nhìn tổng quan cấp cao, xem [Onboarding Wizard](/start/wizard).

## Chi tiết luồng (chế độ local)

<Steps>
  <Step title="Existing config detection">
    - Nếu `~/.openclaw/openclaw.json` tồn tại, chọn **Keep / Modify / Reset**.
    - Chạy lại trình hướng dẫn **không** xóa bất cứ thứ gì trừ khi bạn chủ động chọn **Reset**
      (hoặc truyền `--reset`).
    - Nếu cấu hình không hợp lệ hoặc chứa khóa cũ, trình hướng dẫn sẽ dừng và yêu cầu
      bạn chạy `openclaw doctor` trước khi tiếp tục.
    - Reset sử dụng `trash` (không bao giờ dùng `rm`) và cung cấp các phạm vi:
      - Chỉ config
      - Config + credentials + sessions
      - Reset toàn bộ (cũng xóa workspace)
  </Step>
  <Step title="Model/Auth">
    - **Anthropic API key (recommended)**: sử dụng `ANTHROPIC_API_KEY` nếu có hoặc yêu cầu nhập key, sau đó lưu để daemon sử dụng.
    - **Anthropic OAuth (Claude Code CLI)**: trên macOS, trình hướng dẫn kiểm tra mục Keychain "Claude Code-credentials" (chọn "Always Allow" để các lần khởi động qua launchd không bị chặn); trên Linux/Windows, tái sử dụng `~/.claude/.credentials.json` nếu có.
    - **Anthropic token (paste setup-token)**: chạy `claude setup-token` trên bất kỳ máy nào, sau đó dán token (bạn có thể đặt tên; để trống = mặc định).
    - **OpenAI Code (Codex) subscription (Codex CLI)**: nếu `~/.codex/auth.json` tồn tại, trình hướng dẫn có thể tái sử dụng.
    - **OpenAI Code (Codex) subscription (OAuth)**: luồng qua trình duyệt; dán `code#state`.
      - Đặt `agents.defaults.model` thành `openai-codex/gpt-5.2` khi model chưa được đặt hoặc là `openai/*`.
    - **OpenAI API key**: sử dụng `OPENAI_API_KEY` nếu có hoặc yêu cầu nhập key, sau đó lưu vào `~/.openclaw/.env` để launchd có thể đọc.
    - **xAI (Grok) API key**: yêu cầu `XAI_API_KEY` và cấu hình xAI làm model provider.
    - **OpenCode Zen (multi-model proxy)**: yêu cầu `OPENCODE_API_KEY` (hoặc `OPENCODE_ZEN_API_KEY`, lấy tại https://opencode.ai/auth).
    - **API key**: lưu key cho bạn.
    - **Vercel AI Gateway (multi-model proxy)**: yêu cầu `AI_GATEWAY_API_KEY`.
    - Chi tiết thêm: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: yêu cầu Account ID, Gateway ID và `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    - Chi tiết thêm: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: cấu hình được ghi tự động.
    - Chi tiết thêm: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: yêu cầu `SYNTHETIC_API_KEY`.
    - Chi tiết thêm: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: cấu hình được ghi tự động.
    - **Kimi Coding**: cấu hình được ghi tự động.
    - Chi tiết thêm: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: chưa cấu hình xác thực.
    - Chọn model mặc định từ các tùy chọn được phát hiện (hoặc nhập provider/model thủ công).
    - Trình hướng dẫn chạy kiểm tra model và cảnh báo nếu model đã cấu hình không xác định hoặc thiếu xác thực.
    - Thông tin OAuth nằm tại `~/.openclaw/credentials/oauth.json`; auth profiles nằm tại `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth).
    - Xem thêm chi tiết: [/concepts/oauth](/concepts/oauth)
    <Note>
    Mẹo cho headless/server: hoàn tất OAuth trên một máy có trình duyệt, sau đó sao chép
    `~/.openclaw/credentials/oauth.json` (hoặc `$OPENCLAW_STATE_DIR/credentials/oauth.json`) sang
    máy chủ gateway.
    </Note>
  </Step>
  <Step title="Workspace">
    - Mặc định `~/.openclaw/workspace` (có thể cấu hình).
    - Tạo các tệp workspace cần thiết cho nghi thức bootstrap của agent.
    - Bố cục workspace đầy đủ + hướng dẫn sao lưu: [Agent workspace](/concepts/agent-workspace)
  </Step>
  <Step title="Gateway">
    - Port, bind, chế độ xác thực, phơi bày qua tailscale.
    - Khuyến nghị xác thực: giữ **Token** ngay cả cho loopback để các client WS cục bộ vẫn phải xác thực.
    - Chỉ tắt xác thực nếu bạn hoàn toàn tin tưởng mọi tiến trình cục bộ.
    - Các bind không phải loopback vẫn yêu cầu xác thực.
  </Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): đăng nhập QR tùy chọn.
    - [Telegram](/channels/telegram): bot token.
    - [Discord](/channels/discord): bot token.
    - [Google Chat](/channels/googlechat): JSON tài khoản dịch vụ + webhook audience.
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL.
    - [Signal](/channels/signal): cài đặt `signal-cli` tùy chọn + cấu hình tài khoản.
    - [BlueBubbles](/channels/bluebubbles): **khuyến nghị cho iMessage**; server URL + password + webhook.
    - [iMessage](/channels/imessage): đường dẫn CLI `imsg` legacy + quyền truy cập DB.
    - Bảo mật DM: mặc định là ghép cặp. DM đầu tiên gửi một mã; phê duyệt qua `openclaw pairing approve <channel> <code>` hoặc dùng allowlists.
  </Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Yêu cầu có phiên người dùng đang đăng nhập; với môi trường headless, dùng LaunchDaemon tùy chỉnh (không kèm theo).
    - Linux (và Windows qua WSL2): systemd user unit
      - Trình hướng dẫn cố gắng bật lingering bằng `loginctl enable-linger <user>` để Gateway tiếp tục chạy sau khi đăng xuất.
      - Có thể yêu cầu sudo (ghi vào `/var/lib/systemd/linger`); sẽ thử không dùng sudo trước.
    - **Lựa chọn runtime:** Node (khuyến nghị; bắt buộc cho WhatsApp/Telegram). Bun **không được khuyến nghị**.
  </Step>
  <Step title="Health check">
    - Khởi động Gateway (nếu cần) và chạy `openclaw health`.
    - Mẹo: `openclaw status --deep` bổ sung các kiểm tra tình trạng gateway vào kết quả trạng thái (yêu cầu gateway có thể truy cập được).
  </Step>
  <Step title="Skills (recommended)">
    - Đọc các skills hiện có và kiểm tra các yêu cầu.
    - Cho phép bạn chọn trình quản lý node: **npm / pnpm** (bun không được khuyến nghị).
    - Cài đặt các phụ thuộc tùy chọn (một số sử dụng Homebrew trên macOS).
  </Step>
  <Step title="Finish">
    - Tóm tắt + các bước tiếp theo, bao gồm ứng dụng iOS/Android/macOS để có thêm tính năng.
  </Step>
</Steps>

<Note>
Nếu không phát hiện GUI, trình hướng dẫn sẽ in hướng dẫn chuyển tiếp cổng SSH cho Control UI thay vì mở trình duyệt.
Nếu thiếu tài nguyên Control UI, trình hướng dẫn sẽ cố gắng build chúng; phương án dự phòng là `pnpm ui:build` (tự động cài đặt các phụ thuộc UI).
</Note>

## Chế độ không tương tác

Dùng `--non-interactive` để tự động hóa hoặc viết script cho onboarding:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Thêm `--json` để có bản tóm tắt dạng máy đọc được.

<Note>
`--json` **không** tự động bật chế độ không tương tác. Dùng `--non-interactive` (và `--workspace`) cho các script.
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  </Accordion>
</AccordionGroup>

### Thêm agent (không tương tác)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## RPC trình hướng dẫn Gateway

Gateway cung cấp luồng trình hướng dẫn qua RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Các client (ứng dụng macOS, Control UI) có thể render các bước mà không cần tái triển khai logic onboarding.

## Thiết lập Signal (signal-cli)

Trình hướng dẫn có thể cài đặt `signal-cli` từ GitHub releases:

- Tải xuống asset phát hành phù hợp.
- Lưu trữ dưới `~/.openclaw/tools/signal-cli/<version>/`.
- Ghi `channels.signal.cliPath` vào cấu hình của bạn.

Ghi chú:

- Bản dựng JVM yêu cầu **Java 21**.
- Bản dựng native được dùng khi có sẵn.
- Windows dùng WSL2; cài đặt signal-cli theo luồng Linux bên trong WSL.

## Những gì trình hướng dẫn ghi

Các trường điển hình trong `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (nếu chọn Minimax)
- `gateway.*` (chế độ, bind, xác thực, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Allowlists kênh (Slack/Discord/Matrix/Microsoft Teams) khi bạn chọn trong các lời nhắc (tên sẽ được phân giải thành ID khi có thể).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` ghi `agents.list[]` và các `bindings` tùy chọn.

Thông tin xác thực WhatsApp nằm tại `~/.openclaw/credentials/whatsapp/<accountId>/`.
Sessions được lưu tại `~/.openclaw/agents/<agentId>/sessions/`.

Một số channel được cung cấp dưới dạng plugin. Khi bạn chọn trong quá trình onboarding, trình hướng dẫn
sẽ yêu cầu cài đặt (npm hoặc đường dẫn cục bộ) trước khi có thể cấu hình.

## Tài liệu liên quan

- Tổng quan trình hướng dẫn: [Onboarding Wizard](/start/wizard)
- Onboarding ứng dụng macOS: [Onboarding](/start/onboarding)
- Tài liệu tham chiếu cấu hình: [Gateway configuration](/gateway/configuration)
- Nhà cung cấp: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)
