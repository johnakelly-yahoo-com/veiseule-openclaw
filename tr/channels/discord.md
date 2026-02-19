---
summary: "Discord botu destek durumu, yetenekleri ve yapılandırması"
read_when:
  - Discord kanal özellikleri üzerinde çalışırken
title: "Discord"
---

# Discord (Bot API)

Durum: Resmî Discord bot gateway üzerinden DM ve sunucu (guild) metin kanalları için hazır.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Discord DM’leri varsayılan olarak eşleştirme modundadır.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Yerel komut davranışı ve komut kataloğu.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallar arası tanılama ve onarım akışı.
  
</Card>
</CardGroup>

## Hızlı kurulum (başlangıç seviyesi)

<Steps>
  <Step title="Create a Discord bot and enable intents">Discord Developer Portal’da bir uygulama oluşturun, bir bot ekleyin, ardından şunları etkinleştirin:

    ```
    Discord uygulama ayarlarında **Message Content Intent**’i (ve izin listeleri veya ad aramaları kullanacaksanız **Server Members Intent**’i) etkinleştirin.
    ```

  
</Step>

  <Step title="Configure token">

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

    ```
    Varsayılan hesap için ortam değişkeni geri dönüşü:
    ```

```bash
`DISCORD_BOT_TOKEN=...`
```

  
</Step>

  <Step title="Invite the bot and start gateway">Botu, kullanmak istediğiniz yerlerde mesaj okuma/gönderme izinleriyle sunucunuza davet edin.

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first DM pairing">

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

    ```
    Eşleştirme kodları 1 saat sonra süresi dolar.
    ```

  
</Step>
</Steps>

<Note>
Token çözümlemesi hesap farkındalıklıdır. Yapılandırmadaki token değerleri ortam değişkeni geri dönüşüne göre önceliklidir. `DISCORD_BOT_TOKEN` yalnızca varsayılan hesap için kullanılır.
</Note>

## Çalışma zamanı modeli

- Gateway, Discord bağlantısını yönetir.
- Yanıt yönlendirme deterministiktir: Discord’dan gelen yanıtlar tekrar Discord’a gider.
- `discord:`/`user:` (kullanıcılar) ve `channel:` (grup DM’leri) gibi önekler desteklenir.
- Doğrudan sohbetler ajanın ana oturumunda birleşir (varsayılan `agent:main:main`); sunucu kanalları `agent:<agentId>:discord:channel:<channelId>` olarak yalıtılmış kalır (görünen adlar `discord:<guildSlug>#<channelSlug>` kullanır).
- Grup DM’leri varsayılan olarak yok sayılır; `channels.discord.dm.groupEnabled` ile etkinleştirin ve isteğe bağlı olarak `channels.discord.dm.groupChannels` ile kısıtlayın.
- Yerel komutlar, paylaşılan `main` oturumu yerine yalıtılmış oturum anahtarları (`agent:<agentId>:discord:slash:<userId>`) kullanır.

## Erişim kontrolü ve yönlendirme

<Tabs>
  <Tab title="DM policy">Tüm DM’leri yok saymak için: `channels.discord.dm.enabled=false` veya `channels.discord.dm.policy="disabled"` ayarlayın.

    ```
    Katı izin listesi için: `channels.discord.dm.policy="allowlist"` ayarlayın ve gönderenleri `channels.discord.dm.allowFrom` içinde listeleyin.
    ```

  
