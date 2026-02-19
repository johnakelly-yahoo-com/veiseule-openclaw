---
title: "Yapılandırma Referansı"
description: "~/.openclaw/openclaw.json için alan bazında eksiksiz referans"
---

# Yapılandırma Referansı

`~/.openclaw/openclaw.json` içindeki tüm mevcut alanlar. Görev odaklı bir genel bakış için bkz. [Configuration](/gateway/configuration).

Yapılandırma formatı **JSON5**’tir (yorumlar + sondaki virgüllere izin verilir). Tüm alanlar isteğe bağlıdır — belirtilmediğinde OpenClaw güvenli varsayılanları kullanır.

---

## Kanallar

Her kanal, yapılandırma bölümü mevcut olduğunda (`enabled: false` değilse) otomatik olarak başlatılır.

### DM ve grup erişimi

Tüm kanallar DM politikalarını ve grup politikalarını destekler:

| DM politikası                             | Davranış                                                                                               |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| `pairing` (varsayılan) | Bilinmeyen gönderenler tek kullanımlık bir eşleştirme kodu alır; sahibin onaylaması gerekir            |
| `allowlist`                               | Yalnızca `allowFrom` içinde (veya eşleştirilmiş izin deposunda) bulunan gönderenler |
| `open`                                    | Tüm gelen DM’lere izin ver (şunu gerektirir: `allowFrom: ["*"]`)    |
| `disabled`                                | Tüm gelen DM’leri yok say                                                                              |

| Grup politikası                             | Davranış                                                                                |
| ------------------------------------------- | --------------------------------------------------------------------------------------- |
| `allowlist` (varsayılan) | Yalnızca yapılandırılmış allowlist ile eşleşen gruplar                                  |
| `open`                                      | Grup allowlist’lerini atla (bahsetme kısıtlaması yine de geçerlidir) |
| `disabled`                                  | Tüm grup/oda mesajlarını engelle                                                        |

<Note>
`channels.defaults.groupPolicy`, bir sağlayıcının `groupPolicy` değeri ayarlanmadığında varsayılanı belirler.
Eşleştirme kodlarının süresi 1 saat sonra dolar. Bekleyen DM eşleştirme istekleri **kanal başına 3** ile sınırlandırılmıştır.
Slack/Discord için özel bir geri dönüş mekanizması vardır: sağlayıcı bölümü tamamen eksikse, çalışma zamanı grup politikası `open` olarak çözümlenebilir (başlatma sırasında bir uyarı ile).
</Note>

### WhatsApp

