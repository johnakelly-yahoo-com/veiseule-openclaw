---
title: CI 파이프라인
description: OpenClaw CI 파이프라인 작동 방식
---

# CI 파이프라인

CI는 `main`에 대한 모든 push와 모든 pull request에서 실행됩니다. 문서 또는 네이티브 코드만 변경된 경우 비용이 큰 작업을 건너뛰도록 스마트 스코핑을 사용합니다.

## 작업 개요

| 작업                | 목적                                               | 실행 시점                   |
| ----------------- | ------------------------------------------------ | ----------------------- |
| `docs-scope`      | 문서 전용 변경 사항 감지                                   | 항상                      |
| `changed-scope`   | 변경된 영역(node/macos/android) 감지 | 문서가 아닌 PR               |
| `check`           | TypeScript 타입, lint, 포맷 검사                       | 문서 외 변경 사항              |
| `check-docs`      | Markdown lint + 깨진 링크 검사                         | 문서 변경 시                 |
| `code-analysis`   | LOC 임계값 검사(1000줄)             | PR 전용                   |
| `secrets`         | 유출된 시크릿 감지                                       | 항상                      |
| `build-artifacts` | dist를 한 번 빌드하고 다른 작업과 공유                         | 문서 외, node 변경 사항        |
| `release-check`   | npm pack 내용 검증                                   | 빌드 이후                   |
| `checks`          | Node/Bun 테스트 + 프로토콜 검사                           | 문서 외, node 변경 사항        |
| `checks-windows`  | Windows 전용 테스트                                   | 문서 외, node 변경 사항        |
| `macos`           | Swift lint/빌드/테스트 + TS 테스트                       | macos 변경이 포함된 PR        |
| `android`         | Gradle 빌드 + 테스트                                  | 문서 이외 변경 사항, android 변경 |

## Fail-Fast 순서

저비용 검사가 고비용 검사보다 먼저 실패하도록 작업이 정렬됩니다:

1. `docs-scope` + `code-analysis` + `check` (병렬, 약 1~2분)
2. `build-artifacts` (위 작업 완료 후 실행)
3. `checks`, `checks-windows`, `macos`, `android` (빌드 완료 후 실행)

## 러너

| 러너                              | 작업                            |
| ------------------------------- | ----------------------------- |
| `blacksmith-4vcpu-ubuntu-2404`  | 대부분의 Linux 작업                 |
| `blacksmith-4vcpu-windows-2025` | `checks-windows`              |
| `macos-latest`                  | `macos`, `ios`                |
| `ubuntu-latest`                 | 범위 감지 (경량) |

## 로컬 대체 명령어

```bash
pnpm check          # 타입 + lint + 포맷
pnpm test           # vitest 테스트
pnpm check:docs     # 문서 포맷 + lint + 깨진 링크
pnpm release:check  # npm pack 검증
```

