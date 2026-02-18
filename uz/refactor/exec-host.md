---
title: "Exec xostni refaktorlash"
---

# Exec xost refaktor rejasi

## Maqsadlar

- `exec.host` + `exec.security` ni qo‘shib, ijroni **sandbox**, **gateway** va **node** bo‘ylab marshrutlash.
- Standartlarni **xavfsiz** saqlash: aniq yoqilmaguncha xostlararo ijro yo‘q.
- Ijroni **UI’siz runner xizmati** ga ajratish, ixtiyoriy UI (macOS ilova) bilan lokal IPC orqali.
- **Har bir agent** uchun siyosat, allowlist, so‘rash rejimi va node bog‘lashni ta’minlash.
- Allowlistsiz ham yoki ular bilan ishlaydigan **so‘rash rejimlari** ni qo‘llab-quvvatlash.
- Kross-platforma: Unix soketi + token autentifikatsiyasi (macOS/Linux/Windows pariteti).

## Maqsadga kirmaydiganlar

- Legacy allowlist migratsiyasi yoki legacy sxema qo‘llab-quvvatlanmaydi.
- Node exec uchun PTY/streaming yo‘q (faqat yig‘ilgan chiqish).
- Mavjud Bridge + Gateway’dan tashqari yangi tarmoq qatlami yo‘q.

## Qarorlar (bloklangan)

- **Konfiguratsiya kalitlari:** `exec.host` + `exec.security` (har bir agent uchun override ruxsat etiladi).
- **Ko‘tarilgan huquq:** gateway to‘liq kirishi uchun `/elevated` ni alias sifatida saqlash.
- **So‘rash standarti:** `on-miss`.
- **Tasdiqlar ombori:** `~/.openclaw/exec-approvals.json` (JSON, legacy migratsiyasiz).
- **Runner:** UI’siz tizim xizmati; UI ilova tasdiqlar uchun Unix soketni joylashtiradi.
- **Node identifikatori:** mavjud `nodeId` dan foydalanish.
- **Soket autentifikatsiyasi:** Unix soketi + token (kross-platforma); kerak bo‘lsa keyin ajratiladi.
- **Node xost holati:** `~/.openclaw/node.json` (node id + juftlash tokeni).
- **macOS exec xosti:** macOS ilovasi ichida `system.run` ni ishga tushirish; node xost xizmati so‘rovlarni lokal IPC orqali uzatadi.
- **XPC helper yo‘q:** Unix soketi + token + peer tekshiruvlariga amal qilish.

## Asosiy tushunchalar

### Xost

- `sandbox`: Docker exec (joriy xatti-harakat).
- `gateway`: gateway xostida exec.
- `node`: Bridge orqali node runner’da exec (`system.run`).

### Xavfsizlik rejimi

- `deny`: har doim bloklash.
- `allowlist`: faqat mos keladiganlarga ruxsat.
- `full`: hammasiga ruxsat (ko‘tarilgan huquqqa teng).

### So‘rash rejimi

- `off`: hech qachon so‘ramaslik.
- `on-miss`: faqat allowlist mos kelmaganda so‘rash.
- `always`: har safar so‘rash.

So‘rash allowlistdan **mustaqil**; allowlist `always` yoki `on-miss` bilan ishlatilishi mumkin.

### Siyosatni aniqlash (har bir exec uchun)

1. `exec.host` ni aniqlash (tool param → agent override → global default).
2. `exec.security` va `exec.ask` ni aniqlash (xuddi shu ustuvorlik).
3. Agar xost `sandbox` bo‘lsa, lokal sandbox exec bilan davom etish.
4. Agar xost `gateway` yoki `node` bo‘lsa, o‘sha xostda xavfsizlik + so‘rash siyosatini qo‘llash.

## Standart xavfsizlik

- Standart `exec.host = sandbox`.
- `gateway` va `node` uchun standart `exec.security = deny`.
- Standart `exec.ask = on-miss` (faqat xavfsizlik ruxsat bersa tegishli).
- 1. Agar tugun bog‘lanishi o‘rnatilmagan bo‘lsa, **agent istalgan tugunni nishonga olishi mumkin**, ammo faqat siyosat ruxsat bergan taqdirdagina.

## 2. Konfiguratsiya yuzasi

### 3. Asbob parametrlari

