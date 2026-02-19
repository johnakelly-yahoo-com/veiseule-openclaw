---
summary: "Signal-Unterstützung über signal-cli (JSON-RPC + SSE), Einrichtung und Nummernmodell"
read_when:
  - Einrichten der Signal-Unterstützung
  - Debuggen von Signal Senden/Empfangen
title: "Signal"
---

# Signal (signal-cli)

Status: externe CLI-Integration. Das Gateway kommuniziert über HTTP JSON-RPC + SSE mit `signal-cli`.

## Voraussetzungen

- OpenClaw ist auf deinem Server installiert (Linux-Ablauf unten getestet auf Ubuntu 24).
- `signal-cli` ist auf dem Host verfügbar, auf dem das Gateway läuft.
- Eine Telefonnummer, die eine einmalige Verifizierungs-SMS empfangen kann (für den SMS-Registrierungspfad).
- Browserzugriff für das Signal-Captcha (`signalcaptchas.org`) während der Registrierung.

## Schnellstart (für Einsteiger)

1. Verwenden Sie **eine separate Signal-Nummer** für den Bot (empfohlen).
2. Installieren Sie `signal-cli` (Java erforderlich).
3. Wähle einen Einrichtungsweg:
   - `signal-cli link -n "OpenClaw"`
   - **Pfad B (SMS-Registrierung):** Registriere eine dedizierte Nummer mit Captcha + SMS-Verifizierung.
4. Konfigurieren Sie OpenClaw und starten Sie das Gateway.
5. Sende eine erste DM und bestätige das Pairing (`openclaw pairing approve signal <CODE>`).

Minimale Konfiguration:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Feldreferenz:

| Feld                                         | Beschreibung                                                                          |
| -------------------------------------------- | ------------------------------------------------------------------------------------- |
| `account`                                    | Bot-Telefonnummer im E.164-Format (`+15551234567`) |
| Einrichtung (Schnellpfad) | Pfad zu `signal-cli` (`signal-cli`, wenn im `PATH`)                |
| `dmPolicy`                                   | DM-Zugriffsrichtlinie (`pairing` empfohlen)                        |
| `allowFrom`                                  | Telefonnummern oder `uuid:&lt;id&gt;`-Werte, die DMs senden dürfen                          |

## Was es ist

- Signal-Kanal über `signal-cli` (keine eingebettete libsignal).
- Deterministisches Routing: Antworten gehen immer zurück zu Signal.
- Direktnachrichten teilen sich die Hauptsitzung des Agenten; Gruppen sind isoliert (`agent:<agentId>:signal:group:<groupId>`).

## Konfigurationsschreibzugriffe

Standardmäßig darf Signal Konfigurationsaktualisierungen schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktivieren mit:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Das Nummernmodell (wichtig)

- Das Gateway verbindet sich mit einem **Signal-Gerät** (dem `signal-cli`-Konto).
- Wenn Sie den Bot über **Ihr persönliches Signal-Konto** betreiben, ignoriert er Ihre eigenen Nachrichten (Schleifenschutz).
- Für „Ich schreibe dem Bot und er antwortet“ verwenden Sie eine **separate Bot-Nummer**.

## Setup-Pfad A: vorhandenes Signal-Konto verknüpfen (QR)

1. Installieren Sie `signal-cli` (Java erforderlich).
2. Verknüpfen Sie ein Bot-Konto:
   - `signal-cli link -n "OpenClaw"` und scannen Sie anschließend den QR-Code in Signal.
3. Konfigurieren Sie Signal und starten Sie das Gateway.

Beispiel:

```json5
{
  channels: {
    signal: {
      enabled: true,
      account: "+15551234567",
      cliPath: "signal-cli",
      dmPolicy: "pairing",
      allowFrom: ["+15557654321"],
    },
  },
}
```

Unterstützung mehrerer Konten: Verwenden Sie `channels.signal.accounts` mit konto­spezifischer Konfiguration und optional `name`. Siehe [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) für das gemeinsame Muster.

## Setup-Pfad B: dedizierte Bot-Nummer registrieren (SMS, Linux)

Verwende dies, wenn du eine dedizierte Bot-Nummer nutzen möchtest, anstatt ein bestehendes Signal-App-Konto zu verknüpfen.

1. Besorge dir eine Nummer, die SMS empfangen kann (oder Sprachverifizierung für Festnetznummern).
   - Verwende eine dedizierte Bot-Nummer, um Konto-/Sitzungskonflikte zu vermeiden.
2. Installiere `signal-cli` auf dem Gateway-Host:

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Wenn du den JVM-Build (`signal-cli-${VERSION}.tar.gz`) verwendest, installiere zuerst JRE 25+.
Halte `signal-cli` aktuell; laut Upstream können alte Releases nicht mehr funktionieren, wenn sich die Signal-Server-APIs ändern.

