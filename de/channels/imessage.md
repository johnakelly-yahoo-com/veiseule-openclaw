---
summary: "Legacy-iMessage-Unterstützung über imsg (JSON-RPC über stdio). Neue Setups sollten BlueBubbles verwenden."
read_when:
  - Einrichten der iMessage-Unterstützung
  - Debugging von iMessage-Senden/Empfangen
title: "iMessage"
---

# iMessage (Legacy: imsg)

<Warning>
**Empfohlen:** Verwenden Sie [BlueBubbles](/channels/bluebubbles) für neue iMessage-Setups.

Der Kanal `imsg` ist eine Legacy-Integration über eine externe CLI und kann in einer zukünftigen Version entfernt werden. 
</Warning>

Status: Legacy-Integration über externe CLI. Der Gateway startet `imsg rpc` (JSON-RPC über stdio).

<CardGroup cols={3}>
  <Card title="BlueBubbles (recommended)" icon="message-circle" href="/channels/bluebubbles">
    Bevorzugter iMessage-Pfad für neue Setups.
  
</Card>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    iMessage-DMs verwenden standardmäßig den Pairing-Modus.
  
</Card>
  <Card title="Configuration reference" icon="settings" href="/gateway/configuration-reference#imessage">
    Vollständige Referenz der iMessage-Felder.
  
</Card>
</CardGroup>

## Schnellsetup (Einsteiger)

<Tabs>
  <Tab title="Local Mac (fast path)">
    <Steps>
      <Step title="Install and verify imsg">

```bash
`brew install steipete/tap/imsg`
```

        
</Step>
      
        <Step title="OpenClaw konfigurieren">

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db",
    },
  },
}
```

        
</Step>
      
        <Step title="Gateway starten">

```bash
openclaw gateway
```

        
</Step>
      
        <Step title="Erstes DM-Pairing genehmigen (Standard-dmPolicy)">

```bash
`openclaw pairing approve imessage <CODE>`
```

        ```
            Pairing-Anfragen laufen nach 1 Stunde ab.
          
</Step>
        
</Steps>
        ```

  
</Tab>

  <Tab title="Remote Mac over SSH">Wenn Sie iMessage auf einem anderen Mac betreiben möchten, setzen Sie `channels.imessage.cliPath` auf einen Wrapper, der `imsg` auf dem entfernten macOS-Host über SSH ausführt. OpenClaw benötigt nur stdio.

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

    ```
    Empfohlene Konfiguration, wenn Anhänge aktiviert sind:
    ```

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh", // SSH wrapper to remote Mac
      remoteHost: "user@gateway-host", // for SCP file transfer
      includeAttachments: true,
    },
  },
}
```

    ```
    Wenn `remoteHost` nicht gesetzt ist, versucht OpenClaw, dies durch Parsen des SSH-Befehls in Ihrem Wrapper-Skript automatisch zu erkennen.
    ```

  
</Tab>
</Tabs>

## Anforderungen und Berechtigungen (macOS)

- iMessage-Kanal auf Basis von `imsg` unter macOS.
- Vollzugriff auf die Festplatte für OpenClaw + `imsg` (Zugriff auf die Nachrichten-Datenbank).
- Automationsberechtigung beim Senden.

<Tip>
Berechtigungen werden pro Prozesskontext erteilt. Wenn das Gateway headless (LaunchAgent/SSH) läuft, führe einen einmaligen interaktiven Befehl im selben Kontext aus, um die Abfragen auszulösen:

```bash
imsg chats --limit 1
# or
imsg send <handle> "test"
```

</Tip>

## Zugriffskontrolle und Routing

<Tabs>
  <Tab title="DM policy">
    `channels.imessage.dmPolicy` steuert Direktnachrichten:


    ```
    `channels.imessage.groupPolicy`: `open | allowlist | disabled` (Standard: Allowlist).
    ```

  
</Tab>

  <Tab title="Group policy + mentions">`channels.imessage.groupAllowFrom`: Gruppen-Absender-Allowlist.

    ```
    {
      channels: {
        imessage: {
          enabled: true,
          accounts: {
            bot: {
              name: "Bot",
              enabled: true,
              cliPath: "/path/to/imsg-bot",
              dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db",
            },
          },
        },
      },
    }
    ```

  
</Tab>

  <Tab title="Sessions and deterministic replies">
    - DMs verwenden direktes Routing; Gruppen verwenden Gruppen-Routing.
    - Mit der Standardkonfiguration `session.dmScope=main` werden iMessage-DMs in die Hauptsitzung des Agenten zusammengeführt.
    - Gruppensitzungen sind isoliert (`agent:
<agentId>Gruppen:<chat_id>`).
    - Antworten werden anhand der Metadaten des ursprünglichen Kanals/Ziels zurück an iMessage geleitet.

    ```
    Wenn ein Thread mit mehreren Teilnehmern mit `is_group=false` eingeht, können Sie ihn dennoch isolieren, indem Sie `chat_id` unter Verwendung von `channels.imessage.groups` konfigurieren (siehe „Gruppenähnliche Threads“ unten).
    ```

  
