---
title: "WhatsApp"
---

# WhatsApp (webbkanal)

Status: WhatsApp Web via Baileys bara. Gateway äger session(erna).

## Snabbstart (nybörjare)

1. Använd ett **separat telefonnummer** om möjligt (rekommenderas).
2. Konfigurera WhatsApp i `~/.openclaw/openclaw.json`.
3. Kör `openclaw channels login` för att skanna QR‑koden (Länkade enheter).
4. Starta gatewayen.

Minimal konfiguration:

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

## Mål

- Flera WhatsApp‑konton (multi‑account) i en Gateway‑process.
- Deterministisk routning: svar går tillbaka till WhatsApp, ingen modellroutning.
- Modellen ser tillräcklig kontext för att förstå citerade svar.

## Konfigskrivningar

Som standard får WhatsApp skriva konfiguppdateringar som triggas av `/config set|unset` (kräver `commands.config: true`).

Inaktivera med:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Arkitektur (vem äger vad)

- **Gateway** äger Baileys‑socketen och inkorgsloopen.
- **CLI / macOS‑app** pratar med gatewayen; ingen direkt användning av Baileys.
- **Aktiv lyssnare** krävs för utgående sändningar; annars misslyckas sändningen direkt.

## Skaffa ett telefonnummer (två lägen)

WhatsApp kräver ett riktigt mobilnummer för verifiering. VoIP och virtuella nummer är oftast blockerade. Det finns två sätt att köra OpenClaw på WhatsApp:

### Dedikerat nummer (rekommenderas)

Använd ett **separat telefonnummer** för OpenClaw. Bästa UX, ren routing, inga självchatt egendomar. Idealisk inställning: **reserv/gammal Android-telefon + eSIM**. Lämna den på Wi‐Fi och makt, och länka den via QR.

**WhatsApp Business:** Du kan använda WhatsApp Business på samma enhet med ett annat nummer. Bra för att hålla din personliga WhatsApp separat — installera WhatsApp Business och registrera OpenClaw nummer där.

**Exempelkonfig (dedikerat nummer, allowlist för en användare):**

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

**Parkopplingsläge (valfritt):**
Om du vill para ihop istället för tillåten lista, ange `channels.whatsapp.dmPolicy` till `parkoppling`. Okända avsändare får en parningskod; godkänn med:
`openclaw parkoppling godkänna whatsapp <code>`

### Personligt nummer (reserv)

Snabb reserv: kör OpenClaw på **ditt eget nummer**. Meddelande själv (WhatsApp “Meddelande själv”) för att testa så att du inte spam kontakter. Räkna med att läsa verifieringskoder på din huvudtelefon under installation och experiment. \*\*Måste aktivera självchatt läge. \*
När guiden frågar efter ditt personliga WhatsApp-nummer, ange telefonen kommer du att meddelande från (ägaren/avsändaren), inte assistentnumret.

**Exempelkonfig (personligt nummer, egen‑chatt):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Självchatt svarar standard på `[{identity.name}]` när satt (annars `[openclaw]`)
om `messages.responsePrefix` är unset. Ange uttryckligen att anpassa eller inaktivera
prefixet (använd `""` för att ta bort det).

### Tips för nummeranskaffning

- **Lokalt eSIM** från ditt lands mobiloperatör (mest tillförlitligt)
  - Österrike: [hot.at](https://www.hot.at)
  - Storbritannien: [giffgaff](https://www.giffgaff.com) — gratis SIM, inget avtal
- **Kontantkort** — billigt, behöver bara ta emot ett SMS för verifiering

**Undvik:** TextNow, Google Voice, de flesta ”gratis SMS”‑tjänster — WhatsApp blockerar dessa aggressivt.

**Tips:** Numret behöver endast ta emot ett verifierings-SMS. Efter det, WhatsApp Web sessioner kvarstår via `creds.json`.

## Varför inte Twilio?

- Tidiga OpenClaw‑byggen stödde Twilios WhatsApp Business‑integration.
- WhatsApp Business‑nummer passar dåligt för en personlig assistent.
- Meta upprätthåller ett 24‑timmars svarsfönster; om du inte har svarat de senaste 24 timmarna kan företagsnumret inte initiera nya meddelanden.
- Hög volym eller ”pratig” användning triggar aggressiv blockering, eftersom företagskonton inte är avsedda att skicka dussintals personliga assistentmeddelanden.
- Resultat: opålitlig leverans och frekventa blockeringar, därför togs stödet bort.

## Inloggning + autentiseringsuppgifter

- Inloggningskommando: `openclaw channels login` (QR via Länkade enheter).
- Multi‑account‑inloggning: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Standardkonto (när `--account` utelämnas): `default` om det finns, annars första konfigurerade konto‑ID (sorterat).
- Autentiseringsuppgifter lagras i `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`.
- Säkerhetskopia i `creds.json.bak` (återställs vid korruption).
- Bakåtkompatibilitet: äldre installationer lagrade Baileys‑filer direkt i `~/.openclaw/credentials/`.
- Utloggning: `openclaw channels logout` (eller `--account <id>`) raderar WhatsApp‑auth‑tillståndet (men behåller delad `oauth.json`).
- Utloggad socket ⇒ fel som instruerar att länka igen.

## Inkommande flöde (DM + grupp)

- WhatsApp‑händelser kommer från `messages.upsert` (Baileys).
- Inkorgslyssnare kopplas bort vid nedstängning för att undvika ackumulerade händelsehanterare vid tester/omstarter.
- Status-/broadcast‑chattar ignoreras.
- Direktchattar använder E.164; grupper använder grupp‑JID.
- **DM‑policy**: `channels.whatsapp.dmPolicy` styr åtkomst till direktchatt (standard: `pairing`).
  - Parning: okända avsändare får en parningskod (godkänn via `openclaw pairing approve whatsapp <code>`; koder upphör efter 1 timme).
  - Öppen: kräver att `channels.whatsapp.allowFrom` inkluderar `"*"`.
  - Ditt länkade WhatsApp‑nummer är implicit betrott, så egna meddelanden hoppar över kontrollerna `channels.whatsapp.dmPolicy` och `channels.whatsapp.allowFrom`.

### Personligt‑nummer‑läge (reserv)

Om du kör OpenClaw på **ditt personliga WhatsApp‑nummer**, aktivera `channels.whatsapp.selfChatMode` (se exempel ovan).

Beteende:

- Utgående DM triggar aldrig parningssvar (förhindrar spam till kontakter).
- Inkommande okända avsändare följer fortfarande `channels.whatsapp.dmPolicy`.
- Egen‑chatt‑läge (allowFrom inkluderar ditt nummer) undviker automatiska läskvitton och ignorerar mention‑JID.
- Läskvitton skickas för DM som inte är egen‑chatt.

## Läskvitton

Som standard markerar gatewayen inkommande WhatsApp‑meddelanden som lästa (blå bockar) när de accepteras.

Inaktivera globalt:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Inaktivera per konto:

```json5
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

Noteringar:

- Egen‑chatt‑läge hoppar alltid över läskvitton.

## WhatsApp‑FAQ: skicka meddelanden + parning

**Kommer OpenClaw meddelanden slumpmässiga kontakter när jag länkar WhatsApp?**  
Nej. Standardpolicy för DM är **ihopkoppling**, så okända avsändare får bara en parningskod och deras meddelande **behandlas inte**. OpenClaw svarar bara på chattar som den tar emot, eller för att skicka dig explicit trigger (agent/CLI).

**Hur fungerar parning på WhatsApp?**  
Parning är en DM‑grind för okända avsändare:

- Första DM från en ny avsändare returnerar en kort kod (meddelandet behandlas inte).
- Godkänn med: `openclaw pairing approve whatsapp <code>` (lista med `openclaw pairing list whatsapp`).
- Koder upphör efter 1 timme; väntande förfrågningar är begränsade till 3 per kanal.

**Can multiple people use different OpenClaw instances on one WhatsApp number?**  
Yes, by routing each sender to a different agent via `bindings` (peer `kind: "direct"`, sender E.164 like `+15551234567`). Replies still come from the **same WhatsApp account**, and direct chats collapse to each agent's main session, so use **one agent per person**. DM åtkomstkontroll (`dmPolicy`/`allowFrom`) är global per WhatsApp-konto. Se [Multi-Agent Routing](/concepts/multi-agent).

\*\*Varför ber du om mitt telefonnummer i guiden? \*  
Guiden använder den för att ställa in din **allowlist/owner** så att dina egna DMs är tillåtna. Det används inte för automatisk sändning. Om du kör på ditt personliga WhatsApp-nummer, använd samma nummer och aktivera `channels.whatsapp.selfChatMode`.

## Meddelandenormalisering (vad modellen ser)

- `Body` är aktuell meddelandetext med kuvert.

- Kontext för citerade svar **läggs alltid till**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Svarsmetadata sätts också:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = citerad text eller mediaplatshållare
  - `ReplyToSender` = E.164 när känt

- Inkommande meddelanden med enbart media använder platshållare:
  - `<media:image|video|audio|document|sticker>`

## Grupper

- Grupper mappar till `agent:<agentId>:whatsapp:group:<jid>`‑sessioner.
- Gruppolicy: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (standard `allowlist`).
- Aktiveringslägen:
  - `mention` (standard): kräver @omnämnande eller regex‑träff.
  - `always`: triggar alltid.
- `/activation mention|always` är endast för ägare och måste skickas som ett fristående meddelande.
- Ägare = `channels.whatsapp.allowFrom` (eller egen E.164 om ej satt).
- **Historikinjektion** (endast väntande):
  - Nyliga _obehandlade_ meddelanden (standard 50) infogas under:
    `[Chat messages since your last reply - for context]` (meddelanden som redan finns i sessionen återinjekteras inte)
  - Aktuellt meddelande under:
    `[Current message - respond to this]`
  - Avsändarsuffix läggs till: `[from: Name (+E164)]`
- Gruppmetadata cachelagras i 5 min (ämne + deltagare).

## Leverans av svar (trådning)

- WhatsApp Web skickar standardmeddelanden (ingen citerad svars‑trådning i nuvarande gateway).
- Svarstaggar ignoreras på denna kanal.

## Bekräftelsereaktioner (auto‑reaktion vid mottagning)

WhatsApp kan automatiskt skicka emoji-reaktioner till inkommande meddelanden omedelbart vid mottagandet, innan boten genererar ett svar. Detta ger omedelbar feedback till användare som deras meddelande mottogs.

**Konfiguration:**

```json
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

**Alternativ:**

- `emoji` (sträng): Emoji att använda för bekräftelse (t.ex., "👀", "✅", "📨"). Tom eller utelämnad = funktion inaktiverad.
- `direct` (boolesk, standard: `true`): Skicka reaktioner i direkt-/DM‑chattar.
- `group` (sträng, standard: `"mentions"`): Beteende i gruppchattar:
  - `"always"`: Reagera på alla gruppmeddelanden (även utan @omnämnande)
  - `"mentions"`: Reagera endast när boten @omnämns
  - `"never"`: Reagera aldrig i grupper

**Åsidosättning per konto:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Beteendenoteringar:**

- Reaktioner skickas **omedelbart** vid mottagning av meddelandet, före skrivindikatorer eller botsvar.
- I grupper med `requireMention: false` (aktivering: alltid) kommer `group: "mentions"` att reagera på alla meddelanden (inte bara @omnämnanden).
- Fire‑and‑forget: misslyckade reaktioner loggas men hindrar inte boten från att svara.
- Deltagar‑JID inkluderas automatiskt för gruppreaktioner.
- WhatsApp ignorerar `messages.ackReaction`; använd `channels.whatsapp.ackReaction` i stället.

## Agentverktyg (reaktioner)

- Verktyg: `whatsapp` med åtgärden `react` (`chatJid`, `messageId`, `emoji`, valfri `remove`).
- Valfritt: `participant` (gruppavsändare), `fromMe` (reagera på eget meddelande), `accountId` (multi‑account).
- Semantik för borttagning av reaktioner: se [/tools/reactions](/tools/reactions).
- Verktygsgating: `channels.whatsapp.actions.reactions` (standard: aktiverad).

## Begränsningar

- Utgående text delas upp till `channels.whatsapp.textChunkLimit` (standard 4000).
- Valfri radbrytnings‑chunkning: sätt `channels.whatsapp.chunkMode="newline"` för att dela på tomma rader (styckegränser) före längd‑chunkning.
- Sparade inkommande media begränsas av `channels.whatsapp.mediaMaxMb` (standard 50 MB).
- Utgående mediaobjekt begränsas av `agents.defaults.mediaMaxMb` (standard 5 MB).

## Utgående sändning (text + media)

- Använder aktiv webblyssnare; fel om gatewayen inte körs.
- Text‑chunkning: max 4k per meddelande (konfigurerbart via `channels.whatsapp.textChunkLimit`, valfri `channels.whatsapp.chunkMode`).
- Media:
  - Bild/video/ljud/dokument stöds.
  - Ljud skickas som PTT; `audio/ogg` ⇒ `audio/ogg; codecs=opus`.
  - Bildtext endast på första mediaobjektet.
  - Mediahämtning stöder HTTP(S) och lokala sökvägar.
  - Animerade GIF: WhatsApp förväntar sig MP4 med `gifPlayback: true` för inline‑loopning.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send`‑parametrar inkluderar `gifPlayback: true`

## Röstmeddelanden (PTT‑ljud)

WhatsApp skickar ljud som **röstmeddelanden** (PTT‑bubbla).

- Bästa resultat: OGG/Opus. OpenClaw skriver om `audio/ogg` till `audio/ogg; codecs=opus`.
- `[[audio_as_voice]]` ignoreras för WhatsApp (ljud skickas redan som röstmeddelande).

## Mediebegränsningar + optimering

- Standardtak utgående: 5 MB (per mediaobjekt).
- Åsidosättning: `agents.defaults.mediaMaxMb`.
- Bilder optimeras automatiskt till JPEG under taket (storleksändring + kvalitets‑svep).
- För stora media ⇒ fel; mediasvar faller tillbaka till textvarning.

## Heartbeats

- **Gateway‑heartbeat** loggar anslutningshälsa (`web.heartbeatSeconds`, standard 60 s).
- **Agent‑heartbeat** kan konfigureras per agent (`agents.list[].heartbeat`) eller globalt
  via `agents.defaults.heartbeat` (reserv när inga per‑agent‑poster är satta).
  - Använder den konfigurerade hjärtslagsprompten (standard: `Read HEARTBEAT.md om den existerar (arbetsytans sammanhang). Följ den strikt. Sluta inte eller upprepa gamla uppgifter från tidigare chattar. Om inget behöver uppmärksamhet, svara HEARTBEAT_OK.`) + `HEARTBEAT_OK` hoppa över beteende.
  - Leverans går som standard till senast använda kanal (eller konfigurerat mål).

## Återanslutningsbeteende

- Backoff‑policy: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- Om maxAttempts nås stoppas webbövervakning (degraderat läge).
- Utloggad ⇒ stoppa och kräv återlänkning.

## Snabbkarta för konfig

- `channels.whatsapp.dmPolicy` (DM‑policy: parning/allowlist/öppen/inaktiverad).
- `channels.whatsapp.selfChatMode` (samma‑telefon‑setup; boten använder ditt personliga WhatsApp‑nummer).
- `channels.whatsapp.allowFrom` (DM allowlist). WhatsApp använder E.164 telefonnummer (inga användarnamn).
- `channels.whatsapp.mediaMaxMb` (tak för sparad inkommande media).
- `channels.whatsapp.ackReaction` (auto‑reaktion vid mottagning: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (inställningar per konto + valfria `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (per konto inkommande media cap).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (reaktionsförflyttning per konto)
- `channels.whatsapp.groupAllowFrom` (allowlist för gruppavsändare).
- `channels.whatsapp.groupPolicy` (gruppolicy).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (grupp historia sammanhang; `0` inaktiveras).
- `channels.whatsapp.dmHistoryLimit` (DM historikgräns i användarvänder). Åsidosättningar per användare: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (grupp‑allowlist + standard för mention‑gating; använd `"*"` för att tillåta alla)
- `channels.whatsapp.actions.reactions` (grind för WhatsApp‑verktygsreaktioner).
- `agents.list[].groupChat.mentionPatterns` (eller `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (inkommande prefix; per-account: `channels.whatsapp.accounts.<accountId>.messagePrefix`; föråldrat: `messages.messagePrefix`)
- `messages.responsePrefix` (utgående prefix)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (valfri åsidosättning)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (åsidosättningar per agent)
- `session.*` (scope, idle, store, mainKey)
- `web.enabled` (inaktivera kanalstart när false)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Loggar + felsökning

- Delssystem: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Loggfil: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (konfigurerbar).
- Felsökningsguide: [Gateway troubleshooting](/gateway/troubleshooting).

## Felsökning (snabb)

**Inte länkad / QR‑inloggning krävs**

- Symptom: `channels status` visar `linked: false` eller varnar ”Not linked”.
- Åtgärd: kör `openclaw channels login` på gateway‑värden och skanna QR‑koden (WhatsApp → Inställningar → Länkade enheter).

**Länkad men frånkopplad / återanslutningsloop**

- Symptom: `channels status` visar `running, disconnected` eller varnar ”Linked but disconnected”.
- Fix: `openclaw doctor` (eller starta om gateway). Om det kvarstår, länka tillbaka via `kanalers inloggning` och inspektera `openclaw loggar --follow`.

**Bun‑runtime**

- Bun är **inte rekommenderas**. WhatsApp (Baileys) och Telegram är opålitliga på Bun.
  Kör gateway med **Node**. (Se Kom igång runtime note.)

