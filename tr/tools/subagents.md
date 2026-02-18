---
title: "50. Alt Ajanlar"
---

# 50. Alt Ajanlar

Alt ajanlar, mevcut bir ajan çalışmasından başlatılan arka plan ajan çalıştırmalarıdır. Kendi oturumlarında (`agent:<agentId>:subagent:<uuid>`) çalışırlar ve tamamlandıklarında sonucu talepte bulunan sohbet kanalına **duyururlar**.

## Eğik çizgi komutu

**Geçerli oturum** için alt ajan çalışmalarını incelemek veya kontrol etmek üzere `/subagents` kullanın:

- `/subagents list`
- `/subagents kill <id|#|all>`
- `/subagents log <id|#> [limit] [tools]`
- `/subagents info <id|#>`
- `/subagents send <id|#> <message>`

`/subagents info`, çalışma meta verilerini gösterir (durum, zaman damgaları, oturum kimliği, döküm yolu, temizleme).

Temel amaçlar:

- Ana çalışmayı engellemeden “araştırma / uzun görev / yavaş araç” işlerini paralelleştirmek.
- Alt ajanları varsayılan olarak izole tutmak (oturum ayrımı + isteğe bağlı sandbox).
- Araç yüzeyini kötüye kullanıma kapalı tutmak: alt ajanlar varsayılan olarak oturum araçlarını **almaz**.
- Orkestratör desenleri için yapılandırılabilir iç içe derinlik desteği.

Maliyet notu: her alt ajanın **kendi** bağlamı ve token kullanımı vardır. Ağır veya tekrarlı görevler için alt ajanlarda daha ucuz bir model kullanın, ana ajanı ise daha yüksek kaliteli bir modelde tutun. Bunu `agents.defaults.subagents.model` veya ajan başına geçersiz kılmalar ile yapılandırabilirsiniz.

## Araç

`sessions_spawn` kullanın:

- Bir alt ajan çalıştırması başlatır (`deliver: false`, global lane: `subagent`)
- Ardından bir duyuru adımı çalıştırır ve duyuru yanıtını talepte bulunan sohbet kanalına gönderir
- Varsayılan model: `agents.defaults.subagents.model` (veya ajan başına `agents.list[].subagents.model`) ayarlamadığınız sürece çağıranı devralır; açık bir `sessions_spawn.model` her zaman önceliklidir.
- Varsayılan düşünme: `agents.defaults.subagents.thinking` (veya ajan başına `agents.list[].subagents.thinking`) ayarlamadığınız sürece çağıranı devralır; açık bir `sessions_spawn.thinking` her zaman önceliklidir.

Araç parametreleri:

- `task` (zorunlu)
- `label?` (isteğe bağlı)
- `agentId?` (isteğe bağlı; izin verildiyse başka bir ajan kimliği altında başlatır)
- `model?` (isteğe bağlı; alt ajan modelini geçersiz kılar; geçersiz değerler atlanır ve araç sonucunda bir uyarıyla birlikte varsayılan model kullanılır)
- `thinking?` (isteğe bağlı; alt ajan çalıştırması için düşünme seviyesini geçersiz kılar)
- `runTimeoutSeconds?` (varsayılan `0`; ayarlandığında alt ajan çalıştırması N saniye sonra iptal edilir)
- `cleanup?` (`delete|keep`, varsayılan `keep`)

İzin listesi:

- `agents.list[].subagents.allowAgents`: `agentId` ile hedeflenebilecek ajan kimliklerinin listesi (`["*"]` herhangi birine izin verir). Varsayılan: yalnızca talepte bulunan ajan.

Keşif:

- `sessions_spawn` için şu anda hangi ajan kimliklerine izin verildiğini görmek üzere `agents_list` kullanın.

Otomatik arşivleme:

- Alt ajan oturumları `agents.defaults.subagents.archiveAfterMinutes` süresi sonunda otomatik olarak arşivlenir (varsayılan: 60).
- Arşivleme `sessions.delete` kullanır ve dökümü `*.deleted.<timestamp>` olarak yeniden adlandırır (aynı klasörde).
- `cleanup: "delete"` duyurudan hemen sonra arşivler (döküm yeniden adlandırılarak korunur).
- Otomatik arşivleme en iyi çaba esaslıdır; gateway yeniden başlatılırsa bekleyen zamanlayıcılar kaybolur.
- `runTimeoutSeconds` otomatik arşivleme yapmaz; yalnızca çalıştırmayı durdurur. Oturum, otomatik arşivleme gerçekleşene kadar kalır.
- Otomatik arşivleme hem derinlik-1 hem de derinlik-2 oturumlara eşit şekilde uygulanır.

