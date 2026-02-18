---
title: "doctor"
---

# `openclaw doctor`

Sundhedstjek + hurtige rettelser for gateway og kanaler.

Relateret:

- Fejlfinding: [Troubleshooting](/gateway/troubleshooting)
- Sikkerhedsaudit: [Security](/gateway/security)

## Eksempler

```bash
openclaw doctor
openclaw doctor --repair
openclaw doctor --deep
```

Noter:

- Interaktive prompts (som keychain/OAuth fixes) kører kun, når stdin er en TTY og `--non-interactive` er **ikke** sæt. Hovedløse kørsler (cron, Telegram, ingen terminal) vil springe prompter.
- `--fix` (alias for `--repair`) skriver en backup til `~/.openclaw/openclaw.json.bak` og fjerner ukendte konfigurationsnøgler, hvor hver fjernelse listes.

## macOS: `launchctl` miljøoverstyringer

Hvis du tidligere kørte `launchctl setenv OPENCLAW_GATEWAY_TOKEN ...` (eller `...PASSWORD`), overstyrer den værdi din konfigurationsfil og kan forårsage vedvarende “unauthorized”-fejl.

```bash
launchctl getenv OPENCLAW_GATEWAY_TOKEN
launchctl getenv OPENCLAW_GATEWAY_PASSWORD

launchctl unsetenv OPENCLAW_GATEWAY_TOKEN
launchctl unsetenv OPENCLAW_GATEWAY_PASSWORD
```