</Tab>

  <Tab title="Guild policy">
    Sunucu (Guild) işleme `channels.discord.groupPolicy` tarafından kontrol edilir:

    ```
    Yerel komutlar, DM’ler/sunucu mesajları ile aynı izin listelerini uygular (`channels.discord.dm.allowFrom`, `channels.discord.guilds`, kanal başına kurallar).
    ```

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true },
          },
        },
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
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
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false,
        presence: false,
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["openclaw-dm"],
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-openclaw",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short.",
            },
          },
        },
      },
    },
  },
}
```

    ```
    Yalnızca `DISCORD_BOT_TOKEN` ayarlayıp hiç `channels.discord` bölümü oluşturmazsanız, çalışma zamanı
    `groupPolicy`’yi `open` olarak varsayar.
    ```

  
</Tab>

  <Tab title="Mentions and group DMs">
    Sunucu mesajları varsayılan olarak mention-gated yapıdadır.

    ```
    Mention algılama şunları içerir:
    
    - açık bot mention
    - yapılandırılmış mention desenleri (`agents.list[].groupChat.mentionPatterns`, geri dönüş olarak `messages.groupChat.mentionPatterns`)
    - desteklenen durumlarda bota örtük yanıt davranışı
    
    `requireMention`, her sunucu/kanal için ayrı yapılandırılır (`channels.discord.guilds...`).
    
    Grup DM’leri:
    
    - varsayılan: yok sayılır (`dm.groupEnabled=false`)
    - isteğe bağlı allowlist: `dm.groupChannels` (kanal kimlikleri veya slug’lar)
    ```

  
</Tab>
</Tabs>

### Role dayalı agent yönlendirme

Discord sunucu üyelerini rol kimliğine göre farklı agent’lara yönlendirmek için `bindings[].match.roles` kullanın. Role dayalı binding’ler yalnızca rol kimliklerini kabul eder ve peer veya parent-peer binding’lerinden sonra, yalnızca sunucuya özel binding’lerden önce değerlendirilir. Bir binding başka eşleşme alanları da ayarlıyorsa (örneğin `peer` + `guildId` + `roles`), yapılandırılmış tüm alanların eşleşmesi gerekir.

```json5
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## Developer Portal kurulumu

<AccordionGroup>
  <Accordion title="Create app and bot">

    ```
    Discord Developer Portal → **Applications** → **New Application**
    ```

  
</Accordion>

  <Accordion title="Privileged intents">**Bot** → **Privileged Gateway Intents** altında şunları etkinleştirin:

    ```
    Genellikle **Presence Intent** gerekmez. Botun kendi varlığını ayarlamak (`setPresence` eylemi) gateway OP3 kullanır ve bu intent’i gerektirmez; yalnızca diğer sunucu üyelerinin varlık güncellemelerini almak istiyorsanız gereklidir.
    ```

  
</Accordion>

  <Accordion title="OAuth scopes and baseline permissions">Uygulamanızda: **OAuth2** → **URL Generator**

    ```
    - kapsamlar: `bot`, `applications.commands`
    
    Tipik temel izinler:
    
    - Kanalları Görüntüle
    - Mesaj Gönder
    - Mesaj Geçmişini Oku
    - Bağlantı Göm
    - Dosya Ekle
    - Tepki Ekle (isteğe bağlı)
    
    Açıkça gerekmedikçe `Administrator` kullanmaktan kaçının.
    ```

  
</Accordion>

  <Accordion title="Copy IDs">
    Discord Developer Mode’u etkinleştirin, ardından şunları kopyalayın:

    ```
    - sunucu kimliği
    - kanal kimliği
    - kullanıcı kimliği
    
    Güvenilir denetimler ve prob işlemleri için OpenClaw yapılandırmasında sayısal kimlikleri tercih edin.
    ```

  
</Accordion>
</AccordionGroup>

## Yerel komutlar ve komut yetkilendirmesi

- `commands.native` varsayılan olarak `"auto"` değerindedir ve Discord için etkindir.
- Veya yapılandırma: `channels.discord.token: "..."`.
- `channels.discord.commands.native: true|false|"auto"` ile geçersiz kılın; `false` daha önce kaydedilmiş komutları temizler.
- Yerel komut yetkilendirmesi, normal mesaj işleme ile aynı Discord allowlist/politikalarını kullanır.
- Slash komutları, izin listesinde olmayan kullanıcılara Discord UI’da yine de görünebilir; OpenClaw yürütmede izin listelerini uygular ve “yetkili değil” yanıtını verir.

Daha geniş onay ve komut akışı için [Exec approvals](/tools/exec-approvals) ve [Slash commands](/tools/slash-commands) bölümlerine bakın.

## Özellik ayrıntıları

