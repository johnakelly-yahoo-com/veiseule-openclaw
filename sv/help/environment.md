---
title: "Miljövariabler"
---

# Miljövariabler

OpenClaw drar miljövariabler från flera källor. Regeln är **aldrig åsidosätta befintliga värden**.

## Prioritet (högst → lägst)

1. **Processmiljö** (det som Gateway-processen (nätverksgateway) redan har från överordnat skal/daemon).
2. **`.env` i aktuell arbetskatalog** (dotenv-standard; skriver inte över).
3. **Global `.env`** på `~/.openclaw/.env` (även kallad `$OPENCLAW_STATE_DIR/.env`; skriver inte över).
4. **Konfig `env`-block** i `~/.openclaw/openclaw.json` (tillämpas endast om saknas).
5. **Valfri import från inloggningsskal** (`env.shellEnv.enabled` eller `OPENCLAW_LOAD_SHELL_ENV=1`), tillämpas endast för saknade förväntade nycklar.

Om konfigfilen saknas helt hoppas steg 4 över; skalimport körs fortfarande om den är aktiverad.

## Konfig `env`-block

Två likvärdiga sätt att sätta inbäddade miljövariabler (båda skriver inte över):

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

## Skalimport av miljövariabler

`env.shellEnv` kör ditt inloggningsskal och importerar endast **saknade** förväntade nycklar:

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

Motsvarande miljövariabler:

- `OPENCLAW_LOAD_SHELL_ENV=1`
- `OPENCLAW_SHELL_ENV_TIMEOUT_MS=15000`

## Ersättning av miljövariabler i konfig

Du kan referera till miljövariabler direkt i konfigens strängvärden med syntaxen `${VAR_NAME}`:

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

Se [Konfiguration: Ersättning av miljövariabler](/gateway/configuration#env-var-substitution-in-config) för fullständiga detaljer.

## Miljövariabler relaterade till sökvägar

| Variabel               | Syfte                                                                                                                                                                                                                               |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `OPENCLAW_HOME`        | Åsidosätt hemkatalogen som används för all intern sökvägsupplösning (`~/.openclaw/`, agentkataloger, sessioner, autentiseringsuppgifter). Useful when running OpenClaw as a dedicated service user. |
| `OPENCLAW_STATE_DIR`   | Åsidosätt tillståndskatalogen (standard `~/.openclaw`).                                                                                                                                            |
| `OPENCLAW_CONFIG_PATH` | Åsidosätt sökvägen till konfigurationsfilen (standard `~/.openclaw/openclaw.json`).                                                                                                                             |

### `OPENCLAW_HOME`

När den är satt ersätter `OPENCLAW_HOME` systemets hemkatalog (`$HOME` / `os.homedir()`) för all intern sökvägsupplösning. Detta möjliggör fullständig filsystemisolering för huvudlösa tjänstkonton.

**Prioritet:** `OPENCLAW_HOME` > `$HOME` > `USERPROFILE` > `os.homedir()`

**Exempel** (macOS LaunchDaemon):

```xml
<key>EnvironmentVariables</key>
<dict>
  <key>OPENCLAW_HOME</key>
  <string>/Users/kira</string>
</dict>
```

`OPENCLAW_HOME` kan också sättas till en tilde-sökväg (t.ex. `~/svc`), som expanderas med hjälp av `$HOME` innan den används.

## Relaterat

- [Gateway-konfiguration](/gateway/configuration)
- [Vanliga frågor: miljövariabler och .env-laddning](/help/faq#env-vars-and-env-loading)
- [Översikt över modeller](/concepts/models)
