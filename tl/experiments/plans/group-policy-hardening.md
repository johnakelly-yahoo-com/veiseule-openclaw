---
title: "Pagpapatibay ng Telegram Allowlist"
---

# Pagpapatibay ng Telegram Allowlist

**Petsa**: 2026-01-05  
**Katayuan**: Kumpleto  
**PR**: #216

## Buod

Ang mga allowlist ng Telegram ay tumatanggap na ngayon ng mga prefix na `telegram:` at `tg:` nang hindi sensitibo sa laki ng titik, at pinahihintulutan ang
accidental whitespace. This aligns inbound allowlist checks with outbound send normalization.

## Ano ang nagbago

- Ang mga prefix na `telegram:` at `tg:` ay tinatrato nang pareho (case-insensitive).
- Ang mga entry sa allowlist ay tinitrim; ang mga walang laman na entry ay binabalewala.

## Mga halimbawa

Lahat ng ito ay tinatanggap para sa parehong ID:

- `telegram:123456`
- `TG:123456`
- `tg:123456`

## Bakit ito mahalaga

Ang pag-normalize ay umiiwas sa
mga false negative kapag nagpapasya kung tutugon sa mga DM o grupo. Normalizing avoids
false negatives when deciding whether to respond in DMs or groups.

## Kaugnay na docs

- [Mga Group Chat](/channels/groups)
- [Provider ng Telegram](/channels/telegram)


