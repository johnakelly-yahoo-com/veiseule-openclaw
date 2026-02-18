# OpenClaw Tehdit Modeli v1.0

## MITRE ATLAS Çerçevesi

**Sürüm:** 1.0-draft  
**Son Güncelleme:** 2026-02-04  
**Metodoloji:** MITRE ATLAS + Veri Akış Diyagramları  
**Çerçeve:** [MITRE ATLAS](https://atlas.mitre.org/) (Yapay Zekâ Sistemleri için Düşmanca Tehdit Ortamı)

### Çerçeve Atıfı

Bu tehdit modeli, yapay zekâ/ML sistemlerine yönelik düşmanca tehditleri belgelendirmek için endüstri standardı olan [MITRE ATLAS](https://atlas.mitre.org/) temel alınarak hazırlanmıştır. ATLAS, yapay zekâ güvenliği topluluğu ile iş birliği içinde [MITRE](https://www.mitre.org/) tarafından sürdürülmektedir.

**Temel ATLAS Kaynakları:**

- [ATLAS Techniques](https://atlas.mitre.org/techniques/)
- [ATLAS Tactics](https://atlas.mitre.org/tactics/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [ATLAS GitHub](https://github.com/mitre-atlas/atlas-data)
- [Contributing to ATLAS](https://atlas.mitre.org/resources/contribute)

### Bu Tehdit Modeline Katkı

Bu belge, OpenClaw topluluğu tarafından sürdürülen yaşayan bir dokümandır. Katkı yönergeleri için [CONTRIBUTING-THREAT-MODEL.md](./CONTRIBUTING-THREAT-MODEL.md) dosyasına bakın:

- Yeni tehditleri bildirme  
- Mevcut tehditleri güncelleme  
- Saldırı zincirleri önerme  
- Önleyici tedbirler (mitigations) önerme  

---

## 1. Giriş

### 1.1 Amaç

Bu tehdit modeli, özellikle AI/ML sistemleri için tasarlanmış MITRE ATLAS çerçevesini kullanarak OpenClaw AI ajan platformu ve ClawHub beceri pazarına yönelik düşmanca tehditleri dokümante eder.

### 1.2 Kapsam

| Bileşen                | Dahil | Notlar                                           |
| ---------------------- | ----- | ------------------------------------------------ |
| OpenClaw Agent Runtime | Evet  | Temel ajan yürütme, araç çağrıları, oturumlar   |
| Gateway                | Evet  | Kimlik doğrulama, yönlendirme, kanal entegrasyonu |
| Kanal Entegrasyonları  | Evet  | WhatsApp, Telegram, Discord, Signal, Slack vb.  |
| ClawHub Marketplace    | Evet  | Beceri yayımlama, moderasyon, dağıtım           |
| MCP Sunucuları         | Evet  | Harici araç sağlayıcıları                        |
| Kullanıcı Cihazları    | Kısmi | Mobil uygulamalar, masaüstü istemciler          |

### 1.3 Kapsam Dışı

Bu tehdit modeli için açıkça kapsam dışı bırakılmış herhangi bir alan yoktur.

---

## 2. Sistem Mimarisi

### 2.1 Güven Sınırları

```
┌─────────────────────────────────────────────────────────────────┐
│                    GÜVENİLMEYEN BÖLGE                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐              │
│  │  WhatsApp   │  │  Telegram   │  │   Discord   │  ...         │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘              │
│         │                │                │                      │
└─────────┼────────────────┼────────────────┼──────────────────────┘
          │                │                │
          ▼                ▼                ▼
┌─────────────────────────────────────────────────────────────────┐
│            GÜVEN SINIRI 1: Kanal Erişimi                        │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      GATEWAY                              │   │
│  │  • Cihaz Eşleştirme (30 sn tolerans süresi)              │   │
│  │  • AllowFrom / AllowList doğrulaması                      │   │
│  │  • Token/Parola/Tailscale kimlik doğrulama               │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          GÜVEN SINIRI 2: Oturum İzolasyonu                     │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   AGENT OTURUMLARI                       │   │
│  │  • Oturum anahtarı = agent:channel:peer                 │   │
│  │  • Ajan başına araç politikaları                        │   │
│  │  • Transkript kaydı                                     │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          GÜVEN SINIRI 3: Araç Yürütme                          │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                  YÜRÜTME SANDBOX'I                        │   │
│  │  • Docker sandbox VEYA Host (exec-approvals)            │   │
│  │  • Node uzaktan yürütme                                  │   │
│  │  • SSRF koruması (DNS sabitleme + IP engelleme)         │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          GÜVEN SINIRI 4: Harici İçerik                         │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │           GETİRİLEN URL'LER / E-POSTALAR / WEBHOOK'lar  │   │
│  │  • Harici içerik sarmalama (XML etiketleri)             │   │
│  │  • Güvenlik bildirimi ekleme                            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│          GÜVEN SINIRI 5: Tedarik Zinciri                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                      CLAWHUB                              │   │
│  │  • Beceri yayımlama (semver, SKILL.md zorunlu)          │   │
│  │  • Desen tabanlı moderasyon bayrakları                  │   │
│  │  • VirusTotal taraması (yakında)                        │   │
│  │  • GitHub hesap yaşı doğrulaması                        │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 Veri Akışları

| Akış | Kaynak  | Hedef     | Veri                | Koruma                |
| ---- | ------- | --------- | ------------------- | --------------------- |
| F1   | Kanal   | Gateway   | Kullanıcı mesajları | TLS, AllowFrom        |
| F2   | Gateway | Agent     | Yönlendirilmiş mesajlar | Oturum izolasyonu |
| F3   | Agent   | Araçlar   | Araç çağrıları      | Politika uygulama     |
| F4   | Agent   | Harici    | web_fetch istekleri | SSRF engelleme        |
| F5   | ClawHub | Agent     | Beceri kodu         | Moderasyon, tarama    |
| F6   | Agent   | Kanal     | Yanıtlar            | Çıktı filtreleme      |

---

## 3. ATLAS Taktiklerine Göre Tehdit Analizi

### 3.1 Keşif (AML.TA0002)

#### T-RECON-001: Ajan Uç Noktası Keşfi

| Özellik                | Değer                                                               |
| ---------------------- | ------------------------------------------------------------------- |
| **ATLAS ID**           | AML.T0006 - Active Scanning                                         |
| **Açıklama**           | Saldırganın açık OpenClaw gateway uç noktalarını taraması          |
| **Saldırı Vektörü**    | Ağ taraması, shodan sorguları, DNS keşfi                            |
| **Etkilenen Bileşenler** | Gateway, açık API uç noktaları                                    |
| **Mevcut Önlemler**    | Tailscale kimlik doğrulama seçeneği, varsayılan olarak loopback'e bağlama |
| **Kalan Risk**         | Orta - Genel erişime açık gateway'ler keşfedilebilir               |
| **Öneriler**           | Güvenli dağıtımı dokümante edin, keşif uç noktalarına oran sınırlama ekleyin |

#### T-RECON-002: Kanal Entegrasyonu Yoklama

| Özellik                | Değer                                                              |
| ---------------------- | ------------------------------------------------------------------ |
| **ATLAS ID**           | AML.T0006 - Active Scanning                                        |
| **Açıklama**           | Saldırganın AI tarafından yönetilen hesapları tespit etmek için mesajlaşma kanallarını yoklaması |
| **Saldırı Vektörü**    | Test mesajları gönderme, yanıt kalıplarını gözlemleme             |
| **Etkilenen Bileşenler** | Tüm kanal entegrasyonları                                       |
| **Mevcut Önlemler**    | Spesifik bir önlem yok                                            |
| **Kalan Risk**         | Düşük - Sadece keşfin sınırlı değeri var                          |
| **Öneriler**           | Yanıt zamanlamasında rastgeleleştirme değerlendirilebilir         |

---