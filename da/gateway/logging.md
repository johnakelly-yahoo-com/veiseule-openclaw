---
summary: "Logningsflader, fillogs, WS-logstile og konsolformatering"
read_when:
  - Ændring af logningsoutput eller -formater
  - Fejlfinding af CLI- eller gateway-output
title: "Logning"
---

# Logning

For et brugerrettet overblik (CLI + Control UI + konfiguration), se [/logging](/logging).

OpenClaw har to log-“flader”:

- **Konsoloutput** (det, du ser i terminalen / Debug UI).
- **Fillogs** (JSON-linjer) skrevet af gateway-loggeren.

## Filbaseret logger

- Standard roterende logfil ligger under `/tmp/openclaw/` (én fil pr. dag): `openclaw-YYYY-MM-DD.log`
  - Datoen bruger gateway-værtens lokale tidszone.
- Logfilens sti og niveau kan konfigureres via `~/.openclaw/openclaw.json`:
  - `logging.file`
  - `logging.level`

Filformatet er ét JSON-objekt pr. linje.

Kontrol-UI logger fanen haler denne fil via gateway (`logs.tail`).
CLI kan gøre det samme:

```bash
openclaw logs --follow
```

**Verbose vs. logniveauer**

- **Fillogs** styres udelukkende af `logging.level`.
- `--verbose` påvirker kun **konsolens verbositet** (og WS-logstil); det hæver **ikke**
  fil-logniveauet.
- For at indfange detaljer, der kun findes i verbose, i fillogs, skal du sætte `logging.level` til `debug` eller
  `trace`.

## Konsolopsamling

CLI’en opsamler `console.log/info/warn/error/debug/trace` og skriver dem til fillogs,
mens der stadig udskrives til stdout/stderr.

Du kan justere konsolens verbositet uafhængigt via:

- `logging.consoleLevel` (standard `info`)
- `logging.consoleStyle` (`pretty` | `compact` | `json`)

## Redigering af værktøjsresumeer

Verbose værktøj resuméer (f.eks. `🛠️ Exec: ...`) kan maskere følsomme tokens før de rammer
konsollen stream. Dette er **tools-only** og ændrer ikke fillogs.

- `logging.redactSensitive`: `off` | `tools` (standard: `tools`)
- `logging.redactPatterns`: array af regex-strenge (tilsidesætter standarder)
  - Brug rå regex-strenge (auto `gi`), eller `/pattern/flags` hvis du har brug for brugerdefinerede flag.
  - Matches maskeres ved at bevare de første 6 + sidste 4 tegn (længde >= 18), ellers `***`.
  - Standarder dækker almindelige nøgle-tildelinger, CLI-flag, JSON-felter, bearer-headere, PEM-blokke og populære token-præfikser.

## Gateway WebSocket-logs

Gatewayen udskriver WebSocket-protokollogs i to tilstande:

- **Normal tilstand (ingen `--verbose`)**: kun “interessante” RPC-resultater udskrives:
  - fejl (`ok=false`)
  - langsomme kald (standardtærskel: `>= 50ms`)
  - parse-fejl
- **Verbose tilstand (`--verbose`)**: udskriver al WS request/response-trafik.

### WS-logstil

`openclaw gateway` understøtter et stilskift pr. gateway:

- `--ws-log auto` (standard): normal tilstand er optimeret; verbose tilstand bruger kompakt output
- `--ws-log compact`: kompakt output (parret request/response) i verbose
- `--ws-log full`: fuldt per-frame-output i verbose
- `--compact`: alias for `--ws-log compact`

Eksempler:

```bash
# optimized (only errors/slow)
openclaw gateway

# show all WS traffic (paired)
openclaw gateway --verbose --ws-log compact

# show all WS traffic (full meta)
openclaw gateway --verbose --ws-log full
```

## Konsolformatering (undersystem-logning)

Konsolformatteren er **TTY-aware** og udskriver konsistente, præfikserede linjer.
Delsystemloggere holder output grupperet og scannbar.

Adfærd:

- **Præfikser** på hver linje (f.eks. `[gateway]`, `[canvas]`, `[tailscale]`)
- **Undersystemfarver** (stabile pr. undersystem) plus niveaufarver
- **Farver når output er en TTY, eller miljøet ligner en rig terminal** (`TERM`/`COLORTERM`/`TERM_PROGRAM`), respekterer `NO_COLOR`
- **Forkortede præfikser for delsystemer**: dråber ledende `gateway/` + `kanaler/`, holder de sidste 2 segmenter (f.eks. `whatsapp/outbound`)
- **Underloggere pr. undersystem** (automatisk præfiks + struktureret felt `{ subsystem }`)
- **`logRaw()`** til QR/UX-output (ingen præfiks, ingen formatering)
- **Konsolstil** (f.eks.`smuk autentisk kompakt autentisk json`)
- **Konsollogniveau** adskilt fra fillogniveau (filen bevarer fuld detalje, når `logging.level` er sat til `debug`/`trace`)
- **WhatsApp-meddelelsesindhold** logges ved `debug` (brug `--verbose` for at se dem)

Dette holder eksisterende fillogs stabile, mens interaktivt output bliver let at skimme.

