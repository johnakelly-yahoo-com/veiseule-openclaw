---
summary: "Qanday qilib yuqori signalga ega PR yuborish kerak"
title: "PR yuborish"
---

Yaxshi PRlarni ko‘rib chiqish oson bo‘ladi: ko‘rib chiquvchilar tezda maqsadni tushunishi, xatti-harakatni tekshirishi va o‘zgarishlarni xavfsiz tarzda qabul qilishi kerak. Ushbu qo‘llanma insonlar va LLM tomonidan ko‘rib chiqish uchun qisqa va mazmunli (yuqori signalga ega) yuborishlarni qamrab oladi.

## Yaxshi PR nimasi bilan ajralib turadi

- [ ] Muammoni, uning nega muhimligini va kiritilgan o‘zgarishni tushuntiring.
- [ ] O‘zgarishlarni aniq va yo‘naltirilgan holda saqlang. Keng qamrovli refaktorlardan qoching.
- [ ] Foydalanuvchiga ko‘rinadigan/konfiguratsiya/standart sozlamalardagi o‘zgarishlarni qisqacha bayon qiling.
- [ ] Test qamrovini, o‘tkazib yuborilganlarini va sabablarini sanab chiqing.
- [ ] Dalillar qo‘shing: loglar, skrinshotlar yoki yozuvlar (UI/UX).
- [ ] Kod so‘zi: agar ushbu qo‘llanmani o‘qigan bo‘lsangiz, PR tavsifiga “lobster-biscuit” deb yozing.
- [ ] PR yaratishdan oldin tegishli `pnpm` buyruqlarini ishga tushiring/tuzating.
- [ ] Search codebase and GitHub for related functionality/issues/fixes.
- [ ] Base claims on evidence or observation.
- [ ] Yaxshi sarlavha: fe’l + qamrov + natija (masalan, `Docs: add PR and issue templates`).

Be concise; concise review > grammar. Omit any non-applicable sections.

### Baseline validation commands (run/fix failures for your change)

- `pnpm lint`
- `pnpm check`
- `pnpm build`
- `pnpm test`
- Protocol changes: `pnpm protocol:check`

## Progressive disclosure

- Yuqori: xulosa/niyat
- Next: changes/risks
- Next: test/verification
- Last: implementation/evidence

## Common PR types: specifics

- [ ] Fix: Add repro, root cause, verification.
- [ ] Feature: Add use cases, behavior/demos/screenshots (UI).
- [ ] Refactor: "xulq-atvorda o‘zgarish yo‘q" deb yozing, ko‘chirilgan/soddalashtirilgan narsalarni sanab chiqing.
- [ ] Chore: nima uchunligini yozing (masalan, build vaqti, CI, bog‘liqliklar).
- [ ] Docs: Before/after context, link updated page, run `pnpm format`.
- [ ] Test: What gap is covered; how it prevents regressions.
- [ ] Perf: Add before/after metrics, and how measured.
- [ ] UX/UI: Screenshots/video, note accessibility impact.
- [ ] Infra/Build: Environments/validation.
- [ ] Security: Summarize risk, repro, verification, no sensitive data. Grounded claims only.

## Checklist

- [ ] Aniq muammo/niyat
- [ ] Focused scope
- [ ] List behavior changes
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
#### Summary

#### Use Cases

#### Behavior Changes

#### Existing Functionality Check

- [ ] I searched the codebase for existing functionality.
      Searches performed (1-3 bullets):
  -
  -

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
#### Xulosa

#### Yangilangan sahifalar

#### Oldin/Keyin

#### Formatlash

pnpm format

#### Dalillar (N/A bo‘lsa, tashlab yuboring)

**Tasdiqlash**

- Ishlatilgan modellar:
- Yuboruvchi hissasi:
- Agent eslatmalari:
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
