---
title: "WhatsApp"
---

# WhatsApp (web kanalı)

Durum: Yalnızca Baileys üzerinden WhatsApp Web. Oturum(lar) Gateway’e aittir.

## Hızlı kurulum (başlangıç)

1. Mümkünse **ayrı bir telefon numarası** kullanın (önerilir).
2. WhatsApp’ı `~/.openclaw/openclaw.json` içinde yapılandırın.
3. QR kodunu taramak için `openclaw channels login` çalıştırın (Bağlı Cihazlar).
4. Gateway’i başlatın.

Asgari yapılandırma:

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

## Hedefler

- Tek bir Gateway sürecinde birden fazla WhatsApp hesabı (çoklu hesap).
- Deterministik yönlendirme: yanıtlar WhatsApp’a geri döner, model yönlendirmesi yoktur.
- Modelin alıntılanan yanıtları anlayabilmesi için yeterli bağlam görmesi.

## Yapılandırma yazımları

Varsayılan olarak, WhatsApp `/config set|unset` tarafından tetiklenen yapılandırma güncellemelerini yazmaya izinlidir (`commands.config: true` gerektirir).

Şununla devre dışı bırakın:

```json5
{
  channels: { whatsapp: { configWrites: false } },
}
```

## Mimari (kimin neye sahip olduğu)

- **Gateway**, Baileys soketine ve gelen kutusu döngüsüne sahiptir.
- **CLI / macOS uygulaması** gateway ile konuşur; Baileys doğrudan kullanılmaz.
- **Aktif dinleyici**, giden gönderimler için gereklidir; aksi halde gönderim hızlıca başarısız olur.

## Telefon numarası edinme (iki mod)

WhatsApp doğrulama için gerçek bir mobil numara ister. VoIP ve sanal numaralar genellikle engellenir. OpenClaw’ı WhatsApp’ta çalıştırmanın iki desteklenen yolu vardır:

### Ayrı numara (önerilir)

OpenClaw için **ayrı bir telefon numarası** kullanın. En iyi UX, temiz yönlendirme, kendi kendine sohbet tuhaflıkları yoktur. İdeal kurulum: **yedek/eski bir Android telefon + eSIM**. Wi‑Fi ve güçte bırakın ve QR ile bağlayın.

**WhatsApp Business:** Aynı cihazda farklı bir numarayla WhatsApp Business kullanabilirsiniz. Kişisel WhatsApp’ınızı ayrı tutmak için harikadır — WhatsApp Business’ı kurun ve OpenClaw numarasını orada kaydedin.

**Örnek yapılandırma (ayrı numara, tek kullanıcı izin listesi):**

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

**Eşleştirme modu (isteğe bağlı):**  
İzin listesi yerine eşleştirme istiyorsanız, `channels.whatsapp.dmPolicy`’ü `pairing` olarak ayarlayın. Bilinmeyen gönderenler bir eşleştirme kodu alır; onaylamak için:
`openclaw pairing approve whatsapp <code>`

### Kişisel numara (yedek)

Hızlı bir yedek: OpenClaw’ı **kendi numaranızda** çalıştırın. Kişilerinizi spam’lememek için testte kendinize mesaj atın (WhatsApp “Kendine mesaj”). Kurulum ve denemeler sırasında doğrulama kodlarını ana telefonunuzdan okumanız beklenir. **Kendi kendine sohbet modu etkinleştirilmelidir.**  
Sihirbaz kişisel WhatsApp numaranızı sorduğunda, asistan numarasını değil, mesaj göndereceğiniz telefonu (sahip/gönderen) girin.

**Örnek yapılandırma (kişisel numara, kendi kendine sohbet):**