<AccordionGroup>
  <Accordion title="Reply tags and native replies">
    Discord, agent çıktısında yanıt etiketlerini destekler:

    ```
    - `[[reply_to_current]]`
    - `[[reply_to:<id>]]`
    
    `channels.discord.replyToMode` tarafından kontrol edilir:
    
    - `off` (varsayılan)
    - `first`
    - `all`
    
    Not: `off`, örtük yanıt iş parçacığını devre dışı bırakır. Açık `[[reply_to_*]]` etiketleri yine de uygulanır.
    
    Mesaj kimlikleri, agent’ların belirli mesajları hedefleyebilmesi için bağlam/geçmişte sunulur.
    ```

  
</Accordion>

  <Accordion title="History, context, and thread behavior">
    Sunucu geçmişi bağlamı:

    ```
    - `channels.discord.historyLimit` varsayılan `20`
    - geri dönüş: `messages.groupChat.historyLimit`
    - `0` devre dışı bırakır
    
    DM geçmişi kontrolleri:
    
    - `channels.discord.dmHistoryLimit`
    - `channels.discord.dms["<user_id>"].historyLimit`
    
    İş parçacığı davranışı:
    
    - Discord iş parçacıkları kanal oturumları olarak yönlendirilir
    - üst iş parçacığı meta verileri, üst oturum bağlantısı için kullanılabilir
    - iş parçacığı yapılandırması, iş parçacığına özel bir giriş yoksa üst kanal yapılandırmasını devralır
    
    Kanal konuları **güvenilmeyen** bağlam olarak eklenir (system prompt olarak değil).
    ```

  
</Accordion>

  <Accordion title="Reaction notifications">.reactionNotifications` kullanır:

    ```
    `guilds.<id> .reactionNotifications`: tepki sistemi olay modu (`off`, `own`, `all`, `allowlist`).
    ```

  
</Accordion>

  <Accordion title="Ack reactions">
    `ackReaction`, OpenClaw gelen bir mesajı işlerken bir onay emojisi gönderir.

    ```
    Çözümleme sırası:
    
    - `channels.discord.accounts.<accountId>.ackReaction`
    - `channels.discord.ackReaction`
    - `messages.ackReaction`
    - agent kimlik emojisi geri dönüşü (`agents.list[].identity.emoji`, aksi halde "👀")
    
    Notlar:
    
    - Discord, unicode emoji veya özel emoji adlarını kabul eder.
    - Bir kanal veya hesap için tepkiyi devre dışı bırakmak üzere `""` kullanın.
    ```

  
</Accordion>

  <Accordion title="Config writes">`channels` mevcutsa, listelenmeyen herhangi bir kanal varsayılan olarak reddedilir.

    ```
    Bu, `/config set|unset` akışlarını etkiler (komut özellikleri etkin olduğunda).
    
    Devre dışı bırak:
    ```

```json5
{
  channels: { discord: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Gateway proxy">
    Discord gateway WebSocket trafiğini `channels.discord.proxy` ile bir HTTP(S) proxy üzerinden yönlendirin.

```json5
PK aramaları başarısız olursa (ör. belirteci olmayan özel sistem), proxy’li mesajlar
  bot mesajları olarak değerlendirilir ve `channels.discord.allowBots=true` yoksa düşürülür.
```

    ```
    Hesap başına geçersiz kılma:
    ```

```json5
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

  
</Accordion>

  <Accordion title="PluralKit support">`pluralkit`: PluralKit proxy’li mesajları çözerek sistem üyelerinin ayrı göndericiler olarak görünmesini sağlar.

```json5
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // optional; required for private systems
      },
    },
  },
}
```

    ```
    Notlar:
    
    - allowlist’ler `pk:<memberId>` kullanabilir
    - üye görünen adları ad/slug ile eşleştirilir
    - aramalar orijinal mesaj kimliğini kullanır ve zaman aralığıyla sınırlıdır
    - arama başarısız olursa, proxy edilen mesajlar bot mesajı olarak değerlendirilir ve `allowBots=true` değilse yok sayılır
    ```

  
</Accordion>

  <Accordion title="Presence configuration">
    Presence güncellemeleri yalnızca bir durum veya etkinlik alanı ayarladığınızda uygulanır.

    ```
    Yalnızca durum örneği:
    ```

```json5
`channelInfo`, `channelList`, `voiceStatus`, `eventList`, `eventCreate`
```

    ```
    Etkinlik örneği (özel durum varsayılan etkinlik türüdür):
    ```

```json5
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

    ```
    Yayın (streaming) örneği:
    ```