- 4. `exec.host` (ixtiyoriy): `sandbox | gateway | node`.
- 5. `exec.security` (ixtiyoriy): `deny | allowlist | full`.
- 6. `exec.ask` (ixtiyoriy): `off | on-miss | always`.
- 7. `exec.node` (ixtiyoriy): `host=node` bo‘lganda foydalaniladigan tugun id/nomi.

### 8. Konfiguratsiya kalitlari (global)

- 9. `tools.exec.host`
- 10. `tools.exec.security`
- 11. `tools.exec.ask`
- 12. `tools.exec.node` (standart tugun bog‘lanishi)

### 13. Konfiguratsiya kalitlari (agent bo‘yicha)

- 14. `agents.list[].tools.exec.host`
- 15. `agents.list[].tools.exec.security`
- 16. `agents.list[].tools.exec.ask`
- 17. `agents.list[].tools.exec.node`

### 18. Alias

- 19. `/elevated on` = agent sessiyasi uchun `tools.exec.host=gateway`, `tools.exec.security=full` qilib o‘rnatadi.
- 20. `/elevated off` = agent sessiyasi uchun oldingi exec sozlamalarini tiklaydi.

## 21. Tasdiqlar ombori (JSON)

22. Yo‘l: `~/.openclaw/exec-approvals.json`

23. Maqsad:

- 24. **Ijro xosti** (gateway yoki node runner) uchun mahalliy siyosat + ruxsat ro‘yxatlari.
- 25. UI mavjud bo‘lmaganda so‘rov uchun zaxira mexanizmi.
- 26. UI mijozlari uchun IPC hisob ma’lumotlari.

27. Taklif etilgan sxema (v1):

```json
28. {
  "version": 1,
  "socket": {
    "path": "~/.openclaw/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

29. Eslatmalar:

- 30. Meros (legacy) ruxsat ro‘yxati formatlari yo‘q.
- 31. `askFallback` faqat `ask` talab qilinganda va hech qanday UI bilan aloqa qilib bo‘lmaganda qo‘llaniladi.
- 32. Fayl ruxsatlari: `0600`.

## 33. Runner xizmati (headless)

### 34. Rol

- 35. `exec.security` + `exec.ask` ni mahalliy darajada majburiy qo‘llash.
- 36. Tizim buyruqlarini bajarish va natijani qaytarish.
- 37. Exec hayotiy sikli uchun Bridge hodisalarini chiqarish (ixtiyoriy, ammo tavsiya etiladi).

### 38. Xizmat hayotiy sikli

- 39. macOS’da Launchd/daemon; Linux/Windows’da tizim xizmati.
- 40. Tasdiqlar JSON’i ijro xostiga lokal hisoblanadi.
- 41. UI lokal Unix soketini joylashtiradi; runnerlar talab bo‘yicha ulanadi.

## 42. UI integratsiyasi (macOS ilovasi)

### 43. IPC

- 44. `~/.openclaw/exec-approvals.sock` dagi Unix soketi (0600).
- 45. Token `exec-approvals.json` da saqlanadi (0600).
- Peer tekshiruvlari: faqat bir xil UID.
- 47. Challenge/response: qayta ijroni oldini olish uchun nonce + HMAC(token, request-hash).
- Qisqa TTL (masalan, 10s) + maksimal yuk hajmi + tezlikni cheklash.

### 49. So‘rov oqimi (macOS ilovasi ijro xosti)

1. 50. Tugun xizmati gateway’dan `system.run` ni qabul qiladi.
2. Node xizmati lokal socket’ga ulanadi va prompt/exec so‘rovini yuboradi.
3. Ilova peer + token + HMAC + TTL ni tekshiradi, so‘ng kerak bo‘lsa dialog oynasini ko‘rsatadi.
4. Ilova buyruqni UI kontekstida bajaradi va natijani qaytaradi.
5. Node xizmati natijani gateway’ga qaytaradi.

Agar UI mavjud bo‘lmasa:

- `askFallback` ni qo‘llang (`deny|allowlist|full`).

### Diagramma (SCI)

```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

## Node identifikatsiyasi va bog‘lanishi

- Use existing `nodeId` from Bridge pairing.
- Binding model:
  - `tools.exec.node` restricts the agent to a specific node.
  - If unset, agent can pick any node (policy still enforces defaults).
- Node selection resolution:
  - `nodeId` exact match
  - `displayName` (normalized)
  - `remoteIp`
  - `nodeId` prefix (>= 6 chars)

