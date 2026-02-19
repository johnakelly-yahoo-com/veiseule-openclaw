---
summary: "Reja: CDP yordamida browser act:evaluate’ni Playwright navbatidan ajratish, end-to-end deadline’lar va xavfsizroq ref resolution bilan"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refaktor"
---

# Browser Evaluate CDP Refaktor Rejasi

## Kontekst

`act:evaluate` foydalanuvchi tomonidan berilgan JavaScript kodini sahifada bajaradi. Hozirda u Playwright orqali ishlaydi
(`page.evaluate` yoki `locator.evaluate`). Playwright har bir sahifa uchun CDP buyruqlarini ketma-ket bajaradi, shuning uchun
osilib qolgan yoki uzoq ishlaydigan evaluate sahifa buyruqlari navbatini bloklab qo‘yishi va shu tabdagi keyingi barcha amallarni "osilib qolgandek" ko‘rsatishi mumkin.

PR #13498 amaliy xavfsizlik mexanizmini qo‘shadi (chegaralangan evaluate, abort uzatilishi va imkon qadar
tiklash). Ushbu hujjat `act:evaluate`ni Playwright’dan tabiatan
ajratilgan qiladigan kattaroq refaktorni tasvirlaydi, shunda osilib qolgan evaluate oddiy Playwright operatsiyalarini to‘sib qo‘ya olmaydi.

## Maqsadlar

- `act:evaluate` bir xil tabdagi keyingi brauzer amallarini doimiy ravishda bloklay olmaydi.
- Timeout’lar boshidan oxirigacha yagona haqiqat manbai bo‘ladi, shunda chaqiruvchi belgilangan vaqt budjetiga ishonishi mumkin.
- Abort va timeout HTTP hamda jarayon ichidagi dispatch’da bir xil tarzda ko‘rib chiqiladi.
- Evaluate uchun elementni nishonga olish Playwright’dan to‘liq voz kechmasdan qo‘llab-quvvatlanadi.
- Mavjud chaqiruvchilar va payload’lar uchun orqaga moslik saqlanadi.

## Maqsadga kirmaydiganlar

- Barcha brauzer amallarini (click, type, wait va boshqalar) almashtirish ularni CDP implementatsiyalari bilan.
- PR #13498’da kiritilgan mavjud xavfsizlik mexanizmini olib tashlash (u foydali zaxira sifatida qoladi).
- Mavjud `browser.evaluateEnabled` cheklovidan tashqari yangi xavfli imkoniyatlarni joriy etish.
- Evaluate uchun jarayon izolyatsiyasini (worker process/thread) qo‘shish. Agar ushbu refaktordan keyin ham tiklash qiyin bo‘lgan
  osilib qolish holatlarini ko‘rsak, bu keyingi bosqich g‘oyasi bo‘ladi.

## Joriy Arxitektura (Nima uchun u osilib qoladi)

Yuqori darajada:

- Chaqiruvchilar brauzerni boshqarish xizmatiga `act:evaluate` yuboradi.
- Route handler JavaScript’ni bajarish uchun Playwright’ni chaqiradi.
- Playwright sahifa buyruqlarini ketma-ket bajaradi, shuning uchun hech qachon tugamaydigan evaluate navbatni bloklaydi.
- Navbatning osilib qolishi tabdagi keyingi click/type/wait amallarini ham osilib qolgandek ko‘rsatishi mumkin.

## Taklif etilayotgan Arxitektura

### 1. Deadline Uzatilishi

Yagona budjet tushunchasini joriy etish va hamma narsani undan kelib chiqib hisoblash:

- Chaqiruvchi `timeoutMs`ni (yoki kelajakdagi deadline’ni) belgilaydi.
- Tashqi so‘rov timeout’i, route handler mantiqi va sahifa ichidagi bajarilish budjeti
  barchasi bir xil budjetdan foydalanadi, zarur joylarda serializatsiya xarajatlari uchun kichik zaxira bilan.
- Abort hamma joyda `AbortSignal` sifatida uzatiladi, shunda bekor qilish bir xil ishlaydi.

Amalga oshirish yo‘nalishi:

- Kichik yordamchi funksiya qo‘shish (masalan, `createBudget({ timeoutMs, signal })`), u quyidagilarni qaytaradi:
  - `signal`: bog‘langan AbortSignal
  - `deadlineAtMs`: mutlaq deadline
  - `remainingMs()`: ichki operatsiyalar uchun qolgan budjet
- Ushbu yordamchini quyidagilarda ishlatish:
  - `src/browser/client-fetch.ts` (HTTP va jarayon ichidagi dispatch)
  - `src/node-host/runner.ts` (proxy yo‘li)
  - brauzer action implementatsiyalari (Playwright va CDP)

### 2. Alohida Evaluate Engine (CDP yo‘li)

Playwright’ning har bir sahifa uchun buyruqlar navbatini bo‘lishmaydigan CDP asosidagi evaluate implementatsiyasini qo‘shing. Asosiy xususiyat shundaki, evaluate transporti alohida WebSocket ulanishi
va target’ga ulangan alohida CDP sessiyasidan iborat.

Implementatsiya yo‘nalishi:

- Masalan, `src/browser/cdp-evaluate.ts` kabi yangi modul, u:
  - Sozlangan CDP endpoint’ga ulanadi (brauzer darajasidagi socket).
  - `Target.attachToTarget({ targetId, flatten: true })` dan foydalanib `sessionId` oladi.
  - Quyidagilardan birini bajaradi:
    - Sahifa darajasidagi evaluate uchun `Runtime.evaluate`, yoki
    - Element evaluate uchun `DOM.resolveNode` hamda `Runtime.callFunctionOn`.
  - Timeout yoki bekor qilish holatida:
    - Sessiya uchun imkon qadar `Runtime.terminateExecution` yuboradi.
    - WebSocket’ni yopadi va aniq xatolikni qaytaradi.

Izohlar:

- Bu baribir sahifada JavaScript bajaradi, shuning uchun to‘xtatish yon ta’sirlarga ega bo‘lishi mumkin. Yutuq shundaki
  bu Playwright navbatini tiqib qo‘ymaydi va CDP sessiyasini yopish orqali transport
  darajasida bekor qilish mumkin.

### 3. Ref Story (To‘liq qayta yozmasdan elementni nishonga olish)

Eng qiyin qismi — elementni nishonga olish. CDP’ga DOM handle yoki `backendDOMNodeId` kerak, hozirda esa
ko‘pchilik brauzer action’lari snapshot’lardagi ref’lar asosidagi Playwright locator’lardan foydalanadi.

Tavsiya etilgan yondashuv: mavjud ref’larni saqlab qolish, lekin ixtiyoriy CDP yechiladigan id biriktirish.

#### 3.1 Saqlangan Ref Ma’lumotini Kengaytirish

Saqlangan role ref metama’lumotini ixtiyoriy CDP id ni o‘z ichiga oladigan qilib kengaytiring:

- Hozir: `{ role, name, nth }`
- Taklif: `{ role, name, nth, backendDOMNodeId?: number }`

Bu barcha mavjud Playwright asosidagi action’larning ishlashini saqlab qoladi va `backendDOMNodeId` mavjud bo‘lganda CDP evaluate’ga
xuddi shu `ref` qiymatini qabul qilish imkonini beradi.

#### 3.2 Snapshot Vaqtida backendDOMNodeId ni To‘ldirish

Role snapshot yaratilganda:

1. Mavjud role ref xaritasini hozirgidek yarating (role, name, nth).
2. CDP orqali AX tree’ni oling (`Accessibility.getFullAXTree`) va
   xuddi shu dublikatlarni boshqarish qoidalaridan foydalangan holda
   `(role, name, nth) -> backendDOMNodeId` parallel xaritasini hisoblang.
3. Id ni joriy tab uchun saqlangan ref ma’lumotiga qayta birlashtiring.

Agar ref uchun moslashtirish muvaffaqiyatsiz bo‘lsa, `backendDOMNodeId` ni undefined qoldiring. Bu funksiyani
imkon qadar (best-effort) qiladi va xavfsiz tarzda joriy etish imkonini beradi.

