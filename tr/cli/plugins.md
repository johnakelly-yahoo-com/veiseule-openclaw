---
summary: "`openclaw plugins` için CLI başvurusu (listeleme, yükleme, etkinleştirme/devre dışı bırakma, doctor)"
read_when:
  - Süreç içi Gateway eklentilerini yüklemek veya yönetmek istiyorsunuz
  - Eklenti yükleme hatalarını ayıklamak istiyorsunuz
title: "plugins"
---

# `openclaw plugins`

Gateway eklentilerini/uzantılarını (süreç içinde yüklenen) yönetin.

İlgili:

- Eklenti sistemi: [Plugins](/tools/plugin)
- Eklenti bildirimi + şema: [Plugin manifest](/plugins/manifest)
- Güvenlik sertleştirme: [Security](/gateway/security)

## Komutlar

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
openclaw plugins update <id>
openclaw plugins update --all
```

Birlikte gelen eklentiler OpenClaw ile birlikte gelir ancak başlangıçta devre dışıdır. Bunları etkinleştirmek için `plugins enable` kullanın.

Tüm eklentiler, satır içi bir JSON Şeması (`configSchema`, boş olsa bile) içeren bir `openclaw.plugin.json` dosyası sağlamalıdır. Eksik/geçersiz bildiriler veya şemalar, eklentinin yüklenmesini engeller ve yapılandırma doğrulamasının başarısız olmasına neden olur.

### Kurulum

```bash
openclaw plugins install <path-or-spec>
```

Güvenlik notu: eklenti kurulumlarını kod çalıştırma gibi değerlendirin. Sabitlenmiş sürümleri tercih edin.

Npm özellikleri yalnızca **registry-only**’dir (paket adı + isteğe bağlı sürüm/etiket). Git/URL/file
özellikleri reddedilir. Bağımlılık kurulumları güvenlik için `--ignore-scripts` ile çalıştırılır.

Desteklenen arşivler: `.zip`, `.tgz`, `.tar.gz`, `.tar`.

Yerel bir dizini kopyalamaktan kaçınmak için `--link` kullanın (`plugins.load.paths`'e ekler):

```bash
openclaw plugins install -l ./my-plugin
```

### Kaldır

```bash
openclaw plugins uninstall <id>
openclaw plugins uninstall <id> --dry-run
openclaw plugins uninstall <id> --keep-files
```

`uninstall`, uygun olduğunda `plugins.entries`, `plugins.installs`, eklenti izin listesi ve bağlantılı `plugins.load.paths` girdilerinden eklenti kayıtlarını kaldırır.
Etkin bellek eklentileri için bellek yuvası `memory-core` olarak sıfırlanır.

Varsayılan olarak uninstall, etkin durum dizini extensions kökü altındaki eklenti kurulum dizinini de kaldırır (`$OPENCLAW_STATE_DIR/extensions/<id>`). Diskteki dosyaları korumak için
`--keep-files` kullanın.

`--keep-config`, `--keep-files` için kullanımdan kaldırılmış bir takma ad olarak desteklenir.

### Update

```bash
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins update <id> --dry-run
```

Güncellemeler yalnızca npm'den yüklenen eklentilere uygulanır (`plugins.installs`'te izlenir).

