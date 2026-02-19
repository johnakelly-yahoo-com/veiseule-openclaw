---
title: CI Pipeline
description: Cách CI pipeline của OpenClaw hoạt động
---

# CI Pipeline

CI chạy mỗi khi có push lên `main` và mỗi pull request. Nó sử dụng phạm vi thông minh để bỏ qua các job tốn tài nguyên khi chỉ có tài liệu hoặc mã native thay đổi.

## Tổng quan Job

| Job               | Mục đích                                                                  | Khi chạy                                             |
| ----------------- | ------------------------------------------------------------------------- | ---------------------------------------------------- |
| `docs-scope`      | Phát hiện thay đổi chỉ liên quan đến tài liệu                             | Luôn luôn                                            |
| `changed-scope`   | Phát hiện khu vực nào đã thay đổi (node/macos/android) | PR không liên quan đến tài liệu                      |
| `check`           | Kiểm tra kiểu TypeScript, lint, định dạng                                 | Thay đổi không liên quan đến tài liệu                |
| `check-docs`      | Lint Markdown + kiểm tra liên kết hỏng                                    | Tài liệu thay đổi                                    |
| `code-analysis`   | Kiểm tra ngưỡng LOC (1000 dòng)                        | Chỉ PR                                               |
| `secrets`         | Phát hiện secret bị rò rỉ                                                 | Luôn luôn                                            |
| `build-artifacts` | Build dist một lần, chia sẻ với các job khác                              | Thay đổi không liên quan đến tài liệu, thay đổi node |
| `release-check`   | Xác thực nội dung npm pack                                                | Sau khi build                                        |
| `checks`          | Test Node/Bun + kiểm tra protocol                                         | Thay đổi không liên quan đến tài liệu, thay đổi node |
| `checks-windows`  | Test dành riêng cho Windows                                               | Thay đổi không liên quan đến tài liệu, thay đổi node |
| `macos`           | Swift lint/build/test + test TS                                           | PR có thay đổi macos                                 |
| `android`         | Build Gradle + chạy test                                                  | Các thay đổi không liên quan đến tài liệu, android   |

## Thứ tự Fail-Fast

Các job được sắp xếp để những kiểm tra chi phí thấp thất bại trước khi các job tốn kém hơn chạy:

1. `docs-scope` + `code-analysis` + `check` (song song, ~1-2 phút)
2. `build-artifacts` (bị chặn bởi các bước trên)
3. `checks`, `checks-windows`, `macos`, `android` (bị chặn bởi bước build)

## Runners

| Runner                          | Jobs                                       |
| ------------------------------- | ------------------------------------------ |
| `blacksmith-4vcpu-ubuntu-2404`  | Hầu hết các job Linux                      |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`                           |
| `macos-latest`                  | `macos`, `ios`                             |
| `ubuntu-latest`                 | Phát hiện phạm vi (nhẹ) |

## Tương đương khi chạy cục bộ

```bash
pnpm check          # types + lint + format
pnpm test           # vitest tests
pnpm check:docs     # định dạng + lint tài liệu + liên kết hỏng
pnpm release:check  # xác thực npm pack
```
