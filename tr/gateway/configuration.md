---
summary: "Yapılandırma genel bakış: yaygın görevler, hızlı kurulum ve tam referansa bağlantılar"
read_when:
  - OpenClaw’u ilk kez kurma
  - Yaygın yapılandırma desenlerini arama
  - Belirli yapılandırma bölümlerine gitme
title: "Yapılandırma"
---

# Yapılandırma 🔧

OpenClaw, `~/.openclaw/openclaw.json` konumundan isteğe bağlı bir **JSON5** yapılandırması okur (yorumlar + sonda virgül serbesttir).

Dosya eksikse, OpenClaw güvenli varsayılanları kullanır. Yapılandırma eklemek için yaygın nedenler:

- Kanalları bağlayın ve bot’a kimlerin mesaj gönderebileceğini kontrol edin
- Modelleri, araçları, sandboxing’i veya otomasyonu (cron, hook’lar) ayarlayın
- Oturumları, medyayı, ağ ayarlarını veya UI’ı ince ayarlayın

Mevcut tüm alanlar için [tam referansa](/gateway/configuration-reference) bakın.

<Tip>
`openclaw --profile <name> …` → `~/.openclaw-<name>` kullanır (port yapılandırma/env/bayraklar üzerinden)
</Tip>

## Minimal yapılandırma (önerilen başlangıç noktası)

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## Yapılandırmayı düzenleme

<Tabs>
  <Tab title="Interactive wizard">```bash
openclaw onboard       # tam kurulum sihirbazı
openclaw configure     # yapılandırma sihirbazı
```
</Tab>
  <Tab title="CLI (one-liners)">```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```
</Tab>
  <Tab title="Control UI">
    Gateway, UI düzenleyiciler için yapılandırmanın JSON Şema temsilimini `config.schema` üzerinden sunar.
    Control UI, bu şemadan bir form üretir; kaçış yolu olarak **Raw JSON** düzenleyicisi de vardır.
  
