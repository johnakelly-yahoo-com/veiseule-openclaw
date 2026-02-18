---
status: experimental
title: "Yayın Grupları"
---

# Yayın Grupları

**Durum:** Deneysel  
**Sürüm:** 2026.1.9 sürümünde eklendi

## Genel Bakış

Yayın Grupları, birden fazla ajanın aynı mesajı eşzamanlı olarak işlemesini ve yanıtlamasını sağlar. Bu, tek bir WhatsApp grubu veya DM içinde birlikte çalışan, uzmanlaşmış ajan ekipleri oluşturmanıza olanak tanır — hepsi tek bir telefon numarası kullanarak.

Mevcut kapsam: **Yalnızca WhatsApp** (web kanalı).

Yayın grupları, kanal izin listeleri ve grup etkinleştirme kurallarından sonra değerlendirilir. WhatsApp gruplarında bu, OpenClaw normalde ne zaman yanıt verecekse (örneğin: grup ayarlarınıza bağlı olarak bahsedildiğinde) yayınların da o zaman gerçekleştiği anlamına gelir.

## Kullanım Senaryoları

### 1. Uzmanlaşmış Ajan Ekipleri

Atomik ve odaklı sorumluluklara sahip birden fazla ajanı devreye alın:

```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Her ajan aynı mesajı işler ve kendi uzmanlık perspektifini sunar.

### 2. Çok Dilli Destek

```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Kalite Güvence İş Akışları

```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Görev Otomasyonu

```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## Yapılandırma

### Temel Kurulum

Üst düzeyde bir `broadcast` bölümü ekleyin (`bindings`’nin yanına). Anahtarlar WhatsApp eş kimlikleridir:

- grup sohbetleri: grup JID’si (örn. `120363403215116621@g.us`)
- DM’ler: E.164 telefon numarası (örn. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**Sonuç:** OpenClaw bu sohbette normalde yanıt vereceği zaman, üç ajanın tamamını çalıştırır.

### İşleme Stratejisi

Ajanların mesajları nasıl işleyeceğini kontrol edin:

#### Paralel (Varsayılan)

Tüm ajanlar eşzamanlı olarak işler:

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Ardışık

Ajanlar sırayla işlem yapar (öncekinin bitmesini bekler):

```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Eksiksiz Örnek

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## Nasıl Çalışır

### Mesaj Akışı

1. **Gelen mesaj** bir WhatsApp grubuna ulaşır
2. **Yayın kontrolü**: Sistem, eş kimliğin `broadcast` içinde olup olmadığını kontrol eder
3. **Yayın listesinde ise**:
   - Listelenen tüm ajanlar mesajı işler
   - Her ajanın kendi oturum anahtarı ve yalıtılmış bağlamı vardır
   - Ajanlar paralel (varsayılan) veya sıralı olarak işler
4. **Yayın listesinde değilse**:
   - Normal yönlendirme uygulanır (ilk eşleşen bağlama)

Not: yayın grupları kanal izin listelerini veya grup etkinleştirme kurallarını (bahsetmeler/komutlar vb.) atlatmaz. Yalnızca bir mesaj işleme için uygun olduğunda _hangi ajanların çalışacağını_ değiştirir.

### Oturum Yalıtımı

Bir yayın grubundaki her ajan tamamen ayrı olan şunları korur:

- **Oturum anahtarları** (`agent:alfred:whatsapp:group:120363...` ile `agent:baerbel:whatsapp:group:120363...`)
- **Konuşma geçmişi** (ajan diğer ajanların mesajlarını görmez)
- **Çalışma alanı** (yapılandırılmışsa ayrı sandbox’lar)
- **Araç erişimi** (farklı izin/verme listeleri)
- **Bellek/bağlam** (ayrı IDENTITY.md, SOUL.md vb.)
- **Grup bağlam arabelleği** (bağlam için kullanılan son grup mesajları) eş başına paylaşılır; bu nedenle tetiklendiğinde tüm yayın ajanları aynı bağlamı görür

Bu, her ajanın şunlara sahip olmasını sağlar:

- Farklı kişilikler
- Farklı araç erişimleri (örn. salt okunur vs. okuma-yazma)
- Farklı modeller (örn. opus vs. sonnet)
- Farklı Skills kurulumları

### Örnek: Yalıtılmış Oturumlar

`120363403215116621@g.us` grubunda, `["alfred", "baerbel"]` ajanları ile:

**Alfred’in bağlamı:**

```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/openclaw-alfred/
Tools: read, write, exec
```

**Bärbel’in bağlamı:**

```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/openclaw-baerbel/
Tools: read only
```

## En İyi Uygulamalar

### 1. Ajanları Odaklı Tutun

Her ajanı tek ve net bir sorumlulukla tasarlayın:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **İyi:** Her ajanın tek bir işi vardır  
❌ **Kötü:** Tek bir genel “dev-helper” ajan

### 2. Açıklayıcı İsimler Kullanın

Her ajanın ne yaptığını netleştirin:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Farklı Araç Erişimleri Yapılandırın

Ajanlara yalnızca ihtiyaç duydukları araçları verin:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] } // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] } // Read-write
    }
  }
}
```

### 4. Performansı İzleyin

Çok sayıda ajanla şunları göz önünde bulundurun:

- Hız için `"strategy": "parallel"` (varsayılan) kullanımı
- Yayın gruplarını 5–10 ajanla sınırlama
- Daha basit ajanlar için daha hızlı modeller kullanma

### 5. Hataları Zarif Şekilde Yönetin

Ajanlar bağımsız olarak hata verir. Bir ajanın hatası diğerlerini engellemez:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Uyumluluk

### Sağlayıcılar

Yayın grupları şu anda şunlarla çalışır:

- ✅ WhatsApp (uygulandı)
- 🚧 Telegram (planlanıyor)
- 🚧 Discord (planlanıyor)
- 🚧 Slack (planlanıyor)

### Yönlendirme

Yayın grupları mevcut yönlendirme ile birlikte çalışır:

```json
{
  "bindings": [
    {
      "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } },
      "agentId": "alfred"
    }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Yalnızca alfred yanıtlar (normal yönlendirme)
- `GROUP_B`: agent1 VE agent2 yanıtlar (yayın)

**Öncelik:** `broadcast`, `bindings`’ye göre önceliklidir.

## Sorun Giderme

### Ajanlar Yanıt Vermiyor

**Kontrol edin:**

1. Ajan kimliklerinin `agents.list` içinde mevcut olması
2. Eş kimliği biçiminin doğru olması (örn. `120363403215116621@g.us`)
3. Ajanlar engelleme listelerinde değil

**Hata Ayıklama:**

```bash
tail -f ~/.openclaw/logs/gateway.log | grep broadcast
```

### Yalnızca Bir Ajan Yanıt Veriyor

**Neden:** Eş kimliği `bindings` içinde olabilir ancak `broadcast` içinde olmayabilir.

**Çözüm:** Yayın yapılandırmasına ekleyin veya bağlamalardan kaldırın.

### Performans Sorunları

**Çok sayıda ajanla yavaşsa:**

- Grup başına ajan sayısını azaltın
- Daha hafif modeller kullanın (opus yerine sonnet)
- sandbox başlatma süresini kontrol edin

## Örnekler

### Örnek 1: Kod İnceleme Ekibi

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      {
        "id": "code-formatter",
        "workspace": "~/agents/formatter",
        "tools": { "allow": ["read", "write"] }
      },
      {
        "id": "security-scanner",
        "workspace": "~/agents/security",
        "tools": { "allow": ["read", "exec"] }
      },
      {
        "id": "test-coverage",
        "workspace": "~/agents/testing",
        "tools": { "allow": ["read", "exec"] }
      },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**Kullanıcı gönderir:** Kod parçası  
**Yanıtlar:**

- code-formatter: "Girintiyi düzelttim ve tür ipuçları ekledim"
- security-scanner: "⚠️ 12. satırda SQL enjeksiyonu zafiyeti"
- test-coverage: "Kapsama %45, hata durumları için testler eksik"
- docs-checker: "`process_data` fonksiyonu için docstring eksik"

### Örnek 2: Çok Dilli Destek

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## API Referansı

### Yapılandırma Şeması

```typescript
interface OpenClawConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Alanlar

- `strategy` (isteğe bağlı): Ajanların nasıl işleneceği
  - `"parallel"` (varsayılan): Tüm ajanlar eşzamanlı işlem yapar
  - `"sequential"`: Ajanlar dizi sırasına göre işlem yapar
- `[peerId]`: WhatsApp grup JID’si, E.164 numarası veya diğer eş kimliği
  - Değer: Mesajları işlemesi gereken ajan kimliklerinin dizisi

## Sınırlamalar

1. **Maks. ajanlar:** Kesin bir sınır yok, ancak 10+ ajan yavaş olabilir
2. **Paylaşılan bağlam:** Ajanlar birbirlerinin yanıtlarını görmez (tasarım gereği)
3. **Mesaj sıralaması:** Paralel yanıtlar herhangi bir sırayla gelebilir
4. **Hız sınırları:** Tüm ajanlar WhatsApp hız sınırlarına dahil edilir

## Gelecek Geliştirmeler

Planlanan özellikler:

- [ ] Paylaşılan bağlam modu (ajanlar birbirlerinin yanıtlarını görür)
- [ ] Ajan koordinasyonu (ajanlar birbirlerine sinyal gönderebilir)
- [ ] Dinamik ajan seçimi (mesaj içeriğine göre ajan seçme)
- [ ] Ajan öncelikleri (bazı ajanlar diğerlerinden önce yanıtlar)

## Ayrıca Bakınız

- [Çoklu Ajan Yapılandırması](/tools/multi-agent-sandbox-tools)
- [Yönlendirme Yapılandırması](/channels/channel-routing)
- [Oturum Yönetimi](/concepts/sessions)