## İç İçe Alt Ajanlar

Varsayılan olarak alt ajanlar kendi alt ajanlarını başlatamaz (`maxSpawnDepth: 1`). `maxSpawnDepth: 2` ayarlayarak bir seviye iç içe çalışmayı etkinleştirebilirsiniz; bu da **orkestratör deseni**ni mümkün kılar: ana → orkestratör alt ajan → çalışan alt-alt ajanlar.

### Nasıl etkinleştirilir

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxSpawnDepth: 2, // alt ajanların çocuk başlatmasına izin ver (varsayılan: 1)
        maxChildrenPerAgent: 5, // ajan oturumu başına maksimum aktif çocuk (varsayılan: 5)
        maxConcurrent: 8, // global eşzamanlılık üst sınırı (varsayılan: 8)
      },
    },
  },
}
```

### Derinlik seviyeleri

| Depth | Session key shape                            | Rol                                           | Başlatabilir mi?              |
| ----- | -------------------------------------------- | --------------------------------------------- | ----------------------------- |
| 0     | `agent:<id>:main`                            | Ana ajan                                      | Her zaman                    |
| 1     | `agent:<id>:subagent:<uuid>`                 | Alt ajan (derinlik 2 izinliyse orkestratör)   | Yalnızca `maxSpawnDepth >= 2` |
| 2     | `agent:<id>:subagent:<uuid>:subagent:<uuid>` | Alt-alt ajan (çalışan)                        | Asla                         |

### Duyuru zinciri

Sonuçlar zincir boyunca yukarı akar:

1. Derinlik-2 çalışan tamamlar → ebeveynine (derinlik-1 orkestratör) duyurur
2. Derinlik-1 orkestratör duyuruyu alır, sonuçları sentezler, tamamlar → ana ajana duyurur
3. Ana ajan duyuruyu alır ve kullanıcıya iletir

Her seviye yalnızca doğrudan çocuklarından gelen duyuruları görür.

### Derinliğe göre araç politikası

- **Derinlik 1 (orkestratör, `maxSpawnDepth >= 2` iken)**: Çocuklarını yönetebilmesi için `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` alır. Diğer oturum/sistem araçları yine reddedilir.
- **Derinlik 1 (çalışan, `maxSpawnDepth == 1` iken)**: Oturum araçları yoktur (mevcut varsayılan davranış).
- **Derinlik 2 (çalışan)**: Oturum araçları yoktur — derinlik 2’de `sessions_spawn` her zaman reddedilir. Daha fazla çocuk başlatamaz.

### Ajan başına başlatma limiti

Her ajan oturumu (herhangi bir derinlikte) aynı anda en fazla `maxChildrenPerAgent` (varsayılan: 5) aktif çocuğa sahip olabilir. Bu, tek bir orkestratörden kontrolsüz fan-out oluşmasını engeller.

### Zincirleme durdurma

Bir derinlik-1 orkestratörü durdurmak, tüm derinlik-2 çocuklarını otomatik olarak durdurur:

- Ana sohbette `/stop`, tüm derinlik-1 ajanları durdurur ve onların derinlik-2 çocuklarına zincirleme etki eder.
- `/subagents kill <id>`, belirli bir alt ajanı durdurur ve çocuklarına zincirleme etki eder.
- `/subagents kill all`, talepte bulunan için tüm alt ajanları durdurur ve zincirleme etki eder.

## Kimlik Doğrulama

Alt ajan kimlik doğrulaması oturum türüne göre değil, **ajan kimliğine** göre çözülür:

- Alt ajan oturum anahtarı `agent:<agentId>:subagent:<uuid>` şeklindedir.
- Kimlik doğrulama deposu ilgili ajanın `agentDir` dizininden yüklenir.
- Ana ajanın kimlik doğrulama profilleri **yedek** olarak birleştirilir; çakışmalarda ajan profilleri önceliklidir.

Not: Birleştirme ekleyicidir, bu nedenle ana profiller her zaman yedek olarak kullanılabilir. Ajan başına tamamen izole kimlik doğrulama henüz desteklenmemektedir.

## Duyuru

Alt ajanlar bir duyuru adımı aracılığıyla geri bildirim yapar:

- Duyuru adımı alt ajan oturumu içinde çalışır (talepte bulunan oturumda değil).
- Alt ajan tam olarak `ANNOUNCE_SKIP` yanıtını verirse hiçbir şey gönderilmez.
- Aksi takdirde duyuru yanıtı, talepte bulunan sohbet kanalına bir takip `agent` çağrısı (`deliver=true`) ile gönderilir.
- Duyuru yanıtları, mevcut olduğunda iş parçacığı/konu yönlendirmesini korur (Slack threads, Telegram topics, Matrix threads).
- Duyuru mesajları sabit bir şablona normalize edilir:
  - `Status:` çalışma sonucundan türetilir (`success`, `error`, `timeout` veya `unknown`).
  - `Result:` duyuru adımındaki özet içerik (yoksa `(not available)`).
  - `Notes:` hata ayrıntıları ve diğer yararlı bağlam.
- `Status`, model çıktısından çıkarılmaz; çalışma zamanı sinyallerinden gelir.

Duyuru yükleri, sonda bir istatistik satırı içerir (sarmalanmış olsa bile):

- Çalışma süresi (ör. `runtime 5m12s`)
- Token kullanımı (input/output/total)
- Model fiyatlandırması yapılandırılmışsa tahmini maliyet (`models.providers.*.models[].cost`)
- `sessionKey`, `sessionId` ve döküm yolu (ana ajanın `sessions_history` ile geçmişi alabilmesi veya dosyayı diskten inceleyebilmesi için)

## Araç Politikası (alt ajan araçları)

Varsayılan olarak alt ajanlar **oturum araçları** ve sistem araçları hariç tüm araçları alır:

- `sessions_list`
- `sessions_history`
- `sessions_send`
- `sessions_spawn`

`maxSpawnDepth >= 2` olduğunda, derinlik-1 orkestratör alt ajanlar ayrıca çocuklarını yönetebilmek için `sessions_spawn`, `subagents`, `sessions_list` ve `sessions_history` alır.

Yapılandırma ile geçersiz kılma:

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 1,
      },
    },
  },
  tools: {
    subagents: {
      tools: {
        // deny her zaman kazanır
        deny: ["gateway", "cron"],
        // allow ayarlanırsa yalnızca izin verilenler geçerli olur (deny yine kazanır)
        // allow: ["read", "exec", "process"]
      },
    },
  },
}
```

