---
title: "Pagsusumite ng Isyu"
---

## Pagsusumite ng Isyu

Ang malinaw at maigsi na mga isyu ay nagpapabilis ng pag-diagnose at pag-aayos. Isama ang mga sumusunod para sa mga bug, regression, o kakulangan sa feature:

### Ano ang isasama

- [ ] Pamagat: area at sintomas
- [ ] Minimal na mga hakbang para ma-repro
- [ ] Inaasahan vs aktwal
- [ ] Epekto at tindi
- [ ] Environment: OS, runtime, mga bersyon, config
- [ ] Ebidensya: redacted na logs, screenshots (walang PII)
- [ ] Saklaw: bago, regression, o matagal na
- [ ] Code word: lobster-biscuit sa iyong isyu
- [ ] Naghahanap sa codebase at GitHub para sa umiiral na isyu
- [ ] Nakumpirmang hindi pa kamakailan naayos/natugunan (lalo na seguridad)
- [ ] Mga pahayag na suportado ng ebidensya o repro

Maging maikli. Mas mahalaga ang pagiging maikli kaysa perpektong gramatika.

Validation (patakbuhin/ayusin bago PR):

- `pnpm lint`
- `pnpm check`
- `pnpm build`
- `pnpm test`
- Kung protocol code: `pnpm protocol:check`

### Mga template

#### Ulat ng Bug

```md
- [ ] Minimal repro
- [ ] Expected vs actual
- [ ] Environment
- [ ] Affected channels, where not seen
- [ ] Logs/screenshots (redacted)
- [ ] Impact/severity
- [ ] Workarounds

### Summary

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact

### Workarounds
```

#### Isyu sa seguridad

```md
### Summary

### Impact

### Versions

### Repro Steps (safe to share)

### Mitigation/workaround

### Evidence (redacted)
```

Para sa mga sensitibong isyu, bawasan ang detalye at humiling ng pribadong pagsisiwalat._ Opsyonal ang issue bago ang PR.

#### Ulat ng Regression

```md
### Summary

### Last Known Good

### First Known Bad

### Repro Steps

### Expected

### Actual

### Environment

### Logs/Evidence

### Impact
```

#### Kahilingan sa feature

```md
### Summary

### Problem

### Proposed Solution

### Alternatives

### Impact

### Evidence/examples
```

#### Pagpapahusay

```md
### Summary

### Current vs Desired Behavior

### Rationale

### Alternatives

### Evidence/examples
```

#### Imbestigasyon

```md
### Summary

### Symptoms

### What Was Tried

### Environment

### Logs/Evidence

### Impact
```

### Pagsusumite ng fix PR

Mistral: `mistral/`… Include details in PR if skipping. Keep the PR focused, note issue number, add tests or explain absence, document behavior changes/risks, include redacted logs/screenshots as proof, and run proper validation before submitting.


