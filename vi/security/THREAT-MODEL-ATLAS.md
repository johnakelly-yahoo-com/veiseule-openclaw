# Mô hình Đe doạ OpenClaw v1.0

## Khung MITRE ATLAS

**Phiên bản:** 1.0-draft  
**Cập nhật lần cuối:** 2026-02-04  
**Phương pháp luận:** MITRE ATLAS + Sơ đồ Luồng Dữ liệu  
**Khung:** [MITRE ATLAS](https://atlas.mitre.org/) (Bức tranh Đe doạ Đối kháng cho Hệ thống AI)

### Ghi nhận khung

Mô hình đe doạ này được xây dựng dựa trên [MITRE ATLAS](https://atlas.mitre.org/), khung tiêu chuẩn ngành để ghi nhận các mối đe doạ đối kháng đối với hệ thống AI/ML. ATLAS được duy trì bởi [MITRE](https://www.mitre.org/) với sự hợp tác của cộng đồng an ninh AI.

**Tài nguyên ATLAS chính:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Đóng góp cho ATLAS](https://atlas.mitre.org/resources/contribute)

### Đóng góp cho mô hình đe doạ này

Đây là tài liệu “sống” được duy trì bởi cộng đồng OpenClaw. Xem [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) để biết hướng dẫn đóng góp:

- Báo cáo mối đe doạ mới  
- Cập nhật mối đe doạ hiện có  
- Đề xuất chuỗi tấn công  
- Đề xuất biện pháp giảm thiểu  

---

## 1. Giới thiệu

### 1.1 Mục đích

Mô hình đe doạ này ghi nhận các mối đe doạ đối kháng đối với nền tảng AI agent OpenClaw và chợ kỹ năng ClawHub, sử dụng khung MITRE ATLAS được thiết kế chuyên biệt cho hệ thống AI/ML.

### 1.2 Phạm vi

| Thành phần              | Bao gồm | Ghi chú                                          |
| ----------------------- | ------- | ------------------------------------------------ |
| Môi trường chạy Tác nhân OpenClaw | Có      | Thực thi agent cốt lõi, gọi công cụ, phiên làm việc |
| Gateway                | Có      | Xác thực, định tuyến, tích hợp kênh             |
| Tích hợp Kênh          | Có      | WhatsApp, Telegram, Discord, Signal, Slack, v.v. |
| Chợ ClawHub    | Có      | Xuất bản kỹ năng, kiểm duyệt, phân phối         |
| MCP Servers            | Có      | Nhà cung cấp công cụ bên ngoài                  |
| Thiết bị Người dùng    | Một phần| Ứng dụng di động, ứng dụng desktop              |

### 1.3 Ngoài phạm vi

Không có nội dung nào được loại trừ rõ ràng khỏi mô hình đe doạ này.

---

## 2. Kiến trúc Hệ thống

### 2.1 Ranh giới Tin cậy

```
┌─────────────────────────────────────────────────────────────────┐
│                    VÙNG KHÔNG TIN CẬY                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 1: Truy cập Kênh                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Ghép nối thiết bị (thời gian chờ 30s)                  │   │
│  │  • Xác thực AllowFrom / AllowList                         │   │
│  │  • Xác thực Token/Password/Tailscale                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 2: Cô lập Phiên                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT SESSIONS                          │   │
│  │  • Khóa phiên = agent:channel:peer                       │   │
│  │  • Chính sách công cụ theo từng agent                    │   │
│  │  • Ghi log hội thoại                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 3: Thực thi Công cụ                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  EXECUTION SANDBOX                        │   │
│  │  • Docker sandbox HOẶC Host (exec-approvals)             │   │
│  │  • Thực thi từ xa qua Node                               │   │
│  │  • Bảo vệ SSRF (ghim DNS + chặn IP)                      │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 4: Nội dung Bên ngoài               │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │          URL / EMAIL / WEBHOOK ĐƯỢC TẢI VỀ               │   │
│  │  • Bao bọc nội dung bên ngoài (thẻ XML)                  │   │
│  │  • Chèn thông báo bảo mật                                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│           RANH GIỚI TIN CẬY 5: Chuỗi Cung ứng                   │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Xuất bản kỹ năng (semver, bắt buộc SKILL.md)          │   │
│  │  • Cờ kiểm duyệt dựa trên mẫu                             │   │
│  │  • Quét VirusTotal (sắp ra mắt)                          │   │
│  │  • Xác minh tuổi tài khoản GitHub                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Luồng Dữ liệu

| Luồng | Nguồn   | Đích        | Dữ liệu            | Bảo vệ               |
| ----- | ------- | ----------- | ------------------ | -------------------- |
| F1    | Kênh    | Gateway     | Tin nhắn người dùng| TLS, AllowFrom       |
| F2    | Gateway | Agent       | Tin nhắn đã định tuyến | Cô lập phiên    |
| F3    | Agent   | Công cụ     | Lời gọi công cụ    | Thực thi chính sách  |
| F4    | Agent   | Bên ngoài   | Yêu cầu web_fetch  | Chặn SSRF            |
| F5    | ClawHub | Agent       | Mã kỹ năng         | Kiểm duyệt, quét     |
| F6    | Agent   | Kênh        | Phản hồi           | Lọc đầu ra           |

---

## 3. Phân tích Đe doạ theo Chiến thuật ATLAS

### 3.1 Trinh sát (AML.TA0002)

#### T-RECON-001: Phát hiện Endpoint Agent

| Thuộc tính             | Giá trị                                                             |
| ---------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**           | AML.T0006 - Quét Chủ động                                           |
| **Mô tả**              | Kẻ tấn công quét các endpoint gateway OpenClaw bị lộ               |
| **Vector tấn công**    | Quét mạng, truy vấn shodan, liệt kê DNS                            |
| **Thành phần bị ảnh hưởng** | Gateway, các API endpoint công khai                          |
| **Giảm thiểu hiện tại** | Tùy chọn xác thực Tailscale, mặc định bind loopback               |
| **Rủi ro còn lại**     | Trung bình - Gateway công khai có thể bị phát hiện                |
| **Khuyến nghị**        | Tài liệu triển khai an toàn, thêm rate limiting cho endpoint phát hiện |

#### T-RECON-002: Thăm dò Tích hợp Kênh

| Thuộc tính             | Giá trị                                                             |
| ---------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**           | AML.T0006 - Quét Chủ động                                           |
| **Mô tả**              | Kẻ tấn công thăm dò kênh nhắn tin để xác định tài khoản do AI quản lý |
| **Vector tấn công**    | Gửi tin nhắn thử nghiệm, quan sát mẫu phản hồi                    |
| **Thành phần bị ảnh hưởng** | Tất cả tích hợp kênh                                         |
| **Giảm thiểu hiện tại** | Không có biện pháp cụ thể                                          |
| **Rủi ro còn lại**     | Thấp - Giá trị hạn chế chỉ từ việc phát hiện                      |
| **Khuyến nghị**        | Cân nhắc ngẫu nhiên hóa thời gian phản hồi                        |

---