## Eşzamanlılık

Alt ajanlar ayrılmış bir süreç içi kuyruk hattı kullanır:

- Hat adı: `subagent`
- Eşzamanlılık: `agents.defaults.subagents.maxConcurrent` (varsayılan `8`)

## Durdurma

- Talepte bulunan sohbette `/stop` göndermek, talep eden oturumu iptal eder ve ondan başlatılmış tüm aktif alt ajan çalışmalarını durdurur; iç içe çocuklara zincirleme etki eder.
- `/subagents kill <id>`, belirli bir alt ajanı durdurur ve çocuklarına zincirleme etki eder.

## Sınırlamalar

- Alt ajan duyurusu **en iyi çaba** esaslıdır. Gateway yeniden başlatılırsa bekleyen “geri duyuru” işleri kaybolur.
- Alt ajanlar aynı gateway süreci kaynaklarını paylaşır; `maxConcurrent` bir güvenlik supabı olarak düşünülmelidir.
- `sessions_spawn` her zaman engelleyici değildir: hemen `{ status: "accepted", runId, childSessionKey }` döner.
- Alt ajan bağlamı yalnızca `AGENTS.md` + `TOOLS.md` enjekte eder (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` veya `BOOTSTRAP.md` yoktur).
- Maksimum iç içe derinlik 5’tir (`maxSpawnDepth` aralığı: 1–5). Çoğu kullanım durumu için derinlik 2 önerilir.
- `maxChildrenPerAgent`, oturum başına aktif çocuk sayısını sınırlar (varsayılan: 5, aralık: 1–20).

## Ayrıca Bakın

- [Oturum Araçları](/concepts/session-tool) — `sessions_spawn` ve diğer oturum araçları hakkında ayrıntılar
- [Çok Ajanlı Sandbox ve Araçlar](/tools/multi-agent-sandbox-tools) — ajan başına araç kısıtlamaları ve sandboxing
- [Yapılandırma](/gateway/configuration) — `agents.defaults.subagents` referansı
- [Kuyruk](/concepts/queue) — `subagent` hattının nasıl çalıştığı