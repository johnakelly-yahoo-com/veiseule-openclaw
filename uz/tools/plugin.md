---
summary: "OpenClaw plaginlari/kengaytmalari: kashf etish, sozlash va xavfsizlik"
read_when:
  - Adding or modifying plugins/extensions
  - Documenting plugin install or load rules
title: "Plaginlar"
---

# Plaginlar (Kengaytmalar)

## Tez boshlash (plaginlarga yangi misiz?)

Plagin — bu OpenClaw’ni qo‘shimcha imkoniyatlar bilan kengaytiradigan **kichik kod moduli** xolos
features (commands, tools, and Gateway RPC).

Ko‘p hollarda, o‘rnatilgan funksiyalar orasida mavjud bo‘lmagan imkoniyat kerak bo‘lganda plaginlardan foydalanasiz
into core OpenClaw yet (or you want to keep optional features out of your main
install).

Tezkor yo‘l:

1. Allaqachon yuklanganlarni ko‘ring:

```bash
openclaw plugins list
```

2. Rasmiy plaginni o‘rnating (misol: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

Npm spetsifikatsiyalari **faqat registry** uchun (paket nomi + ixtiyoriy versiya/tag). Git/URL/file
spetsifikatsiyalari rad etiladi.

3. Restart the Gateway, then configure under `plugins.entries.<id>.config`.

See [Voice Call](/plugins/voice-call) for a concrete example plugin.

## Available plugins (official)

- Microsoft Teams is plugin-only as of 2026.1.15; install `@openclaw/msteams` if you use Teams.
- Memory (Core) — bundled memory search plugin (enabled by default via `plugins.slots.memory`)
- Memory (LanceDB) — bundled long-term memory plugin (auto-recall/capture; set `plugins.slots.memory = "memory-lancedb"`)
- [Voice Call](/plugins/voice-call) — `@openclaw/voice-call`
- [Zalo Personal](/plugins/zalouser) — `@openclaw/zalouser`
- [Matrix](/channels/matrix) — `@openclaw/matrix`
- [Nostr](/channels/nostr) — `@openclaw/nostr`
- [Zalo](/channels/zalo) — `@openclaw/zalo`
- [Microsoft Teams](/channels/msteams) — `@openclaw/msteams`
- Google Antigravity OAuth (provider auth) — bundled as `google-antigravity-auth` (disabled by default)
- Gemini CLI OAuth (provider auth) — bundled as `google-gemini-cli-auth` (disabled by default)
- Qwen OAuth (provider auth) — bundled as `qwen-portal-auth` (disabled by default)
- Copilot Proxy (provider auth) — local VS Code Copilot Proxy bridge; distinct from built-in `github-copilot` device login (bundled, disabled by default)

OpenClaw plugins are **TypeScript modules** loaded at runtime via jiti. **Config
validation does not execute plugin code**; it uses the plugin manifest and JSON
Schema instead. See [Plugin manifest](/plugins/manifest).

Plugins can register:

- Gateway RPC methods
- Gateway HTTP handlers
- Agent tools
- CLI commands
- Background services
- Optional config validation
- **Skills** (by listing `skills` directories in the plugin manifest)
- **Auto-reply commands** (execute without invoking the AI agent)

Plugins run **in‑process** with the Gateway, so treat them as trusted code.
Tool authoring guide: [Plugin agent tools](/plugins/agent-tools).

## Runtime helpers

Plugins can access selected core helpers via `api.runtime`. For telephony TTS:

```ts
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

Notes:

- Uses core `messages.tts` configuration (OpenAI or ElevenLabs).
- Returns PCM audio buffer + sample rate. Plugins must resample/encode for providers.
- Edge TTS is not supported for telephony.

## Discovery & precedence

OpenClaw scans, in order:

1. Config paths

- `plugins.load.paths` (file or directory)

2. Workspace extensions

- `<workspace>/.openclaw/extensions/*.ts`
- `<workspace>/.openclaw/extensions/*/index.ts`

3. Global extensions

- `~/.openclaw/extensions/*.ts`
- `~/.openclaw/extensions/*/index.ts`

4. Bundled extensions (shipped with OpenClaw, **disabled by default**)

- `<openclaw>/extensions/*`

Bundled plugins must be enabled explicitly via `plugins.entries.<id>.enabled`
or `openclaw plugins enable <id>`. Installed plugins are enabled by default,
but can be disabled the same way.

Each plugin must include a `openclaw.plugin.json` file in its root. If a path
points at a file, the plugin root is the file's directory and must contain the
manifest.

If multiple plugins resolve to the same id, the first match in the order above
wins and lower-precedence copies are ignored.

### Package packs

A plugin directory may include a `package.json` with `openclaw.extensions`:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

Each entry becomes a plugin. If the pack lists multiple extensions, the plugin id
becomes `name/<fileBase>`.

If your plugin imports npm deps, install them in that directory so
`node_modules` is available (`npm install` / `pnpm install`).

Xavfsizlik eslatmasi: `openclaw plugins install` plugin bog‘liqliklarini
`npm install --ignore-scripts` bilan o‘rnatadi (lifecycle scriptlarsiz). Plugin dependency
daraxtlarini "toza JS/TS" holatda saqlang va `postinstall` build talab qiladigan paketlardan saqlaning.

### Channel catalog metadata

Channel plugins can advertise onboarding metadata via `openclaw.channel` and
install hints via `openclaw.install`. This keeps the core catalog data-free.

Example:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Nextcloud Talk webhook botlari orqali self-hosted chat.",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClaw **tashqi kanal kataloglarini** ham birlashtira oladi (masalan, MPM reyestri eksporti). JSON faylini quyidagilardan biriga joylashtiring:

- `~/.openclaw/mpm/plugins.json`
- `~/.openclaw/mpm/catalog.json`
- `~/.openclaw/plugins/catalog.json`

Yoki `OPENCLAW_PLUGIN_CATALOG_PATHS` (yoki `OPENCLAW_MPM_CATALOG_PATHS`) ni bir yoki bir nechta JSON fayllarga yo‘naltiring (vergul/nuqta-vergul/`PATH` bilan ajratilgan). Har bir fayl  \`{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...}

## } } ] }\` ni o‘z ichiga olishi kerak.

Alohida fayl: faylning asosiy nomi (`~/.../voice-call.ts` → `voice-call`)

- Standart plagin id’lari:
- Paket to‘plamlari: `package.json` dagi `name`

Alohida fayl: faylning asosiy nomi (`~/.../voice-call.ts` → `voice-call`)

## Agar plagin `id` ni eksport qilsa, OpenClaw undan foydalanadi, ammo u sozlangan id bilan mos kelmasa, ogohlantiradi.

```json5
Konfiguratsiya
```

: har bir plagin uchun yoqish/o‘chirish + konfiguratsiya

- Maydonlar:
- `enabled`: asosiy yoqib/o‘chirish tugmasi (standart: true)
- `allow`: ruxsat etilganlar ro‘yxati (ixtiyoriy)
- `deny`: taqiqlanganlar ro‘yxati (ixtiyoriy; taqiq ustun turadi)
- `load.paths`: qo‘shimcha plagin fayllari/kataloglari`entries.<id>`

: har bir plagin uchun yoqish/o‘chirish + konfiguratsiya

Konfiguratsiya o‘zgarishlari **gateway qayta ishga tushirilishini talab qiladi**.

- Tekshiruv qoidalari (qat’iy):
- `entries`, `allow`, `deny` yoki `slots` ichidagi noma’lum plagin id’lari **xato** hisoblanadi.Noma’lum `channels.<id>`
- kalitlari **xato** hisoblanadi, agar plagin manifesti kanal id’sini e’lon qilmagan bo‘lsa.
- Plagin konfiguratsiyasi `openclaw.plugin.json` ichiga joylangan JSON Schema (`configSchema`) yordamida tekshiriladi.

## Agar plagin o‘chirilgan bo‘lsa, uning konfiguratsiyasi saqlanadi va **ogohlantirish** chiqariladi.

Plagin slotlari (eksklyuziv kategoriyalar) Ba’zi plagin kategoriyalari **eksklyuziv** (bir vaqtda faqat bittasi faol bo‘lishi mumkin).

```json5
Qaysi plagin slotga egalik qilishini tanlash uchun
```

`plugins.slots` dan foydalaning: {
plugins: {
slots: {
memory: "memory-core", // yoki xotira plaginlarini o‘chirish uchun "none"
},
},
}

## Agar bir nechta plagin `kind: "memory"` ni e’lon qilsa, faqat tanlangani yuklanadi.

Boshqalari diagnostika bilan o‘chiriladi.

Boshqaruv UI (sxema + yorliqlar)

- Boshqaruv UI yaxshiroq shakllarni chizish uchun `config.schema` (JSON Schema + `uiHints`) dan foydalanadi.OpenClaw aniqlangan plaginlarga asoslanib, ish vaqtida `uiHints` ni kengaytiradi:
- `plugins.entries.<id>` uchun har bir plagin bo‘yicha yorliqlar qo‘shadi`/`.enabled`/`.config\`Quyidagi ostida plagin tomonidan taqdim etilgan ixtiyoriy konfiguratsiya maydoni ko‘rsatmalarini birlashtiradi:

`plugins.entries.<id>`

Example:

```json
Misol:
```

## Agar plagin konfiguratsiya maydonlaringiz yaxshi yorliqlar/placeholder’lar bilan ko‘rinishini (va sirlarni maxfiy deb belgilashni) istasangiz, plagin manifestida JSON Schema bilan birga `uiHints` ni taqdim eting.

```bash
Misol:
```

`plugins update` only works for npm installs tracked under `plugins.installs`.

Plugins may also register their own top‑level commands (example: `openclaw voicecall`).

## Plugin API (overview)

Plugins export either:

- A function: `(api) => { ... }`
- An object: `{ id, name, configSchema, register(api) { ... } }`

## Plugin hooks

Plugins can ship hooks and register them at runtime. This lets a plugin bundle
event-driven automation without a separate hook pack install.

### Example

```
import { registerPluginHooksFromDir } from "openclaw/plugin-sdk";

export default function register(api) {
  registerPluginHooksFromDir(api, "./hooks");
}
```

Notes:

- Hook directories follow the normal hook structure (`HOOK.md` + `handler.ts`).
- Hook eligibility rules still apply (OS/bins/env/config requirements).
- Plugin-managed hooks show up in `openclaw hooks list` with `plugin:<id>`.
- You cannot enable/disable plugin-managed hooks via `openclaw hooks`; enable/disable the plugin instead.

## Provider plugins (model auth)

Plugins can register **model provider auth** flows so users can run OAuth or
API-key setup inside OpenClaw (no external scripts needed).

Register a provider via `api.registerProvider(...)`. Each provider exposes one
or more auth methods (OAuth, API key, device code, etc.). These methods power:

- `openclaw models auth login --provider <id> [--method <id>]`

Example:

```ts
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // Run OAuth flow and return auth profiles.
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

Notes:

- `run` receives a `ProviderAuthContext` with `prompter`, `runtime`,
  `openUrl`, and `oauth.createVpsAwareHandlers` helpers.
- Return `configPatch` when you need to add default models or provider config.
- Return `defaultModel` so `--set-default` can update agent defaults.

### Register a messaging channel

Plugins can register **channel plugins** that behave like built‑in channels
(WhatsApp, Telegram, etc.). Channel config lives under `channels.<id>` and is
validated by your channel plugin code.

```ts
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "demo channel plugin.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async () => ({ ok: true }),
  },
};

export default function (api) {
  api.registerChannel({ plugin: myChannel });
}
```

Notes:

- Put config under `channels.<id>` (not `plugins.entries`).
- `meta.label` is used for labels in CLI/UI lists.
- `meta.aliases` adds alternate ids for normalization and CLI inputs.
- `meta.preferOver` lists channel ids to skip auto-enable when both are configured.
- `meta.detailLabel` and `meta.systemImage` let UIs show richer channel labels/icons.

### Write a new messaging channel (step‑by‑step)

Buni **yangi chat yuzasi** ("messaging channel") kerak bo‘lganda ishlating, model provider emas.
Model provider docs live under `/providers/*`.

1. Pick an id + config shape

- All channel config lives under `channels.<id>`.
- Prefer `channels.<id>.accounts.<accountId>` for multi‑account setups.

2. Majburiy adapterlarni amalga oshiring

- `meta.label`, `meta.selectionLabel`, `meta.docsPath`, `meta.blurb` CLI/UI roʻyxatlarini boshqaradi.
- `capabilities` (chat turlari, media, tarmoqlar va h.k.)
- `outbound.deliveryMode` + `outbound.sendText` (asosiy yuborish uchun)
- `meta.detailLabel` va `meta.systemImage` UIʼlar tomonidan tafsilot matni/ikonlar uchun ishlatiladi.

3. Kerak bo‘lganda ixtiyoriy adapterlarni qo‘shing

- `config.listAccountIds` + `config.resolveAccount`
- `capabilities` (chat turlari, media, tarmoqlar va h.k.)
- `actions` (xabar amallari), `commands` (mahalliy buyruq xatti-harakati)

4. Kanalni plaginingizda ro‘yxatdan o‘tkazing

- `setup` (ustoz), `security` (DM siyosati), `status` (sog‘liq/diagnostika)
- `gateway` (start/stop/login), `mentions`, `threading`, `streaming`
- `actions` (xabar amallari), `commands` (mahalliy buyruq xatti-harakati)

5. Kanalni plaginingizda ro‘yxatdan o‘tkazing

- `api.registerChannel({ plugin })`

Minimal kanal plagini (faqat outbound):

```json5
{
  channels: {
    acmechat: {
      accounts: {
        default: { token: "ACME_TOKEN", enabled: true },
      },
    },
  },
}
```

Minimal kanal plagini (faqat outbound):

```ts
const plugin = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docsPath: "/channels/acmechat",
    blurb: "AcmeChat messaging channel.",
    aliases: ["acme"],
  },
  capabilities: { chatTypes: ["direct"] },
  config: {
    listAccountIds: (cfg) => Object.keys(cfg.channels?.acmechat?.accounts ?? {}),
    resolveAccount: (cfg, accountId) =>
      cfg.channels?.acmechat?.accounts?.[accountId ?? "default"] ?? {
        accountId,
      },
  },
  outbound: {
    deliveryMode: "direct",
    sendText: async ({ text }) => {
      // deliver `text` to your channel here
      return { ok: true };
    },
  },
};

export default function (api) {
  api.registerChannel({ plugin });
}
```

Plaginni yuklang (extensions katalogi yoki `plugins.load.paths`), gatewayʼni qayta ishga tushiring,
so‘ng `channels.<id>` ni sozlang\` konfiguratsiyangizda.

### Agent tools

Maxsus qo‘llanmaga qarang: [Plugin agent tools](/plugins/agent-tools).

### CLI buyruqlarini ro‘yxatdan o‘tkazing

```ts
export default function (api) {
  api.registerGatewayMethod("myplugin.status", ({ respond }) => {
    respond(true, { ok: true });
  });
}
```

### Avto-javob buyruqlarini ro‘yxatdan o‘tkazing

```ts
export default function (api) {
  api.registerCli(
    ({ program }) => {
      program.command("mycmd").action(() => {
        console.log("Hello");
      });
    },
    { commands: ["mycmd"] },
  );
}
```

### Avto-javob buyruqlarini ro‘yxatdan o‘tkazing

Plaginlar **AI agentini chaqirmasdan** bajariladigan maxsus slash-buyruqlarni ro‘yxatdan o‘tkazishi mumkin. Bu LLM ishloviga ehtiyoj bo‘lmagan yoqish/o‘chirish buyruqlari, holat tekshiruvlari yoki tezkor amallar uchun foydalidir.

```ts
export default function (api) {
  api.registerCommand({
    name: "mystatus",
    description: "Show plugin status",
    handler: (ctx) => ({
      text: `Plugin is running! Channel: ${ctx.channel}`,
    }),
  });
}
```

Buyruq parametrlari:

- `name`: Buyruq nomi (oldidagi `/`siz)
- `channel`: The channel where the command was sent
- `isAuthorizedSender`: Yuboruvchi vakolatli foydalanuvchi ekanligi
- `requireAuth`: Vakolatli yuboruvchini talab qilish (standart: true)
- `handler`: `{ text: string }` qaytaradigan funksiya (async bo‘lishi mumkin)
- `config`: Joriy OpenClaw konfiguratsiyasi

Avtorizatsiya va argumentlar bilan misol:

- `name`: Buyruq nomi (oldidagi `/`siz)
- `description`: Buyruqlar ro‘yxatida ko‘rsatiladigan yordam matni
- `acceptsArgs`: Buyruq argumentlarni qabul qiladimi (standart: false). Agar false bo‘lsa va argumentlar berilsa, buyruq mos kelmaydi va xabar boshqa ishlovchilarga o‘tadi
- `requireAuth`: Vakolatli yuboruvchini talab qilish (standart: true)
- `handler`: `{ text: string }` qaytaradigan funksiya (async bo‘lishi mumkin)

Eslatmalar:

```ts
api.registerCommand({
  name: "setmode",
  description: "Set plugin mode",
  acceptsArgs: true,
  requireAuth: true,
  handler: async (ctx) => {
    const mode = ctx.args?.trim() || "default";
    await saveMode(mode);
    return { text: `Mode set to: ${mode}` };
  },
});
```

Notes:

- Plagin buyruqlari **o‘rnatilgan** buyruqlar va AI agentdan **oldin** qayta ishlanadi
- Commands are registered globally and work across all channels
- Command names are case-insensitive (`/MyStatus` matches `/mystatus`)
- Command names must start with a letter and contain only letters, numbers, hyphens, and underscores
- Reserved command names (like `help`, `status`, `reset`, etc.) cannot be overridden by plugins
- Duplicate command registration across plugins will fail with a diagnostic error

### Register background services

```ts
export default function (api) {
  api.registerService({
    id: "my-service",
    start: () => api.logger.info("ready"),
    stop: () => api.logger.info("bye"),
  });
}
```

## Naming conventions

- Gateway methods: `pluginId.action` (example: `voicecall.status`)
- Tools: `snake_case` (example: `voice_call`)
- CLI commands: kebab or camel, but avoid clashing with core commands

## Skills

Plugins can ship a skill in the repo (`skills/<name>/SKILL.md`).
Enable it with `plugins.entries.<id>.enabled` (or other config gates) and ensure
it’s present in your workspace/managed skills locations.

## Distribution (npm)

Recommended packaging:

- Main package: `openclaw` (this repo)
- Plugins: separate npm packages under `@openclaw/*` (example: `@openclaw/voice-call`)

Publishing contract:

- Plugin `package.json` must include `openclaw.extensions` with one or more entry files.
- Entry files can be `.js` or `.ts` (jiti loads TS at runtime).
- `openclaw plugins install <npm-spec>` uses `npm pack`, extracts into `~/.openclaw/extensions/<id>/`, and enables it in config.
- Config key stability: scoped packages are normalized to the **unscoped** id for `plugins.entries.*`.

## Example plugin: Voice Call

This repo includes a voice‑call plugin (Twilio or log fallback):

- Source: `extensions/voice-call`
- Skill: `skills/voice-call`
- CLI: `openclaw voicecall start|status`
- Tool: `voice_call`
- RPC: `voicecall.start`, `voicecall.status`
- Config (twilio): `provider: "twilio"` + `twilio.accountSid/authToken/from` (optional `statusCallbackUrl`, `twimlUrl`)
- Config (dev): `provider: "log"` (no network)

See [Voice Call](/plugins/voice-call) and `extensions/voice-call/README.md` for setup and usage.

## Safety notes

Plugins run in-process with the Gateway. Treat them as trusted code:

- Only install plugins you trust.
- Prefer `plugins.allow` allowlists.
- Restart the Gateway after changes.

## Testing plugins

Plugins can (and should) ship tests:

- In-repo plugins can keep Vitest tests under `src/**` (example: `src/plugins/voice-call.plugin.test.ts`).
- Separately published plugins should run their own CI (lint/build/test) and validate `openclaw.extensions` points at the built entrypoint (`dist/index.js`).

