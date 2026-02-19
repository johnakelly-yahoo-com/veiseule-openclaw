---
title: IRC
description: OpenClaw’ı IRC kanallarına ve özel mesajlara bağlayın.
---

OpenClaw’ı klasik kanallarda (`#room`) ve özel mesajlarda kullanmak istediğinizde IRC’yi tercih edin.
IRC bir eklenti olarak gelir, ancak ana yapılandırmada `channels.irc` altında ayarlanır.

## Hızlı başlangıç

1. `~/.openclaw/openclaw.json` içinde IRC yapılandırmasını etkinleştirin.
2. En azından şunları ayarlayın:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3. Gateway’i başlatın/yeniden başlatın:

```bash
openclaw gateway run
```

## Varsayılan güvenlik ayarları

- `channels.irc.dmPolicy` varsayılan olarak `"pairing"` değerine ayarlıdır.
- `channels.irc.groupPolicy` varsayılan olarak `"allowlist"` değerine ayarlıdır.
- `groupPolicy="allowlist"` ile izin verilen kanalları tanımlamak için `channels.irc.groups` ayarını yapın.
- Bilinçli olarak düz metin iletimini kabul etmiyorsanız TLS kullanın (`channels.irc.tls=true`).

## Erişim kontrolü

IRC kanalları için iki ayrı “geçit” vardır:

1. **Kanal erişimi** (`groupPolicy` + `groups`): Botun bir kanaldan gelen mesajları genel olarak kabul edip etmeyeceği.
2. **Gönderen erişimi** (`groupAllowFrom` / kanal bazlı `groups["#channel"].allowFrom`): O kanal içinde botu kimin tetikleyebileceği.

Yapılandırma anahtarları:

- DM allowlist (DM gönderen erişimi): `channels.irc.allowFrom`
- Grup gönderen allowlist (kanal gönderen erişimi): `channels.irc.groupAllowFrom`
- Kanal bazlı kontroller (kanal + gönderen + mention kuralları): `channels.irc.groups["#channel"]`
- `channels.irc.groupPolicy="open"` yapılandırılmamış kanallara izin verir (**varsayılan olarak yine mention-gated’dir**)

Allowlist girdileri nick veya `nick!user@host` biçimlerini kullanabilir.

### Yaygın bir hata: `allowFrom` DM’ler içindir, kanallar için değil

Şu tür günlük kayıtları görürseniz:

- `irc: drop group sender alice!ident@host (policy=allowlist)`

…bu, gönderenin **grup/kanal** mesajları için izinli olmadığı anlamına gelir. Bunu şu yollardan biriyle düzeltin:

- `channels.irc.groupAllowFrom` ayarlayarak (tüm kanallar için genel), veya
- kanal bazlı gönderen allowlist’leri ayarlayarak: `channels.irc.groups["#channel"].allowFrom`

Örnek (`#tuirc-dev` içindeki herkesin botla konuşmasına izin vermek için):

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## Yanıt tetikleme (mention’lar)

Bir kanala izin verilmiş olsa bile (`groupPolicy` + `groups` aracılığıyla) ve gönderen de izinli olsa, OpenClaw grup bağlamlarında varsayılan olarak **mention-gating** kullanır.

Bu, botla eşleşen bir mention deseni içermediği sürece `drop channel …` gibi günlük kayıtları görebileceğiniz anlamına gelir. (missing-mention)\` unless the message includes a mention pattern that matches the bot.

Botun bir IRC kanalında **mention gerektirmeden** yanıt vermesini sağlamak için, o kanal için mention gating’i devre dışı bırakın:

```json5
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

Veya **tüm** IRC kanallarına izin vermek (kanal bazlı allowlist olmadan) ve yine de bahsetme olmadan yanıt vermek için:

```json5
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## Güvenlik notu (herkese açık kanallar için önerilir)

Genel bir kanalda `allowFrom: ["*"]` ayarlarsanız, herkes botu tetikleyebilir.
Riski azaltmak için o kanal için araçları kısıtlayın.

### Kanalda herkes için aynı araçlar

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### Göndericiye göre farklı araçlar (sahip daha fazla yetki alır)

`"*"` için daha katı, kendi takma adınız için daha esnek bir politika uygulamak üzere `toolsBySender` kullanın:

```json5
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            eigen: {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

Notlar:

- `toolsBySender` anahtarları bir takma ad (ör. `"eigen"`) veya daha güçlü kimlik eşleştirmesi için tam bir hostmask (`"eigen!~eigen@174.127.248.171"`) olabilir.
- İlk eşleşen gönderici politikası geçerli olur; `"*"` joker karakter geri dönüş seçeneğidir.

Grup erişimi ile bahsetme zorunluluğu (ve nasıl etkileştikleri) hakkında daha fazla bilgi için bkz: [/channels/groups](/channels/groups).

## NickServ

Bağlantıdan sonra NickServ ile kimlik doğrulamak için:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

Bağlantı sırasında isteğe bağlı tek seferlik kayıt:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

Takma ad kaydedildikten sonra tekrar eden REGISTER denemelerini önlemek için `register` ayarını devre dışı bırakın.

## Ortam değişkenleri

Varsayılan hesap şunları destekler:

- `IRC_HOST`
- `IRC_PORT`
- `IRC_TLS`
- `IRC_NICK`
- `IRC_USERNAME`
- `IRC_REALNAME`
- `IRC_PASSWORD`
- `IRC_CHANNELS` (virgülle ayrılmış)
- `IRC_NICKSERV_PASSWORD`
- `IRC_NICKSERV_REGISTER_EMAIL`

## Sorun giderme

- Bot bağlanıyor ancak kanallarda hiç yanıt vermiyorsa, `channels.irc.groups` ayarını **ve** bahsetme zorunluluğunun mesajları engelleyip engellemediğini (`missing-mention`) kontrol edin. Ping olmadan yanıt vermesini istiyorsanız, kanal için `requireMention:false` ayarlayın.
- Giriş başarısız olursa, takma adın kullanılabilirliğini ve sunucu parolasını doğrulayın.
- Özel bir ağda TLS başarısız olursa, host/port ve sertifika yapılandırmasını doğrulayın.
