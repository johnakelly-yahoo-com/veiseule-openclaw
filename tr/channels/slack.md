---
summary: "Socket veya HTTP webhook modu için Slack kurulumu"
read_when:
  - Slack kurulurken veya Slack socket/HTTP modu hata ayıklanırken
title: "Slack"
---

# Slack

Durum: Slack uygulama entegrasyonları aracılığıyla DM’ler + kanallar için üretime hazır. HTTP modu (Events API)

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">
    Slack DM’leri varsayılan olarak eşleştirme modundadır.
  
</Card>
  <Card title="Slash commands" icon="terminal" href="/tools/slash-commands">
    Yerel komut davranışı ve komut kataloğu.
  
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallar arası tanılama ve onarım kılavuzları.
  
</Card>
</CardGroup>

## Hızlı kurulum (başlangıç)

<Tabs>
  <Tab title="Socket Mode (default)">
    <Steps>
      <Step title="Create Slack app and tokens">        Slack uygulama ayarlarında:

        ```
        **Socket Mode** → açın. Ardından **Basic Information** → **App-Level Tokens** → `connections:write` kapsamı ile **Generate Token and Scopes**. **App Token**’ı (`xapp-...`) kopyalayın.
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      appToken: "xapp-...",
      botToken: "xoxb-...",
    },
  },
}
```

        ```
            Ortam değişkeni yedeği (yalnızca varsayılan hesap):
        ```

```bash
`SLACK_APP_TOKEN=xapp-...`
```

        
</Step>
      
        <Step title="Uygulama olaylarına abone ol">
          Bot olaylarına abone olun:
      
          - `app_mention`
          - `message.channels`, `message.groups`, `message.im`, `message.mpim`
          - `reaction_added`, `reaction_removed`
          - `member_joined_channel`, `member_left_channel`
          - `channel_rename`
          - `pin_added`, `pin_removed`
      
          Ayrıca DM’ler için App Home **Messages Tab** özelliğini etkinleştirin.
        
</Step>
      
        <Step title="Gateway’i başlatın">

```bash
openclaw gateway
```

        
</Step>
      
</Steps>

  
</Tab>

  <Tab title="HTTP Events API mode">
    <Steps>
      <Step title="Configure Slack app for HTTP">

        ```
            - modu HTTP olarak ayarlayın (`channels.slack.mode="http"`)
            - Slack **Signing Secret** değerini kopyalayın
            - Event Subscriptions + Interactivity + Slash command Request URL alanlarını aynı webhook yoluna ayarlayın (varsayılan `/slack/events`)
        
          
</Step>
        
          <Step title="OpenClaw HTTP modunu yapılandırın">
        ```

```json5
{
  channels: {
    slack: {
      enabled: true,
      mode: "http",
      botToken: "xoxb-...",
      signingSecret: "your-signing-secret",
      webhookPath: "/slack/events",
    },
  },
}
```

      Çoklu hesap HTTP modu: `channels.slack.accounts.<id> .mode = "http"` ayarlayın ve her Slack uygulamasının kendi URL’sine işaret edebilmesi için hesap başına benzersiz bir `webhookPath` sağlayın.

  
</Tab>
</Tabs>

## Token modeli

- Socket Mode için `botToken` + `appToken` gereklidir.
- HTTP modu `botToken` + `signingSecret` gerektirir.
- Yapılandırma token’ları, ortam değişkeni yedeğini geçersiz kılar.
- `SLACK_BOT_TOKEN` / `SLACK_APP_TOKEN` ortam değişkeni yedeği yalnızca varsayılan hesap için geçerlidir.
- userTokenReadOnly açıkça ayarlanmış örnek (kullanıcı belirteci yazmalarına izin verir):
- İsteğe bağlı: Giden mesajların etkin ajan kimliğini (özel `username` ve simge) kullanmasını istiyorsanız `chat:write.customize` ekleyin. `icon_emoji` `:emoji_name:` sözdizimini kullanır.

<Tip>
Eylemler/dizin okumaları için, yapılandırıldığında kullanıcı token’ı tercih edilebilir. `userTokenReadOnly: false` olsa bile, bot belirteci mevcutken yazmalar için tercih edilir.
</Tip>

## Erişim kontrolü ve yönlendirme