## Eventing

### Who sees events

- System events are **per session** and shown to the agent on the next prompt.
- Stored in the gateway in-memory queue (`enqueueSystemEvent`).

### Event text

- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + optional output tail
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### Transport

Option A (recommended):

- Runner sends Bridge `event` frames `exec.started` / `exec.finished`.
- Gateway `handleBridgeEvent` maps these into `enqueueSystemEvent`.

Option B:

- Gateway `exec` tool handles lifecycle directly (synchronous only).

## Exec flows

### Sandbox host

- Existing `exec` behavior (Docker or host when unsandboxed).
- PTY supported in non-sandbox mode only.

### Gateway host

- Gateway process executes on its own machine.
- Enforces local `exec-approvals.json` (security/ask/allowlist).

### Node host

- Gateway calls `node.invoke` with `system.run`.
- Runner enforces local approvals.
- Runner returns aggregated stdout/stderr.
- Optional Bridge events for start/finish/deny.

## Output caps

- Cap combined stdout+stderr at **200k**; keep **tail 20k** for events.
- Truncate with a clear suffix (e.g., `"… (truncated)"`).

## Slash commands

- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- 1. Har bir agent va har bir sessiya uchun override’lar; konfiguratsiya orqali saqlanmasa, doimiy emas.
- 2. `/elevated on|off|ask|full` `host=gateway security=full` uchun qisqa yo‘l bo‘lib qoladi (`full` tasdiqlashlarni o‘tkazib yuboradi).

## 3. Kross-platforma hikoyasi

- 4. Runner xizmati — ko‘chma ijro maqsadi.
- UI is optional; if missing, `askFallback` applies.
- 6. Windows/Linux bir xil tasdiqlashlar JSON’i + soket protokolini qo‘llab-quvvatlaydi.

## 7. Amalga oshirish bosqichlari

### 8. 1-bosqich: konfiguratsiya + exec marshrutlash

- 9. `exec.host`, `exec.security`, `exec.ask`, `exec.node` uchun konfiguratsiya sxemasini qo‘shish.
- 10. Asbob infratuzilmasini `exec.host`ni hisobga oladigan qilib yangilash.
- Add `/exec` slash command and keep `/elevated` alias.

### 12. 2-bosqich: tasdiqlashlar ombori + gateway orqali majburiy ijro

- 13. `exec-approvals.json` o‘quvchi/yozuvchini amalga oshirish.
- Enforce allowlist + ask modes for `gateway` host.
- 15. Chiqish (output) limitlarini qo‘shish.

### 16. 3-bosqich: node runner majburiy ijrosi

- 17. Node runner’ni allowlist + ask’ni majburiy qo‘llaydigan qilib yangilash.
- 18. macOS ilovasi UI’iga Unix soket prompt ko‘prigini qo‘shish.
- Wire `askFallback`.

### 20. 4-bosqich: hodisalar

- 21. Exec hayotiy sikli uchun node → gateway Bridge hodisalarini qo‘shish.
- 22. Agent promptlari uchun `enqueueSystemEvent`ga moslash.

### 23. 5-bosqich: UI sayqallash

- 24. Mac ilovasi: allowlist muharriri, har bir agent uchun almashtirgich, ask siyosati UI’i.
- 25. Node bog‘lash boshqaruvlari (ixtiyoriy).

## 26. Sinov rejasi

- Unit tests: allowlist matching (glob + case-insensitive).
- 28. Unit testlar: siyosatni aniqlash ustuvorligi (asbob parametri → agent override → global).
- 29. Integratsion testlar: node runner deny/allow/ask oqimlari.
- 30. Bridge hodisa testlari: node hodisasi → tizim hodisasi marshrutlash.

## Open risks

- 32. UI mavjud emasligi: `askFallback` hurmat qilinishini ta’minlash.
- 33. Uzoq davom etadigan buyruqlar: timeout + output limitlariga tayanish.
- 34. Ko‘p-node noaniqligi: node bog‘lanmagan bo‘lsa yoki aniq node parametri bo‘lmasa, xato.

## 35. Tegishli hujjatlar

- 36. [Exec tool](/tools/exec)
- 37. [Exec approvals](/tools/exec-approvals)
- 38. [Nodes](/nodes)
- 39. [Elevated mode](/tools/elevated)


