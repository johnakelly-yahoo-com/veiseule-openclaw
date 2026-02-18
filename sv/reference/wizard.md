---
title: "Referens för introduktionsguide"
sidebarTitle: "Referens för guiden"
---

# Referens för introduktionsguide

Detta är den fullständiga referensen för `openclaw onboard` CLI-guiden.
För en överblick på hög nivå, se [Onboarding Wizard](/start/wizard).

## Flödesdetaljer (lokalt läge)

<Steps>
  <Step title="Existing config detection">
    - Om `~/.openclaw/openclaw.json` finns, välj **Behåll / Ändra / Återställ**.
    - Att köra om guiden raderar **inte** något om du inte uttryckligen väljer **Återställ**
      (eller skickar `--reset`).
    - Om konfigurationen är ogiltig eller innehåller äldre nycklar, stannar guiden och ber
      dig köra `openclaw doctor` innan du fortsätter.
    - Återställ använder `trash` (aldrig `rm`) och erbjuder följande omfattning:
      - Endast konfiguration
      - Konfiguration + autentiseringsuppgifter + sessioner
      - Fullständig återställning (tar även bort arbetsytan)
  
</Step>
  <Step title="Model/Auth">
    - **Anthropic API-nyckel (rekommenderas)**: använder `ANTHROPIC_API_KEY` om den finns eller ber om en nyckel och sparar den för daemon-användning.
    - **Anthropic OAuth (Claude Code CLI)**: på macOS kontrollerar guiden nyckelringselementet "Claude Code-credentials" (välj "Always Allow" så att launchd-starter inte blockeras); på Linux/Windows återanvänds `~/.claude/.credentials.json` om den finns.
    - **Anthropic token (klistra in setup-token)**: kör `claude setup-token` på valfri maskin och klistra sedan in token (du kan namnge den; tomt = standard).
    - **OpenAI Code (Codex) subscription (Codex CLI)**: om `~/.codex/auth.json` finns kan guiden återanvända den.
    - **OpenAI Code (Codex) subscription (OAuth)**: webbläsarflöde; klistra in `code#state`.
      - Sätter `agents.defaults.model` till `openai-codex/gpt-5.2` när modellen är unset eller `openai/*`.
    - **OpenAI API-nyckel**: använder `OPENAI_API_KEY` om den finns eller ber om en nyckel och sparar den i `~/.openclaw/.env` så att launchd kan läsa den.
    - **xAI (Grok) API-nyckel**: ber om `XAI_API_KEY` och konfigurerar xAI som modellleverantör.
    - **OpenCode Zen (multi-model proxy)**: ber om `OPENCODE_API_KEY` (eller `OPENCODE_ZEN_API_KEY`, hämta den på https://opencode.ai/auth).
    - **API-nyckel**: lagrar nyckeln åt dig.
    - **Vercel AI Gateway (multi-model proxy)**: ber om `AI_GATEWAY_API_KEY`.
    - Mer information: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: ber om Account ID, Gateway ID och `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    - Mer information: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: konfiguration skrivs automatiskt.
    - Mer information: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-kompatibel)**: ber om `SYNTHETIC_API_KEY`.
    - Mer information: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: konfiguration skrivs automatiskt.
    - **Kimi Coding**: konfiguration skrivs automatiskt.
    - Mer information: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: ingen autentisering konfigurerad ännu.
    - Välj en standardmodell från upptäckta alternativ (eller ange leverantör/modell manuellt).
    - Guiden kör en modellkontroll och varnar om den konfigurerade modellen är okänd eller saknar autentisering.
    - OAuth-autentiseringsuppgifter finns i `~/.openclaw/credentials/oauth.json`; auth-profiler finns i `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API-nycklar + OAuth).
    - Mer information: [/concepts/oauth](/concepts/oauth)
    <Note>
    Tips för headless/server: slutför OAuth på en maskin med webbläsare och kopiera
    `~/.openclaw/credentials/oauth.json` (eller `$OPENCLAW_STATE_DIR/credentials/oauth.json`) till
    gateway-värden.
    
</Note>
  
</Step>
  <Step title="Workspace">
    - Standard är `~/.openclaw/workspace` (kan konfigureras).
    - Skapar de arbetsytefiler som behövs för agentens bootstrap-ritual.
    - Fullständig arbetsytelayout + guide för säkerhetskopiering: [Agent workspace](/concepts/agent-workspace)
  
</Step>
  <Step title="Gateway">
    - Port, bindning, autentiseringsläge, Tailscale-exponering.
    - Rekommendation: behåll **Token** även för loopback så att lokala WS-klienter måste autentisera.
    - Inaktivera autentisering endast om du helt litar på alla lokala processer.
    - Icke‑loopback-bindningar kräver fortfarande autentisering.
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): valfri QR-inloggning.
    - [Telegram](/channels/telegram): bot-token.
    - [Discord](/channels/discord): bot-token.
    - [Google Chat](/channels/googlechat): servicekonto-JSON + webhook audience.
    - [Mattermost](/channels/mattermost) (plugin): bot-token + bas-URL.
    - [Signal](/channels/signal): valfri installation av `signal-cli` + kontokonfiguration.
    - [BlueBubbles](/channels/bluebubbles): **rekommenderas för iMessage**; server-URL + lösenord + webhook.
    - [iMessage](/channels/imessage): äldre `imsg` CLI-sökväg + DB-åtkomst.
    - DM-säkerhet: standard är parkoppling. Första DM skickar en kod; godkänn via `openclaw pairing approve <channel> <code>` eller använd allowlists.
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Kräver en inloggad användarsession; för headless, använd en anpassad LaunchDaemon (medföljer inte).
    - Linux (och Windows via WSL2): systemd user unit
      - Guiden försöker aktivera lingering via `loginctl enable-linger <user>` så att Gateway fortsätter köra efter utloggning.
      - Kan be om sudo (skriver `/var/lib/systemd/linger`); den försöker utan sudo först.
    - **Val av runtime:** Node (rekommenderas; krävs för WhatsApp/Telegram). Bun är **inte rekommenderat**.
  