<Tabs>
  <Tab title="DM policy">Herkese izin vermek için: `channels.slack.dm.policy="open"` ve `channels.slack.dm.allowFrom=["*"]` ayarlayın.

    ```
    - `pairing` (varsayılan)
    - `allowlist`
    - `open` (`channels.slack.allowFrom` içinde `"*"` bulunmasını gerektirir; eski kullanım: `channels.slack.dm.allowFrom`)
    - `disabled`
    
    DM bayrakları:
    
    - `dm.enabled` (varsayılan true)
    - `channels.slack.allowFrom` (tercih edilen)
    - `dm.allowFrom` (eski)
    - `dm.groupEnabled` (grup DM’ler varsayılan false)
    - `dm.groupChannels` (isteğe bağlı MPIM allowlist)
    
    DM’lerde eşleştirme için `openclaw pairing approve slack <code>` kullanılır.
    ```

  
</Tab>

  <Tab title="Channel policy">`channels.slack.groupPolicy` kanal işlemeyi kontrol eder (`open|disabled|allowlist`).

    ```
    Bağlı ama kanallarda yanıt yok: kanal `groupPolicy` tarafından engellenmiş veya `channels.slack.channels` izin listesinde değil.
    ```

  
</Tab>

  <Tab title="Mentions and channel users">
    Kanal mesajları varsayılan olarak mention ile sınırlandırılmıştır.
  

    ```
    Mention geçidi `channels.slack.channels` ile kontrol edilir (`requireMention`’yi `true` olarak ayarlayın); `agents.list[].groupChat.mentionPatterns` (veya `messages.groupChat.mentionPatterns`) de mention olarak sayılır.
    ```

  
</Tab>
</Tabs>

## Komutlar ve slash davranışı

- Yerel komut kaydı `commands.native` kullanır (genel varsayılan `"auto"` → Slack kapalı) ve çalışma alanı bazında `channels.slack.commands.native` ile geçersiz kılınabilir.
- {
  channels: {
  slack: {
  enabled: true,
  appToken: "xapp-...",
  botToken: "xoxb-...",
  userToken: "xoxp-...",
  userTokenReadOnly: false,
  },
  },
  }
- Yerel komutlar etkinleştirildiğinde, Slack’te eşleşen slash komutlarını (`/<command>` adları) kaydedin.
- Yerel komutlar etkin değilse, `channels.slack.slashCommand` aracılığıyla tek bir yapılandırılmış slash komutu çalıştırabilirsiniz.

Varsayılan slash komut ayarları:

- `enabled: false`
- `name: "openclaw"`
- `sessionPrefix: "slack:slash"`
- `ephemeral: true`

Slash oturumları izole anahtarlar kullanır:

- Slash komutları `agent:<agentId>:slack:slash:<userId>` oturumlarını kullanır (ön ek `channels.slack.slashCommand.sessionPrefix` ile yapılandırılabilir).

ve komut yürütmeyi yine hedef konuşma oturumuna (`CommandTargetSessionKey`) yönlendirir.

## İş parçacığı (thread), oturumlar ve yanıt etiketleri

- Grup DM’leri thread’e al, kanalları kökte bırak:
- Varsayılan `session.dmScope=main` ile Slack DM’leri ajan ana oturumuna daraltılır.
- Kanallar `agent:<agentId>:slack:channel:<channelId>` oturumlarına eşlenir.
- Uygun olduğunda, thread yanıtları thread oturum sonekleri (`:thread:<threadTs>`) oluşturabilir.
- `channels.slack.thread.historyScope` varsayılanı `thread` değeridir; `thread.inheritParent` varsayılanı `false` değeridir.
- `channels.slack.thread.initialHistoryLimit`, yeni bir thread oturumu başladığında mevcut thread mesajlarından kaç tanesinin alınacağını kontrol eder (varsayılan `20`; devre dışı bırakmak için `0` ayarlayın).

Yanıt thread kontrolü:

- {
  channels: {
  slack: {
  replyToMode: "off",
  replyToModeByChatType: { group: "first" },
  },
  },
  }
- {
  channels: {
  slack: {
  replyToMode: "first",
  replyToModeByChatType: { direct: "off", group: "off" },
  },
  },
  }
- doğrudan sohbetler için legacy geri dönüş: `channels.slack.dm.replyToMode`

Manuel yanıt etiketleri desteklenir:

- `[[reply_to_current]]`
- `[[reply_to:<id>]]`