```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

Kendi kendine sohbet yanıtları, ayarlandığında varsayılan olarak `[{identity.name}]`’ya gider (aksi halde `[openclaw]`)  
eğer `messages.responsePrefix` ayarlı değilse. Özelleştirmek veya  
ön eki devre dışı bırakmak için açıkça ayarlayın (kaldırmak için `""` kullanın).

### Numara temin ipuçları

- Ülkenizin mobil operatöründen **yerel eSIM** (en güvenilir)
  - Avusturya: [hot.at](https://www.hot.at)
  - Birleşik Krallık: [giffgaff](https://www.giffgaff.com) — ücretsiz SIM, sözleşme yok
- **Ön ödemeli SIM** — ucuzdur, doğrulama için yalnızca bir SMS alması yeterlidir

**Kaçının:** TextNow, Google Voice, çoğu “ücretsiz SMS” hizmeti — WhatsApp bunları agresif biçimde engeller.

**İpucu:** Numaranın yalnızca bir doğrulama SMS’i alması gerekir. Sonrasında WhatsApp Web oturumları `creds.json` üzerinden kalıcı olur.

## Neden Twilio Değil?

- OpenClaw’ın erken sürümleri Twilio’nun WhatsApp Business entegrasyonunu destekliyordu.
- WhatsApp Business numaraları kişisel asistan için uygun değildir.
- Meta 24 saatlik yanıt penceresi uygular; son 24 saatte yanıt vermediyseniz, business numarası yeni mesaj başlatamaz.
- Yüksek hacimli veya “sohbetçi” kullanım agresif engellemeleri tetikler; çünkü business hesaplar kişisel asistan gibi onlarca mesaj göndermek için tasarlanmamıştır.
- Sonuç: güvenilmez teslimat ve sık engellemeler; bu nedenle destek kaldırıldı.

## Giriş + kimlik bilgileri

- Giriş komutu: `openclaw channels login` (Bağlı Cihazlar üzerinden QR).
- Çoklu hesap girişi: `openclaw channels login --account <id>` (`<id>` = `accountId`).
- Varsayılan hesap (`--account` atlanırsa): varsa `default`, yoksa yapılandırılmış ilk hesap kimliği (sıralı).
- Kimlik bilgileri `~/.openclaw/credentials/whatsapp/<accountId>/creds.json` içinde saklanır.
- Yedek kopya `creds.json.bak`’dedir (bozulmada geri yüklenir).
- Eski uyumluluk: daha eski kurulumlar Baileys dosyalarını doğrudan `~/.openclaw/credentials/` içine kaydederdi.
- Çıkış: `openclaw channels logout` (veya `--account <id>`) WhatsApp yetkilendirme durumunu siler (paylaşılan `oauth.json` korunur).
- Çıkış yapılmış soket ⇒ yeniden bağlama talimatı veren hata.

## Gelen akış (DM + grup)

- WhatsApp olayları `messages.upsert`’ten (Baileys) gelir.
- Testler/yeniden başlatmalarda olay işleyicilerinin birikmesini önlemek için kapatmada gelen kutusu dinleyicileri ayrılır.
- Durum/yayın sohbetleri yok sayılır.
- Direkt sohbetler E.164 kullanır; gruplar grup JID kullanır.
- **DM politikası**: `channels.whatsapp.dmPolicy` doğrudan sohbet erişimini kontrol eder (varsayılan: `pairing`).
  - Eşleştirme: bilinmeyen gönderenler bir eşleştirme kodu alır (onay `openclaw pairing approve whatsapp <code>` ile; kodlar 1 saat sonra dolar).
  - Açık: `channels.whatsapp.allowFrom`’nin `"*"` içermesini gerektirir.
  - Bağlı WhatsApp numaranız örtük olarak güvenilirdir; bu nedenle kendi mesajlarınız `channels.whatsapp.dmPolicy` ve `channels.whatsapp.allowFrom` kontrollerini atlar.

### Kişisel numara modu (yedek)

OpenClaw’ı **kişisel WhatsApp numaranızda** çalıştırıyorsanız, `channels.whatsapp.selfChatMode`’i etkinleştirin (yukarıdaki örneğe bakın).

Davranış:

- Giden DM’ler asla eşleştirme yanıtlarını tetiklemez (kişileri spam’lemeyi önler).
- Gelen bilinmeyen gönderenler yine `channels.whatsapp.dmPolicy`’yi izler.
- Kendi kendine sohbet modu (allowFrom numaranızı içerir) otomatik okundu bildirimlerini önler ve mention JID’lerini yok sayar.
- Kendi kendine olmayan DM’ler için okundu bildirimleri gönderilir.

## Read receipts

Varsayılan olarak gateway, kabul edildikten sonra gelen WhatsApp mesajlarını okundu (mavi tikler) olarak işaretler.

Genel olarak devre dışı bırakmak için:

```json5
{
  channels: { whatsapp: { sendReadReceipts: false } },
}
```

Hesap bazında devre dışı bırakmak için:

```json5
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

Notlar:

- Kendi kendine sohbet modu okundu bildirimlerini her zaman atlar.

## WhatsApp SSS: mesaj gönderme + eşleştirme