3. Registriere und verifiziere die Nummer:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Falls ein Captcha erforderlich ist:

1. Öffne `https://signalcaptchas.org/registration/generate.html`.
2. Schließe das Captcha ab und kopiere das `signalcaptcha://...`-Linkziel aus „Open Signal“.
3. Führe den Befehl nach Möglichkeit von derselben externen IP aus wie die Browser-Sitzung.
4. Führe die Registrierung sofort erneut aus (Captcha-Tokens laufen schnell ab):

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. Verknüpfen Sie das Bot-Gerät und starten Sie den Daemon:

```bash
# Wenn du das Gateway als systemd-User-Service ausführst:
systemctl --user restart openclaw-gateway

# Dann prüfen:
openclaw doctor
openclaw channels status --probe
```

5. Kopple deinen DM-Absender:
   - Sende eine beliebige Nachricht an die Bot-Nummer.
   - Bestätige den Code auf dem Server: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Speichere die Bot-Nummer als Kontakt auf deinem Telefon, um „Unbekannter Kontakt“ zu vermeiden.

Wichtig: Das Registrieren eines Telefonnummern-Kontos mit `signal-cli` kann die Haupt-Signal-App-Sitzung für diese Nummer abmelden. Bevorzuge eine dedizierte Bot-Nummer oder verwende den QR-Verknüpfungsmodus, wenn du deine bestehende Telefon-App-Konfiguration beibehalten möchtest.

Upstream-Referenzen:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captcha-Ablauf: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Verknüpfungsablauf: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

## Externer Daemon-Modus (httpUrl)

Wenn Sie `signal-cli` selbst verwalten möchten (langsamer JVM-Kaltstart, Container-Initialisierung oder geteilte CPUs), führen Sie den Daemon separat aus und verweisen Sie OpenClaw darauf:

```json5
{
  channels: {
    signal: {
      httpUrl: "http://127.0.0.1:8080",
      autoStart: false,
    },
  },
}
```

Dies überspringt das automatische Starten und die Start-Wartezeit innerhalb von OpenClaw. Für langsame Starts beim automatischen Starten setzen Sie `channels.signal.startupTimeoutMs`.

## Zugriffskontrolle (DMs + Gruppen)

DMs:

- Standard: `channels.signal.dmPolicy = "pairing"`.
- Unbekannte Absender erhalten einen Kopplungscode; Nachrichten werden ignoriert, bis sie freigegeben sind (Codes laufen nach 1 Stunde ab).
- Freigabe über:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Kopplung ist der Standard-Token-Austausch für Signal-DMs. Details: [Pairing](/channels/pairing)
- Absender nur mit UUID (von `sourceUuid`) werden als `uuid:<id>` in `channels.signal.allowFrom` gespeichert.

Gruppen:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` steuert, wer in Gruppen auslösen darf, wenn `allowlist` gesetzt ist.

## Funktionsweise (Verhalten)

- `signal-cli` läuft als Daemon; das Gateway liest Ereignisse über SSE.
- Eingehende Nachrichten werden in den gemeinsamen Kanal-Umschlag normalisiert.
- Antworten werden immer an dieselbe Nummer oder Gruppe zurückgeleitet.

## Medien + Limits

- Ausgehender Text wird in Blöcke bis `channels.signal.textChunkLimit` aufgeteilt (Standard 4000).
- Optionale Zeilenumbruch-Aufteilung: Setzen Sie `channels.signal.chunkMode="newline"`, um vor der Längenaufteilung an Leerzeilen (Absatzgrenzen) zu trennen.
- Anhänge werden unterstützt (Base64 aus `signal-cli` abgerufen).
- Standard-Medienlimit: `channels.signal.mediaMaxMb` (Standard 8).
- Verwenden Sie `channels.signal.ignoreAttachments`, um das Herunterladen von Medien zu überspringen.
- Gruppen-Historienkontext verwendet `channels.signal.historyLimit` (oder `channels.signal.accounts.*.historyLimit`) und fällt auf `messages.groupChat.historyLimit` zurück. Setzen Sie `0` zum Deaktivieren (Standard 50).

## Tippstatus + Lesebestätigungen

- **Tippindikatoren**: OpenClaw sendet Tipp-Signale über `signal-cli sendTyping` und aktualisiert sie, während eine Antwort läuft.
- **Lesebestätigungen**: Wenn `channels.signal.sendReadReceipts` wahr ist, leitet OpenClaw Lesebestätigungen für erlaubte DMs weiter.
- Signal-cli stellt keine Lesebestätigungen für Gruppen bereit.

## Reaktionen (Nachrichten-Werkzeug)

- Verwenden Sie `message action=react` mit `channel=signal`.
- Ziele: Absender E.164 oder UUID (verwenden Sie `uuid:<id>` aus der Kopplungsausgabe; eine nackte UUID funktioniert ebenfalls).
- `messageId` ist der Signal-Zeitstempel der Nachricht, auf die Sie reagieren.
- Gruppenreaktionen erfordern `targetAuthor` oder `targetAuthorUuid`.

Beispiele:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Konfiguration:

- `channels.signal.actions.reactions`: Reaktionsaktionen aktivieren/deaktivieren (Standard true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` deaktiviert Agentenreaktionen (das Nachrichten-Werkzeug `react` liefert einen Fehler).
  - `minimal`/`extensive` aktiviert Agentenreaktionen und legt den Leitfaden-Level fest.
