---
title: "FAQ"
---

# FAQ

Kurze Antworten plus vertiefte Fehlerbehebung für reale Setups (lokale Entwicklung, VPS, Multi-Agent, OAuth/API-Schlüssel, Modell-Failover). Für Laufzeitdiagnosen siehe [Troubleshooting](/gateway/troubleshooting). Für die vollständige Konfigurationsreferenz siehe [Configuration](/gateway/configuration).

## Inhaltsverzeichnis

- [Schnellstart und Ersteinrichtung]
  - [Ich stecke fest – was ist der schnellste Weg, wieder weiterzukommen?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Was ist der empfohlene Weg, OpenClaw zu installieren und einzurichten?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Wie öffne ich das Dashboard nach dem Onboarding?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Wie authentifiziere ich das Dashboard (Token) auf localhost vs. remote?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Welche Runtime benötige ich?](#what-runtime-do-i-need)
  - [Läuft es auf einem Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [Gibt es Tipps für Raspberry-Pi-Installationen?](#any-tips-for-raspberry-pi-installs)
  - [Es hängt bei „wake up my friend“ / das Onboarding schlüpft nicht. Was nun?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Kann ich mein Setup auf eine neue Maschine (Mac mini) migrieren, ohne das Onboarding erneut zu machen?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Wo sehe ich, was in der neuesten Version neu ist?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Ich kann docs.openclaw.ai nicht erreichen (SSL-Fehler). Was nun?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Was ist der Unterschied zwischen stable und beta?](#whats-the-difference-between-stable-and-beta)
  - [Wie installiere ich die Beta-Version, und was ist der Unterschied zwischen beta und dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Wie probiere ich die neuesten Bits aus?](#how-do-i-try-the-latest-bits)
  - [Wie lange dauern Installation und Onboarding normalerweise?](#how-long-does-install-and-onboarding-usually-take)
  - [Installer hängt? Wie bekomme ich mehr Feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows-Installation sagt „git nicht gefunden“ oder „openclaw nicht erkannt“](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [Die Doku hat meine Frage nicht beantwortet – wie bekomme ich eine bessere Antwort?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Wie installiere ich OpenClaw unter Linux?](#how-do-i-install-openclaw-on-linux)
  - [Wie installiere ich OpenClaw auf einem VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Wo sind die Cloud-/VPS-Installationsanleitungen?](#where-are-the-cloudvps-install-guides)
  - [Kann ich OpenClaw bitten, sich selbst zu aktualisieren?](#can-i-ask-openclaw-to-update-itself)
  - [Was macht der Onboarding-Assistent eigentlich?](#what-does-the-onboarding-wizard-actually-do)
  - [Benötige ich ein Claude- oder OpenAI-Abonnement, um das zu betreiben?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Kann ich ein Claude-Max-Abonnement ohne API-Schlüssel nutzen?](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Wie funktioniert die Anthropic-„setup-token“-Authentifizierung?](#how-does-anthropic-setuptoken-auth-work)
  - [Wo finde ich ein Anthropic setup-token?](#where-do-i-find-an-anthropic-setuptoken)
  - [Unterstützen Sie Claude-Abonnement-Authentifizierung (Claude Pro oder Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Warum sehe ich `HTTP 429: rate_limit_error` von Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Wird AWS Bedrock unterstützt?](#is-aws-bedrock-supported)
  - [Wie funktioniert die Codex-Authentifizierung?](#how-does-codex-auth-work)
  - [Unterstützen Sie OpenAI-Abonnement-Authentifizierung (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Wie richte ich Gemini CLI OAuth ein?](#how-do-i-set-up-gemini-cli-oauth)
  - [Ist ein lokales Modell für lockere Chats in Ordnung?](#is-a-local-model-ok-for-casual-chats)
  - [Wie halte ich den Traffic zu gehosteten Modellen in einer bestimmten Region?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Muss ich einen Mac mini kaufen, um das zu installieren?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Benötige ich einen Mac mini für iMessage-Unterstützung?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Wenn ich einen Mac mini kaufe, um OpenClaw auszuführen, kann ich ihn mit meinem MacBook Pro verbinden?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Kann ich Bun verwenden?](#can-i-use-bun)
  - [Telegram: Was gehört in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Können mehrere Personen eine WhatsApp-Nummer mit verschiedenen OpenClaw-Instanzen nutzen?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Kann ich einen „Fast-Chat“-Agenten und einen „Opus fürs Coden“-Agenten betreiben?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Funktioniert Homebrew unter Linux?](#does-homebrew-work-on-linux)
  - [Was ist der Unterschied zwischen der hackbaren (git) Installation und der npm-Installation?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Kann ich später zwischen npm- und git-Installationen wechseln?](#can-i-switch-between-npm-and-git-installs-later)
  - [Sollte ich den Gateway auf meinem Laptop oder auf einem VPS betreiben?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [Wie wichtig ist es, OpenClaw auf einer dedizierten Maschine zu betreiben?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Was sind die minimalen VPS-Anforderungen und das empfohlene Betriebssystem?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Kann ich OpenClaw in einer VM betreiben, und welche Anforderungen gibt es?](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [Was ist OpenClaw?](#what-is-openclaw)
  - [Was ist OpenClaw, in einem Absatz?](#what-is-openclaw-in-one-paragraph)
  - [Was ist das Wertversprechen?](#whats-the-value-proposition)
  - [Ich habe es gerade eingerichtet – was sollte ich zuerst tun?](#i-just-set-it-up-what-should-i-do-first)
  - [Was sind die fünf besten alltäglichen Anwendungsfälle für OpenClaw?](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [Kann OpenClaw bei Lead-Gen, Outreach, Anzeigen und Blogs für ein SaaS helfen?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Was sind die Vorteile gegenüber Claude Code für Webentwicklung?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills und Automatisierung](#skills-and-automation)
  - [Wie passe ich Skills an, ohne das Repo „dirty“ zu halten?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Kann ich Skills aus einem benutzerdefinierten Ordner laden?](#can-i-load-skills-from-a-custom-folder)
  - [Wie kann ich verschiedene Modelle für unterschiedliche Aufgaben verwenden?](#how-can-i-use-different-models-for-different-tasks)
  - [Der Bot friert bei schwerer Arbeit ein. Wie lagere ich das aus?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron oder Erinnerungen werden nicht ausgelöst. Was sollte ich prüfen?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Wie installiere ich Skills unter Linux?](#how-do-i-install-skills-on-linux)
  - [Kann OpenClaw Aufgaben nach Zeitplan oder kontinuierlich im Hintergrund ausführen?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Kann ich Apple-macOS-only-Skills von Linux aus verwenden?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Gibt es eine Notion- oder HeyGen-Integration?](#do-you-have-a-notion-or-heygen-integration)
  - [Wie installiere ich die Chrome-Erweiterung für Browser-Takeover?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandboxing und Speicher](#sandboxing-and-memory)
  - [Gibt es eine dedizierte Sandbox-Dokumentation?](#is-there-a-dedicated-sandboxing-doc)
  - [Wie binde ich einen Host-Ordner in die Sandbox ein?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Wie funktioniert Speicher?](#how-does-memory-work)
  - [Der Speicher vergisst Dinge. Wie mache ich sie dauerhaft?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Bleibt Speicher für immer erhalten? Was sind die Grenzen?](#does-memory-persist-forever-what-are-the-limits)
  - [Benötigt die semantische Speichersuche einen OpenAI-API-Schlüssel?](#does-semantic-memory-search-require-an-openai-api-key)

## First 60 seconds if something's broken

1. **Quick status (first check)**

   ```bash
   openclaw status
   ```

   Schnelle lokale Übersicht: OS + Update, Gateway-/Service-Erreichbarkeit, Agenten/Sitzungen, Provider-Konfiguration + Laufzeitprobleme (wenn das Gateway erreichbar ist).

2. **Pastebarer Bericht (sicher zum Teilen)**

   ```bash
   openclaw status --all
   ```

   Nur-Lese-Diagnose mit Log-Tail (Tokens redacted).

3. **Daemon + Port-Status**

   ```bash
   openclaw gateway status
   ```

   Zeigt Supervisor-Laufzeit vs. RPC-Erreichbarkeit, die Probe-Ziel-URL und welche Konfiguration der Service wahrscheinlich verwendet hat.

4. **Deep probes**

   ```bash
   openclaw status --deep
   ```

   Führt Gateway-Health-Checks + Provider-Probes aus (benötigt ein erreichbares Gateway). Siehe [Health](/gateway/health).

5. **Aktuelles Log verfolgen**

   ```bash
   openclaw logs --follow
   ```

   Wenn RPC down ist, fallback:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   Dateilogs sind getrennt von Service-Logs; siehe [Logging](/logging) und [Troubleshooting](/gateway/troubleshooting).

6. **Doctor ausführen (Reparaturen)**

   ```bash
   openclaw doctor
   ```

   Repariert/migriert Config/State + führt Health-Checks aus. Siehe [Doctor](/gateway/doctor).

7. **Gateway-Snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # zeigt Ziel-URL + Config-Pfad bei Fehlern
   ```

   Fragt das laufende Gateway nach einem vollständigen Snapshot (nur WS). Siehe [Health](/gateway/health).

## Schnellstart und Ersteinrichtung

### Im stuck was ist der schnellste Weg wieder weiterzukommen

Benutze einen lokalen KI-Agenten, der deine Maschine **sehen kann**. Das ist viel effektiver als in Discord zu fragen, weil die meisten „Ich stecke fest“-Fälle **lokale Konfigurations- oder Umgebungsprobleme** sind, die entfernte Helfer nicht inspizieren können.

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

Diese Tools können das Repo lesen, Befehle ausführen, Logs inspizieren und dir helfen, dein maschinennahes Setup zu reparieren (PATH, Services, Berechtigungen, Auth-Dateien). Gib ihnen den **vollständigen Source-Checkout** über die hackbare (git) Installation:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Das installiert OpenClaw **aus einem Git-Checkout**, sodass der Agent Code + Docs lesen und über die exakte Version nachdenken kann, die du gerade nutzt. Du kannst jederzeit zurück zu stable wechseln, indem du den Installer ohne `--install-method git` erneut ausführst.

Tipp: Bitte den Agenten, den Fix **zu planen und zu überwachen** (Schritt für Schritt) und dann nur die notwendigen Befehle auszuführen. Das hält Änderungen klein und auditierbar.

Wenn du einen echten Bug oder Fix findest, erstelle bitte ein GitHub-Issue oder sende eine PR:
- https://github.com/openclaw/openclaw/issues
- https://github.com/openclaw/openclaw/pulls

Starte mit diesen Befehlen (Ausgaben teilen, wenn du Hilfe suchst):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Was sie tun:

- `openclaw status`: schneller Snapshot von Gateway-/Agent-Health + Basis-Config.
- `openclaw models status`: prüft Provider-Auth + Modellverfügbarkeit.
- `openclaw doctor`: validiert und repariert häufige Config-/State-Probleme.

Weitere nützliche CLI-Checks: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`.

Schnelle Debug-Schleife: [First 60 seconds if something's broken](#first-60-seconds-if-somethings-broken).
