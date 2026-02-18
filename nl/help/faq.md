---
title: "FAQ"
---

# FAQ

Snelle antwoorden plus diepere probleemoplossing voor praktijksituaties (lokale dev, VPS, multi‑agent, OAuth/API‑sleutels, model‑failover). Voor runtime‑diagnostiek, zie [Problemen oplossen](/gateway/troubleshooting). Voor de volledige config‑referentie, zie [Configuratie](/gateway/configuration).

## Inhoudsopgave

- [Snelle start en eerste installatie]
  - [Ik zit vast, wat is de snelste manier om los te komen?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Wat is de aanbevolen manier om OpenClaw te installeren en in te stellen?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Hoe open ik het dashboard na onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Hoe authenticeer ik het dashboard (token) op localhost versus remote?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Welke runtime heb ik nodig?](#what-runtime-do-i-need)
  - [Draait het op Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [Tips voor Raspberry Pi‑installaties?](#any-tips-for-raspberry-pi-installs)
  - [Het blijft hangen op "wake up my friend" / onboarding komt niet door. Wat nu?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Kan ik mijn setup migreren naar een nieuwe machine (Mac mini) zonder onboarding opnieuw te doen?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Waar zie ik wat er nieuw is in de laatste versie?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Ik kan docs.openclaw.ai niet openen (SSL‑fout). Wat nu?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Wat is het verschil tussen stable en beta?](#whats-the-difference-between-stable-and-beta)
  - [Hoe installeer ik de beta‑versie, en wat is het verschil tussen beta en dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Hoe probeer ik de nieuwste bits?](#how-do-i-try-the-latest-bits)
  - [Hoe lang duren installatie en onboarding meestal?](#how-long-does-install-and-onboarding-usually-take)
  - [Installer vastgelopen? Hoe krijg ik meer feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows‑installatie zegt git niet gevonden of openclaw niet herkend](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [De docs beantwoorden mijn vraag niet — hoe krijg ik een beter antwoord?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Hoe installeer ik OpenClaw op Linux?](#how-do-i-install-openclaw-on-linux)
  - [Hoe installeer ik OpenClaw op een VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Waar zijn de cloud/VPS‑installatiegidsen?](#where-are-the-cloudvps-install-guides)
  - [Kan ik OpenClaw vragen zichzelf te updaten?](#can-i-ask-openclaw-to-update-itself)
  - [Wat doet de onboarding‑wizard eigenlijk?](#what-does-the-onboarding-wizard-actually-do)
  - [Heb ik een Claude‑ of OpenAI‑abonnement nodig om dit te draaien?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Kan ik een Claude Max‑abonnement gebruiken zonder API‑sleutel](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Hoe werkt Anthropic "setup-token"‑auth?](#how-does-anthropic-setuptoken-auth-work)
  - [Waar vind ik een Anthropic setup-token?](#where-do-i-find-an-anthropic-setuptoken)
  - [Ondersteunen jullie Claude‑abonnementsauth (Claude Pro of Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Waarom zie ik `HTTP 429: rate_limit_error` van Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Wordt AWS Bedrock ondersteund?](#is-aws-bedrock-supported)
  - [Hoe werkt Codex‑auth?](#how-does-codex-auth-work)
  - [Ondersteunen jullie OpenAI‑abonnementsauth (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Hoe stel ik Gemini CLI OAuth in](#how-do-i-set-up-gemini-cli-oauth)
  - [Is een lokaal model oké voor casual chats?](#is-a-local-model-ok-for-casual-chats)
  - [Hoe houd ik gehost modelverkeer in een specifieke regio?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Moet ik een Mac Mini kopen om dit te installeren?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Heb ik een Mac mini nodig voor iMessage‑ondersteuning?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Als ik een Mac mini koop om OpenClaw te draaien, kan ik die verbinden met mijn MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Kan ik Bun gebruiken?](#can-i-use-bun)
  - [Telegram: wat hoort er in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Kunnen meerdere mensen één WhatsApp‑nummer gebruiken met verschillende OpenClaw‑instanties?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Kan ik een "fast chat"‑agent en een "Opus for coding"‑agent draaien?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Werkt Homebrew op Linux?](#does-homebrew-work-on-linux)
  - [Wat is het verschil tussen de hackable (git)‑installatie en npm‑installatie?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Kan ik later wisselen tussen npm‑ en git‑installaties?](#can-i-switch-between-npm-and-git-installs-later)
  - [Moet ik de Gateway op mijn laptop of op een VPS draaien?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [Hoe belangrijk is het om OpenClaw op een dedicated machine te draaien?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Wat zijn de minimale VPS‑vereisten en aanbevolen OS?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Kan ik OpenClaw in een VM draaien en wat zijn de vereisten](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [Wat is OpenClaw?](#what-is-openclaw)
  - [Wat is OpenClaw, in één alinea?](#what-is-openclaw-in-one-paragraph)
  - [Wat is de waardepropositie?](#whats-the-value-proposition)
  - [Ik heb het net opgezet, wat moet ik eerst doen](#i-just-set-it-up-what-should-i-do-first)
  - [Wat zijn de vijf belangrijkste dagelijkse use‑cases voor OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [Kan OpenClaw helpen met lead‑gen, outreach, advertenties en blogs voor een SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Wat zijn de voordelen ten opzichte van Claude Code voor webontwikkeling?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills en automatisering](#skills-and-automation)
- [Sandboxing en geheugen](#sandboxing-and-memory)
- [Waar dingen op schijf staan](#where-things-live-on-disk)
- [Config‑basis](#config-basics)
- [Remote gateways en nodes](#remote-gateways-and-nodes)
- [Env vars en .env‑laden](#env-vars-and-env-loading)
- [Sessies en meerdere chats](#sessions-and-multiple-chats)
- [Modellen: standaardwaarden, selectie, aliassen, wisselen](#models-defaults-selection-aliases-switching)
- [Model‑failover en "All models failed"](#model-failover-and-all-models-failed)
- [Auth‑profielen: wat ze zijn en hoe je ze beheert](#auth-profiles-what-they-are-and-how-to-manage-them)
- [Gateway: poorten, "already running" en remote mode](#gateway-ports-already-running-and-remote-mode)
- [Logging en debuggen](#logging-and-debugging)
- [Media en bijlagen](#media-and-attachments)
- [Beveiliging en toegangsbeheer](#security-and-access-control)
- [Chatopdrachten, taken afbreken en "het stopt niet"](#chat-commands-aborting-tasks-and-it-wont-stop)

## Eerste 60 seconden als er iets kapot is

1. **Snelle status (eerste check)**

   ```bash
   openclaw status
   ```

   Snelle lokale samenvatting: OS + update, bereikbaarheid van gateway/service, agents/sessies, providerconfig + runtime‑problemen (wanneer de gateway bereikbaar is).

2. **Plakbaar rapport (veilig om te delen)**

   ```bash
   openclaw status --all
   ```

   Read‑only diagnose met log‑tail (tokens gemaskeerd).

3. **Daemon + poortstatus**

   ```bash
   openclaw gateway status
   ```

   Toont supervisor‑runtime versus RPC‑bereikbaarheid, de probe‑doel‑URL en welke config de service waarschijnlijk gebruikte.

4. **Diepe probes**

   ```bash
   openclaw status --deep
   ```

   Draait gateway‑healthchecks + provider‑probes (vereist een bereikbare gateway). Zie [Health](/gateway/health).

5. **Tail de nieuwste log**

   ```bash
   openclaw logs --follow
   ```

   Als RPC down is, val terug op:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   Bestandslogs staan los van servicelogs; zie [Logging](/logging) en [Problemen oplossen](/gateway/troubleshooting).

6. **Run de doctor (reparaties)**

   ```bash
   openclaw doctor
   ```

   Repareert/migreert config/state + draait healthchecks. Zie [Doctor](/gateway/doctor).

7. **Gateway‑snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   Vraagt de draaiende gateway om een volledige snapshot (alleen WS). Zie [Health](/gateway/health).

## Snelle start en eerste installatie

### Ik zit vast, wat is de snelste manier om los te komen?

Gebruik een lokale AI‑agent die **je machine kan zien**. Dat is veel effectiever dan vragen in Discord, omdat de meeste "ik zit vast"‑gevallen **lokale config‑ of omgevingsproblemen** zijn die helpers op afstand niet kunnen inspecteren.

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

Geef ze de **volledige broncheckout** via de hackable (git)‑installatie:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Begin met deze opdrachten (deel uitvoer wanneer je om hulp vraagt):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Snelle debug‑lus: [Eerste 60 seconden als er iets kapot is](#eerste-60-seconden-als-er-iets-kapot-is).

---

## Beantwoord de exacte vraag uit het schermafbeeld/chatlogboek

**Q: "Wat is het standaardmodel voor Anthropic met een API-sleutel?"**

**A:** In OpenClaw zijn referenties en modelselectie gescheiden. Het instellen van `ANTHROPIC_API_KEY` (of het opslaan van een Anthropic API‑sleutel in auth‑profielen) schakelt authenticatie in, maar het daadwerkelijke standaardmodel is wat je configureert in `agents.defaults.model.primary` (bijvoorbeeld `anthropic/claude-sonnet-4-5` of `anthropic/claude-opus-4-6`).  

Als je `No credentials found for profile "anthropic:default"` ziet, betekent dit dat de Gateway geen Anthropic‑referenties kon vinden in het verwachte `auth-profiles.json` voor de agent die draait.

---

Nog steeds vast? Vraag het in [Discord](https://discord.com/invite/clawd) of open een [GitHub‑discussie](https://github.com/openclaw/openclaw/discussions).