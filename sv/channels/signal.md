---
title: "Signal"
---

# Signal (signal-cli)

Status: extern CLI-integration. Gateway talar med `signal-cli` över HTTP JSON-RPC + SSE.

## Snabb konfiguration (nybörjare)

1. Använd ett **separat Signal-nummer** för boten (rekommenderas).
2. Installera `signal-cli` (Java krävs).
3. Länka bot-enheten och starta daemonen:
   - `signal-cli link -n "OpenClaw"`
4. Konfigurera OpenClaw och starta gateway.

Minimal konfig:

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

## Vad det är

- Signal-kanal via `signal-cli` (inte inbäddat libsignal).
- Deterministisk routning: svar går alltid tillbaka till Signal.
- Direktmeddelanden delar agentens huvudsession; grupper är isolerade (`agent:<agentId>:signal:group:<groupId>`).

## Konfigskrivningar

Som standard får Signal skriva konfiguppdateringar som triggas av `/config set|unset` (kräver `commands.config: true`).

Inaktivera med:

```json5
{
  channels: { signal: { configWrites: false } },
}
```

## Nummermodellen (viktigt)

- Gateway ansluter till en **Signal-enhet** (kontot `signal-cli`).
- Om du kör boten på **ditt personliga Signal-konto** kommer den att ignorera dina egna meddelanden (loopskydd).
- För ”jag sms:ar boten och den svarar”, använd ett **separat bot-nummer**.

## Konfigurering (snabb väg)

1. Installera `signal-cli` (Java krävs).
2. Länka ett bot-konto:
   - `signal-cli link -n "OpenClaw"` och skanna sedan QR-koden i Signal.
3. Konfigurera Signal och starta gateway.

Exempel:

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

Stöd för flera konton: använd `channels.signal.accounts` med konfiguration per konto och valfri `name`. Se [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) för det delade mönstret.

## Externt daemon-läge (httpUrl)

Om du vill hantera `signal-cli` själv (långsamma JVM-kallstarter, container-init eller delade CPU:er), kör daemonen separat och peka OpenClaw mot den:

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

Detta hoppar över auto-spawn och start vänta inuti OpenClaw. För långsam startar vid auto-spawning, ange `channels.signal.startupTimeoutMs`.

## Åtkomstkontroll (DMs + grupper)

Direktmeddelanden:

- Standard: `channels.signal.dmPolicy = "pairing"`.
- Okända avsändare får en parningskod; meddelanden ignoreras tills de godkänts (koder upphör efter 1 timme).
- Godkänn via:
  - `openclaw pairing list signal`
  - `openclaw pairing approve signal <CODE>`
- Parkoppling är standard token utbyte för Signal DMs. Detaljer: [Pairing](/channels/pairing)
- Endast-UUID-avsändare (från `sourceUuid`) lagras som `uuid:<id>` i `channels.signal.allowFrom`.

Grupper:

- `channels.signal.groupPolicy = open | allowlist | disabled`.
- `channels.signal.groupAllowFrom` styr vem som kan trigga i grupper när `allowlist` är satt.

## Hur det fungerar (beteende)

- `signal-cli` körs som en daemon; gateway läser händelser via SSE.
- Inkommande meddelanden normaliseras till det delade kanalomslaget.
- Svar routas alltid tillbaka till samma nummer eller grupp.

## Media + begränsningar

- Utgående text delas upp till `channels.signal.textChunkLimit` (standard 4000).
- Valfri radbrytningsuppdelning: sätt `channels.signal.chunkMode="newline"` för att dela på tomma rader (styckegränser) före längduppdelning.
- Bilagor stöds (base64 hämtas från `signal-cli`).
- Standardgräns för media: `channels.signal.mediaMaxMb` (standard 8).
- Använd `channels.signal.ignoreAttachments` för att hoppa över nedladdning av media.
- Grupphistorik sammanhang använder `channels.signal.historyLimit` (eller `channels.signal.accounts.*.historyLimit`), faller tillbaka till `messages.groupChat.historyLimit`. Sätt `0` till att inaktivera (standard 50).

## Skrivindikatorer + läskvitton

- **Skrivindikatorer**: OpenClaw skickar skrivsignaler via `signal-cli sendTyping` och uppdaterar dem medan ett svar körs.
- **Läskvitton**: när `channels.signal.sendReadReceipts` är true vidarebefordrar OpenClaw läskvitton för tillåtna DMs.
- Signal-cli exponerar inte läskvitton för grupper.

## Reaktioner (meddelandeverktyg)

