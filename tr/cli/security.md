---
title: "güvenlik"
---

# `openclaw security`

Güvenlik araçları (denetim + isteğe bağlı düzeltmeler).

İlgili:

- Güvenlik kılavuzu: [Güvenlik](/gateway/security)

## Denetim

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
```

Denetim, birden fazla DM göndereninin ana oturumu paylaştığı durumlarda uyarır ve paylaşılan gelen kutuları için **güvenli DM modu**: `session.dmScope="per-channel-peer"` (çoklu hesap kanalları için `per-account-channel-peer`) önerir.
Ayrıca, sandboxing olmadan ve web/tarayıcı araçları etkinleştirilmiş şekilde küçük modellerin (`<=300B`) kullanılması durumunda da uyarır.
