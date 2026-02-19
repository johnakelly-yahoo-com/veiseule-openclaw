---
summary: "CLI-Referenz für `openclaw security` (Audit und Behebung gängiger Sicherheits-Footguns)"
read_when:
  - Sie möchten ein schnelles Sicherheits-Audit von Konfiguration/Zustand durchführen
  - Sie möchten sichere „Fix“-Vorschläge anwenden (chmod, Standardwerte verschärfen)
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
Für Webhook-Ingress wird gewarnt, wenn `hooks.defaultSessionKey` nicht gesetzt ist, wenn Request-`sessionKey`-Overrides aktiviert sind und wenn Overrides ohne `hooks.allowedSessionKeyPrefixes` aktiviert sind.
Es wird außerdem gewarnt, wenn Sandbox-Docker-Einstellungen konfiguriert sind, während der Sandbox-Modus deaktiviert ist, wenn `gateway.nodes.denyCommands` ineffektive musterähnliche oder unbekannte Einträge verwendet, wenn das globale `tools.profile="minimal"` durch Agent-Tool-Profile überschrieben wird und wenn installierte Extension-Plugin-Tools unter einer permissiven Tool-Policy erreichbar sein könnten.