#### 3.3 Ref bilan Evaluate Xatti-harakati

`act:evaluate` ichida:

- Agar `ref` mavjud bo‘lsa va unda `backendDOMNodeId` bo‘lsa, element evaluate’ni CDP orqali bajaring.
- Agar `ref` mavjud bo‘lsa, lekin `backendDOMNodeId` bo‘lmasa, Playwright yo‘liga qayting (xavfsizlik mexanizmi bilan
  birga).

Ixtiyoriy qo‘shimcha imkoniyat:

- So‘rov shaklini kengaytirib, ilg‘or foydalanuvchilar (va
  diagnostika uchun) `backendDOMNodeId`ni bevosita qabul qilishini ta’minlang, shu bilan birga `ref`ni asosiy interfeys sifatida saqlang.

### 4. Oxirgi Chora Sifatida Tiklash Yo‘lini Saqlab Qoling

Hatto CDP evaluate bilan ham, tab yoki ulanishni ishdan chiqarishning boshqa usullari mavjud. Quyidagi holatlar uchun
mavjud tiklash mexanizmlarini (bajarilishni to‘xtatish + Playwright bilan aloqani uzish) oxirgi chora sifatida saqlab qoling:

- eski chaqiruvchilar
- CDP attach bloklangan muhitlar
- kutilmagan Playwright chekka holatlari

## Amalga Oshirish Rejasi (Yagona Iteratsiya)

### Yetkazib Beriladigan Natijalar

- Playwright’ning har bir sahifa buyruqlar navbatidan tashqarida ishlaydigan CDP asosidagi evaluate mexanizmi.
- Chaqiruvchilar va handlerlar tomonidan izchil qo‘llaniladigan yagona end-to-end timeout/abort byudjeti.
- Element evaluate uchun ixtiyoriy ravishda `backendDOMNodeId`ni o‘z ichiga olishi mumkin bo‘lgan ref metama’lumoti.
- `act:evaluate` imkon qadar CDP mexanizmini afzal ko‘radi va imkoni bo‘lmaganda Playwright’ga qaytadi.
- Qotib qolgan evaluate keyingi harakatlarni bloklamasligini isbotlaydigan testlar.
- Xatolar va fallback holatlarini ko‘rinadigan qiladigan loglar/metriclar.

### Amalga Oshirish Cheklisti

1. `timeoutMs` + yuqori darajadagi `AbortSignal`ni quyidagilarga bog‘lash uchun umumiy "budget" yordamchisini qo‘shing:
   - yagona `AbortSignal`
   - mutlaq deadline
   - quyi operatsiyalar uchun `remainingMs()` yordamchisi
2. Barcha chaqiruv yo‘llarini ushbu yordamchidan foydalanadigan qilib yangilang, shunda `timeoutMs` hamma joyda bir xil ma’noni anglatadi:
   - `src/browser/client-fetch.ts` (HTTP va in-process dispatch)
   - `src/node-host/runner.ts` (node proxy yo‘li)
   - `/act`ni chaqiradigan CLI wrapperlar (`browser evaluate`ga `--timeout-ms` qo‘shing)
3. `src/browser/cdp-evaluate.ts`ni amalga oshiring:
   - brauzer darajasidagi CDP socket’iga ulaning
   - `sessionId` olish uchun `Target.attachToTarget`
   - sahifa evaluate uchun `Runtime.evaluate`ni ishga tushiring
   - element evaluate uchun `DOM.resolveNode` + `Runtime.callFunctionOn`ni ishga tushiring
   - timeout/abort holatida: imkon qadar `Runtime.terminateExecution`, so‘ng socket’ni yoping
4. Saqlangan role ref metama’lumotini ixtiyoriy ravishda `backendDOMNodeId`ni o‘z ichiga oladigan qilib kengaytiring:
   - Playwright harakatlari uchun mavjud `{ role, name, nth }` xatti-harakatini saqlang
   - CDP orqali elementni nishonga olish uchun `backendDOMNodeId?: number` qo‘shing
