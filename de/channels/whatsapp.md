---
summary: "WhatsApp-Channel-Unterstützung, Zugriffskontrollen, Zustellverhalten und Betrieb"
read_when:
  - Arbeit am Verhalten des WhatsApp/Web-Kanals oder an der Inbox-Routinglogik
title: "WhatsApp"
---

# WhatsApp (Web-Kanal)

Status: Nur WhatsApp Web über Baileys. Das Gateway besitzt die Sitzung(en).

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Standardmäßige DM-Richtlinie ist Kopplung für unbekannte Absender.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanalübergreifende Diagnosen und Reparaturleitfäden.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Vollständige Channel-Config-Muster und Beispiele.
  
</Card>
</CardGroup>

## Schnellstart

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Für ein bestimmtes Konto:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Genehmigen mit: `openclaw pairing approve whatsapp <code>` (Liste mit `openclaw pairing list whatsapp`).
```

    ```
    Codes laufen nach 1 Stunde ab; ausstehende Anfragen sind auf 3 pro Kanal begrenzt.
    ```

  
</Step>
</Steps>

<Note>
Ideal, um Ihr persönliches WhatsApp getrennt zu halten — installieren Sie WhatsApp Business und registrieren Sie dort die OpenClaw-Nummer. (Die Channel-Metadaten und der Onboarding-Ablauf sind für dieses Setup optimiert, aber Setups mit persönlicher Nummer werden ebenfalls unterstützt.)
</Note>

## Bereitstellungsmuster

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Dies ist der sauberste Betriebsmodus:


    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Das Onboarding unterstützt den Modus mit persönlicher Nummer und schreibt eine selbstchat-freundliche Baseline:


    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">Der Messaging-Plattform-Channel ist WhatsApp-Web-basiert (`Baileys`) in der aktuellen OpenClaw-Channel-Architektur.

    ```
    Es gibt keinen separaten Twilio-WhatsApp-Messaging-Channel im integrierten Chat-Channel-Register.
    ```

  
</Accordion>
</AccordionGroup>

## Laufzeitmodell

- Gateway besitzt den WhatsApp-Socket und die Reconnect-Schleife.
- Ausgehende Nachrichten erfordern einen aktiven WhatsApp-Listener für das Zielkonto.
- Status-/Broadcast-Chats werden ignoriert.
- Direktchats verwenden DM-Sitzungsregeln (`session.dmScope`; Standard `main` fasst DMs in der Hauptsitzung des Agenten zusammen).
- Gruppen werden auf `agent:<agentId>:whatsapp:group:<jid>`-Sitzungen abgebildet.

## Zugriffskontrolle und Aktivierung

<Tabs>
  <Tab title="DM policy">**DM-Richtlinie**: `channels.whatsapp.dmPolicy` steuert den Zugriff auf Direktchats (Standard: `pairing`).

    ```
    Offen: erfordert, dass `channels.whatsapp.allowFrom` `"*"` enthält.
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Der Gruppenzugriff hat zwei Ebenen:

    ```
    `channels.whatsapp.groups` (Gruppen-Allowlist + Mention-Gating-Defaults; verwenden Sie `"*"`, um alle zu erlauben)
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Gruppenantworten erfordern standardmäßig eine Erwähnung.

    ```
    Erwähnungserkennung umfasst:
    
    - explizite WhatsApp-Erwähnungen der Bot-Identität
    - konfigurierte Mention-Regex-Muster (`agents.list[].groupChat.mentionPatterns`, Fallback `messages.groupChat.mentionPatterns`)
    - implizite Antwort-an-Bot-Erkennung (Antwortsender entspricht der Bot-Identität)
    
    Aktivierungsbefehl auf Sitzungsebene:
    
    - `/activation mention`
    - `/activation always`
    
    `activation` aktualisiert den Sitzungsstatus (keine globale Konfiguration). Es ist auf den Owner beschränkt.
    ```

  
</Tab>
</Tabs>

## Verhalten bei persönlicher Nummer und Selbst-Chat

Wenn die verknüpfte eigene Nummer auch in `allowFrom` enthalten ist, werden WhatsApp-Schutzmechanismen für Selbst-Chats aktiviert:

- Lesebestätigungen für Selbst-Chat-Nachrichten überspringen
- Automatisches Auslösen durch Mention-JID ignorieren, das dich sonst selbst anpingen würde
- Self-Chat-Antworten verwenden standardmäßig `[{identity.name}]`, wenn gesetzt (ansonsten `[openclaw]`),  
  falls `messages.responsePrefix` nicht gesetzt ist.

## Nachrichtennormalisierung und Kontext

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">
    Eingehende WhatsApp-Nachrichten werden in das gemeinsame Inbound-Envelope-Format verpackt.

    ````
    Wenn eine zitierte Antwort vorhanden ist, wird der Kontext in folgender Form angehängt:
    
    ```text
    [Replying to <sender> id:<stanzaId>]
    <quoted body or media placeholder>
    [/Replying]
    ```
    
    Metadatenfelder für Antworten werden ebenfalls gesetzt, sofern verfügbar (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, Sender-JID/E.164).
    ````

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Eingehende Nachrichten nur mit Medien verwenden Platzhalter:

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Standort- und Kontakt-Payloads werden vor dem Routing in textuellen Kontext normalisiert.
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">
    Bei Gruppen können unverarbeitete Nachrichten gepuffert und als Kontext eingefügt werden, wenn der Bot schließlich ausgelöst wird.

    ```
    Aktuelle _unverarbeitete_ Nachrichten (Standard 50) werden eingefügt unter:
    `[Chat messages since your last reply - for context]` (Nachrichten, die bereits in der Sitzung sind, werden nicht erneut injiziert)
    ```

  
</Accordion>

  <Accordion title="Read receipts">
    Lesebestätigungen sind standardmäßig für akzeptierte eingehende WhatsApp-Nachrichten aktiviert.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## Zustellung, Chunking und Medien

<AccordionGroup>
  <Accordion title="Text chunking">Optionale Newline-Chunking: Setzen Sie `channels.whatsapp.chunkMode="newline"`, um vor dem Längen-Chunking an Leerzeilen (Absatzgrenzen) zu trennen.
</Accordion>

  <Accordion title="Outbound media behavior">
    - unterstützt Bild-, Video-, Audio- (PTT-Sprachnotiz) und Dokument-Payloads
    - `audio/ogg` wird zu `audio/ogg; codecs=opus` umgeschrieben für Sprachnotiz-Kompatibilität
    - animierte GIF-Wiedergabe wird über `gifPlayback: true` beim Senden von Videos unterstützt
    - Bildunterschriften werden auf das erste Medienelement angewendet, wenn Multi-Media-Antworten gesendet werden
    - Medienquelle kann HTTP(S), `file://` oder ein lokaler Pfad sein
  
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - Speicherlimit für eingehende Medien: `channels.whatsapp.mediaMaxMb` (Standard `50`)
    - Medienlimit für ausgehende Auto-Antworten: `agents.defaults.mediaMaxMb` (Standard `5MB`)
    - Bilder werden automatisch optimiert (Größen-/Qualitätsanpassung), um die Limits einzuhalten
    - bei Fehlern beim Medienversand wird für das erste Element ersatzweise eine Textwarnung gesendet, anstatt die Antwort stillschweigend zu verwerfen
  
