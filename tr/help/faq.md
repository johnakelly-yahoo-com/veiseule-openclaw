---
title: "SSS"
---

# FAQ

Gerçek dünyadaki kurulumlar için hızlı yanıtlar ve daha derin sorun giderme (yerel geliştirme, VPS, çoklu ajan, OAuth/API anahtarları, model devre dışı bırakma). Çalışma zamanı tanılamaları için [Sorun Giderme](/gateway/troubleshooting) sayfasına bakın. Tam yapılandırma referansı için [Yapılandırma](/gateway/configuration) sayfasını inceleyin.

## İçindekiler

- [Hızlı başlangıç ve ilk çalıştırma kurulumu]
  - [Takıldım, takılmaktan kurtulmanın en hızlı yolu nedir?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [OpenClaw’ı kurmak ve ayarlamak için önerilen yol nedir?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [Onboarding sonrası panoyu nasıl açarım?](#how-do-i-open-the-dashboard-after-onboarding)
  - [Panoyu localhost’ta ve uzaktan nasıl doğrularım (belirteç)?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [Hangi çalışma zamanına ihtiyacım var?](#what-runtime-do-i-need)
  - [Raspberry Pi üzerinde çalışır mı?](#does-it-run-on-raspberry-pi)
  - [Raspberry Pi kurulumları için ipuçları var mı?](#any-tips-for-raspberry-pi-installs)
  - ["wake up my friend" ekranında takılı kaldı / onboarding çıkmıyor. Ne yapmalıyım?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [Kurulumumu yeniden onboarding yapmadan yeni bir makineye (Mac mini) taşıyabilir miyim?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [En son sürümde nelerin yeni olduğunu nerede görebilirim?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [docs.openclaw.ai’ye erişemiyorum (SSL hatası). Ne yapmalıyım?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [Stable ile beta arasındaki fark nedir?](#whats-the-difference-between-stable-and-beta)
  - [Beta sürümü nasıl kurarım ve beta ile dev arasındaki fark nedir?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [En güncel sürümü nasıl denerim?](#how-do-i-try-the-latest-bits)
  - [Kurulum ve onboarding genellikle ne kadar sürer?](#how-long-does-install-and-onboarding-usually-take)
  - [Kurucu takıldı mı? Daha fazla geri bildirim nasıl alırım?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows kurulumu git bulunamadı veya openclaw tanınmıyor diyor](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [Dokümanlar sorumu yanıtlamadı - daha iyi bir yanıtı nasıl alırım?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [OpenClaw’ı Linux’ta nasıl kurarım?](#how-do-i-install-openclaw-on-linux)
  - [OpenClaw’ı bir VPS’e nasıl kurarım?](#how-do-i-install-openclaw-on-a-vps)
  - [Bulut/VPS kurulum kılavuzları nerede?](#where-are-the-cloudvps-install-guides)
  - [OpenClaw’dan kendini güncellemesini isteyebilir miyim?](#can-i-ask-openclaw-to-update-itself)
  - [Onboarding sihirbazı aslında ne yapar?](#what-does-the-onboarding-wizard-actually-do)
  - [Bunu çalıştırmak için Claude veya OpenAI aboneliğine ihtiyacım var mı?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [API anahtarı olmadan Claude Max aboneliğini kullanabilir miyim](#can-i-use-claude-max-subscription-without-an-api-key)
  - [Anthropic "setup-token" kimlik doğrulaması nasıl çalışır?](#how-does-anthropic-setuptoken-auth-work)
  - [Anthropic setup-token’ı nereden bulurum?](#where-do-i-find-an-anthropic-setuptoken)
  - [Claude abonelik kimlik doğrulamasını (Claude Pro veya Max) destekliyor musunuz?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Neden Anthropic’ten `HTTP 429: rate_limit_error` görüyorum?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [AWS Bedrock destekleniyor mu?](#is-aws-bedrock-supported)
  - [Codex kimlik doğrulaması nasıl çalışır?](#how-does-codex-auth-work)
  - [OpenAI abonelik kimlik doğrulamasını (Codex OAuth) destekliyor musunuz?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [Gemini CLI OAuth’u nasıl kurarım](#how-do-i-set-up-gemini-cli-oauth)
  - [Gündelik sohbetler için yerel bir model uygun mu?](#is-a-local-model-ok-for-casual-chats)
  - [Barındırılan model trafiğini belirli bir bölgede nasıl tutarım?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Bunu kurmak için Mac Mini satın almam gerekiyor mu?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [iMessage desteği için Mac mini gerekli mi?](#do-i-need-a-mac-mini-for-imessage-support)
  - [OpenClaw’ı çalıştırmak için bir Mac mini alırsam, MacBook Pro’ma bağlayabilir miyim?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Bun kullanabilir miyim?](#can-i-use-bun)
  - [Telegram: `allowFrom` alanına ne girilir?](#telegram-what-goes-in-allowfrom)
  - [Birden fazla kişi, farklı OpenClaw örnekleriyle tek bir WhatsApp numarasını kullanabilir mi?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - ["Hızlı sohbet" ajanı ve "kodlama için Opus" ajanı çalıştırabilir miyim?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Homebrew Linux’ta çalışır mı?](#does-homebrew-work-on-linux)
  - [Hacklenebilir (git) kurulum ile npm kurulumu arasındaki fark nedir?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Daha sonra npm ve git kurulumları arasında geçiş yapabilir miyim?](#can-i-switch-between-npm-and-git-installs-later)
  - [Gateway’i dizüstü bilgisayarımda mı yoksa bir VPS’te mi çalıştırmalıyım?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [OpenClaw’ı adanmış bir makinede çalıştırmak ne kadar önemli?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [Minimum VPS gereksinimleri ve önerilen işletim sistemi nedir?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [OpenClaw’ı bir VM içinde çalıştırabilir miyim ve gereksinimler nelerdir](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [OpenClaw nedir?](#what-is-openclaw)
  - [Tek paragrafta OpenClaw nedir?](#what-is-openclaw-in-one-paragraph)
  - [Değer önerisi nedir?](#whats-the-value-proposition)
  - [Yeni kurdum, önce ne yapmalıyım](#i-just-set-it-up-what-should-i-do-first)
  - [OpenClaw için en iyi beş günlük kullanım senaryosu nelerdir](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [OpenClaw, bir SaaS için lead generation outreach, reklamlar ve blog yazılarında yardımcı olabilir mi?](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Web geliştirme için Claude Code'a kıyasla avantajları nelerdir?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Yetenekler ve otomasyon](#skills-and-automation)
  - [Depoyu kirletmeden yetenekleri nasıl özelleştiririm?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Yetenekleri özel bir klasörden yükleyebilir miyim?](#can-i-load-skills-from-a-custom-folder)
  - [Farklı görevler için farklı modelleri nasıl kullanabilirim?](#how-can-i-use-different-models-for-different-tasks)
  - [Bot ağır işler yaparken donuyor. Bunu nasıl dışarı alırım?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron veya hatırlatıcılar tetiklenmiyor. Neyi kontrol etmeliyim?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Linux'ta yetenekleri nasıl kurarım?](#how-do-i-install-skills-on-linux)
  - [OpenClaw görevleri zamanlanmış olarak veya arka planda sürekli çalıştırabilir mi?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Apple macOS'a özel yetenekleri Linux'tan çalıştırabilir miyim?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Notion veya HeyGen entegrasyonunuz var mı?](#do-you-have-a-notion-or-heygen-integration)
  - [Tarayıcı kontrolü için Chrome uzantısını nasıl yüklerim?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandbox ve bellek](#sandboxing-and-memory)
  - [Sandboxing için özel bir doküman var mı?](#is-there-a-dedicated-sandboxing-doc)
  - [Bir ana makine klasörünü sandbox içine nasıl bağlarım?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Bellek nasıl çalışır?](#how-does-memory-work)
  - [Bellek sürekli unutuyor. Kalıcı olmasını nasıl sağlarım?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Bellek sonsuza kadar kalıcı mı? Sınırlar nelerdir?](#does-memory-persist-forever-what-are-the-limits)
  - [Anlamsal bellek araması için bir OpenAI API anahtarı gerekli mi?](#does-semantic-memory-search-require-an-openai-api-key)
- [Where things live on disk](#where-things-live-on-disk)
  - [Is all data used with OpenClaw saved locally?](#is-all-data-used-with-openclaw-saved-locally)
  - [OpenClaw verilerini nerede saklar?](#where-does-openclaw-store-its-data)
  - [AGENTS.md / SOUL.md / USER.md / MEMORY.md nerede olmalı?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [Önerilen yedekleme stratejisi nedir?](#whats-the-recommended-backup-strategy)
  - [OpenClaw'ı tamamen nasıl kaldırırım?](#how-do-i-completely-uninstall-openclaw)
  - [Ajanlar çalışma alanı dışında çalışabilir mi?](#can-agents-work-outside-the-workspace)
  - [Uzak moddayım - oturum deposu nerede?](#im-in-remote-mode-where-is-the-session-store)
- [Yapılandırma temelleri](#config-basics)
  - [Yapılandırma hangi formatta? Nerede?](#what-format-is-the-config-where-is-it)
  - [I set `gateway.bind: "lan"` (or `"tailnet"`) and now nothing listens / the UI says unauthorized](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Neden artık localhost'ta bir token'a ihtiyacım var?](#why-do-i-need-a-token-on-localhost-now)
  - [Yapılandırmayı değiştirdikten sonra yeniden başlatmam gerekiyor mu?](#do-i-have-to-restart-after-changing-config)
  - [Web arama (ve web fetch) nasıl etkinleştirilir?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply yapılandırmamı sildi. Nasıl kurtarırım ve bunu nasıl önlerim?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [Cihazlar arasında uzmanlaşmış worker'larla merkezi bir Gateway'i nasıl çalıştırırım?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [OpenClaw tarayıcısı headless çalışabilir mi?](#can-the-openclaw-browser-run-headless)
  - [Tarayıcı kontrolü için Brave'i nasıl kullanırım?](#how-do-i-use-brave-for-browser-control)
- [Uzak gateway'ler ve node'lar](#remote-gateways-and-nodes)
  - [Komutlar Telegram, gateway ve node'lar arasında nasıl iletilir?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Gateway uzakta barındırılıyorsa ajanım bilgisayarıma nasıl erişebilir?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale bağlı ama yanıt alamıyorum. Ne yapmalıyım?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [İki OpenClaw örneği (yerel + VPS) birbirleriyle konuşabilir mi?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Birden fazla ajan için ayrı VPS’lere ihtiyacım var mı](#do-i-need-separate-vpses-for-multiple-agents)
  - [Bir VPS’ten SSH yerine kişisel dizüstü bilgisayarımda node kullanmanın faydası var mı?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Node'lar bir gateway servisi çalıştırır mı?](#do-nodes-run-a-gateway-service)
  - [Yapılandırmayı uygulamak için bir API / RPC yolu var mı?](#is-there-an-api-rpc-way-to-apply-config)
  - [İlk kurulum için minimal, “makul” bir config nedir?](#whats-a-minimal-sane-config-for-a-first-install)
  - [Bir VPS'te Tailscale'i nasıl kurarım ve Mac'imden nasıl bağlanırım?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Bir Mac node’u uzak bir Gateway’e (Tailscale Serve) nasıl bağlarım?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [İkinci bir dizüstü bilgisayara mı kurmalıyım yoksa sadece bir node mu eklemeliyim?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Env vars and .env loading](#env-vars-and-env-loading)
  - [OpenClaw ortam değişkenlerini nasıl yükler?](#how-does-openclaw-load-environment-variables)
  - ["Gateway’i servis üzerinden başlattım ve env değişkenlerim kayboldu." Ne yapmalıyım?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [`COPILOT_GITHUB_TOKEN` ayarladım ama models status "Shell env: off." diyor. Neden?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Oturumlar ve çoklu sohbetler](#sessions-and-multiple-chats)
  - [Yeni bir konuşma nasıl başlatırım?](#how-do-i-start-a-fresh-conversation)
  - [`/new` hiç göndermesem oturumlar otomatik olarak sıfırlanır mı?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [Bir CEO ve birçok ajandan oluşan bir OpenClaw ekibi kurmanın yolu var mı?](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Bağlam neden görev ortasında kesildi? Nasıl önlerim?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [OpenClaw'ı tamamen sıfırlayıp kurulu tutabilir miyim?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - ["context too large" hataları alıyorum - nasıl sıfırlarım veya compact yaparım?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - [Neden her 30 dakikada bir heartbeat mesajları alıyorum?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [Bir WhatsApp grubunun JID’sini nasıl alırım?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [OpenClaw neden bir grupta yanıt vermiyor?](#why-doesnt-openclaw-reply-in-a-group)
  - [Gruplar/iş parçacıkları DM’lerle bağlam paylaşır mı?](#do-groupsthreads-share-context-with-dms)
  - [Kaç çalışma alanı ve ajan oluşturabilirim?](#how-many-workspaces-and-agents-can-i-create)
  - [Aynı anda birden fazla bot veya sohbet (Slack) çalıştırabilir miyim ve bunu nasıl kurmalıyım?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Modeller: varsayılanlar, seçim, takma adlar, değiştirme](#models-defaults-selection-aliases-switching)
  - [Varsayılan model nedir?](#what-is-the-default-model)
  - [Yapılandırmamı silmeden modelleri nasıl değiştiririm?](#how-do-i-switch-models-without-wiping-my-config)
  - [Kendi barındırdığım modelleri kullanabilir miyim (llama.cpp, vLLM, Ollama)?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [OpenClaw, Flawd ve Krill hangi modelleri kullanıyor?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [Yeniden başlatmadan, anında modelleri nasıl değiştiririm?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [Günlük işler için GPT 5.2’yi, kodlama için Codex 5.3’ü kullanabilir miyim?](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - [Neden "Model … is not allowed" görüyorum ve ardından yanıt gelmiyor?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - [Neden "Unknown model: minimax/MiniMax-M2.1" görüyorum?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [MiniMax’i varsayılan, OpenAI’yi karmaşık görevler için kullanabilir miyim?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [opus / sonnet / gpt yerleşik kısayollar mı?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [Model takma adlarını nasıl tanımlarım/geçersiz kılarım?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [OpenRouter veya Z.AI gibi diğer sağlayıcılardan modelleri nasıl eklerim?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Model devre dışı bırakma (failover) ve "All models failed"](#model-failover-and-all-models-failed)
  - [Failover nasıl çalışır?](#how-does-failover-work)
  - [Bu hata ne anlama geliyor?](#what-does-this-error-mean)
  - [`No credentials found for profile "anthropic:default"` için düzeltme kontrol listesi](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Neden Google Gemini’yi de denedi ve başarısız oldu?](#why-did-it-also-try-google-gemini-and-fail)
- [Auth profilleri: nedir ve nasıl yönetilir](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [Auth profili nedir?](#what-is-an-auth-profile)
  - [Tipik profil kimlikleri nelerdir?](#what-are-typical-profile-ids)
  - [Hangi auth profilinin önce deneneceğini kontrol edebilir miyim?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs API key: fark nedir?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: portlar, "already running" ve uzak mod](#gateway-ports-already-running-and-remote-mode)
  - [Gateway hangi portu kullanır?](#what-port-does-the-gateway-use)
  - [Neden `openclaw gateway status` `Runtime: running` ama `RPC probe: failed` diyor?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [Neden `openclaw gateway status` içinde `Config (cli)` ve `Config (service)` farklı görünüyor?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - ["another gateway instance is already listening" ne demek?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [OpenClaw’ı uzak modda nasıl çalıştırırım?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [Control UI "unauthorized" diyor (veya sürekli yeniden bağlanıyor). Ne yapmalıyım?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [`gateway.bind: "tailnet"` ayarladım ama bağlanamıyor / hiçbir şey dinlemiyor](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Aynı ana bilgisayarda birden fazla Gateway çalıştırabilir miyim?](#can-i-run-multiple-gateways-on-the-same-host)
  - ["invalid handshake" / 1008 kodu ne anlama geliyor?](#what-does-invalid-handshake-code-1008-mean)
- [Günlükleme ve hata ayıklama](#logging-and-debugging)
  - [Loglar nerede?](#where-are-logs)
  - [Gateway servisini nasıl başlatırım/durdururum/yeniden başlatırım?](#how-do-i-startstoprestart-the-gateway-service)
  - [Windows’ta terminali kapattım - OpenClaw’ı nasıl yeniden başlatırım?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Gateway çalışıyor ama yanıtlar gelmiyor. Neyi kontrol etmeliyim?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Disconnected from gateway: no reason" - şimdi ne olacak?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands ağ hatası veriyor. Neyi kontrol etmeliyim?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI hiçbir çıktı göstermiyor. Neyi kontrol etmeliyim?](#tui-shows-no-output-what-should-i-check)
  - [Gateway’i tamamen nasıl durdurup yeniden başlatırım?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [Bir şey başarısız olduğunda daha fazla ayrıntıyı almanın en hızlı yolu nedir?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Medya ve ekler](#media-and-attachments)
  - [Yeteneğim bir image/PDF üretti ama hiçbir şey gönderilmedi](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Güvenlik ve erişim kontrolü](#security-and-access-control)
  - [OpenClaw’u gelen DM’lere açmak güvenli mi?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [Prompt injection yalnızca herkese açık botlar için mi bir endişe?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Botumun kendine ait e-posta/GitHub hesabı veya telefon numarası olmalı mı?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Metin mesajlarım üzerinde ona özerklik verebilir miyim ve bu güvenli mi?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Kişisel asistan görevleri için daha ucuz modeller kullanabilir miyim?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [Telegram’da `/start` çalıştırdım ama eşleştirme kodu almadım](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: kişilerime mesaj atacak mı? Eşleştirme nasıl çalışır?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Sohbet komutları, görevleri iptal etme ve "durmuyor"](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [Dahili sistem mesajlarının sohbette görünmesini nasıl engellerim?](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [Çalışan bir görevi nasıl durdurur/iptal ederim?](#how-do-i-stopcancel-a-running-task)
  - [Telegram’dan Discord’a nasıl mesaj gönderirim? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Bot neden hızlı ardışık mesajları “görmezden geliyor” gibi hissediliyor?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## First 60 seconds if something's broken

1. **Quick status (first check)**

   ```bash
   openclaw status
   ```

   Fast local summary: OS + update, gateway/service reachability, agents/sessions, provider config + runtime issues (when gateway is reachable).

2. **Pasteable report (safe to share)**

   ```bash
   openclaw status --all
   ```

   Read-only diagnosis with log tail (tokens redacted).

3. **Daemon + port state**

   ```bash
   openclaw gateway status
   ```

   Shows supervisor runtime vs RPC reachability, the probe target URL, and which config the service likely used.

4. **Deep probes**

   ```bash
   openclaw status --deep
   ```

   Runs gateway health checks + provider probes (requires a reachable gateway). See [Health](/gateway/health).

5. **Tail the latest log**

   ```bash
   openclaw logs --follow
   ```

   If RPC is down, fall back to:

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   File logs are separate from service logs; see [Logging](/logging) and [Troubleshooting](/gateway/troubleshooting).

6. **Run the doctor (repairs)**

   ```bash
   openclaw doctor
   ```

   Repairs/migrates config/state + runs health checks. See [Doctor](/gateway/doctor).

7. **Gateway snapshot**

   ```bash
   openclaw health --json
   openclaw health --verbose   # shows the target URL + config path on errors
   ```

   Asks the running gateway for a full snapshot (WS-only). See [Health](/gateway/health).

## Hızlı başlangıç ve ilk çalıştırma kurulumu

### Im stuck whats the fastest way to get unstuck

Makinenizi **görebilen** yerel bir AI ajanı kullanın. Discord’da sormaktan çok daha etkilidir, çünkü “takıldım” vakalarının çoğu uzaktan yardımcıların inceleyemeyeceği **yerel config veya ortam sorunlarıdır**.

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

Bu araçlar repo’yu okuyabilir, komut çalıştırabilir, logları inceleyebilir ve makine seviyesindeki kurulumunuzu (PATH, servisler, izinler, auth dosyaları) düzeltebilir. Onlara hacklenebilir (git) kurulum üzerinden **tam kaynak checkout** verin:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

Bu, OpenClaw’ı **git checkout’tan** kurar; böylece ajan çalıştırdığınız sürümün kodunu + dokümanlarını okuyabilir. Daha sonra installer’ı `--install-method git` olmadan yeniden çalıştırarak stable’a geri dönebilirsiniz.

İpucu: Ajanın düzeltmeyi **planlamasını ve denetlemesini** (adım adım) isteyin, ardından yalnızca gerekli komutları çalıştırın. Bu, değişiklikleri küçük ve denetlenebilir tutar.

Gerçek bir bug veya düzeltme bulursanız, lütfen bir GitHub issue açın veya PR gönderin:
https://github.com/openclaw/openclaw/issues  
https://github.com/openclaw/openclaw/pulls

Yardım isterken şu komutlarla başlayın (çıktıları paylaşın):

```bash
openclaw status
openclaw models status
openclaw doctor
```

Ne yaparlar:

- `openclaw status`: gateway/agent sağlığı + temel config’in hızlı özeti.
- `openclaw models status`: sağlayıcı auth + model erişilebilirliğini kontrol eder.
- `openclaw doctor`: yaygın config/durum sorunlarını doğrular ve onarır.

Diğer faydalı CLI kontrolleri: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`.

Hızlı debug döngüsü: [First 60 seconds if something's broken](#first-60-seconds-if-somethings-broken).  
Kurulum dokümanları: [Install](/install), [Installer flags](/install/installer), [Updating](/install/updating).

---

## Ekran görüntüsündeki/sohbet kaydındaki soruya birebir yanıt

**S: "Anthropic için bir API anahtarıyla varsayılan model nedir?"**

**C:** OpenClaw’da kimlik bilgileri (credentials) ile model seçimi birbirinden ayrıdır. `ANTHROPIC_API_KEY` ayarlamak (veya Anthropic API anahtarını auth profillerinde saklamak) yalnızca kimlik doğrulamayı etkinleştirir. Gerçek varsayılan model, `agents.defaults.model.primary` içinde yapılandırdığınız değerdir (örneğin `anthropic/claude-sonnet-4-5` veya `anthropic/claude-opus-4-6`).  

Eğer `No credentials found for profile "anthropic:default"` hatasını görüyorsanız, bu çalışan ajan için beklenen `auth-profiles.json` dosyasında Anthropic kimlik bilgilerinin bulunamadığı anlamına gelir.

---

Hâlâ takıldınız mı? [Discord](https://discord.com/invite/clawd) üzerinden sorun veya bir [GitHub discussion](https://github.com/openclaw/openclaw/discussions) açın.