**WhatsApp’ı bağladığımda OpenClaw rastgele kişilere mesaj atar mı?**  
Hayır. Varsayılan DM politikası **eşleştirme**dir; bu nedenle bilinmeyen gönderenler yalnızca bir eşleştirme kodu alır ve mesajları **işlenmez**. OpenClaw yalnızca aldığı sohbetlere veya sizin açıkça tetiklediğiniz gönderimlere (ajan/CLI) yanıt verir.

**WhatsApp’ta eşleştirme nasıl çalışır?**  
Eşleştirme, bilinmeyen gönderenler için bir DM kapısıdır:

- Yeni bir gönderenden gelen ilk DM kısa bir kod döndürür (mesaj işlenmez).
- Onaylamak için: `openclaw pairing approve whatsapp <code>` (listelemek için `openclaw pairing list whatsapp`).
- Kodlar 1 saat sonra dolar; bekleyen istekler kanal başına 3 ile sınırlıdır.

**Bir WhatsApp numarası üzerinde birden fazla kişi farklı OpenClaw örneklerini kullanabilir mi?**  
Evet, her göndericiyi `bindings` üzerinden farklı bir agente yönlendirerek (peer `kind: "direct"`, gönderici E.164 örn. `+15551234567`). Yanıtlar yine **aynı WhatsApp hesabından** gelir ve direkt sohbetler her agentin ana oturumuna çöker; bu nedenle **kişi başına bir agent** kullanın. DM erişim denetimi (`dmPolicy`/`allowFrom`) WhatsApp hesabı başına globaldir. [Çoklu Ajan Yönlendirme](/concepts/multi-agent) bölümüne bakın.

**Sihirbaz neden telefon numaramı istiyor?**  
Sihirbaz bunu **izin listesi/sahip** ayarlamak için kullanır; böylece kendi DM’lerinize izin verilir. Otomatik gönderim için kullanılmaz. Kişisel WhatsApp numaranızda çalıştırıyorsanız, aynı numarayı kullanın ve `channels.whatsapp.selfChatMode`’yi etkinleştirin.

## Mesaj normalizasyonu (modelin gördüğü)

- `Body` mevcut mesaj gövdesidir (zarf ile).

- Alıntılanan yanıt bağlamı **her zaman eklenir**:

  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```

- Yanıt meta verileri de ayarlanır:
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = alıntılanan gövde veya medya yer tutucusu
  - `ReplyToSender` = biliniyorsa E.164

- Yalnızca medyadan oluşan gelen mesajlar yer tutucular kullanır:
  - `<media:image|video|audio|document|sticker>`

## Gruplar

- Gruplar `agent:<agentId>:whatsapp:group:<jid>` oturumlarına eşlenir.
- Grup politikası: `channels.whatsapp.groupPolicy = open|disabled|allowlist` (varsayılan `allowlist`).
- Etkinleştirme modları:
  - `mention` (varsayılan): @mention veya regex eşleşmesi gerektirir.
  - `always`: her zaman tetikler.
- `/activation mention|always` yalnızca sahibe özeldir ve tek başına bir mesaj olarak gönderilmelidir.
- Sahip = `channels.whatsapp.allowFrom` (veya ayarlı değilse kendi E.164).
- **Geçmiş enjeksiyonu** (yalnızca bekleyenler):
  - Son _işlenmemiş_ mesajlar (varsayılan 50) şu başlık altına eklenir:
    `[Chat messages since your last reply - for context]` (oturumda zaten olan mesajlar yeniden enjekte edilmez)
  - Mevcut mesaj şu başlık altında:
    `[Current message - respond to this]`
  - Gönderen soneki eklenir: `[from: Name (+E164)]`
- Grup meta verileri 5 dk önbelleğe alınır (konu + katılımcılar).

## Yanıt teslimi (iş parçacığı)

- WhatsApp Web standart mesajlar gönderir (mevcut gateway’de alıntılı yanıt iş parçacığı yoktur).
- Yanıt etiketleri bu kanalda yok sayılır.

## Onay reaksiyonları (alımda otomatik reaksiyon)

WhatsApp, bot yanıt üretmeden önce, gelen mesajlara alındığı anda otomatik emoji reaksiyonları gönderebilir. Bu, kullanıcılara mesajlarının alındığına dair anında geri bildirim sağlar.

**Yapılandırma:**

```json
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

**Seçenekler:**