</Step>
  <Step title="Health check">
    - Startar Gateway (vid behov) och kör `openclaw health`.
    - Tips: `openclaw status --deep` lägger till gateway-hälsokontroller i statusutdata (kräver en nåbar gateway).
  
</Step>
  <Step title="Skills (recommended)">
    - Läser tillgängliga skills och kontrollerar krav.
    - Låter dig välja en node manager: **npm / pnpm** (bun rekommenderas inte).
    - Installerar valfria beroenden (vissa använder Homebrew på macOS).
  
</Step>
  <Step title="Finish">
    - Sammanfattning + nästa steg, inklusive iOS/Android/macOS-appar för extra funktioner.
  
</Step>
</Steps>

<Note>
Om inget GUI upptäcks skriver guiden ut SSH port-forward-instruktioner för Control UI istället för att öppna en webbläsare.
Om Control UI-resurser saknas försöker guiden bygga dem; fallback är `pnpm ui:build` (installerar UI-beroenden automatiskt).
</Note>

## Icke-interaktivt läge

Använd `--non-interactive` för att automatisera eller skripta introduktionen:

```bash
openclaw onboard --non-interactive \
  --mode local \
  --auth-choice apiKey \
  --anthropic-api-key "$ANTHROPIC_API_KEY" \
  --gateway-port 18789 \
  --gateway-bind loopback \
  --install-daemon \
  --daemon-runtime node \
  --skip-skills
```

Lägg till `--json` för en maskinläsbar sammanfattning.

<Note>
`--json` innebär **inte** icke-interaktivt läge. Använd `--non-interactive` (och `--workspace`) för skript.
</Note>

<AccordionGroup>
  <Accordion title="Gemini example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice gemini-api-key \
      --gemini-api-key "$GEMINI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Z.AI example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice zai-api-key \
      --zai-api-key "$ZAI_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Vercel AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice ai-gateway-api-key \
      --ai-gateway-api-key "$AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Cloudflare AI Gateway example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice cloudflare-ai-gateway-api-key \
      --cloudflare-ai-gateway-account-id "your-account-id" \
      --cloudflare-ai-gateway-gateway-id "your-gateway-id" \
      --cloudflare-ai-gateway-api-key "$CLOUDFLARE_AI_GATEWAY_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Moonshot example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice moonshot-api-key \
      --moonshot-api-key "$MOONSHOT_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="Synthetic example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice synthetic-api-key \
      --synthetic-api-key "$SYNTHETIC_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
  <Accordion title="OpenCode Zen example">
    ```bash
    openclaw onboard --non-interactive \
      --mode local \
      --auth-choice opencode-zen \
      --opencode-zen-api-key "$OPENCODE_API_KEY" \
      --gateway-port 18789 \
      --gateway-bind loopback
    ```
  
</Accordion>
</AccordionGroup>

### Lägg till agent (icke-interaktivt)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

Gateway exponerar wizard-flödet över RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Klienter (macOS-app, Control UI) kan rendera steg utan att återimplementera onboarding-logiken.

## Signal setup (signal-cli)

Guiden kan installera `signal-cli` från GitHub-releaser:

- Laddar ner lämplig release‑asset.
- Lagrar den under `~/.openclaw/tools/signal-cli/<version>/`.
- Skriver `channels.signal.cliPath` till din konfiguration.

Observera:

- JVM‑byggen kräver **Java 21**.
- Native‑byggen används när de finns tillgängliga.
- Windows använder WSL2; installation av signal-cli följer Linux-flödet inuti WSL.

## Vad guiden skriver

Typiska fält i `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (om Minimax valts)
- `gateway.*` (läge, bindning, autentisering, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Kanal‑allowlists (Slack/Discord/Matrix/Microsoft Teams) när du väljer det under stegen (namn löses till ID:n när möjligt).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

`openclaw agents add` skriver `agents.list[]` och valfria `bindings`.

WhatsApp‑uppgifter lagras under `~/.openclaw/credentials/whatsapp/<accountId>/`.
Sessioner lagras under `~/.openclaw/agents/<agentId>/sessions/`.

Vissa kanaler levereras som plugins. När du väljer en under onboarding kommer guiden
att be dig installera den (npm eller en lokal sökväg) innan den kan konfigureras.

## Relaterad dokumentation

- Guideöversikt: [Introduktionsguide](/start/wizard)
- Onboarding i macOS-appen: [Onboarding](/start/onboarding)
- Konfigurationsreferens: [Gateway configuration](/gateway/configuration)
- Leverantörer: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)