```json5
Eski “herkese açık” davranışı sürdürmek için: `channels.discord.dm.policy="open"` ve `channels.discord.dm.allowFrom=["*"]` ayarlayın.
```

    ```
    Etkinlik türü haritası:
    
    - 0: Playing
    - 1: Streaming (`activityUrl` gerektirir)
    - 2: Listening
    - 3: Watching
    - 4: Custom (etkinlik metnini durum state’i olarak kullanır; emoji isteğe bağlıdır)
    - 5: Competing
    ```

  
</Accordion>

  <Accordion title="Exec approvals in Discord">
    Discord, DM’lerde buton tabanlı exec onaylarını destekler ve isteğe bağlı olarak onay istemlerini başlatılan kanalda yayınlayabilir.

    ```
    Yapılandırma yolu:
    
    - `channels.discord.execApprovals.enabled`
    - `channels.discord.execApprovals.approvers`
    - `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, varsayılan: `dm`)
    - `agentFilter`, `sessionFilter`, `cleanupAfterResolve`
    
    `target` değeri `channel` veya `both` olduğunda, onay istemi kanalda görünür. Yalnızca yapılandırılmış onaylayıcılar butonları kullanabilir; diğer kullanıcılar geçici (ephemeral) bir reddetme mesajı alır. Onay istemleri komut metnini içerir, bu nedenle kanal iletimini yalnızca güvenilir kanallarda etkinleştirin. Kanal kimliği oturum anahtarından türetilemezse, OpenClaw DM iletimine geri döner.
    
    Onaylar bilinmeyen onay kimlikleri hatasıyla başarısız olursa, onaylayıcı listesini ve özelliğin etkinleştirildiğini doğrulayın.
    
    İlgili dokümanlar: [Exec approvals](/tools/exec-approvals)
    ```

  
</Accordion>
</AccordionGroup>

## Tool action defaults

Discord mesaj eylemleri; mesajlaşma, kanal yönetimi, moderasyon, durum (presence) ve meta veri eylemlerini içerir.

Temel örnekler:

- `readMessages`, `sendMessage`, `editMessage`, `deleteMessage`
- Tepki ver + tepkileri listele + emojiList
- `timeout`, `kick`, `ban`
- presence: `setPresence`

Eylem kapıları `channels.discord.actions.*` altında bulunur.

Varsayılan kapı davranışı:

| Eylem grubu                                                                                                   | Varsayılan |
| ------------------------------------------------------------------------------------------------------------- | ---------- |
| `stickers`, `emojiUploads`, `stickerUploads`, `polls`, `permissions`, `messages`, `threads`, `pins`, `search` | enabled    |
| roleInfo                                                                                                      | devre dışı |
| moderasyon                                                                                                    | devre dışı |
| presence                                                                                                      | devre dışı |

## Components v2 UI

OpenClaw, exec approvals ve bağlamlar arası işaretleyiciler için Discord components v2 kullanır. Discord mesaj eylemleri, özel UI için `components` da kabul edebilir (gelişmiş; Carbon bileşen örnekleri gerektirir); eski `embeds` hâlâ kullanılabilir ancak önerilmez.

- `channels.discord.ui.components.accentColor`, Discord bileşen kapsayıcıları tarafından kullanılan vurgu rengini (hex) ayarlar.
- Hesap bazında `channels.discord.accounts.<id>.ui.components.accentColor` ile ayarlayın.
- components v2 mevcutsa `embeds` yok sayılır.

Örnek:

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        YOUR_GUILD_ID: {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true },
          },
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

## messages

Discord sesli mesajları dalga formu önizlemesi gösterir ve OGG/Opus ses ile meta veri gerektirir. OpenClaw dalga formunu otomatik olarak oluşturur, ancak ses dosyalarını incelemek ve dönüştürmek için gateway sunucusunda `ffmpeg` ve `ffprobe` bulunmalıdır.

Yetenekler ve sınırlar

- **Yerel bir dosya yolu** sağlayın (URL’ler reddedilir).
- Metin içeriğini eklemeyin (Discord aynı payload içinde metin + sesli mesaja izin vermez).
- Herhangi bir ses formatı kabul edilir; gerekirse OpenClaw OGG/Opus formatına dönüştürür.

Örnek:

```bash
message(action="send", channel="discord", target="channel:123", path="/path/to/audio.mp3", asVoice=true)
```

## Sorun Giderme

<AccordionGroup>
  <Accordion title="Used disallowed intents or bot sees no guild messages">

    ```
    **“Used disallowed intents”**: Developer Portal’da **Message Content Intent**’i (ve büyük olasılıkla **Server Members Intent**’i) etkinleştirin, ardından gateway’i yeniden başlatın.
    ```

  
</Accordion>

  <Accordion title="Guild messages blocked unexpectedly">

    ```
    `channels.discord.groupPolicy` varsayılanı **allowlist**’tir; `"open"` olarak ayarlayın veya `channels.discord.guilds` altında bir sunucu girdisi ekleyin (isteğe bağlı olarak `channels.discord.guilds.<id> .channels` altında kanalları listeleyerek kısıtlayın).
    ```

```bash
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

  
</Accordion>

  <Accordion title="Require mention false but still blocked">
    Yaygın nedenler:

    ```
    `groupPolicy`: sunucu kanalı işleyişini kontrol eder (`open|disabled|allowlist`); `allowlist` kanal izin listeleri gerektirir.
    ```

  