- `emoji` (string): Onay için kullanılacak emoji (örn. "👀", "✅", "📨"). Boş veya atlanmış = özellik devre dışı.
- `direct` (boolean, varsayılan: `true`): Direkt/DM sohbetlerinde reaksiyon gönder.
- `group` (string, varsayılan: `"mentions"`): Grup sohbeti davranışı:
  - `"always"`: Tüm grup mesajlarına reaksiyon ver (@mention olmasa bile)
  - `"mentions"`: Yalnızca bot @mention edildiğinde reaksiyon ver
  - `"never"`: Gruplarda asla reaksiyon verme

**Hesap bazlı geçersiz kılma:**

```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**Davranış notları:**

- Reaksiyonlar, yazıyor göstergeleri veya bot yanıtlarından önce, mesaj alındığı **hemen** gönderilir.
- `requireMention: false` (etkinleştirme: her zaman) olan gruplarda, `group: "mentions"` tüm mesajlara reaksiyon verir (yalnızca @mention’lara değil).
- Ateşle-ve-unut: reaksiyon hataları kaydedilir ancak botun yanıt vermesini engellemez.
- Grup reaksiyonları için katılımcı JID otomatik eklenir.
- WhatsApp `messages.ackReaction`’yı yok sayar; bunun yerine `channels.whatsapp.ackReaction` kullanın.

## Ajan aracı (reaksiyonlar)

- Araç: `whatsapp` ve `react` eylemi (`chatJid`, `messageId`, `emoji`, isteğe bağlı `remove`).
- İsteğe bağlı: `participant` (grup göndereni), `fromMe` (kendi mesajınıza reaksiyon), `accountId` (çoklu hesap).
- Reaksiyon kaldırma semantiği: [/tools/reactions](/tools/reactions) bölümüne bakın.
- Araç geçitleme: `channels.whatsapp.actions.reactions` (varsayılan: etkin).

## Sınırlar

- Giden metin `channels.whatsapp.textChunkLimit`’e bölünür (varsayılan 4000).
- İsteğe bağlı satır sonu bölme: uzunluk bölmeden önce boş satırlarda (paragraf sınırları) bölmek için `channels.whatsapp.chunkMode="newline"`’u ayarlayın.
- Gelen medya kayıtları `channels.whatsapp.mediaMaxMb` ile sınırlandırılır (varsayılan 50 MB).
- Giden medya öğeleri `agents.defaults.mediaMaxMb` ile sınırlandırılır (varsayılan 5 MB).

## Giden gönderim (metin + medya)

- Aktif web dinleyicisini kullanır; gateway çalışmıyorsa hata verir.
- Metin bölme: mesaj başına en fazla 4k ( `channels.whatsapp.textChunkLimit` ile yapılandırılabilir, isteğe bağlı `channels.whatsapp.chunkMode`).
- Medya:
  - Görsel/video/ses/belge desteklenir.
  - Ses PTT olarak gönderilir; `audio/ogg` ⇒ `audio/ogg; codecs=opus`.
  - Başlık yalnızca ilk medya öğesinde kullanılır.
  - Medya alma HTTP(S) ve yerel yolları destekler.
  - Animasyonlu GIF’ler: WhatsApp, satır içi döngü için `gifPlayback: true` içeren MP4 bekler.
    - CLI: `openclaw message send --media <mp4> --gif-playback`
    - Gateway: `send` parametreleri `gifPlayback: true` içerir

## Sesli notlar (PTT ses)

WhatsApp sesi **sesli not** (PTT balonu) olarak gönderir.

- En iyi sonuçlar: OGG/Opus. OpenClaw `audio/ogg`’ü `audio/ogg; codecs=opus`’e dönüştürür.
- `[[audio_as_voice]]` WhatsApp için yok sayılır (ses zaten sesli not olarak gönderilir).

## Medya sınırları + optimizasyon

- Varsayılan giden sınır: 5 MB (medya öğesi başına).
- Geçersiz kılma: `agents.defaults.mediaMaxMb`.
- Görseller, sınır altında JPEG’e otomatik optimize edilir (yeniden boyutlandırma + kalite taraması).
- Aşırı büyük medya ⇒ hata; medya yanıtı metin uyarısına düşer.

## Heartbeat’ler

- **Gateway heartbeat’i** bağlantı sağlığını günlüğe yazar (`web.heartbeatSeconds`, varsayılan 60 sn).
- **Ajan heartbeat’i** ajan başına (`agents.list[].heartbeat`) veya global olarak
  `agents.defaults.heartbeat` üzerinden yapılandırılabilir (ajan başına giriş yoksa yedek).
  - Yapılandırılmış heartbeat istemini kullanır (varsayılan: `Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`) + `HEARTBEAT_OK` atlama davranışı.
  - Teslimat varsayılan olarak son kullanılan kanala gider (veya yapılandırılmış hedefe).

## Yeniden bağlanma davranışı

- Geri çekilme politikası: `web.reconnect`:
  - `initialMs`, `maxMs`, `factor`, `jitter`, `maxAttempts`.
- maxAttempts’e ulaşılırsa, web izleme durur (bozulmuş).
- Çıkış yapılmış ⇒ dur ve yeniden bağlama gerektir.

## Yapılandırma hızlı haritası

- `channels.whatsapp.dmPolicy` (DM politikası: eşleştirme/izin listesi/açık/devre dışı).
- `channels.whatsapp.selfChatMode` (aynı telefon kurulumu; bot kişisel WhatsApp numaranızı kullanır).
- `channels.whatsapp.allowFrom` (DM izin listesi). WhatsApp E.164 telefon numaralarını kullanır (kullanıcı adı yok).
- `channels.whatsapp.mediaMaxMb` (gelen medya kayıt sınırı).
- `channels.whatsapp.ackReaction` (mesaj alımında otomatik reaksiyon: `{emoji, direct, group}`).
- `channels.whatsapp.accounts.<accountId>.*` (hesap bazlı ayarlar + isteğe bağlı `authDir`).
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb` (hesap bazlı gelen medya sınırı).
- `channels.whatsapp.accounts.<accountId>.ackReaction` (hesap bazlı onay reaksiyonu geçersiz kılma).
- `channels.whatsapp.groupAllowFrom` (grup gönderen izin listesi).
- `channels.whatsapp.groupPolicy` (grup politikası).
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit` (grup geçmişi bağlamı; `0` devre dışı bırakır).
- `channels.whatsapp.dmHistoryLimit` (kullanıcı dönüşleri cinsinden DM geçmiş sınırı). Kullanıcı başına geçersiz kılmalar: `channels.whatsapp.dms["<phone>"].historyLimit`.
- `channels.whatsapp.groups` (grup izin listesi + mention geçitleme varsayılanları; tümüne izin vermek için `"*"` kullanın)
- `channels.whatsapp.actions.reactions` (WhatsApp araç reaksiyonlarını geçitle).
- `agents.list[].groupChat.mentionPatterns` (veya `messages.groupChat.mentionPatterns`)
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix` (gelen ön ek; hesap bazında: `channels.whatsapp.accounts.<accountId>.messagePrefix`; kullanımdan kaldırıldı: `messages.messagePrefix`)
- `messages.responsePrefix` (giden ön ek)
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model` (isteğe bağlı geçersiz kılma)
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*` (ajan bazlı geçersiz kılmalar)
- `session.*` (kapsam, idle, store, mainKey)
- `web.enabled` (false olduğunda kanal başlangıcını devre dışı bırak)
- `web.heartbeatSeconds`
- `web.reconnect.*`

