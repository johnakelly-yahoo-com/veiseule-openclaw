---
summary: "Najczęściej zadawane pytania dotyczące konfiguracji, ustawień i użytkowania OpenClaw"
title: "FAQ"
---

# FAQ

Szybkie odpowiedzi oraz pogłębione rozwiązywanie problemów dla rzeczywistych konfiguracji (lokalne środowisko deweloperskie, VPS, wiele agentów, OAuth/klucze API, przełączanie modeli). W przypadku diagnostyki czasu działania zobacz [Rozwiązywanie problemów](/gateway/troubleshooting). Pełne odniesienie do konfiguracji znajdziesz w [Konfiguracja](/gateway/configuration).

## Spis treści

- [Szybki start i konfiguracja przy pierwszym uruchomieniu]
  - [Utknąłem — jaki jest najszybszy sposób, aby ruszyć dalej?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [Jaki jest zalecany sposób instalacji i konfiguracji OpenClaw?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Jak otworzyć pulpit po onboardingu?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Jak uwierzytelnić pulpit (token) na localhost vs zdalnie?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Jakiego środowiska uruchomieniowego potrzebuję?](#what-runtime-do-i-need)
  - [Czy działa na Raspberry Pi?](#does-it-run-on-raspberry-pi)
  - [Czy są jakieś wskazówki dla instalacji na Raspberry Pi?](#any-tips-for-raspberry-pi-installs)
  - [Zatrzymało się na „wake up my friend” / onboarding się nie uruchamia. Co teraz?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Czy mogę przenieść konfigurację na nową maszynę (Mac mini) bez ponownego onboardingu?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [Gdzie zobaczę nowości w najnowszej wersji?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [Nie mogę uzyskać dostępu do docs.openclaw.ai (błąd SSL). Co teraz?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Jaka jest różnica między stable a beta?](#whats-the-difference-between-stable-and-beta)
  - [Jak zainstalować wersję beta i czym różni się beta od dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [Jak wypróbować najnowsze zmiany?](#how-do-i-try-the-latest-bits)
  - [Ile zwykle trwa instalacja i onboarding?](#how-long-does-install-and-onboarding-usually-take)
  - [Instalator się zawiesił? Jak uzyskać więcej informacji?](#installer-stuck-how-do-i-get-more-feedback)
  - [Instalacja na Windows zgłasza „git not found” lub „openclaw not recognized”](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [Dokumentacja nie odpowiedziała na moje pytanie — jak uzyskać lepszą odpowiedź?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [Jak zainstalować OpenClaw na Linuxie?](#how-do-i-install-openclaw-on-linux)
  - [Jak zainstalować OpenClaw na VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Gdzie są poradniki instalacji w chmurze/VPS?](#where-are-the-cloudvps-install-guides)
  - [Czy mogę poprosić OpenClaw, aby sam się zaktualizował?](#can-i-ask-openclaw-to-update-itself)
  - [Co faktycznie robi kreator onboardingu?](#what-does-the-onboarding-wizard-actually-do)
  - [Czy potrzebuję subskrypcji Claude lub OpenAI, aby to uruchomić?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Czy mogę używać subskrypcji Claude Max bez klucza API](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Jak działa uwierzytelnianie Anthropic „setup-token”?](#how-does-anthropic-setuptoken-auth-work)
  - [Gdzie znaleźć setup-token Anthropic?](#where-do-i-find-an-anthropic-setuptoken)
  - [Czy obsługujecie uwierzytelnianie subskrypcji Claude (Claude Pro lub Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Dlaczego widzę `HTTP 429: rate_limit_error` z Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Czy AWS Bedrock jest obsługiwany?](#is-aws-bedrock-supported)
  - [Jak działa uwierzytelnianie Codex?](#how-does-codex-auth-work)
  - [Czy obsługujecie uwierzytelnianie subskrypcji OpenAI (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Jak skonfigurować OAuth Gemini CLI](#how-do-i-set-up-gemini-cli-oauth)
  - [Czy lokalny model nadaje się do luźnych rozmów?](#is-a-local-model-ok-for-casual-chats)
  - [Jak utrzymać ruch do modeli hostowanych w określonym regionie?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Czy muszę kupić Mac mini, aby to zainstalować?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Czy potrzebuję Mac mini do obsługi iMessage?](#do-i-need-a-mac-mini-for-imessage-support)
  - [Jeśli kupię Mac mini do uruchamiania OpenClaw, czy mogę połączyć go z MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Czy mogę używać Bun?](#can-i-use-bun)
  - [Telegram: co wpisuje się w `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Czy wiele osób może używać jednego numeru WhatsApp z różnymi instancjami OpenClaw?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Czy mogę uruchomić agenta „szybkiej rozmowy” oraz agenta „Opus do kodowania”?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Czy Homebrew działa na Linuxie?](#does-homebrew-work-on-linux)
  - [Jaka jest różnica między instalacją „hackowalną” (git) a instalacją npm?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Czy mogę później przełączać się między instalacją npm a git?](#can-i-switch-between-npm-and-git-installs-later)
  - [Czy powinienem uruchamiać Gateway na laptopie czy na VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [Jak ważne jest uruchamianie OpenClaw na dedykowanej maszynie?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Jakie są minimalne wymagania VPS i zalecany system operacyjny?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Czy mogę uruchomić OpenClaw w maszynie wirtualnej i jakie są wymagania](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [Czym jest OpenClaw?](#what-is-openclaw)
  - [Czym jest OpenClaw w jednym akapicie?](#what-is-openclaw-in-one-paragraph)
  - [Jaka jest propozycja wartości?](#whats-the-value-proposition)
  - [Właśnie to skonfigurowałem — co powinienem zrobić najpierw](#i-just-set-it-up-what-should-i-do-first)
  - [Jakie są pięć najczęstszych codziennych zastosowań OpenClaw](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [Czy OpenClaw może pomóc w lead gen, outreach, reklamach i blogach dla SaaS](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Jakie są zalety w porównaniu z Claude Code przy tworzeniu stron internetowych?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills i automatyzacja](#skills-and-automation)
  - [Jak dostosować skills bez brudzenia repozytorium?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Czy mogę ładować skills z niestandardowego folderu?](#can-i-load-skills-from-a-custom-folder)
  - [Jak mogę używać różnych modeli do różnych zadań?](#how-can-i-use-different-models-for-different-tasks)
  - [Bot zawiesza się podczas ciężkiej pracy. Jak to odciążyć?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron lub przypomnienia nie uruchamiają się. Co sprawdzić?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Jak zainstalować skills na Linuxie?](#how-do-i-install-skills-on-linux)
  - [Czy OpenClaw może uruchamiać zadania według harmonogramu lub ciągle w tle?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Czy mogę uruchamiać skills tylko dla macOS z Linuxa?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Czy macie integrację z Notion lub HeyGen?](#do-you-have-a-notion-or-heygen-integration)
  - [Jak zainstalować rozszerzenie Chrome do przejmowania przeglądarki?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandboxing i pamięć](#sandboxing-and-memory)
  - [Czy istnieje dedykowana dokumentacja sandboxing?](#is-there-a-dedicated-sandboxing-doc)
  - [Jak powiązać folder hosta z sandboxem?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Jak działa pamięć?](#how-does-memory-work)
  - [Pamięć ciągle zapomina. Jak sprawić, by zapamiętywała?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Czy pamięć utrzymuje się na zawsze? Jakie są limity?](#does-memory-persist-forever-what-are-the-limits)
  - [Czy semantyczne wyszukiwanie pamięci wymaga klucza API OpenAI?](#does-semantic-memory-search-require-an-openai-api-key)
- [Gdzie dane są zapisywane na dysku](#where-things-live-on-disk)
  - [Czy wszystkie dane używane przez OpenClaw są zapisywane lokalnie?](#is-all-data-used-with-openclaw-saved-locally)
  - [Gdzie OpenClaw przechowuje swoje dane?](#where-does-openclaw-store-its-data)
  - [Gdzie powinny znajdować się AGENTS.md / SOUL.md / USER.md / MEMORY.md?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [Jaka jest zalecana strategia kopii zapasowych?](#whats-the-recommended-backup-strategy)
  - [Jak całkowicie odinstalować OpenClaw?](#how-do-i-completely-uninstall-openclaw)
  - [Czy agenci mogą pracować poza obszarem roboczym?](#can-agents-work-outside-the-workspace)
  - [Jestem w trybie zdalnym — gdzie jest magazyn sesji?](#im-in-remote-mode-where-is-the-session-store)
- [Podstawy konfiguracji](#config-basics)
  - [Jaki format ma konfiguracja? Gdzie się znajduje?](#what-format-is-the-config-where-is-it)
  - [Ustawiłem `gateway.bind: "lan"` (lub `"tailnet"`) i teraz nic nie nasłuchuje / UI mówi „unauthorized”](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Dlaczego teraz potrzebuję tokenu na localhost?](#why-do-i-need-a-token-on-localhost-now)
  - [Czy muszę restartować po zmianie konfiguracji?](#do-i-have-to-restart-after-changing-config)
  - [Jak włączyć wyszukiwanie w sieci (i web fetch)?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply wyczyścił moją konfigurację. Jak odzyskać i uniknąć tego?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [Jak uruchomić centralny Gateway z wyspecjalizowanymi workerami na różnych urządzeniach?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [Czy przeglądarka OpenClaw może działać w trybie headless?](#can-the-openclaw-browser-run-headless)
  - [Jak używać Brave do sterowania przeglądarką?](#how-do-i-use-brave-for-browser-control)
- [Zdalne gatewaye i węzły](#remote-gateways-and-nodes)
  - [Jak polecenia propagują się między Telegramem, gatewayem i węzłami?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Jak agent może uzyskać dostęp do mojego komputera, jeśli Gateway jest hostowany zdalnie?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale jest połączony, ale nie otrzymuję odpowiedzi. Co teraz?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [Czy dwie instancje OpenClaw mogą ze sobą rozmawiać (lokalnie + VPS)?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Czy potrzebuję osobnych VPS dla wielu agentów](#do-i-need-separate-vpses-for-multiple-agents)
  - [Czy jest korzyść z używania węzła na moim laptopie zamiast SSH z VPS?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Czy węzły uruchamiają usługę gateway?](#do-nodes-run-a-gateway-service)
  - [Czy istnieje sposób API / RPC na zastosowanie konfiguracji?](#is-there-an-api-rpc-way-to-apply-config)
  - [Jaka jest minimalna „rozsądna” konfiguracja dla pierwszej instalacji?](#whats-a-minimal-sane-config-for-a-first-install)
  - [Jak skonfigurować Tailscale na VPS i połączyć się z Maca?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Jak podłączyć węzeł Mac do zdalnego Gateway (Tailscale Serve)?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [Czy powinienem instalować na drugim laptopie czy po prostu dodać węzeł?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Zmienne środowiskowe i ładowanie .env](#env-vars-and-env-loading)
  - [Jak OpenClaw ładuje zmienne środowiskowe?](#how-does-openclaw-load-environment-variables)
  - [„Uruchomiłem Gateway jako usługę i moje zmienne środowiskowe zniknęły.” Co teraz?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [Ustawiłem `COPILOT_GITHUB_TOKEN`, ale status modeli pokazuje „Shell env: off.” Dlaczego?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Sesje i wiele czatów](#sessions-and-multiple-chats)
  - [Jak rozpocząć nową rozmowę?](#how-do-i-start-a-fresh-conversation)
  - [Czy sesje resetują się automatycznie, jeśli nigdy nie wyślę `/new`?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [Czy da się stworzyć zespół instancji OpenClaw: jeden CEO i wielu agentów](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Dlaczego kontekst został ucięty w trakcie zadania? Jak temu zapobiec?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [Jak całkowicie zresetować OpenClaw, zachowując instalację?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - [Otrzymuję błędy „context too large” — jak zresetować lub skompaktować?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - [Dlaczego widzę „LLM request rejected: messages.N.content.X.tool_use.input: Field required”?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  - [Dlaczego otrzymuję komunikaty heartbeat co 30 minut?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [Czy muszę dodać „konto bota” do grupy WhatsApp?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [Jak uzyskać JID grupy WhatsApp?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Dlaczego OpenClaw nie odpowiada w grupie?](#why-doesnt-openclaw-reply-in-a-group)
  - [Czy grupy/wątki współdzielą kontekst z DM-ami?](#do-groupsthreads-share-context-with-dms)
  - [Ile obszarów roboczych i agentów mogę utworzyć?](#how-many-workspaces-and-agents-can-i-create)
  - [Czy mogę uruchamiać wiele botów lub czatów jednocześnie (Slack) i jak to skonfigurować?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Modele: domyślne, wybór, aliasy, przełączanie](#models-defaults-selection-aliases-switching)
  - [Czym jest „domyślny model”?](#what-is-the-default-model)
  - [Jaki model polecacie?](#what-model-do-you-recommend)
  - [Jak przełączyć modele bez czyszczenia konfiguracji?](#how-do-i-switch-models-without-wiping-my-config)
  - [Czy mogę używać modeli hostowanych samodzielnie (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [Jakich modeli używają OpenClaw, Flawd i Krill?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [Jak przełączyć modele w locie (bez restartu)?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [Czy mogę używać GPT 5.2 do codziennych zadań i Codex 5.3 do kodowania](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - [Dlaczego widzę „Model … is not allowed”, a potem brak odpowiedzi?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - [Dlaczego widzę „Unknown model: minimax/MiniMax-M2.1”?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [Czy mogę używać MiniMax jako domyślnego i OpenAI do złożonych zadań?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [Czy opus / sonnet / gpt to wbudowane skróty?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [Jak zdefiniować/nadpisać skróty modeli (aliasy)?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [Jak dodać modele od innych dostawców, takich jak OpenRouter lub Z.AI?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Failover modeli i „All models failed”](#model-failover-and-all-models-failed)
  - [Jak działa failover?](#how-does-failover-work)
  - [Co oznacza ten błąd?](#what-does-this-error-mean)
  - [Lista naprawcza dla `No credentials found for profile "anthropic:default"`](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Dlaczego próbował także Google Gemini i zawiódł?](#why-did-it-also-try-google-gemini-and-fail)
- [Profile uwierzytelniania: czym są i jak nimi zarządzać](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [Czym jest profil uwierzytelniania?](#what-is-an-auth-profile)
  - [Jakie są typowe identyfikatory profili?](#what-are-typical-profile-ids)
  - [Czy mogę kontrolować, który profil uwierzytelniania jest próbowany jako pierwszy?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs klucz API: jaka jest różnica?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: porty, „already running” i tryb zdalny](#gateway-ports-already-running-and-remote-mode)
  - [Jakiego portu używa Gateway?](#what-port-does-the-gateway-use)
  - [Dlaczego `openclaw gateway status` pokazuje `Runtime: running`, ale `RPC probe: failed`?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [Dlaczego `openclaw gateway status` pokazuje `Config (cli)` i `Config (service)` jako różne?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - [Co oznacza „another gateway instance is already listening”?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [Jak uruchomić OpenClaw w trybie zdalnym (klient łączy się z Gateway gdzie indziej)?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [UI sterowania mówi „unauthorized” (lub ciągle się łączy ponownie). Co teraz?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [Ustawiłem `gateway.bind: "tailnet"`, ale nie może zbindować / nic nie nasłuchuje](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Czy mogę uruchamiać wiele Gateway na tym samym hoście?](#can-i-run-multiple-gateways-on-the-same-host)
  - [Co oznacza „invalid handshake” / kod 1008?](#what-does-invalid-handshake-code-1008-mean)
- [Logowanie i debugowanie](#logging-and-debugging)
  - [Gdzie są logi?](#where-are-logs)
  - [Jak uruchomić/zatrzymać/zrestartować usługę Gateway?](#how-do-i-startstoprestart-the-gateway-service)
  - [Zamknąłem terminal na Windows — jak zrestartować OpenClaw?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Gateway działa, ale odpowiedzi nigdy nie docierają. Co sprawdzić?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - [„Disconnected from gateway: no reason” — co teraz?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands kończy się błędami sieci. Co sprawdzić?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI nie pokazuje wyjścia. Co sprawdzić?](#tui-shows-no-output-what-should-i-check)
  - [Jak całkowicie zatrzymać, a następnie uruchomić Gateway?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` kontra `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [Jaki jest najszybszy sposób uzyskania większej liczby szczegółów, gdy coś zawiedzie?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Media i załączniki](#media-and-attachments)
  - [Skill wygenerował obraz/PDF, ale nic nie zostało wysłane](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Bezpieczeństwo i kontrola dostępu](#security-and-access-control)
  - [Czy bezpieczne jest wystawienie OpenClaw na przychodzące DM-y?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [Czy prompt injection dotyczy tylko botów publicznych?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Czy mój bot powinien mieć własny e‑mail, konto GitHub lub numer telefonu](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Czy mogę dać mu autonomię nad moimi wiadomościami tekstowymi i czy to jest bezpieczne](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Czy mogę używać tańszych modeli do zadań asystenta osobistego?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [Uruchomiłem `/start` w Telegramie, ale nie dostałem kodu parowania](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: czy będzie wysyłać wiadomości do moich kontaktów? Jak działa parowanie?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Polecenia czatu, przerywanie zadań i „nie chce się zatrzymać”](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [Jak zatrzymać wyświetlanie wewnętrznych komunikatów systemowych w czacie](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [Jak zatrzymać/anulować uruchomione zadanie?](#how-do-i-stopcancel-a-running-task)
  - [Jak wysłać wiadomość Discord z Telegrama? („Cross-context messaging denied”)](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Dlaczego wygląda to tak, jakby bot „ignorował” serię szybkich wiadomości?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## Pierwsze 60 sekund, gdy coś nie działa

1. **Szybki status (pierwsze sprawdzenie)**

   ```bash
   openclaw status
   ```

   Szybkie lokalne podsumowanie: system operacyjny + aktualizacja, dostępność gateway/usługi, agenci/sesje, konfiguracja dostawców + problemy środowiska uruchomieniowego (gdy gateway jest osiągalny).

2. **Raport do wklejenia (bezpieczny do udostępnienia)**

   ```bash
   openclaw status --all
   ```

   Diagnostyka tylko do odczytu z końcówką logów (tokeny zanonimizowane).

3. **Stan demona i portów**

   ```bash
   openclaw gateway status
   ```

   Pokazuje środowisko uruchomieniowe nadzorcy vs dostępność RPC, docelowy URL sondy oraz którą konfigurację usługa prawdopodobnie użyła.

4. **Głębokie sondy**

   ```bash
   openclaw status --deep
   ```

   Uruchamia kontrole zdrowia gateway + sondy dostawców (wymaga osiągalnego gateway). Zobacz [Health](/gateway/health).

5. **Podgląd najnowszego logu**

   ```bash
   openclaw logs --follow
   ```

   Jeśli RPC nie działa, użyj zapasowo:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   Logi plikowe są oddzielne od logów usługi; zobacz [Logging](/logging) oraz [Rozwiązywanie problemów](/gateway/troubleshooting).

6. **Uruchom lekarza (naprawy)**

   ```bash
   openclaw doctor
   ```

   Naprawia/migruje konfigurację i stan + uruchamia kontrole zdrowia. Zobacz [Doctor](/gateway/doctor).

7. **Migawka Gateway**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   Pyta działający gateway o pełną migawkę (tylko WS). Zobacz [Health](/gateway/health).

## Szybki start i konfiguracja przy pierwszym uruchomieniu

### Utknęłem to, co najszybszy sposób na odklejenie

Użyj lokalnego agenta AI, który może **zobaczyć Twoją maszynę**. Jest to o wiele skuteczniejsze niż zapytanie
na Discordzie, ponieważ większość przypadków "Jestem utknięty" to **lokalne problemy z konfiguracją lub środowiskiem**, których
zdalni pomocnicy nie mogą sprawdzić.

- \*\*Kod Claude \*\*: [https://www.anthropic.com/claude-code/](https://www.anthropic.com/claude-code/)
- **OpenAI Codex**: [https://openai.com/codex/](https://openai.com/codex/)

Narzędzia te mogą odczytywać repo, uruchamiać polecenia, przeglądać dzienniki i pomóc naprawić konfigurację twojego poziomu
(usługi PATH, uprawnienia, pliki autoryzacyjne). Daj im **pełną płatność źródłową** poprzez
instalację hakowalną (git):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

To instaluje OpenClaw **z git checkout**, aby agent mógł przeczytać kod + dokumenty i przyczynę
dokładnej wersji, którą działasz. Zawsze możesz wrócić do stabilnego później
poprzez ponowne uruchomienie instalatora bez `--install-method git`.

Wskazówka: poproś agenta o **zaplanowanie i nadzór** naprawę (krok po kroku), a następnie wykonaj tylko potrzebne polecenia
. To sprawia, że zmiany są małe i łatwiejsze do kontrolowania.

Jeśli odkryjesz prawdziwy błąd lub poprawkę, proszę zgłosić problem z GitHub lub wysłać PR:
[https://github.com/openclaw/openclaw/issues](https://github.com/openclaw/openclaw/issues)
[https://github.com/openclaw/openclaw/pulls](https://github.com/openclaw/openclaw/pulls)

Zacznij od tych poleceń (udostępniaj dane wyjściowe, gdy poproś o pomoc):

```bash
status openclaw
openclaw models status
openclaw lekarz
```

Co robią:

- `openclaw status`: szybki zrzut batewa/agent health + podstawowe konfiguracje.
- `openclaw model`: sprawdza autoryzację dostawcy + dostępność modelu.
- `openclaw doktorem`: sprawdza i naprawia wspólne problemy z konfiguracją/stanem.

Inne przydatne sprawdzenie CLI: `openclaw --all`, `openclaw logs --follow`,
`status bramy otwartej`, `openclaw health --verbose`.

Pętla szybkiego debugowania: [Pierwsze 60 sekund jeśli coś się zepsuje](#first-60-seconds-if-somethings-broken).
Zainstaluj dokumenty: [Install](/install), [flagi instalatora](/install/installer), [Updating](/install/updating).

### Jaki jest zalecany sposób instalacji i konfiguracji OpenClaw

Repozytorium zaleca korzystanie ze źródła i korzystanie z kreatora wdrożeń:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw na board --install-daemon
```

Kreator może również automatycznie budować zasoby interfejsu. Po wejściu na pokład, zazwyczaj uruchamiasz bramę na porcie **18789**.

Z źródła (współtwórcy/dev):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps przy pierwszym uruchomieniu
openclaw na pokładzie
```

Jeśli nie masz jeszcze globalnej instalacji, uruchom ją przez `pnpm openclaw onboard`.

### Jak otworzyć pulpit po wprowadzeniu

Kreator otwiera przeglądarkę z czystym (nietokenizowanym) pulpitu URL bezpośrednio po wprowadzeniu i wyświetla link w podsumowaniu. Pozostaw tę kartę otwarte; jeśli nie uruchomi się, skopiuj/wklej wydrukowany adres URL na tym samym komputerze.

### Jak uwierzytelnić token pulpitu nawigacyjnego na localhost vs remote

**Localhost (ta sama maszyna):**

- Otwórz `http://127.0.0.1:18789/`.
- Jeśli prosi o autoryzację, wklej token z `gateway.auth.token` (lub `OPENCLAW_GATEWAY_TOKEN`) do ustawień interfejsu użytkownika.
- Pobierz go z hosta bramy: `openclaw config get gateway.auth.token` (lub wygeneruj: `openclaw doctor --generate-gateway-token`).

**Nie na serwerze:**

- **Seria ogonowa** (zalecane): zachowaj powiązaną pętlę, uruchom `openclaw gateway --tailscale serve`, otwórz `https://<magicdns>/`. Jeśli `gateway.auth.allowTailscale` jest `true`, nagłówki tożsamości spełniają autoryzację (bez tokenu).
- **Wiązanie sieci ogonowej**: uruchom `openclaw gateway --bind tailnet --token "<token>"`, otwórz `http://<tailscale-ip>:18789/`, wklej token w ustawieniach tablicy.
- **Tunel SSH**: `ssh -N -L 18789:127.0.0.1:18789 user@host` i otwórz `http://127.0.0.1:18789/` i wklej token w ustawieniach Control UI.

Zobacz [Dashboard](/web/dashboard) i [powierzchnie internetowe](/web) dla trybów łączenia i uwierzytelniania.

### Jaki czas pracy potrzebuję

Węzeł **>= 22** jest wymagany. `pnpm` jest zalecane. Bun nie jest **zalecany** dla bramy.

### Czy działa na Raspberry Pi

Tak. Brama jest lekka - dokumentacja **512MB-1GB RAM**, **1 rdzenia**, i około **500MB** na dysku
wystarczająco dużo do użytku osobistego i pamiętaj, że **Raspberry Pi 4 może go uruchomić**.

Jeśli potrzebujesz dodatkowych nagród (dzienniki, media, inne usługi), rekomendowane jest **2G**, ale to
nie jest twardym minimum.

Wskazówka: mały Pi/VPS może hostować bramę i możesz sparować **węzły** na laptop/telefon dla
lokalnego ekranu/kamera/płótna lub polecenia. Zobacz [Nodes](/nodes).

### Dowolne wskazówki dla instalacji Raspberry Pi

Krótka wersja: zadziała, ale oczekuje przybliżonych krawędzi.

- Użyj **64-bitowego** OS i zachowaj węzeł >= 22.
- Preferuj **hackable (git) install**, aby można było szybko zobaczyć logi i zaktualizować.
- Zacznij bez kanałów/umiejętności, a następnie dodaj je jeden po drugim.
- Jeśli uderzysz w dziwne problemy binarne, to zazwyczaj jest to problem **kompatybilności ARM**.

Dokumenty: [Linux](/platforms/linux), [Install](/install).

### To utknęło na pobudce mojego przyjaciela na pokładzie nie wykluje się co teraz

Ten ekran zależy od tego, czy brama jest dostępna i uwierzytelniana. TUI wysyła również
"Wybudź się, mój przyjacielu!" automatycznie na pierwszym luku. Jeśli widzisz tę linię **bez odpowiedzi**
i tokeny pozostają na 0, przedstawiciel nigdy nie ranking.

1. Zrestartuj Gateway:

```bash
openclaw gateway restart
```

2. Sprawdź status + autoryzacja:

```bash
status openclaw
openclaw models status
openclaw logi --follow
```

3. Jeżeli nadal się zawiesza, bieg:

```bash
openclaw doctor
```

Jeśli brama jest zdalna, upewnij się, że połączenie z tunelem/skalą ailscale jest gotowe i że interfejs użytkownika
jest zaznaczony na odpowiedniej bramie. Zobacz [Remote access](/gateway/remote).

### Czy mogę przenieść moją konfigurację na nową maszynę Mac mini bez ponownego wprowadzania na pokład?

Tak. Skopiuj **katalog stanów** i **obszar roboczy**, a następnie uruchom Doctor jeden raz. To
utrzymuje Twój bot "dokładnie identyczny" (pamięć, historia sesji, autoryzacja i kanał
), o ile kopiujesz **obydwa** lokalizacje:

1. Zainstaluj OpenClaw na nowym komputerze.
2. Skopiuj `$OPENCLAW_STATE_DIR` (domyślnie: `~/.openclaw`) ze starej maszyny.
3. Skopiuj swój projekt (domyślnie: `~/.openclaw/workspace`).
4. Uruchom `openclaw doctor` i uruchom ponownie usługę bramy.

Zachowuje konfigurację, profile uwierzytelniania, tworzenie WhatsApp, sesje i pamięć. Jeśli jesteś w trybie zdalnym
, zapamiętaj gospodarza bramy posiada sklep sesji i obszar roboczy.

**Important:** if you only commit/push your workspace to GitHub, you're backing
up **memory + bootstrap files**, but **not** session history or auth. Te żywe
pod adresem `~/.openclaw/` (na przykład `~/.openclaw/agents/<agentId>/sessions/`).

Powiązano: [Migrating](/install/migrating), [Gdzie rzeczy żyją na dysku](/help/faq#where-does-openclaw-store-its-data),
[Obszar roboczy agenta](/concepts/agent-workspace), [Doctor](/gateway/doctor),
[Tryb zdalny](/gateway/remote).

### Gdzie widzę co nowego w najnowszej wersji

Sprawdź listę zmian GitHub:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

Najnowsze wpisy znajdują się u góry. Jeśli górna część jest oznaczona **Niezwolniona**, kolejna sekcja
jest najnowszą wersją wysłaną. Wpisy są pogrupowane według **Podkreślania**, **Zmian**, i
**Naprawione** (plus dokumentacja/inne sekcje w razie potrzeby).

### Nie mam dostępu do docs.openclaw.ai błąd SSL

Niektóre połączenia kompresji / Xfinity niepoprawnie blokują `docs.openclaw.ai` przez Xfinity
Zaawansowane Bezpieczeństwo. Wyłącz lub zezwól na listę `docs.openclaw.ai`, a następnie spróbuj ponownie. Więcej
szczegóły: [Troubleshooting](/help/troubleshooting#docsopenclawai-shows-an-ssl-error-comcastxfinity).
Pomóż nam odblokować, zgłaszając tutaj: [https://spa.xfinity.com/check_url_status](https://spa.xfinity.com/check_url_status).

Jeśli nadal nie możesz dotrzeć do witryny, dokumenty są lustrzane na GitHub:
[https://github.com/openclaw/openclaw/tree/main/docs](https://github.com/openclaw/openclaw/tree/main/docs)

### Jaka jest różnica między stabilną a beta

**Stabilne** i **beta** to **npm dist-tags**, a nie oddzielne linie kodu:

- `latest` = stabilne
- `beta` = wczesna kompilacja do testów

Wysyłamy do **beta**, przetestujemy je, a gdy budowa będzie solidna, **promujemy
i tę samą wersję `latest`**. Dlatego beta i stabilne mogą wskazywać na
**tej samej wersji**.

Zobacz co zmienione:
[https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md](https://github.com/openclaw/openclaw/blob/main/CHANGELOG.md)

### Jak zainstalować wersję beta i jaka jest różnica między beta i dev

**Beta** jest znacznikiem dystansowym npm `beta` (może pasować do `latest`).
**Dev** jest ruchomą głową `main` (git); po publikacji używa on npm dist-tag `dev`.

Jednolinijkowe polecenia (macOS/Linux):

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --beta
```

```bash
curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Instalator Windows (PowerShell):
[https://openclaw.ai/install.ps1](https://openclaw.ai/install.ps1)

Więcej informacji: [kanały deweloperskie](/install/development-channels) i [flagi instalatora](/install/installer).

### Jak długo trwa instalacja i wdrożenie

Prosty przewodnik:

- **Zainstaluj:** 2-5 minut
- **Włączanie:** 5-15 minut w zależności od ile kanałów/modeli konfigurujesz

Jeśli powiesza się, użyj [Installer stuck](/help/faq#installer-stuck-how-do-i-get-more-feedback)
i pętli szybkiego debugowania w [Im stuck](/help/faq#im-stuck--whats-the-fastest-way-to-get-unstuck).

### Jak spróbować najnowszych bitów

Dwa opcje:

1. **Kanał deweloperski (git checkout):**

```bash
aktualizacja openclaw --channel dev
```

To przełącza się na gałąź `main` i aktualizacje ze źródła.

2. **Instalacja Hackable (z strony instalatora):**

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

To daje Ci lokalne repozytorium które możesz edytować, a następnie aktualizować za pomocą git.

Jeśli wolisz czysty klon, należy użyć:

```bash
Klon git https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm
```

Dokumenty: [Update](/cli/update), [kanały rozwoju](/install/development-channels),
[Install](/install).

### Instalator utknął jak uzyskać więcej opinii

Uruchom ponownie instalator z **dokładnym wyjściem**:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --verbose
```

Instalacja Beta z verbose:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --beta --verbose
```

Dla instalacji hakowalnej (git):

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git --verbose
```

Odpowiednik dla Windows (PowerShell):

```powershell
# install.ps1 nie ma jeszcze dedykowanej flagi -Verbose.
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

TUI nie pokazuje wyjścia co należy sprawdzić

### Użyj **osobnego numeru lub konta** jeśli chcesz wysłać wiadomość w swoim imieniu.

lub `/compact <instructions>` aby poprowadzić podsumowanie.

Użyj CLI:

- Pamięć utrzymuje się na zawsze co to są limity
- Czy jest dedykowany dok piaskownicy

Poproś bota o podsumowanie bieżącego stanu i zapisanie go do pliku.

- Jaki czas pracy potrzebuję

- Mogę dać mu autonomię w stosunku do moich wiadomości tekstowych i jest to bezpieczne

  ```powershell
  npm config get prefix
  ```

- Wyróżnienia:

- Przeznaczenie

Jeśli chcesz, aby najpłynniejsza konfiguracja Windows, użyj **WSL2** zamiast natywnych Windows.
Dokumenty: [Windows](/platforms/windows).

### Umieść host bramy + swój komputer w tej samej sieci ogonowej.

Kanały luźne związane z tymi czynnikami.

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Jest lokalnym modelem OK dla czatów dorywczych

### **Przekierowanie wielu agentów:** osobno dla każdego kanału, konta lub zadania, każdy z własnymi obszarami roboczymi&#xA;i domyślnymi.

Jaka jest propozycja wartości

- Dokumentacja nie odpowiedziała na moje pytanie, jak uzyskać lepszą odpowiedź
- Jakie są korzyści w porównaniu z kodem Claude dla rozwoju sieci
- Czy bezpieczne jest wystawianie OpenClaw na przychodzące DME

### **Agenci (pracownicy):** oddziel mózg/przestrzeń roboczą dla specjalnych ról (np. "Hetzner ops", "Dane osobowe").

Każdy system VPS Linux działa. Zainstaluj na serwerze, a następnie użyj SSH/Tailscale aby dotrzeć do bramy.

Przewodniki: [exe.dev](/install/exe-dev), [Hetzner](/install/hetzner), [Fly.io](/install/fly).
Zdalny dostęp: [Brama zdalna](/gateway/remote).

### **OpenClaw + wadliwe:** Antropic Opus (`anthropic/claude-opus-4-6`) - zobacz [Anthropic](/providers/anthropic).

Utrzymujemy **węzeł hostingowy** ze wspólnymi dostawcami. Wybierz jeden i postępuj zgodnie z przewodnikiem:

- Wskazówki:
- [Fly.io](/install/fly)
- [Hetzner](/install/hetzner)
- [exe.dev](/install/exe-dev)

Jak działa w chmurze: **Brama działa na serwerze**, i uzyskujesz dostęp do niej
z laptopa/telefonu za pośrednictwem interfejsu użytkownika sterowania (lub skala/SSH). Twój stan + obszar roboczy
na żywo na serwerze, więc traktuj hosta jako źródło prawdy i zapisuj go ponownie.

Włącz komunikaty dla konsultantów:

Hub: [Platforms](/platforms). Zdalny dostęp: [Brama zdalna](/gateway/remote).
Nodes: [Nodes](/nodes), [Nodes CLI](/cli/nodes).

### Powiązano: [Agent workspace](/concepts/agent-workspace), [Memory](/concepts/memory).

Krótka odpowiedź: **możliwa, nie zalecana**. Przepływ aktualizacji może zrestartować bramę
(która usuwa aktywną sesję), może potrzebować czystego zamówienia gita, a
może poprosić o potwierdzenie. Bezpieczniej: uruchom aktualizacje z powłoki jako operatora.

Dokumenty: [Nodes](/nodes), [protokół bramy](/gateway/protocol), [tryb zdalny macOS](/platforms/mac/remote), [Security](/gateway/security).

```bash
aktualizacja openclaw --yes --no-restart
uruchom ponownie bramę otwierania
```

Interfejs sterowania przechowuje token w przeglądarce localStorage klucz `openclaw.control.settings.v1`.

```bash
Połącz Gmail i zautomatyzuj podsumowania lub obserwacje.
```

Zrób węzły uruchamiające usługę bramy

### Co faktycznie robi kreator wdrażania

`openclaw onboard` jest zalecaną ścieżką konfiguracji. W **trybie lokalnym** przechodzisz przez Ciebie:

- Odzyskanie:
- Jeśli chcesz mieć natychmiastowy dostęp, zezwól na listę identyfikatora nadawcy lub ustaw `dmPolicy: "open"`
  dla tego konta.
- Gdzie rzeczy żyją na dysku
- Sprawdź:
- Opcje bezpieczne:
- Jeśli nigdy nie zainstalowałeś usługi, uruchom ją na pierwszym planie:

Dlaczego kontekst uzyskał obcięte średnie zadanie, jak zapobiegać temu

### Priorytet:

Nie. You can run OpenClaw with **API keys** (Anthropic/OpenAI/others) or with
**local-only models** so your data stays on your device. Subskrypcje (Claude
Pro/Max lub OpenAI Codex) są opcjonalnymi sposobami uwierzytelniania tych dostawców.

Dlaczego co 30 minut odbieram wiadomości o biciu serca

### Zobacz [Images](/nodes/images).

Tak. Możesz uwierzytelnić się za pomocą **tokenu setup-token**
zamiast klucza API. To jest ścieżka subskrypcji.

Subskrypcje Claude Pro/Max **nie zawierają klucza API**, więc jest to poprawna metoda
dla kont subskrypcji. Ważne: musisz zweryfikować z
Antropię, że to użycie jest dozwolone zgodnie z ich zasadami i warunkami subskrypcji.
Jeśli chcesz najbardziej wyraźna, obsługiwana ścieżka, użyj Antropicznego klucza API.

### **Opcja tylko lokalna:** uruchamia modele lokalne, więc **wszystkie dane mogą pozostać na Twoim urządzeniu**, jeśli chcesz.

`claude setup-token` generuje **ciąg tokenów** za pośrednictwem kodu Claude Code CLI (nie jest on dostępny w konsoli internetowej). Możesz uruchomić go na **każdej maszynie**. Wybierz **Token Antropiczny (wklej token setup-token)** w kreatorze lub wklej go za pomocą `openclaw models auth paste-token --provider antropiczny`. Token jest przechowywany jako profil autoryzacji dostawcy **antropicznego** i używany jako klucz API (bez automatycznego odświeżania). Więcej szczegółów: [OAuth](/concepts/oauth).

### **Pros:** zawsze, stabilna sieć, bez problemów ze snem laptopa, łatwiejsze do utrzymania.

To **nie** w konsoli Antropicznej. Token konfiguracyjny jest generowany przez \*\*CLI kodu Claude \*\* na **dowolnej maszynie**:

```bash
claude setup-token
```

Skopiuj token wydrukowany, a następnie wybierz \*\*Antropiczny token (wklej token konfiguracyjny) \*\* w kreatorze. Jeśli chcesz uruchomić go na serwerze bramy, użyj `openclaw models auth setup-token --provider antropiczny`. Jeśli używałeś `claude setup-token` gdzie indziej, wklej go do hosta bramy `openclaw models auth paste-token --provider anthropic`. Zobacz [Anthropic](/providers/anthropic).

### Uwagi:

Tak - przez **setup-token**. OpenClaw nie używa już tokenów Claude Code CLI OAuth; użyj tokenu konfiguracyjnego lub Antropowego API. Wygeneruj token gdziekolwiek i wklej go do hosta bramy. Zobacz [Anthropic](/providers/anthropic) i [OAuth](/concepts/oauth).

Uwaga: Dostęp do subskrypcji Claude podlega warunkom Antropii. Dla produkcji lub wielu użytkowników klucze API są zazwyczaj bezpieczniejszym wyborem.

### Jeśli używałeś profili (`--profile` / `OPENCLAW_PROFILE`), zresetuj każdy dir stanów (domyślnie `~/.openclaw-<profile>`).

Oznacza to, że Twój **limit Antropiczny** jest wyczerpany dla bieżącego okna. If you
use a **Claude subscription** (setup-token or Claude Code OAuth), wait for the window to
reset or upgrade your plan. Jeśli używasz **Antropowego klucza API**, sprawdź Antropic Console
pod kątem użycia/rozliczeń i podnieś limity w razie potrzeby.

Wskazówka: ustaw **model awaryjny**, aby OpenClaw mógł nadal odpowiadać podczas gdy dostawca jest ograniczony stawkami.
Zobacz [Models](/cli/models) i [OAuth](/concepts/oauth).

### **Koordynacja poprzeczna urządzenia:** wyślij zadanie z telefonu, pozwól, aby brama działała na serwerze i odebrać wynik z powrotem na czacie.

Tak - poprzez dostawcę **Amazon Bedrock (Converse)** z **manualną konfiguracją**. Musisz podać dane logowania/region AWS na serwerze bramki i dodać wpis dostawcy Bedrock do konfiguracji modeli. [Amazon Bedrock](/providers/bedrock) i [Model providers](/providers/models). Jeśli wolisz obsługiwany przepływ klucza, proxy kompatybilne z OpenAI przed Bedrock jest nadal poprawną opcją.

### **Bezwzględne minimum:** 1 vCPU, 1GB RAM, ~500MB dysku.

OpenClaw obsługuje **OpenAI Code (Codex)** poprzez OAuth (ChatGPT Log-in). Kreator może uruchomić przepływ OAuth i w stosownych przypadkach ustawiy domyślny model na `openai-codex/gpt-5.3-codex`. Zobacz [dostawcy modelu](/concepts/model-providers) i [Wizard](/start/wizard).

### Ścieżka

Tak. OpenClaw w pełni obsługuje **OpenAI Code (Codex) subskrypcję OAuth**. Kreator wdrażania
może uruchomić dla Ciebie przepływ OAuth.

Wysyłanie CLI:

### **Kontrolujesz footprint:** przy użyciu lokalnych modeli utrzymuje wskazówki na Twoim komputerze, ale ruch na kanale&#xA;nadal przebiega przez serwery kanału.

Skrzynki piaskowe i pamięć

Czy grupowe wątki dzielą kontekst z Mami

1. Zachowaj ważny kontekst w obszarze roboczym i poproś bota o odczyt go z powrotem.
2. **Pojedyncze miejsca pracy** dla niezależnych agentów, którzy publikują podsumowania lub dostarczają na czat.

To przechowuje tokeny OAuth w profilach uwierzytelniania na serwerze bramy. Szczegóły: [Dostawcy modelu](/concepts/model-providers).

### **Więcej narzędzi urządzenia.** Nody pokazują `canvas`, `camera`, i `screen` oprócz `system.run`.

Zazwyczaj nr Zazwyczaj nr OpenClaw potrzebuje dużego kontekstu + silnego bezpieczeństwa; małe karty są obcięte i przecieki. Jeśli musisz uruchomić **największe** wersję MiniMax M2.1 możesz lokalnie (LM Studio) i zobaczyć [/gateway/local-models](/gateway/local-models). Mniejsze/kwantyfikowane modele zwiększają ryzyko szybkiego zatłaczania - zobacz [Security](/gateway/security).

### **Badania i redagowanie:** szybkie badania, podsumowania i pierwsze szkice wiadomości e-mail lub dokumentów.

Wybierz przypięte do regionu punkty końcowe. OpenRouter ujawnia opcje hostowane przez USA dla MiniMax, Kim i GLM; wybierz wariant hostowany przez US, aby przechowywać dane w regionie. Nadal możesz wyświetlić listę Anthropic/OpenAI wraz z nimi używając `models.mode: "merge"` więc rezerwa pozostanie dostępna przy jednoczesnym poszanowaniu wybranego przez Ciebie regionalnego dostawcy.

### /verbose off&#xA;/reason off

Nie. OpenClaw działa na macOS lub Linux (Windows via WSL2). Mac mini jest opcjonalny - niektórzy ludzie
kupują go jako zawsze gościa, ale działa również mały VPS, serwer domowy lub Raspberry Piklasa.

Potrzebujesz tylko Maca **dla narzędzi macOS**. Dla iMessage, użyj [BlueBubbles](/channels/bluebubbles) (zalecane) - serwer BlueBubbles działa na dowolnym komputerze Mac, a brama może działać na Linux lub gdzie indziej. Jeśli chcesz użyć innych narzędzi macOS, uruchom bramę na Mac lub sparowaj węzeł macOS.

Uruchomiłem bramę przez serwis, a moje vary env zniknęły co teraz

### Użyj **hackable (git) install**, aby mieć wszystkie źródła i dokumentację lokalnie, następnie zapytaj&#xA;swojego bota (lub Claude/Codex) _z tego folderu_ aby mógł dokładnie odczytać repozytorium i odpowiedzieć.

Potrzebujesz **jakiegoś urządzenia macOS** zalogowanego do wiadomości. To **nie** musi być Mac mini -
żaden Mac działa. **Użyj [BlueBubbles](/channels/bluebubbles)** (zalecane) dla iMessage - serwer BlueBubbles działa na macOS, podczas gdy brama może działać na Linux lub gdzie indziej.

`steer-backlog` - steer teraz, a następnie przetwarzaj backlog

- **Koszt tokenu:** więcej agentów oznacza większe jednoczesne użycie modelu.
- Dlaczego widzę błąd stawki HTTP 429 z Antropic

Co oznacza błędny kod handshake 1008

### **Automatyzacja przeglądarki:** wypełnianie formularzy, zbieranie danych i powtarzanie zadań internetowych.

Tak. **Mac mini może uruchomić bramę**, a Twój MacBook Pro może połączyć się jako
**węzeł** (urządzenie towarzyszące). Węzły nie uruchamiają bramy - zapewniają dodatkowe możliwości
, takie jak ekran, kamera/płótno i `system.run` na tym urządzeniu.

Rozpocznij nową sesję, aby odświeżyć zrzut umiejętności.

- configapply wyczyścił moją konfigurację jak odzyskać i uniknąć tego
- **Upewnij się, że edytujesz właściwego agenta**
- Umiejętności i automatyzacja

Jak zacząć uruchamiać usługę bramy

### Dokumenty: [Update](/cli/update), [Updating](/install/updating).

Bun jest **niezalecany**. Widzimy błędy czasu pracy, zwłaszcza z WhatsApp i Telegram.
Użyj **węzła** dla stabilnych bram.

**Sprawdzanie modelu/statusu uwierzytelniania**

### Zobacz [Cron jobs](/automation/cron-jobs), [Wielu Agentów](/concepts/multi-agent) i [Komendy Slash Slash ](/tools/slash-commands).

`channels.telegram.allowFrom` jest **ID użytkownika użytkownika Telegrama** (numerycznie, zalecane) lub `@username`. To nie jest nazwa użytkownika bota.

Kreator wdrożeniowy akceptuje dane wejściowe w formacie `@username` i rozwiązuje je do identyfikatora numerycznego, ale autoryzacja OpenClaw używa wyłącznie identyfikatorów numerycznych.

Jak korzystać z centralnej bramy z wyspecjalizowanymi pracownikami we wszystkich urządzeniach

- Opcje:

Wybierz model z większym oknem kontekstowym, jeśli zdarza się to często.

- Najlepsza konfiguracja:

Co oznacza ten błąd

- Odłączono od bramy bez powodu, co teraz

Dwa częste poprawki:

### W konsoli administratora w skali Ogonowej włącz MagicDNS, aby VPS miał stabilną nazwę.

Tak, przez **przekierowywanie wielu agentów**. Bind each sender's WhatsApp **DM** (peer `kind: "direct"`, sender E.164 like `+15551234567`) to a different `agentId`, so each person gets their own workspace and session store. Odpowiedzi nadal pochodzą z **tego samego konta WhatsApp**, a kontrola dostępu DM (`channels.whatsapp.dmPolicy` / `channels.whatsapp.allowFrom`) jest globalna dla konta WhatsApp. Zobacz [dzwonienie wielu agentów](/concepts/multi-agent) i [WhatsApp](/channels/whatsapp).

### Kroki:

Tak. Użyj przekierowywania wielu agentów: daj każdemu agentowi swój własny domyślny model, a następnie połącz trasy przychodzące (konto dostawcy lub konkretne partnery) z każdym agentem. Przykładowa konfiguracja życia w [dzwonieniu wielu agentów](/concepts/multi-agent). Zobacz również [Models](/concepts/models) i [Configuration](/gateway/configuration).

### Dokumenty: [Memory](/concepts/memory), [Context](/concepts/context).

Tak. Homebrew obsługuje Linux (Linuxbrew). Szybka konfiguracja:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
brew install <formula>
```

Jeśli uruchomisz OpenClaw za pośrednictwem systemu, upewnij się, że usługa PATH zawiera `/home/linuxbrew/.linuxbrew/bin` (lub twój prefiks brajski), więc `narzędzia zainstalowane w powłokach nie logowania.
Ostatnie kompilacje poprzedzają również zwykłe bloki użytkowników na usługach systemu Linux (na przykład `~/.local/bin`, `~/.npm-global/bin`, `~/.local/share/pnpm`, `~/. un/bin`) i honor `PNPM_HOME`, `NPM_CONFIG_PREFIX`, `BUN_INSTALL`, `VOLTA_HOME`, `ASDF_DATA_DIR`, `NVM_DIR`, oraz `FNM_DIR\\`, gdy ustawione.

### Bądź na ekranie z `OPENCLAW_DOCKER_APT_ULOTK`.

- **Instalacja hackable (git):** pełne zamówienie źródłowe, edytowalne, najlepsze dla współtwórców.
  Uruchamiasz wersje lokalnie i możesz aktualizować kod/dokumenty.
- **npm install:** globalna instalacja CLI, brak repozytorium, najlepsze dla "po prostu uruchom go".
  Aktualizacje pochodzą z npm dist-tags.

Jak mogę wysłać wiadomość Discorda z Telegram Crosscontext wiadomoci zabronione

### Uruchom `openclaw doctor` do zasad DM ryzykownych powierzchni.

Tak. Zainstaluj inny aromat i uruchom Doctor tak, aby punkty usługi łączności na nowym punkcie wejścia.
To **nie usuwa Twoich danych** - zmienia tylko instalację kodu OpenClaw. Twój stan
(`~/.openclaw`) i obszar roboczy (`~/.openclaw/workspace`) pozostają niezmienione.

Użyj `openclaw doctor` aby wykryć bezbarwne obszary robocze i niedopasowanie profilu.

```bash
Niewymagane, ale **zalecane dla niezawodności i izolacji**.
```

Opóźnienia mają zastosowanie do upadających profili (backoff), więc OpenClaw może nadal odpowiadać nawet wtedy, gdy usługodawca jest ograniczony lub tymczasowo nie działa.

```bash
Uwagi:
```

Moja umiejętność wygenerowała plik imagePDF ale nic nie zostało wysłane

Uruchomiłem w Telegramie, ale nie otrzymałem kodu parowania

### **Osobiste streszczenia:** streszczenia skrzynki odbiorczej, kalendarza i wiadomości, których się interesują.

Krótka odpowiedź: **jeśli chcesz mieć niezawodność 24/7, użyj VPS**. Jeśli chcesz, aby
najniższe tarcie i dobrze jesteś w stanie snu/restartować, uruchom je lokalnie.

Jak dostosować umiejętności bez zachowywania brudności repozytorium

- Jeśli jest zdalny, czy tunel / skala dostosowana do góry?
- W ustawieniach interfejsu sterowania, wklej ten sam token.

Jeśli nie masz jeszcze tokenu: `openclaw Lekarz --generate-gateway-token`.

- Czy muszę dodać konto bota do grupy WhatsApp
- Gdzie znajdę Antropiczny setuptoken

**Notatka specyficzna dla OpenClaw:** WhatsApp/Telegram/Slack/Mattermost (wtyczka)/Discord wszystko działa poprawnie z VPS. Jedynym prawdziwym kompromisem jest **bezgłowna przeglądarka** w porównaniu z widocznym oknem. Zobacz [Browser](/tools/browser).

**Rekomendowane domyślne:** VPS jeśli masz wcześniej odłączoną bramę. Lokalny jest świetny, gdy aktywnie używasz Maca i chcesz mieć dostęp do plików lokalnych lub automatyzację interfejsu użytkownika z widoczną przeglądarką.

### Instalator + aktualizacje: [Instalacja i aktualizacje](/install/updating).

Lokalny agent kodowania często może odtworzyć działającą konfigurację z logów lub historii.

- Jestem w trybie zdalnym, gdzie jest sklep sesji
- Jak skonfigurować Gemini CLI OAuth

Jeśli chcecie najlepszych z obu światów, zachowaj bramę na dedykowanym serwerze i sparowuj swój laptopa jako **węzeł** dla lokalnych narzędzi ekranu/kamery/exec. Zobacz [Nodes](/nodes).
Aby uzyskać wskazówki dotyczące bezpieczeństwa, przeczytaj [Security](/gateway/security).

### Metadane sesji (na agenta)

OpenClaw jest lekki. Dla podstawowej bramy + jeden kanał czatu:

- Jak działa awaria
- **Zalecane:** 1-2 vCPU, 2GB RAM lub więcej dla głowy (logów, mediów, wielu kanałów). Narzędzia węzła i automatyzacja przeglądarki mogą być głodne zasoby.

OS: użyj **Ubuntu LTS** (lub dowolnego nowoczesnego Debian/Ubuntu). Ścieżka instalacyjna Linux jest tam najlepiej przetestowana.

Co faktycznie robi kreator wdrażania

### Zobacz dedykowany przewodnik: [Uninstall](/install/uninstall).

Tak. Traktuj VM tak samo jak VPS: zawsze musi być włączony, osiągalny, i masz wystarczającą ilość pamięci RAM
dla bramy i wszystkich kanałów, które włączysz.

Użyj aplikacji skali ogonowej i zaloguj się do tej samej sieci ogonowej.

- Jak propagować polecenia pomiędzy Telegramem a węzłami
- Nie znaleziono danych logowania dla profilu "anthropic:default"
- Plik znajduje się w granicach rozmiaru dostawcy (obrazy są przeskalowane do maksymalnie 2048px).

Jeśli korzystasz z systemu Windows, \*\*WSL2 jest najprostszą konfiguracją stylu VM \*\* i ma najlepszą kompatybilność z oprogramowaniem
. Zobacz [Windows](/platforms/windows), [hosting] VPS (/vps).
Jeśli używasz macOS w VM, zobacz [macOS VM](/install/macos-vm).

## Naprawda: albo podaj autoryzację Google, albo usuń/unikaj modeli Google w `agents.defaults.model.fallbacks` / aliasy, więc rezerwacja nie jest tam trasą.

### Następnie `/model sonnet` (lub `/<alias>` gdy obsługiwany) rozwiązuje się do tego modelu ID.

OpenClaw jest osobistym asystentem AI, który pracujesz na własnych urządzeniach. Odpowiada na powierzchnie wiadomości, które już używasz (WhatsApp, Telegram, Slack, Mattermost (plugin), Discord, Czat Google, Signal, iMessage, WebChat) i może również głosować + na żywo płótno na obsługiwanych platformach. \*\*Brama \*\* jest samolotem kontrolnym zawsze używanym; asystentem jest produkt.

### Napraw listę kontrolną dla braku danych logowania dla profilu antropikalnego.

OpenClaw to nie tylko opakowanie Claude To **lokalny samolot kontrolny**, który pozwala Ci uruchomić
zdolnego asystenta na **własnym urządzeniu**, osiągalny dzięki aplikacjom na czacie, które już używasz, za pomocą
sesji, pamięci i narzędzi - bez kontroli przepływu pracy do hostowanego
SaaS.

\*\*Potwierdź, że Twój var env jest załadowany przez bramę \*\*

- Dlaczego widzę nieznany model minimaxMiniMaxM21
- Czy mogę kontrolować, który profil autoryzacji jest najpierw wypróbowany
- Gdzie są logi
- Brama otwarta ELI5 zrestartowana vs openclaw bateway
- Gdzie OpenClaw przechowuje swoje dane
- WhatsApp będzie wysyłać wiadomości do moich kontaktów, jak działa parowanie

Po zainstalowaniu usługi:

### Brak widocznych okien przeglądarki (użyj zrzutów ekranu, jeśli potrzebujesz wizualności).

**Obrót profilu uwierzytelniania** w ramach tego samego dostawcy.

- Dokujący czuje się ograniczony jak włączyć pełne funkcje
- Napraw na czacie gdzie go widzisz:
- Użyj jednej z opcji:
- **2) openclaw nie jest rozpoznany po zainstalowaniu**

Windows install mówi, że git nie został znaleziony lub openclaw nie został rozpoznany

### Ustaw `PLAYWRIGHT_BROWSERS_PATH` i upewnij się, że ścieżka jest kontynuowana.

Oznacza to, że operacja jest przypięta do profilu Antropowego autoryzacji, ale brama
nie może znaleźć jej w swoim sklepie autoryzacyjnym.

- Jeśli korzystasz z Tailscale Serve, upewnij się, że plik `gateway.auth.allowTailscale` jest poprawnie ustawiony.
- Jak podłączyć węzeł Mac do zdalnej bramy Gateway Tailscale Serve
- Jak mój agent ma dostęp do mojego komputera, jeśli brama jest zdalnie hostowana
- Jeśli połączysz się przez tunel SSH, potwierdź, że miejscowy tunel jest w górę i wskazuje w prawym porcie.
- Jakie są typowe identyfikatory profilu

### Sprawdź oczekujące żądania:

Tak dla **badań, kwalifikacji i projektowania**. Może skanować strony, budować krótkie listy,
podsumować perspektywy i pisać kontakty informacyjne lub kopiować szkice reklam.

Dla **działań informacyjnych lub reklamowych**, zachowaj ludzkość w pętli. Unikaj spamu, przestrzegaj lokalnych przepisów i zasad platformy
i sprawdzaj cokolwiek zanim zostanie wysłane. The safest pattern is to let
OpenClaw draft and you approve.

Jaki jest najszybszy sposób, aby uzyskać więcej szczegółów gdy coś się nie powiedzie

### Jeśli chcesz, aby \*\*ty \*\* był w stanie wyzwalać odpowiedzi grupowe:

OpenClaw jest **osobistym asystentem** i warstwą koordynacyjną, a nie zastępcą IDE. Użyj
Claude Code lub Codex dla najszybszej bezpośredniej pętli kodowania wewnątrz repozytorium. Użyj OpenClaw, gdy
chcesz trwałą pamięć, dostęp między urządzeniami i orkiestrację narzędzi.

Szukaj `chatId` (lub `from`) kończącego się w `@g.us`, jak
`1234567890-1234567890@g.us`.

- Przewodnik wyjściowy:
- Naprawa:
- Unikaj:
- Uwagi:
- Porażka odbywa się w dwóch etapach:

Doktor wykrywa niedopasowanie wejściowego punktu usługi i oferuje przepisanie konfiguracji usługi do dopasowania do bieżącej instalacji (użyj `--repair` w automatyzacji).

## Umieść `ANTHROPIC_API_KEY` w `~/.openclaw/.env` na **hostze bramy**.

### `openclaw gateway`: uruchamia bramę \*\*na pierwszym planie \*\* dla tej sesji terminalu.

Użyj zarządzanych nadpisań zamiast edycji kopii repozytorium. Umieść swoje zmiany w `~/.openclaw/skills/<name>/SKILL.md` (lub dodaj folder za pomocą `skills.load.extraDirs` w `~/.openclaw/openclaw.json`). Precedence is `<workspace>/skills` > `~/.openclaw/skills` > wiązane, więc zarządzane nadpisywanie wygrywa bez dotykania git. Tylko edycje warte na wcześniejszych etapach powinny żyć w repozytorium i wychodzić jako PR.

### Większość komunikatów wewnętrznych lub komunikatów narzędzi pojawia się tylko wtedy, gdy **verbose** lub **rozumowanie** jest włączone&#xA;dla tej sesji.

Tak. Dodaj dodatkowe katalogi za pomocą `skills.load.extraDirs` w `~/.openclaw/openclaw.json` (najniższy pierwszeństwo). Domyślny pierwszeństwo pozostało: `<workspace>/skills` → `~/.openclaw/skills` → połączone → `skills.load.extraDirs`. `clawhub` instaluje się domyślnie do `./skills`, co OpenClaw traktuje jako `<workspace>/skills`.

### Dziś nie jest wbudowany.

Zobacz [Troubleshooting](/gateway/troubleshooting#log-locations) po więcej.

- Media i załączniki
- Wczytywanie var i .env
- Dlaczego również wypróbowała Google Gemini i zakończyła się niepowodzeniem

Telegram co jest dozwolone

### Więcej kontekstu: [Models](/concepts/models).

Użyj **subagentów** do długich lub równoległych zadań. Subagenci działają na własnej sesji,
zwraca podsumowanie i zachowaj odpowiedź głównego czatu.

Poproś bota o "spawn subagent dla tego zadania" lub użyj `/subagents`.
Użyj `/status` na czacie, aby zobaczyć, co brama robi teraz (i czy jest zajęta).

Wskazówka: długie zadania i subagenci zużywają tokeny. Jeśli koszt jest obawą, ustaw tańszy model
dla podagentów poprzez `agents.defaults.subagents.model`.

Jak całkowicie zatrzymać, a następnie uruchomić bramę

### Tak, jeśli Twój prywatny ruch to **DM**, a Twój ruch publiczny to **grupy**.

Cron działa wewnątrz procesu bramy. Jeśli brama nie działa ciągle, zaplanowane zadania
nie będą działać.

Użyj modelu z większym oknem kontekstowym.

- Masz koncepcję lub integrację HeyGen
- **Czy mogę zachować osobiste emotikony, ale upublicznić grupy z jednym agentem**
- Minimalne kroki:

Debug:

```bash
Uwagi:
```

instalacja bramy openclaw --force

### Kanał docelowy obsługuje media wychodzące i nie jest blokowany przez dozwolone listy.

Użyj **ClawHub** (CLI) lub umieść umiejętności w swoim obszarze roboczym. Umiejętności macOS nie są dostępne na Linux.
Przeglądaj umiejętności na [https://clawhub.com](https://clawhub.com).

/wzór 3

```bash
npm i -g clawhub
```

```bash
pnpm add -g clawhub
```

### **Użyj tokenu konfiguracyjnego**

Tak. Użyj harmonogramu bram:

- Jeśli tak się stanie:
- Jak całkowicie odinstalować OpenClaw
- Częste konfiguracje:

Jaka jest minimalna konfiguracja dla pierwszej instalacji

### Dokumenty: [Telegram](/channels/telegram), [Rozwiązywanie problemów z kanałem](/channels/troubleshooting).

Niebezpośrednio. umiejętności macOS są bramkowane przez `metadata.openclaw.os` plus wymagane binary, a umiejętności pojawiają się tylko wtedy, gdy kwalifikują się na **Gateway host**. Na Linuksie, umiejętności `darwin`-only (takie jak `apple-notes`, `apple-reminders`, `things-mac`) nie będą ładowane, chyba że nadpiszesz bramkę.

Poproś agenta o pobranie tej strony na początku sesji.

\*\*Opcja A - uruchom bramę na Mac (najprostszy). \*
Uruchom bramę, w której istnieją pliki binarne macOS, a następnie połącz się z Linux w [trybie zdalnym](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere) lub w skali Tailscale. Zwykle ładuje się umiejętności, ponieważ gospodarzem bramy jest macOS.

\*\*Opcja B - użyj węzła macOS (bez SSH). \*
Uruchom bramę na Linux, paruj węzeł macOS (aplikacja paska menu), i ustaw **polecenia uruchamiania węzła** na "Always Ask" lub "Always Allow" na Mac. OpenClaw może traktować umiejętności tylko macOS jako kwalifikujące się, gdy wymagane pliki binarne istnieją na węźle. Agent obsługuje te umiejętności za pomocą narzędzia `nodes`. Jeśli wybierzesz "Zawsze pytaj się", zatwierdzając "Zawsze Zezwalaj" w okienku dodaje to polecenie do listy dozwolonych.

\*\*Opcja C - proxy macOS w SSH (zaawansowane). \*
Zachowaj bramę na Linux, ale spraw, aby wymagane pliki binarne CLI rozwiązywały na opakowania SSH uruchamiane na Mac. Następnie nadpisz umiejętność, aby umożliwić Linuksowi więc pozostanie kwalifikowalna.

1. Interfejs sterowania mówi, że jest nieautoryzowany lub nadal łączy się ponownie co teraz

   ```bash
   Lista oczekujących żądań:
   ```

2. Model: przelover i "Wszystkie modele nie powiodły się"

3. Sesje i wiele czatów

   ```markdown
   Nieinteraktywne pełne zresetowanie:
   ```

4. Fakty (z kodu):

### **Pros:** brak kosztów serwera, bezpośredni dostęp do plików lokalnych, okno przeglądarki na żywo.

Jeśli chcesz zachować kontekst dla każdego klienta (przepływ pracy agencji), prosty wzór:

**Nie** ponownie użyj `agentDir` przez różnych agentów; powoduje kolizje z autentykacją/sesją.

- Jeśli działasz na pierwszym planie, zatrzymaj się z Ctrl-C, następnie:
- Jeśli to było niespodziewane, utwórz błąd i dołącz ostatnią znaną konfigurację lub dowolną kopię zapasową.

Kontrola bezpieczeństwa i dostępu

- Sprawdź ścieżkę:
- Opcje naprawy:

Typowe przyczyny:

`interrupt` - przerwanie bieżącego biegu i rozpoczęcie nowego

```bash
Organizuj pliki i foldery (czyszczenie, nazwa, znakowanie).
```

ClawHub instaluje się w `. umiejętności` w bieżącym katalogu (lub powróci do skonfigurowanego obszaru roboczego OpenClaw); OpenClaw traktuje to jako `<workspace>/skills` podczas następnej sesji. W przypadku wspólnych umiejętności między agentami umieść je w `~/.openclaw/skills/<name>/SKILL.md`. Niektóre umiejętności oczekują plików binarnych zainstalowanych przez Homebrew; na Linux to znaczy Linuxbrew (zobacz powyższy wpis Homebrew Linux FAQ). Zobacz [Skills](/tools/skills) i [ClawHub](/tools/clawhub).

### OpenClaw to nie tylko opakowanie Claude To **lokalny samolot kontrolny**, który pozwala Ci uruchomić&#xA;zdolnego asystenta na **własnym urządzeniu**, osiągalny dzięki aplikacjom na czacie, które już używasz, za pomocą&#xA;sesji, pamięci i narzędzi - bez kontroli przepływu pracy do hostowanego&#xA;SaaS.

**Opcja A: przełącznik na sesję**

```bash
openclaw browser extension install
openclaw browser extension path
```

wsl
status bramki otwartej
zrestartuj bramę otwierania

Jeśli ustawisz swój własny alias o tej samej nazwie, Twoja wartość wygrywa.

Jeśli brama działa na tym samym urządzeniu co Chrome (domyślna konfiguracja), zazwyczaj \*\*nie potrzebujesz niczego dodatkowego.
Jeśli Gateway działa gdzie indziej, uruchom host węzła na maszynie z przeglądarką, aby Gateway mógł pośredniczyć w akcjach przeglądarki.
Nadal musisz kliknąć przycisk rozszerzenia na karcie, którą chcesz kontrolować (nie jest to automatyczne załącznik).

## Uaktualnij do **2026.1.12** (lub uruchom z źródłowego `main`), a następnie zrestartuj bramę.

### Czy jest dedykowany dok piaskownicy

Tak. Zobacz [Sandboxing](/gateway/sandboxing). Ustawienia specyficzne dla dokera (pełna brama w oknie dokującym lub piaskownicach) zobacz [Docker](/install/docker).

### Zbuduj stronę internetową (WordPress, Shopify lub prostą witrynę statyczną).

The default image is security-first and runs as the `node` user, so it does not
include system packages, Homebrew, or bundled browsers. Dla pełniejszej konfiguracji:

- Jaki model rekomendujesz
- Bądź na ekranie z `OPENCLAW_DOCKER_APT_ULOTK`.
- `hot`, `restart`, `off` są również obsługiwane
- Można używać jednego numeru WhatsApp z różnymi instancjami OpenClaw

Czy są wbudowane skróty gp'a opus sonnet

Ustaw `gateway.mode: "remote"` i ustaw zdalny adres URL WebSocket, opcjonalnie z tokenem/hasłem:

Napraw listę kontrolną:

Użyj `agents.defaults.sandbox.mode: "non-main"` więc sesje grupowe/kanały (klucze niegłówne) uruchamiane w Docker, podczas gdy główna sesja DM pozostaje na serwerze. Następnie ograniczyć, jakie narzędzia są dostępne w sesjach piaskownicowych za pomocą `tools.sandbox.tools`.

Odpowiedz na dokładne pytanie ze zrzutu ekranu/czatu

Wyczyść stare sesje (usuń JSONL lub zapisy sklepu), jeśli dysk rośnie.

### Naprawa:

Ustaw `agents.defaults.sandbox.docker.binds` na `["host:path:mode"]` (np. `"/home/user/src:/src:ro"`). Globalny + per-agent wiąże połączenia; powiązania dla agenta są ignorowane gdy `zakres: "dzielona"`. Użyj `:ro` dla wszystkiego, co jest wrażliwe i zapamiętaj pominąć ściany systemu plików sandbox. Zobacz [Sandboxing](/gateway/sandboxing#custom-bind-mounts) i [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated#bind-mounts-security-quick-check), aby uzyskać przykłady i informacje o bezpieczeństwie.

### Edytujesz jeden plik konfiguracyjny podczas uruchamiania innej usługi (często niezgodność `--profile` / `OPENCLAW_STATE_DIR`).

Zobacz [Groups](/channels/groups) i [Wiadomości grupowe](/channels/group-messages).

- Jest sposób na stworzenie zespołu OpenClaw instancji jednego CEO i wielu agentów
- Dwa wspólne przyczyny:

OpenClaw uruchamia również **cichą pamięć kompaktową**, aby przypomnieć modelowi
o zapisywaniu trwałych notatek przed automatycznym zagęszczaniem. To działa tylko wtedy, gdy obszar roboczy
jest zapisywalny (pomijaj piaskownice tylko do odczytu). Zobacz [Pamięć](/concepts/memory).

### **Bicie serca** dla okresowych kontroli "głównej sesji".

Poproś bota o **zapis faktu do pamięci**. Notatki długoterminowe należą do `MEMORY.md`,
kontekst krótkoterminowy zawiera `memory/YYYY-MM-DD.md`.

Jest to obszar, który wciąż się poprawiamy. Pomaga to przypomnieć modelowi o przechowywaniu pamięci;
będzie wiedział, co robić. Jeśli zapomni, sprawdź czy brama używa tego samego obszaru roboczego
przy każdym uruchomieniu.

Zdalne bramy i węzły

### Następnie:

Tylko jeśli używasz \*\*osadzonych OpenAI \*\*. Codex OAuth pokrywa czat/uzupełnienia, a
**nie** udziela dostępu do osadzeń, więc **zalogowanie się za pomocą Codex (OAuth lub
Codex CLI)** nie pomaga w wyszukiwaniu pamięci semantycznej. OpenAI embeddings
nadal potrzebuje prawdziwego klucza API (`OPENAI_API_KEY` lub `models.providers.openai.apiKey`).

Jeśli nie ustawiłeś wyraźnie, OpenClaw automatycznie wybiera dostawcę, gdy
może rozwiązać klucz API (profile auth profile, `models.providers.*.apiKey`, lub env vars).
Wolą OpenAI jeśli klucz OpenAI rozwiązuje, w przeciwnym razie Gemini jeśli klucz Gemini
się rozwiązuje. Jeśli żaden klucz nie jest dostępny, wyszukiwanie pamięci pozostaje wyłączone do czasu konfiguracji
. Jeśli masz skonfigurowaną i obecną lokalną ścieżkę modelu, OpenClaw
preferuje `local`.

Jeśli chcesz pozostać lokalnym, ustaw `memorySearch.provider = "local"` (i opcjonalnie
`memorySearch.fallback = "none"`). Jeśli chcesz, aby Gemini embeddings, ustaw
`memorySearch.provider = "gemini"` i wprowadź `GEMINI_API_KEY` (lub
`memorySearch.remote.apiKey`). Wspieramy **OpenAI, Gemini, lub local** osadzanie modeli\* zobacz [Memory](/concepts/memory), aby uzyskać szczegóły konfiguracji.

### **Domyślnie + switch:** ustaw `agents.defaults.model.primary` na `openai/gpt-5.2`, a następnie przełącz na `openai-codex/gpt-5.3-codex` podczas kodowania (lub w odwrotnym kierunku).

Pliki pamięci na żywo na dysku i utrzymują się aż do ich usunięcia. Limit jest twoim magazynem
a nie modelu. **Kontekst sesji** jest nadal ograniczony przez model
kontekstu, tak długie rozmowy mogą być kompaktowe lub obcięte. Dlatego wyszukiwanie pamięci
istnieje - pociąga tylko odpowiednie części z powrotem do kontekstu.

Następnie zrestartuj bramę i sprawdź:

## `gateway.port` kontroluje pojedynczy multipleksowany port dla WebSocket + HTTP (Control UI, hooks, itp.).

### Typowe przyczyny:

Zainstaluj umiejętności:

- Lista kontrolna:
- **Zdalnie od konieczności:** wiadomości wysyłane do dostawców modeli (Antropiczka/OpenAI/itp.) przejdź do
  ich API i platformy czatu (WhatsApp/Telegram/Slack/etc.) przechowuj dane wiadomości na ich serwerach
  .
- Otrzymuję kontekst zbyt duże błędy jak zresetować lub kompaktować

Zamknij i ponownie otwórz PowerShell po aktualizacji PATH.

### Konfiguracja walkthrough + przykładowa konfiguracja: [Grupy: osobiste DM + publiczne grupy](/channels/groups#pattern-personal-dms-public-groups-single-agent)

Potrzebuję oddzielnych VPS dla wielu agentów

| Zweryfikuj ustawienia strefy czasowej dla zadania (`--tz` vs host zone). | Zresetuj sesje automatycznie, jeśli nigdy nie wysyłam nowych                                                                      |
| ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| `$OPENCLAW_STATE_DIR/openclaw.json`                                                                         | Węzły nie widzą ruchu przychodzącego; odbierają tylko połączenia RPC z węzłem.                                    |
| `$OPENCLAW_STATE_DIR/credentials/oauth.json`                                                                | Jakie są minimalne wymogi VPS i zalecane systemy operacyjne.                                                      |
| `$OPENCLAW_STATE_DIR/agents/&lt;agentId&gt;/agent/auth-profiles.json`                                             | Krótka odpowiedź: postępuj zgodnie z przewodnikiem Linux, a następnie uruchom kreatora wdrożenia. |
| `$OPENCLAW_STATE_DIR/agents/&lt;agentId&gt;/agent/auth.json`                                                      | Pamięć podręczna autoryzacji (zarządzana automatycznie)                                                        |
| `$OPENCLAW_STATE_DIR/credentials/`                                                                          | Uruchom wszystko na Macie, jeśli chcesz najprostszą konfigurację jednomaszynową.                                  |
| `$OPENCLAW_STATE_DIR/agents/`                                                                               | Dlaczego widzę model nie jest dozwolony, a następnie nie ma odpowiedzi                                                            |
| `$OPENCLAW_STATE_DIR/agents/&lt;agentId&gt;/sessions/`                                                            | Jeśli używasz dozwolonych list dodaj `web_search`/`web_fetch` lub `group:web`.                                    |
| `$OPENCLAW_STATE_DIR/agents/&lt;agentId&gt;/sessions/sessions.json`                                               | Nie - **Stan OpenClaw jest lokalny**, ale **zewnętrzne usługi wciąż widzą, co je wysyłasz**.                      |

To, co robi inna instancja bramy, to już nasłuchiwanie średniej

Twój **obszar roboczy** (AGENTS.md, pliki pamięci, umiejętności itp.) jest oddzielny i skonfigurowany przez `agents.defaults.workspace` (domyślnie: `~/.openclaw/workspace`).

### Typowa konfiguracja:

Jak rozpocząć nową rozmowę

- Rozpocznij z logami i statusem kanału:
- Może pracować agenci poza obszarem roboczym

Nie jest wymagany oddzielny most TCP; węzły łączą się przez bramę WebSocket.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

Utknęłem to, co najszybszy sposób na odklejenie

Wskazówka: jeśli chcesz zachować lub preferencje, poproś bota o \*\*zapisanie go do
AGENTS. a nie polegać na historii czatu.

brama openclaw stop
zaczyna się brama otwierania

### Użyj `openclaw --profile <name> …` na instancję (automatycznie tworzy `~/.openclaw-<name>`).

Umieść swój **obszar roboczy agenta** w **prywatnym** repozytorium git i utworzyć kopię zapasową gdzieś
prywatny (na przykład GitHub prywatny). To przechwytuje pamięć + AGENTS/SOUL/USER
i pozwala przywrócić "umysł" asystenta później.

**Nie** zatwierdzaj cokolwiek pod `~/.openclaw` (poświadczenia, sesje, tokeny).
Jeśli potrzebujesz pełnego przywrócenia, wykonaj kopię zapasową zarówno obszaru roboczego, jak i katalogu stanu
osobno (patrz pytanie migracji powyżej).

Brama jest aktywna, ale odpowiedzi nigdy nie dotrą do co powinienem sprawdzić

### Jeden agent na rolę (wiążące).

Skala ogona jest połączona, ale nie otrzymuję odpowiedzi Co teraz

### Dokumenty: [Compaction](/concepts/compaction), [Session pruning](/concepts/session-pruning), [Zarządzanie sesją](/concepts/session).

Tak. Obszar roboczy jest **domyślnym cwd** i kotwicą pamięci, a nie twardym piaskownikiem.
Ścieżki względne rozwiązują wewnątrz obszaru roboczego, ale ścieżki bezwzględne mogą mieć dostęp do innych lokalizacji hosta
, chyba że sandboxing jest włączony. Jeśli potrzebujesz izolacji, użyj ustawień
[`agents.defaults.sandbox`](/gateway/sandboxing) lub sandboxa dla agenta Jeśli potrzebujesz izolacji, użyj ustawień
[`agents.defaults.sandbox`](/gateway/sandboxing) lub sandboxa dla agenta Jeśli
chcesz, aby repozytorium było domyślnym katalogiem roboczym, ustaw ten agent
`workspace` na roota repozytorium. Repozytorium OpenClaw jest tylko kodem źródłowym; zachowaj oddzielny obszar roboczy
chyba że celowo chcesz, aby agent działał wewnątrz niego.

Jeśli jesteś zdalny, potwierdź połączenie tunelu/skali ailscale i że WebSocket bramki
jest osiągalny.

```json5
{
  agents: {
    defaults: {
      workspace: "~/Projects/my-repo",
    },
  },
}
```

### Dokumenty: [Myślenie i werbose](/tools/thinking), [Security](/gateway/security#reasoning--verbose-output-in-groups).

Stan sesji jest własnością **hosta bramy**. Jeśli jesteś w trybie zdalnym, magazyn sesyjny, którego jesteś zainteresowany, znajduje się na urządzeniu zdalnym, a nie na lokalnym laptopie. Zobacz [Zarządzanie sesją](/concepts/session).

## Starsza ścieżka jednego agenta: `~/.openclaw/agent/*` (migrowana przez `openclaw do`).

### Naprawda: zatrzymaj drugą instancję, uwolnij port lub uruchom `openclaw gateway --port <port>`.

Jaka jest zalecana strategia tworzenia kopii zapasowych

```
$OPENCLAW_CONFIG_PATH
```

Dlaczego bot lekceważy komunikaty o szybkim ogniu

### Dokumenty: [Memory](/concepts/memory), [Agent workspace](/concepts/agent-workspace).

Powiązania niezwiązane z pętlą **wymagają autora**. Konfiguruj `gateway.auth.mode` + `gateway.auth.token` (lub użyj `OPENCLAW_GATEWAY_TOKEN`).

```json5
{
  gateway: {
    bind: "lan",
    auth: {
      mode: "token",
      token: "replace-me",
    },
  },
}
```

Jak uruchomić OpenClaw w trybie zdalnym klient łączy się z bramą gdzie indziej

- Zacznij od szybkiego przeglądu zdrowia:
- Interfejs sterowania uwierzytelnia się przez `connect.params.auth.token` (przechowywane w ustawieniach app/UI). Unikaj wprowadzania tokenów w adresach URL.

### Zobacz [MiniMax](/providers/minimax) i [Models](/concepts/models).

Kreator generuje domyślny token bramy (nawet przy pętli), więc **lokalni klienci WS muszą uwierzytelniać się**. To blokuje inne lokalne procesy wywoływania bramy. Wklej token do ustawień Control UI (lub konfiguracji klienta), aby się połączyć.

Jeśli **naprawdę** chcesz otworzyć pętlę, usuń `gateway.auth` ze swojej konfiguracji. Doktor może wygenerować token dla Ciebie w dowolnym momencie: `openclaw doctor --generate-gateway-token`.

### **Kanały rzeczywiste, a nie piaskownica internetowa:** WhatsApp/Telegram/Slack/Discord/Signal/iMessage/etc,&#xA;plus głos mobilny i płótno na obsługiwanych platformach.

**Obszar roboczy (na agenta)**: `AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`,
`MEMORY.md` (lub `memory.md`), `memory/YYYY-MM-DD.md`, opcjonalny `HEARTBEAT.md`.

- Czy wspierasz autoryzację subskrypcji Claude (Claude Pro lub Max)
- Wyślij dowolną z tych **jako samodzielną wiadomość** (bez ukośnika):

### Podmiot trzeci (mniej prywatne):

`web_fetch` działa bez klucza API. `web_search` wymaga klucza Brave Search API
. **Zalecane:** uruchom `openclaw configure --section web` aby przechowywać go w
`tools.web.search.apiKey`. Alternatywa środowiska naturalnego: ustaw `BRAVE_API_KEY` dla procesu bramy
.

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "BRAVE_API_KEY_HERE",
        maxResults: 5,
      },
      fetch: {
        enabled: true,
      },
    },
  },
}
```

Ustawia Twój obszar roboczy i ogranicza, kto może wyzwalać bota.

- Ustawiam COPILOTGITHUBTOKEN, ale status modeli pokazuje Shell env off Dlaczego
- Najszybszy ogon dziennika:
- Podstawy konfiguracji

Jest szybkim wstrzyknięciem tylko dla robotów publicznych

### Zalety:

Gdzie powinien być AGENTSmd SOULmd USERmd MEMORYmd żyć

- Bot zamraża się podczas wykonywania ciężkiej pracy Jak je odładuję
- Czym jest profil autoryzacji
- Jak zainstalować OpenClaw na VPS
- Logowanie i debugowanie
- Pamięć nieustannie zapomina o rzeczach, jak to zrobić

Zamknąłem terminal w systemie Windows, jak zrestartuję OpenClaw

### DM twojego bota, a następnie wywołaj `https://api.telegram.org/bot<bot_token>/getUpdates` i przeczytaj `message.from.id`.

Tak. Jest to opcja konfiguracyjna:

```json5
{
  browser: { headless: true },
  agents: {
    defaults: {
      sandbox: { browser: { headless: true } },
    },
  },
}
```

Domyślnie jest `false` (headful). Prawdopodobieństwo uruchomienia kontroli przeciwbotów w niektórych miejscach jest bardziej prawdopodobne. Zobacz [Browser](/tools/browser).

Bezgłowy używają **tego samego silnika chromowego** i pracują dla większości automatyzacji (formularze, kliknięcia, skrawanie, logowania). Główne różnice:

- **Subagentów**: trasa zadań do odrębnych agentów z różnymi modelami domyślnymi.
- Niektóre strony są bardziej restrykcyjne w zakresie automatyzacji w trybie bezgłowy (CAPTCHA, antybot).
  Na przykład X/Twitter często blokuje sesje bezgłowne.

### Powiązano: [/concepts/oauth](/concepts/oauth) (OAuth flows, pamięć tokenów, wzory wielu kont)

Ustaw `browser.executablePath` na swój plik binarny Brave (lub dowolną przeglądarkę opartą na Chromium) i zrestartuj bramę.
Zobacz pełne przykłady konfiguracji w [Browser](/tools/browser#use-brave-or-another-chromium-based-browser).

## Lub skopiuj `auth-profiles.json` z `agentDir` głównego konsultanta do `agentDir`.

### Naprawa:

Wiadomości Telegram są obsługiwane przez **bramę**. Brama uruchamia agenta i
tylko wtedy wywołuje węzły nad **Gateway WebSocket**, gdy potrzebne jest narzędzie węzła:

Sprawdź podstawy:

Mogę używać Buna

### i wybierz z listy (lub `/model list` na czacie).

Krótka odpowiedź: **sparować komputer jako węzeł**. Brama działa gdzie indziej, ale może
wywołać narzędzia `node.*` (ekran, kamera, system) na lokalnej maszynie przez Gateway WebSocket.

za pomocą agenta "czytnika" tylko do odczytu lub wyłączonego narzędziami do podsumowania niezaufanych treści

1. **Wariant B: odrębne agenty**
2. Oznacza to, że system próbował użyć identyfikatora profilu autoryzacji `anthropic:default`, ale nie mógł znaleźć poświadczeń w oczekiwanym sklepie autoryzacyjnym.
3. Często wzorze:
4. Profil uwierzytelniania: co oni są i jak zarządzać nimi
5. /kompaktowy

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

prototyp aplikacji mobilnej (obrys, ekrany, plan API).

Przypomnienie bezpieczeństwa: parowanie węzła macOS pozwala na `system.run` na tym komputerze. Tylko
paruj urządzenia, które ufasz, i przejrzyj [Security](/gateway/security).

Zamknij i ponownie otwórz PowerShell, a następnie uruchom ponownie instalator.

### Dokumentacja: [Web tools](/tools/web).

**Rozrost dysku:** sesje + transkrypty są dostępne pod `~/.openclaw/agents/<agentId>/sessions/`.

- **VPS / chmura**
- Zobacz [Obszar roboczy Agenta](/concepts/agent-workspace) i [Memory](/concepts/memory).
- Jak włączyć wyszukiwanie i pobieranie stron internetowych

Żaden plik `.env` nie nadpisuje istniejących zmiennych środowiskowych.

- Dwa częste problemy z systemem Windows:
- **Jak odpiąć profil ustawiony z profilem**
- openclaw update
  openclaw update
  openclaw update --channel stable|beta|dev
  openclaw update --tag <dist-tag|version>
  openclaw update --no-restart

Czy AWS jest obsługiwany

### Użyj dokładnego identyfikatora modelu (rozróżniana jest wielkość liter): `minimax/MiniMax-M2.1` lub&#xA;`minimax/MiniMax-M2.1-lightning`.

Tak. Nie ma wbudowanego mostu „bot-to-bot”, ale możesz go zdemontować za kilka
niezawodnych sposobów:

**Proste:** użyj normalnego kanału czatu, do którego mają dostęp oba boty (Telegram/Slack/WhatsApp).
Bota A wyśle wiadomość do bota B, a następnie pozwól botowi B odpowiedzieć jak zwykle.

**Most CLI (generic):** uruchom skrypt, który wywołuje drugą bramę z
`openclaw agent --mess... --deliver`, ukierunkowanie czatu, w którym słucha innego bota
. Jeśli jeden bot jest na zdalnym VPS, ustaw swój CLI na zdalnej bramie
przez SSH/Tailscale (zob. [zdalny dostęp] (/gateway/remote)).

Wybór **Ocena zdrowia** i **umiejętności**

```bash
openclaw agent --message "Hello from local bot" --deliver --channel telegram --reply-to <chat-id>
```

Jak otworzyć pulpit po wprowadzeniu

Czy muszę kupić Mac Mini aby to zainstalować

### **Dziennik stanu (`~/.openclaw`)**: config, dane logowania, profile uwierzytelniania, sesje, dzienniki,&#xA;i wspólne umiejętności (`~/.openclaw/skills`).

Nie. Jedna brama może hostować wielu agentów, z których każdy ma własny obszar roboczy, domyślne modele,
i routing. To jest normalna konfiguracja i jest o wiele tańsza i prostsza niż działanie
jednego VPS na agenta.

Używaj oddzielnych VPSes tylko wtedy, gdy potrzebujesz twardej izolacji (granice bezpieczeństwa) lub bardzo
różnych konfiguracji, których nie chcesz udostępniać. W przeciwnym razie utrzymuj jedną bramę i
używaj wielu agentów lub subagentów.

### **Przełączanie awaryjne modelu** do następnego modelu w `agents.defaults.model.fallbacks`.

Yes - nodes are the first-class way to reach your laptop from a remote Gateway, and they
unlock more than shell access. Brama działa na macOS/Linux (Windows via WSL2) i jest
lekka (małe pole klasy VPS lub Raspberry Pi-class jest w porządku; 4 GB pamięci RAM jest duży), więc powszechnie używanym
jest zawsze domieszką hosta oraz laptopa jako węzeł.

- Codzienna wygrana wygląda zwykle jako:
- Jeśli musisz zautomatyzować od agenta:
- Upewnij się, że miniMax jest skonfigurowany (kreator lub JSON) lub że miniMax klucz API
  istnieje w profilach env/auth, aby dostawca mógł być wstrzyknięty.
- \*\*Automatyzacja lokalnej przeglądarki. \* Zachowaj bramę w sieci VPS, ale uruchom Chrome lokalnie i przekaż kontrolę
  z rozszerzeniem Chrome + hostem węzła na laptopie.

Wyczyść dowolną przypiętą kolejność, która wymusza brakujący profil:

Jak mogę utrzymać ruch modelowy w określonym regionie

### Otwierasz adres URL **HTTP** w przeglądarce (`http://...`) zamiast klienta WS.

Jeśli potrzebujesz tylko **narzędzi lokalnych** (ekran/kamera/exec) na drugim laptopie, dodaj go jako
**węzeł**. To utrzymuje jedną bramę i pozwala uniknąć duplikowanej konfiguracji. Narzędzia węzła lokalnego są
obecnie tylko macOS, ale planujemy rozszerzyć je na inne systemy operacyjne.

Jaki jest model domyślny

Jak działa autoryzacja antropowego setuptoken

### Więcej informacji: [Install](/install) i [flagi instalatora](/install/installer).

Nie. Tylko **jedna brama** powinna działać na hosta, chyba że celowo uruchomisz oddzielone profile (patrz [Wiele bramek](/gateway/multiple-gateways)). Węzły są peryferiami, które łączą się
z bramą (iOS/Android nodes, lub "tryb węzła" macOS w aplikacji paska menu). Dla węzła bezgłowy
hosty i kontrola CLI zobacz [CLI hosta](/cli/node).

Może obsługiwać duże zadania, ale działa najlepiej, gdy podzielisz je na fazy i
używasz podagentów do pracy równoległej.

### Zainstaluj drugą bramę tylko wtedy, gdy potrzebujesz **twardej izolacji** lub dwóch całkowicie oddzielnych botów.

Tak. `config.apply` sprawdza + zapisuje pełną konfigurację i uruchamia bramę ponownie jako część operacji.

### Zdrowie bramy : `openclaw status`

`config.apply` zastępuje **całą konfigurację**. Jeśli wyślesz obiekt częściowy, wszystko
zostało usunięte.

Jak mogę używać różnych modeli dla różnych zadań

- Zalecane konfiguracje:
- Dlaczego teraz potrzebuję tokenu na localhost
- SSH jest dobry dla dostępu do powłoki ad-hoc, ale węzły są prostsze dla bieżących przepływów pracy agenta i
  automatyzacji urządzenia.
- Zatwierdź parowanie z:

`.env` z bieżącego katalogu roboczego

- Cron lub przypomnienia nie wystrzeliwują Co należy sprawdzić
- [VPS hosting](/vps) (wszyscy dostawcy w jednym miejscu)

czyste zlecenie autoryzacji modeli openclaw --provider antropic

### Historia konwersacji i stan (na agenta)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

Dobre pierwsze projekty:

### Zaawansowane długoterminowe notatki w `MEMORY.md` (tylko sesje główne/prywatne)

Zobacz [/environment](/help/environment), aby poznać pełną kolejność i źródła.

1. Modele: domyślne, wybór, aliasy, przełączanie

   ```bash
   Trzymaj DM w **trybie parowania** lub w ścisłej liście dozwolonych.
   ```

2. Jeśli używasz CLI lub TUI, adres URL powinien wyglądać jako:
   - Szybkie poprawki:

3. Jeśli nadal chcesz eksperymentować z Bunem, zrób to na bramie nieprodukcyjnej
   bez WhatsApp/Telegram.
   - Jak uzyskać JID grupy WhatsApp

4. Opcja 1 (najszybsza): kłody ogona i wysłanie komunikatu testowego w grupie:
   - SSH: `ssh user@your-vps.tailnet-xxxx.ts.net`
   - Użyj podczynników do pracy długiej lub równoległej, aby główny czat pozostawał mniejszy.

Jak działa pamięć

```bash
openclaw gateway --tailscale serve
```

Utrzymuje bramy związane z pętlą i wystawia HTTPS przez Tailscale. Zobacz [Tailscale](/gateway/tailscale).

### {&#xA;sesja: {&#xA;idleMinutes: 240,&#xA;},&#xA;}

Służba ujawnia interfejs interfejsu użytkownika Bramki + WS\*\*. Węzły łączą się przez ten sam punkt końcowy WS bramy.

Oczekujące żądania są ograniczone do **3 na kanale**; sprawdź `openclaw parowanie listy <channel>`, jeśli kod nie dotarł.

1. Użyj polecenia resetującego:
2. **Użyj aplikacji macOS w trybie zdalnym** (celem SH może być nazwa hostnetu).
   Aplikacja będzie tunelować port bramy i połączyć się jako węzeł.
3. Jeśli nie masz kopii zapasowej, uruchom ponownie `openclaw doctor` i skonfiguruj kanale/modele.

   ```bash
   openclaw nodes pending
   openclaw nodes approve <requestId>
   ```

Czy muszę zrestartować po zmianie konfiguracji

status openclaw
openclaw models status
openclaw logi --follow
----------------------

### **Brama (centralna):** jest właścicielem kanałów (Signal/WhatsApp), routingu i sesji.

OpenClaw odczytuje zmienne środowiskowe z procesu nadrzędnego (powłoka, launchd/systemd, CI itd.) i dodatkowo obciążenia:

- Kreator wyraźnie obsługuje Antropowy token konfiguracyjny i OpenAI Codex OAuth i może przechowywać dla Ciebie klucze API.
- Ile miejsc roboczych i agentów mogę utworzyć

**Dostawcy** (WhatsApp, Telegram, Discord, Najlepsze (plugin), Signal, iMessage)

Czym jest OpenClaw?

```json5
Ostrzega również, czy skonfigurowany model jest nieznany lub brakuje autoryzacji.
```

Nieznani nadawcy otrzymują kod parowania; bot nie przetwarza wiadomości.

### Szybka konfiguracja (zalecane):

Zobacz [/channels/telegram](/channels/telegram#access-control-dms--groups).

1. Dziś wspierane wzorce są następujące:
2. **Zalecane:** 2GB RAM lub więcej jeśli używasz wielu kanałów, automatyzacji przeglądarki lub narzędzi multimedialnych.

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

To uruchamia twoje pociski logowania i importuje tylko brakujące oczekiwane klucze (nigdy nie nadpisuj). Równoważniki Env var:
`OPENCLAW_LOAD_SHELL_ENV=1`, `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`.

### Użyj adresu URL WS: `ws://<host>:18789` (lub `wss://...` jeśli HTTPS).

`openclaw model` zgłasza czy **shell env import** jest włączony. "Shell env: off"
**nie** oznacza, że brakuje twoich var env - po prostu oznacza, że OpenClaw nie załaduje
twoich pocisków logowania automatycznie.

Jeśli brama działa jako usługa (uruchomiona/systemowa), nie odziedziczy środowiska
twojej powłoki. Napraw wykonując jedną z nich:

1. Telegram setMyCommands nie powiódł się z błędami sieciowymi Co powinienem sprawdzić

   ```
   COPILOT_GITHUB_TOKEN=...
   ```

2. Właśnie stworzyłem to, co powinienem zrobić najpierw

3. Co to jest OpenClaw w jednym ustępie

**Węzły (urządzenia):** Macs/iOS/Android łączy się jako urządzenia peryferyjne i ujawnia lokalne narzędzia (`system.run`, `canvas`, `camera`).

```bash
openclaw models status
```

Kopiowanie tokenów jest odczytywane z `COPILOT_GITHUB_TOKEN` (także `GH_TOKEN` / `GITHUB_TOKEN`).
Zobacz [/concepts/model-providers](/concepts/model-providers) i [/environment](/help/environment).

## Autoryzacja modelu nie załadowana do **hosta wejściowego** (sprawdź `status modelów`).

### Dokumenty: [Linux](/platforms/linux), [VPS hosting](/vps).

Wyślij `/new` lub `/reset` jako samodzielną wiadomość. Zobacz [Zarządzanie sesją](/concepts/session).

### Dokumenty: [zdalny dostęp](/gateway/remote), [Agent CLI](/cli/agent), [Agent send](/tools/agent-send).

Tak. Sesje wygasają po `session.idleMinutes` (domyślnie **60**). **Następna** wiadomość
rozpoczyna nowe ID sesji dla tego klucza czatu. To nie usuwa transkryptów\* po prostu rozpoczyna nową sesję.

```json5
Jak dodać modele od innych dostawców, takich jak OpenRouter lub ZAI
```

### `Probe target:` (adres URL rzeczywiście użytej sondy)

Tak, przez **przekierowywanie wielu agentów** i **subagentów**. Możesz utworzyć jednego koordynatora
agenta i kilku pracowników z ich własnymi projektami i modelami.

To powiedziawszy, najlepiej postrzegać to jako **zabawny eksperyment**. To token ciężki i często
mniej wydajny niż jeden bot z osobnymi sesjami. Typowym modelem, który
jest jeden bot, z którym rozmawiasz, z różnymi sesjami dla pracy równoległej. Bot
może również pojawić podagentów w razie potrzeby.

Czy przeglądarka OpenClaw może działać bez słuchawek

### Możesz wyświetlić dostępne modele z `/model`, `/model`, lub `/model`.

Kontekst sesji jest ograniczony przez okno modelu. Długie czaty, duże wyjścia narzędzi lub wiele plików
może wywołać zagęszczenie lub obcięcie.

Pozwól na wersję roboczą i **zatwierdzić przed wysłaniem**.

- Opcja 2 (jeżeli jest już skonfigurowana/dozwolona): grupy list z konfiguracji:
- Zatwierdź węzeł na bramie:
- Istnieją **dwa tryby instalacji systemu Windows**:
- globalny fallback `.env` z `~/.openclaw/.env` (czyli `$OPENCLAW_STATE_DIR/.env`)
- **TUI:** podłącz się do bramki i przełączników/sesji.

### **Ops overhead:** dla każdego agenta profil auth profile, obszar roboczy i routing kanałów.

**Upewnij się, że VPS + Mac są na tej samej sieci ogonowej**.

```bash
openclaw reset
```

Zobacz [Models](/concepts/models) i [komendy Slash ](/tools/slash-commands).

```bash
Z.AI (modele GLM):
```

**Model/auth setup** (Antropowy **setup-token** zalecany dla subskrypcji Claude'a, obsługiwany OpenAI Codex OAuth, klucze API opcjonalne, lokalne modele LM Studio)

```bash
openclaw onboard --install-daemon
```

**OAuth** często wykorzystuje dostęp do subskrypcji (w stosownych przypadkach).

- Kreator wdrożenia oferuje również **Reset**, jeśli widzi istniejącą konfigurację. Zobacz [Wizard](/start/wizard).
- Gdzie są przewodniki instalacji cloudVPS
- Masz trzy wspierane wzory:

### WebChat/Dashboard jest otwarty bez prawego tokenu.

**Zadania Crona** dla zaplanowanych lub powtarzających się zadań (utrzymują się podczas restartów).

- MacBook Pro uruchamia aplikację macOS lub host węzła i pary do bramy.

  ```
  Brama ogląda konfigurację i obsługuje przeładowanie na gorąco:
  ```

  Najpierw potwierdź, że brama jest dostępna i agent może uruchomić:

- Użyłeś złego portu lub ścieżki.

  ```
  /new
  /reset
  ```

**Napraw listę kontrolną dla braku danych logowania dla profilu antropicznego**

- Następnie dzienniki ogonów:
- **Cons:** uśpienia/uśpienie sieci = odłączanie, przerywanie aktualizacji systemu operacyjnego/restartów, musi pozostać wybudzone.

Proxy lub tunel usunął nagłówki uwierzytelniania lub wysłał żądanie nie-bramy.

### Dokumenty: [Pierwsze kroki](/start/getting-started), [Updating](/install/updating).

To jest błąd walidacji dostawcy: model wysłał blok `tool_use` bez wymaganego
`input`. Zazwyczaj oznacza to, że historia sesji jest przestarzała lub uszkodzona (często po długich wątkach
lub zmianie narzędzia/schematu).

piaskownica i ścisłe listy dozwolonych narzędzi

### Jeśli brakuje pliku, używa on bezpiecznych ustawień domyślnych (w tym domyślnego obszaru roboczego `~/.openclaw/workspace`).

Serce działają domyślnie co **30 m**. Dostosuj lub wyłącz je:

```json5
W trybie zdalnym, profile uwierzytelniające na żywo na komputerze bramy, a nie na laptopie.
```

Jeśli `HEARTBEAT.md` istnieje, ale jest w praktyce puste (tylko puste linie i nagłówki
Markdown, takie jak `# Heading`), OpenClaw pomija uruchomienie heartbeat, aby oszczędzać wywołania API.
Jeśli plik nie istnieje, heartbeat nadal się uruchamia, a model decyduje, co zrobić.

Nadpisywanie per-agenta używa `agents.list[].uderzenie serca`. Dokumenty: [Heartbeat](/gateway/heartbeat).

### **Orkiestracja narzędzi** (browser, pliki, planowanie, haczyki)

Nie. OpenClaw działa na **Twoim koncie**, więc jeśli jesteś w grupie, OpenClaw może to zobaczyć.
Domyślnie odpowiedzi grupowe są zablokowane, dopóki nie zezwolisz nadawcom (`groupPolicy: "allowlist"`).

Dokumenty: [protokół bramy](/gateway/protocol), [Discovery](/gateway/discovery), [tryb zdalny macOS](/platforms/mac/remote).

```json5
Homebrew pracuje na Linux
```

### **Twoje urządzenia, Twoje dane:** uruchom bramę gdziekolwiek chcesz, (Mac, Linux, VPS) i utrzymuj obszar roboczy\* historię sesji lokalnie.

**Użyj nazwy hosta sieci ogonowej**

```bash
**Podagenty:** spawn w tle działa od głównego agenta, gdy chcesz równolegle.
```

Aplikacja macOS ogląda plik konfiguracyjny i przełącza tryby na żywo po zmianie tych wartości.

**Klucze API** używaj płatności za token.

```bash
Możesz dodać opcje takie jak `debounce:2s cap:25 drop:summarize` dla trybów monitorowania.
```

Użyj wbudowanego instalatora, a następnie załaduj rozpakowane rozszerzenie w Chrome:

### Dokumenty: [Ollama](/providers/ollama), [modele lokalne](/gateway/local-models),&#xA;[Dostawcy modelów](/concepts/model-providers), [Security](/gateway/security),&#xA;[Sandboxing](/gateway/sandboxing).

Lub dodaj go do bloku `env` konfiguracji (dotyczy tylko jeśli brakuje).

- Bramowanie wzmiankowe jest włączone (domyślnie). Musisz @wspomnieć bota (lub dopasować `wzmiankiPatterns`).
- Co pomoże:

Jak ważne jest uruchomienie OpenClaw na dedykowanym urządzeniu

### Więcej opcji: [flagi instalatora](/install/installer).

Bezpośrednie czaty upadają domyślnie do sesji głównej. Grupy/kanały mają własne klucze sesji, a tematy Telegram / wątki Discorda są oddzielnymi sesjami. Zobacz [Groups](/channels/groups) i [Wiadomości grupowe](/channels/group-messages).

### **Przychodzący SSH nie jest wymagany.** Węzły podłącz się do Gateway WebSocket i użyj parowania urządzenia.

Brak twardych limitów. Dziesiątki (nawet setki) jest w porządku, ale uważaj:

- Jak przełączyć modele bez usuwania mojej konfiguracji
- Jeśli autoryzacja jest włączona, dołącz token lub hasło do ramki `connect`.
- Można rozmawiać ze sobą o dwóch instancjach OpenClaw

Dotychczasowy import OAuth (skopiowany do profili autoryzacji przy pierwszym użyciu)

- Co używa OpenClaw, Flawe i Krill do modelowania
- zestaw autoryzacji modeli openclaw --provider antropic --agent główny antropiczny:default
- Uruchom bramę na ciągłym serwerze (VPS/serwer domowy).

### Twój globalny katalog npm nie jest na PATH.

Tak. Użyj **dzwonka multiagenta** do uruchamiania wielu izolowanych agentów i kierowania przychodzących wiadomości przez
kanału/konto/peer. Slack jest obsługiwany jako kanał i może być związany z określonymi czynnikami.

Dostęp do przeglądarki jest potężny, ale nie "zrób cokolwiek ludzkiego" - antybot, CAPTCHA i MFA mogą
nadal blokować automatyzację. Dla najbardziej niezawodnej kontroli przeglądarki, użyj przekaźnika rozszerzeń Chrome
na komputerze, który obsługuje przeglądarkę (i utrzymuje bramę gdziekolwiek).

Umieść brakujące klucze w `~/.openclaw/.env` więc są one pobierane nawet wtedy, gdy usługa nie odziedziczy twojego pocisku.

- **Żądanie LLM odrzuciło wiadomość myślenia o podpisie wymaganą do google antigrawitacji**
- Jak zainstalować OpenClaw na Linux
- Kanały luźne związane z tymi czynnikami.
- lista grup katalogów openclaw --channel whatsapp

Jak włączyć modele na locie bez ponownego uruchamiania

## **Zainstaluj + zaloguj się na VPS**

### **Przypomnienia i obserwacje:** cron lub bicie serca impulsów i listy kontrolne.

**Trwała pamięć + obszar roboczy** w trakcie sesji

```
agents.defaults.model.primary
```

Modele są określane jako `provider/model` (przykład: `anthropic/claude-opus-4-6`). Jeśli opuścisz dostawcę, OpenClaw obecnie przyjmuje `anthropic` jako tymczasowe rozwiązanie deprekacji - ale nadal powinieneś \*\*jednoznacznie \*\* ustawić `provider/model`.

### `gateway.remote.token` jest tylko dla **zdalnych połączeń CLI**; nie włącza lokalnego uwierzytelniania bramy.

**Zalecane domyślne:** `anthropic/claude-opus-4-6`.
**Dobra alternatywa:** `anthropic/claude-sonnet-4-5`.
**Wiarygodny (mniej znaków):** `openai/gpt-5.2` - prawie tak dobry jak Opus, po prostu mniej osobowości.
**Budget:** `zai/glm-4.7`.

Jak zakończyć anulowanie uruchomionego zadania

Reguła kciuka: użyj **najlepszego modelu, który możesz zapłacić** dla pracy na wysokim poziomie i tańszego modelu
dla rutynowego czatu lub podsumowań. Możesz przekierować modele na konsultanta i używać podagentów do
równolegle do długich zadań (każdy subagent zużywa tokeny). Zobacz [Models](/concepts/models) i
[Sub-agents](/tools/subagents).

Silne ostrzeżenie: słabsze/zbyt ilościowe modele są bardziej podatne na szybkie wstrzyknięcie
i niebezpieczne zachowanie. Zobacz [Security](/gateway/security).

Czy wszystkie dane są używane w OpenClaw zapisywane lokalnie

### Dokumenty: [Security](/gateway/security), [Pairing](/channels/pairing).

Tak. Jeśli twój lokalny serwer wyświetla API kompatybilne z OpenAI, możesz wyznaczyć na niego niestandardowego dostawcę
. Ollama jest obsługiwana bezpośrednio i jest najprostszą ścieżką.

Uwaga dotycząca bezpieczeństwa: mniejsze lub bardzo ilościowe modele są bardziej podatne na szybkie wstrzyknięcie
. Zdecydowanie zalecamy **duże modele** dla każdego bota, który może używać narzędzi.
Jeśli nadal chcesz małe modele, włącz piaskownice i ścisłe listy dozwolonych narzędzi.

Otwórz PowerShell i uruchom:

### Dokumenty: [Docker](/install/docker), [Browser](/tools/browser).

Użyj **poleceń modelu** lub edytuj tylko pola **model**. Unikaj pełnych zamienników konfiguracji.

Zachowaj jeden **aktywny** obszar roboczy na przedstawiciela (`agents.defaults.workspace`).

- Czy mogę używać GPT 5.2 do codziennych zadań i Kodeksu 5.3 do kodowania
- Trasa według agenta lub użyj `/agent` do przełączania
- Potwierdź swoje listy uprawnień (DM lub grupa) dołącz do swojego konta.
- Tak, ale musisz odizolować:

Unikaj `config.apply` z obiektem częściowym, chyba że zamierzasz zastąpić całą konfigurację.
Jeśli nadpisałeś konfigurację, przywróć z kopii zapasowej lub ponownie uruchom `openclaw doctor` aby naprawić.

Jakie są pięć najlepszych przypadków codziennego stosowania OpenClaw

### Upewnij się, że `<prefix>\\bin` jest na PATH (w większości systemów, to `%AppData%\\npm`).

- Dlaczego widzę wymagane pole polecenia LLM odrzucające wiadomościXtooluseinput
- **Krill:** MiniMax M2.1 (`minimax/MiniMax-M2.1`) - see [MiniMax](/providers/minimax).

### `gateway.port` (unikalne porty)

Konfiguracja wielu agentów oznacza, że może być wiele plików `auth-profiles.json`.

```
/model sonnet
/model haiku
/model opus
/model gpt
/model gpt-mini
/model gemini
/model gemini-flash
```

Pamięć OpenClaw to tylko pliki Markdown w obszarze roboczym agenta:

`/model` (i `/model list`) pokazuje kompaktowy, numerowany selektor. Wybierz według liczby:

```
Zatwierdź według: `openclaw pairing zatwierdza <channel> <code>`
```

**OS:** Ubuntu LTS lub innego nowoczesnego Debian/Ubuntu.

```
/model opus@anthropic:default
/model opus@anthropic:work
```

Wskazówka: `/model status` pokazuje który agent jest aktywny, który plik `auth-profiles.json` jest używany i który profil autoryzacji zostanie wypróbowany.
Pokazuje również skonfigurowany punkt końcowy dostawcy (`baseUrl`) i tryb API (`api`), gdy jest dostępny.

Sprawdź, czy brama jest uruchomiona 24/7 (bez uśpienia/restartów).

Jak używać Brave do kontroli przeglądarki

```
To są przerwane wyzwalacze (nie ukośnik poleceń).
```

Jeśli chcesz powrócić do domyślnego, wybierz go z `/model` (lub wyślij `/model <default provider/model>`).
Użyj `/model`, aby potwierdzić, który profil autoryzacji jest aktywny.

### Pełne przechodzenie: [Pierwsze kroki](/start/getting-started).

Tak. Ustaw jeden jako domyślny i przełącz w razie potrzeby:

- Jak zainstalować rozszerzenie Chrome do przejęcia przeglądarki
- Jak zatrzymać wyświetlanie wewnętrznych komunikatów systemowych na czacie
- Jak całkowicie zresetować OpenClaw ale zachować zainstalowane

Dlaczego status bramki openclaw mówi, że Runtime działa, ale sonda RPC nie powiodła się

### Szczegóły protokołu: [Gateway protocol](/gateway/protocol).

Jeśli `agents.defaults.models` jest ustawiony, staje się **dopuszczalną listą** dla `/model` i każdego nadpisania sesji
. Wybór modelu, który nie znajduje się w tej liście zwrotów:

```
Model "provider/model" is not allowed. Use /model to list available models.
```

Ten błąd został zwrócony \*\*zamiast \*\* zwykłą odpowiedź. Naprawda: dodaj model do
`agents.defaults.models`, usuń dozwoloną listę lub wybierz model z `/model`.

### Jeśli chcesz, aby interfejs użytkownika bez SSH, użyj skali Ogonowej na VPS:

Oznacza to, że **dostawca nie jest skonfigurowany** (nie znaleziono profilu MiniMax dostawcy lub autoryzacja
), więc model nie może zostać rozwiązany. Naprawiona dla tej detekcji jest
w **2026.1.12** (niezwolniona w momencie zapisu).

\*\*Brama zawsze włączona \*\* (uruchom na VPS, interakcja z dowolnego miejsca)

1. Czy istnieje sposób API RPC do zastosowania konfiguracji
2. `steer` - nowe wiadomości przekierowują bieżące zadanie
3. status kanałów openclaw
   otwiera dzienniki kanałów --channel telegram
4. Run:

   ```bash
   openclaw models list
   ```

   Czy wspierasz subskrypcję OpenAI auth Codex OAuth

`followup` - uruchom wiadomości jednorazowo

### **Zatwierdź węzeł** na bramce:

Tak. Użyj **MiniMax jako domyślnego** i w razie potrzeby przełącz modele **na sesję**.
Fallbacks dotyczy **błędów**, nie "twardych zadań", więc użyj `/model` lub osobnego agenta.

Zainstaluj przeglądarki Playwright za pomocą pakietu CLI:
`node /app/node_modules/playwright-core/cli.js install chromium`

```json5
Lokalna przeglądarka za pośrednictwem przekaźnika rozszerzeń (lub węzła) w razie potrzeby.
```

Czy mogę używać tańszych modeli do osobistych zadań asystentów

```
/model gpt
```

DM twojego bota, a następnie uruchom `openclaw logs --follow` i przeczytaj `from.id`.

- Zainstaluj **Git dla Windows** i upewnij się, że `git` jest na Twoim PATH.
- **Potwierdź, gdzie żyją profile uwierzytelniania** (nowe vs starsze ścieżki)
- **Bezwzględne minimum:** 1 vCPU, 1 GB RAM.

Następnie uruchom ponownie pokład:

### Dokumenty: [Anthropic](/providers/anthropic), [OpenAI](/providers/openai),&#xA;[modele lokalne](/gateway/local-models), [Models](/concepts/models).

Tak. OpenClaw statków kilka domyślnych skrótów (stosowanych tylko wtedy, gdy model istnieje w `agents.defaults.models`):

- `opus` → `anthropic/claude-opus-4-6`
- `sonnet` → `anthropic/claude-sonnet-4-5`
- `gpt` → `openai/gpt-5.2`
- `gpt-mini` → `openai/gpt-5-mini`
- `gemini` → `google/gemini-3-pro-preview`
- `gemini-flash` → `google/gemini-3-flash-preview`

Te pliki są dostępne w **obszarze roboczym agenta**, a nie `~/.openclaw`.

### Możesz sparować **węzły** (Mac/iOS/Android/headless) z tą bramą chmury aby uzyskać dostęp do&#xA;lokalnego ekranu/kamery/płótna lub uruchomić polecenia na swoim laptopie, trzymając bramę&#xA;w chmurze.

Aliasy pochodzą z `agents.defaults.models.<modelId>.alias`. Przykład:

```json5
Uruchom bramę na Linux/VPS i uruchom serwer BlueBubbles na dowolnym Mac podpisanym w wiadomości.
```

status bramki openclaw
zrestartuj bramę otwierania

### akcja procesu:zabicie sessionId:XXX

**Współdzielony laptop/desktop:** całkowicie dobrze, jeśli chodzi o testowanie i aktywne użytkowanie, ale oczekujemy pauzy, gdy maszyna uśpi lub aktualizuje.

```json5
Jak zainstalować umiejętności na Linux
```

Użyj `openclaw nodes status` / `openclaw nodes list` aby to zobaczyć.

```json5
status bramki openclaw
zrestartuj bramę otwierania
```

Jeśli odwołujesz się do dostawcy/modelu, ale brakuje wymaganego klucza dostawcy, otrzymasz błąd autoryzacji runtime (e. . `Nie znaleziono klucza API dla dostawcy "zai"`).

Brama na Mac mini (zawsze włączona).

Zazwyczaj oznacza to, że **nowy agent** ma pusty sklep autoryzacyjny. Auth jest dla agenta i
przechowywany w:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

**Nie znaleziono klucza API dla dostawcy po dodaniu nowego agenta**

- Wskazówki dotyczące kopii zapasowej: zobacz [Kopia zapasowa](/help/faq#whats-the-recommended-backup-strategy).
- \*\*Laptop (lokalna brama) \*\*

Jaki jest format konfiguracji, Gdzie to jest

## Uruchom to z tego samego `--profile` / środowiska, którego usługa ma używać.

### Możesz również ustawić nadpisanie zamówienia **per-agent** (przechowywane w `auth-profiles.json`) tego agenta za pośrednictwem CLI:

Użyj `openclaw models status` aby zobaczyć skonfigurowane modele i czy dostawcy są uwierzytelniani.

1. Użyj pomocników bramy:
2. Jaki port używa brama

**Obszar roboczy** lokalizacja + pliki bootstrap

### Możesz również wymusić określony profil autoryzacji dla dostawcy (na sesję):

```
Użyj `/compact` przed długimi zadaniami i `/new` podczas przełączania tematów.
```

**Włącz MagicDNS (zalecane)**

### Dokumenty: [TUI](/web/tui), [komendy Slash ](/tools/slash-commands).

- Otwórz PowerShell, wpisz WSL, a następnie zrestartuj:
  - Potrzebuję Mac mini dla wsparcia iMessage
  - Ustawiam gatewaybind lan lub tailnet i teraz nic nie słucha interfejsu użytkownika mówi o nieautoryzowanych
- Agent A domyślnie: MiniMax
  - Jeśli ustawisz `ANTHROPIC_API_KEY` w swojej powłoki, ale uruchom bramę za pomocą systemu/uruchamiania, może nie dziedziczyć. Umieść go w `~/.openclaw/.env` lub włącz `env.shellEnv`.
- Brama jest uruchomiona: `openclaw gateway status`
  - Potrzebuję subskrypcji Claude lub OpenAI aby uruchomić to
- Użyj `openclaw configure` do interaktywnych edycji.
  - Mogę załadować umiejętności z niestandardowego folderu

Jeśli uruchomisz go ręcznie (bez usługi), użyj:

`collect` - wiadomości zbiorcze i odpowiedź raz (domyślnie)

- Wskazówka: dodaj strażnika, aby dwa boty nie pętlały bez końca (tylko do kanału
  dopuszczają listy lub reguła "nie odpowiadaj na wiadomości bota").
  - Dzienne notatki w `memory/YYYY-MM-DD.md`
  - Główna konfiguracja (JSON5)

- MiniMax M2.1 ma własne dokumenty: [MiniMax](/providers/minimax) i
  [modele lokalne](/gateway/local-models).
  - Czy mogę uruchomić wiele bramek na tym samym hoście
  - **Cons:** często uruchamiaj bezgłowne (używaj zrzutów ekranu), tylko zdalny dostęp do plików, musisz mieć SSH dla aktualizacji.

    ```bash
    Dokumenty: [Config](/cli/config), [Configure](/cli/configure), [Doctor](/gateway/doctor).
    ```

- /model antropikalny/claudeopus-4-6
  - **Kompaktowa** (zachowuje konwersację, ale podsumowuje starsze tury):

### **Przełącznik na żądanie**: użyj `/model` aby zmienić obecny model sesji w dowolnym momencie.

Jeśli konfiguracja Twojego modelu zawiera Google Gemini jako rezerwę (lub przełączyłeś się na skrót Geminiego), OpenClaw spróbuje go podczas awaryjnego modelu. Jeśli nie skonfigurowałeś danych logowania Google, zobaczysz \\`Nie znaleziono klucza API dla dostawcy "google".

Jeśli kupię Mac mini do uruchomienia OpenClaw mogę podłączyć go do mojego MacBook Pro

Jeśli zdalny tunel, najpierw `ssh -N -L 18789:127.0.0.1:18789 user@host` i otwórz `http://127.0.0.1:18789/`.

Powód: historia sesji zawiera **bloki myślenia bez podpisów** (często z
przerwanego/częściowego strumienia). Google Antigravity wymaga podpisów do myślenia bloków.

Naprawda: OpenClaw oddziela niepodpisane bloki myślenia dla klauzuli antygrawitacji Google. Jeśli nadal się pojawi, rozpocznij **nową sesję** lub ustaw `/thinking off` dla tego agenta.

Jeśli bot "zapomnisz" po ponownym uruchomieniu, potwierdź, że brama używa tego samego obszaru roboczego
przy każdym uruchomieniu (i pamiętaj: tryb zdalny używa projektu **hostów bramy**
, nie twój lokalny laptop).
-------------------------------------------

Powinienem uruchomić bramę na moim laptopie lub VPS

### Szybka ścieżka Linux + instalacja usług: [Linux](/platforms/linux).

Profil autoryzacji to nazwany rekord (klucz OAuth lub API) powiązany z dostawcą. Profile na żywo w:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

### Zresetowanie deweloperów: `openclaw gateway --dev --reset` (dev-only; wyskakuje dev config + poświadczenia + sesje + obszar roboczy).

**1) błąd spawn git / git nie znaleziony**

- Czy mogę prosić OpenClaw o aktualizację
- Może OpenClaw uruchamiać zadania na harmonogramie lub ciągle w tle
- Jak skonfigurować Skalę Ogonową na VPS i połączyć się z Mac

### Dokumenty: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat),&#xA;[Heartbeat](/gateway/heartbeat).

Tak. Konfiguracja obsługuje opcjonalne metadane dla profili i zamawiania według dostawcy (`auth.order.<provider>`). To **nie** przechowuje tajemnice; mapuje identyfikatory do dostawcy/trybu i ustawia kolejność obrotu.

OpenClaw może tymczasowo pominąć profil, jeśli jest w krótkim **czasie odnowienia** (limity/czasy/błędy autoryzacji) lub w dłuższym **wyłączonym** stanie (rozliczenie/niewystarczające kredyty). Aby to sprawdzić, uruchom `openclaw models status --json` i sprawdź `auth.unusableProfiles`. Tuning: `auth.cooldowns.billingBackoffHours*`.

Dlaczego nie odpowiada OpenClaw w grupie

```bash
Dokumenty: [Dashboard](/web/dashboard), [dostęp zdalny](/gateway/remote), [Troubleshooting](/gateway/troubleshooting).
```

status openclaw
openclaw models status
openclaw channels status
openclaw logs --follow

```bash
Dokumenty: [Windows (WSL2)](/platforms/windows), [Bateway service runbook](/gateway).
```

### Przegląd poleceń cięcia: zobacz [komendy cięcia](/tools/slash-commands).

Dokumenty: [Gateway](/gateway), [Channels](/channels), [Multi-agent](/concepts/multi-agent),
[Memory](/concepts/memory).

- Zainstaluj ClawHub CLI (wybierz jeden menedżer pakietów):
- Następnie sprawdź autoryzację i przekierowanie:

Przełącz na `gateway.bind: "loopback"` / `"lan"`.

## Umieść opakowanie na pliku `PATH` na serwerze Linux (na przykład `~/bin/memo`).

### Następnie Chrome → `chrome://extensions` → włącz "Tryb programisty" → "Załaduj rozpakowane" → wybierz ten folder.

Jak zdefiniować aliasy skrótów modelu

Czy mogę uruchomić wiele botów lub czatów w tym samym czasie i jak powinienem to ustawić

```
Aby skierować konkretny agent:
```

### **Niestandardowe umiejętności / wtyczka:** najlepsze dla niezawodnego dostępu API (Notion/HeyGen oba mają API).

Ponieważ "running" jest widokiem **supervisor's** (launchd/systemd/schtasks). Sonda RPC jest prawdziwym połączeniem CLI z bramą WebSocket i wywołaniem `status`.

**Zadania Cron**: izolowane zadania mogą ustawić nadpisanie `model` na jedno zadanie.

- Czy mogę uruchomić OpenClaw w VM i jakie są wymagania
- Czy mogę użyć MiniMax jako domyślnego i OpenAI dla skomplikowanych zadań
- Jeśli chcesz natywną integrację, otwórz żądanie funkcji lub zbuduj umiejętność
  ukierunkowaną na te API.

### Dokumenty: [Sub-agents](/tools/subagents).

Jak działa autoryzacja Codex

Czy mogę używać subskrypcji Claude Max bez klucza API

```bash
Dokumenty: [Cron jobs](/automation/cron-jobs), [Cron vs Heartbeat](/automation/cron-vs-heartbeat).
```

**Dostęp wieloplatformowy** (WhatsApp, Telegram, TUI, WebChat)

### Dokumenty: [Agent workspace](/concepts/agent-workspace).

OpenClaw egzekwuje blokadę przez powiązanie nasłuchu WebSocket natychmiast przy starcie (domyślnie `ws://127.0.0.1:18789`). Jeśli powiązanie nie powiedzie się z `EADDRINUSE`, rzuca `GatewayLockError` wskazując, że inna instancja jest już słuchana.

Pamięć podręczna autoryzacji (zarządzana automatycznie)

### wybrane niestandardowe identyfikatory (np. `anthropic:work`)

Domyślny agent B: OpenAI

```json5
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://gateway.tailnet:18789",
      token: "your-token",
      password: "your-password",
    },
  },
}
```

Jeśli token został utworzony na innej maszynie, użyj `openclaw models auth paste-token --provider antropiczny`.

- Profile autoryzacji (OAuth + klucze API)
- Powinienem zainstalować na drugim laptopie lub po prostu dodać węzeł

### Dokumenty: [Models](/concepts/models), [dzwonek wielu agentów](/concepts/multi-agent), [MiniMax](/providers/minimax), [OpenAI](/providers/openai).

Mogę używać modeli samodzielnych llamacpp vLLM Ollama

utrzymywanie `web_search` / `web_fetch` / `browser` dla agentów z dostępem do narzędzi

- Użyj komendy `/model` jako samodzielnej wiadomości:

Dokumenty: [Nodes](/nodes), [Dostęp zdalny](/gateway/remote), [Wysyłanie wielu agentów](/concepts/multi-agent), [Sub-agents](/tools/subagents), [TUI](/web/tui).

- Domyślne zachowanie na kanałach DM-zdolnych to **sparowanie**:
- **Nodes** dla lokalnej przeglądarki/ekranu/kamery/exec
- **Zainstaluj + zaloguj się na Mac**
- Czy mogę uruchomić szybki agent czatu i Opus dla agenta kodującego
- `Ostatni błąd bramy:` (zwykła przyczyna główna, gdy proces jest żywy, ale port nie jest słuchany)
- Wciąż utknąłeś? Uruchom `openclaw status --all` i postępuj zgodnie z [Troubleshooting](/gateway/troubleshooting). Zobacz szczegóły uwierzytelniania [Dashboard](/web/dashboard).

### **Dedykowany host (VPS/Mac mini/Pi):** zawsze, mniej przerw w uśpieniu/restarcie, czystsze uprawnienia, łatwiejsze do dalszego działania.

`tailnet` pobrał adres IP skali ogona z interfejsów sieciowych (100.64.0.0/10). Jeśli maszyna nie jest w skali ogonowej (lub interfejs jest w dół), nie ma nic do wiązania.

Nie otwieraj portu WS w normalnej karcie przeglądarki.

- Zdrowie kanału: `status kanałów openclaw`
- Wyszukiwanie pamięci semantycznej wymaga klucza API OpenAI

Uwaga: `tailnet` jest jasny. `auto` preferuje pętlę zwrotną; użyj `gateway.bind: "tailnet"` gdy chcesz powiązać tylko sieć końcową.

### Czy mogę uruchomić wiele bramek na tym samym hoście

Zazwyczaj nie - jedna brama może uruchomić wiele kanałów i agentów komunikacyjnych. Używaj wielu bramek tylko wtedy, gdy potrzebujesz nadmiarowości (np. bota ratunkowego) lub twardej izolacji.

Plik `@userinfobot` lub `@getidsbot`.

- Czy mogę korzystać z umiejętności Apple macOS tylko z Linux?
- Jaka jest różnica między instalacją hackable git a instalacją npm
- Pokaż: [https://openclaw.ai/showcase](https://openclaw.ai/showcase)
- Można pomóc OpenClaw w kontaktach z potencjalnymi genami i blogami dla SaaS

**Bezpieczniejsze sterowanie wykonaniem.** `system.run` jest bramkowane przez węzły dopuszczalne/zatwierdzenia na tym laptopie.

- Mogę przełączyć się między instalacjami npm i git później
- Jak OpenClaw ładuje zmienne środowiskowe
- Install a per-profile service: `openclaw --profile <name> gateway install`.

Profile również nazwy usług sufiksu (`bot.molt.<profile>`; starsza `com.openclaw.*`, `openclaw-gateway-<profile>.service`, `OpenClaw Bateway (<profile>)`).
Pełny przewodnik: [Multiple gateways](/gateway/multiple-gateways).

### Logi plików (strukturalne):

Brama jest \*\*serwerem WebSocket \*\*, i oczekuje, że pierwsza wiadomość do
będzie ramką `connect`. Jeśli otrzyma cokolwiek innego, zamyka połączenie
**kodem 1008** (naruszenie zasad).

Użyj `openclaw config set` dla małych zmian.

- Bezpieczniej (bez bota podmiotu trzeciego):
- `anthropic:default` (często gdy nie istnieje żadna tożsamość e-mail)
- **2) Natywne Windows (nie zalecane):** Brama działa bezpośrednio w Windows.

Zobacz [OAuth](/concepts/oauth), [Dostawcy modelów](/concepts/model-providers) i [Wizard](/start/wizard).

1. openclaw reset --scope pełne --yes --non-interactive
2. Stan agenta (agentDir + sesje)
3. Przykładowy wzór (biegnij z maszyny, która może dotrzeć do docelowej bramy):

Telegram → Brama → Agent → `node.*` → Węzeł → Brama → Telegram

```
openclaw tui --url ws://<host>:18789 --token <token>
```

Czy istnieje korzyść z używania węzła na moim osobistym laptopie zamiast SSH od VPS

## Ustaw unikalny `gateway.port` w każdej konfiguracji profilu (lub przejdź `--port` do ręcznego uruchomienia).

### Włącz lub dostosuj **sesje pruning** (`agents.defaults.contextPruning`) do przycinania starego wyjścia narzędzia.

{
pl: { MINIMAX_API_KEY: "sk-...", OPENAI_API_KEY: "sk-... },
agents: {
defaults: {
model: { primary: "minimax/MiniMax-M2. " },
modele: {
"minimax/MiniMax-M2. ": { alias: "minimax" },
"openai/gpt-5. ": { alias: "gpt" },
},
},
},
}

```
/tmp/openclaw/openclaw-YYYY-MM-DD.log
```

Możesz ustawić stabilną ścieżkę za pomocą `logging.file`. Poziom dziennika pliku jest kontrolowany przez `logging.level`. Werbosity konsoli jest kontrolowany przez `--verbose` i `logging.consoleLevel`.

`web_fetch` jest włączone domyślnie (chyba że zostanie jawnie wyłączone).

```bash
openclaw logs --follow
```

**Jeśli chcesz zamiast tego użyć klucza API**

- Od git → npm:
- Linux: `journalctl --user -u openclaw-gateway[-<profile>].service -n 200 --no-pager`
- Oficjalny bot API:

OpenClaw używa wcześniejszych identyfikatorów dostawcy, takich jak:

### Dokumentacja: [Nodes](/nodes), [Nodes CLI](/cli/nodes).

Dokumenty: [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes),
[Tryb zdalny Mac](/platforms/mac/remote).

```bash
**Automatyzacja przeglądarki:** działa bez kodu, ale jest wolniejszy i bardziej kruchy.
```

Jeśli uruchomisz bramę ręcznie, `openclaw gateway --force` może odzyskać port. Zobacz [Gateway](/gateway).

### Dokumenty: [Models](/concepts/models), [Configure](/cli/configure), [Config](/cli/config), [Doctor](/gateway/doctor).

Upewnij się, że brama WS jest osiągalna (bind lub tunel SSH).

OAuth vs API key what the difference

Jedna strona pojęcia na klienta (kontekst + preferencje + aktywna praca).

```powershell
Dokumenty: [BlueBubbles](/channels/bluebubbles), [Nodes](/nodes), [Tryb zdalny Mac](/platforms/mac/remote).
```

Dokumenty: [Security](/gateway/security).

```bash
openclaw gateway run
```

curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale

`/model` na czacie (szybka, na sesję)

```powershell
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      modely: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "anthropic/claude-sonnet-4-5": { alias: "sonnet" },
        "anthropic/claude-haiku-4-5": { alias: "haiku" },
      },
    },
  },
}
```

Kanał parowanie / dozwolone blokowanie odpowiedzi (sprawdź konfigurację kanału + log).

```powershell
openclaw gateway run
```

**Zresetuj** (nowy identyfikator sesji dla tego samego klucza czatu):

### **Szybki przełącznik (na sesję):** `/model gpt-5.2` dla zadań dziennych, `/model gpt-5.3-codex` dla kodowania.

Dokumenty: [Nodes](/nodes), [Węzły CLI](/cli/nodes), [Wiele bram](/gateway/multiple-gateways).

```bash
Od npm → git:
```

openclaw cron run <jobId> --force
openclaw cron uruchamia --id <jobId> --limit 50

- ---
  nazwa: opis japple-notes
  : Zarządzaj notatkami Apple poprzez notatkę CLI na macOS.
  metadane: { "openclaw": { "os": ["darwin", "linux"], "Requres": { "bins": ["memo"] } } }
  ---
- Dla procesów w tle (z narzędzia exec), możesz poprosić agenta o uruchomienie:
- **Subagentów:** kodowanie trasy zadań dla podagentów z innym domyślnym modelem.

Przykład (repozytorium jako domyślny cwd):

Czy mój bot ma własne konto lub numer telefonu na GitHub

### {&#xA;agents: {&#xA;domyślnie: {&#xA;model: { primary: "zai/glm-4. " },&#xA;modele: { "zai/glm-4. ": {} },&#xA;},&#xA;},&#xA;env: { ZAI_API_KEY: "..." },&#xA;}

Zazwyczaj oznacza to, że interfejs użytkownika utracił połączenie WebSocket. Sprawdź:

1. Czy brama jest uruchomiona? `openclaw gateway status`
2. Czy brama jest zdrowa? `openclaw status`
3. Czy interfejs ma odpowiedni token? `openclaw dashboard`
4. Utwórz opakowanie SSH dla binarnego (np. `memo` dla Apple Notes):

{
agents: {
domyślnie: {
tools: {
message: {
crossContext: {
allowAcrossProviders: true, Znacznik
{ włączony: true, prefiks: "[z {channel}] },
},
},
},
},
},
}

```bash
openclaw logs --follow
```

zainstaluj klawhub <skill-slug>
aktualizacja klawhub --all

### Szczegóły: [Security](/gateway/security).

Dokumenty: [Nodes](/nodes), [Nodes CLI](/cli/nodes), [Chrome extension](/tools/chrome-extension).

```bash
{
  agents: {
    domyślnie: {
      heartbeat: {
        zawsze: "2h", // lub "0m", aby wyłączyć
      },
    },
  },
}
```

Jeśli jesteś na VPS lub za proxy, potwierdź wychodzący HTTPS, a DNS działa.
Jeśli brama jest zdalna, upewnij się, że patrzysz na logi na Gateway host.

Dlaczego status bramki openclaw pokazuje inną usługę Config cli i Config

### Otwórz aplikację macOS lokalnie i połącz się w trybie **Zdalne przez SSH** (lub bezpośredni net)&#xA;, aby mógł zarejestrować się jako węzeł.

`OPENCLAW_CONFIG_PATH` (konfiguracja dla każdej instancji)

```bash
Trzyliterowy host bramki (VPS/Mac mini).
```

W TUI użyj `/status` aby zobaczyć bieżący stan. Jeśli oczekujesz odpowiedzi na czacie
, upewnij się, że dostawa jest włączona (`/delivery on`).

dzienniki openclaw --follow --json

### Stan dostawcy (np. `whatsapp/<accountId>/creds.json`)

Dokumenty: [Tailscale](/gateway/tailscale), [dostęp zdalny](/gateway/remote), [Channels](/channels).

```bash
Możesz również zdefiniować var env in line w konfiguracji (stosowane tylko wtedy, gdy brakuje go w env):
```

To zatrzymuje/uruchamia **nadzorowaną usługę** (uruchom na macOS, systemd na Linux).
Użyj tego, gdy brama działa w tle jako demon.

Dokumenty: [Channels](/channels), [Troubleshooting](/gateway/troubleshooting), [dostęp zdalny](/gateway/remote).

```bash
openclaw gateway run
```

Docs: [Gateway service runbook](/gateway).

### Większość poleceń musi być wysyłana jako **niezależna** wiadomość, która zaczyna się od `/`, ale kilka skrótów (takich jak `/status`) działa również w linii dla dozwolonych nadawców.

- Jak powiązać folder hosta z sandboxem
- `anthropic:<email>` dla tożsamości OAuth

Jeśli zainstalowałeś usługę, użyj poleceń bramy. Użyj `openclaw gateway` gdy
chcesz uruchomić jednorazowo, na pierwszym planie.

### Twoja brama jest uruchomiona z włączoną autoryzacją (`gateway.auth.*`), ale interfejs użytkownika nie wysyła pasującego tokenu/hasła.

Uruchom bramę z `--verbose` aby uzyskać więcej szczegółów konsoli. Następnie sprawdź plik dziennika w celu uzyskania autoryzacji kanału, routingu modeli i błędów RPC.

## Ustaw `gateway.auth.token` (lub `OPENCLAW_GATEWAY_TOKEN`) na hostze bramy.

### **1) WSL2 (zalecane):** Brama działa wewnątrz Linux.

Załączniki wychodzące od agenta muszą zawierać wiersz `MEDIA:<path-or-url>` (na jego własnym wierszu). Zobacz [OpenClaw assistant setup](/start/openclaw) i [Agent send](/tools/agent-send).

**Otwarte źródło i hackable:** sprawdzaj, rozszerzaj i samemu hostowi bez blokady dostawcy.

```bash
`agents.defaults.workspace` (izolacja obszaru roboczego)
```

# Domyślnie skonfigurowany domyślny agent (omit --agent)openclaw model auth order get --provider antropic# Zablokuj rotację do pojedynczego profilu (wypróbuj tylko tę opcję)openclaw model auth order --provider antropic anthropic:default# Lub ustaw wyraźne zamówienie (przejście wewnątrz dostawcy)openclaw model auth order set --provider antropic anthropic:work antropic anthropic:default# Wyczyść nadpisanie (powrót do autoryzacji konfiguracji). rder / okrągły robina)Kolejność autoryzacji modeli openclaw jest jasna --provider antropic

- Wszystko żyje pod `$OPENCLAW_STATE_DIR` (domyślnie: `~/.openclaw`):
- Brama: porty, "już uruchomione" i tryb zdalny

Uruchom `claude setup-token`, a następnie wklej go za pomocą `openclaw models auth setup-token --provider anthropic`.

## Naprawda: rozpocznij nową sesję z `/new` (niezależna wiadomość).

### Uruchom ponownie `/model` **bez** sufiks `@profile`:

Traktuj przychodzące maszyny DM jako niezaufane dane wejściowe. Wartości domyślne mają na celu zmniejszenie ryzyka:

- Przywróć z kopii zapasowej (git lub skopiowany `~/.openclaw/openclaw.json`).
  - Dokumenty: [Multi-agent routing](/concepts/multi-agent), [Sub-agents](/tools/subagents), [Agent CLI](/cli/agents).
  - Rozpocznij skalę ognia na tym serwerze (aby miał adres 100.x), lub
  - `gateway.reload.mode: "hybrid"` (domyślnie): Zastosuj bezpieczne zmiany, zrestartuj dla krytycznych
- Ustawiam gatewaybind tailnet, ale nie pasuje do nasłuchiwania

Sygnatura konfiguracji klucza: [Konfiguracja bramy](/gateway/configuration#agentsdefaultssandbox)

### Jest szybkim wstrzyknięciem tylko dla robotów publicznych

Nie. Szybkie wstrzyknięcie dotyczy **niezaufanej treści**, a nie tylko kto może DM bota.
Jeśli twój asystent czyta zewnętrzną zawartość (wyszukiwanie/pobieranie, strony przeglądarki, e-maile,
docs, załączniki, wklejone wpisy), ta zawartość może zawierać instrukcje, które próbują
przechwycić model. Może się to zdarzyć, nawet jeśli **jesteś jedynym nadawcą**.

Największe ryzyko polega na włączeniu narzędzi: model może być wytrząsany do
exfiltracji kontekstu lub wywoływania narzędzi w Twoim imieniu. Zmniejsz promień rażenia, stosując:

- **Domyślnie lokalne:** sesje, pliki pamięci, konfiguracje i obszar roboczy na żywo hosta bramy
  (`~/.openclaw` + Twój katalog roboczy).
- `openclaw models set ...` (aktualizuje tylko konfigurację modelu)
- Dokumenty: [WhatsApp](/channels/whatsapp), [Directory](/cli/directory), [Logs](/cli/logs).

Polecenia czatu, przerywanie zadań i "nie zatrzyma się"

### `openclaw Bateway restart`: zrestartuje **usługę w tle** (uruchomiony/system).

Tak, dla większości konfiguracji. Izolacja bota z osobnymi kontami i numerami telefonów
zmniejsza promień wybuchu, jeśli coś pójdzie nie tak. Ułatwia to również rotację poświadczeń
lub cofnięcie dostępu bez wpływu na Twoje konta osobiste.

Rozpocznij mało. Daj dostęp tylko do narzędzi i kont, których naprawdę potrzebujesz, i rozszerzaj
w razie potrzeby.

Domyślny model OpenClawa to cokolwiek ustawiłeś:

### Brama WS: `ws://your-vps.tailnet-xxxx.ts.net:18789`

**Nie** zalecamy pełną autonomię w odniesieniu do Twoich osobistych wiadomości. Najbezpieczniejszy wzór:

- Najszybciej: `openclaw dashboard` (drukuje + kopiuje adres URL pulpitu menedżerskiego, próbuje otworzyć; pokazuje podpowiedź SSH, jeśli bez głowy).
- OpenClaw obsługuje oba:
- Windows: `schtasks /Query /TN "OpenClaw Bateway (<profile>)" /V /FO LIST`

Jeśli chcesz eksperymentować, zrób to na dedykowanym koncie i zachowaj go w izolacji. Zobacz
[Security](/gateway/security).

### Umieść token w `~/.openclaw/.env`:

Tak, **jeśli** agent jest tylko czatem, a dane wejściowe są zaufane. Smaller tiers are
more susceptible to instruction hijacking, so avoid them for tool-enabled agents
or when reading untrusted content. Jeśli musisz użyć mniejszego modelu, zablokuj narzędzia
i uruchom wewnątrz piaskownicy. Zobacz [Security](/gateway/security).

### Pełna instrukcja (w tym zdalna brama + zabezpieczenia): [Rozszerzenie Chrome'a](/tools/chrome-extension)

Kody parowania są wysyłane **tylko** gdy nieznany nadawca wiadomości bota i
`dmPolicy: "pairing"` jest włączony. `/start` sam w sobie nie generuje kodu.

\#!/usr/bin/env bash
ustaw -euo pipefailed
exec ssh -T user@mac-host /opt/homebrew/bin/memo "$@"

```bash
openclaw pairing list telegram
```

Powszechnym wzorcem jest **jedna brama** (np. Raspberry Pi) plus **węzły** i **agentów**:

### Utrzymaj `/home/node` używając `OPENCLAW_HOME_VOLUME`, aby przetrwać.

Nie. Domyślna polityka aplikacji WhatsApp to **sparowanie**. Nieznani nadawcy otrzymują tylko kod parowujący, a ich wiadomość nie jest **przetworzona**. OpenClaw odpowiada tylko na otrzymywane czaty lub na wyraźne wysłanie wyzwalaczy.

`openclaw gateway` zaczyna się tylko, gdy `gateway.mode` jest `local` (lub przeszedłeś flagę zastąpienia).

```bash
openclaw pairing approve whatsapp <code>
```

[PLACEHOLDER] git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
openclaw lekarz
openclaw gateway restart

```bash
openclaw pairing list whatsapp
```

Zapytanie o numer telefonu kreatora: służy do ustawiania Twojej **listy dozwolonych/właściciela**, aby Twoje własne DM były dozwolone. Nie jest używany do automatycznego wysyłania. Jeśli używasz swojego osobistego numeru WhatsApp, użyj tego numeru i włącz `channels.whatsapp.selfChatMode`.

## **Pyt.: "Jaki jest domyślny model dla Antropii z kluczem API?"**

### **Model-agnostic:** używaj Antropii, OpenAI, MiniMax, OpenRouter, itp., z podziałem na agentów trasowania&#xA;i przegrywania.

OpenRouter (pay-per token; wiele modeli):

Uruchom `openclaw agent add <id>` i skonfiguruj autoryzację podczas kreatora.

```
Wymagany jest pełny restart dla zmian `gateway`, `discovery`, i `canvasHost`.
```

Jeśli nadal jest głośny, sprawdź ustawienia sesji w interfejsie sterowania i ustaw wierzchołek
na **dziedzicza**. Potwierdź, że nie używasz profilu bota z `verboseDefault` ustaw
na `on` w konfiguracji.

npm install -g openclaw@latest
openclaw lekarz
uruchom ponownie bramę openclaw

### {&#xA;agents: {&#xA;defaults: {&#xA;model: { primary: "openrouter/anthropic/claude-sonnet-4-5" },&#xA;modely: { "openrouter/anthropic/claude-sonnet-4-5": {} },&#xA;},&#xA;},&#xA;pl: { OPENROUTER_API_KEY: "sk-or-. ." },&#xA;}

Aktualnie: `~/.openclaw/agents/<agentId>/agent/auth-profiles.json`

```
stop
abort
esc
wait
exit
interrupt
```

\--port > OPENCLAW_GATEWAY_PORT > gateway.port > domyślnie 18789

Skonfigurowałeś `channels.whatsapp.groups` bez `"*"` i grupa nie jest dozwolona.

```
Dokumenty: [Multi-Agent Routing](/concepts/multi-agent), [Slack](/channels/slack),
[Browser](/tools/browser), [Chrome extension](/tools/chrome-extension), [Nodes](/nodes).
```

Logowanie: `openclaw models auth login --provider google-gemini-cli --set-default`

Potwierdź włączenie cron (`cron.enabled`) i `OPENCLAW_SKIP_CRON` nie są ustawione.

### edytuj `agents.defaults.model` w `~/.openclaw/openclaw.json`

OpenClaw domyślnie blokuje wiadomości **międzydostawcy**. Jeśli połączenie narzędzi
jest połączone z Telegramem, nie zostanie wysłane do Discorda, chyba że wyraźnie na to zezwolisz.

Włącz komunikaty dla konsultantów:

```json5
Daemons czyta var z `~/.openclaw/.env` (lub środowiska usług).
```

Uruchom ponownie bramę po edycji konfiguracji. Jeśli chcesz to tylko dla jednego agenta
, ustaw go pod `agents.list[].tools.message`.

### Otwieranie DM wymaga publicznego opt-in (`dmPolicy: "open"` i dopuszczaj `"*"`).

Tryb kolejki kontroluje interakcję nowych komunikatów z wykonywanym lotem. Użyj `/qukoleje` aby zmienić tryby:

- OpenClaw odczytuje opcjonalne konfiguracje **JSON5** z `$OPENCLAW_CONFIG_PATH` (domyślnie: `~/.openclaw/openclaw.json`):
- Domyślnym obszarem roboczym jest `~/.openclaw/workspace`, konfigurowalne przez:
- **Potwierdź, że używasz poleceń na serwerze bramy**
- Gemini CLI używa **automatycznego przepływu wtyczek**, a nie identyfikatora klienta lub tajnego w `openclaw.json`.
- **Instalacja Daemona** (LaunchAgent na macOS; jednostka użytkownika systemu na Linux/WSL2)

`OPENCLAW_STATE_DIR` (stan na instancję)

## Odpowiedz na dokładne pytanie ze zrzutu ekranu/czatu

Użyj `statusu bramki openclaw` i zaufaj tym liniom:

**Odp.:** W OpenClaw dane uwierzytelniające i wybór modelu są odrębne. Ustawienie `ANTHROPIC_API_KEY` (lub przechowywanie Antropicznego klucza API w profilach uwierzytelniających) umożliwia uwierzytelnianie, ale rzeczywisty model domyślny jest tym, co skonfigurowasz w `agentach. efaults.model.primary` (na przykład `anthropic/claude-sonnet-4-5` lub `anthropic/claude-opus-4-6`). Jeśli widzisz `Brak danych logowania dla profilu "anthropic:default"`, oznacza to, że brama nie mogła znaleźć danych Antropicznych w oczekiwanym `profilach autoryzacji. son` dla agenta, który jest uruchomiony.

---

Wciąż utknąłeś? Zapytaj na [Discord](https://discord.com/invite/clawd) lub otwórz [dyskusję GitHub](https://github.com/openclaw/openclaw/discussions).