- Konto­spezifische Überschreibungen: `channels.signal.accounts.<id>.actions.reactions`, `channels.signal.accounts.<id>.reactionLevel`.

## Zustellziele (CLI/cron)

- DMs: `signal:+15551234567` (oder einfache E.164).
- UUID-DMs: `uuid:<id>` (oder nackte UUID).
- Gruppen: `signal:group:<groupId>`.
- Benutzernamen: `username:<name>` (falls von Ihrem Signal-Konto unterstützt).

## Fehlerbehebung

Führen Sie zuerst diese Abfolge aus:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Bestätigen Sie dann bei Bedarf den DM-Kopplungsstatus:

```bash
openclaw pairing list signal
```

Häufige Fehler:

- Daemon erreichbar, aber keine Antworten: Überprüfen Sie Konto-/Daemon-Einstellungen (`httpUrl`, `account`) und den Empfangsmodus.
- DMs werden ignoriert: Absender wartet auf Kopplungsfreigabe.
- Gruppennachrichten werden ignoriert: Einschränkungen für Gruppenabsender/Erwähnungen blockieren die Zustellung.
- Konfigurationsvalidierungsfehler nach Änderungen: führe `openclaw doctor --fix` aus.
- Signal fehlt in der Diagnose: bestätige `channels.signal.enabled: true`.

Zusätzliche Prüfungen:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

Ablauf zur Diagnose: [/channels/troubleshooting](/channels/troubleshooting).

## Sicherheitshinweise

- `signal-cli` speichert Kontoschlüssel lokal (typischerweise `~/.local/share/signal-cli/data/`).
- Sichern Sie den Signal-Kontostatus vor einer Servermigration oder einem Neuaufbau.
- Behalten Sie `channels.signal.dmPolicy: "pairing"` bei, es sei denn, Sie möchten ausdrücklich einen breiteren DM-Zugriff.
- Die SMS-Verifizierung wird nur für Registrierungs- oder Wiederherstellungsabläufe benötigt, aber der Verlust der Kontrolle über die Nummer bzw. das Konto kann eine erneute Registrierung erschweren.

## Konfigurationsreferenz (Signal)

Vollständige Konfiguration: [Konfiguration](/gateway/configuration)

Anbieteroptionen:

- `channels.signal.enabled`: Kanalstart aktivieren/deaktivieren.
- `channels.signal.account`: E.164 für das Bot-Konto.
- `channels.signal.cliPath`: Pfad zu `signal-cli`.
- `channels.signal.httpUrl`: vollständige Daemon-URL (überschreibt Host/Port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: Daemon-Bindung (Standard 127.0.0.1:8080).
- `channels.signal.autoStart`: Daemon automatisch starten (Standard true, wenn `httpUrl` nicht gesetzt ist).
- `channels.signal.startupTimeoutMs`: Start-Wartezeit-Timeout in ms (Obergrenze 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: Download von Anhängen überspringen.
- `channels.signal.ignoreStories`: Stories vom Daemon ignorieren.
- `channels.signal.sendReadReceipts`: Lesebestätigungen weiterleiten.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: Kopplung).
- `channels.signal.allowFrom`: DM-Allowlist (E.164 oder `uuid:<id>`). `open` erfordert `"*"`. Signal hat keine Benutzernamen; verwenden Sie Telefon-/UUID-IDs.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (Standard: Allowlist).
- `channels.signal.groupAllowFrom`: Allowlist für Gruppenabsender.
- `channels.signal.historyLimit`: maximale Anzahl an Gruppennachrichten, die als Kontext einbezogen werden (0 deaktiviert).
- `channels.signal.dmHistoryLimit`: DM-Historienlimit in Benutzerzügen. Benutzer­spezifische Überschreibungen: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: Größe der ausgehenden Textblöcke (Zeichen).
- `channels.signal.chunkMode`: `length` (Standard) oder `newline`, um vor der Längenaufteilung an Leerzeilen (Absatzgrenzen) zu trennen.
- `channels.signal.mediaMaxMb`: Medienlimit für ein- und ausgehende Inhalte (MB).

Zugehörige globale Optionen:

- `agents.list[].groupChat.mentionPatterns` (Signal unterstützt keine nativen Erwähnungen).
- `messages.groupChat.mentionPatterns` (globaler Fallback).
- `messages.responsePrefix`.