Not: `replyToMode="off"` örtük yanıt thread’ini devre dışı bırakır. Açık `[[reply_to_*]]` etiketleri yine de dikkate alınır.

## Medya, parçalama ve teslimat

<AccordionGroup>
  <Accordion title="Inbound attachments">
    Slack dosya ekleri, Slack tarafından barındırılan özel URL’lerden (token kimlik doğrulamalı istek akışı) indirilir ve indirme başarılı olduğunda ve boyut sınırları izin verdiğinde medya deposuna yazılır.

    ```
    Medya yüklemeleri `channels.slack.mediaMaxMb` ile sınırlandırılmıştır (varsayılan 20).
    ```

  
</Accordion>

  <Accordion title="Outbound text and files">
    - metin parçaları `channels.slack.textChunkLimit` (varsayılan 4000) değerini kullanır
    - `channels.slack.chunkMode="newline"` paragraf öncelikli bölmeyi etkinleştirir
    - dosya gönderimleri Slack yükleme API’lerini kullanır ve thread yanıtlarını (`thread_ts`) içerebilir
    - yapılandırıldığında giden medya üst sınırı `channels.slack.mediaMaxMb` değerini izler; aksi takdirde kanal gönderimleri medya işlem hattındaki MIME türü varsayılanlarını kullanır
  
</Accordion>

  <Accordion title="Delivery targets">Teslim hedefleri

    ```
    - DM’ler için `user:<id>`
    - kanallar için `channel:<id>`
    
    Slack DM’leri, kullanıcı hedeflerine gönderim yapılırken Slack conversation API’leri aracılığıyla açılır.
    ```

  
</Accordion>
</AccordionGroup>

## Aksiyonlar ve geçitler

Slack araç eylemleri `channels.slack.actions.*` ile kapatılabilir:

Mevcut Slack araçlarında kullanılabilir aksiyon grupları:

| Eylem grubu | Varsayılan |
| ----------- | ---------- |
| messages    | etkin      |
| reactions   | etkin      |
| pins        | etkin      |
| memberInfo  | etkin      |
| emojiList   | etkin      |

## Olaylar ve operasyonel davranış

- Mesaj düzenleme/silme/thread yayınları sistem olaylarına eşlenir.
- Reaksiyon ekleme/kaldırma olayları sistem olaylarına eşlenir.
- Üye katılma/ayrılma, kanal oluşturma/yeniden adlandırma ve sabitleme ekleme/kaldırma olayları sistem olaylarına eşlenir.
- `channel_id_changed`, `configWrites` etkinleştirildiğinde kanal yapılandırma anahtarlarını taşıyabilir.
- Kanal konu/amaç meta verileri güvenilmeyen bağlam olarak değerlendirilir ve yönlendirme bağlamına enjekte edilebilir.

## Ack reaksiyonları

`ackReaction`, OpenClaw gelen bir mesajı işlerken bir onay emojisi gönderir.

Çözümleme sırası:

- Çoklu hesap için `channels.slack.accounts.<id>.ackReaction`
- `channels.slack.ackReaction`
- `replyToMode`
- ajan kimliği emoji geri dönüşü (`agents.list[].identity.emoji`, aksi halde "👀")

Notlar

- Slack kısa kodlar bekler (örneğin "eyes").
- Bir kanal veya hesap için reaksiyonu devre dışı bırakmak üzere `""` kullanın.

## Manifest ve kapsam kontrol listesi

<AccordionGroup>
  <Accordion title="Slack app manifest example">

