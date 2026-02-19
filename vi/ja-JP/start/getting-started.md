---
read_when:
  - Thiết lập lần đầu từ con số 0
  - Tìm con đường ngắn nhất để có một cuộc trò chuyện hoạt động
summary: Hãy cài đặt OpenClaw và thực hiện cuộc trò chuyện đầu tiên chỉ trong vài phút.
title: Bắt đầu
x-i18n:
  generated_at: "2026-02-08T17:15:16Z"
  model: claude-opus-4-5
  provider: pi
  source_hash: 27aeeb3d18c495380e94e6b011b0df3def518535c9f1eee504f04871d8a32269
  source_path: start/getting-started.md
  workflow: 15
---

# Bắt đầu

Mục tiêu: Từ con số 0, thiết lập tối thiểu để thực hiện cuộc trò chuyện hoạt động đầu tiên.

<Info>
Cách trò chuyện nhanh nhất: Mở Control UI (không cần cấu hình kênh). Chạy `openclaw dashboard` để trò chuyện trên trình duyệt, hoặc
<Tooltip headline="Gatewayホスト" tip="OpenClaw Gatewayサービスを実行しているマシン。">Gatewayホスト</Tooltip>mở `http://127.0.0.1:18789/` trên
Gateway host.
Tài liệu: [Dashboard](/web/dashboard) và [Control UI](/web/control-ui)。
</Info>

## Điều kiện tiên quyết

- Node 22 trở lên

<Tip>
Nếu không chắc, hãy kiểm tra phiên bản Node bằng `node --version`.
</Tip>

## Thiết lập nhanh (CLI)

<Steps>
  <Step title="OpenClawをインストール（推奨）">
    <Tabs>
      <Tab title="macOS/Linux">
        ```bash
        curl -fsSL https://openclaw.ai/install.sh | bash
        ```
      
</Tab>
      <Tab title="Windows (PowerShell)">
        ```powershell
        iwr -useb https://openclaw.ai/install.ps1 | iex
        ```
      
</Tab>
    
</Tabs>

    ```
    <Note>
    Các phương thức cài đặt và yêu cầu khác: [Cài đặt](/install)。
    
</Note>
    ```

  
</Step>
  <Step title="オンボーディングウィザードを実行">
    ```bash
    openclaw onboard --install-daemon
    ```

    ```
    Trình hướng dẫn sẽ cấu hình xác thực, thiết lập Gateway và các kênh tùy chọn.
    Xem chi tiết tại [Onboarding Wizard](/start/wizard)。
    ```

  
</Step>
  <Step title="Gatewayを確認">
    Nếu bạn đã cài đặt dịch vụ, nó sẽ đang chạy sẵn:


    ````
    ```bash
    openclaw gateway status
    ```
    ````

  
</Step>
  <Step title="Control UIを開く">
    ```bash
    openclaw dashboard
    ```
  
</Step>
</Steps>

<Check>
Khi Control UI được tải, Gateway đã sẵn sàng sử dụng.
</Check>

## Tùy chọn kiểm tra và tính năng bổ sung

<AccordionGroup>
  <Accordion title="Gatewayをフォアグラウンドで実行">
    Hữu ích cho kiểm tra nhanh và khắc phục sự cố.


    ````
    ```bash
    openclaw gateway --port 18789
    ```
    ````

  
</Accordion>
  <Accordion title="テストメッセージを送信">
    Yêu cầu kênh đã được cấu hình.


    ````
    ```bash
    openclaw message send --target +15555550123 --message "Hello from OpenClaw"
    ```
    ````

  
</Accordion>
</AccordionGroup>

## Tìm hiểu thêm

<Columns>
  <Card title="オンボーディングウィザード（詳細）" href="/start/wizard">
    Tham khảo đầy đủ về CLI wizard và các tùy chọn nâng cao.
  
</Card>
  <Card title="macOSアプリのオンボーディング" href="/start/onboarding">
    Quy trình chạy lần đầu của ứng dụng macOS.
  
</Card>
</Columns>

## Trạng thái sau khi hoàn tất

- Gateway đang chạy
- Xác thực đã được cấu hình
- Truy cập Control UI hoặc kênh đã kết nối

## Bước tiếp theo

- Bảo mật và phê duyệt DM: [Ghép nối](/channels/pairing)
- Kết nối thêm kênh: [Kênh](/channels)
- Quy trình nâng cao và build từ mã nguồn: [Thiết lập](/start/setup)
