---
summary: "Phát sóng một tin nhắn WhatsApp tới nhiều tác tử"
read_when:
  - Cấu hình broadcast groups
  - Gỡ lỗi phản hồi đa tác tử trong WhatsApp
status: experimental
title: "Nhóm Phát Sóng"
---

# Nhóm Phát Sóng

**Trạng thái:** Thử nghiệm  
**Phiên bản:** Được thêm trong 2026.1.9

## Tổng quan

Nhóm Phát Sóng cho phép nhiều tác nhân xử lý và phản hồi cùng một tin nhắn đồng thời. Điều này giúp bạn tạo các nhóm tác nhân chuyên biệt làm việc cùng nhau trong một nhóm WhatsApp hoặc DM — tất cả chỉ với một số điện thoại.

Phạm vi hiện tại: **Chỉ WhatsApp** (kênh web).

Nhóm phát sóng được đánh giá sau danh sách cho phép kênh (channel allowlists) và các quy tắc kích hoạt nhóm. Trong các nhóm WhatsApp, điều này có nghĩa là phát sóng sẽ diễn ra khi OpenClaw thường phản hồi (ví dụ: khi được nhắc đến, tùy thuộc vào cài đặt nhóm của bạn).

## Trường hợp sử dụng

### 1. Nhóm Tác Nhân Chuyên Biệt

Triển khai nhiều tác tử với trách nhiệm nguyên tử, tập trung:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Mỗi tác tử xử lý cùng một tin nhắn và cung cấp góc nhìn chuyên môn riêng của mình.

### 2. Hỗ Trợ Đa Ngôn Ngữ

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Quy Trình Đảm Bảo Chất Lượng

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Tự Động Hóa Tác Vụ

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Cấu hình

### Thiết lập cơ bản

37. Thêm một mục `broadcast` ở cấp cao nhất (bên cạnh `bindings`). Keys are WhatsApp peer ids:

- chat nhóm: JID của nhóm (ví dụ: `120363403215116621@g.us`)
- DM: số điện thoại chuẩn E.164 (ví dụ: `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Kết quả:** Khi OpenClaw sẽ phản hồi trong cuộc trò chuyện này, nó sẽ chạy cả ba tác tử.

### Chiến lược xử lý

Kiểm soát cách các tác tử xử lý tin nhắn:

#### Song song (Mặc định)

Tất cả các tác tử xử lý đồng thời:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Tuần tự

Các tác tử xử lý theo thứ tự (tác tử sau chờ tác tử trước hoàn thành):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Ví dụ hoàn chỉnh

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Cách hoạt động

### Luồng tin nhắn

1. **Tin nhắn đến** xuất hiện trong một nhóm WhatsApp
2. **Kiểm tra broadcast**: Hệ thống kiểm tra xem peer ID có nằm trong `broadcast` hay không
3. **Nếu nằm trong danh sách broadcast**:
   - Tất cả các tác tử được liệt kê đều xử lý tin nhắn
   - Mỗi tác tử có khóa phiên riêng và ngữ cảnh tách biệt
   - Các tác tử xử lý song song (mặc định) hoặc tuần tự
4. **Nếu không nằm trong danh sách broadcast**:
   - Áp dụng định tuyến thông thường (binding khớp đầu tiên)

Lưu ý: nhóm phát sóng không bỏ qua danh sách cho phép kênh hoặc các quy tắc kích hoạt nhóm (nhắc đến/lệnh/v.v.). Chúng chỉ thay đổi _những tác nhân nào sẽ chạy_ khi một tin nhắn đủ điều kiện để được xử lý.

### Cách ly phiên

Mỗi tác tử trong một broadcast group duy trì hoàn toàn tách biệt:

- **Khóa phiên** (`agent:alfred:whatsapp:group:120363...` so với `agent:baerbel:whatsapp:group:120363...`)
- **Lịch sử hội thoại** (tác tử không thấy tin nhắn của tác tử khác)
- **Không gian làm việc** (sandbox riêng nếu được cấu hình)
- **Quyền truy cập công cụ** (danh sách cho phép/từ chối khác nhau)
- **Bộ nhớ/ngữ cảnh** (IDENTITY.md, SOUL.md riêng biệt, v.v.)
- **Bộ đệm ngữ cảnh nhóm** (các tin nhắn nhóm gần đây dùng cho ngữ cảnh) được chia sẻ theo từng peer, vì vậy tất cả các tác tử broadcast đều thấy cùng một ngữ cảnh khi được kích hoạt

Điều này cho phép mỗi tác tử có:

- Cá tính khác nhau
- Quyền truy cập công cụ khác nhau (ví dụ: chỉ đọc so với đọc-ghi)
- Mô hình khác nhau (ví dụ: opus so với sonnet)
- Các Skills khác nhau được cài đặt

### Ví dụ: Các phiên được cách ly

Trong nhóm `120363403215116621@g.us` với các tác tử `["alfred", "baerbel"]`:

**Ngữ cảnh của Alfred:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Ngữ cảnh của Bärbel:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## Thực hành tốt nhất

### 1. Giữ cho các Agent tập trung

Thiết kế mỗi tác tử với một trách nhiệm rõ ràng, duy nhất:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Tốt:** Mỗi tác tử có một nhiệm vụ  
❌ **Không tốt:** Một tác tử "dev-helper" chung chung

### 2. 2. Use Descriptive Names

Làm rõ mỗi tác tử làm gì:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. 38. Cấu hình quyền truy cập tool khác nhau

Chỉ cấp cho tác tử những công cụ chúng cần:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. 4. 39. Giám sát hiệu năng

Với nhiều tác tử, hãy cân nhắc:

- Sử dụng `"strategy": "parallel"` (mặc định) để có tốc độ
- Giới hạn broadcast groups ở mức 5–10 tác tử
- Dùng mô hình nhanh hơn cho các tác tử đơn giản

### 40. 5. 41. Xử lý lỗi một cách nhẹ nhàng

42. Các agent thất bại độc lập. 43. Lỗi của một agent không chặn các agent khác:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Khả năng tương thích

### Nhà cung cấp

Broadcast groups hiện hoạt động với:

- ✅ WhatsApp (đã triển khai)
- 🚧 Telegram (dự kiến)
- 🚧 Discord (dự kiến)
- 🚧 Slack (dự kiến)

### Định tuyến

Broadcast groups hoạt động song song với định tuyến hiện có:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Chỉ alfred phản hồi (định tuyến thông thường)
- `GROUP_B`: agent1 VÀ agent2 cùng phản hồi (broadcast)

**Thứ tự ưu tiên:** `broadcast` có ưu tiên cao hơn `bindings`.

## Xử lý sự cố

### Tác tử không phản hồi

**Kiểm tra:**

1. ID tác tử tồn tại trong `agents.list`
2. Định dạng peer ID chính xác (ví dụ: `120363403215116621@g.us`)
3. Tác tử không nằm trong danh sách từ chối

**Gỡ lỗi:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Chỉ một tác tử phản hồi

**Nguyên nhân:** Peer ID có thể nằm trong `bindings` nhưng không nằm trong `broadcast`.

**Cách khắc phục:** Thêm vào cấu hình broadcast hoặc loại bỏ khỏi bindings.

### Vấn đề hiệu năng

**Nếu chậm khi có nhiều tác tử:**

- Giảm số lượng tác tử mỗi nhóm
- Dùng mô hình nhẹ hơn (sonnet thay vì opus)
- Kiểm tra thời gian khởi động sandbox

## Ví dụ

### Ví dụ 1: Nhóm review mã nguồn

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Người dùng gửi:** Đoạn mã  
**Phản hồi:**

- code-formatter: "Đã sửa thụt lề và thêm type hints"
- security-scanner: "⚠️ Lỗ hổng SQL injection ở dòng 12"
- test-coverage: "Độ bao phủ là 45%, thiếu test cho các trường hợp lỗi"
- docs-checker: "Thiếu docstring cho hàm `process_data`"

### Ví dụ 2: Hỗ trợ đa ngôn ngữ

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## Tham chiếu API

### Schema cấu hình

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Các trường

- `strategy` (tùy chọn): Cách xử lý các tác tử
  - `"parallel"` (mặc định): Tất cả tác tử xử lý đồng thời
  - `"sequential"`: Các tác tử xử lý theo thứ tự trong mảng
- `[peerId]`: JID nhóm WhatsApp, số E.164, hoặc peer ID khác
  - Giá trị: Mảng ID tác tử nên xử lý tin nhắn

## Giới hạn

1. **Số tác tử tối đa:** Không có giới hạn cứng, nhưng 10+ tác tử có thể chậm
2. **Ngữ cảnh chia sẻ:** Các tác tử không thấy phản hồi của nhau (theo thiết kế)
3. **Thứ tự tin nhắn:** Phản hồi song song có thể đến theo bất kỳ thứ tự nào
4. **Giới hạn tốc độ:** Tất cả tác tử đều tính vào giới hạn tốc độ của WhatsApp

## Cải tiến trong tương lai

Các tính năng dự kiến:

- [ ] Chế độ ngữ cảnh chia sẻ (các tác tử thấy phản hồi của nhau)
- [ ] Điều phối tác tử (các tác tử có thể báo hiệu cho nhau)
- [ ] Lựa chọn tác tử động (chọn tác tử dựa trên nội dung tin nhắn)
- [ ] Ưu tiên tác tử (một số tác tử phản hồi trước các tác tử khác)

## Xem thêm

- [Cấu Hình Đa Tác Nhân](/tools/multi-agent-sandbox-tools)
- [Routing Configuration](/channels/channel-routing)
- [Session Management](/concepts/sessions)
