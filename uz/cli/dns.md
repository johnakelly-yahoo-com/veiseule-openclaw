---
summary: "`openclaw dns` uchun CLI ma’lumotnomasi (keng hududli kashfiyot yordamchilari)"
read_when:
  - Sizga Tailscale + CoreDNS orqali keng hududli kashfiyot (DNS-SD) kerak bo‘lsa
  - Maxsus kashfiyot domeni (masalan: openclaw.internal) uchun split DNS sozlayotgan bo‘lsangiz
title: "dns"
---

# `openclaw dns`

Keng hududli kashfiyot (Tailscale + CoreDNS) uchun DNS yordamchilari. Hozircha macOS + Homebrew CoreDNS’ga qaratilgan.

Bog‘liq:

- Gateway kashfiyoti: [Discovery](/gateway/discovery)
- Keng hududli kashfiyot sozlamalari: [Configuration](/gateway/configuration)

## O‘rnatish

```bash
openclaw dns setup
openclaw dns setup --apply
```