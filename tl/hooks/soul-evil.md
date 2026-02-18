---
title: "SOUL Evil Hook"
---

# SOUL Evil Hook

Pinapalitan ng SOUL Evil hook ang **ini-inject** na nilalaman ng `SOUL.md` ng `SOUL_EVIL.md` habang
a purge window or by random chance. It does **not** modify files on disk.

## Paano Ito Gumagana

Kapag tumatakbo ang `agent:bootstrap`, maaaring palitan ng hook ang nilalaman ng `SOUL.md` sa memorya
before the system prompt is assembled. If `SOUL_EVIL.md` is missing or empty,
OpenClaw logs a warning and keeps the normal `SOUL.md`.

Ang mga sub-agent run ay **hindi** kasama ang `SOUL.md` sa kanilang mga bootstrap file, kaya
walang epekto ang hook na ito sa mga sub-agent.

## Paganahin

```bash
openclaw hooks enable soul-evil
```

Pagkatapos, itakda ang config:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

Likhain ang `SOUL_EVIL.md` sa root ng agent workspace (katabi ng `SOUL.md`).

## Mga Opsyon

- `file` (string): alternatibong SOUL filename (default: `SOUL_EVIL.md`)
- `chance` (number 0–1): random na tsansa bawat run na gamitin ang `SOUL_EVIL.md`
- `purge.at` (HH:mm): araw-araw na simula ng purge (24-oras na format)
- `purge.duration` (duration): haba ng window (hal. `30s`, `10m`, `1h`)

**Precedence:** ang purge window ang nangingibabaw kaysa sa tsansa.

**Timezone:** gumagamit ng `agents.defaults.userTimezone` kapag naka-set; kung hindi, timezone ng host.

## Mga Tala

- Walang anumang file na sinusulat o binabago sa disk.
- Kung ang `SOUL.md` ay wala sa bootstrap list, walang gagawin ang hook.

## Tingnan Din

- [Hooks](/automation/hooks)