## Logs + troubleshooting

- Alt sistemler: `whatsapp/inbound`, `whatsapp/outbound`, `web-heartbeat`, `web-reconnect`.
- Günlük dosyası: `/tmp/openclaw/openclaw-YYYY-MM-DD.log` (yapılandırılabilir).
- Sorun giderme kılavuzu: [Gateway sorun giderme](/gateway/troubleshooting).

## Sorun giderme (hızlı)

**Bağlı değil / QR girişi gerekli**

- Belirti: `channels status` `linked: false` gösterir veya “Not linked” uyarır.
- Çözüm: gateway ana makinesinde `openclaw channels login` çalıştırın ve QR’ı tarayın (WhatsApp → Ayarlar → Bağlı Cihazlar).

**Bağlı ama kopuk / yeniden bağlanma döngüsü**

- Belirti: `channels status` `running, disconnected` gösterir veya “Linked but disconnected” uyarır.
- Çözüm: `openclaw doctor` (veya gateway’i yeniden başlatın). Devam ederse, `channels login` ile yeniden bağlayın ve `openclaw logs --follow`’i inceleyin.

**Bun çalışma zamanı**

- Bun **önerilmez**. WhatsApp (Baileys) ve Telegram Bun üzerinde güvenilir değildir.
  Gateway’i **Node** ile çalıştırın. (Başlarken çalışma zamanı notuna bakın.)

