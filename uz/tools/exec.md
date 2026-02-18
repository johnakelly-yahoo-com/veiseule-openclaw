---
title: "Exec vositasi"
---

# Exec vositasi

Ish maydonida shell buyruqlarini ishga tushiring. `process` orqali foreground + background bajarilishini qo‘llab-quvvatlaydi.  
Agar `process` ruxsat etilmagan bo‘lsa, `exec` sinxron ishlaydi va `yieldMs`/`background` ni e’tiborsiz qoldiradi.  
Background sessiyalar har bir agent doirasida cheklangan; `process` faqat shu agentga tegishli sessiyalarni ko‘ra oladi.

## Parametrlar

- `command` (majburiy)
- `workdir` (standart: cwd)
- `env` (kalit/qiymat o‘zgartirishlari)
- `yieldMs` (standart 10000): kechikishdan so‘ng avtomatik background
- `background` (bool): darhol background
- `timeout` (sekund, standart 1800): muddati tugaganda to‘xtatadi
- `pty` (bool): mavjud bo‘lsa, pseudo-terminalda ishga tushiradi (faqat TTY CLI’lar, kodlash agentlari, terminal UI’lar)
- `host` (`sandbox | gateway | node`): qayerda bajariladi
- `security` (`deny | allowlist | full`): `gateway`/`node` uchun majburiy nazorat rejimi
- `ask` (`off | on-miss | always`): `gateway`/`node` uchun tasdiqlash so‘rovlari
- `node` (string): `host=node` uchun node identifikatori/nomi
- `elevated` (bool): yuqori rejimni so‘rash (`gateway` host); `security=full` faqat elevated `full` ga o‘tganda majburan qo‘llanadi

Izohlar:

- `host` standart bo‘yicha `sandbox`.
- Agar sandboxing o‘chiq bo‘lsa, `elevated` e’tiborga olinmaydi (`exec` allaqachon hostda ishlaydi).
- `gateway`/`node` tasdiqlashlari `~/.openclaw/exec-approvals.json` orqali boshqariladi.
- `node` juftlangan node’ni talab qiladi (companion ilova yoki headless node host).
- Agar bir nechta node mavjud bo‘lsa, bittasini tanlash uchun `exec.node` yoki `tools.exec.node` ni sozlang.
- Windows bo‘lmagan hostlarda, agar `SHELL` o‘rnatilgan bo‘lsa, exec undan foydalanadi; agar `SHELL` `fish` bo‘lsa, fish bilan mos kelmaydigan skriptlardan qochish uchun `PATH` dan `bash` (yoki `sh`) ni afzal ko‘radi, agar ikkalasi ham bo‘lmasa `SHELL` ga qaytadi.
- Hostda bajarish (`gateway`/`node`) `env.PATH` va loader override’larni (`LD_*`/`DYLD_*`) rad etadi — bu binary hijacking yoki kod kiritilishini oldini olish uchun.
- Muhim: sandboxing **standart bo‘yicha o‘chiq**. Agar sandboxing o‘chiq bo‘lsa, `host=sandbox` buyruqni to‘g‘ridan-to‘g‘ri gateway hostda (konteynersiz) bajaradi va **tasdiqlash talab qilinmaydi**. Tasdiqlashni majburiy qilish uchun `host=gateway` bilan ishga tushiring va exec tasdiqlarini sozlang (yoki sandboxing’ni yoqing).

## Sozlama

- `tools.exec.notifyOnExit` (standart: true): true bo‘lsa, background qilingan exec sessiyalari yakunlanganda tizim hodisasini navbatga qo‘yadi va heartbeat so‘raydi.
- `tools.exec.approvalRunningNoticeMs` (standart: 10000): tasdiqlash talab qiladigan exec shu vaqtdan ko‘proq ishlasa, bitta “running” bildirishnoma yuboradi (0 — o‘chiradi).
- `tools.exec.host` (standart: `sandbox`)
- `tools.exec.security` (standart: sandbox uchun `deny`, gateway + node uchun (agar belgilanmagan bo‘lsa) `allowlist`)
- `tools.exec.ask` (standart: `on-miss`)
- `tools.exec.node` (standart: o‘rnatilmagan)
- `tools.exec.pathPrepend`: exec ishga tushirilganda `PATH` boshiga qo‘shiladigan kataloglar ro‘yxati (faqat gateway + sandbox).
- `tools.exec.safeBins`: explicit allowlist yozuvlarisiz ishlashi mumkin bo‘lgan, faqat stdin xavfsiz binary’lar.

Misol:

