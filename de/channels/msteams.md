---
summary: "Supportstatus, Fähigkeiten und Konfiguration des Microsoft-Teams-Bots"
read_when:
  - Arbeit an MS-Teams-Kanalfunktionen
title: "Microsoft Teams"
---

# Microsoft Teams (Plugin)

> „Lasst alle Hoffnung fahren, ihr, die ihr hier eintretet.“

Aktualisiert: 2026-01-21

Status: Text + DM-Anhänge werden unterstützt; Dateiübertragung in Kanälen/Gruppen erfordert `sharePointSiteId` + Graph-Berechtigungen (siehe [Dateien in Gruppenchats senden](#sending-files-in-group-chats)). Umfragen werden über Adaptive Cards gesendet.

## Erforderliches Plugin

Microsoft Teams wird als Plugin ausgeliefert und ist nicht im Core-Install enthalten.

**Breaking Change (2026.1.15):** MS Teams wurde aus dem Core ausgelagert. Wenn Sie es verwenden, müssen Sie das Plugin installieren.

Begründung: Hält Core-Installationen schlanker und ermöglicht unabhängige Updates der MS-Teams-Abhängigkeiten.

Installation via CLI (npm-Registry):

```bash
openclaw plugins install @openclaw/msteams
```

Lokaler Checkout (bei Ausführung aus einem Git-Repo):

```bash
openclaw plugins install ./extensions/msteams
```

Wenn Sie Teams während Konfiguration/Onboarding auswählen und ein Git-Checkout erkannt wird,
bietet OpenClaw den lokalen Installationspfad automatisch an.

Details: [Plugins](/tools/plugin)

## Schnellstart (Anfänger)

1. Installieren Sie das Microsoft-Teams-Plugin.
2. Erstellen Sie einen **Azure Bot** (App-ID + Client Secret + Tenant-ID).
3. Konfigurieren Sie OpenClaw mit diesen Zugangsdaten.
4. Exponieren Sie `/api/messages` (Standardport 3978) über eine öffentliche URL oder einen Tunnel.
5. Installieren Sie das Teams-App-Paket und starten Sie das Gateway.

Minimale Konfiguration:

```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

Hinweis: Gruppenchats sind standardmäßig blockiert (`channels.msteams.groupPolicy: "allowlist"`). Um Gruppenantworten zu erlauben, setzen Sie `channels.msteams.groupAllowFrom` (oder verwenden Sie `groupPolicy: "open"`, um jedes Mitglied zuzulassen, erwähnungsbasiert).

## Ziele

- Mit OpenClaw über Teams-DMs, Gruppenchats oder Kanäle sprechen.
- Deterministisches Routing beibehalten: Antworten gehen immer an den Kanal zurück, aus dem sie kamen.
- Standardmäßig sicheres Kanalverhalten (Erwähnungen erforderlich, sofern nicht anders konfiguriert).

## Konfigurationsschreibzugriffe

Standardmäßig darf Microsoft Teams Konfigurationsupdates schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktivieren mit:

```json5
{
  channels: { msteams: { configWrites: false } },
}
```

## Zugriffskontrolle (DMs + Gruppen)

**DM-Zugriff**

- Standard: `channels.msteams.dmPolicy = "pairing"`. Unbekannte Absender werden ignoriert, bis sie genehmigt sind.
- `channels.msteams.allowFrom` akzeptiert AAD-Objekt-IDs, UPNs oder Anzeigenamen. Der Assistent löst Namen bei ausreichenden Berechtigungen über Microsoft Graph in IDs auf.

**Gruppenzugriff**

- Standard: `channels.msteams.groupPolicy = "allowlist"` (blockiert, sofern Sie nicht `groupAllowFrom` hinzufügen). Verwenden Sie `channels.defaults.groupPolicy`, um den Standard zu überschreiben, wenn nicht gesetzt.
- `channels.msteams.groupAllowFrom` steuert, welche Absender in Gruppenchats/Kanälen auslösen dürfen (Fallback auf `channels.msteams.allowFrom`).
- Setzen Sie `groupPolicy: "open"`, um jedes Mitglied zuzulassen (standardmäßig weiterhin erwähnungsbasiert).
- Um **keine Kanäle** zuzulassen, setzen Sie `channels.msteams.groupPolicy: "disabled"`.

Beispiel:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams- und Kanal-Allowlist**

- Begrenzen Sie Gruppen-/Kanalantworten, indem Sie Teams und Kanäle unter `channels.msteams.teams` auflisten.
- Schlüssel können Team-IDs oder -Namen sein; Kanalschlüssel können Konversations-IDs oder -Namen sein.
- Wenn `groupPolicy="allowlist"` gesetzt ist und eine Teams-Allowlist vorhanden ist, werden nur gelistete Teams/Kanäle akzeptiert (erwähnungsbasiert).
- Der Konfigurationsassistent akzeptiert `Team/Channel`-Einträge und speichert sie für Sie.
- Beim Start löst OpenClaw Team-/Kanal- und Benutzer-Allowlist-Namen in IDs auf (wenn Graph-Berechtigungen vorhanden sind)
  und protokolliert die Zuordnung; nicht auflösbare Einträge bleiben unverändert.

Beispiel:

```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## Wie es funktioniert

1. Installieren Sie das Microsoft-Teams-Plugin.
2. Erstellen Sie einen **Azure Bot** (App-ID + Secret + Tenant-ID).
3. Erstellen Sie ein **Teams-App-Paket**, das auf den Bot verweist und die unten aufgeführten RSC-Berechtigungen enthält.
4. Laden/installieren Sie die Teams-App in ein Team (oder im persönlichen Bereich für DMs).
5. Konfigurieren Sie `msteams` in `~/.openclaw/openclaw.json` (oder per Umgebungsvariablen) und starten Sie das Gateway.
6. Das Gateway lauscht standardmäßig auf Bot-Framework-Webhook-Traffic unter `/api/messages`.

## Azure Bot Setup (Voraussetzungen)

Bevor Sie OpenClaw konfigurieren, müssen Sie eine Azure-Bot-Ressource erstellen.

### Schritt 1: Azure Bot erstellen

1. Gehen Sie zu [Azure Bot erstellen](https://portal.azure.com/#create/Microsoft.AzureBot)
2. Füllen Sie den Reiter **Basics** aus:

   | Feld                 | Wert                                                                                                           |
   | -------------------- | -------------------------------------------------------------------------------------------------------------- |
   | **Bot-Name**         | Ihr Botname, z. B. `openclaw-msteams` (muss eindeutig sein) |
   | **Abonnement**       | Wählen Sie Ihr Azure-Abonnement                                                                                |
   | **Ressourcengruppe** | Neu erstellen oder bestehende verwenden                                                                        |
   | **Tarifstufe**       | **Free** für Dev/Test                                                                                          |
   | **App-Typ**          | **Single Tenant** (empfohlen – siehe Hinweis unten)                                         |
   | **Erstellungstyp**   | **Neue Microsoft App ID erstellen**                                                                            |

> **Hinweis zur Abkündigung:** Die Erstellung neuer Multi-Tenant-Bots wurde nach dem 2025-07-31 eingestellt. Verwenden Sie **Single Tenant** für neue Bots.

3. Klicken Sie auf **Review + create** → **Create** (Wartezeit ~1–2 Minuten)

### Schritt 2: Zugangsdaten abrufen

1. Gehen Sie zu Ihrer Azure-Bot-Ressource → **Configuration**
2. Kopieren Sie **Microsoft App ID** → das ist Ihre `appId`
3. Klicken Sie auf **Manage Password** → zur App-Registrierung
4. Unter **Certificates & secrets** → **New client secret** → kopieren Sie den **Value** → das ist Ihre `appPassword`
5. Gehen Sie zu **Overview** → kopieren Sie **Directory (tenant) ID** → das ist Ihre `tenantId`

### Schritt 3: Messaging Endpoint konfigurieren

1. In Azure Bot → **Konfiguration**
2. Setzen Sie **Messaging endpoint** auf Ihre Webhook-URL:
   - Produktion: `https://your-domain.com/api/messages`
   - Lokale Entwicklung: Verwenden Sie einen Tunnel (siehe [Lokale Entwicklung](#local-development-tunneling) unten)

### Schritt 4: Teams-Kanal aktivieren

1. In Azure Bot → **Channels**
2. Klicken Sie auf **Microsoft Teams** → Configure → Save
3. Akzeptieren Sie die Nutzungsbedingungen

## Lokale Entwicklung (Tunneling)

Teams kann `localhost` nicht erreichen. Verwenden Sie für die lokale Entwicklung einen Tunnel:

**Option A: ngrok**

```bash
ngrok http 3978
# Copy the https URL, e.g., https://abc123.ngrok.io
# Set messaging endpoint to: https://abc123.ngrok.io/api/messages
```

**Option B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Use your Tailscale funnel URL as the messaging endpoint
```

## Teams Developer Portal (Alternative)

Statt ein Manifest-ZIP manuell zu erstellen, können Sie das [Teams Developer Portal](https://dev.teams.microsoft.com/apps) verwenden:

1. Klicken Sie auf **+ New app**
2. Füllen Sie die Basisinformationen aus (Name, Beschreibung, Entwicklerinfos)
3. Gehen Sie zu **App features** → **Bot**
4. Wählen Sie **Enter a bot ID manually** und fügen Sie Ihre Azure-Bot-App-ID ein
5. Aktivieren Sie die Scopes: **Personal**, **Team**, **Group Chat**
6. Klicken Sie auf **Distribute** → **Download app package**
7. In Teams: **Apps** → **Manage your apps** → **Upload a custom app** → ZIP auswählen

Dies ist oft einfacher als das manuelle Bearbeiten von JSON-Manifests.

## Bot testen

**Option A: Azure Web Chat (Webhook zuerst verifizieren)**

1. Im Azure-Portal → Ihre Azure-Bot-Ressource → **Test in Web Chat**
2. Senden Sie eine Nachricht – Sie sollten eine Antwort sehen
3. Dies bestätigt, dass Ihr Webhook-Endpunkt funktioniert, bevor Teams eingerichtet wird

**Option B: Teams (nach App-Installation)**

1. Installieren Sie die Teams-App (Sideload oder Org-Katalog)
2. Finden Sie den Bot in Teams und senden Sie eine DM
3. Prüfen Sie die Gateway-Logs auf eingehende Aktivitäten

## Setup (minimal, nur Text)

1. **Microsoft-Teams-Plugin installieren**
   - Aus npm: `openclaw plugins install @openclaw/msteams`
   - Aus lokalem Checkout: `openclaw plugins install ./extensions/msteams`

2. **Bot-Registrierung**
   - Erstellen Sie einen Azure Bot (siehe oben) und notieren Sie:
     - App-ID
     - Client Secret (App-Passwort)
     - Tenant-ID (Single Tenant)

3. **Teams-App-Manifest**
   - Fügen Sie einen `bot`-Eintrag mit `botId = <App ID>` hinzu.
   - Scopes: `personal`, `team`, `groupChat`.
   - `supportsFiles: true` (erforderlich für Dateiverarbeitung im persönlichen Bereich).
   - RSC-Berechtigungen hinzufügen (siehe unten).
   - Icons erstellen: `outline.png` (32×32) und `color.png` (192×192).
   - Alle drei Dateien zusammen zippen: `manifest.json`, `outline.png`, `color.png`.

4. **OpenClaw konfigurieren**

   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   Sie können statt Konfigurationsschlüsseln auch Umgebungsvariablen verwenden:

   - `MSTEAMS_APP_ID`
   - `MSTEAMS_APP_PASSWORD`
   - `MSTEAMS_TENANT_ID`

5. **Bot-Endpunkt**
   - Setzen Sie den Azure-Bot-Messaging-Endpunkt auf:
     - `https://<host>:3978/api/messages` (oder Ihren gewählten Pfad/Port).

6. **Gateway starten**
   - Der Teams-Kanal startet automatisch, wenn das Plugin installiert ist und eine `msteams`-Konfiguration mit Zugangsdaten existiert.

## Verlaufskontext

- `channels.msteams.historyLimit` steuert, wie viele aktuelle Kanal-/Gruppennachrichten in den Prompt aufgenommen werden.
- Fallback auf `messages.groupChat.historyLimit`. Setzen Sie `0`, um zu deaktivieren (Standard 50).
- DM-Verlauf kann mit `channels.msteams.dmHistoryLimit` (Benutzer-Turns) begrenzt werden. Pro-Benutzer-Overrides: `channels.msteams.dms["<user_id>"].historyLimit`.

## Aktuelle Teams-RSC-Berechtigungen (Manifest)

Dies sind die **vorhandenen resourceSpecific permissions** in unserem Teams-App-Manifest. Sie gelten nur innerhalb des Teams/Chats, in dem die App installiert ist.

**Für Kanäle (Team-Scope):**

- `ChannelMessage.Read.Group` (Application) – alle Kanalnachrichten ohne @Erwähnung empfangen
- `ChannelMessage.Send.Group` (Application)
- `Member.Read.Group` (Application)
- `Owner.Read.Group` (Application)
- `ChannelSettings.Read.Group` (Application)
- `TeamMember.Read.Group` (Application)
- `TeamSettings.Read.Group` (Application)

**Für Gruppenchats:**

- `ChatMessage.Read.Chat` (Application) – alle Gruppenchats ohne @Erwähnung empfangen

## Beispiel-Teams-Manifest (redigiert)

Minimales, gültiges Beispiel mit den erforderlichen Feldern. Ersetzen Sie IDs und URLs.

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### Manifest-Hinweise (Pflichtfelder)

- `bots[].botId` **muss** exakt der Azure-Bot-App-ID entsprechen.
- `webApplicationInfo.id` **muss** der Azure-Bot-App-ID entsprechen.
- `bots[].scopes` muss die von Ihnen geplanten Oberflächen enthalten (`personal`, `team`, `groupChat`).
- `bots[].supportsFiles: true` ist für die Dateiverarbeitung im persönlichen Bereich erforderlich.
- `authorization.permissions.resourceSpecific` muss Kanal-Lesen/Senden enthalten, wenn Sie Kanalverkehr möchten.

### Aktualisieren einer bestehenden App

So aktualisieren Sie eine bereits installierte Teams-App (z. B. zum Hinzufügen von RSC-Berechtigungen):

1. Aktualisieren Sie Ihr `manifest.json` mit den neuen Einstellungen
2. **Erhöhen Sie das Feld `version`** (z. B. `1.0.0` → `1.1.0`)
3. **Erneut zippen** des Manifests mit Icons (`manifest.json`, `outline.png`, `color.png`)
4. Neues ZIP hochladen:
   - **Option A (Teams Admin Center):** Teams Admin Center → Teams apps → Manage apps → App finden → Upload new version
   - **Option B (Sideload):** In Teams → Apps → Manage your apps → Upload a custom app
5. **Für Teamkanäle:** App in jedem Team neu installieren, damit neue Berechtigungen wirksam werden
6. **Teams vollständig beenden und neu starten** (nicht nur das Fenster schließen), um zwischengespeicherte Metadaten zu löschen

## Fähigkeiten: Nur RSC vs. Graph

### Mit **Teams RSC בלבד** (App installiert, keine Graph-API-Berechtigungen)

Funktioniert:

- Lesen von Kanalnachrichten (**Text**).
- Senden von Kanalnachrichten (**Text**).
- Empfangen von **persönlichen (DM)** Dateianhängen.

Funktioniert NICHT:

- **Bild- oder Dateiinhalte** in Kanälen/Gruppen (Payload enthält nur HTML-Stub).
- Herunterladen von Anhängen aus SharePoint/OneDrive.
- Lesen des Nachrichtenverlaufs (über das Live-Webhook-Ereignis hinaus).

### Mit **Teams RSC + Microsoft Graph Application-Berechtigungen**

Zusätzlich:

- Herunterladen gehosteter Inhalte (in Nachrichten eingefügte Bilder).
- Herunterladen von Dateianhängen aus SharePoint/OneDrive.
- Lesen des Kanal-/Chat-Nachrichtenverlaufs über Graph.

### RSC vs. Graph API

| Fähigkeit                   | RSC-Berechtigungen                    | Graph API                                  |
| --------------------------- | ------------------------------------- | ------------------------------------------ |
| **Echtzeitnachrichten**     | Ja (via Webhook)   | Nein (nur Polling)      |
| **Historische Nachrichten** | Nein                                  | Ja (Verlauf abfragen)   |
| **Setup-Komplexität**       | Nur App-Manifest                      | Erfordert Admin-Consent + Token-Flow       |
| **Offline nutzbar**         | Nein (muss laufen) | Ja (jederzeit abfragen) |

**Fazit:** RSC ist für Echtzeit-Zuhören; Graph API für historischen Zugriff. Um verpasste Nachrichten im Offline-Zustand nachzuholen, benötigen Sie die Graph API mit `ChannelMessage.Read.All` (erfordert Admin-Consent).

## Graph-aktivierte Medien + Verlauf (erforderlich für Kanäle)

Wenn Sie Bilder/Dateien in **Kanälen** benötigen oder den **Nachrichtenverlauf** abrufen möchten, müssen Sie Microsoft-Graph-Berechtigungen aktivieren und Admin-Consent erteilen.

1. In Entra ID (Azure AD) **App Registration** Microsoft Graph **Application permissions** hinzufügen:
   - `ChannelMessage.Read.All` (Kanalanhänge + Verlauf)
   - `Chat.Read.All` oder `ChatMessage.Read.All` (Gruppenchats)
2. **Admin-Consent** für den Tenant erteilen.
3. Teams-App-**Manifest-Version** erhöhen, erneut hochladen und **App in Teams neu installieren**.
4. **Teams vollständig beenden und neu starten**, um Cache zu leeren.

**Zusätzliche Berechtigung für Benutzererwähnungen:** Benutzer-@mentions funktionieren standardmäßig für Benutzer in der Konversation. Wenn du jedoch Benutzer dynamisch suchen und erwähnen möchtest, die sich **nicht in der aktuellen Konversation** befinden, füge die Berechtigung `User.Read.All` (Application) hinzu und erteile Admin-Zustimmung.

## Bekannte Einschränkungen

### Webhook-Timeouts

Teams liefert Nachrichten per HTTP-Webhook. Wenn die Verarbeitung zu lange dauert (z. B. langsame LLM-Antworten), können auftreten:

- Gateway-Timeouts
- Teams wiederholt die Nachricht (verursacht Duplikate)
- Verlorene Antworten

OpenClaw handhabt dies, indem es schnell zurückkehrt und Antworten proaktiv sendet; sehr langsame Antworten können dennoch Probleme verursachen.

### Formatierung

Teams-Markdown ist eingeschränkter als Slack oder Discord:

- Grundformatierung funktioniert: **fett**, _kursiv_, `code`, Links
- Komplexes Markdown (Tabellen, verschachtelte Listen) wird möglicherweise nicht korrekt dargestellt
- Adaptive Cards werden für Umfragen und beliebige Karten unterstützt (siehe unten)

## Konfiguration

Wichtige Einstellungen (siehe `/gateway/configuration` für gemeinsame Kanal-Muster):

- `channels.msteams.enabled`: Kanal aktivieren/deaktivieren.
- `channels.msteams.appId`, `channels.msteams.appPassword`, `channels.msteams.tenantId`: Bot-Zugangsdaten.
- `channels.msteams.webhook.port` (Standard `3978`)
- `channels.msteams.webhook.path` (Standard `/api/messages`)
- `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (Standard: pairing)
- `channels.msteams.allowFrom`: Allowlist für DMs (AAD-Objekt-IDs, UPNs oder Anzeigenamen). Der Assistent löst Namen bei verfügbarem Graph-Zugriff während des Setups in IDs auf.
- `channels.msteams.textChunkLimit`: Ausgehende Text-Chunk-Größe.
- `channels.msteams.chunkMode`: `length` (Standard) oder `newline`, um vor der Längenaufteilung an Leerzeilen (Absatzgrenzen) zu splitten.
- `channels.msteams.mediaAllowHosts`: Allowlist für eingehende Anhang-Hosts (Standard: Microsoft-/Teams-Domains).
- `channels.msteams.mediaAuthAllowHosts`: Allowlist zum Anhängen von Authorization-Headern bei Medien-Retries (Standard: Graph- + Bot-Framework-Hosts).
- `channels.msteams.requireMention`: @Erwähnung in Kanälen/Gruppen erforderlich (Standard true).
- `channels.msteams.replyStyle`: `thread | top-level` (siehe [Antwortstil](#reply-style-threads-vs-posts)).
- `channels.msteams.teams.<teamId>.replyStyle`: Pro-Team-Override.
- `channels.msteams.teams.<teamId>.requireMention`: Pro-Team-Override.
- `channels.msteams.teams.<teamId>.tools`: Standardmäßige Pro-Team-Werkzeugrichtlinien-Overrides (`allow`/`deny`/`alsoAllow`), die verwendet werden, wenn ein Kanal-Override fehlt.
- `channels.msteams.teams.<teamId>.toolsBySender`: Standardmäßige Pro-Team-Pro-Absender-Werkzeugrichtlinien-Overrides (`"*"`-Wildcard unterstützt).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: Pro-Kanal-Override.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: Pro-Kanal-Override.
- `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: Pro-Kanal-Werkzeugrichtlinien-Overrides (`allow`/`deny`/`alsoAllow`).
- `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: Pro-Kanal-Pro-Absender-Werkzeugrichtlinien-Overrides (`"*"`-Wildcard unterstützt).
- `channels.msteams.sharePointSiteId`: SharePoint-Site-ID für Datei-Uploads in Gruppenchats/Kanälen (siehe [Dateien in Gruppenchats senden](#sending-files-in-group-chats)).

## Routing & Sitzungen

- Sitzungsschlüssel folgen dem Standard-Agent-Format (siehe [/concepts/session](/concepts/session)):
  - Direktnachrichten teilen sich die Hauptsitzung (`agent:<agentId>:<mainKey>`).
  - Kanal-/Gruppennachrichten verwenden die Konversations-ID:
    - `agent:<agentId>:msteams:channel:<conversationId>`
    - `agent:<agentId>:msteams:group:<conversationId>`

## Antwortstil: Threads vs. Posts

Teams hat kürzlich zwei Kanal-UI-Stile über demselben zugrunde liegenden Datenmodell eingeführt:

| Stil                                           | Beschreibung                                             | Empfohlenes `replyStyle`               |
| ---------------------------------------------- | -------------------------------------------------------- | -------------------------------------- |
| **Posts** (klassisch)       | Nachrichten erscheinen als Karten mit Antworten darunter | `thread` (Standard) |
| **Threads** (Slack-ähnlich) | Nachrichten fließen linear, ähnlich wie Slack            | `top-level`                            |

**Das Problem:** Die Teams-API legt nicht offen, welchen UI-Stil ein Kanal verwendet. Wenn Sie das falsche `replyStyle` verwenden:

- `thread` in einem Threads-Kanal → Antworten werden unglücklich verschachtelt angezeigt
- `top-level` in einem Posts-Kanal → Antworten erscheinen als separate Top-Level-Posts statt im Thread

**Lösung:** Konfigurieren Sie `replyStyle` pro Kanal basierend auf der Kanaleinrichtung:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## Anhänge & Bilder

**Aktuelle Einschränkungen:**

- **DMs:** Bilder und Dateianhänge funktionieren über die Teams-Bot-Datei-APIs.
- **Kanäle/Gruppen:** Anhänge liegen in M365-Speichern (SharePoint/OneDrive). Der Webhook-Payload enthält nur einen HTML-Stub, nicht die tatsächlichen Dateibytes. **Graph-API-Berechtigungen sind erforderlich**, um Kanalanhänge herunterzuladen.

Ohne Graph-Berechtigungen werden Kanalnachrichten mit Bildern nur als Text empfangen (der Bildinhalt ist für den Bot nicht zugänglich).
Standardmäßig lädt OpenClaw Medien nur von Microsoft-/Teams-Hostnamen herunter. Überschreiben mit `channels.msteams.mediaAllowHosts` (verwenden Sie `["*"]`, um jeden Host zuzulassen).
Authorization-Header werden nur für Hosts in `channels.msteams.mediaAuthAllowHosts` angehängt (Standard: Graph- + Bot-Framework-Hosts). Halten Sie diese Liste strikt (vermeiden Sie Multi-Tenant-Suffixe).

## Dateien in Gruppenchats senden

Bots können Dateien in DMs über den FileConsentCard-Flow (integriert) senden. **Das Senden von Dateien in Gruppenchats/Kanälen** erfordert jedoch zusätzliche Einrichtung:

| Kontext                                       | Wie Dateien gesendet werden                         | Erforderliches Setup                                |
| --------------------------------------------- | --------------------------------------------------- | --------------------------------------------------- |
| **DMs**                                       | FileConsentCard → Nutzer akzeptiert → Bot lädt hoch | Funktioniert außerhalb der Box                      |
| **Gruppenchats/Kanäle**                       | Upload zu SharePoint → Freigabelink                 | Erfordert `sharePointSiteId` + Graph-Berechtigungen |
| **Bilder (jeder Kontext)** | Base64-kodiert inline                               | Funktioniert außerhalb der Box                      |

### Warum Gruppenchats SharePoint benötigen

Bots haben kein persönliches OneDrive-Laufwerk (der `/me/drive`-Graph-API-Endpunkt funktioniert nicht für Anwendungsidentitäten). Um Dateien in Gruppenchats/Kanälen zu senden, lädt der Bot in eine **SharePoint-Site** hoch und erstellt einen Freigabelink.

### Setup

1. **Graph-API-Berechtigungen hinzufügen** in Entra ID (Azure AD) → App Registration:
   - `Sites.ReadWrite.All` (Application) – Dateien zu SharePoint hochladen
   - `Chat.Read.All` (Application) – optional, ermöglicht benutzerspezifische Freigabelinks

2. **Admin-Consent** für den Tenant erteilen.

3. **SharePoint-Site-ID ermitteln:**

   ```bash
   # Via Graph Explorer or curl with a valid token:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # Example: for a site at "contoso.sharepoint.com/sites/BotFiles"
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # Response includes: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **OpenClaw konfigurieren:**

   ```json5
   {
     channels: {
       msteams: {
         // ... other config ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2",
       },
     },
   }
   ```

### Freigabeverhalten

| Berechtigung                            | Freigabeverhalten                                                            |
| --------------------------------------- | ---------------------------------------------------------------------------- |
| `Sites.ReadWrite.All` only              | Organisationsweiter Freigabelink (alle im Org zugänglich) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | Benutzerspezifischer Freigabelink (nur Chat-Mitglieder)   |

Benutzerspezifische Freigaben sind sicherer, da nur Chat-Teilnehmer Zugriff haben. Fehlt die Berechtigung `Chat.Read.All`, fällt der Bot auf organisationsweite Freigabe zurück.

### Fallback-Verhalten

| Szenario                                               | Ergebnis                                                                          |
| ------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Gruppenchats + Datei + `sharePointSiteId` konfiguriert | Upload zu SharePoint, Freigabelink senden                                         |
| Gruppenchats + Datei + kein `sharePointSiteId`         | OneDrive-Upload versuchen (kann fehlschlagen), nur Text senden |
| Persönlicher Chat + Datei                              | FileConsentCard-Flow (funktioniert ohne SharePoint)            |
| Jeder Kontext + Bild                                   | Base64-kodiert inline (funktioniert ohne SharePoint)           |

### Speicherort der Dateien

Hochgeladene Dateien werden in einem `/OpenClawShared/`-Ordner in der Standard-Dokumentbibliothek der konfigurierten SharePoint-Site gespeichert.

## Umfragen (Adaptive Cards)

OpenClaw sendet Teams-Umfragen als Adaptive Cards (es gibt keine native Teams-Umfrage-API).

- CLI: `openclaw message poll --channel msteams --target conversation:<id> ...`
- Stimmen werden vom Gateway in `~/.openclaw/msteams-polls.json` gespeichert.
- Das Gateway muss online bleiben, um Stimmen zu erfassen.
- Umfragen posten derzeit noch keine Ergebniszusammenfassungen automatisch (prüfen Sie bei Bedarf die Store-Datei).

## Adaptive Cards (beliebig)

Senden Sie beliebiges Adaptive-Card-JSON an Teams-Benutzer oder -Konversationen mit dem Werkzeug oder der CLI `message`.

Der Parameter `card` akzeptiert ein Adaptive-Card-JSON-Objekt. Wenn `card` angegeben ist, ist der Nachrichtentext optional.

**Agent-Werkzeug:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello!" }]
  }
}
```

**CLI:**

```bash
openclaw message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

Siehe [Adaptive-Cards-Dokumentation](https://adaptivecards.io/) für Schema und Beispiele. Details zum Zielformat finden Sie unten unter [Zielformate](#target-formats).

## Zielformate

MSTeams-Ziele verwenden Präfixe zur Unterscheidung zwischen Benutzern und Konversationen:

| Zieltyp                                 | Format                           | Beispiel                                                                  |
| --------------------------------------- | -------------------------------- | ------------------------------------------------------------------------- |
| Benutzer (nach ID)   | `user:<aad-object-id>`           | `user:40a1a0ed-4ff2-4164-a219-55518990c197`                               |
| Benutzer (nach Name) | `user:<display-name>`            | `user:John Smith` (erfordert Graph API)                |
| Gruppe/Kanal                            | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2`                                  |
| Gruppe/Kanal (raw)   | `<conversation-id>`              | `19:abc123...@thread.tacv2` (wenn `@thread` enthalten) |

**CLI-Beispiele:**

```bash
# Send to a user by ID
openclaw message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# Send to a user by display name (triggers Graph API lookup)
openclaw message send --channel msteams --target "user:John Smith" --message "Hello"

# Send to a group chat or channel
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# Send an Adaptive Card to a conversation
openclaw message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**Agent-Werkzeug-Beispiele:**

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{ "type": "TextBlock", "text": "Hello" }]
  }
}
```

Hinweis: Ohne das Präfix `user:` werden Namen standardmäßig als Gruppe/Team aufgelöst. Verwenden Sie immer `user:`, wenn Sie Personen nach Anzeigenamen ansprechen.

## Proaktive Nachrichten

- Proaktive Nachrichten sind nur **nachdem** ein Benutzer interagiert hat möglich, da wir zu diesem Zeitpunkt Konversationsreferenzen speichern.
- Siehe `/gateway/configuration` für `dmPolicy` und Allowlist-Gating.

## Team- und Kanal-IDs (häufige Stolperfalle)

Der Query-Parameter `groupId` in Teams-URLs ist **NICHT** die Team-ID, die für die Konfiguration verwendet wird. Extrahieren Sie stattdessen die IDs aus dem URL-Pfad:

**Team-URL:**

```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    Team ID (URL-decode this)
```

**Kanal-URL:**

```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      Channel ID (URL-decode this)
```

**Für die Konfiguration:**

- Team-ID = Pfadsegment nach `/team/` (URL-dekodiert, z. B. `19:Bk4j...@thread.tacv2`)
- Kanal-ID = Pfadsegment nach `/channel/` (URL-dekodiert)
- **Ignorieren** Sie den Query-Parameter `groupId`

## Private Kanäle

Bots haben eingeschränkte Unterstützung in privaten Kanälen:

| Funktion                                         | Standardkanäle | Private Kanäle                             |
| ------------------------------------------------ | -------------- | ------------------------------------------ |
| Bot-Installation                                 | Ja             | Eingeschränkt                              |
| Echtzeitnachrichten (Webhook) | Ja             | Funktioniert sofort                        |
| RSC-Berechtigungen                               | Ja             | Können sich anders verhalten               |
| @Erwähnungen                        | Ja             | Wenn Bot zugänglich                        |
| Graph-API-Verlauf                                | Ja             | Ja (mit Berechtigungen) |

**Workarounds, falls private Kanäle nicht funktionieren:**

1. Standardkanäle für Bot-Interaktionen verwenden
2. DMs verwenden – Benutzer können den Bot jederzeit direkt anschreiben
3. Graph API für historischen Zugriff verwenden (erfordert `ChannelMessage.Read.All`)

## Häufige Probleme

### Häufige Probleme

- **Bilder werden in Kanälen nicht angezeigt:** Graph-Berechtigungen oder Admin-Consent fehlen. Teams-App neu installieren und Teams vollständig beenden/neu öffnen.
- **Keine Antworten im Kanal:** Erwähnungen sind standardmäßig erforderlich; setzen Sie `channels.msteams.requireMention=false` oder konfigurieren Sie pro Team/Kanal.
- **Versionskonflikt (Teams zeigt noch altes Manifest):** App entfernen + erneut hinzufügen und Teams vollständig beenden, um zu aktualisieren.
- **401 Unauthorized vom Webhook:** Erwartet bei manuellem Test ohne Azure-JWT – bedeutet, der Endpunkt ist erreichbar, aber die Authentifizierung schlug fehl. Verwenden Sie Azure Web Chat für korrekte Tests.

### Fehler beim Manifest-Upload

- **„Icon file cannot be empty“:** Das Manifest verweist auf Icon-Dateien mit 0 Byte. Erstellen Sie gültige PNG-Icons (32×32 für `outline.png`, 192×192 für `color.png`).
- **„webApplicationInfo.Id already in use“:** Die App ist noch in einem anderen Team/Chat installiert. Deinstallieren Sie sie dort oder warten Sie 5–10 Minuten auf die Propagierung.
- **„Something went wrong“ beim Upload:** Laden Sie stattdessen über [https://admin.teams.microsoft.com](https://admin.teams.microsoft.com) hoch, öffnen Sie die Browser-DevTools (F12) → Network-Tab und prüfen Sie den Response-Body auf den eigentlichen Fehler.
- **Sideload schlägt fehl:** Versuchen Sie „Upload an app to your org's app catalog“ statt „Upload a custom app“ – das umgeht häufig Sideload-Beschränkungen.

### RSC-Berechtigungen funktionieren nicht

1. Verifizieren Sie, dass `webApplicationInfo.id` exakt der App-ID Ihres Bots entspricht
2. App erneut hochladen und im Team/Chat neu installieren
3. Prüfen Sie, ob Ihr Org-Admin RSC-Berechtigungen blockiert hat
4. Bestätigen Sie den richtigen Scope: `ChannelMessage.Read.Group` für Teams, `ChatMessage.Read.Chat` für Gruppenchats

## Referenzen

- [Azure Bot erstellen](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) – Leitfaden zur Azure-Bot-Einrichtung
- [Teams Developer Portal](https://dev.teams.microsoft.com/apps) – Teams-Apps erstellen/verwalten
- [Teams-App-Manifest-Schema](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
- [Kanalnachrichten mit RSC empfangen](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
- [RSC-Berechtigungsreferenz](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
- [Teams-Bot-Dateiverarbeitung](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4) (Kanal/Gruppe erfordert Graph)
- [Proaktive Nachrichten](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)
