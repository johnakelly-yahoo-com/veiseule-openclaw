---
title: "FAQ"
---

# FAQ

လက်တွေ့အသုံးချ setup များအတွက် (local dev, VPS, multi-agent, OAuth/API keys, model failover) အမြန်အဖြေများနှင့် ပိုမိုနက်ရှိုင်းသော troubleshooting များ။ runtime diagnostics အတွက် [Troubleshooting](/gateway/troubleshooting) ကို ကြည့်ပါ။ config အပြည့်အစုံအတွက် [Configuration](/gateway/configuration) ကို ကြည့်ပါ။

## Table of contents

- [အမြန်စတင်ခြင်းနှင့် ပထမဆုံး အသုံးပြုမှုအတွက် စနစ်တကျပြင်ဆင်ခြင်း]
  - [ကျွန်ုပ် အခက်အခဲကြုံနေပါတယ် — အမြန်ဆုံး ဖြေရှင်းနိုင်မည့် နည်းလမ်းက ဘာလဲ?](#im-stuck-whats-the-fastest-way-to-get-unstuck)
  - [OpenClaw ကို ထည့်သွင်းပြီး စနစ်တကျပြင်ဆင်ရန် အကြံပြုထားသော နည်းလမ်းက ဘာလဲ?](#whats-the-recommended-way-to-install-and-set-up-openclaw)
  - [onboarding ပြီးနောက် dashboard ကို ဘယ်လိုဖွင့်ရမလဲ?](#how-do-i-open-the-dashboard-after-onboarding)
  - [localhost နှင့် remote တွင် dashboard (token) ကို ဘယ်လို authenticate လုပ်ရမလဲ?](#how-do-i-authenticate-the-dashboard-token-on-localhost-vs-remote)
  - [မည်သည့် runtime ကို လိုအပ်ပါသလဲ?](#what-runtime-do-i-need)
  - [Raspberry Pi ပေါ်တွင် အလုပ်လုပ်ပါသလား?](#does-it-run-on-raspberry-pi)
  - [Raspberry Pi တွင် ထည့်သွင်းရာတွင် အသုံးဝင်သော အကြံပြုချက်များ ရှိပါသလား?](#any-tips-for-raspberry-pi-installs)
  - ["wake up my friend" မှာပဲ ပိတ်မိနေပြီး onboarding မဖွင့်နိုင်ပါ။ ဘာလုပ်ရမလဲ?](#it-is-stuck-on-wake-up-my-friend-onboarding-will-not-hatch-what-now)
  - [onboarding ကို ပြန်မလုပ်ဘဲ setup ကို စက်အသစ် (Mac mini) သို့ ပြောင်းရွှေ့နိုင်ပါသလား?](#can-i-migrate-my-setup-to-a-new-machine-mac-mini-without-redoing-onboarding)
  - [နောက်ဆုံးဗားရှင်းတွင် အသစ်ပါဝင်သော အရာများကို ဘယ်မှာကြည့်နိုင်မလဲ?](#where-do-i-see-what-is-new-in-the-latest-version)
  - [I can't access docs.openclaw.ai (SSL error). What now?](#i-cant-access-docsopenclawai-ssl-error-what-now)
  - [What's the difference between stable and beta?](#whats-the-difference-between-stable-and-beta)
  - [How do I install the beta version, and what's the difference between beta and dev?](#how-do-i-install-the-beta-version-and-whats-the-difference-between-beta-and-dev)
  - [How do I try the latest bits?](#how-do-i-try-the-latest-bits)
  - [How long does install and onboarding usually take?](#how-long-does-install-and-onboarding-usually-take)
  - [Installer stuck? How do I get more feedback?](#installer-stuck-how-do-i-get-more-feedback)
  - [Windows install says git not found or openclaw not recognized](#windows-install-says-git-not-found-or-openclaw-not-recognized)
  - [The docs didn't answer my question - how do I get a better answer?](#the-docs-didnt-answer-my-question-how-do-i-get-a-better-answer)
  - [How do I install OpenClaw on Linux?](#how-do-i-install-openclaw-on-linux)
  - [How do I install OpenClaw on a VPS?](#how-do-i-install-openclaw-on-a-vps)
  - [Where are the cloud/VPS install guides?](#where-are-the-cloudvps-install-guides)
  - [Can I ask OpenClaw to update itself?](#can-i-ask-openclaw-to-update-itself)
  - [What does the onboarding wizard actually do?](#what-does-the-onboarding-wizard-actually-do)
  - [Do I need a Claude or OpenAI subscription to run this?](#do-i-need-a-claude-or-openai-subscription-to-run-this)
  - [Can I use Claude Max subscription without an API key](#can-i-use-claude-max-subscription-without-an-api-key)
  - [How does Anthropic "setup-token" auth work?](#how-does-anthropic-setuptoken-auth-work)
  - [Where do I find an Anthropic setup-token?](#where-do-i-find-an-an-anthropic-setuptoken)
  - [Do you support Claude subscription auth (Claude Pro or Max)?](#do-you-support-claude-subscription-auth-claude-pro-or-max)
  - [Why am I seeing `HTTP 429: rate_limit_error` from Anthropic?](#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)
  - [Is AWS Bedrock supported?](#is-aws-bedrock-supported)
  - [How does Codex auth work?](#how-does-codex-auth-work)
  - [Do you support OpenAI subscription auth (Codex OAuth)?](#do-you-support-openai-subscription-auth-codex-oauth)
  - [How do I set up Gemini CLI OAuth](#how-do-i-set-up-gemini-cli-oauth)
  - [Is a local model OK for casual chats?](#is-a-local-model-ok-for-casual-chats)
  - [How do I keep hosted model traffic in a specific region?](#how-do-i-keep-hosted-model-traffic-in-a-specific-region)
  - [Do I have to buy a Mac Mini to install this?](#do-i-have-to-buy-a-mac-mini-to-install-this)
  - [Do I need a Mac mini for iMessage support?](#do-i-need-a-mac-mini-for-imessage-support)
  - [If I buy a Mac mini to run OpenClaw, can I connect it to my MacBook Pro?](#if-i-buy-a-mac-mini-to-run-openclaw-can-i-connect-it-to-my-macbook-pro)
  - [Can I use Bun?](#can-i-use-bun)
  - [Telegram: what goes in `allowFrom`?](#telegram-what-goes-in-allowfrom)
  - [Can multiple people use one WhatsApp number with different OpenClaw instances?](#can-multiple-people-use-one-whatsapp-number-with-different-openclaw-instances)
  - [Can I run a "fast chat" agent and an "Opus for coding" agent?](#can-i-run-a-fast-chat-agent-and-an-opus-for-coding-agent)
  - [Does Homebrew work on Linux?](#does-homebrew-work-on-linux)
  - [What's the difference between the hackable (git) install and npm install?](#whats-the-difference-between-the-hackable-git-install-and-npm-install)
  - [Can I switch between npm and git installs later?](#can-i-switch-between-npm-and-git-installs-later)
  - [Should I run the Gateway on my laptop or a VPS?](#should-i-run-the-gateway-on-my-laptop-or-a-vps)
  - [How important is it to run OpenClaw on a dedicated machine?](#how-important-is-it-to-run-openclaw-on-a-dedicated-machine)
  - [What are the minimum VPS requirements and recommended OS?](#what-are-the-minimum-vps-requirements-and-recommended-os)
  - [Can I run OpenClaw in a VM and what are the requirements](#can-i-run-openclaw-in-a-vm-and-what-are-the-requirements)
- [OpenClaw ဆိုတာဘာလဲ?](#what-is-openclaw)
  - [OpenClaw ဆိုတာ တစ်ပိုဒ်အကျဉ်းချုပ်ဖြင့် ဘာလဲ?](#what-is-openclaw-in-one-paragraph)
  - [တန်ဖိုးအဓိပ္ပါယ်က ဘာလဲ?](#whats-the-value-proposition)
  - [ခုတင်ပဲ set up လုပ်ပြီးပြီ — ဘာလုပ်ရင်ကောင်းမလဲ](#i-just-set-it-up-what-should-i-do-first)
  - [OpenClaw ရဲ့ နေ့စဉ်အသုံးအများဆုံး use cases ၅ ခုက ဘာတွေလဲ](#what-are-the-top-five-everyday-use-cases-for-openclaw)
  - [SaaS တစ်ခုအတွက် lead gen outreach ads နဲ့ blogs တွေမှာ ကူညီနိုင်ပါသလား](#can-openclaw-help-with-lead-gen-outreach-ads-and-blogs-for-a-saas)
  - [Web development မှာ Claude Code နဲ့ နှိုင်းယှဉ်ရင် ဘာအားသာချက်တွေ ရှိသလဲ?](#what-are-the-advantages-vs-claude-code-for-web-development)
- [Skills and automation](#skills-and-automation)
  - [Repo ကို မညစ်စေဘဲ skills ကို ဘယ်လို customize လုပ်ရမလဲ?](#how-do-i-customize-skills-without-keeping-the-repo-dirty)
  - [Custom folder မှ skills ကို load လုပ်လို့ ရပါသလား?](#can-i-load-skills-from-a-custom-folder)
  - [Tasks အလိုက် model မတူအောင် ဘယ်လို သုံးရမလဲ?](#how-can-i-use-different-models-for-different-tasks)
  - [Bot က အလုပ်ကြီးလုပ်နေချိန် freeze ဖြစ်သွားတယ်။ ဘယ်လို offload လုပ်ရမလဲ?](#the-bot-freezes-while-doing-heavy-work-how-do-i-offload-that)
  - [Cron သို့မဟုတ် reminders မလုပ်ဆောင်ပါ။ ဘာစစ်ဆေးရမလဲ?](#cron-or-reminders-do-not-fire-what-should-i-check)
  - [Linux မှာ skills ကို ဘယ်လို install လုပ်ရမလဲ?](#how-do-i-install-skills-on-linux)
  - [OpenClaw က schedule အတိုင်း သို့မဟုတ် background မှာ ဆက်တိုက် အလုပ်လုပ်နိုင်ပါသလား?](#can-openclaw-run-tasks-on-a-schedule-or-continuously-in-the-background)
  - [Linux မှာ Apple macOS-only skills များကို run လုပ်နိုင်ပါသလား?](#can-i-run-apple-macos-only-skills-from-linux)
  - [Notion သို့မဟုတ် HeyGen integration ရှိပါသလား?](#do-you-have-a-notion-or-heygen-integration)
  - [Browser takeover အတွက် Chrome extension ကို ဘယ်လို install လုပ်ရမလဲ?](#how-do-i-install-the-chrome-extension-for-browser-takeover)
- [Sandboxing and memory](#sandboxing-and-memory)
  - [Sandboxing အတွက် သီးသန့် doc ရှိပါသလား?](#is-there-a-dedicated-sandboxing-doc)
  - [Sandbox ထဲကို host folder တစ်ခု bind လုပ်မလား?](#how-do-i-bind-a-host-folder-into-the-sandbox)
  - [Memory က ဘယ်လို အလုပ်လုပ်သလဲ?](#how-does-memory-work)
  - [Memory က မေ့နေတတ်တယ်။ ဘယ်လို ထိန်းထားမလဲ?](#memory-keeps-forgetting-things-how-do-i-make-it-stick)
  - [Memory က အမြဲတမ်း တည်ရှိနေပါသလား? ကန့်သတ်ချက်တွေ ဘာတွေလဲ?](#does-memory-persist-forever-what-are-the-limits)
  - [Semantic memory search အတွက် OpenAI API key လိုအပ်ပါသလား?](#does-semantic-memory-search-require-an-openai-api-key)
- [Where things live on disk](#where-things-live-on-disk)
  - [OpenClaw နဲ့ သုံးတဲ့ data အားလုံးကို local မှာ သိမ်းထားပါသလား?](#is-all-data-used-with-openclaw-saved-locally)
  - [OpenClaw က data ကို ဘယ်မှာ သိမ်းထားသလဲ?](#where-does-openclaw-store-its-data)
  - [AGENTS.md / SOUL.md / USER.md / MEMORY.md ကို ဘယ်မှာထားသင့်လဲ?](#where-should-agentsmd-soulmd-usermd-memorymd-live)
  - [အကြံပြုထားတဲ့ backup strategy က ဘာလဲ?](#whats-the-recommended-backup-strategy)
  - [OpenClaw ကို အပြည့်အဝ uninstall ဘယ်လိုလုပ်မလဲ?](#how-do-i-completely-uninstall-openclaw)
  - [Agents များဟာ workspace အပြင်ဘက်မှာ အလုပ်လုပ်နိုင်ပါသလား?](#can-agents-work-outside-the-workspace)
  - [Remote mode မှာ session store ဘယ်မှာလဲ?](#im-in-remote-mode-where-is-the-session-store)
- [Config basics](#config-basics)
  - [Config format ကဘာလဲ? ဘယ်မှာရှိလဲ?](#what-format-is-the-config-where-is-it)
  - [`gateway.bind: "lan"` (သို့မဟုတ် `"tailnet"`) ကို သတ်မှတ်လိုက်ပြီးနောက် ဘာမှမနားထောင်တော့ပါ / UI မှာ unauthorized လို့ ပြနေပါတယ်](#i-set-gatewaybind-lan-or-tailnet-and-now-nothing-listens-the-ui-says-unauthorized)
  - [Localhost မှာ token လိုအပ်လာတာ ဘာကြောင့်လဲ?](#why-do-i-need-a-token-on-localhost-now)
  - [Config ပြောင်းပြီးနောက် restart လုပ်ရပါသလား?](#do-i-have-to-restart-after-changing-config)
  - [Web search (နှင့် web fetch) ကို ဘယ်လို enable လုပ်မလဲ?](#how-do-i-enable-web-search-and-web-fetch)
  - [config.apply က config ကို ဖျက်သွားတယ်။ ဘယ်လို ပြန်ယူပြီး ဘယ်လို ရှောင်ရှားမလဲ?](#configapply-wiped-my-config-how-do-i-recover-and-avoid-this)
  - [Devices အမျိုးမျိုးပေါ်မှာ specialized workers တွေနဲ့ central Gateway ကို ဘယ်လို chạy လုပ်မလဲ?](#how-do-i-run-a-central-gateway-with-specialized-workers-across-devices)
  - [OpenClaw browser ကို headless အဖြစ် chạy လုပ်လို့ရပါသလား?](#can-the-openclaw-browser-run-headless)
  - [Browser control အတွက် Brave ကို ဘယ်လို အသုံးပြုရမလဲ?](#how-do-i-use-brave-for-browser-control)
- [Remote gateways and nodes](#remote-gateways-and-nodes)
  - [Telegram, gateway နှင့် nodes ကြား commands များ ဘယ်လို propagate ဖြစ်သလဲ?](#how-do-commands-propagate-between-telegram-the-gateway-and-nodes)
  - [Gateway ကို remote မှ host လုပ်ထားလျှင် agent က ကျွန်ုပ်၏ computer ကို ဘယ်လို ဝင်ရောက်နိုင်မလဲ?](#how-can-my-agent-access-my-computer-if-the-gateway-is-hosted-remotely)
  - [Tailscale ချိတ်ဆက်ထားပေမယ့် အကြောင်းပြန်မရပါ။ ဘာလုပ်ရမလဲ?](#tailscale-is-connected-but-i-get-no-replies-what-now)
  - [OpenClaw instance နှစ်ခု (local + VPS) အချင်းချင်း စကားပြောနိုင်ပါသလား?](#can-two-openclaw-instances-talk-to-each-other-local-vps)
  - [Multiple agents အတွက် VPS သီးသန့်လိုအပ်ပါသလား](#do-i-need-separate-vpses-for-multiple-agents)
  - [VPS မှ SSH မသုံးဘဲ personal laptop ကို node အဖြစ် သုံးတာက အကျိုးရှိပါသလား?](#is-there-a-benefit-to-using-a-node-on-my-personal-laptop-instead-of-ssh-from-a-vps)
  - [Nodes များက gateway service ကို chạy လုပ်ပါသလား?](#do-nodes-run-a-gateway-service)
  - [Config ကို apply လုပ်ဖို့ API / RPC နည်းလမ်း ရှိပါသလား?](#is-there-an-api-rpc-way-to-apply-config)
  - [ပထမဆုံး install အတွက် အနည်းဆုံး “sane” config က ဘာလဲ?](#whats-a-minimal-sane-config-for-a-first-install)
  - [VPS ပေါ်မှာ Tailscale ကို ဘယ်လို setup လုပ်ပြီး Mac မှ ချိတ်ဆက်ရမလဲ?](#how-do-i-set-up-tailscale-on-a-vps-and-connect-from-my-mac)
  - [Mac node တစ်ခုကို remote Gateway (Tailscale Serve) သို့ ဘယ်လို ချိတ်ဆက်ရမလဲ?](#how-do-i-connect-a-mac-node-to-a-remote-gateway-tailscale-serve)
  - [Laptop ဒုတိယတစ်လုံးမှာ install လုပ်မလား ဒါမှမဟုတ် node တစ်ခု ထည့်ရုံလား?](#should-i-install-on-a-second-laptop-or-just-add-a-node)
- [Env vars and .env loading](#env-vars-and-env-loading)
  - [OpenClaw က environment variables များကို ဘယ်လို load လုပ်သလဲ?](#how-does-openclaw-load-environment-variables)
  - ["Service ကနေ Gateway ကို စတင်လိုက်တော့ env vars များ ပျောက်သွားတယ်" — အခု ဘာလုပ်ရမလဲ?](#i-started-the-gateway-via-the-service-and-my-env-vars-disappeared-what-now)
  - [`COPILOT_GITHUB_TOKEN` ကို သတ်မှတ်ထားပေမယ့် models status မှာ "Shell env: off." လို့ ပြနေတယ်။ ဘာကြောင့်လဲ?](#i-set-copilotgithubtoken-but-models-status-shows-shell-env-off-why)
- [Sessions and multiple chats](#sessions-and-multiple-chats)
  - [Conversation အသစ်ကို ဘယ်လို စတင်မလဲ?](#how-do-i-start-a-fresh-conversation)
  - [`/new` မပို့ဘူးဆိုရင် sessions က အလိုအလျောက် reset ဖြစ်ပါသလား?](#do-sessions-reset-automatically-if-i-never-send-new)
  - [OpenClaw instances များကို CEO တစ်ယောက်နဲ့ agents များအဖြစ် team ဖန်တီးနိုင်ပါသလား](#is-there-a-way-to-make-a-team-of-openclaw-instances-one-ceo-and-many-agents)
  - [Task အလယ်မှာ context truncate ဖြစ်သွားတယ်။ ဘယ်လို ကာကွယ်မလဲ?](#why-did-context-get-truncated-midtask-how-do-i-prevent-it)
  - [Install ထားပြီးသားကို မဖျက်ဘဲ OpenClaw ကို အပြည့်အဝ reset ဘယ်လိုလုပ်မလဲ?](#how-do-i-completely-reset-openclaw-but-keep-it-installed)
  - ["context too large" errors တွေ ရနေတယ် — ဘယ်လို reset သို့မဟုတ် compact လုပ်မလဲ?](#im-getting-context-too-large-errors-how-do-i-reset-or-compact)
  - ["LLM request rejected: messages.N.content.X.tool_use.input: Field required" ကို ဘာကြောင့် တွေ့နေရတာလဲ?](#why-am-i-seeing-llm-request-rejected-messagesncontentxtooluseinput-field-required)
  - [မိနစ် 30 တစ်ကြိမ် heartbeat messages တွေ ရနေတာ ဘာကြောင့်လဲ?](#why-am-i-getting-heartbeat-messages-every-30-minutes)
  - [WhatsApp group ထဲမှာ "bot account" တစ်ခု ထည့်ဖို့ လိုအပ်ပါသလား?](#do-i-need-to-add-a-bot-account-to-a-whatsapp-group)
  - [WhatsApp group ရဲ့ JID ကို ဘယ်လို ရယူမလဲ?](#how-do-i-get-the-jid-of-a-whatsapp-group)
  - [Group ထဲမှာ OpenClaw က reply မလုပ်တာ ဘာကြောင့်လဲ?](#why-doesnt-openclaw-reply-in-a-group)
  - [Groups/threads တွေက DMs နဲ့ context မျှဝေပါသလား?](#do-groupsthreads-share-context-with-dms)
  - [Workspace နဲ့ agents ဘယ်လောက် ဖန်တီးနိုင်ပါသလဲ?](#how-many-workspaces-and-agents-can-i-create)
  - [Slack မှာ bots/chats အများကြီးကို တပြိုင်နက် chạy လုပ်နိုင်ပါသလား? ဘယ်လို setup လုပ်မလဲ?](#can-i-run-multiple-bots-or-chats-at-the-same-time-slack-and-how-should-i-set-that-up)
- [Models: defaults, selection, aliases, switching](#models-defaults-selection-aliases-switching)
  - ["Default model" ဆိုတာ ဘာလဲ?](#what-is-the-default-model)
  - [ဘယ် model ကို အကြံပြုပါသလဲ?](#what-model-do-you-recommend)
  - [Config ကို မဖျက်ဘဲ model များကို ဘယ်လို ပြောင်းလဲရမလဲ?](#how-do-i-switch-models-without-wiping-my-config)
  - [Self-hosted models (llama.cpp, vLLM, Ollama) ကို အသုံးပြုနိုင်ပါသလား?](#can-i-use-selfhosted-models-llamacpp-vllm-ollama)
  - [OpenClaw, Flawd, Krill တို့က model အတွက် ဘာကို သုံးပါသလဲ?](#what-do-openclaw-flawd-and-krill-use-for-models)
  - [Restart မလုပ်ဘဲ model ကို ချက်ချင်း ပြောင်းနိုင်ပါသလား?](#how-do-i-switch-models-on-the-fly-without-restarting)
  - [နေ့စဉ်အလုပ်များအတွက် GPT 5.2 နဲ့ coding အတွက် Codex 5.3 ကို သုံးနိုင်ပါသလား](#can-i-use-gpt-52-for-daily-tasks-and-codex-53-for-coding)
  - ["Model … is not allowed" လို့ ပြပြီး reply မလာတာ ဘာကြောင့်လဲ?](#why-do-i-see-model-is-not-allowed-and-then-no-reply)
  - ["Unknown model: minimax/MiniMax-M2.1" ကို ဘာကြောင့် တွေ့နေရတာလဲ?](#why-do-i-see-unknown-model-minimaxminimaxm21)
  - [MiniMax ကို default အဖြစ်ထားပြီး OpenAI ကို complex tasks အတွက် သုံးနိုင်ပါသလား?](#can-i-use-minimax-as-my-default-and-openai-for-complex-tasks)
  - [opus / sonnet / gpt က built-in shortcuts တွေလား?](#are-opus-sonnet-gpt-builtin-shortcuts)
  - [Model shortcuts (aliases) ကို ဘယ်လို define/override လုပ်မလဲ?](#how-do-i-defineoverride-model-shortcuts-aliases)
  - [OpenRouter သို့မဟုတ် Z.AI ကဲ့သို့ provider များမှ models ကို ဘယ်လို ထည့်မလဲ?](#how-do-i-add-models-from-other-providers-like-openrouter-or-zai)
- [Model failover and "All models failed"](#model-failover-and-all-models-failed)
  - [Failover က ဘယ်လို အလုပ်လုပ်သလဲ?](#how-does-failover-work)
  - [ဒီ error က ဘာကို ဆိုလိုတာလဲ?](#what-does-this-error-mean)
  - [`No credentials found for profile "anthropic:default"` အတွက် Fix checklist](#fix-checklist-for-no-credentials-found-for-profile-anthropicdefault)
  - [Google Gemini ကိုပါ စမ်းပြီး fail ဖြစ်သွားတာ ဘာကြောင့်လဲ?](#why-did-it-also-try-google-gemini-and-fail)
- [Auth profiles: what they are and how to manage them](#auth-profiles-what-they-are-and-how-to-manage-them)
  - [Auth profile ဆိုတာ ဘာလဲ?](#what-is-an-auth-profile)
  - [ပုံမှန် profile IDs တွေက ဘာတွေလဲ?](#what-are-typical-profile-ids)
  - [ဘယ် auth profile ကို အရင် စမ်းမလဲဆိုတာ ထိန်းချုပ်လို့ ရပါသလား?](#can-i-control-which-auth-profile-is-tried-first)
  - [OAuth vs API key: ဘာကွာခြားလဲ?](#oauth-vs-api-key-whats-the-difference)
- [Gateway: ports, "already running", and remote mode](#gateway-ports-already-running-and-remote-mode)
  - [Gateway က ဘယ် port ကို သုံးသလဲ?](#what-port-does-the-gateway-use)
  - [`openclaw gateway status` မှာ `Runtime: running` လို့ ပြပေမယ့် `RPC probe: failed` လို့ ပြတာ ဘာကြောင့်လဲ?](#why-does-openclaw-gateway-status-say-runtime-running-but-rpc-probe-failed)
  - [`openclaw gateway status` မှာ `Config (cli)` နဲ့ `Config (service)` မတူတာ ဘာကြောင့်လဲ?](#why-does-openclaw-gateway-status-show-config-cli-and-config-service-different)
  - ["another gateway instance is already listening" ဆိုတာ ဘာကို ဆိုလိုတာလဲ?](#what-does-another-gateway-instance-is-already-listening-mean)
  - [OpenClaw ကို remote mode (client က အခြား Gateway သို့ ချိတ်ဆက်) နဲ့ ဘယ်လို chạy လုပ်မလဲ?](#how-do-i-run-openclaw-in-remote-mode-client-connects-to-a-gateway-elsewhere)
  - [Control UI မှာ "unauthorized" လို့ ပြသခြင်း (သို့မဟုတ် အမြဲ reconnect ဖြစ်နေခြင်း) ဖြစ်ရင် ဘာလုပ်ရမလဲ?](#the-control-ui-says-unauthorized-or-keeps-reconnecting-what-now)
  - [`gateway.bind: "tailnet"` ကို သတ်မှတ်ထားပေမယ့် bind မလုပ်နိုင် / ဘာမှမနားထောင်တော့ပါ](#i-set-gatewaybind-tailnet-but-it-cant-bind-nothing-listens)
  - [Host တစ်ခုတည်းပေါ်မှာ Gateways များစွာ chạy လုပ်နိုင်ပါသလား?](#can-i-run-multiple-gateways-on-the-same-host)
  - ["invalid handshake" / code 1008 ဆိုတာ ဘာကို ဆိုလိုတာလဲ?](#what-does-invalid-handshake-code-1008-mean)
- [Logging and debugging](#logging-and-debugging)
  - [Logs တွေ ဘယ်မှာလဲ?](#where-are-logs)
  - [Gateway service ကို ဘယ်လို start/stop/restart လုပ်မလဲ?](#how-do-i-startstoprestart-the-gateway-service)
  - [Windows မှာ terminal ကို ပိတ်လိုက်ပြီး OpenClaw ကို ဘယ်လို ပြန်စမလဲ?](#i-closed-my-terminal-on-windows-how-do-i-restart-openclaw)
  - [Gateway က chạy နေပြီ ဒါပေမယ့် reply မရောက်ဘူး။ ဘာစစ်ဆေးရမလဲ?](#the-gateway-is-up-but-replies-never-arrive-what-should-i-check)
  - ["Disconnected from gateway: no reason" — အခု ဘာလုပ်ရမလဲ?](#disconnected-from-gateway-no-reason-what-now)
  - [Telegram setMyCommands fails with network errors. ဘာစစ်ဆေးရမလဲ?](#telegram-setmycommands-fails-with-network-errors-what-should-i-check)
  - [TUI မှာ output မပေါ်ပါ။ ဘာစစ်ဆေးရမလဲ?](#tui-shows-no-output-what-should-i-check)
  - [Gateway ကို အပြည့်အဝ ရပ်ပြီး ပြန်စတင်မလား?](#how-do-i-completely-stop-then-start-the-gateway)
  - [ELI5: `openclaw gateway restart` vs `openclaw gateway`](#eli5-openclaw-gateway-restart-vs-openclaw-gateway)
  - [တစ်ခုခု fail ဖြစ်တဲ့အခါ အသေးစိတ်အချက်အလက်များကို အမြန်ဆုံး ဘယ်လို ရယူမလဲ?](#whats-the-fastest-way-to-get-more-details-when-something-fails)
- [Media and attachments](#media-and-attachments)
  - [Skill က image/PDF ထုတ်ပေးပေမယ့် ဘာမှ မပို့ခဲ့ပါ](#my-skill-generated-an-imagepdf-but-nothing-was-sent)
- [Security and access control](#security-and-access-control)
  - [OpenClaw ကို inbound DMs အတွက် ဖွင့်ထားရင် လုံခြုံပါသလား?](#is-it-safe-to-expose-openclaw-to-inbound-dms)
  - [Prompt injection က public bots အတွက်ပဲ စိုးရိမ်ရတာလား?](#is-prompt-injection-only-a-concern-for-public-bots)
  - [Bot အတွက် သီးခြား email / GitHub account / phone number တစ်ခု လိုအပ်ပါသလား?](#should-my-bot-have-its-own-email-github-account-or-phone-number)
  - [Text messages များကို အလိုအလျောက် စီမံခန့်ခွဲခွင့် ပေးနိုင်ပါသလား? လုံခြုံပါသလား?](#can-i-give-it-autonomy-over-my-text-messages-and-is-that-safe)
  - [Personal assistant tasks အတွက် စျေးသက်သာသော models ကို သုံးနိုင်ပါသလား?](#can-i-use-cheaper-models-for-personal-assistant-tasks)
  - [Telegram မှာ `/start` ကို chạy လုပ်ပေမယ့် pairing code မရပါ](#i-ran-start-in-telegram-but-didnt-get-a-pairing-code)
  - [WhatsApp: ကျွန်ုပ်၏ contacts များကို message ပို့မလား? Pairing က ဘယ်လို အလုပ်လုပ်သလဲ?](#whatsapp-will-it-message-my-contacts-how-does-pairing-work)
- [Chat commands, aborting tasks, and "it won't stop"](#chat-commands-aborting-tasks-and-it-wont-stop)
  - [Chat ထဲမှာ internal system messages မပေါ်အောင် ဘယ်လိုလုပ်ရမလဲ](#how-do-i-stop-internal-system-messages-from-showing-in-chat)
  - [လည်ပတ်နေသော task ကို ဘယ်လို ရပ်/ပယ်ဖျက်မလဲ?](#how-do-i-stopcancel-a-running-task)
  - [Telegram မှ Discord သို့ message ပို့မလား? ("Cross-context messaging denied")](#how-do-i-send-a-discord-message-from-telegram-crosscontext-messaging-denied)
  - [Bot က မြန်မြန်ဆန်ဆန် ပို့သော messages များကို "လစ်လျူရှု" သလို ခံစားရတာ ဘာကြောင့်လဲ?](#why-does-it-feel-like-the-bot-ignores-rapidfire-messages)

## First 60 seconds if something's broken

1. **အမြန်အခြေအနေ (ပထမစစ်ဆေးရန်)**

   ```bash
   openclaw status
   ```

   အမြန် local အကျဉ်းချုပ်: OS + update, gateway/service ရရှိနိုင်မှု, agents/sessions, provider config + runtime ပြဿနာများ (gateway ရရှိနိုင်ပါက)။

2. **မျှဝေနိုင်သော အစီရင်ခံစာ (အန္တရာယ်ကင်း)**

   ```bash
   openclaw status --all
   ```

   Read-only စစ်ဆေးချက်နှင့် log tail (tokens မပါဝင်)။

3. **Daemon + port အခြေအနေ**

   ```bash
   openclaw gateway status
   ```

   Supervisor runtime နှင့် RPC ရရှိနိုင်မှု၊ probe target URL နှင့် service သုံးထားနိုင်သည့် config ကို ပြသည်။

4. **နက်ရှိုင်းသော စစ်ဆေးမှုများ**

   ```bash
   openclaw status --deep
   ```

   Runs gateway health checks + provider probes (requires a reachable gateway). See [Health](/gateway/health).

5. **နောက်ဆုံး log ကို tail လုပ်ရန်**

   ```bash
   openclaw logs --follow
   ```

   RPC မရပါက အစားထိုးအသုံးပြုပါ—

   ```bash
   tail -f "$(ls -t /tmp/openclaw/openclaw-*.log | head -1)"
   ```

   File logs သည် service logs နှင့် သီးခြားဖြစ်သည်; [Logging](/logging) နှင့် [Troubleshooting](/gateway/troubleshooting) ကို ကြည့်ပါ။

6. **Doctor ကို run လုပ်ရန် (ပြုပြင်ခြင်း)**

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

## Quick start and first-run setup

### Im stuck whats the fastest way to get unstuck

သင့်စက်ကို **တိုက်ရိုက်မြင်နိုင်သော** local AI agent တစ်ခုကို အသုံးပြုပါ။ Discord မှာ မေးတာထက် ဒီနည်းလမ်းက ပိုထိရောက်ပါတယ်၊ ဘာကြောင့်လဲဆိုတော့ “I'm stuck” ပြဿနာအများစုက **local config သို့မဟုတ် environment ပြဿနာများ** ဖြစ်ပြီး remote helpers များက စစ်ဆေးလို့ မရနိုင်ပါ။

- **Claude Code**: https://www.anthropic.com/claude-code/
- **OpenAI Codex**: https://openai.com/codex/

ဒီ tools များက repo ကို ဖတ်နိုင်ပြီး command များကို chạy လုပ်နိုင်သလို logs များကို စစ်ဆေးပြီး သင့်စက်အဆင့် setup (PATH, services, permissions, auth files) ကို ပြုပြင်ရန် ကူညီနိုင်ပါသည်။ hackable (git) install ဖြင့် **full source checkout** ကို ပေးပါ:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash -s -- --install-method git
```

ဒါက OpenClaw ကို **git checkout မှ** install လုပ်ပြီး agent ကို သင် chạy နေသော version အတိအကျကို ဖတ်ရှုနိုင်စေပါသည်။ နောက်မှ `--install-method git` မပါဘဲ installer ကို ပြန် chạy လုပ်ပြီး stable သို့ ပြန်ပြောင်းနိုင်ပါသည်။

Tip: ပြုပြင်မှုကို **plan and supervise** (အဆင့်လိုက် စီစဉ်ပြီး စောင့်ကြည့်စစ်ဆေး) ခိုင်းစေပြီး လိုအပ်သော command များကိုသာ chạy လုပ်ပါ။ ဒါက ပြောင်းလဲမှုများကို သေးငယ်စေပြီး audit လုပ်ရန် လွယ်ကူစေပါသည်။

တကယ့် bug သို့မဟုတ် fix ကို တွေ့ရှိပါက GitHub issue တင်ပါ သို့မဟုတ် PR ပို့ပါ:
https://github.com/openclaw/openclaw/issues
https://github.com/openclaw/openclaw/pulls

အကူအညီတောင်းရာတွင် အောက်ပါ command များဖြင့် စတင်ပြီး output များကို မျှဝေပါ—

```bash
openclaw status
openclaw models status
openclaw doctor
```

၎င်းတို့၏ လုပ်ဆောင်ချက်များ—

- `openclaw status`: gateway/agent health + အခြေခံ config ကို အမြန် snapshot ရယူသည်။
- `openclaw models status`: provider auth + model ရရှိနိုင်မှုကို စစ်ဆေးသည်။
- `openclaw doctor`: အများဆုံးတွေ့ရသော config/state ပြဿနာများကို စစ်ဆေးပြီး ပြုပြင်သည်။

အသုံးဝင်သော CLI စစ်ဆေးချက်များ: `openclaw status --all`, `openclaw logs --follow`, `openclaw gateway status`, `openclaw health --verbose`။

Quick debug loop: [First 60 seconds if something's broken](#first-60-seconds-if-somethings-broken)။
Install docs: [Install](/install), [Installer flags](/install/installer), [Updating](/install/updating)။

### What's the recommended way to install and set up OpenClaw

Repo က source မှ chạy လုပ်ပြီး onboarding wizard ကို အသုံးပြုရန် အကြံပြုထားပါသည်:

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
openclaw onboard --install-daemon
```

Wizard က UI assets များကိုလည်း အလိုအလျောက် build လုပ်ပေးနိုင်ပါသည်။ onboarding ပြီးသွားပြီးနောက် Gateway ကို ပုံမှန်အားဖြင့် port **18789** ပေါ်တွင် chạy လုပ်ပါလိမ့်မည်။

Source မှ (contributors/dev များအတွက်):

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm build
pnpm ui:build # auto-installs UI deps on first run
openclaw onboard
```

Global install မရှိသေးပါက `pnpm openclaw onboard` ဖြင့် chạy လုပ်နိုင်ပါသည်။