```json5
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH ishlov berilishi

- `host=gateway`: login-shell’dagi `PATH` ni exec muhiti bilan birlashtiradi. Hostda bajarishda `env.PATH` o‘zgartirishlari rad etiladi. Deymonning o‘zi esa minimal `PATH` bilan ishlaydi:
  - macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
  - Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
- `host=sandbox`: konteyner ichida `sh -lc` (login shell) ni ishga tushiradi, shuning uchun `/etc/profile` `PATH` ni qayta o‘rnatishi mumkin. OpenClaw `env.PATH` ni profil yuklangandan so‘ng ichki env o‘zgaruvchi orqali (shell interpolatsiyasiz) boshiga qo‘shadi; `tools.exec.pathPrepend` ham shu yerda qo‘llanadi.
- `host=node`: siz uzatgan, bloklanmagan env o‘zgartirishlargina node’ga yuboriladi. `env.PATH` o‘zgartirishlari hostda bajarish uchun rad etiladi va node hostlar tomonidan e’tiborsiz qoldiriladi. Agar node’da qo‘shimcha PATH yozuvlari kerak bo‘lsa, node host xizmati muhitini (systemd/launchd) sozlang yoki vositalarni standart joylarga o‘rnating.

Har bir agent uchun node bog‘lash (config’da agent ro‘yxati indeksidan foydalaning):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

Boshqaruv UI: Nodes yorlig‘ida shu sozlamalar uchun kichik “Exec node binding” paneli mavjud.

## Sessiya bo‘yicha o‘zgartirishlar (`/exec`)

`/exec` yordamida sessiya darajasida `host`, `security`, `ask` va `node` uchun standart qiymatlarni belgilang.  
Joriy qiymatlarni ko‘rish uchun argumentlarsiz `/exec` yuboring.

Misol:

```
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## Avtorizatsiya modeli

`/exec` faqat **ruxsat etilgan yuboruvchilar** uchun amal qiladi (kanal allowlist/pairing va `commands.useAccessGroups`).  
U faqat **sessiya holatini** yangilaydi va konfiguratsiyani yozmaydi. Exec’ni to‘liq o‘chirish uchun uni tool siyosati orqali rad eting (`tools.deny: ["exec"]` yoki agent bo‘yicha). Agar aniq `security=full` va `ask=off` o‘rnatilmagan bo‘lsa, host tasdiqlashlari baribir qo‘llanadi.

## Exec tasdiqlashlari (companion ilova / node host)

Sandbox qilingan agentlar gateway yoki node hostda `exec` bajarilishidan oldin har bir so‘rov uchun tasdiqlash talab qilishi mumkin.  
Siyosat, allowlist va UI jarayoni haqida [Exec approvals](/tools/exec-approvals) sahifasiga qarang.

Tasdiqlash talab qilinganda, exec vositasi darhol `status: "approval-pending"` va tasdiqlash identifikatori bilan qaytadi. Tasdiqlangach (yoki rad etilgach / vaqti tugagach), Gateway tizim hodisalarini yuboradi (`Exec finished` / `Exec denied`). Agar buyruq `tools.exec.approvalRunningNoticeMs` dan ko‘proq ishlasa, bitta `Exec running` bildirishnomasi yuboriladi.

## Allowlist + safe bins

Allowlist nazorati faqat **resolved binary path** bo‘yicha moslikni tekshiradi (basename bo‘yicha emas).  
`security=allowlist` bo‘lganda, shell buyruqlari faqat pipeline’dagi har bir segment allowlist’da yoki safe bin bo‘lsa avtomatik ruxsat etiladi.  
Zanjirlash (`;`, `&&`, `||`) va yo‘naltirishlar allowlist rejimida rad etiladi, agar har bir yuqori darajadagi segment allowlist (shu jumladan safe bins) talablariga javob bermasa.  
Yo‘naltirishlar hali ham qo‘llab-quvvatlanmaydi.

## Misollar

Foreground:

```json
{ "tool": "exec", "command": "ls -la" }
```

Background + poll:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

Tugmalar yuborish (tmux uslubida):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

Yuborish (faqat CR yuborish):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

Qo‘yish (standart bo‘yicha bracketed):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply_patch (eksperimental)

`apply_patch` — bu strukturalangan ko‘p faylli tahrirlar uchun `exec` ning subtool’i.  
Uni alohida yoqing:

```json5
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

Izohlar:

- Faqat OpenAI/OpenAI Codex modellari uchun mavjud.
- Tool siyosati baribir qo‘llanadi; `allow: ["exec"]` avtomatik ravishda `apply_patch` ga ham ruxsat beradi.
- Konfiguratsiya `tools.exec.applyPatch` ostida joylashgan.
- `tools.exec.applyPatch.workspaceOnly` standart bo‘yicha `true` (faqat workspace ichida). Agar `apply_patch` workspace katalogidan tashqariga yozishi/o‘chirishi kerak bo‘lsa, uni ataylab `false` ga o‘rnating.