</Accordion>

  <Accordion title="Permissions audit mismatches">**İzin denetimleri** (`channels status --probe`) yalnızca sayısal kanal kimliklerini kontrol eder.

    ```
    Slug anahtarları kullanıyorsanız, çalışma zamanı eşleştirmesi yine de çalışabilir; ancak probe izinleri tam olarak doğrulayamaz.
    ```

  
</Accordion>

  <Accordion title="DM and pairing issues">

    ```
    **DM’ler çalışmıyor**: `channels.discord.dm.enabled=false`, `channels.discord.dm.policy="disabled"` veya henüz onaylanmamışsınız (`channels.discord.dm.policy="pairing"`).
    ```

  
</Accordion>

  <Accordion title="Bot to bot loops">
    Varsayılan olarak bot tarafından yazılan mesajlar yok sayılır.

    ```
    Uyarı: Diğer botlara yanıt vermeye izin verirseniz (`channels.discord.allowBots=true`), botlar arası yanıt döngülerini `requireMention`, `channels.discord.guilds.*.channels.<id> .users` izin listeleri ve/veya `AGENTS.md` ve `SOUL.md` içindeki korumaları temizleyerek önleyin.
    ```

  
</Accordion>
</AccordionGroup>

## Yapılandırma referans bağlantıları

Birincil referans:

- [Configuration reference - Discord](/gateway/configuration-reference#discord)

Önemli Discord alanları:

- startup/auth: `enabled`, `token`, `accounts.*`, `allowBots`
- policy: `groupPolicy`, `dm.*`, `guilds.*`, `guilds.*.channels.*`
- Komutlar için erişim grubu kontrollerini atlamak üzere `commands.useAccessGroups: false` kullanın.
- reply/history: `replyToMode`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
- delivery: `textChunkLimit`, `chunkMode`, `maxLinesPerMessage`
- media/retry: `mediaMaxMb`, `retry`
- actions: `actions.*`
- presence: `activity`, `status`, `activityType`, `activityUrl`
- UI: `ui.components.accentColor`
- features: `pluralkit`, `execApprovals`, `intents`, `agentComponents`, `heartbeat`, `responsePrefix`

## Güvenli kullanım & operasyonlar

- Bot tokenlerini gizli bilgi olarak değerlendirin (denetimli ortamlarda `DISCORD_BOT_TOKEN` tercih edilir).
- En az ayrıcalık ilkesine göre Discord izinleri verin.
- Komut dağıtımı/durumu güncel değilse gateway’i yeniden başlatın ve `openclaw channels status --probe` ile tekrar kontrol edin.

## İlgili

- [Eşleştirme](/channels/pairing)
- [Kanal yönlendirme](/channels/channel-routing)
- [Sorun Giderme](/channels/troubleshooting)
- Tam komut listesi + yapılandırma: [Slash commands](/tools/slash-commands)

