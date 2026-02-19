---
summary: "WhatsApp kanal desteği, erişim kontrolleri, teslim davranışı ve operasyonlar"
read_when:
  - WhatsApp/web kanalı davranışı veya gelen kutusu yönlendirmesi üzerinde çalışırken
title: "WhatsApp"
---

# WhatsApp (web kanalı)

Durum: Yalnızca Baileys üzerinden WhatsApp Web. Oturum(lar) Gateway’e aittir.

<CardGroup cols={3}>
  <Card title="Pairing" icon="link" href="/channels/pairing">Varsayılan DM politikası **eşleştirme**dir; bu nedenle bilinmeyen gönderenler yalnızca bir eşleştirme kodu alır ve mesajları **işlenmez**.
</Card>
  <Card title="Channel troubleshooting" icon="wrench" href="/channels/troubleshooting">
    Kanallar arası tanılama ve onarım kılavuzları.
  
</Card>
  <Card title="Gateway configuration" icon="settings" href="/gateway/configuration">
    Tam kanal yapılandırma desenleri ve örnekleri.
  
</Card>
</CardGroup>

## Hızlı kurulum (başlangıç)

<Steps>
  <Step title="Configure WhatsApp access policy">

```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"],
    },
  },
}
```

  
</Step>

  <Step title="Link WhatsApp (QR)">

```bash
openclaw channels login --channel whatsapp
```

    ```
    Belirli bir hesap için:
    ```

```bash
openclaw channels login --channel whatsapp --account work
```

  
</Step>

  <Step title="Start the gateway">

```bash
openclaw gateway
```

  
</Step>

  <Step title="Approve first pairing request (if using pairing mode)">

```bash
Onaylamak için: `openclaw pairing approve whatsapp <code>` (listelemek için `openclaw pairing list whatsapp`).
```

    ```
    Kodlar 1 saat sonra dolar; bekleyen istekler kanal başına 3 ile sınırlıdır.
    ```

  
</Step>
</Steps>

<Note>
Kişisel WhatsApp’ınızı ayrı tutmak için harikadır — WhatsApp Business’ı kurun ve OpenClaw numarasını orada kaydedin. (Kanal meta verileri ve onboarding akışı bu kurulum için optimize edilmiştir, ancak kişisel numara kurulumları da desteklenir.)
</Note>

## Dağıtım desenleri

<AccordionGroup>
  <Accordion title="Dedicated number (recommended)">
    Bu, operasyonel olarak en temiz moddur:
  

    ```
    {
      channels: {
        whatsapp: {
          dmPolicy: "allowlist",
          allowFrom: ["+15551234567"],
        },
      },
    }
    ```

  
</Accordion>

  <Accordion title="Personal-number fallback">
    Onboarding, kişisel numara modunu destekler ve self-chat dostu bir temel yapı yazar:
  

    ```
    {
      "whatsapp": {
        "selfChatMode": true,
        "dmPolicy": "allowlist",
        "allowFrom": ["+15551234567"]
      }
    }
    ```

  
</Accordion>

  <Accordion title="WhatsApp Web-only channel scope">Mesajlaşma platformu kanalı, mevcut OpenClaw kanal mimarisinde WhatsApp Web tabanlıdır (`Baileys`).

    ```
    Yerleşik sohbet-kanalı kayıt defterinde ayrı bir Twilio WhatsApp mesajlaşma kanalı yoktur.
    ```

  
</Accordion>
</AccordionGroup>

## Çalışma zamanı modeli

- Gateway, WhatsApp soketini ve yeniden bağlanma döngüsünü yönetir.
- Giden gönderimler, hedef hesap için aktif bir WhatsApp dinleyicisi gerektirir.
- Durum ve yayın sohbetleri yok sayılır (`@status`, `@broadcast`).
- Doğrudan sohbetler DM oturum kurallarını kullanır (`session.dmScope`; varsayılan `main`, DM’leri aracının ana oturumunda birleştirir).
- Gruplar `agent:<agentId>:whatsapp:group:<jid>` oturumlarına eşlenir.

## Erişim kontrolü ve etkinleştirme

<Tabs>
  <Tab title="DM policy">**DM politikası**: `channels.whatsapp.dmPolicy` doğrudan sohbet erişimini kontrol eder (varsayılan: `pairing`).

    ```
    Grup politikası: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (varsayılan `allowlist`).
    ```

  
</Tab>

  <Tab title="Group policy + allowlists">
    Grup erişiminin iki katmanı vardır:
  

    ```
    `channels.whatsapp.groupAllowFrom` (grup gönderen izin listesi).
    ```

  
</Tab>

  <Tab title="Mentions + /activation">
    Varsayılan olarak grup yanıtları için bahsetme gereklidir.

Bahsetme tespiti şunları içerir:

- bot kimliğinin açık WhatsApp mention’ları
- yapılandırılmış mention regex desenleri (`agents.list[].groupChat.mentionPatterns`, geri dönüş olarak `messages.groupChat.mentionPatterns`)
- bot’a yanıt verme durumunun örtük tespiti (yanıt gönderen bot kimliğiyle eşleşir)

Oturum düzeyinde etkinleştirme komutu:

- `/activation mention`
- `/activation always`

`activation`, oturum durumunu günceller (genel yapılandırmayı değil). Yalnızca owner tarafından kullanılabilir.

    ```
      
</Tab>
    ```

  
</Tab>
</Tabs>

## Bağlı self numarası `allowFrom` içinde de mevcutsa, WhatsApp self-chat korumaları devreye girer:

self-chat mesajları için okundu bilgilerini atla

- aksi halde kendinizi etiketleyecek olan mention-JID otomatik tetikleme davranışını yok say
- Mesaj normalizasyonu ve bağlam
- Kendi kendine sohbet yanıtları, ayarlandığında varsayılan olarak `[{identity.name}]`’ya gider (aksi halde `[openclaw]`)  
  eğer `messages.responsePrefix` ayarlı değilse.

## ```
Gelen WhatsApp mesajları paylaşılan inbound zarfı içine alınır.
```

<AccordionGroup>
  <Accordion title="Inbound envelope + reply context">Alıntılanmış bir yanıt varsa, bağlam şu biçimde eklenir:

```text
[Replying to <sender> id:<stanzaId>]
<quoted body or media placeholder>
[/Replying]
```

Yanıt meta veri alanları da mevcutsa doldurulur (`ReplyToId`, `ReplyToBody`, `ReplyToSender`, sender JID/E.164).

    ```
    - `<media:image>`
    - `<media:video>`
    - `<media:audio>`
    - `<media:document>`
    - `<media:sticker>`
    
    Konum ve kişi içerikleri yönlendirmeden önce metinsel bağlama dönüştürülür.
    ```

  
</Accordion>

  <Accordion title="Media placeholders and location/contact extraction">Yalnızca medyadan oluşan gelen mesajlar yer tutucular kullanır:

    ```
    
        Gruplar için, işlenmemiş mesajlar arabelleğe alınabilir ve bot nihayet tetiklendiğinde bağlam olarak eklenebilir.
      
    ```

  
</Accordion>

  <Accordion title="Pending group history injection">  
</Accordion>

    ```
    Son _işlenmemiş_ mesajlar (varsayılan 50) şu başlık altına eklenir:
    `[Chat messages since your last reply - for context]` (oturumda zaten olan mesajlar yeniden enjekte edilmez)
    ```

  
</Accordion>

  <Accordion title="Read receipts">Varsayılan olarak gateway, kabul edildikten sonra gelen WhatsApp mesajlarını okundu (mavi tikler) olarak işaretler.

    ```
    {
      channels: {
        whatsapp: {
          accounts: {
            personal: { sendReadReceipts: false },
          },
        },
      },
    }
    ```

  
</Accordion>
</AccordionGroup>

```
- görsel, video, ses (PTT voice-note) ve belge yüklerini destekler
- voice-note uyumluluğu için `audio/ogg`, `audio/ogg; codecs=opus` olarak yeniden yazılır
- video gönderimlerinde `gifPlayback: true` ile animasyonlu GIF oynatma desteklenir
- çoklu medya yanıt yükleri gönderilirken başlıklar ilk medya öğesine uygulanır
- medya kaynağı HTTP(S), `file://` veya yerel yollar olabilir
```
---

<AccordionGroup>
  <Accordion title="Text chunking">İsteğe bağlı satır sonu bölme: uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `channels.whatsapp.chunkMode="newline"`’u ayarlayın.
</Accordion>

  <Accordion title="Outbound media behavior">
    - gelen medya kaydetme sınırı: `channels.whatsapp.mediaMaxMb` (varsayılan `50`)
    - otomatik yanıtlar için giden medya sınırı: `agents.defaults.mediaMaxMb` (varsayılan `5MB`)
    - görseller sınırlara uyması için otomatik olarak optimize edilir (yeniden boyutlandırma/kalite ayarı)
    - medya gönderim hatasında, yanıtın sessizce düşmesi yerine ilk öğe için metin uyarısı gönderilir
   
</Accordion>

  <Accordion title="Media size limits and fallback behavior">
    - inbound media save cap: `channels.whatsapp.mediaMaxMb` (default `50`)
    - outbound media cap for auto-replies: `agents.defaults.mediaMaxMb` (default `5MB`)
    - images are auto-optimized (resize/quality sweep) to fit limits
    - on media send failure, first-item fallback sends text warning instead of dropping the response silently
  
</Accordion>
</AccordionGroup>

## Onay reaksiyonları

`channels.whatsapp.ackReaction` (mesaj alımında otomatik reaksiyon: `{emoji, direct, group}`).

```json5
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

Notlar:

- gelen mesaj kabul edildikten hemen sonra gönderilir (yanıt öncesi)
- hatalar günlüğe kaydedilir ancak normal yanıt teslimini engellemez
- grup modu `mentions`, mention ile tetiklenen turlarda tepki verir; grup etkinleştirme `always` bu kontrol için baypas görevi görür
- WhatsApp `messages.ackReaction`’yı yok sayar; bunun yerine `channels.whatsapp.ackReaction` kullanın.

## Çoklu hesap ve kimlik bilgileri

<AccordionGroup>
  <Accordion title="Account selection and defaults">`channels.whatsapp.accounts.<accountId> .*` (hesap bazlı ayarlar + isteğe bağlı `authDir`).
</Accordion>

  <Accordion title="Credential paths and legacy compatibility">
    - mevcut kimlik doğrulama yolu: `~/.openclaw/credentials/whatsapp/<accountId>/creds.json`
    - yedek dosya: `creds.json.bak`
    - `~/.openclaw/credentials/` içindeki eski varsayılan kimlik doğrulama hâlâ varsayılan hesap akışları için tanınır/taşınır
  
</Accordion>

  <Accordion title="Logout behavior">Çoklu hesap girişi: `openclaw channels login --account <id>` (`<id>` = `accountId`).<id>]` o hesap için WhatsApp kimlik doğrulama durumunu temizler.

    ```
    Eski kimlik doğrulama dizinlerinde `oauth.json` korunur, Baileys kimlik doğrulama dosyaları kaldırılır.
    ```

  
</Accordion>
</AccordionGroup>

## Araçlar, eylemler ve yapılandırma yazımları

- Araç: `whatsapp` ve `react` eylemi (`chatJid`, `messageId`, `emoji`, isteğe bağlı `remove`).
- Eylem kapıları:
  - `channels.whatsapp.actions.reactions` (WhatsApp araç reaksiyonlarını geçitle).
  - \`channels.whatsapp.accounts.<accountId>
- {
  channels: { whatsapp: { configWrites: false } },
  }

## Sorun Giderme

<AccordionGroup>
  <Accordion title="Not linked (QR required)">Belirti: `channels status` `linked: false` gösterir veya “Not linked” uyarır.

    ```
    Çıkış: `openclaw channels logout` (veya `--account <id>`) WhatsApp yetkilendirme durumunu siler (paylaşılan `oauth.json` korunur).
    ```

  
</Accordion>

  <Accordion title="Linked but disconnected / reconnect loop">
    Belirti: tekrar eden bağlantı kesilmeleri veya yeniden bağlanma denemeleri olan bağlı hesap.


    ```
    Çözüm: `openclaw doctor` (veya gateway’i yeniden başlatın). Devam ederse, `channels login` ile yeniden bağlayın ve `openclaw logs --follow`’i inceleyin.
    ```

  
</Accordion>

  <Accordion title="No active listener when sending">Hedef hesap için etkin bir gateway dinleyicisi yoksa giden gönderimler hızlıca başarısız olur.

    ```
    Gateway’in çalıştığından ve hesabın bağlı olduğundan emin olun.
    ```

  
</Accordion>

  <Accordion title="Group messages unexpectedly ignored">
    Bu sırayla kontrol edin:


    ```
    `channels.whatsapp.groups` (grup izin listesi + mention geçitleme varsayılanları; tümüne izin vermek için `"*"` kullanın)
    ```

  
</Accordion>

  <Accordion title="Bun runtime warning">
    WhatsApp gateway çalışma zamanı Node kullanmalıdır. Bun **önerilmez**. WhatsApp (Baileys) ve Telegram Bun üzerinde güvenilir değildir. Gateway’i **Node** ile çalıştırın.
  
</Accordion>
</AccordionGroup>

## Yapılandırma referans işaretçileri

Birincil referans:

- [Yapılandırma referansı - WhatsApp](/gateway/configuration-reference#whatsapp)

Yüksek öneme sahip WhatsApp alanları:

- erişim: `dmPolicy`, `allowFrom`, `groupPolicy`, `groupAllowFrom`, `groups`
- teslimat: `textChunkLimit`, `chunkMode`, `mediaMaxMb`, `sendReadReceipts`, `ackReaction`
- çoklu hesap: `accounts.<id> .enabled`, `accounts.<id> .authDir`, hesap düzeyinde geçersiz kılmalar
- operasyonlar: `configWrites`, `debounceMs`, `web.enabled`, `web.heartbeatSeconds`, `web.reconnect.*`
- oturum davranışı: `session.dmScope`, `historyLimit`, `dmHistoryLimit`, `dms.<id>`messages.groupChat.historyLimit\`

## İlgili

- [Eşleştirme](/channels/pairing)
- [Kanal yönlendirme](/channels/channel-routing)
- Sorun giderme kılavuzu: [Gateway sorun giderme](/gateway/troubleshooting).