- Använd `message action=react` med `channel=signal`.
- Mål: avsändarens E.164 eller UUID (använd `uuid:<id>` från parningsutdata; bar UUID fungerar också).
- `messageId` är Signal-tidsstämpeln för meddelandet du reagerar på.
- Gruppreaktioner kräver `targetAuthor` eller `targetAuthorUuid`.

Exempel:

```
message action=react channel=signal target=uuid:123e4567-e89b-12d3-a456-426614174000 messageId=1737630212345 emoji=🔥
message action=react channel=signal target=+15551234567 messageId=1737630212345 emoji=🔥 remove=true
message action=react channel=signal target=signal:group:<groupId> targetAuthor=uuid:<sender-uuid> messageId=1737630212345 emoji=✅
```

Konfig:

- `channels.signal.actions.reactions`: aktivera/inaktivera reaktionsåtgärder (standard true).
- `channels.signal.reactionLevel`: `off | ack | minimal | extensive`.
  - `off`/`ack` inaktiverar agentreaktioner (meddelandeverktyget `react` ger fel).
  - `minimal`/`extensive` aktiverar agentreaktioner och sätter vägledningsnivån.
- Ersätter varje konto: `channels.signal.accounts.<id>.actions.reactions`, \`channels.signal.accounts.<id>.reaktionNivå.

## Leveransmål (CLI/cron)

- DMs: `signal:+15551234567` (eller vanlig E.164).
- UUID-DMs: `uuid:<id>` (eller bar UUID).
- Grupper: `signal:group:<groupId>`.
- Användarnamn: `username:<name>` (om det stöds av ditt Signal-konto).

## Felsökning

Kör denna stege först:

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

Bekräfta sedan DM-parningsstatus vid behov:

```bash
openclaw pairing list signal
```

Vanliga fel:

- Daemonen nås men inga svar: verifiera konto-/daemoninställningar (`httpUrl`, `account`) och mottagningsläge.
- DMs ignoreras: avsändaren väntar på parningsgodkännande.
- Gruppmeddelanden ignoreras: spärrar för gruppavsändare/omnämnanden blockerar leverans.

För triage-flöde: [/channels/troubleshooting](/channels/troubleshooting).

## Konfigurationsreferens (Signal)

Fullständig konfiguration: [Konfiguration](/gateway/configuration)

Leverantörsalternativ:

- `channels.signal.enabled`: aktivera/inaktivera kanalstart.
- `channels.signal.account`: E.164 för bot-kontot.
- `channels.signal.cliPath`: sökväg till `signal-cli`.
- `channels.signal.httpUrl`: full daemon-URL (åsidosätter värd/port).
- `channels.signal.httpHost`, `channels.signal.httpPort`: daemon-bindning (standard 127.0.0.1:8080).
- `channels.signal.autoStart`: auto-starta daemon (standard true om `httpUrl` inte är satt).
- `channels.signal.startupTimeoutMs`: tidsgräns för startväntan i ms (tak 120000).
- `channels.signal.receiveMode`: `on-start | manual`.
- `channels.signal.ignoreAttachments`: hoppa över nedladdning av bilagor.
- `channels.signal.ignoreStories`: ignorera stories från daemonen.
- `channels.signal.sendReadReceipts`: vidarebefordra läskvitton.
- `channels.signal.dmPolicy`: `pairing | allowlist | open | disabled` (standard: parning).
- `channels.signal.allowFrom`: DM allowlist (E.164 eller `uuid:<id>`). `open` kräver `"*"`. Signalen har inga användarnamn; använd telefon/UUID-ID.
- `channels.signal.groupPolicy`: `open | allowlist | disabled` (standard: tillåtelselista).
- `channels.signal.groupAllowFrom`: tillåtelselista för gruppavsändare.
- `channels.signal.historyLimit`: max antal gruppmeddelanden att inkludera som kontext (0 inaktiverar).
- `channels.signal.dmHistorikLimit`: DM historikgräns i användarens varv. Per-user overrides: `channels.signal.dms["<phone_or_uuid>"].historyLimit`.
- `channels.signal.textChunkLimit`: storlek på utgående uppdelning (tecken).
- `channels.signal.chunkMode`: `length` (standard) eller `newline` för att dela på tomma rader (styckegränser) före längduppdelning.
- `channels.signal.mediaMaxMb`: gräns för inkommande/utgående media (MB).

Relaterade globala alternativ:

- `agents.list[].groupChat.mentionPatterns` (Signal stöder inte inbyggda omnämnanden).
- `messages.groupChat.mentionPatterns` (global fallback).
- `messages.responsePrefix`.
