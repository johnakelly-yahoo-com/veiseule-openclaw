---
title: "Kopplung"
---

# Kopplung

„Pairing“ ist der explizite Schritt zur **Genehmigung durch den Eigentümer** in OpenClaw.
Er wird an zwei Stellen verwendet:

1. **DM-Pairing** (wer mit dem Bot sprechen darf)
2. **Node-Pairing** (welche Geräte/Nodes dem Gateway-Netzwerk beitreten dürfen)

Sicherheitskontext: [Security](/gateway/security)

## 1. DM-Pairing (eingehender Chat-Zugriff)

Wenn ein Kanal mit der DM-Richtlinie `pairing` konfiguriert ist, erhalten unbekannte Absender einen Kurzcode und ihre Nachricht wird **nicht verarbeitet**, bis Sie genehmigen.

Standard-DM-Richtlinien sind dokumentiert unter: [Security](/gateway/security)

Pairing-Codes:

- 8 Zeichen, Großbuchstaben, keine mehrdeutigen Zeichen (`0O1I`).
- **Laufen nach 1 Stunde ab**. Der Bot sendet die Pairing-Nachricht nur, wenn eine neue Anfrage erstellt wird (ungefähr einmal pro Stunde und Absender).
- Ausstehende DM-Pairing-Anfragen sind standardmäßig auf **3 pro Kanal** begrenzt; zusätzliche Anfragen werden ignoriert, bis eine abläuft oder genehmigt wird.

### Absender genehmigen

```bash
openclaw pairing list telegram
openclaw pairing approve telegram <CODE>
```

Unterstützte Kanäle: `telegram`, `whatsapp`, `signal`, `imessage`, `discord`, `slack`.

### Wo der Status gespeichert wird

Gespeichert unter `~/.openclaw/credentials/`:

- Ausstehende Anfragen: `<channel>-pairing.json`
- Genehmigte Allowlist-Speicher: `<channel>-allowFrom.json`

Behandeln Sie diese Daten als sensibel (sie steuern den Zugriff auf Ihren Assistenten).

## 2. Node-Geräte-Pairing (iOS-/Android-/macOS-/headless Nodes)

Nodes verbinden sich als **Geräte** mit dem Gateway mit `role: node`. Das Gateway
erstellt eine Geräte-Pairing-Anfrage, die genehmigt werden muss.

### Kopplung über Telegram (empfohlen für iOS)

Wenn du das `device-pair`-Plugin verwendest, kannst du die erstmalige Gerätekopplung vollständig über Telegram durchführen:

1. Sende deinem Bot in Telegram folgende Nachricht: `/pair`
2. Der Bot antwortet mit zwei Nachrichten: einer Anleitungsnachricht und einer separaten **Einrichtungscode**-Nachricht (in Telegram einfach zu kopieren/einzufügen).
3. Öffne auf deinem Smartphone die OpenClaw iOS-App → Einstellungen → Gateway.
4. Füge den Einrichtungscode ein und stelle die Verbindung her.
5. Zurück in Telegram: `/pair approve`

Der Einrichtungscode ist eine base64-kodierte JSON-Nutzlast, die Folgendes enthält:

- `url`: the Gateway WebSocket URL (`ws://...` or `wss://...`)
- `token`: a short-lived pairing token

Treat the setup code like a password while it is valid.

### Node-Gerät genehmigen

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

### Speicherung des Node-Pairing-Status

Gespeichert unter `~/.openclaw/devices/`:

- `pending.json` (kurzlebig; ausstehende Anfragen laufen ab)
- `paired.json` (gepaarte Geräte + Tokens)

### Hinweise

- Die veraltete `node.pair.*`-API (CLI: `openclaw nodes pending/approve`) ist ein
  separates, gateway-eigenes Pairing-Repository. WS-Nodes erfordern weiterhin Geräte-Pairing.

## Verwandte Dokumente

- Sicherheitsmodell + Prompt Injection: [Security](/gateway/security)
- Sicher aktualisieren (Doctor ausführen): [Updating](/install/updating)
- Kanal-Konfigurationen:
  - Telegram: [Telegram](/channels/telegram)
  - WhatsApp: [WhatsApp](/channels/whatsapp)
  - Signal: [Signal](/channels/signal)
  - BlueBubbles (iMessage): [BlueBubbles](/channels/bluebubbles)
  - iMessage (legacy): [iMessage](/channels/imessage)
  - Discord: [Discord](/channels/discord)
  - Slack: [Slack](/channels/slack)


