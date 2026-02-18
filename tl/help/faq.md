---
title: "FAQ"
---

# FAQ

Mga mabilisang sagot kasama ang mas malalim na troubleshooting para sa mga real-world na setup (local dev, VPS, multi-agent, OAuth/API keys, model failover). Para sa runtime diagnostics, tingnan ang [Troubleshooting](/gateway/troubleshooting). Para sa kumpletong sanggunian ng config, tingnan ang [Configuration](/gateway/configuration).

## Talaan ng mga nilalaman

- [Mabilis na pagsisimula at unang setup]
  - [Nastuck ako—ano ang pinakamabilis na paraan para maka-alis?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Ano ang inirerekomendang paraan para i-install at i-set up ang OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Paano ko bubuksan ang dashboard pagkatapos ng onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Paano ko ia-authenticate ang dashboard (token) sa localhost vs remote?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Anong runtime ang kailangan ko?](#what-runtime-do-i-need)
  - [Tumatakbo ba ito sa Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [May mga tip ba para sa Raspberry Pi installs?](#any-tips-for-raspberry-pi-installs)
  - [Naka-stuck ito sa "wake up my friend" / hindi magha-hatch ang onboarding. Ano ngayon?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Maaari ko bang ilipat ang setup ko sa bagong machine (Mac mini) nang hindi inuulit ang onboarding?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Saan ko makikita kung ano ang bago sa pinakabagong bersyon?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Hindi ko ma-access ang docs.openclaw.ai (SSL error). Ano ngayon?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Ano ang pagkakaiba ng stable at beta?](#whats-the-difference-between-stable-and-beta)
  - [Paano ko i-install ang beta version, at ano ang pagkakaiba ng beta at dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Paano ko susubukan ang pinakabagong bits?](#how-do-i-try-the-latest-bits)
  - [Gaano katagal karaniwang tumatagal ang install at onboarding?](#how-long-does-install-and-onboarding-usually-take)
  - [Installer stuck? Paano ako makakakuha ng mas maraming feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Sinasabi ng Windows install na git not found o hindi nakikilala ang openclaw](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [Hindi sinagot ng docs ang tanong ko—paano ako makakakuha ng mas magandang sagot?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Paano ko i-install ang OpenClaw sa Linux?](#how-do-i-install-openclaw-on-linux)
  - [Paano ko i-install ang OpenClaw sa isang VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Saan ang mga cloud/VPS install guides?](#where-are-the-cloudvps-install-guides)
  - [Maaari ko bang utusan ang OpenClaw na i-update ang sarili nito?](#can-i-ask-openclaw-to-update-itself)
  - [Ano ba talaga ang ginagawa ng onboarding wizard?](#what-does-the-onboarding-wizard-actually-do)
  - [Kailangan ko ba ng Claude o OpenAI subscription para patakbuhin ito?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Maaari ko bang gamitin ang Claude Max subscription nang walang API key](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Paano gumagana ang Anthropic "setup-token" auth?](#how-does-anthropic-setuptoken-auth-work)
  - [Saan ko mahahanap ang Anthropic setup-token?](#where-do-i-find-an-anthropic-setuptoken)
  - [Sinusuportahan ba ninyo ang Claude subscription auth (Claude Pro o Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Bakit nakikita ko ang `HTTP 429: rate_limit_error` mula sa Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Sinusuportahan ba ang AWS Bedrock?](#is-aws-bedrock-supported)
  - [Paano gumagana ang Codex auth?](#how-does-codex-auth-work)
  - [Sinusuportahan ba ninyo ang OpenAI subscription auth (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Paano ko ise-set up ang Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  - [OK ba ang local model para sa kaswal na chats?](#is-a-local-model-ok-for-casual-chats)
  - [Paano ko mapapanatili ang hosted model traffic sa isang partikular na rehiyon?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Kailangan ko bang bumili ng Mac Mini para i-install ito?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Kailangan ko ba ng Mac mini para sa iMessage support?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Kung bibili ako ng Mac mini para patakbuhin ang OpenClaw, maaari ko ba itong ikonekta sa MacBook Pro ko?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Maaari ba akong gumamit ng Bun?](#can-i-use-bun)
  - [Telegram: ano ang ilalagay sa `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Maaari bang gumamit ang maraming tao ng iisang WhatsApp number na may iba’t ibang OpenClaw instances?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Maaari ba akong magpatakbo ng "fast chat" agent at "Opus for coding" agent?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Gumagana ba ang Homebrew sa Linux?](#does-homebrew-work-on-linux)
  - [Ano ang pagkakaiba ng hackable (git) install at npm install?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Maaari ba akong lumipat sa pagitan ng npm at git installs sa bandang huli?](#can-i-switch-between-npm-and-git-installs-later)
  - [Dapat ko bang patakbuhin ang Gateway sa laptop ko o sa isang VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [Gaano kahalaga ang pagpapatakbo ng OpenClaw sa isang dedicated machine?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Ano ang minimum na VPS requirements at inirerekomendang OS?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Maaari ba akong magpatakbo ng OpenClaw sa isang VM at ano ang mga requirements](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [Ano ang OpenClaw?](#what-is-openclaw)
- [Skills at automation](#skills-and-automation)
- [Sandboxing at memory](#sandboxing-and-memory)
- [Saan nakatira ang mga bagay sa disk](#where-things-live-on-disk)
- [Mga basic ng config](#config-basics)
- [Remote gateways at nodes](#remote-gateways-and-nodes)
- [Env vars at .env loading](#env-vars-and-env-loading)
- [Sessions at maraming chats](#sessions-and-multiple-chats)
- [Models: mga default, pagpili, aliases, pagpapalit](#models-defaults-selection-aliases-switching)
- [Model failover at "All models failed"](#model-failover-and-all-models-failed)
- [Auth profiles: ano ang mga ito at paano pamahalaan](#auth-profiles-what-they-are-and-how-to-manage-them)
- [Gateway: ports, "already running", at remote mode](#gateway-ports-already-running-and-remote-mode)
- [Logging at debugging](#logging-and-debugging)
- [Media at attachments](#media-and-attachments)
- [Seguridad at kontrol sa access](#security-and-access-control)
- [Chat commands, pag-abort ng tasks, at "hindi ito humihinto"](#chat-commands-aborting-tasks-and-it-wont-stop)

## First 60 seconds if something's broken

1. **Quick status (first check)**

   ```bash
   openclaw status
   ```

2. **Pasteable report (safe to share)**

   ```bash
   openclaw status --all
   ```

3. **Daemon + port state**

   ```bash
   openclaw gateway status
   ```

4. **Deep probes**

   ```bash
   openclaw status --deep
   ```

5. **Tail the latest log**

   ```bash
   openclaw logs --follow
   ```

6. **Run the doctor (repairs)**

   ```bash
   openclaw doctor
   ```

7. **Gateway snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

## Mabilis na pagsisimula at unang setup

### Im stuck whats the fastest way to get unstuck

Gumamit ng lokal na AI agent na kayang **makita ang iyong machine**. Mas epektibo ito kaysa magtanong sa Discord, dahil karamihan sa “I’m stuck” na kaso ay **lokal na config o environment issues** na hindi kayang inspeksyunin ng remote helpers.

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

Ibigay sa kanila ang **full source checkout** gamit ang hackable (git) install:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Magsimula sa:

```bash
openclaw status
openclaw models status
openclaw doctor
```

---

## Sagutin ang eksaktong tanong mula sa screenshot/chat log

**Q: "Ano ang default na modelo para sa Anthropic kapag may API key?"**

**A:** Sa OpenClaw, hiwalay ang credentials at model selection. Ang pagtatakda ng `ANTHROPIC_API_KEY` (o pag-iimbak ng Anthropic API key sa auth profiles) ay nagpapagana ng authentication, ngunit ang aktwal na default model ay kung ano man ang iyong itinakda sa `agents.defaults.model.primary` (halimbawa, `anthropic/claude-sonnet-4-5` o `anthropic/claude-opus-4-6`). Kung makita mo ang `No credentials found for profile "anthropic:default"`, ibig sabihin ay hindi mahanap ng Gateway ang Anthropic credentials sa inaasahang `auth-profiles.json` para sa agent na tumatakbo.

---

Naka-stuck pa rin? Magtanong sa https://discord.com/invite/clawd o magbukas ng discussion sa https://github.com/openclaw/openclaw/discussions.

