---
summary: "Telegram bot destek durumu, yetenekler ve yapılandırma"
read_when:
  - Telegram özellikleri veya webhook’lar üzerinde çalışırken
title: "Telegram"
---

# Telegram (Bot API)

Durum: grammY üzerinden bot DM’leri + gruplar için production‑ready. Varsayılan olarak long‑polling; webhook isteğe bağlıdır.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Eşleştirme, Telegram DM’leri için varsayılan belirteç değişimidir.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallar arası tanılama ve onarım kılavuzları.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Tüm kanal yapılandırma desenleri ve örnekleri.
  
</Card>
</CardGroup>

## Hızlı kurulum (başlangıç)

<Steps>
  <Step title="Create the bot token in BotFather">Telegram’ı açın ve **@BotFather** ile sohbet edin ([doğrudan bağlantı](https://t.me/BotFather)).

    ```
    `/newbot` komutunu çalıştırın, yönlendirmeleri izleyin ve token’ı kaydedin.
    ```

  
</Step>

  <Step title="Configure token and DM policy">

```json5
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",
      groups: { "*": { requireMention: true } },
    },
  },
}
```

    ```
    Ortam değişkeni geri dönüşü: `TELEGRAM_BOT_TOKEN=...` (yalnızca varsayılan hesap).
    ```

  
</Step>

  <Step title="Start gateway and approve first DM">

```bash
`openclaw pairing approve telegram <CODE>`
```

    ```
    Eşleştirme kodları 1 saat sonra süresi dolar.
    ```

  
</Step>

  <Step title="Add the bot to a group">
    Botu grubunuza ekleyin, ardından erişim modelinize uygun olacak şekilde `channels.telegram.groups` ve `groupPolicy` ayarlarını yapın.
  
</Step>
</Steps>

<Note>
Token çözümleme sırası hesap farkındalıklıdır. Pratikte, yapılandırma değerleri ortam değişkeni geri dönüşüne üstün gelir ve `TELEGRAM_BOT_TOKEN` yalnızca varsayılan hesap için geçerlidir.
</Note>

## Telegram tarafı ayarları

<AccordionGroup>
  <Accordion title="Privacy mode and group visibility">Telegram botları varsayılan olarak **Gizlilik Modu** ile gelir; bu mod gruplarda hangi mesajları alabileceklerini sınırlar.

    ```
    `/setprivacy` — botun tüm grup mesajlarını görüp görmeyeceğini denetle.
    ```

  
</Accordion>

  <Accordion title="Group permissions">Yönetici durumu grup içinde (Telegram arayüzü) ayarlanır.

    ```
    Yönetici botlar her zaman tüm grup mesajlarını alır; tam görünürlük gerekiyorsa yöneticiyi kullanın.
    ```

  
</Accordion>

  <Accordion title="Helpful BotFather toggles">

    ```
    `/setjoingroups` — botun gruplara eklenmesine izin ver/engelle.
    ```

  
</Accordion>
</AccordionGroup>

## Erişim kontrolü ve etkinleştirme

<Tabs>
  <Tab title="DM policy">
    `channels.telegram.dmPolicy` doğrudan mesaj erişimini kontrol eder:


    ```
    - `pairing` (varsayılan)
    - `allowlist`
    - `open` (`allowFrom` içinde `"*"` bulunmasını gerektirir)
    - `disabled`
    
    `channels.telegram.allowFrom` sayısal Telegram kullanıcı kimliklerini kabul eder. `telegram:` / `tg:` önekleri kabul edilir ve normalize edilir.
    Karşılama sihirbazı `@username` girdisini kabul eder ve bunu sayısal kimliklere dönüştürür.
    Yükseltme yaptıysanız ve yapılandırmanızda `@username` allowlist girdileri varsa, bunları çözümlemek için `openclaw doctor --fix` çalıştırın (en iyi çaba; Telegram bot token’ı gerektirir).
    
    ### Telegram kullanıcı kimliğinizi bulma
    
    Daha güvenli (üçüncü taraf bot olmadan):
    
    1. Botunuza DM gönderin.
    2. `openclaw logs --follow` çalıştırın.
    3. `from.id` değerini okuyun.
    
    Resmi Bot API yöntemi:
    ```

```bash
curl "https://api.telegram.org/bot<bot_token>/getUpdates"
```

    ```
    Üçüncü taraf yöntemi (daha az gizli): `@userinfobot` veya `@getidsbot`.
    ```

  
</Tab>

  <Tab title="Group policy and allowlists">İki bağımsız denetim:

    ```
    {
      channels: {
        telegram: {
          groups: {
            "*": { requireMention: false }, // all groups, always respond
          },
        },
      },
    }
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "-1001234567890": {
          groupPolicy: "open",
          requireMention: false,
        },
      },
    },
  },
}
```

  
</Tab>

  <Tab title="Mention behavior">Grup yanıtları varsayılan olarak bir mention gerektirir (yerel @mention veya `agents.list[].groupChat.mentionPatterns` / `messages.groupChat.mentionPatterns`).

    ```
    Bahsetme şu yollardan gelebilir:
    
    - yerel `@botusername` bahsetmesi veya
    - şu desenlerdeki bahsetmeler:
      - `agents.list[].groupChat.mentionPatterns`
      - `messages.groupChat.mentionPatterns`
    
    Oturum düzeyinde komut anahtarları:
    
    - `/activation always`
    - `/activation mention`
    
    Bunlar yalnızca oturum durumunu günceller. Kalıcılık için yapılandırmayı kullanın.
    
    Kalıcı yapılandırma örneği:
    ```

```json5
{
  channels: {
    telegram: {
      groups: {
        "*": { requireMention: true }, // or omit groups entirely
      },
    },
  },
}
```

    ```
    Gruptaki herhangi bir mesajı Telegram’da `@userinfobot` veya `@getidsbot`’e iletin; sohbet kimliğini görürsünüz (ör. `-1001234567890` gibi negatif bir sayı).
    ```

  
</Tab>
</Tabs>

## Çalışma zamanı davranışı

- Gateway’e ait bir Telegram Bot API kanalıdır.
- Deterministik yönlendirme: yanıtlar Telegram’a geri döner; model kanal seçmez.
- Gelen mesajlar, yanıt bağlamı ve medya yer tutucuları ile paylaşılan kanal zarfına normalize edilir.
- Grup oturumları grup kimliğine göre izole edilir. Her konunun yalıtılması için Telegram grup oturum anahtarına `:topic:<threadId>` ekler.
- Yanıtların konu içinde kalması için yazıyor göstergeleri ve yanıtları `message_thread_id` ile gönderir.
- Uzun polling, sohbet/konu başına sıralama ile grammY runner kullanır. Long‑polling, sohbet başına sıralama ile grammY runner kullanır; genel eşzamanlılık `agents.defaults.maxConcurrent` ile sınırlandırılır.
- Telegram Bot API okundu bilgilerini desteklemez; `sendReadReceipts` seçeneği yoktur.

## Özellik referansı

<AccordionGroup>
  <Accordion title="Live stream preview (message edits)">OpenClaw, Telegram DM’lerinde `sendMessageDraft` kullanarak kısmi yanıtları akış halinde gönderebilir.

    ```
    Gereksinim:
    
    - `channels.telegram.streamMode` değeri `"off"` olmamalıdır (varsayılan: `"partial"`)
    
    Modlar:
    
    - `off`: canlı önizleme yok
    - `partial`: kısmi metinden sık önizleme güncellemeleri
    - `block`: `channels.telegram.draftChunk` kullanarak parçalı önizleme güncellemeleri
    
    `streamMode: "block"` için `draftChunk` varsayılanları:
    
    - `minChars: 200`
    - `maxChars: 800`
    - `breakPreference: "paragraph"`
    
    `maxChars`, `channels.telegram.textChunkLimit` tarafından sınırlandırılır.
    
    Bu, doğrudan sohbetlerde ve gruplarda/konularda çalışır.
    
    Yalnızca metin yanıtlarında OpenClaw aynı önizleme mesajını korur ve yerinde son düzenlemeyi yapar (ikinci mesaj yok).
    
    Karmaşık yanıtlar için (örneğin medya payload’ları), OpenClaw normal nihai gönderime geri döner ve ardından önizleme mesajını temizler.
    
    `streamMode`, blok akışından ayrıdır. Telegram için blok akışı açıkça etkinleştirildiğinde, OpenClaw çift akışı önlemek için önizleme akışını atlar.
    
    Yalnızca Telegram için muhakeme akışı:
    
    - `/reasoning stream` oluşturma sırasında muhakemeyi canlı önizlemeye gönderir
    - nihai yanıt muhakeme metni olmadan gönderilir
    ```

  
</Accordion>

  <Accordion title="Formatting and HTML fallback">Giden Telegram metni `parse_mode: "HTML"` kullanır (Telegram’ın desteklediği etiket alt kümesi).

    ```
    - Markdown benzeri metin Telegram uyumlu HTML’e dönüştürülür.
    - Ham model HTML’i, Telegram ayrıştırma hatalarını azaltmak için escape edilir.
    - Telegram ayrıştırılmış HTML’i reddederse, OpenClaw düz metin olarak yeniden dener.
    
    Bağlantı önizlemeleri varsayılan olarak etkindir ve `channels.telegram.linkPreview: false` ile devre dışı bırakılabilir.
    ```

  
</Accordion>

  <Accordion title="Native commands and custom commands">Bazı komutlar Telegram’ın komut menüsüne kaydedilmeden eklentiler/yetenekler tarafından işlenebilir.

    ```
    Yerel komut varsayılanları:
    
    - `commands.native: "auto"` Telegram için yerel komutları etkinleştirir
    
    Özel komut menü girdileri ekleyin:
    ```

```json5
{
  channels: {
    telegram: {
      customCommands: [
        { command: "backup", description: "Git backup" },
        { command: "generate", description: "Create an image" },
      ],
    },
  },
}
```

    ```
    Kurallar:
    
    - adlar normalize edilir (başındaki `/` kaldırılır, küçük harfe dönüştürülür)
    - geçerli desen: `a-z`, `0-9`, `_`, uzunluk `1..32`
    - özel komutlar yerel komutların üzerine yazamaz
    - çakışmalar/tekrarlar atlanır ve günlüğe kaydedilir
    
    Notlar:
    
    - özel komutlar yalnızca menü girdileridir; davranışı otomatik olarak uygulamazlar
    - eklenti/skill komutları Telegram menüsünde gösterilmese bile yazıldığında çalışabilir
    
    Yerel komutlar devre dışı bırakılırsa, yerleşik komutlar kaldırılır. Yapılandırılmışsa özel/eklenti komutları yine de kaydedilebilir.
    
    Yaygın kurulum hatası:
    
    - `setMyCommands failed` genellikle `api.telegram.org` adresine giden DNS/HTTPS çıkışının engellendiği anlamına gelir.
    
    ### Cihaz eşleştirme komutları (`device-pair` eklentisi)
    
    `device-pair` eklentisi yüklü olduğunda:
    
    1. `/pair` kurulum kodu üretir
    2. kodu iOS uygulamasına yapıştırın
    3. `/pair approve` en son bekleyen isteği onaylar
    
    Daha fazla ayrıntı: [Eşleştirme](/channels/pairing#pair-via-telegram-recommended-for-ios).
    ```

  
</Accordion>

  <Accordion title="Inline buttons">
    Satır içi klavye kapsamını yapılandırın:

```json5
{
  channels: {
    telegram: {
      capabilities: {
        inlineButtons: "allowlist",
      },
    },
  },
}
```

    ```
    Hesap başına yapılandırma için:
    ```

```json5
{
  channels: {
    telegram: {
      accounts: {
        main: {
          capabilities: {
            inlineButtons: "allowlist",
          },
        },
      },
    },
  },
}
```

    ```
    `"disabled"` = grup mesajları hiç kabul edilmez
      Varsayılan `groupPolicy: "allowlist"`’tir (`groupAllowFrom` eklemediğiniz sürece engelli).
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  message: "Choose an option:",
  buttons: [
    [
      { text: "Yes", callback_data: "yes" },
      { text: "No", callback_data: "no" },
    ],
    [{ text: "Cancel", callback_data: "cancel" }],
  ],
}
```

    ```
    Bir kullanıcı bir düğmeye tıkladığında, geri çağırım verisi ajana şu biçimde bir mesaj olarak gönderilir:
    `callback_data: value`
    ```

  
</Accordion>

  <Accordion title="Telegram message actions for agents and automation">
    Telegram araç eylemleri şunları içerir:

    ```
    - `sendMessage` (`to`, `content`, isteğe bağlı `mediaUrl`, `replyToMessageId`, `messageThreadId`)
    - `react` (`chatId`, `messageId`, `emoji`)
    - `deleteMessage` (`chatId`, `messageId`)
    - `editMessage` (`chatId`, `messageId`, `content`)
    
    Kanal mesaj eylemleri ergonomik takma adlar sunar (`send`, `react`, `delete`, `edit`, `sticker`, `sticker-search`).
    
    Erişim kontrolü ayarları:
    
    - `channels.telegram.actions.sendMessage`
    - `channels.telegram.actions.editMessage`
    - `channels.telegram.actions.deleteMessage`
    - `channels.telegram.actions.reactions`
    - `channels.telegram.actions.sticker` (varsayılan: devre dışı)
    
    Tepki kaldırma semantiği: [/tools/reactions](/tools/reactions)
    ```

  
</Accordion>

  <Accordion title="Reply threading tags">Telegram, etiketler aracılığıyla isteğe bağlı iş parçacıklı yanıtları destekler:

    ```
    - `[[reply_to_current]]` tetikleyen mesaja yanıt verir
    - `[[reply_to:<id>]]` belirli bir Telegram mesaj kimliğine yanıt verir
    
    `channels.telegram.replyToMode` işleme şeklini kontrol eder:
    
    - `off` (varsayılan)
    - `first`
    - `all`
    
    Not: `off`, örtük yanıt zincirlemeyi devre dışı bırakır. Açık `[[reply_to_*]]` etiketleri yine de dikkate alınır.
    ```

  
</Accordion>

  <Accordion title="Forum topics and thread behavior">Konular (forum süper grupları)

    ```
    - konu oturum anahtarları `:topic:<threadId>` ekler
    - yanıtlar ve yazıyor durumu konu başlığını hedefler
    - konu yapılandırma yolu:
      `channels.telegram.groups.<chatId>.topics.<threadId>`
    
    Genel konu (`threadId=1`) özel durumu:
    
    - mesaj gönderimleri `message_thread_id` içermez (Telegram `sendMessage(...thread_id=1)` çağrısını reddeder)
    - yazıyor eylemleri yine de `message_thread_id` içerir
    
    Konu kalıtımı: konu girdileri, üzerine yazılmadıkça grup ayarlarını devralır (`requireMention`, `allowFrom`, `skills`, `systemPrompt`, `enabled`, `groupPolicy`).
    
    Şablon bağlamı şunları içerir:
    
    - `MessageThreadId`
    - `IsForum`
    
    DM konu davranışı:
    
    - `message_thread_id` içeren özel sohbetler DM yönlendirmesini korur ancak konuya duyarlı oturum anahtarları/yanıt hedefleri kullanır.
    ```

  
</Accordion>

  <Accordion title="Audio, video, and stickers">    ### Sesli mesajlar

    ```
    Telegram sesli notlar ile ses dosyalarını ayırt eder.
    
    - varsayılan: ses dosyası davranışı
    - yanıt içinde `[[audio_as_voice]]` etiketi, sesli not olarak göndermeyi zorlar
    
    Mesaj eylemi örneği:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/voice.ogg",
  asVoice: true,
}
```

    ```
    ### Video mesajları
    
    Telegram video dosyaları ile video notlarını ayırt eder.
    
    Mesaj eylemi örneği:
    ```

```json5
{
  action: "send",
  channel: "telegram",
  to: "123456789",
  media: "https://example.com/video.mp4",
  asVideoNote: true,
}
```

    ```
    Video notları açıklama metnini desteklemez; sağlanan mesaj metni ayrı olarak gönderilir.
    
    ### Çıkartmalar
    
    Gelen çıkartma işleme:
    
    - statik WEBP: indirilir ve işlenir (yer tutucu `<media:sticker>`)
    - animasyonlu TGS: atlanır
    - video WEBM: atlanır
    
    Çıkartma bağlam alanları:
    
    - `Sticker.emoji`
    - `Sticker.setName`
    - `Sticker.fileId`
    - `Sticker.fileUniqueId`
    - `Sticker.cachedDescription`
    
    Çıkartma önbellek dosyası:
    
    - `~/.openclaw/telegram/sticker-cache.json`
    
    Çıkartmalar mümkün olduğunda bir kez açıklanır ve tekrarlanan görsel çağrılarını azaltmak için önbelleğe alınır.
    
    Çıkartma eylemlerini etkinleştir:
    ```

```json5
{
  channels: {
    telegram: {
      actions: {
        sticker: true,
      },
    },
  },
}
```

    ```
    Sending stickers
    ```

```json5
{
  action: "sticker",
  channel: "telegram",
  to: "123456789",
  fileId: "CAACAgIAAxkBAAI...",
}
```

    ```
    Sticker cache
    ```

```json5
{
  action: "sticker-search",
  channel: "telegram",
  query: "cat waving",
  limit: 5,
}
```

  
</Accordion>

  <Accordion title="Reaction notifications">Telegram API’den `message_reaction` güncellemesini alır

    ```
    Etkinleştirildiğinde, OpenClaw aşağıdaki gibi sistem olaylarını kuyruğa alır:
    
    - `Telegram reaction added: 👍 by Alice (@alice) on msg 42`
    
    Yapılandırma:
    
    - `channels.telegram.reactionNotifications`: `off | own | all` (varsayılan: `own`)
    - `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` (varsayılan: `minimal`)
    
    Notlar:
    
    - `own`, yalnızca bot tarafından gönderilen mesajlara verilen kullanıcı tepkileri anlamına gelir (gönderilen mesaj önbelleği üzerinden en iyi çaba ile).
    - Telegram tepki güncellemelerinde konu (thread) kimlikleri sağlamaz.
      - forum olmayan gruplar grup sohbeti oturumuna yönlendirilir
      - forum grupları, tam olarak kaynak konu yerine grubun genel konu oturumuna (`:topic:1`) yönlendirilir
    
    Polling/webhook için `allowed_updates`, `message_reaction` öğesini otomatik olarak içerir.
    ```

  
</Accordion>

  <Accordion title="Ack reactions">**Tepkiler nasıl çalışır:**
Telegram tepkileri, mesaj yüklerinde özellik olarak değil, **ayrı `message_reaction` olayları** olarak gelir. Bir kullanıcı tepki eklediğinde OpenClaw:

    ```
    Çözümleme sırası:
    
    - `channels.telegram.accounts.<accountId>.ackReaction`
    - `channels.telegram.ackReaction`
    - `messages.ackReaction`
    - agent kimlik emojisi geri dönüşü (`agents.list[].identity.emoji`, aksi halde "👀")
    
    Notlar:
    
    - Telegram unicode emoji bekler (örneğin "👀").
    - Bir kanal veya hesap için tepkiyi devre dışı bırakmak amacıyla `""` kullanın.
    ```

  
</Accordion>

  <Accordion title="Config writes from Telegram events and commands">    Kanal yapılandırma yazımları varsayılan olarak etkindir (`configWrites !== false`).

    ```
    Telegram tarafından tetiklenen yazımlar şunları içerir:
    
    - `channels.telegram.groups` değerini güncellemek için grup taşıma olayları (`migrate_to_chat_id`)
    - `/config set` ve `/config unset` (komutun etkinleştirilmesini gerektirir)
    
    Devre dışı bırak:
    ```

```json5
{
  channels: { telegram: { configWrites: false } },
}
```

  
</Accordion>

  <Accordion title="Long polling vs webhook">    Varsayılan: long polling.

    ```
    Genel URL’niz farklıysa, bir ters vekil kullanın ve `channels.telegram.webhookUrl`’i genel uç noktaya yönlendirin.
    ```

  
</Accordion>

  <Accordion title="Limits, retry, and CLI targets">
    Giden metin `channels.telegram.textChunkLimit`’a bölünür (varsayılan 4000).
    İsteğe bağlı yeni satıra göre bölme: uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `channels.telegram.chunkMode="newline"` ayarlayın.
    Medya indirme/yükleme `channels.telegram.mediaMaxMb` ile sınırlandırılır (varsayılan 5).
    - `channels.telegram.timeoutSeconds`, Telegram API istemci zaman aşımını geçersiz kılar (ayarlanmamışsa grammY varsayılanı geçerlidir).
    Grup geçmişi bağlamı `channels.telegram.historyLimit` (veya `channels.telegram.accounts.*.historyLimit`) kullanır; aksi halde `messages.groupChat.historyLimit`’ya düşer. Devre dışı bırakmak için `0` ayarlayın (varsayılan 50).
    DM geçmişi `channels.telegram.dmHistoryLimit` (kullanıcı dönüşleri) ile sınırlandırılabilir. Kullanıcı başına geçersiz kılmalar: `channels.telegram.dms["<user_id>"].historyLimit`.<user_id>"].historyLimit`
    - giden Telegram API yeniden denemeleri `channels.telegram.retry` ile yapılandırılabilir.

    ```
    CLI gönderim hedefi sayısal sohbet kimliği veya kullanıcı adı olabilir:
    ```

```bash
Örnek: `openclaw message send --channel telegram --target 123456789 --message "hi"`.
```

  
</Accordion>
</AccordionGroup>

## Sorun giderme

<AccordionGroup>
  <Accordion title="Bot does not respond to non mention group messages">

    ```
    - `requireMention=false` ise, Telegram gizlilik modu tam görünürlüğe izin vermelidir.
      - BotFather: `/setprivacy` -> Disable
      - ardından botu gruptan kaldırıp yeniden ekleyin
    - `openclaw channels status`, yapılandırma bahsedilmemiş grup mesajlarını bekliyorsa uyarı verir.
    - `openclaw channels status --probe`, açık sayısal grup kimliklerini kontrol edebilir; joker karakter `"*"` için üyelik sorgulaması yapılamaz.
    - hızlı oturum testi: `/activation always`.
    ```

  
</Accordion>

  <Accordion title="Bot not seeing group messages at all">

    ```
    - `channels.telegram.groups` mevcutsa, grup listelenmiş olmalıdır (veya `"*"` içermelidir)
    - botun gruptaki üyeliğini doğrulayın
    - atlama nedenleri için günlükleri inceleyin: `openclaw logs --follow`
    ```

  
</Accordion>

  <Accordion title="Commands work partially or not at all">

    ```
    - gönderici kimliğinizi yetkilendirin (eşleştirme ve/veya sayısal `allowFrom`)
    - grup ilkesi `open` olsa bile komut yetkilendirmesi geçerlidir
    - `setMyCommands failed` genellikle `api.telegram.org` adresine DNS/HTTPS erişim sorunlarını gösterir
    ```

  
</Accordion>

  <Accordion title="Polling or network instability">

    ```
    - Node 22+ + özel fetch/proxy, AbortSignal türleri uyuşmazsa anında iptal davranışını tetikleyebilir.
    - Bazı barındırma ortamları `api.telegram.org` adresini önce IPv6’ya çözümler; bozuk IPv6 çıkışı, aralıklı Telegram API hatalarına neden olabilir.
    - DNS yanıtlarını doğrulayın:
    ```

```bash
dig +short api.telegram.org A
dig +short api.telegram.org AAAA
```

  
</Accordion>
</AccordionGroup>

Daha fazla yardım: [Kanal sorun giderme](/channels/troubleshooting).

## Telegram yapılandırma referans bağlantıları

Birincil referans:

- `channels.telegram.enabled`: kanal başlangıcını etkinleştir/devre dışı bırak.

- `channels.telegram.botToken`: bot belirteci (BotFather).

- `channels.telegram.tokenFile`: belirteci dosya yolundan oku.

- `channels.telegram.dmPolicy`: `pairing | allowlist | open | disabled` (varsayılan: eşleştirme).

- `channels.telegram.allowFrom`: DM izin listesi (kimlikler/kullanıcı adları). `open`, `"*"` gerektirir. `openclaw doctor --fix`, eski `@username` girdilerini kimliklere dönüştürebilir.

- `channels.telegram.groupPolicy`: `open | allowlist | disabled` (varsayılan: izin listesi).

- `channels.telegram.groupAllowFrom`: grup gönderen izin listesi (kimlikler/kullanıcı adları). `openclaw doctor --fix`, eski `@username` girdilerini kimliklere dönüştürebilir.

- `channels.telegram.groups`: grup başına varsayılanlar + izin listesi (genel varsayılanlar için `"*"` kullanın).
  - `channels.telegram.groups.<id>.groupPolicy`: groupPolicy için grup başına geçersiz kılma (`open | allowlist | disabled`).
  - `channels.telegram.groups.<id>.requireMention`: mention kapısı varsayılanı.
  - `channels.telegram.groups.<id>.skills`: skill filtresi (yok = tüm Skills, boş = hiçbiri).
  - `channels.telegram.groups.<id>.allowFrom`: grup başına gönderen izin listesi geçersiz kılması.
  - `channels.telegram.groups.<id>.systemPrompt`: grup için ek sistem istemi.
  - `channels.telegram.groups.<id>.enabled`: `false` olduğunda grubu devre dışı bırak.
  - .topics.<threadId>`channels.telegram.groups.<id>.*`: konu başına geçersiz kılmalar (grup ile aynı alanlar).
  - `channels.telegram.groups.<id>.topics.<threadId>.groupPolicy`: groupPolicy için konu başına geçersiz kılma (`open | allowlist | disabled`).
  - .topics.<threadId>`channels.telegram.groups.<id>.requireMention`: konu başına mention kapısı geçersiz kılması.

- `channels.telegram.capabilities.inlineButtons`: `off | dm | group | all | allowlist` (varsayılan: izin listesi).

- `channels.telegram.accounts.<account>.capabilities.inlineButtons`: hesap başına geçersiz kılma.

- `channels.telegram.replyToMode`: `off | first | all` (varsayılan: `first`).

- `channels.telegram.textChunkLimit`: giden parça boyutu (karakter).

- `channels.telegram.chunkMode`: `length` (varsayılan) veya uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `newline`.

- `channels.telegram.linkPreview`: giden mesajlar için bağlantı önizlemelerini aç/kapat (varsayılan: true).

- `channels.telegram.streamMode`: `off | partial | block` (taslak akışı).

- `channels.telegram.mediaMaxMb`: gelen/giden medya üst sınırı (MB).

- `channels.telegram.retry`: giden Telegram API çağrıları için yeniden deneme ilkesi (denemeler, minDelayMs, maxDelayMs, jitter).

- `channels.telegram.network.autoSelectFamily`: Node autoSelectFamily geçersiz kılması (true=etkin, false=devre dışı). Node 22’de Happy Eyeballs zaman aşımını önlemek için varsayılan olarak devre dışıdır.

- `channels.telegram.proxy`: Bot API çağrıları için proxy URL’si (SOCKS/HTTP).

- `channels.telegram.webhookUrl`: webhook modunu etkinleştirir (`channels.telegram.webhookSecret` gerektirir).

- `channels.telegram.webhookSecret`: webhook gizlisi (webhookUrl ayarlıysa zorunlu).

- `channels.telegram.webhookPath`: yerel webhook yolu (varsayılan `/telegram-webhook`).

- Yerel dinleyici `0.0.0.0:8787`’e bağlanır ve varsayılan olarak `POST /telegram-webhook`’ü sunar.

- `channels.telegram.actions.reactions`: Telegram araç tepkilerini kapıla.

- `channels.telegram.actions.sendMessage`: Telegram araç mesaj gönderimlerini kapıla.

- `channels.telegram.actions.deleteMessage`: Telegram araç mesaj silmelerini kapıla.

- `channels.telegram.actions.sticker`: Telegram çıkartma eylemlerini kapıla — gönderme ve arama (varsayılan: false).

- `channels.telegram.reactionNotifications`: `off | own | all` — hangi tepkilerin sistem olaylarını tetikleyeceğini denetler (ayarlı değilse varsayılan: `own`).

- `channels.telegram.reactionLevel`: `off | ack | minimal | extensive` — ajanın tepki yeteneğini denetler (ayarlı değilse varsayılan: `minimal`).

- [Yapılandırma referansı - Telegram](/gateway/configuration-reference#telegram)

Telegram’a özgü yüksek öneme sahip alanlar:

- başlangıç/yetkilendirme: `enabled`, `botToken`, `tokenFile`, `accounts.*`
- erişim kontrolü: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`, `groups.*.topics.*`
- komut/menü: `commands.native`, `customCommands`
- iş parçacığı/yanıtlar: `replyToMode`
- İsteğe bağlı (yalnızca `streamMode: "block"` için):
- biçimlendirme/teslimat: `textChunkLimit`, `chunkMode`, `linkPreview`, `responsePrefix`
- medya/ağ: `mediaMaxMb`, `timeoutSeconds`, `retry`, `network.autoSelectFamily`, `proxy`
- Webhook modu: `channels.telegram.webhookUrl` ve `channels.telegram.webhookSecret` ayarlayın (isteğe bağlı `channels.telegram.webhookPath`).
- eylemler/yetenekler: `capabilities.inlineButtons`, `actions.sendMessage|editMessage|deleteMessage|reactions|sticker`
- `channels.telegram.reactionLevel`: Ajanın tepki yeteneğini denetler
- yazımlar/geçmiş: `configWrites`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`

## İlgili

- Ayrıntılar: [Eşleştirme](/channels/pairing)
- [Kanal yönlendirme](/channels/channel-routing)
- Kurulum sorun giderme (komutlar)
