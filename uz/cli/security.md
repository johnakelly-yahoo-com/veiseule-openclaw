---
summary: "`openclaw security` uchun CLI ma’lumotnomasi (keng tarqalgan xavfsizlik xatolarini audit qilish va tuzatish)"
read_when:
  - You want to run a quick security audit on config/state
  - You want to apply safe “fix” suggestions (chmod, tighten defaults)
title: "security"
---

# `openclaw security`

Xavfsizlik vositalari (audit va ixtiyoriy tuzatishlar).

Bog‘liq:

- `openclaw sessions`

## Audit

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

The audit warns when multiple DM senders share the main session and recommends **secure DM mode**: `session.dmScope="per-channel-peer"` (or `per-account-channel-peer` for multi-account channels) for shared inboxes.
It also warns when small models (`<=300B`) are used without sandboxing and with web/browser tools enabled.
