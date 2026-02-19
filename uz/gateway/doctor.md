---
summary: "Doctor buyrug‘i: tizim holatini tekshirish, konfiguratsiya migratsiyalari va tuzatish bosqichlari"
read_when:
  - Adding or modifying doctor migrations
  - 1. Konfiguratsiyadagi buzuvchi o‘zgarishlarni joriy etish
title: "Doctor"
---

# Doctor

`openclaw doctor` — OpenClaw uchun ta’mirlash va migratsiya vositasi. U eskirgan
konfigurat­siya/holatni tuzatadi, sog‘liqni tekshiradi va aniq ta’mirlash qadamlarini taqdim etadi.

## Tezkor boshlash

```bash
openclaw doctor
```

### Headless / avtomatlashtirish

```bash
openclaw doctor --yes
```

So‘rovsiz standartlarni qabul qilish (zarur bo‘lganda qayta ishga tushirish/xizmat/sandbox ta’mirlash qadamlarini ham).

```bash
openclaw doctor --repair
```

Tavsiya etilgan ta’mirlashlarni so‘rovsiz qo‘llash (xavfsiz bo‘lsa, ta’mirlashlar + qayta ishga tushirishlar).

```bash
openclaw doctor --repair --force
```

Agressiv ta’mirlashlarni ham qo‘llash (maxsus supervisor konfiguratsiyalarini ustiga yozadi).

```bash
openclaw doctor --non-interactive
```

So‘rovlarsiz ishga tushadi va faqat xavfsiz migratsiyalarni qo‘llaydi (konfiguratsiyani normallashtirish + diskdagi holatni ko‘chirish). Inson tasdig‘ini talab qiladigan qayta ishga tushirish/xizmat/sandbox amallarini o‘tkazib yuboradi.
Aniqlanganda meros holat migratsiyalari avtomatik bajariladi.

```bash
openclaw doctor --deep
```

Qo‘shimcha gateway o‘rnatmalarini aniqlash uchun tizim xizmatlarini skanerlash (launchd/systemd/schtasks).

Yozishdan oldin o‘zgarishlarni ko‘rib chiqmoqchi bo‘lsangiz, avval konfiguratsiya faylini oching:

```bash
cat ~/.openclaw/openclaw.json
```

## Nima qiladi (qisqacha)

- Git o‘rnatmalar uchun ixtiyoriy pre-flight yangilash (faqat interaktiv).
- UI protokoli yangiligi tekshiruvi (protokol sxemasi yangiroq bo‘lsa Control UI’ni qayta yig‘adi).
- Sog‘liqni tekshirish + qayta ishga tushirish taklifi.
- Skills holati xulosasi (mos/yo‘q/blocked).
- Meros qiymatlar uchun konfiguratsiyani normallashtirish.
- OpenCode Zen provayderi override ogohlantirishlari (`models.providers.opencode`).
- Diskdagi meros holat migratsiyasi (sessions/agent dir/WhatsApp auth).
- Holat yaxlitligi va ruxsatlar tekshiruvi (sessions, transcripts, state dir).
- Mahalliy ishga tushirilganda konfiguratsiya fayli ruxsatlarini tekshirish (chmod 600).
- Model autentifikatsiyasi sog‘ligi: OAuth muddati tugashini tekshiradi, muddati yaqin tokenlarni yangilashi mumkin va auth-profil cooldown/o‘chirilgan holatlarni hisobot qiladi.
- Qo‘shimcha workspace katalogini aniqlash (`~/openclaw`).
- Sandbox yoqilganida sandbox tasvirini ta’mirlash.
- Meros xizmat migratsiyasi va qo‘shimcha gateway aniqlash.
- Gateway runtime tekshiruvlari (xizmat o‘rnatilgan, ammo ishlamayapti; keshlangan launchd label).
- Kanal holati bo‘yicha ogohlantirishlar (ishlayotgan gateway’dan prob qilingan).
- Supervisor konfiguratsiyasi auditi (launchd/systemd/schtasks) va ixtiyoriy ta’mirlash.
- Gateway runtime eng yaxshi amaliyot tekshiruvlari (Node vs Bun, versiya-menejer yo‘llari).
- Gateway port to‘qnashuvi diagnostikasi (standart `18789`).
- Ochiq DM siyosatlari uchun xavfsizlik ogohlantirishlari.
- `gateway.auth.token` o‘rnatilmaganida gateway autentifikatsiya ogohlantirishlari (mahalliy rejim; token yaratishni taklif qiladi).
- Linux’da systemd linger tekshiruvi.
- Manbadan o‘rnatish tekshiruvlari (pnpm workspace mos kelmasligi, yetishmayotgan UI assetlar, yetishmayotgan tsx binari).
- Yangilangan konfiguratsiya + wizard metama’lumotlarini yozadi.

