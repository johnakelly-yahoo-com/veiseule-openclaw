---
title: "Sanggunian ng Onboarding Wizard"
sidebarTitle: "Sanggunian ng Wizard"
---

# Sanggunian ng Onboarding Wizard

Ito ang buong reference para sa `openclaw onboard` CLI wizard.
Para sa high-level na overview, tingnan ang [Onboarding Wizard](/start/wizard).

## Mga detalye ng daloy (local mode)

<Steps>
  <Step title="Existing config detection">
    - Kung umiiral ang `~/.openclaw/openclaw.json`, piliin ang **Panatilihin / Baguhin / I-reset**.
    - Ang muling pagpapatakbo ng wizard ay **hindi** nagbubura ng anuman maliban kung tahasan mong piliin ang **Reset**
      (o ipasa ang `--reset`).
    - Kung ang config ay invalid o naglalaman ng legacy keys, hihinto ang wizard at hihilingin
      na patakbuhin mo ang `openclaw doctor` bago magpatuloy.
    - Ang pag-reset ay gumagamit ng `trash` (hindi kailanman `rm`) at nag-aalok ng mga saklaw:
      - Config lamang
      - Config + mga kredensyal + mga session
      - Buong pag-reset (inaalis din ang workspace)
  
</Step>
  <Step title="Model/Auth">
    - **Anthropic API key (inirerekomenda)**: ginagamit ang `ANTHROPIC_API_KEY` kung mayroon o hihiling ng key, pagkatapos ay ise-save ito para sa paggamit ng daemon.
    - **Anthropic OAuth (Claude Code CLI)**: sa macOS, sinusuri ng wizard ang Keychain item na "Claude Code-credentials" (piliin ang "Always Allow" upang hindi maharang ang pagsisimula ng launchd); sa Linux/Windows, muling ginagamit ang `~/.claude/.credentials.json` kung mayroon.
    - **Anthropic token (paste setup-token)**: patakbuhin ang `claude setup-token` sa anumang machine, pagkatapos ay i-paste ang token (maaari mo itong pangalanan; blank = default).
    - **OpenAI Code (Codex) subscription (Codex CLI)**: kung umiiral ang `~/.codex/auth.json`, maaaring muling gamitin ito ng wizard.
    - **OpenAI Code (Codex) subscription (OAuth)**: browser flow; i-paste ang `code#state`.
      - Itinatakda ang `agents.defaults.model` sa `openai-codex/gpt-5.2` kapag ang model ay hindi nakatakda o `openai/*`.
    - **OpenAI API key**: ginagamit ang `OPENAI_API_KEY` kung naroon o hihingi ng key, pagkatapos ay ise-save ito sa `~/.openclaw/.env` upang mabasa ng launchd.
    - **xAI (Grok) API key**: hihingi ng `XAI_API_KEY` at iko-configure ang xAI bilang model provider.
    - **OpenCode Zen (multi-model proxy)**: hihingi ng `OPENCODE_API_KEY` (o `OPENCODE_ZEN_API_KEY`, kunin ito sa https://opencode.ai/auth).
    - **API key**: ise-save ang key para sa iyo.
    - **Vercel AI Gateway (multi-model proxy)**: hihingi ng `AI_GATEWAY_API_KEY`.
    - Higit pang detalye: [Vercel AI Gateway](/providers/vercel-ai-gateway)
    - **Cloudflare AI Gateway**: hihingi ng Account ID, Gateway ID, at `CLOUDFLARE_AI_GATEWAY_API_KEY`.
    - Higit pang detalye: [Cloudflare AI Gateway](/providers/cloudflare-ai-gateway)
    - **MiniMax M2.1**: awtomatikong isinusulat ang config.
    - Higit pang detalye: [MiniMax](/providers/minimax)
    - **Synthetic (Anthropic-compatible)**: hihingi ng `SYNTHETIC_API_KEY`.
    - Higit pang detalye: [Synthetic](/providers/synthetic)
    - **Moonshot (Kimi K2)**: awtomatikong isinusulat ang config.
    - **Kimi Coding**: awtomatikong isinusulat ang config.
    - Higit pang detalye: [Moonshot AI (Kimi + Kimi Coding)](/providers/moonshot)
    - **Skip**: wala pang naka-configure na auth.
    - Pumili ng default model mula sa mga natukoy na opsyon (o manu-manong ilagay ang provider/model).
    - Nagpapatakbo ang wizard ng model check at nagbababala kung ang naka-configure na model ay hindi kilala o kulang sa auth.
    - Ang mga OAuth credential ay nasa `~/.openclaw/credentials/oauth.json`; ang mga auth profile ay nasa `~/.openclaw/agents/<agentId>/agent/auth-profiles.json` (API keys + OAuth).
    - Mas detalyado: [/concepts/oauth](/concepts/oauth)
    <Note>
    Tip para sa headless/server: kumpletuhin ang OAuth sa isang machine na may browser, pagkatapos ay kopyahin
    ang `~/.openclaw/credentials/oauth.json` (o `$OPENCLAW_STATE_DIR/credentials/oauth.json`) papunta sa
    host ng Gateway.
    
</Note>
  
</Step>
  <Step title="Workspace">
    - Default `~/.openclaw/workspace` (configurable).
    - Inilalagay ang mga workspace file na kailangan para sa agent bootstrap ritual.
    - Buong layout ng workspace + gabay sa backup: [Agent workspace](/concepts/agent-workspace)
  
</Step>
  <Step title="Gateway">
    - Port, bind, auth mode, tailscale exposure.
    - Rekomendasyon sa auth: panatilihin ang **Token** kahit para sa loopback upang ang mga lokal na WS client ay kailangang mag-authenticate.
    - I-disable lamang ang auth kung lubos mong pinagkakatiwalaan ang bawat lokal na proseso.
    - Ang mga non‑loopback bind ay nangangailangan pa rin ng auth.
  
</Step>
  <Step title="Channels">
    - [WhatsApp](/channels/whatsapp): optional QR login.
    - [Telegram](/channels/telegram): bot token.
    - [Discord](/channels/discord): bot token.
    - [Google Chat](/channels/googlechat): service account JSON + webhook audience.
    - [Mattermost](/channels/mattermost) (plugin): bot token + base URL.
    - [Signal](/channels/signal): optional `signal-cli` install + account config.
    - [BlueBubbles](/channels/bluebubbles): **recommended for iMessage**; server URL + password + webhook.
    - [iMessage](/channels/imessage): legacy `imsg` CLI path + DB access.
    - DM security: default ay pairing. Ang unang DM ay nagpapadala ng code; aprubahan gamit ang `openclaw pairing approve <channel> <code>` o gumamit ng mga allowlist.
  
</Step>
  <Step title="Daemon install">
    - macOS: LaunchAgent
      - Nangangailangan ng naka-login na user session; para sa headless, gumamit ng custom LaunchDaemon (hindi kasama).
    - Linux (at Windows sa pamamagitan ng WSL2): systemd user unit
      - Sinusubukan ng wizard na i-enable ang lingering sa pamamagitan ng `loginctl enable-linger <user>` upang manatiling nakaandar ang Gateway pagkatapos mag-logout.
      - Maaaring humingi ng sudo (nagsusulat sa `/var/lib/systemd/linger`); sinusubukan muna nito nang walang sudo.
    - **Runtime selection:** Node (inirerekomenda; kinakailangan para sa WhatsApp/Telegram). Ang Bun ay **hindi inirerekomenda**.
  
</Step>
  <Step title="Health check">
    - Sinisimulan ang Gateway (kung kinakailangan) at pinapatakbo ang `openclaw health`.
    - Tip: ang `openclaw status --deep` ay nagdaragdag ng gateway health probes sa status output (nangangailangan ng reachable na gateway).
  
</Step>
  <Step title="Skills (recommended)">
    - Binabasa ang mga available na skills at sinusuri ang mga requirement.
    - Hinahayaan kang pumili ng node manager: **npm / pnpm** (hindi inirerekomenda ang bun).
    - Ini-install ang mga optional dependency (ang ilan ay gumagamit ng Homebrew sa macOS).
  
</Step>
  <Step title="Finish">
    - Buod + mga susunod na hakbang, kabilang ang iOS/Android/macOS apps para sa dagdag na feature.
  
</Step>
</Steps>

<Note>
Kung walang na-detect na GUI, ipi-print ng wizard ang SSH port-forward instructions para sa Control UI sa halip na magbukas ng browser.
Kung nawawala ang Control UI assets, susubukan ng wizard na i-build ang mga ito; fallback ay `pnpm ui:build` (awtomatikong ini-install ang UI deps).
</Note>

## Non-interactive mode

Gamitin ang `--non-interactive` upang i-automate o i-script ang onboarding:

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

Idagdag ang `--json` para sa machine‑readable na buod.

<Note>
Ang `--json` ay **hindi** nangangahulugang non-interactive mode. Gamitin ang `--non-interactive` (at `--workspace`) para sa mga script.
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

### Magdagdag ng agent (non-interactive)

```bash
openclaw agents add work \
  --workspace ~/.openclaw/workspace-work \
  --model openai/gpt-5.2 \
  --bind whatsapp:biz \
  --non-interactive \
  --json
```

## Gateway wizard RPC

Inilalantad ng Gateway ang wizard flow sa pamamagitan ng RPC (`wizard.start`, `wizard.next`, `wizard.cancel`, `wizard.status`).
Maaaring i-render ng mga client (macOS app, Control UI) ang mga hakbang nang hindi muling iniimplementa ang onboarding logic.

## Signal setup (signal-cli)

Maaaring i-install ng wizard ang `signal-cli` mula sa GitHub releases:

- Dina-download ang angkop na release asset.
- Iniimbak ito sa ilalim ng `~/.openclaw/tools/signal-cli/<version>/`.
- Isinusulat ang `channels.signal.cliPath` sa iyong config.

Mga tala:

- Nangangailangan ang JVM builds ng **Java 21**.
- Ginagamit ang native builds kapag available.
- Gumagamit ang Windows ng WSL2; ang pag-install ng signal-cli ay sumusunod sa daloy ng Linux sa loob ng WSL.

## Ano ang isinusulat ng wizard

Mga tipikal na field sa `~/.openclaw/openclaw.json`:

- `agents.defaults.workspace`
- `agents.defaults.model` / `models.providers` (kung Minimax ang pinili)
- `gateway.*` (mode, bind, auth, tailscale)
- `channels.telegram.botToken`, `channels.discord.token`, `channels.signal.*`, `channels.imessage.*`
- Mga channel allowlist (Slack/Discord/Matrix/Microsoft Teams) kapag nag-opt in ka sa mga prompt (ang mga pangalan ay nireresolba sa mga ID kapag posible).
- `skills.install.nodeManager`
- `wizard.lastRunAt`
- `wizard.lastRunVersion`
- `wizard.lastRunCommit`
- `wizard.lastRunCommand`
- `wizard.lastRunMode`

Ang `openclaw agents add` ay nagsusulat ng `agents.list[]` at opsyonal na `bindings`.

Ang mga WhatsApp credential ay nasa `~/.openclaw/credentials/whatsapp/<accountId>/`.
Ang mga session ay nakaimbak sa `~/.openclaw/agents/<agentId>/sessions/`.

Ang ilang channel ay inihahatid bilang mga plugin. Kapag pumili ka ng isa sa onboarding, ipo-prompt ka ng wizard
na i-install ito (npm o local path) bago ito ma-configure.

## Kaugnay na docs

- Pangkalahatang-ideya ng wizard: [Onboarding Wizard](/start/wizard)
- Onboarding ng macOS app: [Onboarding](/start/onboarding)
- Reference ng config: [Gateway configuration](/gateway/configuration)
- Mga provider: [WhatsApp](/channels/whatsapp), [Telegram](/channels/telegram), [Discord](/channels/discord), [Google Chat](/channels/googlechat), [Signal](/channels/signal), [BlueBubbles](/channels/bluebubbles) (iMessage), [iMessage](/channels/imessage) (legacy)
- Skills: [Skills](/tools/skills), [Skills config](/tools/skills-config)