WhatsApp, gateway'in web kanalı (Baileys Web) üzerinden çalışır. Bağlı bir oturum mevcut olduğunda otomatik olarak başlar.

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "pairing", // pairing | allowlist | open | disabled
      allowFrom: ["+15555550123", "+447700900123"],
      textChunkLimit: 4000,
      chunkMode: "length", // length | newline
      mediaMaxMb: 50,
      sendReadReceipts: true, // mavi tikler (self-chat modunda false)
      groups: {
        "*": { requireMention: true },
      },
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
    },
  },
  web: {
    enabled: true,
    heartbeatSeconds: 60,
    reconnect: {
      initialMs: 2000,
      maxMs: 120000,
      factor: 1.4,
      jitter: 0.2,
      maxAttempts: 0,
    },
  },
}
```

<Accordion title="Multi-account WhatsApp">

```json5
{
  channels: {
    whatsapp: {
      accounts: {
        default: {},
        personal: {},
        biz: {
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

- Giden komutlar, mevcutsa varsayılan olarak `default` hesabını kullanır; aksi takdirde yapılandırılmış ilk hesap kimliği (sıralanmış) kullanılır.
- Eski tek hesaplı Baileys kimlik doğrulama dizini, `openclaw doctor` tarafından `whatsapp/default` içine taşınır.
- Hesap bazlı geçersiz kılmalar: `channels.whatsapp.accounts.<id> .sendReadReceipts`, `channels.whatsapp.accounts.<id> .dmPolicy`, `channels.whatsapp.accounts.<id> .allowFrom`.

</Accordion>

### Telegram

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "your-bot-token",
      dmPolicy: "pairing",
      allowFrom: ["tg:123456789"],
      groups: {
        "*": { requireMention: true },
        "-1001234567890": {
          allowFrom: ["@admin"],
          systemPrompt: "Yanıtları kısa tut.",
          topics: {
            "99": {
              requireMention: false,
              skills: ["search"],
              systemPrompt: "Konu dışına çıkma.",
            },
          },
        },
      },
      customCommands: [
        { command: "backup", description: "Git yedeği" },
        { command: "generate", description: "Bir görsel oluştur" },
      ],
      historyLimit: 50,
      replyToMode: "first", // off | first | all
      linkPreview: true,
      streamMode: "partial", // off | partial | block
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph", // paragraph | newline | sentence
      },
      actions: { reactions: true, sendMessage: true },
      reactionNotifications: "own", // off | own | all
      mediaMaxMb: 5,
      retry: {
        attempts: 3,
        minDelayMs: 400,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
      network: { autoSelectFamily: false },
      proxy: "socks5://localhost:9050",
      webhookUrl: "https://example.com/telegram-webhook",
      webhookSecret: "secret",
      webhookPath: "/telegram-webhook",
    },
  },
}
```

- Bot token: `channels.telegram.botToken` veya `channels.telegram.tokenFile`; varsayılan hesap için geri dönüş olarak `TELEGRAM_BOT_TOKEN` kullanılır.
- `configWrites: false`, Telegram tarafından başlatılan yapılandırma yazımlarını (supergroup ID taşımaları, `/config set|unset`) engeller.
- Telegram akış önizlemeleri `sendMessage` + `editMessageText` kullanır (özel ve grup sohbetlerinde çalışır).
- Yeniden deneme politikası: bkz. [Retry policy](/concepts/retry).

### Discord

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "your-bot-token",
      mediaMaxMb: 8,
      allowBots: false,
      actions: {
        reactions: true,
        stickers: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        voiceStatus: true,
        events: true,
        moderation: false,
      },
      replyToMode: "off", // off | first | all
      dmPolicy: "pairing",
      allowFrom: ["1234567890", "steipete"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["openclaw-dm"] },
      guilds: {
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["docs"],
              systemPrompt: "Yalnızca kısa yanıtlar.",
            },
          },
        },
      },
      historyLimit: 20,
      textChunkLimit: 2000,
      chunkMode: "length", // length | newline
      maxLinesPerMessage: 17,
      ui: {
        components: {
          accentColor: "#5865F2",
        },
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1,
      },
    },
  },
}
```

- Token: `channels.discord.token`; varsayılan hesap için geri dönüş olarak `DISCORD_BOT_TOKEN` kullanılır.
- Teslim hedefleri için `user:<id>` (DM) veya `channel:<id>` (guild kanalı) kullanın; yalnızca sayısal kimlikler reddedilir.
- Guild slug’ları küçük harflidir ve boşluklar `-` ile değiştirilir; kanal anahtarları slug formatındaki adı kullanır (`#` olmadan). Guild ID’lerini tercih edin.
- Bot tarafından yazılan mesajlar varsayılan olarak yok sayılır. `allowBots: true` bunu etkinleştirir (botun kendi mesajları yine filtrelenir).
- `maxLinesPerMessage` (varsayılan 17), 2000 karakterin altında olsa bile uzun mesajları böler.
- `channels.discord.ui.components.accentColor`, Discord components v2 kapsayıcıları için vurgu rengini ayarlar.

**Tepki bildirim modları:** `off` (yok), `own` (botun mesajları, varsayılan), `all` (tüm mesajlar), `allowlist` (`guilds.<id> .users` içindekilerden gelen tüm mesajlar için).

### Google Chat

```json5
{
  channels: {
    googlechat: {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url", // app-url | project-number
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890",
      dm: {
        enabled: true,
        policy: "pairing",
        allowFrom: ["users/1234567890"],
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": { allow: true, requireMention: true },
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20,
    },
  },
}
```

- Service account JSON: satır içi (`serviceAccount`) veya dosya tabanlı (`serviceAccountFile`).
- Ortam değişkeni geri dönüşleri: `GOOGLE_CHAT_SERVICE_ACCOUNT` veya `GOOGLE_CHAT_SERVICE_ACCOUNT_FILE`.
- Teslim hedefleri için `spaces/<spaceId>` veya `users/<userId|email>` kullanın.

### Slack

```json5
{
  channels: {
    slack: {
      enabled: true,
      botToken: "xoxb-...",
      appToken: "xapp-...",
      dmPolicy: "pairing",
      allowFrom: ["U123", "U456", "*"],
      dm: { enabled: true, groupEnabled: false, groupChannels: ["G123"] },
      channels: {
        C123: { allow: true, requireMention: true, allowBots: false },
        "#general": {
          allow: true,
          requireMention: true,
          allowBots: false,
          users: ["U123"],
          skills: ["docs"],
          systemPrompt: "Yalnızca kısa yanıtlar.",
        },
      },
      historyLimit: 50,
      allowBots: false,
      reactionNotifications: "own",
      reactionAllowlist: ["U123"],
      replyToMode: "off", // off | first | all
      thread: {
        historyScope: "thread", // thread | channel
        inheritParent: false,
      },
      actions: {
        reactions: true,
        messages: true,
        pins: true,
        memberInfo: true,
        emojiList: true,
      },
      slashCommand: {
        enabled: true,
        name: "openclaw",
        sessionPrefix: "slack:slash",
        ephemeral: true,
      },
      textChunkLimit: 4000,
      chunkMode: "length",
      mediaMaxMb: 20,
    },
  },
}
```

- **Socket mode** hem `botToken` hem de `appToken` gerektirir (varsayılan hesap ortam değişkeni geri dönüşü için `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`).
- **HTTP mode** `botToken` ile birlikte `signingSecret` gerektirir (kök düzeyde veya hesap başına).
- `configWrites: false` Slack tarafından başlatılan yapılandırma yazımlarını engeller.
- Teslim hedefleri için `user:<id>` (DM) veya `channel:<id>` kullanın.

**Reaction notification modes:** `off`, `own` (varsayılan), `all`, `allowlist` (`reactionAllowlist` içinden).

**Thread session isolation:** `thread.historyScope` iş parçacığı başına (varsayılan) veya kanal genelinde paylaşılır. `thread.inheritParent` üst kanalın dökümünü yeni iş parçacıklarına kopyalar.

| Eylem grubu | Varsayılan | Notlar                        |
| ----------- | ---------- | ----------------------------- |
| reactions   | etkin      | Tepki ver + tepkileri listele |
| messages    | etkin      | Oku/gönder/düzenle/sil        |
| pins        | etkin      | Sabitle/kaldır/listele        |
| memberInfo  | etkin      | Üye bilgisi                   |
| emojiList   | etkin      | Özel emoji listesi            |

### Mattermost

Mattermost bir eklenti olarak gelir: `openclaw plugins install @openclaw/mattermost`.

```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing",
      chatmode: "oncall", // oncall | onmessage | onchar
      oncharPrefixes: [">", "!"],
      textChunkLimit: 4000,
      chunkMode: "length",
    },
  },
}
```

Sohbet modları: `oncall` (@-mention olduğunda yanıt verir, varsayılan), `onmessage` (her mesaj), `onchar` (tetikleyici önekle başlayan mesajlar).

### Signal

```json5
{
  channels: {
    signal: {
      reactionNotifications: "own", // off | own | all | allowlist
      reactionAllowlist: ["+15551234567", "uuid:123e4567-e89b-12d3-a456-426614174000"],
      historyLimit: 50,
    },
  },
}
```

**Reaction notification modes:** `off`, `own` (varsayılan), `all`, `allowlist` (`reactionAllowlist` içinden).

### iMessage

OpenClaw `imsg rpc` başlatır (stdio üzerinden JSON-RPC). Daemon veya port gerekmez.

```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "imsg",
      dbPath: "~/Library/Messages/chat.db",
      remoteHost: "user@gateway-host",
      dmPolicy: "pairing",
      allowFrom: ["+15555550123", "user@example.com", "chat_id:123"],
      historyLimit: 50,
      includeAttachments: false,
      mediaMaxMb: 16,
      service: "auto",
      region: "US",
    },
  },
}
```

- Messages veritabanı için Full Disk Access gerektirir.
- `chat_id:<id>` hedeflerini tercih edin. Sohbetleri listelemek için `imsg chats --limit 20` kullanın.
- `cliPath` bir SSH sarmalayıcısına işaret edebilir; SCP ile ekleri almak için `remoteHost` ayarlayın.

<Accordion title="iMessage SSH wrapper example">

```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

</Accordion>

### Çoklu hesap (tüm kanallar)

Kanal başına birden fazla hesap çalıştırın (her biri kendi `accountId` değerine sahip):

```json5
{
  channels: {
    telegram: {
      accounts: {
        default: {
          name: "Primary bot",
          botToken: "123456:ABC...",
        },
        alerts: {
          name: "Alerts bot",
          botToken: "987654:XYZ...",
        },
      },
    },
  },
}
```

- `accountId` belirtilmediğinde `default` kullanılır (CLI + yönlendirme).
- Env token’ları yalnızca **default** hesap için geçerlidir.
- Temel kanal ayarları, hesap bazında üzerine yazılmadıkça tüm hesaplara uygulanır.
- Her hesabı farklı bir agente yönlendirmek için `bindings[].match.accountId` kullanın.

### Grup sohbetinde mention zorunluluğu

Grup mesajlarında varsayılan olarak **mention gerekli**’dir (metadata mention veya regex desenleri). WhatsApp, Telegram, Discord, Google Chat ve iMessage grup sohbetleri için geçerlidir.

**Mention türleri:**

- **Metadata mention’lar**: Platformun yerel @-mention’ları. WhatsApp self-chat modunda yok sayılır.
- **Metin desenleri**: `agents.list[].groupChat.mentionPatterns` içindeki regex desenleri. Her zaman kontrol edilir.
- Mention zorunluluğu yalnızca tespit mümkün olduğunda uygulanır (yerel mention’lar veya en az bir desen mevcutsa).

```json5
{
  messages: {
    groupChat: { historyLimit: 50 },
  },
  agents: {
    list: [{ id: "main", groupChat: { mentionPatterns: ["@openclaw", "openclaw"] } }],
  },
}
```

`messages.groupChat.historyLimit` genel varsayılanı belirler. Kanallar `channels.<channel> .historyLimit` (veya hesap bazında) ile üzerine yazabilir. Devre dışı bırakmak için `0` ayarlayın.

#### DM geçmişi limitleri

```json5
{
  channels: {
    telegram: {
      dmHistoryLimit: 30,
      dms: {
        "123456789": { historyLimit: 50 },
      },
    },
  },
}
```

Çözümleme sırası: DM’e özel üzerine yazma → sağlayıcı varsayılanı → limitsiz (tümü saklanır).

Desteklenenler: `telegram`, `whatsapp`, `discord`, `slack`, `signal`, `imessage`, `msteams`.

#### Self-chat modu

Self-chat modunu etkinleştirmek için kendi numaranızı `allowFrom` içine ekleyin (yerel @-mention’ları yok sayar, yalnızca metin desenlerine yanıt verir):

```json5
{
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: { "*": { requireMention: true } },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: { mentionPatterns: ["reisponde", "@openclaw"] },
      },
    ],
  },
}
```

### Komutlar (sohbet komutu işleme)

```json5
{
  commands: {
    native: "auto", // desteklendiğinde yerel komutları kaydet
    text: true, // sohbet mesajlarında /komutları ayrıştır
    bash: false, // ! kullanımına izin ver (takma ad: /bash)
    bashForegroundMs: 2000,
    config: false, // /config kullanımına izin ver
    debug: false, // /debug kullanımına izin ver
    restart: false, // /restart + gateway restart aracına izin ver
    allowFrom: {
      "*": ["user1"],
      discord: ["user:123"],
    },
    useAccessGroups: true,
  },
}
```

<Accordion title="Command details">

- Metin komutları başında `/` bulunan **bağımsız** mesajlar olmalıdır.
- `native: "auto"` Discord/Telegram için yerel komutları etkinleştirir, Slack’i kapalı bırakır.
- Kanal bazında üzerine yazın: `channels.discord.commands.native` (bool veya `"auto"`). `false` daha önce kaydedilmiş komutları temizler.
- `channels.telegram.customCommands` ek Telegram bot menü girdileri ekler.
- `bash: true` `!  <cmd>` kullanımını etkinleştirir (host shell için). `tools.elevated.enabled` gerektirir ve gönderenin `tools.elevated.allowFrom.<channel>` içinde olması gerekir.
- `config: true` `/config`’i etkinleştirir (`openclaw.json` dosyasını okur/yazar).
- `channels.<provider>`.configWrites\` kanal başına yapılandırma değişikliklerini kontrol eder (varsayılan: true).
- `allowFrom` sağlayıcı başınadır. Ayarlandığında **tek** yetkilendirme kaynağı olur (kanal allowlist’leri/eşleştirme ve `useAccessGroups` yok sayılır).
- `useAccessGroups: false`, `allowFrom` ayarlı değilken komutların erişim grubu politikalarını atlamasına izin verir.

</Accordion>

---

## Agent varsayılanları

### `agents.defaults.workspace`

Varsayılan: `~/.openclaw/workspace`.

```json5
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
}
```

### `agents.defaults.repoRoot`

Sistem istemindeki Runtime satırında gösterilen isteğe bağlı depo kök dizini. Ayarlanmazsa, OpenClaw çalışma alanından yukarı doğru tarayarak otomatik olarak tespit eder.

```json5
{
  agents: { defaults: { repoRoot: "~/Projects/openclaw" } },
}
```

### `agents.defaults.skipBootstrap`

Çalışma alanı bootstrap dosyalarının (`AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`, `BOOTSTRAP.md`) otomatik oluşturulmasını devre dışı bırakır.

```json5
{
  agents: { defaults: { skipBootstrap: true } },
}
```

### `agents.defaults.bootstrapMaxChars`

Kırpma uygulanmadan önce çalışma alanı bootstrap dosyası başına maksimum karakter sayısı. Varsayılan: `20000`.

```json5
{
  agents: { defaults: { bootstrapMaxChars: 20000 } },
}
```

### `agents.defaults.bootstrapTotalMaxChars`

Tüm çalışma alanı bootstrap dosyalarına enjekte edilen toplam maksimum karakter sayısı. Varsayılan: `24000`.

```json5
{
  agents: { defaults: { bootstrapTotalMaxChars: 24000 } },
}
```

### `agents.defaults.userTimezone`

Sistem istemi bağlamı için saat dilimi (mesaj zaman damgaları için değil). Ana makinenin saat dilimine geri döner.

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } },
}
```