## Batafsil xatti-harakat va asoslantirish

### 0. Ixtiyoriy yangilash (git o‘rnatmalar)

Agar bu git checkout bo‘lsa va doctor interaktiv ishlayotgan bo‘lsa, u
update (fetch/rebase/build) ni doctor’ni ishga tushirishdan oldin taklif qiladi.

### 1. Konfiguratsiyani normallashtirish

Agar konfiguratsiyada eski qiymat shakllari mavjud bo‘lsa (masalan, `messages.ackReaction`
without a channel-specific override), doctor normalizes them into the current
schema.

### 2. Legacy config key migrations

Agar konfiguratsiyada eskirgan kalitlar mavjud bo‘lsa, boshqa buyruqlar ishga tushmaydi va foydalanuvchidan so‘raydi
you to run `openclaw doctor`.

Doctor quyidagilarni bajaradi:

- Qaysi eski kalitlar topilganini tushuntiradi.
- Qo‘llangan migratsiyani ko‘rsatadi.
- Yangilangan sxema bilan `~/.openclaw/openclaw.json` faylini qayta yozadi.

Gateway ham ishga tushirilganda, agar aniqlasa, doctor migratsiyalarini avtomatik ravishda ishga tushiradi
legacy config format, so stale configs are repaired without manual intervention.

Current migrations:

- `routing.allowFrom` → `channels.whatsapp.allowFrom`
- `routing.groupChat.requireMention` → `channels.whatsapp/telegram/imessage.groups."*".requireMention`
- `routing.groupChat.historyLimit` → `messages.groupChat.historyLimit`
- `routing.groupChat.mentionPatterns` → `messages.groupChat.mentionPatterns`
- `routing.queue` → `messages.queue`
- `routing.bindings` → top-level `bindings`
- `routing.agents`/`routing.defaultAgentId` → `agents.list` + `agents.list[].default`
- `routing.agentToAgent` → `tools.agentToAgent`
- `routing.transcribeAudio` → `tools.media.audio.models`
- `bindings[].match.accountID` → `bindings[].match.accountId`
- `identity` → `agents.list[].identity`
- `agent.*` → `agents.defaults` + `tools.*` (tools/elevated/exec/sandbox/subagents)
- `agent.model`/`allowedModels`/`modelAliases`/`modelFallbacks`/`imageModelFallbacks`
  → `agents.defaults.models` + `agents.defaults.model.primary/fallbacks` + `agents.defaults.imageModel.primary/fallbacks`

### 2b) OpenCode Zen provider overrides

If you’ve added `models.providers.opencode` (or `opencode-zen`) manually, it
overrides the built-in OpenCode Zen catalog from `@mariozechner/pi-ai`. That can
force every model onto a single API or zero out costs. Doctor warns so you can
remove the override and restore per-model API routing + costs.

### 3. Legacy state migrations (disk layout)

Doctor can migrate older on-disk layouts into the current structure:

- Sessions store + transcripts:
  - from `~/.openclaw/sessions/` to `~/.openclaw/agents/<agentId>/sessions/`
- Agent dir:
  - from `~/.openclaw/agent/` to `~/.openclaw/agents/<agentId>/agent/`
- WhatsApp auth state (Baileys):
  - from legacy `~/.openclaw/credentials/*.json` (except `oauth.json`)
  - to `~/.openclaw/credentials/whatsapp/<accountId>/...` (default account id: `default`)

These migrations are best-effort and idempotent; doctor will emit warnings when
it leaves any legacy folders behind as backups. The Gateway/CLI also auto-migrates
the legacy sessions + agent dir on startup so history/auth/models land in the
per-agent path without a manual doctor run. WhatsApp auth is intentionally only
migrated via `openclaw doctor`.

### 4. State integrity checks (session persistence, routing, and safety)

The state directory is the operational brainstem. If it vanishes, you lose
sessions, credentials, logs, and config (unless you have backups elsewhere).

Doctor checks:

- **State dir missing**: warns about catastrophic state loss, prompts to recreate
  the directory, and reminds you that it cannot recover missing data.
- **State dir permissions**: verifies writability; offers to repair permissions
  (and emits a `chown` hint when owner/group mismatch is detected).
