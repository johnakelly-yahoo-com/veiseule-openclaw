---
summary: "Signal-stöd via signal-cli (JSON-RPC + SSE), konfigurering och nummermodell"
read_when:
  - Konfigurering av Signal-stöd
  - Felsökning av Signal sändning/mottagning
title: "Signal"
---

# Signal (signal-cli)

Status: extern CLI-integration. Gateway talar med `signal-cli` över HTTP JSON-RPC + SSE.

## Snabb konfiguration (nybörjare)

- Använd ett **separat Signal-nummer** för boten (rekommenderas).
- Installera `signal-cli` (Java krävs).
- Länka bot-enheten och starta daemonen:
- Konfigurera OpenClaw och starta gateway.

## Snabb konfiguration (nybörjare)

1. Använd ett **separat Signal-nummer** för boten (rekommenderas).
2. Installera `signal-cli` (Java krävs om du använder JVM-bygget).
3. Välj en installationsväg:
   - **Väg A (QR-länk):** `signal-cli link -n "OpenClaw"` och skanna med Signal.
   - **Väg B (SMS-registrering):** registrera ett dedikerat nummer med captcha + SMS-verifiering.
4. Konfigurera OpenClaw och starta om gatewayen.
5. Skicka ett första DM och godkänn parkoppling (`openclaw pairing approve signal <CODE>`).

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

Fältreferens:

| Fält        | Beskrivning                                                                             |
| ----------- | --------------------------------------------------------------------------------------- |
| `account`   | Botens telefonnummer i E.164-format (`+15551234567`) |
| `cliPath`   | Sökväg till `signal-cli` (`signal-cli` om den finns i `PATH`)        |
| `dmPolicy`  | Åtkomstpolicy för DM (`pairing` rekommenderas)                       |
| `allowFrom` | Telefonnummer eller `uuid:&lt;id&gt;`-värden som får skicka DM                                |

## Vad det är

- Signal-kanal via `signal-cli` (inte inbäddat libsignal).
- Deterministisk routning: svar går alltid tillbaka till Signal.
- Direktmeddelanden delar agentens huvudsession; grupper är isolerade (`agent:<agentId>:signal:group:<groupId>`).

## Nummermodellen (viktigt)

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

## Installationsväg A: länka befintligt Signal-konto (QR)

1. Installera `signal-cli` (JVM- eller native‑version).
2. Länka ett bot-konto:
   - `signal-cli link -n "OpenClaw"` och skanna sedan QR-koden i Signal.
3. Konfigurera Signal och starta gateway.

Om du vill hantera `signal-cli` själv (långsamma JVM-kallstarter, container-init eller delade CPU:er), kör daemonen separat och peka OpenClaw mot den:

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

## Åtkomstkontroll (DMs + grupper)

Direktmeddelanden:

1. Standard: `channels.signal.dmPolicy = "pairing"`.
   - Använd ett dedikerat botnummer för att undvika konto-/sessionskonflikter.
2. Okända avsändare får en parningskod; meddelanden ignoreras tills de godkänts (koder upphör efter 1 timme).

```bash
VERSION=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/AsamK/signal-cli/releases/latest | sed -e 's/^.*\/v//')
curl -L -O "https://github.com/AsamK/signal-cli/releases/download/v${VERSION}/signal-cli-${VERSION}-Linux-native.tar.gz"
sudo tar xf "signal-cli-${VERSION}-Linux-native.tar.gz" -C /opt
sudo ln -sf /opt/signal-cli /usr/local/bin/
signal-cli --version
```

Om du använder JVM-versionen (`signal-cli-${VERSION}.tar.gz`), installera JRE 25+ först.
Håll `signal-cli` uppdaterad; upstream påpekar att äldre versioner kan sluta fungera när Signals server-API:er ändras.

3. Registrera och verifiera numret:

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register
```

Om captcha krävs:

1. Utgående text delas upp till `channels.signal.textChunkLimit` (standard 4000).
2. Valfri radbrytningsuppdelning: sätt `channels.signal.chunkMode="newline"` för att dela på tomma rader (styckegränser) före längduppdelning.
3. Bilagor stöds (base64 hämtas från `signal-cli`).
4. Standardgräns för media: `channels.signal.mediaMaxMb` (standard 8).

```bash
signal-cli -a +<BOT_PHONE_NUMBER> register --captcha '<SIGNALCAPTCHA_URL>'
signal-cli -a +<BOT_PHONE_NUMBER> verify <VERIFICATION_CODE>
```

4. **Skrivindikatorer**: OpenClaw skickar skrivsignaler via `signal-cli sendTyping` och uppdaterar dem medan ett svar körs.

```bash
# Om du kör gatewayen som en systemd-användartjänst:
systemctl --user restart openclaw-gateway

# Verifiera sedan:
openclaw doctor
openclaw channels status --probe
```

5. Använd `message action=react` med `channel=signal`.
   - Skicka valfritt meddelande till botnumret.
   - Godkänn koden på servern: `openclaw pairing approve signal <PAIRING_CODE>`.
   - Spara botnumret som en kontakt i din telefon för att undvika "Okänd kontakt".

Viktigt: att registrera ett telefonnummerkonto med `signal-cli` kan avautentisera huvudsessionen i Signal-appen för det numret. Föredra ett dedikerat botnummer, eller använd QR-länkningsläge om du behöver behålla din befintliga appkonfiguration i telefonen.

Upstream-referenser:

- `signal-cli` README: `https://github.com/AsamK/signal-cli`
- Captcha-flöde: `https://github.com/AsamK/signal-cli/wiki/Registration-with-captcha`
- Länkningsflöde: `https://github.com/AsamK/signal-cli/wiki/Linking-other-devices-(Provisioning)`

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

## Konfigurationsreferens (Signal)

- Utgående text delas upp till `channels.signal.textChunkLimit` (standard 4000).
- Valfri radbrytningsuppdelning: sätt `channels.signal.chunkMode="newline"` för att dela på tomma rader (styckegränser) före längduppdelning.
- Bilagor stöds (base64 hämtas från `signal-cli`).
- Standardgräns för media: `channels.signal.mediaMaxMb` (standard 8).
- Använd `channels.signal.ignoreAttachments` för att hoppa över nedladdning av media.
- Grupphistorik sammanhang använder `channels.signal.historyLimit` (eller `channels.signal.accounts.*.historyLimit`), faller tillbaka till `messages.groupChat.historyLimit`. Sätt `0` till att inaktivera (standard 50).

## Skrivindikatorer + läskvitton

- `channels.signal.enabled`: aktivera/inaktivera kanalstart.
- `channels.signal.account`: E.164 för bot-kontot.
- `channels.signal.cliPath`: sökväg till `signal-cli`.

## Reaktioner (meddelandeverktyg)

- `agents.list[].groupChat.mentionPatterns` (Signal stöder inte inbyggda omnämnanden).
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
- Valideringsfel i konfigurationen efter ändringar: kör `openclaw doctor --fix`.
- Signal saknas i diagnostiken: kontrollera `channels.signal.enabled: true`.

Extra kontroller:

```bash
openclaw pairing list signal
pgrep -af signal-cli
grep -i "signal" "/tmp/openclaw/openclaw-$(date +%Y-%m-%d).log" | tail -20
```

För triage-flöde: [/channels/troubleshooting](/channels/troubleshooting).

## Säkerhetsanmärkningar

- `signal-cli` lagrar kontonycklar lokalt (vanligtvis `~/.local/share/signal-cli/data/`).
- Säkerhetskopiera Signal-kontots tillstånd före servermigrering eller ominstallation.
- Behåll `channels.signal.dmPolicy: "pairing"` om du inte uttryckligen vill tillåta bredare DM-åtkomst.
- SMS-verifiering behövs endast vid registrering eller återställning, men att förlora kontrollen över numret/kontot kan försvåra omregistrering.

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

