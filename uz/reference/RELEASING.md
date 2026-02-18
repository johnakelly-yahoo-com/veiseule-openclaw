---
title: "Reliz uchun tekshiruv roʻyxati"
---

# Reliz uchun tekshiruv roʻyxati (npm + macOS)

Repo ildizidan `pnpm` (Node 22+) dan foydalaning. Teg qoʻyish/nashr qilishdan oldin ishchi daraxt toza ekanini tekshiring.

## Operatorni ishga tushirish

Operator “release” deganda, darhol quyidagi preflight bosqichlarini bajaring (toʻsiq boʻlmasa, ortiqcha savollarsiz):

- Ushbu hujjatni va `docs/platforms/mac/release.md` ni oʻqing.
- `~/.profile` dan env o‘zgaruvchilarni yuklang va `SPARKLE_PRIVATE_KEY_FILE` + App Store Connect o‘zgaruvchilari o‘rnatilganini tasdiqlang (`SPARKLE_PRIVATE_KEY_FILE` `~/.profile` ichida bo‘lishi kerak).
- Zarur bo‘lsa, Sparkle kalitlarini `~/Library/CloudStorage/Dropbox/Backup/Sparkle` dan oling.

1. **Versiya va metamaʼlumotlar**

- [ ] `package.json` versiyasini oshiring (masalan, `2026.1.29`).
- [ ] Extension paket versiyalari + changeloglarni moslashtirish uchun `pnpm plugins:sync` ni ishga tushiring.
- [ ] CLI/versiya satrlarini yangilang: [`src/cli/program.ts`](https://github.com/openclaw/openclaw/blob/main/src/cli/program.ts) va Baileys user agent ni [`src/provider-web.ts`](https://github.com/openclaw/openclaw/blob/main/src/provider-web.ts) ichida.
- [ ] Paket metamaʼlumotlarini (name, description, repository, keywords, license) tasdiqlang va `bin` xaritasi `openclaw` uchun [`openclaw.mjs`](https://github.com/openclaw/openclaw/blob/main/openclaw.mjs) ga ishora qilayotganini tekshiring.
- [ ] Agar dependency’lar oʻzgargan boʻlsa, `pnpm-lock.yaml` yangiligi uchun `pnpm install` ni ishga tushiring.

2. **Build va artefaktlar**

- [ ] Agar A2UI kirishlari oʻzgargan boʻlsa, `pnpm canvas:a2ui:bundle` ni ishga tushiring va yangilangan [`src/canvas-host/a2ui/a2ui.bundle.js`](https://github.com/openclaw/openclaw/blob/main/src/canvas-host/a2ui/a2ui.bundle.js) faylini commit qiling.
- [ ] `pnpm run build` ( `dist/` ni qayta generatsiya qiladi).
- [ ] npm paketidagi `files` barcha zarur `dist/*` papkalarni (ayniqsa headless node + ACP CLI uchun `dist/node-host/**` va `dist/acp/**`) oʻz ichiga olganini tekshiring.
- [ ] `dist/build-info.json` mavjudligini va unda kutilgan `commit` xeshi borligini tasdiqlang (npm o‘rnatishlarida CLI banner shundan foydalanadi).
- [ ] Ixtiyoriy: build’dan so‘ng `npm pack --pack-destination /tmp`; tarball tarkibini tekshiring va uni GitHub relizi uchun saqlab qo‘ying (**commit qilmang**).

3. **Changelog va hujjatlar**

- [ ] `CHANGELOG.md` ni foydalanuvchiga ko‘rinadigan yangiliklar bilan yangilang (agar yo‘q bo‘lsa, fayl yarating); yozuvlar versiya bo‘yicha qatʼiy kamayish tartibida bo‘lsin.
- [ ] README misollari/flag’lari joriy CLI xatti-harakatiga (ayniqsa yangi buyruqlar yoki opsiyalar) mos ekanini tekshiring.

4. **Validatsiya**

- [ ] `pnpm build`
- [ ] `pnpm check`
- [ ] `pnpm test` (yoki coverage kerak bo‘lsa `pnpm test:coverage`)
- [ ] `pnpm release:check` (npm pack tarkibini tekshiradi)
- [ ] `OPENCLAW_INSTALL_SMOKE_SKIP_NONROOT=1 pnpm test:install:smoke` (Docker install smoke test, tezkor yo‘l; relizdan oldin majburiy)
  - Agar oldingi npm relizi buzilganligi maʼlum bo‘lsa, preinstall bosqichi uchun `OPENCLAW_INSTALL_SMOKE_PREVIOUS=<last-good-version>` yoki `OPENCLAW_INSTALL_SMOKE_SKIP_PREVIOUS=1` ni o‘rnating.
- [ ] (Ixtiyoriy) To‘liq installer smoke (non-root + CLI coverage qo‘shadi): `pnpm test:install:smoke`
- [ ] (Ixtiyoriy) Installer E2E (Docker, `curl -fsSL https://openclaw.ai/install.sh | bash` ni ishga tushiradi, onboarding qiladi, so‘ng haqiqiy tool call’larni bajaradi):
  - `pnpm test:install:e2e:openai` (`OPENAI_API_KEY` talab qilinadi)
  - `pnpm test:install:e2e:anthropic` (`ANTHROPIC_API_KEY` talab qilinadi)
  - `pnpm test:install:e2e` (ikkala kalit ham talab qilinadi; ikkala provider’ni ishga tushiradi)
- [ ] (Ixtiyoriy) O‘zgartirishlaringiz yuborish/qabul qilish yo‘llariga taʼsir qilgan bo‘lsa, web gateway’ni tezkor tekshirib chiqing.

5. **macOS ilovasi (Sparkle)**

- [ ] macOS ilovasini build qiling + imzolang, so‘ng tarqatish uchun zip qiling.
- [ ] Sparkle appcast’ni (HTML izohlar [`scripts/make_appcast.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/make_appcast.sh) orqali) generatsiya qiling va `appcast.xml` ni yangilang.
- [ ] Ilova zip faylini (va ixtiyoriy dSYM zip) GitHub reliziga biriktirish uchun tayyor saqlang.
- [ ] Aniq buyruqlar va zarur env o‘zgaruvchilar uchun [macOS release](/platforms/mac/release) ga amal qiling.
  - `APP_BUILD` raqamli va monoton bo‘lishi kerak (`-beta` siz), shunda Sparkle versiyalarni to‘g‘ri solishtiradi.
  - Agar notarization qilinsa, App Store Connect API env o‘zgaruvchilaridan yaratilgan `openclaw-notary` keychain profilidan foydalaning (qarang: [macOS release](/platforms/mac/release)).

6. **Nashr qilish (npm)**

- [ ] git status toza ekanini tasdiqlang; kerak bo‘lsa commit va push qiling.
- [ ] Zarur bo‘lsa, `npm login` (2FA ni tekshiring).
- [ ] `npm publish --access public` (pre-relizlar uchun `--tag beta` dan foydalaning).
- [ ] Registry’ni tekshiring: `npm view openclaw version`, `npm view openclaw dist-tags`, va `npx -y openclaw@X.Y.Z --version` (yoki `--help`).

### Nosozliklarni bartaraf etish (2.0.0-beta2 relizidan eslatmalar)

- **npm pack/publish osilib qoladi yoki juda katta tarball hosil qiladi**: `dist/OpenClaw.app` ichidagi macOS app bundle (va reliz zip’lari) paketga qo‘shilib ketadi. Buni `package.json` dagi `files` orqali publish tarkibini whitelist qilish bilan tuzating (dist subdir’lar, docs, skills’ni qo‘shing; app bundle’larni chiqarib tashlang). `npm pack --dry-run` bilan `dist/OpenClaw.app` ro‘yxatda yo‘qligini tasdiqlang.
- **dist-tags uchun npm auth web loop**: OTP so‘rovini olish uchun legacy auth’dan foydalaning:
  - `NPM_CONFIG_AUTH_TYPE=legacy npm dist-tag add openclaw@X.Y.Z latest`
- **`npx` tekshiruvi `ECOMPROMISED: Lock compromised` bilan muvaffaqiyatsiz tugaydi**: yangi cache bilan qayta urinib ko‘ring:
  - `NPM_CONFIG_CACHE=/tmp/npm-cache-$(date +%s) npx -y openclaw@X.Y.Z --version`
- **Kechikkan tuzatishdan keyin tag’ni qayta yo‘naltirish kerak**: tag’ni majburan yangilang va push qiling, so‘ng GitHub relizi artefaktlari hanuz mosligini tekshiring:
  - `git tag -f vX.Y.Z && git push -f origin vX.Y.Z`

7. **GitHub relizi + appcast**

- [ ] Teg qo‘ying va push qiling: `git tag vX.Y.Z && git push origin vX.Y.Z` (yoki `git push --tags`).
- [ ] `vX.Y.Z` uchun GitHub relizini yarating/yangilang, **sarlavha `openclaw X.Y.Z`** bo‘lsin (faqat tag emas); body qismida shu versiya uchun **to‘liq** changelog bo‘limi (Highlights + Changes + Fixes) inline ko‘rinishda bo‘lsin (yalang‘och havolalarsiz) va **body ichida sarlavha takrorlanmasin**.
- [ ] Artefaktlarni biriktiring: `npm pack` tarball (ixtiyoriy), `OpenClaw-X.Y.Z.zip`, va `OpenClaw-X.Y.Z.dSYM.zip` (agar generatsiya qilingan bo‘lsa).
- [ ] Yangilangan `appcast.xml` ni commit qiling va push qiling (Sparkle main’dan feed oladi).
- [ ] Toza vaqtinchalik papkadan (`package.json` siz), `npx -y openclaw@X.Y.Z send --help` ni ishga tushirib, install/CLI entrypoint’lar ishlashini tasdiqlang.
- [ ] Reliz eslatmalarini eʼlon qiling/ulashing.

## Plugin nashr qilish qamrovi (npm)

Biz faqat `@openclaw/*` scope ostidagi **mavjud npm plugin’larni** nashr qilamiz. npm’da mavjud bo‘lmagan, lekin bundle qilingan plugin’lar **faqat disk-tree ko‘rinishida** qoladi (`extensions/**` ichida tarqatiladi).

Ro‘yxatni aniqlash jarayoni:

1. `npm search @openclaw --json` ni ishga tushiring va paket nomlarini oling.
2. `extensions/*/package.json` ichidagi nomlar bilan solishtiring.
3. Faqat **kesishma** (npm’da allaqachon mavjud bo‘lganlar) ni nashr qiling.

Joriy npm plugin ro‘yxati (zaruratga ko‘ra yangilang):

- @openclaw/bluebubbles
- @openclaw/diagnostics-otel
- @openclaw/discord
- @openclaw/feishu
- @openclaw/lobster
- @openclaw/matrix
- @openclaw/msteams
- @openclaw/nextcloud-talk
- @openclaw/nostr
- @openclaw/voice-call
- @openclaw/zalo
- @openclaw/zalouser

Reliz eslatmalarida, shuningdek, **sukut bo‘yicha yoqilmagan yangi ixtiyoriy bundle qilingan plugin’lar** ham alohida qayd etilishi kerak (masalan: `tlon`).