- **Session dirs missing**: `sessions/` and the session store directory are
  required to persist history and avoid `ENOENT` crashes.
- **Transcript mismatch**: warns when recent session entries have missing
  transcript files.
- **Main session “1-line JSONL”**: flags when the main transcript has only one
  line (history is not accumulating).
- **Multiple state dirs**: warns when multiple `~/.openclaw` folders exist across
  home directories or when `OPENCLAW_STATE_DIR` points elsewhere (history can
  split between installs).
- **Remote mode reminder**: if `gateway.mode=remote`, doctor reminds you to run
  it on the remote host (the state lives there).
- **Config file permissions**: warns if `~/.openclaw/openclaw.json` is
  group/world readable and offers to tighten to `600`.

### 5. Model auth health (OAuth expiry)

Doctor inspects OAuth profiles in the auth store, warns when tokens are
expiring/expired, and can refresh them when safe. If the Anthropic Claude Code
profile is stale, it suggests running `claude setup-token` (or pasting a setup-token).
Refresh prompts only appear when running interactively (TTY); `--non-interactive`
skips refresh attempts.

Doctor also reports auth profiles that are temporarily unusable due to:

- short cooldowns (rate limits/timeouts/auth failures)
- longer disables (billing/credit failures)

### 6. Hooks model validation

If `hooks.gmail.model` is set, doctor validates the model reference against the
catalog and allowlist and warns when it won’t resolve or is disallowed.

### 7. Sandbox image repair

When sandboxing is enabled, doctor checks Docker images and offers to build or
switch to legacy names if the current image is missing.

### 8. Gateway service migrations and cleanup hints

Doctor detects legacy gateway services (launchd/systemd/schtasks) and
offers to remove them and install the OpenClaw service using the current gateway
port. It can also scan for extra gateway-like services and print cleanup hints.
Profile-named OpenClaw gateway services are considered first-class and are not
flagged as "extra."

### 9. Security warnings

Doctor emits warnings when a provider is open to DMs without an allowlist, or
when a policy is configured in a dangerous way.

### 10. systemd linger (Linux)

If running as a systemd user service, doctor ensures lingering is enabled so the
gateway stays alive after logout.

### 11. Skills status

Doctor prints a quick summary of eligible/missing/blocked skills for the current
workspace.

### 12. Gateway auth checks (local token)

Doctor warns when `gateway.auth` is missing on a local gateway and offers to
generate a token. Use `openclaw doctor --generate-gateway-token` to force token
creation in automation.

### 13. Gateway health check + restart

Doctor runs a health check and offers to restart the gateway when it looks
unhealthy.

### 14. Channel status warnings

If the gateway is healthy, doctor runs a channel status probe and reports
warnings with suggested fixes.

### 15. Supervisor config audit + repair

Doctor checks the installed supervisor config (launchd/systemd/schtasks) for
missing or outdated defaults (e.g., systemd network-online dependencies and
restart delay). When it finds a mismatch, it recommends an update and can
rewrite the service file/task to the current defaults.

Notes:

- `openclaw doctor` prompts before rewriting supervisor config.
- `openclaw doctor --yes` accepts the default repair prompts.
- `openclaw doctor --repair` applies recommended fixes without prompts.
- `openclaw doctor --repair --force` overwrites custom supervisor configs.
- Siz har doim `openclaw gateway install --force` orqali to‘liq qayta o‘rnatishni majburlashingiz mumkin.

### 16. Gateway runtime + port diagnostics

Doctor inspects the service runtime (PID, last exit status) and warns when the
service is installed but not actually running. It also checks for port collisions
on the gateway port (default `18789`) and reports likely causes (gateway already
running, SSH tunnel).

### 17. Gateway runtime best practices

Doctor warns when the gateway service runs on Bun or a version-managed Node path
(`nvm`, `fnm`, `volta`, `asdf`, etc.). WhatsApp + Telegram channels require Node,
and version-manager paths can break after upgrades because the service does not
load your shell init. Doctor offers to migrate to a system Node install when
available (Homebrew/apt/choco).

### 18. Config write + wizard metadata

Doctor persists any config changes and stamps wizard metadata to record the
doctor run.

### 19. Workspace tips (backup + memory system)

Doctor suggests a workspace memory system when missing and prints a backup tip
if the workspace is not already under git.

See [/concepts/agent-workspace](/concepts/agent-workspace) for a full guide to
workspace structure and git backup (recommended private GitHub or GitLab).

