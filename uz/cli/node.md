---
title: "node"
---

# `openclaw node`

Gateway WebSocket’iga ulanadigan va ushbu mashinada `system.run` / `system.which` ni taqdim etadigan **headless node host** ni ishga tushiring.

## Nega node host’dan foydalanish kerak?

Tarmog‘ingizdagi **boshqa mashinalarda buyruqlarni bajarish** uchun, u yerlarga to‘liq macOS hamroh ilovasini o‘rnatmasdan, node host’dan foydalaning.

Keng tarqalgan foydalanish holatlari:

- Masofadagi Linux/Windows mashinalarida buyruqlarni bajarish (build serverlar, laboratoriya mashinalari, NAS).
- Gateway’da exec’ni **sandbox** holatda saqlang, lekin tasdiqlangan bajarishlarni boshqa xostlarga topshiring.
- Avtomatlashtirish yoki CI tugunlari uchun yengil, headless bajarish maqsadini taqdim etish.

Bajarish hali ham **exec tasdiqlari** va tugun xostidagi har bir agent uchun ruxsat etilgan ro‘yxatlar bilan himoyalangan, shuning uchun buyruqlarga kirishni aniq va cheklangan holda saqlashingiz mumkin.

## Browser proxy (nol konfiguratsiya)

Agar tugunda `browser.enabled` o‘chirilmagan bo‘lsa, tugun xostlari avtomatik ravishda brauzer proksini e’lon qiladi. Bu agentga qo‘shimcha sozlamalarsiz shu tugunda brauzer avtomatlashtirishdan foydalanish imkonini beradi.

Agar kerak bo‘lsa, tugunda o‘chiring:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false,
    },
  },
}
```

## Run (oldingi rejim)

```bash
openclaw node run --host <gateway-host> --port 18789
```

Variantlar:

- `--host <host>`: Gateway WebSocket xosti (standart: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket porti (standart: `18789`)
- `--tls`: Gateway ulanishi uchun TLS dan foydalanish
- `--tls-fingerprint <sha256>`: Kutilayotgan TLS sertifikat barmoq izi (sha256)
- `--node-id <id>`: Tugun identifikatorini almashtirish (juftlash tokenini tozalaydi)
- `--display-name <name>`: Tugun ko‘rinadigan nomini almashtirish

## Service (orqa fonda)

Foydalanuvchi xizmati sifatida headless tugun xostini o‘rnating.

```bash
openclaw node install --host <gateway-host> --port 18789
```

Variantlar:

- `--host <host>`: Gateway WebSocket xosti (standart: `127.0.0.1`)
- `--port <port>`: Gateway WebSocket porti (standart: `18789`)
- `--tls`: Gateway ulanishi uchun TLS dan foydalanish
- `--tls-fingerprint <sha256>`: Kutilayotgan TLS sertifikat barmoq izi (sha256)
- `--node-id <id>`: Tugun identifikatorini almashtirish (juftlash tokenini tozalaydi)
- `--display-name <name>`: Tugun ko‘rinadigan nomini almashtirish
- `--runtime <runtime>`: Xizmat muhiti (`node` yoki `bun`)
- `--force`: Agar allaqachon o‘rnatilgan bo‘lsa, qayta o‘rnatish/ustiga yozish

Xizmatni boshqarish:

```bash
openclaw node status
openclaw node stop
openclaw node restart
openclaw node uninstall
```

Oldingi rejimdagi tugun xosti (xizmatsiz) uchun `openclaw node run` dan foydalaning.

Xizmat buyruqlari mashina o‘qiy oladigan chiqish uchun `--json` ni qabul qiladi.

## Pairing (Juftlash)

Birinchi ulanish Gateway’da kutilayotgan tugun juftlash so‘rovini yaratadi. Uni quyidagilar orqali tasdiqlang:

```bash
openclaw nodes pending
openclaw nodes approve <requestId>
```

Tugun xosti o‘zining tugun identifikatori, tokeni, ko‘rinadigan nomi va gateway ulanish ma’lumotlarini `~/.openclaw/node.json` da saqlaydi.

## Exec approvals (Exec tasdiqlari)

`system.run` mahalliy exec tasdiqlari bilan cheklanadi:

- `~/.openclaw/exec-approvals.json`
- [Exec tasdiqlari](/tools/exec-approvals)
- `openclaw approvals --node <id|name|ip>` (Gateway’dan tahrirlash)