</Accordion>
</AccordionGroup>

## Bestätigungsreaktionen

`channels.whatsapp.ackReaction` (Auto-Reaktion beim Nachrichteneingang: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

Verhalten:

- werden unmittelbar nach Annahme des Eingangs gesendet (vor der Antwort)
- Fehler werden protokolliert, blockieren jedoch nicht die normale Antwortzustellung
- Im Gruppenmodus `mentions` wird bei durch Erwähnung ausgelösten Nachrichten reagiert; Gruppenaktivierung `always` dient als Umgehung dieser Prüfung
- WhatsApp ignoriert `messages.ackReaction`; verwenden Sie stattdessen `channels.whatsapp.ackReaction`.

## Multi-Account und Anmeldedaten

<AccordionGroup>
  <Accordion title="Account selection and defaults">Standardkonto (wenn `--account` weggelassen wird): `default`, falls vorhanden, sonst die erste konfigurierte Konto-ID (sortiert).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">Anmeldedaten gespeichert in `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.<accountId> /creds.json`
    - Backup-Datei: `creds.json.bak`
    - Legacy-Standardauthentifizierung in `~/.openclaw/credentials/` wird für Default-Account-Flows weiterhin erkannt/migriert
  
</Accordion>

  <Accordion title="Logout behavior">Multi-Account-Login: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>]` löscht den WhatsApp-Authentifizierungsstatus für dieses Konto.

    ```
    In Legacy-Authentifizierungsverzeichnissen bleibt `oauth.json` erhalten, während Baileys-Auth-Dateien entfernt werden.
    ```

  
</Accordion>
</AccordionGroup>

## Tools, Aktionen und Konfigurationsschreibvorgänge

- Werkzeug: `whatsapp` mit Aktion `react` (`chatJid`, `messageId`, `emoji`, optional `remove`).
- Aktions-Gates:
  - `channels.whatsapp.actions.reactions` (Gating für WhatsApp-Werkzeugreaktionen).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Symptom: `channels status` zeigt `linked: false` oder warnt „Not linked“.

    ```
    Logout: `openclaw channels logout` (oder `--account <id>`) löscht den WhatsApp-Auth-Status (behält jedoch gemeinsame `oauth.json`).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Symptom: verknüpftes Konto mit wiederholten Verbindungsabbrüchen oder erneuten Verbindungsversuchen.

    ```
    Lösung: `openclaw doctor` (oder Gateway neu starten). Wenn es anhält, erneut verknüpfen über `channels login` und `openclaw logs --follow` prüfen.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">
    Ausgehende Sendungen schlagen sofort fehl, wenn kein aktiver Gateway-Listener für das Zielkonto existiert.

    ```
    Stelle sicher, dass das Gateway läuft und das Konto verknüpft ist.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Prüfe in dieser Reihenfolge:

    ```
    Gruppenrichtlinie: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (Standard `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    Die WhatsApp-Gateway-Laufzeit sollte Node verwenden. WhatsApp (Baileys) und Telegram sind unter Bun unzuverlässig. Führen Sie das Gateway mit **Node** aus.
  
</Accordion>
</AccordionGroup>

## Konfigurationsreferenzen

Primäre Referenz:

- Standardmäßig darf WhatsApp Konfigurationsupdates schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

WhatsApp-Felder mit hoher Relevanz:

- Zugriff: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- Zustellung: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- Multi-Account: `accounts.<id>``.enabled`, `accounts.<id>``.authDir`, Überschreibungen auf Kontoebene
- Betrieb: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- Sitzungsverhalten: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>``messages.groupChat.historyLimit`

## Verwandt

- [Pairing](/channels/pairing)
- [Channel routing](/channels/channel-routing)
- Leitfaden zur Fehlerbehebung: [Gateway troubleshooting](/gateway/troubleshooting).