### `agents.defaults.timeFormat`

Sistem istemindeki saat formatı. Varsayılan: `auto` (OS tercihi).

```json5
{
  agents: { defaults: { timeFormat: "auto" } }, // auto | 12 | 24
}
```

### `agents.defaults.model`

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.1"],
      },
      imageModel: {
        primary: "openrouter/qwen/qwen-2.5-vl-72b-instruct:free",
        fallbacks: ["openrouter/google/gemini-2.0-flash-vision:free"],
      },
      thinkingDefault: "low",
      verboseDefault: "off",
      elevatedDefault: "on",
      timeoutSeconds: 600,
      mediaMaxMb: 5,
      contextTokens: 200000,
      maxConcurrent: 3,
    },
  },
}
```

- `model.primary`: biçim `provider/model` (ör. `anthropic/claude-opus-4-6`). Sağlayıcıyı belirtmezseniz, OpenClaw `anthropic` varsayar (kullanımdan kaldırıldı).
- `models`: `/model` için yapılandırılmış model kataloğu ve allowlist. Her giriş `alias` (kısayol) ve `params` (sağlayıcıya özgü: `temperature`, `maxTokens`) içerebilir.
- `imageModel`: yalnızca birincil model görüntü girdisini desteklemiyorsa kullanılır.
- `maxConcurrent`: oturumlar arasında maksimum paralel agent çalıştırma sayısı (her oturum yine de sıralı yürütülür). Varsayılan: 1.

**Yerleşik alias kısayolları** (yalnızca model `agents.defaults.models` içinde olduğunda uygulanır):

| Alias          | Model                           |
| -------------- | ------------------------------- |
| `opus`         | `anthropic/claude-opus-4-6`     |
| `sonnet`       | `anthropic/claude-sonnet-4-5`   |
| `gpt`          | `openai/gpt-5.2`                |
| `gpt-mini`     | `openai/gpt-5-mini`             |
| `gemini`       | `google/gemini-3-pro-preview`   |
| `gemini-flash` | `google/gemini-3-flash-preview` |

Yapılandırdığınız alias’lar her zaman varsayılanların önüne geçer.

Z.AI GLM-4.x modelleri, `--thinking off` ayarlamadığınız veya `agents.defaults.models["zai/<model>"].params.thinking` değerini kendiniz tanımlamadığınız sürece düşünme modunu otomatik olarak etkinleştirir.

### `agents.defaults.cliBackends`

Metin tabanlı geri dönüş çalıştırmaları için isteğe bağlı CLI backends (araç çağrısı yok). API sağlayıcıları başarısız olduğunda yedek olarak kullanışlıdır.

```json5
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

- CLI backends metin önceliklidir; araçlar her zaman devre dışıdır.
- `sessionArg` ayarlandığında oturumlar desteklenir.
- `imageArg` dosya yollarını kabul ettiğinde görsel iletimi desteklenir.

### `agents.defaults.heartbeat`

