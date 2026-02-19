---
summary: "All configuration options for ~/.openclaw/openclaw.json with examples"
read_when:
  - Adding or modifying config fields
  - Tìm các mẫu cấu hình phổ biến
  - Điều hướng đến các phần cấu hình cụ thể
title: "Cấu hình"
---

# Configuration 🔧

OpenClaw reads an optional **JSON5** config from `~/.openclaw/openclaw.json` (comments + trailing commas allowed).

Nếu tệp bị thiếu, OpenClaw sẽ sử dụng các giá trị mặc định an toàn. Các lý do phổ biến để thêm cấu hình:

- restrict who can trigger the bot (`channels.whatsapp.allowFrom`, `channels.telegram.allowFrom`, etc.)
- control group allowlists + mention behavior (`channels.whatsapp.groups`, `channels.telegram.groups`, `channels.discord.guilds`, `agents.list[].groupChat`)
- customize message prefixes (`messages`)

Xem [tài liệu tham chiếu đầy đủ](/gateway/configuration-reference) để biết tất cả các trường khả dụng.

<Tip>
**Bạn mới làm quen với cấu hình?** Hãy bắt đầu với `openclaw onboard` để thiết lập tương tác, hoặc xem hướng dẫn [Configuration Examples](/gateway/configuration-examples) để có các cấu hình hoàn chỉnh có thể sao chép và dán.
</Tip>

## Cấu hình tối thiểu