</Tab>
  <Tab title="Direct edit">
    `~/.openclaw/openclaw.json` (veya `OPENCLAW_CONFIG_PATH`) Gateway dosyayı izler ve değişiklikleri otomatik olarak uygular (bkz. [hot reload](#config-hot-reload)).
  
</Tab>
</Tabs>

## Sıkı doğrulama

<Warning>
OpenClaw yalnızca şemayla **tam olarak** eşleşen yapılandırmaları kabul eder. Bilinmeyen anahtarlar, hatalı türler veya geçersiz değerler, güvenlik için Gateway’nin **başlamayı reddetmesine** neden olur. Tek kök seviye istisna `$schema`’dır (string); böylece editörler JSON Schema meta verilerini ekleyebilir.
</Warning>

Doğrulama başarısız olduğunda:

- Gateway açılmaz.
- Yalnızca tanılama komutlarına izin verilir (örneğin: `openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`, `openclaw service`, `openclaw help`).
- Tam sorunları görmek için `openclaw doctor` çalıştırın.
- Geçişleri/onarımı uygulamak için `openclaw doctor --fix` (veya `--yes`) çalıştırın.

## Yaygın görevler

<AccordionGroup>
  <Accordion title="Set up a channel (WhatsApp, Telegram, Discord, etc.)">Her kanalın `channels.` altında kendi yapılandırma bölümü vardır.<provider>` altında yer alır. Kurulum adımları için ilgili kanalın özel sayfasına bakın:

    ```
    {
      channels: {
        whatsapp: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        telegram: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["tg:123456789", "@alice"],
        },
        signal: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["+15551234567"],
        },
        imessage: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["chat_id:123"],
        },
        msteams: {
          groupPolicy: "allowlist",
          groupAllowFrom: ["user@org.com"],
        },
        discord: {
          groupPolicy: "allowlist",
          guilds: {
            GUILD_ID: {
              channels: { help: { allow: true } },
            },
          },
        },
        slack: {
          groupPolicy: "allowlist",
          channels: { "#general": { allow: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Choose and configure models">Birincil modeli ve isteğe bağlı yedekleri ayarlayın:

    ```
    {
      agents: {
        defaults: {
          cliBackends: {
            "claude-cli": {
              command: "/opt/homebrew/bin/claude",
            },
            "my-cli": {
              command: "my-cli",
              args: ["--json"],
              output: "json",
              modelArg: "--model",
              sessionArg: "--session",
              sessionMode: "existing",
              systemPromptArg: "--system",
              systemPromptWhen: "first",
              imageArg: "--image",
              imageMode: "repeat",
            },
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Control who can message the bot">DM erişimi kanal başına `dmPolicy` ile kontrol edilir:

    ```
    Varsayılan `groupPolicy: "allowlist"`’dir (`channels.defaults.groupPolicy` ile geçersiz kılınmadıkça); izin listesi yapılandırılmamışsa grup mesajları engellenir.
    ```

  
</Accordion>

  <Accordion title="Set up group chat mention gating">
    Grup mesajları varsayılan olarak **bahsetme gerektirir** (meta veri bahsetmesi veya regex desenleri). `agents.defaults.subagents` configures sub-agent defaults:

    ```
    {
      agents: {
        defaults: { workspace: "~/.openclaw/workspace" },
        list: [
          {
            id: "main",
            groupChat: { mentionPatterns: ["@openclaw", "reisponde"] },
          },
        ],
      },
      channels: {
        whatsapp: {
          // Allowlist is DMs only; including your own number enables self-chat mode.
          allowFrom: ["+15555550123"],
          groups: { "*": { requireMention: true } },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure sessions and resets">Oturumlar, konuşma sürekliliğini ve izolasyonunu kontrol eder:

    ````
    ```json5
    {
      session: {
        dmScope: "per-channel-peer",  // çok kullanıcılı için önerilir
        reset: {
          mode: "daily",
          atHour: 4,
          idleMinutes: 120,
        },
      },
    }
    ```
    
    - `dmScope`: `main` (paylaşılan) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
    - Kapsam, kimlik bağlantıları ve gönderim politikası için [Session Management](/concepts/session) sayfasına bakın.
    - Tüm alanlar için [tam referansa](/gateway/configuration-reference#session) bakın.
    ````

  
</Accordion>

  <Accordion title="Enable sandboxing">Ajan oturumlarını izole Docker konteynerlerinde çalıştırın:

    ```
    // ~/.openclaw/agents.json5
    {
      defaults: { sandbox: { mode: "all", scope: "session" } },
      list: [{ id: "main", workspace: "~/.openclaw/workspace" }],
    }
    ```

  
</Accordion>

  <Accordion title="Set up heartbeat (periodic check-ins)">```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

    ```
    - `every`: süre dizesi (`30m`, `2h`). Devre dışı bırakmak için `0m` ayarlayın.
    - `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
    - Tam kılavuz için [Heartbeat](/gateway/heartbeat) sayfasına bakın.
    ```

  
</Accordion>

  <Accordion title="Configure cron jobs">{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
  },
}

    ```
    See [Cron jobs](/automation/cron-jobs) for the feature overview and CLI examples.
    ```

  
</Accordion>

  <Accordion title="Set up webhooks (hooks)">Gateway HTTP sunucusunda basit bir HTTP webhook uç noktası etkinleştirin.

    ```
    {
      hooks: {
        enabled: true,
        token: "shared-secret",
        path: "/hooks",
        presets: ["gmail"],
        transformsDir: "~/.openclaw/hooks",
        mappings: [
          {
            match: { path: "gmail" },
            action: "agent",
            wakeMode: "now",
            name: "Gmail",
            sessionKey: "hook:gmail:{{messages[0].id}}",
            messageTemplate: "From: {{messages[0].from}}\nSubject: {{messages[0].subject}}\n{{messages[0].snippet}}",
            deliver: true,
            channel: "last",
            model: "openai/gpt-5.2-mini",
          },
        ],
      },
    }
    ```

  
</Accordion>

  <Accordion title="Configure multi-agent routing">Ayrı çalışma alanları ve oturumlarla birden fazla izole ajan çalıştırın:

    ```
    {
      agents: {
        list: [
          { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
          { id: "work", workspace: "~/.openclaw/workspace-work" },
        ],
      },
      bindings: [
        { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
        { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
      ],
      channels: {
        whatsapp: {
          accounts: {
            personal: {},
            biz: {},
          },
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Split config into multiple files ($include)">Yapılandırmanızı `$include` yönergesiyle birden fazla dosyaya bölün. Şunlar için kullanışlıdır:

    ```
    // ~/.openclaw/openclaw.json
    {
      gateway: { port: 18789 },
    
      // Include a single file (replaces the key's value)
      agents: { $include: "./agents.json5" },
    
      // Include multiple files (deep-merged in order)
      broadcast: {
        $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

## `gateway.reload` (Config hot reload)

The Gateway watches `~/.openclaw/openclaw.json` (or `OPENCLAW_CONFIG_PATH`) and applies changes automatically.

### Yeniden yükleme modları

| Modes:                       | Birleştirme davranışı                                                                                                                                                                                 |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`hybrid`** (varsayılan) | Güvenli değişiklikleri anında hot olarak uygular. Kritik olanlar için otomatik olarak yeniden başlatır.                                                               |
| **`hot`**                                    | `hot`: only apply hot-safe changes; log when a restart is required. Yeniden başlatma gerektiğinde bir uyarı günlüğe kaydeder — işlemi siz yaparsınız. |
| **`restart`**                                | `restart`: restart the Gateway on any config change.                                                                                                                  |
| **Satır içi ikame:**         | Dosya izlemeyi devre dışı bırakır. Değişiklikler bir sonraki manuel yeniden başlatmada etkili olur.                                                                   |

```json5
{
  gateway: {
    reload: {
      mode: "hybrid",
      debounceMs: 300,
    },
  },
}
```

### Hangi değişiklikler anında uygulanır, hangileri yeniden başlatma gerektirir

Çoğu alan kesinti olmadan anında uygulanır. `hybrid` modunda, yeniden başlatma gerektiren değişiklikler otomatik olarak ele alınır.

| Kategori                             | Alanlar:                                                              | Yeniden başlatma gerekli mi?             |
| ------------------------------------ | ------------------------------------------------------------------------------------- | ---------------------------------------- |
| \`channels.<channel> | `channels.*`, `web` (WhatsApp) — tüm yerleşik ve eklenti kanalları | Hayır                                    |
| Ajan ve modeller                     | `agent`, `agents`, `models`, `routing`                                                | Hayır                                    |
| Otomasyon                            | `hooks`, `cron`, `agent.heartbeat`                                                    | Hayır                                    |
| messages                             | `messages.queue`                                                                      | Hayır                                    |
| Araçlar ve medya                     | `tools`, `browser`, `skills`, `audio`, `talk`                                         | Hayır                                    |
| Arayüz ve diğer                      | `ui`, `logging`, `identity`, `bindings`                                               | Hayır                                    |
| Gateway sunucusu                     | `gateway` (port/bind/auth/kontrol UI/tailscale)                    | **Kurallar:**            |
| Altyapı                              | `discovery`, `canvasHost`, `plugins`                                                  | **Inline substitution:** |

<Note>
`gateway.reload` ve `gateway.remote` istisnadır — bunların değiştirilmesi **yeniden başlatma** tetiklemez.
</Note>

## Kısmi güncellemeler (RPC)

<AccordionGroup>
  <Accordion title="config.apply (full replace)">`config.apply` kullanarak tüm yapılandırmayı tek adımda doğrulayın + yazın ve Gateway’yi yeniden başlatın.

    ```
    openclaw gateway call config.get --params '{}' # capture payload.hash
    openclaw gateway call config.apply --params '{
      "raw": "{\\n  agents: { defaults: { workspace: \\"~/.openclaw/workspace\\" } }\\n}\\n",
      "baseHash": "<hash-from-config.get>",
      "sessionKey": "agent:main:whatsapp:dm:+15555550123",
      "restartDelayMs": 1000
    }'
    ```

  
</Accordion>

  <Accordion title="config.patch (partial update)">`config.patch` kullanarak, ilişkili olmayan anahtarları ezmeden mevcut yapılandırmaya kısmi bir güncelleme birleştirin. JSON merge patch anlambilimini uygular:

    ````
    - Nesneler özyinelemeli olarak birleştirilir
    - `null` bir anahtarı siler
    - Diziler tamamen değiştirilir
    
    Parametreler:
    
    - `raw` (string) — yalnızca değiştirilecek anahtarları içeren JSON5
    - `baseHash` (zorunlu) — `config.get` komutundan alınan yapılandırma özeti
    - `sessionKey`, `note`, `restartDelayMs` — `config.apply` ile aynı
    
    ```bash
    openclaw gateway call config.patch --params '{
      "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
      "baseHash": "<hash>"
    }'
    ```
    
    ````

  
</Accordion>
</AccordionGroup>

## Ortam değişkenleri

OpenClaw, üst süreçten gelen ortam değişkenlerini ve ayrıca şunları okur:

- mevcut çalışma dizininden `.env` (varsa)
- `~/.openclaw/.env`’den küresel bir yedek `.env` (diğer adıyla `$OPENCLAW_STATE_DIR/.env`)

Bu `.env` dosyalarının hiçbiri mevcut ortam değişkenlerini geçersiz kılmaz. Ayrıca yapılandırmada satır içi ortam değişkenleri de ayarlayabilirsiniz:

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

<Accordion title="Shell env import (optional)">Kolaylık için isteğe bağlıdır: etkinleştirilirse ve beklenen anahtarların hiçbiri henüz ayarlanmamışsa, OpenClaw oturum açma kabuğunuzu çalıştırır ve yalnızca eksik beklenen anahtarları içe aktarır (asla geçersiz kılmaz). Bu, kabuk profilinizin kaynaklanmasıyla eşdeğerdir.

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

`OPENCLAW_LOAD_SHELL_ENV=1`

<Accordion title="Env var substitution in config values">Herhangi bir yapılandırma dizesi değerinde ortam değişkenlerine doğrudan `${VAR_NAME}` sözdizimiyle başvurabilirsiniz.

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}",
      },
    },
  },
  gateway: {
    auth: {
      token: "${OPENCLAW_GATEWAY_TOKEN}",
    },
  },
}
```

Kurallar:

- Yalnızca BÜYÜK HARF ortam değişkeni adları eşleşir: `[A-Z_][A-Z0-9_]*`
- Missing or empty env vars throw an error at config load
- `$${VAR}` ile kaçırarak değişmez `${VAR}` yazdırın
- `$include` ile çalışır (dahil edilen dosyalar da ikame alır)
- Satır içi yer değiştirme: `"${BASE}/v1"` → `"https://api.example.com/v1"`

</Accordion>

Tam öncelik ve kaynaklar için [/environment](/help/environment) sayfasına bakın.

## Tam referans

Aracının dosya işlemleri için kullandığı **tek küresel çalışma alanı dizinini** ayarlar.

---

**Yapılandırmaya yeni misiniz?** Ayrıntılı açıklamalarla eksiksiz örnekler için [Configuration Examples](/gateway/configuration-examples) kılavuzuna göz atın!
