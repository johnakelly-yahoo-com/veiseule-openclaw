---
title: "提交 PR"
---

优秀的 PR 应该易于审查：审阅者应能快速理解意图、验证行为，并安全地合并更改。本指南介绍如何为人工和 LLM 审查提交简洁、高信号的 PR。

## 什么是优秀的 PR

- [ ] 说明问题是什么、为什么重要，以及所做的更改。
- [ ] 保持更改聚焦。避免大范围重构。
- [ ] 总结对用户可见的更改，以及配置/默认值的变更。
- [ ] 列出测试覆盖情况、跳过的测试及其原因。
- [ ] 6. 添加证据：日志、截图或录屏（UI/UX）。
- [ ] 代码暗号：如果你阅读了本指南，请在 PR 描述中写入“lobster-biscuit”。
- [ ] 在创建 PR 之前运行/修复相关的 `pnpm` 命令。
- [ ] Search codebase and GitHub for related functionality/issues/fixes.
- [ ] 7. 基于证据或观察提出结论。
- [ ] Good title: verb + scope + outcome (e.g., `Docs: add PR and issue templates`).

Be concise; concise review > grammar. Omit any non-applicable sections.

### Baseline validation commands (run/fix failures for your change)

- `pnpm lint`
- `pnpm check`
- `pnpm build`
- `pnpm test`
- Protocol changes: `pnpm protocol:check`

## Progressive disclosure

- Top: summary/intent
- Next: changes/risks
- Next: test/verification
- Last: implementation/evidence

## Common PR types: specifics

- [ ] Fix: Add repro, root cause, verification.
- [ ] Feature: Add use cases, behavior/demos/screenshots (UI).
- [ ] Refactor: State "no behavior change", list what moved/simplified.
- [ ] Chore: State why (e.g., build time, CI, dependencies).
- [ ] Docs: Before/after context, link updated page, run `pnpm format`.
- [ ] Test: What gap is covered; how it prevents regressions.
- [ ] Perf: Add before/after metrics, and how measured.
- [ ] UX/UI: Screenshots/video, note accessibility impact.
- [ ] Infra/Build: Environments/validation.
- [ ] Security: Summarize risk, repro, verification, no sensitive data. Grounded claims only.

## Checklist

- [ ] Clear problem/intent
- [ ] Focused scope
- [ ] 8. 列出行为变更
- [ ] List and result of tests
- [ ] Manual test steps (when applicable)
- [ ] No secrets/private data
- [ ] Evidence-based

## General PR Template

```md
#### Summary

#### Behavior Changes

#### Codebase and GitHub Search

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort (self-reported):
- Agent notes (optional, cite evidence):
```

## PR Type templates (replace with your type)

### Fix

```md
#### Summary

#### Repro Steps

#### Root Cause

#### Behavior Changes

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Feature

```md
9. #### 摘要

#### 用例

#### 行为变更

#### 现有功能检查

- [ ] 我已在代码库中搜索现有功能。
      已执行的搜索（1-3 个要点）：
  -
  -

#### 测试

#### 手动测试（如不适用可省略）

### 前置条件

-

### 步骤

1.
2.

#### 证据（如不适用可省略）

**签字确认**

- 使用的模型：
- 提交者投入：
- Agent 备注：
```

### Refactor

```md
#### Summary

#### Scope

#### No Behavior Change Statement

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Chore/Maintenance

```md
#### Summary

#### Why This Matters

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Docs

```md
#### Summary

#### Pages Updated

#### Before/After

#### Formatting

pnpm format

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Test

```md
#### Summary

#### Gap Covered

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Perf

```md
#### Summary

#### Baseline

#### After

#### Measurement Method

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### UX/UI

```md
#### Summary

#### Screenshots or Video

#### Accessibility Impact

#### Tests

#### Manual Testing

### Prerequisites

-

### Steps

1.
2. **Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Infra/Build

```md
#### Summary

#### Environments Affected

#### Validation Steps

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```

### Security

```md
#### Summary

#### Risk Summary

#### Repro Steps

#### Mitigation or Fix

#### Verification

#### Tests

#### Manual Testing (omit if N/A)

### Prerequisites

-

### Steps

1.
2.

#### Evidence (omit if N/A)

**Sign-Off**

- Models used:
- Submitter effort:
- Agent notes:
```