```json5
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Chỉnh sửa cấu hình

<Tabs>
  <Tab title="Interactive wizard">
    ```bash
    openclaw onboard       # trình hướng dẫn thiết lập đầy đủ
    openclaw configure     # trình hướng dẫn cấu hình
    ```
  
</Tab>
  <Tab title="CLI (one-liners)">
    ```bash
    openclaw config get agents.defaults.workspace
    openclaw config set agents.defaults.heartbeat.every "2h"
    openclaw config unset tools.web.search.apiKey
    ```
  
</Tab>
  <Tab title="Control UI">
    Mở [http://127.0.0.1:18789](http://127.0.0.1:18789) và sử dụng tab **Config**.
    Control UI hiển thị một biểu mẫu dựa trên schema cấu hình, với trình chỉnh sửa **Raw JSON** để can thiệp trực tiếp khi cần.
  
</Tab>
  <Tab title="Direct edit">
    Chỉnh sửa trực tiếp `~/.openclaw/openclaw.json`. Gateway theo dõi tệp và tự động áp dụng các thay đổi (xem [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Schema + UI hints

<Warning>
OpenClaw only accepts configurations that fully match the schema. Các khóa không xác định, sai kiểu dữ liệu hoặc giá trị không hợp lệ sẽ khiến Gateway **từ chối khởi động**. Ngoại lệ duy nhất ở cấp gốc là `$schema` (chuỗi), để các trình soạn thảo có thể đính kèm metadata JSON Schema.
</Warning>

When validation fails:

- Gateway không khởi động
- Chỉ các lệnh chẩn đoán hoạt động (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
- Chạy `openclaw doctor` để xem chính xác các vấn đề
- Chạy `openclaw doctor --fix` (hoặc `--yes`) để áp dụng sửa lỗi

## Apply + restart (RPC)

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">
    Mỗi kênh có phần cấu hình riêng dưới `channels.<provider>`. Xem trang kênh tương ứng để biết các bước thiết lập:

    ````
    - [WhatsApp](/channels/whatsapp) — `channels.whatsapp`
    - [Telegram](/channels/telegram) — `channels.telegram`
    - [Discord](/channels/discord) — `channels.discord`
    - [Slack](/channels/slack) — `channels.slack`
    - [Signal](/channels/signal) — `channels.signal`
    - [iMessage](/channels/imessage) — `channels.imessage`
    - [Google Chat](/channels/googlechat) — `channels.googlechat`
    - [Mattermost](/channels/mattermost) — `channels.mattermost`
    - [MS Teams](/channels/msteams) — `channels.msteams`
    
    Tất cả các kênh đều chia sẻ cùng một mẫu chính sách DM:
    
    ```json5
    {
      channels: {
        telegram: {
          enabled: true,
          botToken: "123:abc",
          dmPolicy: "pairing",   // pairing | allowlist | open | disabled
          allowFrom: ["tg:123"], // only for allowlist/open
        },
      },
    }
    ```
    ````

  
</Accordion>

  <Accordion title="Choose and configure models">
    Thiết lập model chính và các model dự phòng (nếu có):

    ````
    ```json5
    {
      agents: {
        defaults: {
          model: {
            primary: "anthropic/claude-sonnet-4-5",
            fallbacks: ["openai/gpt-5.2"],
          },
          models: {
            "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
            "openai/gpt-5.2": { alias: "GPT" },
          },
        },
      },
    }
    ```
    
    - `agents.defaults.models` xác định danh mục model và đóng vai trò là allowlist cho `/model`.
    - Tham chiếu model sử dụng định dạng `provider/model` (ví dụ: `anthropic/claude-opus-4-6`).
    - Xem [Models CLI](/concepts/models) để chuyển model trong chat và [Model Failover](/concepts/model-failover) để biết về luân phiên xác thực và hành vi dự phòng.
    - Đối với provider tùy chỉnh/tự lưu trữ, xem [Custom providers](/gateway/configuration-reference#custom-providers-and-base-urls) trong tài liệu tham chiếu.
    ````

  
</Accordion>

  <Accordion title="Control who can message the bot">
    Quyền truy cập DM được kiểm soát theo từng kênh qua `dmPolicy`:

    ```
    - `"pairing"` (mặc định): người gửi chưa xác định sẽ nhận mã ghép đôi dùng một lần để phê duyệt
    - `"allowlist"`: chỉ những người gửi trong `allowFrom` (hoặc trong danh sách đã ghép đôi)
    - `"open"`: cho phép tất cả DM đến (yêu cầu `allowFrom: ["*"]`)
    - `"disabled"`: bỏ qua tất cả DM
    
    Đối với nhóm, sử dụng `groupPolicy` + `groupAllowFrom` hoặc allowlist riêng của từng kênh.
    
    Xem [tài liệu tham chiếu đầy đủ](/gateway/configuration-reference#dm-and-group-access) để biết chi tiết theo từng kênh.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Tin nhắn nhóm mặc định là **yêu cầu đề cập (mention)**. Cấu hình các mẫu cho từng agent:

    ````
    ```json5
    {
      agents: {
        list: [
          {
            id: "main",
            groupChat: {
              mentionPatterns: ["@openclaw", "openclaw"],
            },
          },
        ],
      },
      channels: {
        whatsapp: {
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```
    
    - **Đề cập qua metadata**: @-mention gốc của nền tảng (WhatsApp chạm để mention, Telegram @bot, v.v.)
    - **Mẫu văn bản**: các biểu thức regex trong `mentionPatterns`
    - Xem [tài liệu tham chiếu đầy đủ](/gateway/configuration-reference#group-chat-mention-gating) để biết ghi đè theo từng kênh và chế độ self-chat.
    ````

  
</Accordion>

  <Accordion title="Configure sessions and resets">
    Sessions kiểm soát tính liên tục và sự tách biệt của cuộc trò chuyện:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // khuyến nghị cho môi trường nhiều người dùng
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (chia sẻ) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Xem [Session Management](/concepts/session) để biết về phạm vi, liên kết định danh và chính sách gửi.
    - Xem [tài liệu tham khảo đầy đủ](/gateway/configuration-reference#session) để biết tất cả các trường.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">    Chạy các phiên agent trong các container Docker cách ly:

    ```
    scripts/sandbox-setup.sh
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">    ```json5
    {
      agents: {
        defaults: {
          heartbeat: {
            every: "30m",
            target: "last",
          },
        },
      },
    }
    ```

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">    ```json5
    {
      cron: {
        enabled: true,
        maxConcurrentRuns: 2,
        sessionRetention: "24h",
      },
    }
    ```

    ```
    **Gateway**: một tiến trình gateway chạy dài hạn duy nhất, sở hữu trạng thái (phiên, ghép cặp, sổ đăng ký node) và chạy các kênh.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">    Bật các endpoint webhook HTTP trên Gateway:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">    Chạy nhiều agent cách ly với workspace và phiên riêng biệt:

    ```
    // Sibling keys override included values
    {
      $include: "./base.json5", // { a: 1, b: 2 }
      b: 99, // Result: { a: 1, b: 99 }
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">    Sử dụng `$include` để tổ chức các cấu hình lớn:

    ```
    // clients/mueller.json5
    {
      agents: { $include: "./mueller/agents.json5" },
      broadcast: { $include: "./mueller/broadcast.json5" },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Tải lại cấu hình nóng

Gateway theo dõi `~/.openclaw/openclaw.json` và tự động áp dụng thay đổi — không cần khởi động lại thủ công cho hầu hết các thiết lập.

### Error handling

| Chế độ                                     | Hành vi                                                                                                                           |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (mặc định) | Áp dụng ngay lập tức các thay đổi an toàn. Tự động khởi động lại đối với các thay đổi quan trọng. |
| **`hot`**                                  | Chỉ áp dụng nóng các thay đổi an toàn. Ghi log cảnh báo khi cần khởi động lại — bạn tự xử lý.     |
| **`restart`**                              | Khởi động lại Gateway khi có bất kỳ thay đổi cấu hình nào, dù an toàn hay không.                                  |
| **`off`**                                  | Tắt theo dõi tệp. Các thay đổi sẽ có hiệu lực ở lần khởi động lại thủ công tiếp theo.             |

```json5
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### Những gì được áp dụng nóng và những gì cần khởi động lại

Hầu hết các trường được áp dụng nóng mà không gây gián đoạn. Trong chế độ `hybrid`, các thay đổi yêu cầu khởi động lại sẽ được xử lý tự động.

| Danh mục                                  | Trường                                                                                           | Cần khởi động lại? |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------ |
| Kênh                                      | `channels.*`, `web` (WhatsApp) — tất cả các kênh tích hợp sẵn và kênh mở rộng | Không              |
| Agent & mô hình       | `agent`, `agents`, `models`, `routing`                                                           | Không              |
| Tự động hóa                               | `hooks`, `cron`, `agent.heartbeat`                                                               | Không              |
| Phiên & tin nhắn      | `session`, `messages`                                                                            | Không              |
| Công cụ & phương tiện | `tools`, `browser`, `skills`, `audio`, `talk`                                                    | Không              |
| UI & linh tinh        | `ui`, `logging`, `identity`, `bindings`                                                          | Không              |
| Máy chủ Gateway                           | `gateway.*` (port, bind, auth, tailscale, TLS, HTTP)                          | **Có**             |
| Hạ tầng                                   | `discovery`, `canvasHost`, `plugins`                                                             | **Có**             |

<Note>
`gateway.reload` và `gateway.remote` là các ngoại lệ — thay đổi chúng **không** kích hoạt khởi động lại.
</Note>

## Env vars + `.env`

<AccordionGroup>
  <Accordion title="config.apply (full replace)">
    Xác thực + ghi toàn bộ cấu hình và khởi động lại Gateway trong một bước.


    ````
    <Warning>
    `config.apply` thay thế **toàn bộ config**. Sử dụng `config.patch` để cập nhật một phần, hoặc `openclaw config set` cho từng khóa riêng lẻ.
    
</Warning>
    
    Params:
    
    - `raw` (string) — payload JSON5 cho toàn bộ config
    - `baseHash` (tùy chọn) — hash config từ `config.get` (bắt buộc khi config đã tồn tại)
    - `sessionKey` (tùy chọn) — khóa phiên cho ping đánh thức sau khi khởi động lại
    - `note` (tùy chọn) — ghi chú cho restart sentinel
    - `restartDelayMs` (tùy chọn) — độ trễ trước khi khởi động lại (mặc định 2000)
    
    ```bash
    openclaw gateway call config.get --params '{}'  # lấy payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
      "baseHash": "<hash>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123"
    }'
    ```
    ````

  
</Accordion>

  <Accordion title="config.patch (partial update)">
    Hợp nhất bản cập nhật một phần vào config hiện có (theo ngữ nghĩa JSON merge patch):


    ````
    - Các object được hợp nhất đệ quy
    - `null` xóa một khóa
    - Các mảng sẽ bị thay thế
    
    Params:
    
    - `raw` (string) — JSON5 chỉ chứa các khóa cần thay đổi
    - `baseHash` (bắt buộc) — hash config từ `config.get`
    - `sessionKey`, `note`, `restartDelayMs` — giống như `config.apply`
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Biến môi trường

OpenClaw đọc các biến môi trường từ tiến trình cha và thêm từ:

- `.env` from the current working directory (if present)
- `~/.openclaw/.env` (mặc định toàn cục)

Cả hai tệp đều không ghi đè các biến môi trường đã tồn tại. Bạn cũng có thể đặt biến môi trường nội tuyến trong config:

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

<Accordion title="Shell env import (optional)">
  Nếu được bật và các khóa dự kiến chưa được thiết lập, OpenClaw sẽ chạy login shell của bạn và chỉ nhập các khóa còn thiếu:


```json5
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

Biến môi trường tương đương: `OPENCLAW_LOAD_SHELL_ENV=1` 
</Accordion>

<Accordion title="Env var substitution in config values">
  Tham chiếu biến môi trường trong bất kỳ giá trị chuỗi nào của config bằng `${VAR_NAME}`:

```json5
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

Quy tắc:

- Chỉ khớp các tên viết hoa: `[A-Z_][A-Z0-9_]*`
- Biến thiếu/rỗng sẽ gây lỗi khi tải
- Thoát bằng `$${VAR}` để xuất ra giá trị nguyên văn
- Hoạt động bên trong các tệp `$include`
- Thay thế nội tuyến: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Xem [Environment](/help/environment) để biết đầy đủ về thứ tự ưu tiên và các nguồn.

## Tham chiếu đầy đủ

Để xem tài liệu tham chiếu đầy đủ theo từng trường, xem **[Configuration Reference](/gateway/configuration-reference)**.

---

Legacy OAuth imports:
