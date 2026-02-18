---
title: "CLI‑referens för introduktion"
sidebarTitle: "CLI‑referens"
---

# CLI‑referens för introduktion

Denna sida är den fullständiga referensen för `openclaw onboard`.
För den korta guiden, se [Onboarding Wizard (CLI)](/start/wizard).

## Vad guiden gör

Lokalt läge (standard) leder dig genom:

- Modell- och autentiseringskonfiguration (OpenAI Code‑prenumeration OAuth, Anthropic API‑nyckel eller setup‑token, samt MiniMax, GLM, Moonshot och AI Gateway‑alternativ)
- Arbetsytans plats och bootstrap‑filer
- Gateway‑inställningar (port, bind, auth, Tailscale)
- Kanaler och leverantörer (Telegram, WhatsApp, Discord, Google Chat, Mattermost‑plugin, Signal)
- Installation av daemon (LaunchAgent eller systemd‑användarenhet)
- Hälsokontroll
- Skills‑konfiguration

Fjärrläge konfigurerar den här maskinen för att ansluta till en gateway någon annanstans.
Det installerar eller ändrar inget på fjärrvärden.

## Detaljer för lokalt flöde

<Steps>
  <Step title="Existing config detection">
    - Om `~/.openclaw/openclaw.json` finns, välj Keep, Modifiera eller Återställ.
    - Att köra om guiden torkar inte någonting om du inte uttryckligen väljer Återställ (eller skickar `--reset`).
    - Om konfigurationen är ogiltig eller innehåller äldre nycklar, stannar guiden och ber dig att köra `openclaw doctor` innan du fortsätter.
    - Återställ använder `trash` och erbjuder omfattning:
      - Endast Config
      - Config + autentiseringsuppgifter + sessioner
      - Fullständig återställning (tar också bort arbetsytan)
  
</Step>
  <Step title="Model and auth">
    - Fullständig alternativmatris finns i [Autentiserings- och modellalternativ](#auth-and-model-options).
  
</Step>
  <Step title="Workspace">
    - Standard `~/.openclaw/workspace` (konfigurerbar).
    - Skapar arbetsytefiler som behövs för första körningens bootstrap‑ritual.
    - Arbetsytans layout: [Agent workspace](/concepts/agent-workspace).
  
</Step>
  <Step title="Gateway">
    - Frågar efter port, bind, auth‑läge och Tailscale‑exponering.
    - Rekommenderat: behåll token‑auth aktiverad även för loopback så lokala WS‑klienter måste autentisera.
    - Inaktivera auth endast om du helt litar på varje lokal process.
    - Icke‑loopback‑bind kräver fortfarande auth.
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): valfri QR‑inloggning
    - [Telegram](/channels/telegram): bot‑token
    - [Discord](/channels/discord): bot‑token
    - [Google Chat](/channels/googlechat): servicekonto‑JSON + webhook‑publik
    - [Mattermost](/channels/mattermost) plugin: bot‑token + bas‑URL
    - [Signal](/channels/signal): valfri `signal-cli`‑installation + kontokonfiguration
    - [BlueBubbles](/channels/bluebubbles): rekommenderas för iMessage; server‑URL + lösenord + webhook
    - [iMessage](/channels/imessage): äldre `imsg` CLI‑sökväg + DB‑åtkomst
    - DM‑säkerhet: standard är parkoppling. Första DM skickar en kod; godkänn via
      `openclaw pairing approve <channel> <code>` eller använd tillåtelselistor.
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Kräver inloggad användarsession; för headless, använd en anpassad LaunchDaemon (ingår inte).
    - Linux och Windows via WSL2: systemd‑användarenhet
      - Guiden försöker `loginctl enable-linger <user>` så gatewayen fortsätter köra efter utloggning.
      - Kan be om sudo (skriver `/var/lib/systemd/linger`); den försöker utan sudo först.
    - Val av runtime: Node (rekommenderas; krävs för WhatsApp och Telegram). Bun rekommenderas inte.
  
</Step>
  <Step title="Health check">
    - Startar gateway (vid behov) och kör `openclaw health`.
    - `openclaw status --deep` lägger till gateway‑hälsoprob till statusutdata.
  
</Step>
  <Step title="Skills">
    - Läser tillgängliga skills och kontrollerar krav.
    - Låter dig välja node‑manager: npm eller pnpm (bun rekommenderas inte).
    - Installerar valfria beroenden (vissa använder Homebrew på macOS).
  
</Step>
  <Step title="Finish">
    - Sammanfattning och nästa steg, inklusive iOS‑, Android‑ och macOS‑appalternativ.
  
</Step>
</Steps>

<Note>
Om inget GUI upptäcks skriver guiden ut SSH‑port‑forward‑instruktioner för Control UI istället för att öppna en webbläsare.
Om Control UI‑tillgångar saknas försöker guiden bygga dem; fallback är `pnpm ui:build` (installerar automatiskt UI‑beroenden).
</Note>

## Detaljer för fjärrläge

Fjärrläge konfigurerar den här maskinen för att ansluta till en gateway någon annanstans.

<Info>
Fjärrläge installerar eller ändrar ingenting på fjärrvärden.
</Info>

Det du anger:

- URL till fjärrgateway (`ws://...`)
- Token om fjärrgateway‑auth krävs (rekommenderas)

<Note>
- Om gatewayen är loopback‑endast, använd SSH‑tunnel eller ett tailnet.
- Upptäcktsledtrådar:
  - macOS: Bonjour (`dns-sd`)
  - Linux: Avahi (`avahi-browse`)
</Note>

## Autentiserings- och modellalternativ

<AccordionGroup>
  <Accordion title="Anthropic API key (recommended)">
    Använder `ANTHROPIC_API_KEY` om den finns eller ber om en nyckel och sparar den för daemon‑användning.
  
</Accordion>
  <Accordion title="Anthropic OAuth (Claude Code CLI)">
    - macOS: kontrollerar nyckelringsposten "Claude Code-credentials"
    - Linux och Windows: återanvänder `~/.claude/.credentials.json` om den finns

    På macOS, välj "Always Allow" så att launchd‑starter inte blockeras.

  
</Accordion>
  <Accordion title="Anthropic token (setup-token paste)">
    Kör `claude setup-token` på valfri maskin och klistra sedan in token.
    Du kan namnge den; tomt använder standard.
  
</Accordion>
  <Accordion title="OpenAI Code subscription (Codex CLI reuse)">
    Om `~/.codex/auth.json` finns kan guiden återanvända den.
  
</Accordion>
  <Accordion title="OpenAI Code subscription (OAuth)">
    Webbläsarflöde; klistra in `code#state`.

    Sätter `agents.defaults.model` till `openai-codex/gpt-5.3-codex` när modellen är osatt eller `openai/*`.

  
</Accordion>
  <Accordion title="OpenAI API key">
    Använder `OPENAI_API_KEY` om den finns eller ber om en nyckel och sparar den i
    `~/.openclaw/.env` så att launchd kan läsa den.

    Sätter `agents.defaults.model` till `openai/gpt-5.1-codex` när modellen är osatt, `openai/*` eller `openai-codex/*`.

  
</Accordion>
  <Accordion title="xAI (Grok) API key">
    Ber om `XAI_API_KEY` och konfigurerar xAI som modellleverantör.
  
</Accordion>
  <Accordion title="OpenCode Zen">
    Frågar efter `OPENCODE_API_KEY` (eller `OPENCODE_ZEN_API_KEY`).
    Setup URL: [opencode.ai/auth](https://opencode.ai/auth).
  
</Accordion>
  <Accordion title="API key (generic)">
    Lagrar nyckeln åt dig.
  
</Accordion>
  <Accordion title="Vercel AI Gateway">
    Frågar efter `AI_GATEWAY_API_KEY`.
    Mer information: [Vercel AI Gateway](/providers/vercel-ai-gateway).
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway">
    Frågar efter konto‑ID, gateway‑ID och `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    Mer information: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway).
  
</Accordion>
  <Accordion title="MiniMax M2.1">
    Konfigurationen skrivs automatiskt.
    Mer information: [MiniMax](/providers/minimax).
  
</Accordion>
  <Accordion title="Synthetic (Anthropic-compatible)">
    Frågar efter `SYNTHETIC_API_KEY`.
    Mer information: [Synthetic](/providers/synthetic).
  
</Accordion>
  <Accordion title="Moonshot and Kimi Coding">
    Moonshot (Kimi K2) och Kimi Coding‑konfigurationer skrivs automatiskt.
    Mer information: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot).
  
</Accordion>
  <Accordion title="Custom provider">
    Fungerar med OpenAI‑kompatibla och Anthropic‑kompatibla endpoints.

    Icke‑interaktiva flaggor:
    - `--auth-choice custom-api-key`
    - `--custom-base-url`
    - `--custom-model-id`
    - `--custom-api-key` (valfri; faller tillbaka till `CUSTOM_API_KEY`)
    - `--custom-provider-id` (valfri)
    - `--custom-compatibility <openai|anthropic>` (valfri; standard `openai`)

  
</Accordion>
  <Accordion title="Skip">
    Lämnar autentisering okonfigurerad.
  
</Accordion>
</AccordionGroup>

Modellbeteende:

- Välj standardmodell från upptäckta alternativ eller ange leverantör och modell manuellt.
- Guiden kör en modellkontroll och varnar om den konfigurerade modellen är okänd eller saknar autentisering.

Sökvägar för autentiseringsuppgifter och profiler:

- OAuth‑uppgifter: `~/.openclaw/credentials/oauth.json`
- Autentiseringsprofiler (API‑nycklar + OAuth): `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

<Note>
Tips för headless och server: slutför OAuth på en maskin med webbläsare och kopiera
`~/.openclaw/credentials/oauth.json` (eller `$OPENCLAW_STATE_DIR/credentials/oauth.json`)
till gateway‑värden.
</Note>

## Utdata och internals

Typiska fält i `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (om Minimax valts)
- `gateway.*` (läge, bind, auth, Tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Kanal‑tillåtelselistor (Slack, Discord, Matrix, Microsoft Teams) när du väljer det under frågorna (namn löses till ID när möjligt)
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` skriver `agents.list[]` och valfria `bindings`.

WhatsApp‑uppgifter lagras under `~/.openclaw/credentials/whatsapp/<accountId>/`.
Sessioner lagras under `~/.openclaw/agents/<agentId>/sessions/`.

<Note>
Vissa kanaler levereras som plugins. När de väljs under introduktionen uppmanar guiden
att installera pluginet (npm eller lokal sökväg) innan kanalkonfigurationen.
</Note>

Gateway wizard RPC:

- `wizard.start`
- `wizard.next`
- `wizard.cancel`
- `wizard.status`

Klienter (macOS‑appen och Control UI) kan rendera steg utan att återimplementera introduktionslogiken.

Signal‑konfigurationsbeteende:

- Laddar ner rätt release‑asset
- Lagrar den under `~/.openclaw/tools/signal-cli/<version>/`
- Skriver `channels.signal.cliPath` i konfigurationen
- JVM‑byggen kräver Java 21
- Native‑byggen används när de är tillgängliga
- Windows använder WSL2 och följer Linux signal‑cli‑flödet inuti WSL

## Relaterad dokumentation

- Introduktionsnav: [Onboarding Wizard (CLI)](/start/wizard)
- Automatisering och skript: [CLI Automation](/start/wizard-cli-automation)
- Kommandoreferens: [`openclaw onboard`](/cli/onboard)