5. Snapshot yaratish vaqtida `backendDOMNodeId`ni to‘ldiring (imkon qadar):
   - CDP orqali AX daraxtini oling (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId`ni hisoblab, saqlangan ref xaritasiga birlashtiring
   - agar moslik noaniq yoki mavjud bo‘lmasa, id qiymatini aniqlanmagan holda qoldiring
6. `act:evaluate` marshrutlashini yangilash:
   - agar `ref` bo‘lmasa: har doim CDP evaluate’dan foydalaning
   - agar `ref` `backendDOMNodeId` ga mos kelsa: CDP element evaluate’dan foydalaning
   - aks holda: Playwright evaluate’ga qayting (hanuz cheklangan va bekor qilinadigan)
7. Mavjud "oxirgi chora" tiklash yo‘lini asosiy yo‘l emas, balki zaxira sifatida saqlang.
8. Testlar qo‘shing:
   - osilib qolgan evaluate ajratilgan vaqt ichida timeout bo‘ladi va keyingi click/type muvaffaqiyatli bajariladi
   - abort evaluate’ni bekor qiladi (mijoz uzilishi yoki timeout) va keyingi harakatlarni blokdan chiqaradi
   - mapping xatolari toza tarzda Playwright’ga qaytadi
9. Kuzatuv imkoniyatlarini qo‘shing:
   - evaluate davomiyligi va timeout hisoblagichlari
   - terminateExecution’dan foydalanish
   - fallback darajasi (CDP -> Playwright) va sabablari

### Qabul Qilish Mezonlari

- Ataylab osiltirilgan `act:evaluate` chaqiruvchi ajratgan vaqt ichida qaytadi va keyingi harakatlar uchun tab’ni bloklab qo‘ymaydi.
- `timeoutMs` CLI, agent tool, node proxy va in-process chaqiruvlarda bir xil ishlaydi.
- Agar `ref` ni `backendDOMNodeId` ga moslashtirish mumkin bo‘lsa, element evaluate CDP orqali bajariladi; aks holda fallback yo‘li hanuz cheklangan va tiklanadigan bo‘ladi.

## Sinov Rejasi

- Unit testlar:
  - `(role, name, nth)` moslashtirish mantiqi role reference’lar va AX tree tugunlari o‘rtasida.
  - Budget helper xatti-harakati (zaxira vaqt, qolgan vaqt hisob-kitobi).
- Integratsion testlar:
  - CDP evaluate timeout ajratilgan vaqt ichida qaytadi va keyingi harakatni bloklamaydi.
  - Abort evaluate’ni bekor qiladi va imkon qadar termination’ni ishga tushiradi.
- Kontrakt testlar:
  - `BrowserActRequest` va `BrowserActResponse` mosligi saqlanib qolishini ta’minlang.

## Xatarlar va Ularni Kamaytirish

- Mapping mukammal emas:
  - Kamaytirish: imkon qadar moslashtirish, Playwright evaluate’ga fallback qilish va debug vositalarini qo‘shish.
- `Runtime.terminateExecution` nojo‘ya ta’sirlarga ega:
  - Kamaytirish: faqat timeout/abort holatlarida foydalanish va xatolarda ushbu xatti-harakatni hujjatlashtirish.
- Qo‘shimcha yuklama:
  - Kamaytirish: AX tree’ni faqat snapshot so‘ralganda olish, har bir target uchun keshlash va CDP session’ni qisqa muddatli saqlash.
- Extension relay cheklovlari:
  - Kamaytirish: har bir sahifa uchun socket mavjud bo‘lmaganda brauzer darajasidagi attach API’lardan foydalanish va joriy Playwright yo‘lini fallback sifatida saqlash.

## Ochiq Savollar

- Yangi engine `playwright`, `cdp` yoki `auto` sifatida sozlanadigan bo‘lishi kerakmi?
- Kengaytirilgan foydalanuvchilar uchun yangi "nodeRef" formatini ochishimiz kerakmi yoki faqat `ref` ni saqlab qolamizmi?
- Frame snapshot’lar va selector doirasidagi snapshot’lar AX mapping’da qanday ishtirok etishi kerak?