</Tab>
</Tabs>

## Deployment-Muster

<AccordionGroup>
  <Accordion title="Dedicated bot macOS user (separate iMessage identity)">Wenn der Bot von einer **separaten iMessage-Identität** senden soll (und Ihre persönlichen Nachrichten sauber bleiben sollen), verwenden Sie eine dedizierte Apple-ID + einen dedizierten macOS-Benutzer.

    ```
    Verweisen Sie `channels.imessage.accounts.bot.cliPath` auf einen SSH-Wrapper, der `imsg` als Bot-Benutzer ausführt.
    ```

  
</Accordion>

  <Accordion title="Remote Mac over Tailscale (example)">
    Gängige Topologie:


    ```
    Wenn der Gateway auf einem Linux-Host/VM läuft, iMessage jedoch auf einem Mac laufen muss, ist Tailscale die einfachste Brücke: Der Gateway spricht über das Tailnet mit dem Mac, führt `imsg` per SSH aus und lädt Anhänge per SCP zurück.
    ```

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.openclaw/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db",
    },
  },
}
```

```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

    ```
    Verwende SSH-Schlüssel, sodass sowohl SSH als auch SCP nicht-interaktiv sind.
    ```

  
</Accordion>

  <Accordion title="Multi-account pattern">Für Setups mit nur einem Konto verwenden Sie flache Optionen (`channels.imessage.cliPath`, `channels.imessage.dbPath`) anstelle der `accounts`-Map.

    ```
    Jedes Konto kann Felder wie `cliPath`, `dbPath`, `allowFrom`, `groupPolicy`, `mediaMaxMb` und Verlaufs-Einstellungen überschreiben.
    ```

  
</Accordion>
</AccordionGroup>

## Medien, Chunking und Zustellziele

<AccordionGroup>
  <Accordion title="Attachments and media">Medien-Uploads sind durch `channels.imessage.mediaMaxMb` begrenzt (Standard 16).
</Accordion>

  <Accordion title="Outbound chunking">Optionale Zeilenumbruch-Segmentierung: Setzen Sie `channels.imessage.chunkMode="newline"`, um an Leerzeilen (Absatzgrenzen) vor der Längensegmentierung zu trennen.
</Accordion>

  <Accordion title="Addressing formats">
    Bevorzugte explizite Ziele:


    ```
    - `chat_id:123` (empfohlen für stabiles Routing)
    - `chat_guid:...`
    - `chat_identifier:...`
    
    Handle-Ziele werden ebenfalls unterstützt:
    
    - `imessage:+1555...`
    - `sms:+1555...`
    - `user@example.com`
    ```

```bash
imsg chats --limit 20
```

  
</Accordion>
</AccordionGroup>

## Konfigurationsschreibzugriffe

Standardmäßig darf iMessage Konfigurationsaktualisierungen schreiben, die durch `/config set|unset` ausgelöst werden (erfordert `commands.config: true`).

Deaktivieren mit:

```json5
{
  channels: { imessage: { configWrites: false } },
}
```

## Fehlerbehebung

<AccordionGroup>
  <Accordion title="imsg not found or RPC unsupported">
    Überprüfe die Binärdatei und die RPC-Unterstützung:


```bash
imsg rpc --help
openclaw channels status --probe
```

    ```
    Wenn die Probe RPC als nicht unterstützt meldet, aktualisiere `imsg`.
    ```

  
</Accordion>

  <Accordion title="DMs are ignored">Hinweise:

    ```
    `channels.imessage.groupPolicy = open | allowlist | disabled`.
    ```

  
</Accordion>

  <Accordion title="Group messages are ignored">Genehmigung über:

    ```
    `channels.imessage.groupAllowFrom` steuert, wer in Gruppen auslösen darf, wenn `allowlist` gesetzt ist.
    ```

  
</Accordion>

  <Accordion title="Remote attachments fail">Checkliste:

    ```
    #!/usr/bin/env bash
    set -euo pipefail
    
    # Run an interactive SSH once first to accept host keys:
    #   ssh <bot-macos-user>@localhost true
    exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
      "/usr/local/bin/imsg" "$@"
    ```

  
</Accordion>

  <Accordion title="macOS permission prompts were missed">    Führe den Befehl erneut in einem interaktiven GUI-Terminal im selben Benutzer-/Sitzungskontext aus und bestätige die Eingabeaufforderungen:

```bash
imsg chats --limit 1
imsg send <handle> "test"
```

    ```
    **Automation → Nachrichten**: Erlauben Sie dem Prozess, der OpenClaw ausführt (und/oder Ihrem Terminal), **Messages.app** für ausgehende Sendungen zu steuern.
    ```

  
</Accordion>
</AccordionGroup>

## Referenzhinweise zur Konfiguration

- Konfigurationsreferenz (iMessage)
- Vollständige Konfiguration: [Konfiguration](/gateway/configuration)
- [Pairing](/channels/pairing)
- [BlueBubbles](/channels/bluebubbles)
