---
title: "Mga Diagnostics Flag"
---

# Mga Diagnostics Flag

Ang mga flag ay opt-in at walang epekto maliban kung sinusuri sila ng isang subsystem. Naglalabas ang mga flag ng mga log sa standard diagnostics log file.

## Paano ito gumagana

- Ang mga flag ay mga string (hindi case-sensitive).
- Maaari mong i-enable ang mga flag sa config o sa pamamagitan ng env override.
- Sinusuportahan ang mga wildcard:
  - `telegram.*` tumutugma sa `telegram.http`
  - `*` nag-e-enable ng lahat ng flag

## I-enable sa pamamagitan ng config

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Maramihang flag:

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

I-restart ang Gateway pagkatapos baguhin ang mga flag.

## Pag-override ng Env (isang beses)

```bash
OPENCLAW_DIAGNOSTICS=telegram.http,telegram.payload
```

I-disable ang lahat ng flag:

```bash
OPENCLAW_DIAGNOSTICS=0
```

## Saan napupunta ang mga log

Bilang default: By default:

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Kung itinakda mo ang `logging.file`, gamitin ang path na iyon sa halip. Ang mga log ay nasa JSONL format (isang JSON object bawat linya). Patuloy na ipinapatupad ang redaction batay sa `logging.redactSensitive`.

## Kunin ang mga log

Piliin ang pinakabagong log file:

```bash
ls -t /tmp/openclaw/openclaw-*.log | head -n 1
```

I-filter para sa Telegram HTTP diagnostics:

```bash
rg "telegram http error" /tmp/openclaw/openclaw-*.log
```

O mag-tail habang nire-reproduce:

```bash
tail -f /tmp/openclaw/openclaw-$(date +%F).log | rg "telegram http error"
```

Para sa mga remote gateway, maaari mo ring gamitin ang `openclaw logs --follow` (tingnan ang [/cli/logs](/cli/logs)).

## Mga tala

- Kung ang `logging.level` ay nakatakda nang mas mataas kaysa `warn`, maaaring hindi maipakita ang mga log na ito. Ang default na `info` ay ayos na.
- Ligtas na iwanang naka-enable ang mga flag; naaapektuhan lang nila ang dami ng log para sa partikular na subsystem.
- Gamitin ang [/logging](/logging) para baguhin ang mga destinasyon ng log, mga level, at redaction.