```json
{
  "display_information": {
    "name": "OpenClaw",
    "description": "Slack connector for OpenClaw"
  },
  "features": {
    "bot_user": {
      "display_name": "OpenClaw",
      "always_online": false
    },
    "app_home": {
      "messages_tab_enabled": true,
      "messages_tab_read_only_enabled": false
    },
    "slash_commands": [
      {
        "command": "/openclaw",
        "description": "Send a message to OpenClaw",
        "should_escape": false
      }
    ]
  },
  "oauth_config": {
    "scopes": {
      "bot": [
        "chat:write",
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "groups:write",
        "im:history",
        "im:read",
        "im:write",
        "mpim:history",
        "mpim:read",
        "mpim:write",
        "users:read",
        "app_mentions:read",
        "reactions:read",
        "reactions:write",
        "pins:read",
        "pins:write",
        "emoji:read",
        "commands",
        "files:read",
        "files:write"
      ],
      "user": [
        "channels:history",
        "channels:read",
        "groups:history",
        "groups:read",
        "im:history",
        "im:read",
        "mpim:history",
        "mpim:read",
        "users:read",
        "reactions:read",
        "pins:read",
        "emoji:read",
        "search:read"
      ]
    }
  },
  "settings": {
    "socket_mode_enabled": true,
    "event_subscriptions": {
      "bot_events": [
        "app_mention",
        "message.channels",
        "message.groups",
        "message.im",
        "message.mpim",
        "reaction_added",
        "reaction_removed",
        "member_joined_channel",
        "member_left_channel",
        "channel_rename",
        "pin_added",
        "pin_removed"
      ]
    }
  }
}
```

  
</Accordion>

  <Accordion title="Optional user-token scopes (read operations)">
    `channels.slack.userToken` yapılandırırsanız, tipik okuma kapsamları şunlardır:

    ```
    `channels:history`, `groups:history`, `im:history`, `mpim:history`
    [https://docs.slack.dev/reference/methods/conversations.history](https://docs.slack.dev/reference/methods/conversations.history)
    ```

  
</Accordion>
</AccordionGroup>

## Sorun Giderme

<AccordionGroup>
  <Accordion title="No replies in channels">
    Sırasıyla kontrol edin:

    ```
    `users`: isteğe bağlı kanal başına kullanıcı izin listesi.
    ```

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

  
</Accordion>

  <Accordion title="DM messages ignored">
    Kontrol edin:

    ```
    `allowlist`, kanalların `channels.slack.channels` içinde listelenmesini gerektirir.
    ```

```bash
openclaw pairing list slack
```

  
</Accordion>

  <Accordion title="Socket mode not connecting">
    Slack uygulama ayarlarında bot + uygulama token’larını ve Socket Mode etkinleştirmesini doğrulayın.
  
</Accordion>

  <Accordion title="HTTP mode not receiving events">
    Doğrulayın:

    ```
    - signing secret
    - webhook yolu
    - Slack Request URL’leri (Events + Interactivity + Slash Commands)
    - her HTTP hesabı için benzersiz `webhookPath`
    ```

  
</Accordion>

  <Accordion title="Native/slash commands not firing">
    Şunu amaçlayıp amaçlamadığınızı doğrulayın:

    ```
    Slash Commands → `channels.slack.slashCommand` kullanıyorsanız `/openclaw` oluşturun. Yerel komutları etkinleştirirseniz, her yerleşik komut için bir slash command ekleyin (`/help` ile aynı adlar). Yerel komutlar, `channels.slack.commands.native: true` ayarlanmadıkça Slack için varsayılan olarak kapalıdır (genel `commands.native` varsayılanı `"auto"` olup Slack’i kapalı bırakır).
    ```

  
</Accordion>
</AccordionGroup>

## Yapılandırma referans işaretçileri

Öncelik sırası:

- [Yapılandırma referansı - Slack](/gateway/configuration-reference#slack)

  Yüksek öncelikli Slack alanları:

  - mode/auth: `mode`, `botToken`, `appToken`, `signingSecret`, `webhookPath`, `accounts.*`
  - DM erişimi: `dm.enabled`, `dmPolicy`, `allowFrom` (eski: `dm.policy`, `dm.allowFrom`), `dm.groupEnabled`, `dm.groupChannels`
  - `allow`: `groupPolicy="allowlist"` iken kanala izin ver/reddet.
  - threading/history: `replyToMode`, `replyToModeByChatType`, `thread.*`, `historyLimit`, `dmHistoryLimit`, `dms.*.historyLimit`
  - teslimat: `textChunkLimit`, `chunkMode`, `mediaMaxMb`
  - operasyonlar/özellikler: `configWrites`, `commands.native`, `slashCommand.*`, `actions.*`, `userToken`, `userTokenReadOnly`

## İlgili

- [Eşleştirme](/channels/pairing)
- `channel`: standart kanallar (herkese açık/özel)
- Triyaj akışı için: [/channels/troubleshooting](/channels/troubleshooting).
- [Yapılandırma](/gateway/configuration)
- Tam komut listesi + yapılandırma: [Slash commands](/tools/slash-commands)

