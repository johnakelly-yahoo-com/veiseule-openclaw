# OpenClaw Tehdit Modeline Katkıda Bulunma

OpenClaw’u daha güvenli hale getirmeye yardımcı olduğunuz için teşekkürler. Bu tehdit modeli yaşayan bir belgedir ve güvenlik uzmanı olmanız gerekmeksizin herkesten katkı bekliyoruz.

## Ways to Contribute

### Add a Threat

Kapsamadığımız bir saldırı vektörü veya risk mi fark ettiniz? [openclaw/trust](https://github.com/openclaw/trust/issues) üzerinde bir issue açın ve kendi kelimelerinizle açıklayın. Herhangi bir çerçeveyi bilmeniz veya tüm alanları doldurmanız gerekmez - yalnızca senaryoyu açıklamanız yeterlidir.

**Eklemek faydalı olabilir (ancak zorunlu değildir):**

- The attack scenario and how it could be exploited
- OpenClaw’un hangi bölümlerinin etkilendiği (CLI, gateway, channels, ClawHub, MCP servers, vb.)
- Bunun ne kadar ciddi olduğunu düşündüğünüz (düşük / orta / yüksek / kritik)
- İlgili araştırmalara, CVE’lere veya gerçek dünyadan örneklere bağlantılar

We'll handle the ATLAS mapping, threat IDs, and risk assessment during review. If you want to include those details, great - but it's not expected.

> **This is for adding to the threat model, not reporting live vulnerabilities.** If you've found an exploitable vulnerability, see our [Trust page](https://trust.openclaw.ai) for responsible disclosure instructions.

### Suggest a Mitigation

Have an idea for how to address an existing threat? Open an issue or PR referencing the threat. Useful mitigations are specific and actionable - for example, "per-sender rate limiting of 10 messages/minute at the gateway" is better than "implement rate limiting."

### Propose an Attack Chain

Attack chains show how multiple threats combine into a realistic attack scenario. If you see a dangerous combination, describe the steps and how an attacker would chain them together. A short narrative of how the attack unfolds in practice is more valuable than a formal template.

### Fix or Improve Existing Content

Typos, clarifications, outdated info, better examples - PRs welcome, no issue needed.

## Ne Kullanıyoruz

### MITRE ATLAS

Bu tehdit modeli, özellikle istem enjeksiyonu, araç kötüye kullanımı ve ajan istismarı gibi AI/ML tehditleri için tasarlanmış bir çerçeve olan [MITRE ATLAS](https://atlas.mitre.org/) (Yapay Zekâ Sistemleri için Karşıt Tehdit Manzarası) üzerine kuruludur. Katkıda bulunmak için ATLAS'ı bilmeniz gerekmez - gönderimleri inceleme sırasında çerçeveye eşliyoruz.

### 5. Tehdit Kimlikleri

Her tehdit `T-EXEC-003` gibi bir kimlik alır. Kategoriler şunlardır:

| Kod     | Category                                                       |
| ------- | -------------------------------------------------------------- |
| RECON   | Keşif - bilgi toplama                                          |
| ACCESS  | İlk erişim - sisteme giriş elde etme                           |
| EXEC    | Yürütme - kötü amaçlı eylemleri çalıştırma                     |
| PERSIST | Kalıcılık - erişimi sürdürme                                   |
| EVADE   | Savunmadan kaçınma - tespitten kaçma                           |
| DISC    | 19. Keşif - ortam hakkında bilgi edinme |
| EXFIL   | 21. Veri sızdırma - veri çalma          |
| IMPACT  | Etki - hasar veya kesinti                                      |

IDs are assigned by maintainers during review. Birini seçmeniz gerekmez.

### Risk Seviyeleri

| Seviye                               | 28. Anlam                              |
| ------------------------------------ | ------------------------------------------------------------- |
| **Kritik**                           | Tam sistem ele geçirilmesi veya yüksek olasılık + kritik etki |
| **Yüksek**                           | Önemli hasar olası veya orta olasılık + kritik etki           |
| 33. **Orta**  | Orta düzey risk veya düşük olasılık + yüksek etki             |
| 35. **Düşük** | Olasılığı düşük ve sınırlı etki                               |

Risk seviyesi konusunda emin değilseniz, sadece etkiyi açıklayın, biz değerlendirelim.

## İnceleme Süreci

1. **Ön Eleme** - Yeni gönderimleri 48 saat içinde inceliyoruz
2. **Değerlendirme** - Uygulanabilirliği doğrular, ATLAS eşlemesini ve tehdit kimliğini atar, risk seviyesini doğrularız
3. **Dokümantasyon** - Her şeyin biçimlendirilmiş ve eksiksiz olmasını sağlarız
4. **Birleştirme** - Tehdit modeline ve görselleştirmeye eklenir

## 43) Kaynaklar

- [ATLAS Web Sitesi](https://atlas.mitre.org/)
- [ATLAS Teknikleri](https://atlas.mitre.org/techniques/)
- [ATLAS Case Studies](https://atlas.mitre.org/studies/)
- [OpenClaw Tehdit Modeli](./THREAT-MODEL-ATLAS.md)

## İletişim

- **Güvenlik açıkları:** Bildirim talimatları için [Güven sayfamıza](https://trust.openclaw.ai) bakın
- **Tehdit modeli soruları:** [openclaw/trust](https://github.com/openclaw/trust/issues) üzerinde bir konu açın
- **General chat:** Discord #security channel

## Recognition

Contributors to the threat model are recognized in the threat model acknowledgments, release notes, and the OpenClaw security hall of fame for significant contributions.

