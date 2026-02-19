---
summary: "Alt ajanlar: sonuçları istekte bulunan sohbet kanalına duyuran, izole ajan çalıştırmaları oluşturma"
read_when:
  - Ajan üzerinden arka plan/paralel çalışma istiyorsunuz
  - sessions_spawn veya alt ajan araç politikasını değiştiriyorsunuz
title: "```\n    /subagents info 1\n    ```"
---

# Alt ajanlar

Alt ajanlar, mevcut bir ajan çalıştırmasından başlatılan arka plan ajan çalıştırmalarıdır. Kendi oturumlarında (`agent:<agentId>:subagent:<uuid>`) çalışırlar ve tamamlandıklarında sonuçlarını talep eden sohbet kanalına **duyururlar**.

## Command

Geçerli oturum için alt aracı çalışmalarını incelemek ve kontrol etmek üzere `/subagents` eğik çizgi komutunu kullanın:

- `/subagents list`
- \`/subagents stop <id\\
- \`/subagents log <id\\
- \`/subagents info <id\\
- \`/subagents send <id\\

`/subagents info`, çalıştırma meta verilerini (durum, zaman damgaları, oturum kimliği, transkript yolu, temizleme) gösterir.

Birincil hedefler:

- Ana çalıştırmayı engellemeden "araştırma / uzun görev / yavaş araç" işlerini paralelleştirmek.
- Alt ajanları varsayılan olarak izole tutmak (oturum ayrımı + isteğe bağlı sandboxing).
- Araç yüzeyini kötüye kullanıma karşı dayanıklı tutmak: alt ajanlar varsayılan olarak oturum araçlarını **almaz**.
- Orkestratör desenleri için yapılandırılabilir iç içe geçme derinliğini desteklemek.

Maliyet notu: her alt ajanın **kendi** bağlamı ve token kullanımı vardır. Ağır veya tekrarlayan
görevler için alt ajanlar adına daha ucuz bir model ayarlayın ve ana ajanınızı daha yüksek kaliteli bir modelde tutun.
Bunu `agents.defaults.subagents.model` veya ajan başına geçersiz kılmalar aracılığıyla yapılandırabilirsiniz.

## Araç

`sessions_spawn` çağrısındaki açık `model` parametresi

- Bir alt ajan çalıştırması başlatır (`deliver: false`, global lane: `subagent`)
- Ardından bir duyuru adımı çalıştırır ve duyuru yanıtını talep eden sohbet kanalına gönderir
- Genel varsayılan: `agents.defaults.subagents.model`
- {
  agents: {
  defaults: {
  subagents: {
  thinking: "low",
  },
  },
  },
  }

Araç parametreleri:

- `task`
- `etiket`
- `agentId?` (isteğe bağlı; izin veriliyorsa başka bir ajan kimliği altında başlatır)
- Model: hedef ajanın normal model seçimi (`subagents.model` ayarlanmadıkça) Düşünme: alt ajan için geçersiz kılma yok (`subagents.thinking` ayarlanmadıkça)
- `thinking?` (isteğe bağlı; alt ajan çalıştırması için düşünme seviyesini geçersiz kılar)
- `runTimeoutSeconds?` (varsayılan `0`; ayarlandığında alt ajan çalıştırması N saniye sonra durdurulur)
- "delete" | "keep"

İzin listesi:

- `agents.list[].subagents.allowAgents`: `agentId` aracılığıyla hedeflenebilecek ajan kimliklerinin listesi (herhangi birine izin vermek için `["*"]`). Varsayılan: yalnızca talep eden ajan.

Keşif:

- <Tip>
`sessions_spawn` için şu anda hangi aracı kimliklerine izin verildiğini keşfetmek için `agents_list` aracını kullanın.
</Tip>

Otomatik Arşivleme

- {
  agents: {
  defaults: {
  subagents: {
  archiveAfterMinutes: 120, // default: 60
  },
  },
  },
  }
- Arşivleme `sessions.delete` kullanır ve transkripti `*.deleted.<timestamp> olarak yeniden adlandırır` (aynı klasör) — dökümler silinmez, korunur.
- "delete" duyurudan hemen sonra arşivler
- Otomatik arşivleme zamanlayıcıları en iyi çaba esaslıdır; ağ geçidi yeniden başlatılırsa bekleyen zamanlayıcılar kaybolur.
- `runTimeoutSeconds`, oturumu otomatik olarak arşivlemez. The session remains until the normal archive timer fires.
- Otomatik arşivleme, derinlik-1 ve derinlik-2 oturumlara eşit şekilde uygulanır.

````
    /subagents list
    ```
