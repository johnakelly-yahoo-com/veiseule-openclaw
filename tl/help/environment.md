---
summary: "Kung saan nilo-load ng OpenClaw ang mga environment variable at ang pagkakasunud-sunod ng precedence"
read_when:
  - Kailangan mong malaman kung aling mga env var ang nilo-load, at sa anong pagkakasunod-sunod
  - Nagde-debug ka ng mga nawawalang API key sa Gateway
  - Dinodokumento mo ang provider auth o mga deployment environment
title: "Mga Environment Variable"
---

# Mga environment variable

Kinukuha ng OpenClaw ang mga environment variable mula sa maraming pinagmulan. Ang patakaran ay **huwag kailanman i-override ang umiiral na mga value**.

## Precedence (pinakamataas → pinakamababa)

1. **Process environment** (kung ano ang mayroon na ang proseso ng Gateway mula sa parent shell/daemon).
2. **`.env` sa kasalukuyang working directory** (dotenv default; hindi nag-o-override).
3. **Global `.env`** sa `~/.openclaw/.env` (aka `$OPENCLAW_STATE_DIR/.env`; hindi nag-o-override).
4. **Config `env` block** sa `~/.openclaw/openclaw.json` (ina-apply lamang kung kulang).
5. **Opsyonal na login-shell import** (`env.shellEnv.enabled` o `OPENCLAW_LOAD_SHELL_ENV=1`), ina-apply lamang para sa mga nawawalang inaasahang key.

Kung ganap na nawawala ang config file, nilalaktawan ang hakbang 4; tatakbo pa rin ang shell import kung naka-enable.

## Bloke ng `env` sa config

Dalawang magkaparehong paraan para magtakda ng inline env vars (parehong hindi nag-o-override):

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
  },
}
```

## Pag-import ng Shell env

Pinapatakbo ng `env.shellEnv` ang iyong login shell at ini-import lamang ang **nawawala** na inaasahang mga key:

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

Mga katumbas na env var:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Env var substitution sa config

Maaari kang mag-refer ng mga env var nang direkta sa mga string value ng config gamit ang `${VAR_NAME}` na syntax:

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
}
```

Tingnan ang [Configuration: Env var substitution](/gateway/configuration#env-var-substitution-in-config) para sa kumpletong detalye.

## Mga env var na may kinalaman sa path

| Baryabol               | Layunin                                                                                                                                                                                                                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `OPENCLAW_HOME`        | I-override ang home directory na ginagamit para sa lahat ng internal path resolution (`~/.openclaw/`, mga agent dir, session, credential). Kapaki-pakinabang kapag pinapatakbo ang OpenClaw bilang isang dedikadong service user. |
| `OPENCLAW_STATE_DIR`   | I-override ang state directory (default `~/.openclaw`).                                                                                                                                                                                           |
| `OPENCLAW_CONFIG_PATH` | I-override ang path ng config file (default `~/.openclaw/openclaw.json`).                                                                                                                                                                              |

### `OPENCLAW_HOME`

Kapag naka-set, pinapalitan ng `OPENCLAW_HOME` ang home directory ng system (`$HOME` / `os.homedir()`) para sa lahat ng internal na pagresolba ng path. Nagbibigay-daan ito sa ganap na paghihiwalay ng filesystem para sa mga headless service account.

**Precedence:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**Halimbawa** (macOS LaunchDaemon):

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

Maaari ring i-set ang `OPENCLAW_HOME` sa isang tilde path (hal. `~/svc`), na ine-expand gamit ang `$HOME` bago gamitin.

## Kaugnay

- [Konfigurasyon ng Gateway](/gateway/configuration)
- [FAQ: env vars at .env loading](/help/faq#env-vars-and-env-loading)
- [Pangkalahatang-ideya ng mga model](/concepts/models)
