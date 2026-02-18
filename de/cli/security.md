---
title: "Sicherheit"
---

# `openclaw security`

Sicherheitswerkzeuge (Audit + optionale Fixes).

Verwandt:

- Sicherheitsleitfaden: [Sicherheit](/gateway/security)

## Überprüfung

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Das Audit warnt, wenn mehrere Absender von Direktnachrichten die Hauptsitzung teilen, und empfiehlt den **sicheren DM-Modus**: `session.dmScope="per-channel-peer"` (oder `per-account-channel-peer` für Mehrkonten-Kanäle) für gemeinsam genutzte Posteingänge.
Es warnt außerdem, wenn kleine Modelle (`<=300B`) ohne sandboxing und mit aktivierten Web-/Browser-Werkzeugen verwendet werden.