Periyodik heartbeat çalıştırmaları.

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m", // 0m disables
        model: "openai/gpt-5.2-mini",
        includeReasoning: false,
        session: "main",
        to: "+15555550123",
        target: "last", // last | whatsapp | telegram | discord | ... | none
        prompt: "Read HEARTBEAT.md if it exists...",
        ackMaxChars: 300,
      },
    },
  },
}
```

- `every`: süre dizesi (ms/s/m/h). Varsayılan: `30m`.
- Agent başına: `agents.list[].heartbeat` ayarlayın. Herhangi bir agent `heartbeat` tanımladığında, **yalnızca bu agent’lar** heartbeat çalıştırır.
- Heartbeat’ler tam agent turu çalıştırır — daha kısa aralıklar daha fazla token tüketir.

### `agents.defaults.compaction`

```json5
{
  agents: {
    defaults: {
      compaction: {
        mode: "safeguard", // default | safeguard
        reserveTokensFloor: 24000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 6000,
          systemPrompt: "Session nearing compaction. Store durable memories now.",
          prompt: "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
        },
      },
    },
  },
}
```

- `mode`: `default` veya `safeguard` (uzun geçmişler için parçalı özetleme). Bkz. [Compaction](/concepts/compaction).
- `memoryFlush`: kalıcı anıları saklamak için otomatik compaction öncesinde sessiz agent turu. Çalışma alanı salt okunur olduğunda atlanır.

### `agents.defaults.contextPruning`

LLM’ye gönderilmeden önce bellek içindeki bağlamdan **eski araç sonuçlarını** temizler. Diskteki oturum geçmişini **değiştirmez**.

```json5
{
  agents: {
    defaults: {
      contextPruning: {
        mode: "cache-ttl", // off | cache-ttl
        ttl: "1h", // süre (ms/s/m/h), varsayılan birim: dakika
        keepLastAssistants: 3,
        softTrimRatio: 0.3,
        hardClearRatio: 0.5,
        minPrunableToolChars: 50000,
        softTrim: { maxChars: 4000, headChars: 1500, tailChars: 1500 },
        hardClear: { enabled: true, placeholder: "[Old tool result content cleared]" },
        tools: { deny: ["browser", "canvas"] },
      },
    },
  },
}
```

<Accordion title="cache-ttl mode behavior">

- `mode: "cache-ttl"` temizleme işlemlerini etkinleştirir.
- `ttl`, temizlemenin tekrar ne sıklıkla çalışabileceğini kontrol eder (son önbellek erişiminden sonra).
- Temizleme işlemi önce aşırı büyük araç sonuçlarını yumuşak şekilde kısaltır (soft-trim), gerekirse daha eski araç sonuçlarını tamamen temizler (hard-clear).

**Soft-trim**, başlangıcı ve sonu korur ve ortasına `...` ekler.

**Hard-clear**, tüm araç sonucunu yer tutucu metinle değiştirir.

Notlar:

- Görsel blokları asla kısaltılmaz veya temizlenmez.
- Oranlar karakter bazlıdır (yaklaşık), kesin token sayımları değildir.
- `keepLastAssistants` değerinden daha az asistan mesajı varsa temizleme atlanır.

</Accordion>

Davranış ayrıntıları için [Session Pruning](/concepts/session-pruning) bölümüne bakın.

### Blok akışı

```json5
{
  agents: {
    defaults: {
      blockStreamingDefault: "off", // on | off
      blockStreamingBreak: "text_end", // text_end | message_end
      blockStreamingChunk: { minChars: 800, maxChars: 1200 },
      blockStreamingCoalesce: { idleMs: 1000 },
      humanDelay: { mode: "natural" }, // off | natural | custom (minMs/maxMs kullanın)
    },
  },
}
```

- Telegram dışındaki kanallarda blok yanıtları etkinleştirmek için `*.blockStreaming: true` açıkça ayarlanmalıdır.
- Kanal geçersiz kılmaları: `channels.<channel>``.blockStreamingCoalesce` (ve hesap başına varyantları). Signal/Slack/Discord/Google Chat için varsayılan `minChars: 1500` değeridir.
- `humanDelay`: blok yanıtlar arasında rastgele duraklama. `natural` = 800–2500ms. Ajan başına geçersiz kılma: `agents.list[].humanDelay`.

Davranış ve parçalara ayırma ayrıntıları için [Streaming](/concepts/streaming) bölümüne bakın.

### Yazıyor göstergeleri

```json5
{
  agents: {
    defaults: {
      typingMode: "instant", // never | instant | thinking | message
      typingIntervalSeconds: 6,
    },
  },
}
```

- Varsayılanlar: doğrudan sohbetler/etiketlemeler için `instant`, etiketlenmemiş grup sohbetleri için `message`.
- Oturum başına geçersiz kılmalar: `session.typingMode`, `session.typingIntervalSeconds`.

[Typing Indicators](/concepts/typing-indicators) bölümüne bakın.

### `agents.defaults.sandbox`

Gömülü ajan için isteğe bağlı **Docker sandbox** özelliği. Tam kılavuz için [Sandboxing](/gateway/sandboxing) bölümüne bakın.

```json5
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // off | non-main | all
        scope: "agent", // session | agent | shared
        workspaceAccess: "none", // none | ro | rw
        workspaceRoot: "~/.openclaw/sandboxes",
        docker: {
          image: "openclaw-sandbox:bookworm-slim",
          containerPrefix: "openclaw-sbx-",
          workdir: "/workspace",
          readOnlyRoot: true,
          tmpfs: ["/tmp", "/var/tmp", "/run"],
          network: "none",
          user: "1000:1000",
          capDrop: ["ALL"],
          env: { LANG: "C.UTF-8" },
          setupCommand: "apt-get update && apt-get install -y git curl jq",
          pidsLimit: 256,
          memory: "1g",
          memorySwap: "2g",
          cpus: 1,
          ulimits: {
            nofile: { soft: 1024, hard: 2048 },
            nproc: 256,
          },
          seccompProfile: "/path/to/seccomp.json",
          apparmorProfile: "openclaw-sandbox",
          dns: ["1.1.1.1", "8.8.8.8"],
          extraHosts: ["internal.service:10.0.0.5"],
          binds: ["/home/user/source:/source:rw"],
        },
        browser: {
          enabled: false,
          image: "openclaw-sandbox-browser:bookworm-slim",
          cdpPort: 9222,
          vncPort: 5900,
          noVncPort: 6080,
          headless: false,
          enableNoVnc: true,
          allowHostControl: false,
          autoStart: true,
          autoStartTimeoutMs: 12000,
        },
        prune: {
          idleHours: 24,
          maxAgeDays: 7,
        },
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        allow: [
          "exec",
          "process",
          "read",
          "write",
          "edit",
          "apply_patch",
          "sessions_list",
          "sessions_history",
          "sessions_send",
          "sessions_spawn",
          "session_status",
        ],
        deny: ["browser", "canvas", "nodes", "cron", "discord", "gateway"],
      },
    },
  },
}
```

<Accordion title="Sandbox details">

**Çalışma alanı erişimi:**

- `none`: `~/.openclaw/sandboxes` altında kapsam başına sandbox çalışma alanı
- `ro`: sandbox çalışma alanı `/workspace` konumunda, ajan çalışma alanı `/agent` konumuna salt okunur olarak bağlanır
- `rw`: ajan çalışma alanı `/workspace` konumuna okuma/yazma olarak bağlanır

**Kapsam:**

- `session`: oturum başına container + çalışma alanı
- `agent`: ajan başına bir container + çalışma alanı (varsayılan)
- `shared`: paylaşılan container ve çalışma alanı (oturumlar arası izolasyon yok)

**`setupCommand`** container oluşturulduktan sonra bir kez çalıştırılır (`sh -lc` aracılığıyla). Ağ çıkışı, yazılabilir root ve root kullanıcısı gerektirir.

**Container’lar varsayılan olarak `network: "none"` ile gelir** — ajan dış erişime ihtiyaç duyuyorsa `"bridge"` olarak ayarlayın.

**Gelen ekler** aktif çalışma alanındaki `media/inbound/*` içine yerleştirilir.

**`docker.binds`** ek ana makine dizinlerini bağlar; global ve ajan bazlı bind’ler birleştirilir.

**Sandboxed browser** (`sandbox.browser.enabled`): container içinde Chromium + CDP. noVNC URL’si sistem istemine enjekte edilir. Ana yapılandırmada `browser.enabled` gerektirmez.

- `allowHostControl: false` (varsayılan) sandbox oturumlarının ana makine tarayıcısını hedeflemesini engeller.
- `sandbox.browser.binds` ek ana makine dizinlerini yalnızca sandbox tarayıcı container’ına bağlar. Ayarlanmışsa (`[]` dahil), tarayıcı container’ı için `docker.binds` değerinin yerine geçer.

</Accordion>

İmajları oluşturun:

```bash
scripts/sandbox-setup.sh           # ana sandbox imajı
scripts/sandbox-browser-setup.sh   # isteğe bağlı tarayıcı imajı
```

### `agents.list` (ajan bazlı geçersiz kılmalar)

```json5
{
  agents: {
    list: [
      {
        id: "main",
        default: true,
        name: "Main Agent",
        workspace: "~/.openclaw/workspace",
        agentDir: "~/.openclaw/agents/main/agent",
        model: "anthropic/claude-opus-4-6", // veya { primary, fallbacks }
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
        groupChat: { mentionPatterns: ["@openclaw"] },
        sandbox: { mode: "off" },
        subagents: { allowAgents: ["*"] },
        tools: {
          profile: "coding",
          allow: ["browser"],
          deny: ["canvas"],
          elevated: { enabled: true },
        },
      },
    ],
  },
}
```

- `id`: sabit ajan kimliği (zorunlu).
- `default`: birden fazla ayarlandığında ilk olan kazanır (uyarı kaydedilir). Hiçbiri ayarlanmazsa, listedeki ilk giriş varsayılan olur.
- `model`: string biçimi yalnızca `primary`’yi geçersiz kılar; nesne biçimi `{ primary, fallbacks }` her ikisini de geçersiz kılar (`[]` global fallbacks’i devre dışı bırakır).
- `identity.avatar`: çalışma alanına göreli yol, `http(s)` URL veya `data:` URI.
- `identity` varsayılanları türetir: `ackReaction` `emoji`’den, `mentionPatterns` `name`/`emoji`’den.
- `subagents.allowAgents`: `sessions_spawn` için izin verilen ajan kimlikleri listesi (`["*"]` = herhangi biri; varsayılan: yalnızca aynı ajan).

---

## Çoklu ajan yönlendirmesi

Tek bir Gateway içinde birden fazla izole ajan çalıştırın. Bkz. [Multi-Agent](/concepts/multi-agent).

```json5
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
}
```

### Binding eşleşme alanları

- `match.channel` (zorunlu)
- `match.accountId` (isteğe bağlı; `*` = herhangi bir hesap; boş bırakılırsa = varsayılan hesap)
- `match.peer` (isteğe bağlı; `{ kind: direct|group|channel, id }`)
- `match.guildId` / `match.teamId` (isteğe bağlı; kanala özgü)

**Deterministik eşleşme sırası:**

1. `match.peer`
2. `match.guildId`
3. `match.teamId`
4. `match.accountId` (tam eşleşme, peer/guild/team olmadan)
5. `match.accountId: "*"` (kanal genelinde)
6. Varsayılan ajan

Her katmanda, eşleşen ilk `bindings` girdisi kazanır.

### Ajan başına erişim profilleri

<Accordion title="Full access (no sandbox)">

```json5
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: { mode: "off" },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="Read-only tools + workspace">

```json5
{
  agents: {
    list: [
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "ro" },
        tools: {
          allow: [
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "exec", "process", "browser"],
        },
      },
    ],
  },
}
```

</Accordion>

<Accordion title="No filesystem access (messaging only)">

```json5
{
  agents: {
    list: [
      {
        id: "public",
        workspace: "~/.openclaw/workspace-public",
        sandbox: { mode: "all", scope: "agent", workspaceAccess: "none" },
        tools: {
          allow: [
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
            "whatsapp",
            "telegram",
            "slack",
            "discord",
            "gateway",
          ],
          deny: [
            "read",
            "write",
            "edit",
            "apply_patch",
            "exec",
            "process",
            "browser",
            "canvas",
            "nodes",
            "cron",
            "gateway",
            "image",
          ],
        },
      },
    ],
  },
}
```

</Accordion>

Öncelik ayrıntıları için [Multi-Agent Sandbox & Tools](/tools/multi-agent-sandbox-tools) bölümüne bakın.

---

## Oturum

```json5
{
  session: {
    scope: "per-sender",
    dmScope: "main", // main | per-peer | per-channel-peer | per-account-channel-peer
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      mode: "daily", // daily | idle
      atHour: 4,
      idleMinutes: 60,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    maintenance: {
      mode: "warn", // warn | enforce
      pruneAfter: "30d",
      maxEntries: 500,
      rotateBytes: "10mb",
    },
    mainKey: "main", // eski (çalışma zamanı her zaman "main" kullanır)
    agentToAgent: { maxPingPongTurns: 5 },
    sendPolicy: {
      rules: [{ action: "deny", match: { channel: "discord", chatType: "group" } }],
      default: "allow",
    },
  },
}
```

<Accordion title="Session field details">

- **`dmScope`**: DM’lerin nasıl gruplandığını belirler.
  - `main`: tüm DM’ler ana oturumu paylaşır.
  - `per-peer`: kanallar arasında gönderici kimliğine göre izole eder.
  - `per-channel-peer`: kanal + gönderici bazında izole eder (çok kullanıcılı gelen kutuları için önerilir).
  - `per-account-channel-peer`: hesap + kanal + gönderici bazında izole eder (çoklu hesaplar için önerilir).
- **`identityLinks`**: kanallar arası oturum paylaşımı için kanonik kimlikleri sağlayıcı önekli eşlerle eşler.
- **`reset`**: birincil sıfırlama politikası. `daily`, yerel saatte `atHour` zamanında sıfırlar; `idle`, `idleMinutes` sonrasında sıfırlar. Her ikisi de yapılandırılmışsa, önce süresi dolan geçerli olur.
- **`resetByType`**: tür bazlı geçersiz kılmalar (`direct`, `group`, `thread`). Eski `dm`, `direct` için takma ad olarak kabul edilir.
- **`mainKey`**: eski alan. Çalışma zamanı artık ana direct-chat kovası için her zaman `"main"` kullanır.
- **`sendPolicy`**: `channel`, `chatType` (`direct|group|channel`, eski `dm` takma adıyla), `keyPrefix` veya `rawKeyPrefix` ile eşleştirme yapar. İlk deny kuralı geçerli olur.
- **`maintenance`**: `warn`, tahliye sırasında etkin oturumu uyarır; `enforce`, budama ve döndürme işlemlerini uygular.

</Accordion>

---

## Mesajlar

```json5
{
  messages: {
    responsePrefix: "🦞", // veya "auto"
    ackReaction: "👀",
    ackReactionScope: "group-mentions", // group-mentions | group-all | direct | all
    removeAckAfterReply: false,
    queue: {
      mode: "collect", // steer | followup | collect | steer-backlog | steer+backlog | queue | interrupt
      debounceMs: 1000,
      cap: 20,
      drop: "summarize", // old | new | summarize
      byChannel: {
        whatsapp: "collect",
        telegram: "collect",
      },
    },
    inbound: {
      debounceMs: 2000, // 0 devre dışı bırakır
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
      },
    },
  },
}
```

### Yanıt öneki

Kanal/hesap bazlı geçersiz kılmalar: `channels.<channel> .responsePrefix`, `channels.<channel> .accounts.<id> .responsePrefix`.

Çözümleme (en özeli kazanır): hesap → kanal → global. `""` devre dışı bırakır ve zincirlemeyi durdurur. `"auto"`, `[{identity.name}]` değerini türetir.

**Şablon değişkenleri:**

| Değişken          | Açıklama                | Örnek                                  |
| ----------------- | ----------------------- | -------------------------------------- |
| `{model}`         | Kısa model adı          | `claude-opus-4-6`                      |
| `{modelFull}`     | Tam model tanımlayıcısı | `anthropic/claude-opus-4-6`            |
| `{provider}`      | Sağlayıcı adı           | `anthropic`                            |
| `{thinkingLevel}` | Mevcut düşünme seviyesi | `high`, `low`, `off`                   |
| `{identity.name}` | Ajan kimlik adı         | (`"auto"` ile aynı) |

Değişkenler büyük/küçük harfe duyarlı değildir. `{think}`, `{thinkingLevel}` için bir takma addır.

### Ack tepkisi

- Varsayılan olarak etkin ajanın `identity.emoji` değeri kullanılır, aksi halde `"👀"`. Devre dışı bırakmak için `""` olarak ayarlayın.
- Kanal bazlı geçersiz kılmalar: `channels.<channel>
  .ackReaction`, `channels.<channel>
  .accounts.<id>
  .ackReaction`.
- Çözümleme sırası: hesap → kanal → `messages.ackReaction` → kimlik yedeği.
- Kapsam: `group-mentions` (varsayılan), `group-all`, `direct`, `all`.
- `removeAckAfterReply`: yanıt sonrası ack’i kaldırır (yalnızca Slack/Discord/Telegram/Google Chat).

### Gelen mesaj gecikme dengelemesi

Aynı göndericiden gelen hızlı, yalnızca metin içeren mesajları tek bir ajan turunda toplar. Medya/ekler anında işlenir. Kontrol komutları gecikme dengelemeyi atlar.

### TTS (metinden konuşmaya)

```json5
{
  messages: {
    tts: {
      auto: "always", // off | always | inbound | tagged
      mode: "final", // final | all
      provider: "elevenlabs",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: { enabled: true },
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.openclaw/settings/tts.json",
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0,
        },
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy",
      },
    },
  },
}
```

- `auto`, otomatik TTS’i kontrol eder. `/tts off|always|inbound|tagged` oturum bazında geçersiz kılar.
- `summaryModel`, otomatik özetleme için `agents.defaults.model.primary` ayarını geçersiz kılar.
- API anahtarları, `ELEVENLABS_API_KEY`/`XI_API_KEY` ve `OPENAI_API_KEY` değerlerine geri düşer.

---

## Konuşma

Konuşma modu için varsayılanlar (macOS/iOS/Android).

```json5
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    voiceAliases: {
      Clawd: "EXAVITQu4vr4xnSDxMaL",
      Roger: "CwhRBWXzGAHq8TQ4Fs17",
    },
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

- Voice ID’ler `ELEVENLABS_VOICE_ID` veya `SAG_VOICE_ID` değerlerine geri düşer.
- `apiKey`, `ELEVENLABS_API_KEY` değerine geri düşer.
- `voiceAliases`, Talk yönergelerinin kolay adlar kullanmasını sağlar.

---

## Araçlar

### Araç profilleri

`tools.profile`, `tools.allow`/`tools.deny` öncesinde temel bir izin listesi belirler:

| Profil      | İçerir                                                                                    |
| ----------- | ----------------------------------------------------------------------------------------- |
| `minimal`   | Yalnızca `session_status`                                                                 |
| `coding`    | `group:fs`, `group:runtime`, `group:sessions`, `group:memory`, `image`                    |
| `messaging` | `group:messaging`, `sessions_list`, `sessions_history`, `sessions_send`, `session_status` |
| `full`      | Kısıtlama yok (ayarlanmamış ile aynı)                                  |

### Araç grupları

| Grup               | Araçlar                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `group:runtime`    | `exec`, `process` (`bash`, `exec` için takma ad olarak kabul edilir)  |
| `group:fs`         | `read`, `write`, `edit`, `apply_patch`                                                   |
| `group:sessions`   | `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status` |
| `group:memory`     | `memory_search`, `memory_get`                                                            |
| `group:web`        | `web_search`, `web_fetch`                                                                |
| `group:ui`         | `browser`, `canvas`                                                                      |
| `group:automation` | `cron`, `gateway`                                                                        |
| `group:messaging`  | `message`                                                                                |
| `group:nodes`      | `nodes`                                                                                  |
| `group:openclaw`   | Tüm yerleşik araçlar (sağlayıcı eklentileri hariç)                    |

### `tools.allow` / `tools.deny`

Genel araç izin/engelleme politikası (engelleme önceliklidir). Büyük/küçük harfe duyarsızdır, `*` joker karakterlerini destekler. Docker sandbox kapalıyken bile uygulanır.

```json5
{
  tools: { deny: ["browser", "canvas"] },
}
```

### `tools.byProvider`

Belirli sağlayıcılar veya modeller için araçları daha da kısıtlayın. Sıra: temel profil → sağlayıcı profili → allow/deny.

```json5
{
  tools: {
    profile: "coding",
    byProvider: {
      "google-antigravity": { profile: "minimal" },
      "openai/gpt-5.2": { allow: ["group:fs", "sessions_list"] },
    },
  },
}
```

### `tools.elevated`

Yükseltilmiş (host) exec erişimini kontrol eder:

```json5
{
  tools: {
    elevated: {
      enabled: true,
      allowFrom: {
        whatsapp: ["+15555550123"],
        discord: ["steipete", "1234567890123"],
      },
    },
  },
}
```

- Ajan başına geçersiz kılma (`agents.list[].tools.elevated`) yalnızca daha fazla kısıtlama uygulayabilir.
- `/elevated on|off|ask|full` durumu oturum başına kaydeder; satır içi yönergeler tek bir mesaja uygulanır.
- Yükseltilmiş `exec` host üzerinde çalışır, sandbox korumasını atlar.

### `tools.exec`

```json5
{
  tools: {
    exec: {
      backgroundMs: 10000,
      timeoutSec: 1800,
      cleanupMs: 1800000,
      notifyOnExit: true,
      notifyOnExitEmptySuccess: false,
      applyPatch: {
        enabled: false,
        allowModels: ["gpt-5.2"],
      },
    },
  },
}
```

### `tools.web`

```json5
{
  tools: {
    web: {
      search: {
        enabled: true,
        apiKey: "brave_api_key", // or BRAVE_API_KEY env
        maxResults: 5,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
      },
      fetch: {
        enabled: true,
        maxChars: 50000,
        maxCharsCap: 50000,
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        userAgent: "custom-ua",
      },
    },
  },
}
```

### `tools.media`

Gelen medya anlama (görsel/ses/video) yapılandırmasını yapar:

```json5
{
  tools: {
    media: {
      concurrency: 2,
      audio: {
        enabled: true,
        maxBytes: 20971520,
        scope: {
          default: "deny",
          rules: [{ action: "allow", match: { chatType: "direct" } }],
        },
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          { type: "cli", command: "whisper", args: ["--model", "base", "{{MediaPath}}"] },
        ],
      },
      video: {
        enabled: true,
        maxBytes: 52428800,
        models: [{ provider: "google", model: "gemini-3-flash-preview" }],
      },
    },
  },
}
```

<Accordion title="Media model entry fields">

**Sağlayıcı girdisi** (`type: "provider"` veya belirtilmemiş):

- `provider`: API sağlayıcı kimliği (`openai`, `anthropic`, `google`/`gemini`, `groq`, vb.)
- `model`: model kimliği geçersiz kılma
- `profile` / `preferredProfile`: kimlik doğrulama profili seçimi

**CLI girdisi** (`type: "cli"`):

- `command`: çalıştırılacak yürütülebilir dosya
- `args`: şablonlu argümanlar (`{{MediaPath}}`, `{{Prompt}}`, `{{MaxChars}}`, vb. desteklenir)

**Ortak alanlar:**

- `capabilities`: isteğe bağlı liste (`image`, `audio`, `video`). Varsayılanlar: `openai`/`anthropic`/`minimax` → image, `google` → image+audio+video, `groq` → audio.
- `prompt`, `maxChars`, `maxBytes`, `timeoutSeconds`, `language`: giriş başına geçersiz kılmalar.
- Hatalarda bir sonraki girdiye geri dönülür.

Sağlayıcı kimlik doğrulaması standart sırayı izler: auth profilleri → env değişkenleri → `models.providers.*.apiKey`.

</Accordion>

### `tools.agentToAgent`

```json5
{
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
}
```

### `tools.subagents`

```json5
{
  agents: {
    defaults: {
      subagents: {
        model: "minimax/MiniMax-M2.1",
        maxConcurrent: 1,
        archiveAfterMinutes: 60,
      },
    },
  },
}
```

- `model`: başlatılan alt ajanlar için varsayılan model. Belirtilmezse, alt ajanlar çağıranın modelini devralır.
- Alt ajan başına araç politikası: `tools.subagents.tools.allow` / `tools.subagents.tools.deny`.

---

## Özel sağlayıcılar ve temel URL’ler

OpenClaw, pi-coding-agent model kataloğunu kullanır. Yapılandırmada veya `~/.openclaw/agents/<agentId>/agent/models.json` içinde `models.providers` aracılığıyla özel sağlayıcılar ekleyin.

```json5
{
  models: {
    mode: "merge", // merge (varsayılan) | replace
    providers: {
      "custom-proxy": {
        baseUrl: "http://localhost:4000/v1",
        apiKey: "LITELLM_KEY",
        api: "openai-completions", // openai-completions | openai-responses | anthropic-messages | google-generative-ai
        models: [
          {
            id: "llama-3.1-8b",
            name: "Llama 3.1 8B",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 128000,
            maxTokens: 32000,
          },
        ],
      },
    },
  },
}
```

- Özel kimlik doğrulama ihtiyaçları için `authHeader: true` + `headers` kullanın.
- Ajan yapılandırma kök dizinini `OPENCLAW_AGENT_DIR` (veya `PI_CODING_AGENT_DIR`) ile geçersiz kılın.

### Sağlayıcı örnekleri

<Accordion title="Cerebras (GLM 4.6 / 4.7)">

```json5
{
  env: { CEREBRAS_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: {
        primary: "cerebras/zai-glm-4.7",
        fallbacks: ["cerebras/zai-glm-4.6"],
      },
      models: {
        "cerebras/zai-glm-4.7": { alias: "GLM 4.7 (Cerebras)" },
        "cerebras/zai-glm-4.6": { alias: "GLM 4.6 (Cerebras)" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      cerebras: {
        baseUrl: "https://api.cerebras.ai/v1",
        apiKey: "${CEREBRAS_API_KEY}",
        api: "openai-completions",
        models: [
          { id: "zai-glm-4.7", name: "GLM 4.7 (Cerebras)" },
          { id: "zai-glm-4.6", name: "GLM 4.6 (Cerebras)" },
        ],
      },
    },
  },
}
```

Cerebras için `cerebras/zai-glm-4.7`; doğrudan Z.AI için `zai/glm-4.7` kullanın.

</Accordion>

<Accordion title="OpenCode Zen">

```json5
{
  agents: {
    defaults: {
      model: { primary: "opencode/claude-opus-4-6" },
      models: { "opencode/claude-opus-4-6": { alias: "Opus" } },
    },
  },
}
```

`OPENCODE_API_KEY` (veya `OPENCODE_ZEN_API_KEY`) ayarlayın. Kısayol: `openclaw onboard --auth-choice opencode-zen`.

</Accordion>

<Accordion title="Z.AI (GLM-4.7)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "zai/glm-4.7" },
      models: { "zai/glm-4.7": {} },
    },
  },
}
```

`ZAI_API_KEY` ayarlayın. `z.ai/*` ve `z-ai/*` kabul edilen takma adlardır. Kısayol: `openclaw onboard --auth-choice zai-api-key`.

- Genel uç nokta: `https://api.z.ai/api/paas/v4`
- Kodlama uç noktası (varsayılan): `https://api.z.ai/api/coding/paas/v4`
- Genel uç nokta için, base URL geçersiz kılma ile özel bir sağlayıcı tanımlayın.

</Accordion>

<Accordion title="Moonshot AI (Kimi)">

```json5
{
  env: { MOONSHOT_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "moonshot/kimi-k2.5" },
      models: { "moonshot/kimi-k2.5": { alias: "Kimi K2.5" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      moonshot: {
        baseUrl: "https://api.moonshot.ai/v1",
        apiKey: "${MOONSHOT_API_KEY}",
        api: "openai-completions",
        models: [
          {
            id: "kimi-k2.5",
            name: "Kimi K2.5",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 256000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

Çin uç noktası için: `baseUrl: "https://api.moonshot.cn/v1"` veya `openclaw onboard --auth-choice moonshot-api-key-cn`.

</Accordion>

<Accordion title="Kimi Coding">

```json5
{
  env: { KIMI_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "kimi-coding/k2p5" },
      models: { "kimi-coding/k2p5": { alias: "Kimi K2.5" } },
    },
  },
}
```

Anthropic uyumlu, yerleşik sağlayıcı. Kısayol: `openclaw onboard --auth-choice kimi-code-api-key`.

</Accordion>

<Accordion title="Synthetic (Anthropic-compatible)">

```json5
{
  env: { SYNTHETIC_API_KEY: "sk-..." },
  agents: {
    defaults: {
      model: { primary: "synthetic/hf:MiniMaxAI/MiniMax-M2.1" },
      models: { "synthetic/hf:MiniMaxAI/MiniMax-M2.1": { alias: "MiniMax M2.1" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      synthetic: {
        baseUrl: "https://api.synthetic.new/anthropic",
        apiKey: "${SYNTHETIC_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "hf:MiniMaxAI/MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 192000,
            maxTokens: 65536,
          },
        ],
      },
    },
  },
}
```

Base URL `/v1` içermemelidir (Anthropic istemcisi bunu otomatik olarak ekler). Kısayol: `openclaw onboard --auth-choice synthetic-api-key`.

</Accordion>

<Accordion title="MiniMax M2.1 (direct)">

```json5
{
  agents: {
    defaults: {
      model: { primary: "minimax/MiniMax-M2.1" },
      models: {
        "minimax/MiniMax-M2.1": { alias: "Minimax" },
      },
    },
  },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

`MINIMAX_API_KEY` ayarlayın. Kısayol: `openclaw onboard --auth-choice minimax-api`.

</Accordion>

<Accordion title="Local models (LM Studio)">

[Local Models](/gateway/local-models) bölümüne bakın. Özetle: MiniMax M2.1’i güçlü donanımda LM Studio Responses API ile çalıştırın; yedekleme için barındırılan modelleri birleştirilmiş şekilde tutun.

</Accordion>

---

## Skills

```json5
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills"],
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn
    },
    entries: {
      "nano-banana-pro": {
        apiKey: "GEMINI_KEY_HERE",
        env: { GEMINI_API_KEY: "GEMINI_KEY_HERE" },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

- `allowBundled`: yalnızca paketle gelen skills için isteğe bağlı izin listesi (yönetilen/çalışma alanı skills etkilenmez).
- `entries.<skillKey>``.enabled: false`, paketlenmiş/yüklenmiş olsa bile bir skill’i devre dışı bırakır.
- `entries.<skillKey>``.apiKey`: birincil ortam değişkeni tanımlayan skills için kolaylık sağlar.

---

## Plugins

```json5
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: [],
    load: {
      paths: ["~/Projects/oss/voice-call-extension"],
    },
    entries: {
      "voice-call": {
        enabled: true,
        config: { provider: "twilio" },
      },
    },
  },
}
```

- `~/.openclaw/extensions`, `<workspace>/.openclaw/extensions` ve `plugins.load.paths` içinden yüklenir.
- **Yapılandırma değişiklikleri gateway yeniden başlatmayı gerektirir.**
- `allow`: isteğe bağlı izin listesi (yalnızca listelenen eklentiler yüklenir). `deny` önceliklidir.

Bkz. [Plugins](/tools/plugin).

---

## Tarayıcı

```json5
{
  browser: {
    enabled: true,
    evaluateEnabled: true,
    defaultProfile: "chrome",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
    color: "#FF4500",
    // headless: false,
    // noSandbox: false,
    // executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    // attachOnly: false,
  },
}
```

- `evaluateEnabled: false`, `act:evaluate` ve `wait --fn` komutlarını devre dışı bırakır.
- Uzak profiller yalnızca attach modundadır (başlat/durdur/sıfırla devre dışıdır).
- Otomatik algılama sırası: varsayılan tarayıcı Chromium tabanlıysa → Chrome → Brave → Edge → Chromium → Chrome Canary.
- Kontrol servisi: yalnızca loopback (port `gateway.port` değerinden türetilir, varsayılan `18791`).

---

## UI

```json5
{
  ui: {
    seamColor: "#FF4500",
    assistant: {
      name: "OpenClaw",
      avatar: "CB", // emoji, short text, image URL, or data URI
    },
  },
}
```

- `seamColor`: yerel uygulama UI chrome’u için vurgu rengi (Talk Mode balon tonu vb.).
- `assistant`: Control UI kimlik geçersiz kılma ayarı. Etkin agent kimliğine geri döner.

---

## Gateway

```json5
{
  gateway: {
    mode: "local", // local | remote
    port: 18789,
    bind: "loopback",
    auth: {
      mode: "token", // token | password | trusted-proxy
      token: "your-token",
      // password: "your-password", // or OPENCLAW_GATEWAY_PASSWORD
      // trustedProxy: { userHeader: "x-forwarded-user" }, // for mode=trusted-proxy; see /gateway/trusted-proxy-auth
      allowTailscale: true,
      rateLimit: {
        maxAttempts: 10,
        windowMs: 60000,
        lockoutMs: 300000,
        exemptLoopback: true,
      },
    },
    tailscale: {
      mode: "off", // off | serve | funnel
      resetOnExit: false,
    },
    controlUi: {
      enabled: true,
      basePath: "/openclaw",
      // root: "dist/control-ui",
      // allowInsecureAuth: false,
      // dangerouslyDisableDeviceAuth: false,
    },
    remote: {
      url: "ws://gateway.tailnet:18789",
      transport: "ssh", // ssh | direct
      token: "your-token",
      // password: "your-password",
    },
    trustedProxies: ["10.0.0.1"],
    tools: {
      // Additional /tools/invoke HTTP denies
      deny: ["browser"],
      // Remove tools from the default HTTP deny list
      allow: ["gateway"],
    },
  },
}
```

<Accordion title="Gateway field details">

- `mode`: `local` (gateway’i çalıştır) veya `remote` (uzak gateway’e bağlan). `local` olmadıkça Gateway başlatmayı reddeder.
- `port`: WS + HTTP için tek ve çoklanmış port. Öncelik sırası: `--port` > `OPENCLAW_GATEWAY_PORT` > `gateway.port` > `18789`.
- `bind`: `auto`, `loopback` (varsayılan), `lan` (`0.0.0.0`), `tailnet` (yalnızca Tailscale IP), veya `custom`.
- **Auth**: varsayılan olarak gereklidir. Loopback dışı bind işlemleri paylaşılan bir token/password gerektirir. Onboarding sihirbazı varsayılan olarak bir token oluşturur.
- `auth.mode: "trusted-proxy"`: kimlik doğrulamayı kimlik farkındalıklı bir reverse proxy’ye devredin ve `gateway.trustedProxies` içindeki kimlik başlıklarına güvenin (bkz. [Trusted Proxy Auth](/gateway/trusted-proxy-auth)).
- `auth.allowTailscale`: `true` olduğunda, Tailscale Serve kimlik başlıkları auth gereksinimini karşılar (`tailscale whois` ile doğrulanır). `tailscale.mode = "serve"` olduğunda varsayılan olarak `true` olur.
- `auth.rateLimit`: isteğe bağlı başarısız auth sınırlayıcısı. İstemci IP’si başına ve auth kapsamı başına uygulanır (shared-secret ve device-token bağımsız olarak izlenir). Engellenen denemeler `429` + `Retry-After` döndürür.
  - `auth.rateLimit.exemptLoopback` varsayılan olarak `true`’dur; localhost trafiğinin de hız sınırına tabi olmasını özellikle istiyorsanız (test kurulumları veya sıkı proxy dağıtımları için) `false` olarak ayarlayın.
- `tailscale.mode`: `serve` (yalnızca tailnet, loopback bind) veya `funnel` (herkese açık, auth gerektirir).
- `remote.transport`: `ssh` (varsayılan) veya `direct` (ws/wss). `direct` için `remote.url` değeri `ws://` veya `wss://` olmalıdır.
- `gateway.remote.token` yalnızca uzak CLI çağrıları içindir; yerel gateway auth’u etkinleştirmez.
- `trustedProxies`: TLS’i sonlandıran reverse proxy IP’leri. Yalnızca kontrolünüzde olan proxy’leri listeleyin.
- `gateway.tools.deny`: HTTP `POST /tools/invoke` için engellenen ek araç adları (varsayılan deny listesini genişletir).
- `gateway.tools.allow`: araç adlarını varsayılan HTTP deny listesinden kaldırır.

</Accordion>

### OpenAI uyumlu uç noktalar

- Chat Completions: varsayılan olarak devre dışıdır. `gateway.http.endpoints.chatCompletions.enabled: true` ile etkinleştirin.
- Responses API: `gateway.http.endpoints.responses.enabled`.
- Responses URL-girdi sıkılaştırması:
  - `gateway.http.endpoints.responses.maxUrlParts`
  - `gateway.http.endpoints.responses.files.urlAllowlist`
  - `gateway.http.endpoints.responses.images.urlAllowlist`

### Çoklu örnek izolasyonu

Tek bir ana makinede benzersiz portlar ve durum dizinleri ile birden fazla gateway çalıştırın:

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json \
OPENCLAW_STATE_DIR=~/.openclaw-a \
openclaw gateway --port 19001
```

Kolaylık bayrakları: `--dev` (`~/.openclaw-dev` + `19001` portunu kullanır), `--profile <name>` (`~/.openclaw-<name>` kullanır).

[Multiple Gateways](/gateway/multiple-gateways) bölümüne bakın.

---

## Hooks

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    maxBodyBytes: 262144,
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    allowedAgentIds: ["hooks", "main"],
    presets: ["gmail"],
    transformsDir: "~/.openclaw/hooks/transforms",
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "hooks",
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

Kimlik doğrulama: `Authorization: Bearer <token>` veya `x-openclaw-token: <token>`.

**Uç Noktalar:**

- `POST /hooks/wake` → `{ text, mode?: "now"|"next-heartbeat" }`
- `POST /hooks/agent` → `{ message, name?, agentId?, sessionKey?, wakeMode?, deliver?, channel?, to?, model?, thinking?, timeoutSeconds? }`
  - `sessionKey`, yalnızca `hooks.allowRequestSessionKey=true` olduğunda (varsayılan: `false`) istek gövdesinden kabul edilir.
- `POST /hooks/<name>` → `hooks.mappings` aracılığıyla çözülür

<Accordion title="Mapping details">

- `match.path`, `/hooks` sonrasındaki alt yolu eşleştirir (ör. `/hooks/gmail` → `gmail`).
- `match.source`, genel yollar için payload içindeki bir alanla eşleşir.
- `{{messages[0].subject}}` gibi şablonlar payload’dan okunur.
- `transform`, bir hook eylemi döndüren bir JS/TS modülünü işaret edebilir.
  - `transform.module` göreli bir yol olmalıdır ve `hooks.transformsDir` içinde kalır (mutlak yollar ve dizin geçişleri reddedilir).
- `agentId`, belirli bir agente yönlendirir; bilinmeyen kimlikler varsayılana geri döner.
- `allowedAgentIds`: açık yönlendirmeyi kısıtlar (`*` veya boş bırakılırsa = tümüne izin ver, `[]` = tümünü reddet).
- `defaultSessionKey`: açık bir `sessionKey` olmadan çalışan hook agent çalıştırmaları için isteğe bağlı sabit oturum anahtarı.
- `allowRequestSessionKey`: `/hooks/agent` çağıranların `sessionKey` ayarlamasına izin verir (varsayılan: `false`).
- `allowedSessionKeyPrefixes`: açık `sessionKey` değerleri (istek + eşleme) için isteğe bağlı önek izin listesi, ör. `["hook:"]`.
- `deliver: true`, son yanıtı bir kanala gönderir; `channel` varsayılan olarak `last` değerini alır.
- `model`, bu hook çalıştırması için LLM’i geçersiz kılar (model kataloğu ayarlıysa izin verilmiş olmalıdır).

</Accordion>

### Gmail entegrasyonu

```json5
{
  hooks: {
    gmail: {
      account: "openclaw@gmail.com",
      topic: "projects/<project-id>/topics/gog-gmail-watch",
      subscription: "gog-gmail-watch-push",
      pushToken: "shared-push-token",
      hookUrl: "http://127.0.0.1:18789/hooks/gmail",
      includeBody: true,
      maxBytes: 20000,
      renewEveryMinutes: 720,
      serve: { bind: "127.0.0.1", port: 8788, path: "/" },
      tailscale: { mode: "funnel", path: "/gmail-pubsub" },
      model: "openrouter/meta-llama/llama-3.3-70b-instruct:free",
      thinking: "off",
    },
  },
}
```

- Yapılandırıldığında Gateway, açılışta `gog gmail watch serve` komutunu otomatik olarak başlatır. Devre dışı bırakmak için `OPENCLAW_SKIP_GMAIL_WATCHER=1` ayarlayın.
- Gateway ile birlikte ayrı bir `gog gmail watch serve` çalıştırmayın.

---

## Canvas ana makinesi

```json5
{
  canvasHost: {
    root: "~/.openclaw/workspace/canvas",
    liveReload: true,
    // enabled: false, // or OPENCLAW_SKIP_CANVAS_HOST=1
  },
}
```

- Gateway portu altında HTTP üzerinden ajan tarafından düzenlenebilir HTML/CSS/JS ve A2UI sunar:
  - `http://<gateway-host>:<gateway.port>/__openclaw__/canvas/`
  - `http://<gateway-host>:<gateway.port>/__openclaw__/a2ui/`
- Yalnızca yerel: `gateway.bind: "loopback"` (varsayılan) olarak bırakın.
- Loopback dışı bind işlemlerinde: canvas rotaları, diğer Gateway HTTP yüzeylerinde olduğu gibi Gateway kimlik doğrulaması (token/password/trusted-proxy) gerektirir.
- Node WebView’ler genellikle kimlik doğrulama başlıkları göndermez; bir node eşleştirilip bağlandıktan sonra, Gateway özel IP fallback’ine izin vererek node’un sırları URL’lere sızdırmadan canvas/A2UI yüklemesini sağlar.
- Sunulan HTML içine live-reload istemcisini enjekte eder.
- Boş olduğunda başlangıç `index.html` dosyasını otomatik oluşturur.
- Ayrıca A2UI’yi `/__openclaw__/a2ui/` altında sunar.
- Değişiklikler için gateway yeniden başlatması gerekir.
- Büyük dizinlerde veya `EMFILE` hatalarında live reload’u devre dışı bırakın.

---

## Keşif

### mDNS (Bonjour)

```json5
{
  discovery: {
    mdns: {
      mode: "minimal", // minimal | full | off
    },
  },
}
```

- `minimal` (varsayılan): TXT kayıtlarından `cliPath` + `sshPort` çıkarılır.
- `full`: `cliPath` + `sshPort` dahil edilir.
- Hostname varsayılan olarak `openclaw` değerine ayarlanır. `OPENCLAW_MDNS_HOSTNAME` ile geçersiz kılın.

### Geniş alan (DNS-SD)

```json5
{
  discovery: {
    wideArea: { enabled: true },
  },
}
```

`~/.openclaw/dns/` altında bir unicast DNS-SD zone yazar. Ağlar arası keşif için bir DNS sunucusu (CoreDNS önerilir) + Tailscale split DNS ile birlikte kullanın.

Kurulum: `openclaw dns setup --apply`.

---

## Ortam

### `env` (satır içi ortam değişkenleri)

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-...",
    },
    shellEnv: {
      enabled: true,
      timeoutMs: 15000,
    },
  },
}
```

- Satır içi ortam değişkenleri yalnızca süreç ortamında anahtar eksikse uygulanır.
- `.env` dosyaları: CWD `.env` + `~/.openclaw/.env` (hiçbiri mevcut değişkenlerin üzerine yazmaz).
- `shellEnv`: eksik beklenen anahtarları login shell profilinizden içe aktarır.
- Tam öncelik sırası için bkz. [Environment](/help/environment).

### Ortam değişkeni yerine koyma

Herhangi bir yapılandırma dizesinde ortam değişkenlerine `${VAR_NAME}` ile başvurun:

```json5
{
  gateway: {
    auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" },
  },
}
```

- Yalnızca büyük harfli adlar eşleşir: `[A-Z_][A-Z0-9_]*`.
- Eksik/boş değişkenler, yapılandırma yüklenirken hataya neden olur.
- Gerçek `${VAR}` ifadesi için `$${VAR}` kullanarak kaçış yapın.
- `$include` ile çalışır.

---

## Kimlik doğrulama depolama

```json5
{
  auth: {
    profiles: {
      "anthropic:me@example.com": { provider: "anthropic", mode: "oauth", email: "me@example.com" },
      "anthropic:work": { provider: "anthropic", mode: "api_key" },
    },
    order: {
      anthropic: ["anthropic:me@example.com", "anthropic:work"],
    },
  },
}
```

- Her aracı (agent) için kimlik doğrulama profilleri `<agentDir>/auth-profiles.json` içinde saklanır.
- `~/.openclaw/credentials/oauth.json` konumundan eski (legacy) OAuth içe aktarımları.
- [OAuth](/concepts/oauth) bölümüne bakın.

---

## Günlükleme

```json5
{
  logging: {
    level: "info",
    file: "/tmp/openclaw/openclaw.log",
    consoleLevel: "info",
    consoleStyle: "pretty", // pretty | compact | json
    redactSensitive: "tools", // off | tools
    redactPatterns: ["\\bTOKEN\\b\\s*[=:]\\s*([\"']?)([^\\s\"']+)\\1"],
  },
}
```

- Varsayılan günlük dosyası: `/tmp/openclaw/openclaw-YYYY-MM-DD.log`.
- Sabit bir yol için `logging.file` ayarlayın.
- `--verbose` kullanıldığında `consoleLevel` `debug` seviyesine yükseltilir.

---

## Sihirbaz

CLI sihirbazları (`onboard`, `configure`, `doctor`) tarafından yazılan meta veriler:

```json5
{
  wizard: {
    lastRunAt: "2026-01-01T00:00:00.000Z",
    lastRunVersion: "2026.1.4",
    lastRunCommit: "abc1234",
    lastRunCommand: "configure",
    lastRunMode: "local",
  },
}
```

---

## Kimlik

```json5
{
  agents: {
    list: [
      {
        id: "main",
        identity: {
          name: "Samantha",
          theme: "helpful sloth",
          emoji: "🦥",
          avatar: "avatars/samantha.png",
        },
      },
    ],
  },
}
```

macOS kurulum (onboarding) yardımcısı tarafından yazılır. Varsayılanları türetir:

- `messages.ackReaction`, `identity.emoji` değerinden türetilir (yoksa 👀 kullanılır)
- `mentionPatterns`, `identity.name`/`identity.emoji` değerlerinden türetilir
- `avatar` şunları kabul eder: çalışma alanına göreli yol, `http(s)` URL veya `data:` URI

---

## Bridge (eski, kaldırıldı)

Mevcut sürümler artık TCP bridge içermez. Düğümler Gateway WebSocket üzerinden bağlanır. `bridge.*` anahtarları artık yapılandırma şemasının bir parçası değildir (kaldırılana kadar doğrulama başarısız olur; `openclaw doctor --fix` bilinmeyen anahtarları temizleyebilir).

<Accordion title="Legacy bridge config (historical reference)">

```json
{
  "bridge": {
    "enabled": true,
    "port": 18790,
    "bind": "tailnet",
    "tls": {
      "enabled": true,
      "autoGenerate": true
    }
  }
}
```

</Accordion>

---

## Cron

```json5
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h", // duration string or false
  },
}
```

- `sessionRetention`: tamamlanan cron oturumlarının temizlenmeden önce ne kadar süre saklanacağını belirler. Varsayılan: `24h`.

[Cron Jobs](/automation/cron-jobs) bölümüne bakın.

---

## Medya model şablon değişkenleri

`tools.media.*.models[].args` içinde genişletilen şablon yer tutucuları:

| Değişken           | Açıklama                                                                              |
| ------------------ | ------------------------------------------------------------------------------------- |
| `{{Body}}`         | Tam gelen mesaj gövdesi                                                               |
| `{{RawBody}}`      | Ham gövde (geçmiş/gönderen sarmalayıcıları olmadan)                |
| `{{BodyStripped}}` | Grup bahsetmeleri çıkarılmış gövde                                                    |
| `{{From}}`         | Gönderen tanımlayıcısı                                                                |
| `{{To}}`           | Hedef tanımlayıcı                                                                     |
| `{{MessageSid}}`   | Kanal mesaj kimliği                                                                   |
| `{{SessionId}}`    | Geçerli oturum UUID’si                                                                |
| `{{IsNewSession}}` | Yeni oturum oluşturulduğunda "true"                                                   |
| `{{MediaUrl}}`     | Gelen medya sözde URL’si                                                              |
| `{{MediaPath}}`    | Yerel medya yolu                                                                      |
| `{{MediaType}}`    | Medya türü (image/audio/document/…)                                |
| `{{Transcript}}`   | Ses metin dökümü                                                                      |
| `{{Prompt}}`       | CLI girdileri için çözümlenmiş medya istemi                                           |
| `{{MaxChars}}`     | CLI girdileri için çözümlenmiş maksimum çıktı karakter sayısı                         |
| `{{ChatType}}`     | "direct" veya "group"                                                                 |
| `{{GroupSubject}}` | Grup konusu (mümkün olan en iyi şekilde)                           |
| `{{GroupMembers}}` | Grup üyeleri önizlemesi (mümkün olan en iyi şekilde)               |
| `{{SenderName}}`   | Gönderen görünen adı (mümkün olan en iyi şekilde)                  |
| `{{SenderE164}}`   | Gönderen telefon numarası (mümkün olan en iyi şekilde)             |
| `{{Provider}}`     | Sağlayıcı ipucu (whatsapp, telegram, discord, vb.) |

---

## Config includes (`$include`)

Yapılandırmayı birden fazla dosyaya bölün:

```json5
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/mueller.json5", "./clients/schmidt.json5"],
  },
}
```

**Birleştirme davranışı:**

- Tek dosya: İçeren nesnenin yerini alır.
- Dosya dizisi: Sırayla derinlemesine birleştirilir (sonraki, öncekini geçersiz kılar).
- Aynı düzeydeki anahtarlar: include’lardan sonra birleştirilir (include edilen değerleri geçersiz kılar).
- İç içe include’lar: En fazla 10 seviye derinliğe kadar.
- Yollar: göreli (dahil eden dosyaya göre), mutlak veya `../` üst dizin referansları.
- Hatalar: eksik dosyalar, ayrıştırma hataları ve döngüsel dahil etmeler için açık mesajlar.

---

_İlgili: [Configuration](/gateway/configuration) · [Configuration Examples](/gateway/configuration-examples) · [Doctor](/gateway/doctor)_