````
----

Varsayılan olarak alt ajanlar kendi alt ajanlarını başlatamaz (`maxSpawnDepth: 1`). `maxSpawnDepth: 2` ayarlayarak bir seviye iç içe geçmeyi etkinleştirebilirsiniz; bu, **orkestratör desenine** izin verir: ana → orkestratör alt ajan → çalışan alt-alt ajanlar.

### Nasıl etkinleştirilir

```json5
{
  agents: {
    defaults: {
      subagents: {
        maxConcurrent: 4, // default: 8
      },
    },
  },
}
```

### Derinlik seviyeleri

| Derinlik                          | Oturum anahtarı biçimi                                                                                                                                                                                                                                                                                                                                                                                                                             | Rol                                                                       | Başlatabilir mi?                  |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- | --------------------------------- |
| `0` (sınırsız) | `agentId`                                                                                                                                                                                                                                                                                                                                                                                                                                          | Ana ajan                                                                  | Her zaman                         |
| 1                                 | {&#xA;agents: {&#xA;defaults: {&#xA;subagents: {&#xA;model: "minimax/MiniMax-M2.1",&#xA;},&#xA;},&#xA;},&#xA;}                                                                                                                                                                                                                                                     | Alt ajan (derinlik 2'ye izin verildiğinde orkestratör) | Yalnızca `maxSpawnDepth >= 2` ise |
| 2                                 | {&#xA;agents: {&#xA;list: [&#xA;{&#xA;id: "orchestrator",&#xA;subagents: {&#xA;allowAgents: ["researcher", "coder"], // veya herhangi birine izin vermek için ["\*"]&#xA;},&#xA;},&#xA;],&#xA;},&#xA;} | Alt-alt ajan (uç çalışan)                              | Asla                              |

### Zinciri duyur

Sonuçlar zincir boyunca yukarı akar:

1. Derinlik-2 çalışanı tamamlar → üst öğesine (derinlik-1 orkestratör) duyuru yapar
2. Derinlik-1 orkestratör duyuruyu alır, sonuçları sentezler, tamamlar → ana ajana duyuru yapar
3. Ana ajan duyuruyu alır ve kullanıcıya iletir

Her seviye yalnızca doğrudan alt öğelerinden gelen duyuruları görür.

### Derinliğe göre araç politikası

- **Derinlik 1 (orkestratör, `maxSpawnDepth >= 2` olduğunda)**: Alt öğelerini yönetebilmesi için `sessions_spawn`, `subagents`, `sessions_list`, `sessions_history` araçlarını alır. Diğer oturum/sistem araçları engellenmiş olarak kalır.
- **Derinlik 1 (uç, `maxSpawnDepth == 1` olduğunda)**: Oturum araçları yok (mevcut varsayılan davranış).
- **Derinlik 2 (uç çalışan)**: Oturum araçları yok — `sessions_spawn` derinlik 2'de her zaman engellenir. Daha fazla alt öğe başlatamaz.

### Stop a running sub-agent

Her ajan oturumu (herhangi bir derinlikte) aynı anda en fazla `maxChildrenPerAgent` (varsayılan: 5) etkin alt öğeye sahip olabilir. Bu, tek bir orkestratörden kontrolsüz yayılmayı önler.

### Kademeli durdurma

Bir derinlik-1 orkestratörü durdurmak, tüm derinlik-2 alt öğelerini otomatik olarak durdurur:

- Ana sohbette `/stop`, tüm derinlik-1 ajanlarını durdurur ve derinlik-2 alt öğelerine kademeli olarak yansır.
- `/subagents stop <id>`
- `/subagents kill all`, istekte bulunan için tüm alt ajanları durdurur ve kademeli olarak yansır.

## Kimlik doğrulama

Alt ajan kimlik doğrulaması oturum türüne göre değil, **ajan kimliğine** göre çözülür:

- Alt ajan oturum anahtarı `agent:<agentId>:subagent:<uuid>` biçimindedir.
- Kimlik doğrulama deposu hedef aracının `agentDir` dizininden yüklenir
- Ana ajanın kimlik doğrulama profilleri **yedek** olarak birleştirilir; çakışmalarda ajan profilleri ana profilleri geçersiz kılar.

Birleştirme ekleyicidir — ana profiller her zaman yedek olarak kullanılabilir Ajan başına tamamen izole kimlik doğrulama henüz desteklenmiyor.

## Duyuru

Alt ajanlar bir duyuru adımıyla geri bildirimde bulunur:

- Duyuru adımı alt ajan oturumu içinde çalışır (istekte bulunanın oturumunda değil).
- Alt ajan tam olarak `ANNOUNCE_SKIP` yanıtını verirse hiçbir şey gönderilmez.
- Aksi halde duyuru yanıtı, takip eden bir `agent` çağrısı (`deliver=true`) aracılığıyla istekte bulunanın sohbet kanalına gönderilir.
- Duyuru yanıtları, mevcut olduğunda iş parçacığı/konu yönlendirmesini korur (Slack iş parçacıkları, Telegram konuları, Matrix iş parçacıkları).
- Duyuru mesajları kararlı bir şablona normalize edilir:
  - `Status:` çalıştırma sonucundan türetilir (`success`, `error`, `timeout` veya `unknown`).
  - `Result:` duyuru adımındaki özet içerik (yoksa `(not available)`).
  - `Notes:` hata ayrıntıları ve diğer faydalı bağlam.
- `Status`, model çıktısından çıkarılmaz; çalışma zamanı sonuç sinyallerinden gelir.

Her duyuru, şu bilgileri içeren bir istatistik satırı içerir:

- Çalışma süresi (örn. `runtime 5m12s`)
- Token kullanımı (girdi/çıktı/toplam)
- Tahmini maliyet (`models.providers.*.models[].cost` üzerinden model fiyatlandırması yapılandırıldığında)
- `sessionKey`, `sessionId` ve transkript yolu (böylece ana ajan `sessions_history` ile geçmişi alabilir veya diskteki dosyayı inceleyebilir)

## Alt Aracıları Yönetme (`/subagents`)

Varsayılan olarak, alt ajanlar **oturum araçları** ve sistem araçları hariç tüm araçları alır:

- `sessions_spawn` çağrısındaki açık `thinking` parametresi
- `runTimeoutSeconds`
- `sessions_send`
- `sessions_spawn` Aracı

`maxSpawnDepth >= 2` olduğunda, derinlik-1 orkestratör alt aracılar ayrıca `sessions_spawn`, `subagents`, `sessions_list` ve `sessions_history` alır; böylece kendi alt öğelerini yönetebilirler.

Yapılandırma üzerinden geçersiz kılın:

```json5
{
  tools: {
    subagents: {
      tools: {
        allow: ["read", "exec", "process", "write", "edit", "apply_patch"],
        // ayarlanmışsa reddetme yine kazanır
      },
    },
  },
}
```

## Eşzamanlılık

Alt aracılar, süreç içi ayrılmış bir kuyruk hattı kullanır:

- Hat adı: `subagent`
- Genel varsayılan: `agents.defaults.subagents.thinking`

## Durdurma

- İstek yapanın sohbetinde `/stop` gönderilmesi, istek yapan oturumu sonlandırır ve buradan başlatılan tüm aktif alt aracı çalıştırmalarını durdurur; iç içe geçmiş alt öğelere kadar zincirleme şekilde uygulanır.
- {
  agents: {
  defaults: {
  subagents: {
  model: "minimax/MiniMax-M2.1",
  },
  },
  },
  }

## Sınırlamalar

- Alt aracı duyurusu **en iyi çaba esaslıdır**. Gateway yeniden başlatılırsa, bekleyen "announce back" işleri kaybolur.
- Alt aracılar yine aynı gateway süreç kaynaklarını paylaşır; `maxConcurrent` değerini bir güvenlik supabı olarak değerlendirin.
- Çağrı **engelleyici değildir** — ana ajan hemen `{ status: "accepted", runId, childSessionKey }` yanıtını alır.
- Alt aracı bağlamı yalnızca `AGENTS.md` + `TOOLS.md` enjekte eder (`SOUL.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md` veya `BOOTSTRAP.md` yoktur).
- Maksimum iç içe geçme derinliği 5’tir (`maxSpawnDepth` aralığı: 1–5). Çoğu kullanım durumu için derinlik 2 önerilir.
- `maxChildrenPerAgent`, oturum başına aktif alt öğe sayısını sınırlar (varsayılan: 5, aralık: 1–20).

