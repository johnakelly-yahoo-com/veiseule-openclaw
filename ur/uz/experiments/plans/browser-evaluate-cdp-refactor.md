---
summary: "Plan: CDP کا استعمال کرتے ہوئے browser act:evaluate کو Playwright queue سے الگ کرنا، end-to-end deadlines اور زیادہ محفوظ ref resolution کے ساتھ"
owner: "openclaw"
status: "draft"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP ریفیکٹر"
---

# Browser Evaluate CDP ریفیکٹر پلان

## سیاق و سباق

`act:evaluate` صفحے میں صارف کی فراہم کردہ JavaScript کو اجرا کرتا ہے۔ فی الحال یہ Playwright کے ذریعے چلتا ہے
(`page.evaluate` یا `locator.evaluate`)۔ Playwright ہر صفحے کے لیے CDP کمانڈز کو سلسلہ وار چلاتا ہے، اس لیے اگر evaluate اٹک جائے یا زیادہ دیر تک چلے تو یہ صفحے کی کمانڈ قطار کو بلاک کر سکتا ہے اور اس ٹیب پر بعد کی ہر کارروائی کو "stuck" ظاہر کر سکتا ہے۔

PR #13498 ایک عملی حفاظتی انتظام شامل کرتا ہے (bounded evaluate، abort propagation، اور best-effort بحالی)۔ یہ دستاویز ایک بڑے ریفیکٹر کی وضاحت کرتی ہے جو `act:evaluate` کو فطری طور پر Playwright سے الگ تھلگ بناتا ہے تاکہ اگر evaluate اٹک جائے تو وہ معمول کی Playwright کارروائیوں کو متاثر نہ کرے۔

## اہداف

- `act:evaluate` اسی ٹیب پر بعد کی براؤزر کارروائیوں کو مستقل طور پر بلاک نہ کر سکے۔
- timeouts ابتدا سے انتہا تک واحد ماخذِ حقیقت ہوں تاکہ کالر اپنے مقررہ وقت کے بجٹ پر بھروسا کر سکے۔
- Abort اور timeout کو HTTP اور in-process dispatch دونوں میں یکساں طور پر ہینڈل کیا جائے۔
- evaluate کے لیے element کو ہدف بنانا Playwright کو مکمل طور پر ہٹائے بغیر ممکن ہو۔
- موجودہ کالرز اور payloads کے ساتھ پسماندہ مطابقت برقرار رکھی جائے۔

## غیر مقاصد

- تمام براؤزر کارروائیوں (click, type, wait، وغیرہ) کو تبدیل کرنا CDP implementations کے ساتھ۔
- PR #13498 میں متعارف کرایا گیا موجودہ حفاظتی انتظام ختم کرنا (یہ ایک مفید fallback کے طور پر برقرار رہے گا)۔
- موجودہ `browser.evaluateEnabled` گیٹ سے آگے نئی غیر محفوظ صلاحیتیں متعارف کرانا۔
- evaluate کے لیے process isolation (worker process/thread) شامل کرنا۔ اگر اس ریفیکٹر کے بعد بھی
  ایسے stuck حالات نظر آئیں جن سے بحالی مشکل ہو، تو یہ ایک اگلا خیال ہوگا۔

## موجودہ آرکیٹیکچر (یہ کیوں اٹکتا ہے)

اعلیٰ سطح پر:

- کالرز `act:evaluate` کو براؤزر کنٹرول سروس کو بھیجتے ہیں۔
- روٹ ہینڈلر JavaScript کو اجرا کرنے کے لیے Playwright کو کال کرتا ہے۔
- Playwright صفحے کی کمانڈز کو سلسلہ وار چلاتا ہے، اس لیے ایسا evaluate جو کبھی مکمل نہ ہو قطار کو بلاک کر دیتا ہے۔
- اٹکی ہوئی قطار کا مطلب ہے کہ ٹیب پر بعد کی click/type/wait کارروائیاں لٹکی ہوئی محسوس ہو سکتی ہیں۔

## مجوزہ آرکیٹیکچر

### 1. Deadline Propagation

ایک واحد بجٹ کا تصور متعارف کرائیں اور ہر چیز اسی سے اخذ کریں:

- کالر `timeoutMs` مقرر کرتا ہے (یا مستقبل میں ایک deadline)۔
- بیرونی درخواست کا timeout، روٹ ہینڈلر کی منطق، اور صفحے کے اندر execution بجٹ
  سب ایک ہی بجٹ استعمال کرتے ہیں، جہاں ضرورت ہو وہاں serialization overhead کے لیے معمولی گنجائش کے ساتھ۔
- Abort کو ہر جگہ `AbortSignal` کے طور پر propagate کیا جاتا ہے تاکہ منسوخی یکساں رہے۔

نفاذ کی سمت:

- ایک چھوٹا helper شامل کریں (مثال کے طور پر `createBudget({ timeoutMs, signal })`) جو واپس کرے:
  - `signal`: منسلک AbortSignal
  - `deadlineAtMs`: مطلق آخری وقت
  - `remainingMs()`: ذیلی آپریشنز کے لیے باقی بجٹ
- اس ہیلپر کو یہاں استعمال کریں:
  - `src/browser/client-fetch.ts` (HTTP اور ان-پروسیس ڈسپیچ)
  - `src/node-host/runner.ts` (پراکسی راستہ)
  - براؤزر ایکشن امپلیمینٹیشنز (Playwright اور CDP)

### 2. الگ Evaluate انجن (CDP راستہ)

ایک CDP پر مبنی evaluate امپلیمینٹیشن شامل کریں جو Playwright کی فی پیج کمانڈ قطار کو شیئر نہ کرے۔ اہم خصوصیت یہ ہے کہ evaluate ٹرانسپورٹ ایک علیحدہ WebSocket کنکشن ہو
اور ہدف کے ساتھ منسلک ایک علیحدہ CDP سیشن ہو۔

امپلیمینٹیشن کی سمت:

- نیا ماڈیول، مثال کے طور پر `src/browser/cdp-evaluate.ts`، جو:
  - کنفیگر شدہ CDP اینڈ پوائنٹ سے کنیکٹ ہو (براؤزر لیول ساکٹ)۔
  - `Target.attachToTarget({ targetId, flatten: true })` استعمال کر کے `sessionId` حاصل کرے۔
  - چلائے:
    - پیج لیول evaluate کے لیے `Runtime.evaluate`، یا
    - ایلیمنٹ evaluate کے لیے `DOM.resolveNode` کے ساتھ `Runtime.callFunctionOn`۔
  - ٹائم آؤٹ یا ابورٹ کی صورت میں:
    - سیشن کے لیے بہترین کوشش کے طور پر `Runtime.terminateExecution` بھیجے۔
    - WebSocket بند کرے اور واضح خرابی واپس کرے۔

نوٹس:

- یہ اب بھی پیج میں JavaScript چلاتا ہے، اس لیے ٹرمینیشن کے سائیڈ ایفیکٹس ہو سکتے ہیں۔ فائدہ
  یہ ہے کہ یہ Playwright قطار کو بلاک نہیں کرتا، اور CDP سیشن کو ختم کر کے ٹرانسپورٹ لیئر پر اسے منسوخ کیا جا سکتا ہے۔

### 3. Ref اسٹوری (بغیر مکمل ری رائٹ کے ایلیمنٹ ٹارگٹنگ)

مشکل حصہ ایلیمنٹ ٹارگٹنگ ہے۔ CDP کو DOM ہینڈل یا `backendDOMNodeId` درکار ہوتا ہے، جبکہ
آج زیادہ تر براؤزر ایکشنز اسنیپ شاٹس سے حاصل کردہ refs پر مبنی Playwright لوکیٹرز استعمال کرتے ہیں۔

تجویز کردہ طریقہ: موجودہ refs برقرار رکھیں، لیکن ایک اختیاری CDP قابلِ حل id منسلک کریں۔

#### 3.1 محفوظ شدہ Ref معلومات میں توسیع

محفوظ شدہ رول ref میٹا ڈیٹا کو بڑھا کر اختیاری طور پر CDP id شامل کریں:

- آج: `{ role, name, nth }`
- مجوزہ: `{ role, name, nth, backendDOMNodeId?: number }`

اس سے تمام موجودہ Playwright پر مبنی ایکشنز کام کرتے رہیں گے اور CDP evaluate کو اجازت ملے گی کہ
جب `backendDOMNodeId` دستیاب ہو تو وہی `ref` ویلیو قبول کرے۔

#### 3.2 اسنیپ شاٹ کے وقت backendDOMNodeId پُر کریں

جب رول اسنیپ شاٹ تیار کیا جائے:

1. موجودہ رول ref میپ آج کی طرح تیار کریں (role, name, nth)۔
2. CDP کے ذریعے AX ٹری حاصل کریں (`Accessibility.getFullAXTree`) اور
   اسی ڈپلیکیٹ ہینڈلنگ قواعد استعمال کرتے ہوئے `(role, name, nth) -> backendDOMNodeId` کا متوازی میپ تیار کریں۔
3. id کو موجودہ ٹیب کے لیے محفوظ شدہ ref معلومات میں واپس ضم کریں۔

اگر کسی ref کے لیے میپنگ ناکام ہو جائے تو `backendDOMNodeId` کو undefined ہی رہنے دیں۔ یہ فیچر کو بہترین کوشش کی بنیاد پر بناتا ہے
اور اسے مرحلہ وار جاری کرنا محفوظ بناتا ہے۔

#### 3.3 Ref کے ساتھ رویے کا جائزہ لیں

`act:evaluate` میں:

- اگر `ref` موجود ہو اور اس میں `backendDOMNodeId` ہو تو CDP کے ذریعے element evaluate چلائیں۔
- اگر `ref` موجود ہو لیکن اس میں `backendDOMNodeId` نہ ہو تو Playwright کے راستے پر واپس جائیں (سیکیورٹی نیٹ کے ساتھ)۔

اختیاری ایمرجنسی راستہ:

- درخواست کی ساخت کو بڑھا کر براہِ راست `backendDOMNodeId` قبول کرنے کے قابل بنائیں جدید صارفین (اور ڈیبگنگ) کے لیے، جبکہ `ref` کو بنیادی انٹرفیس کے طور پر برقرار رکھیں۔

### 4. آخری حل کے طور پر ریکوری کا راستہ برقرار رکھیں

CDP evaluate کے باوجود، ٹیب یا کنکشن کے پھنسنے کے دیگر طریقے بھی ہو سکتے ہیں۔ آخری حل کے طور پر موجودہ ریکوری میکانزم (execution ختم کرنا + Playwright کو disconnect کرنا) برقرار رکھیں ان کے لیے:

- legacy کالرز
- ایسے ماحول جہاں CDP attach بلاک ہو
- Playwright کے غیر متوقع edge cases

## نفاذ کا منصوبہ (سنگل iteration)

### فراہمی کی اشیاء

- ایک CDP پر مبنی evaluate انجن جو Playwright کے فی-پیج کمانڈ کیو سے باہر چلتا ہو۔
- ایک واحد end-to-end timeout/abort بجٹ جو کالرز اور ہینڈلرز کے ذریعے مستقل طور پر استعمال ہو۔
- Ref میٹاڈیٹا جو اختیاری طور پر element evaluate کے لیے `backendDOMNodeId` رکھ سکے۔
- `act:evaluate` جہاں ممکن ہو CDP انجن کو ترجیح دیتا ہے اور بصورتِ دیگر Playwright پر واپس آتا ہے۔
- ٹیسٹس جو ثابت کریں کہ پھنسا ہوا evaluate بعد کی کارروائیوں کو بلاک نہیں کرتا۔
- لاگز/میٹرکس جو ناکامیوں اور fallback کو واضح بنائیں۔

### نفاذ کی چیک لسٹ

1. `timeoutMs` اور upstream `AbortSignal` کو جوڑنے کے لیے ایک مشترکہ "budget" ہیلپر شامل کریں:
   - ایک واحد `AbortSignal`
   - ایک مطلق deadline
   - ڈاؤن اسٹریم آپریشنز کے لیے `remainingMs()` ہیلپر
2. تمام کالر راستوں کو اپڈیٹ کریں تاکہ وہ اس ہیلپر کو استعمال کریں اور `timeoutMs` ہر جگہ ایک ہی معنی رکھے:
   - `src/browser/client-fetch.ts` (HTTP اور in-process dispatch)
   - `src/node-host/runner.ts` (node proxy راستہ)
   - CLI wrappers جو `/act` کو کال کرتے ہیں (`browser evaluate` میں `--timeout-ms` شامل کریں)
3. `src/browser/cdp-evaluate.ts` نافذ کریں:
   - browser-لیول CDP socket سے کنیکٹ کریں
   - `Target.attachToTarget` استعمال کر کے `sessionId` حاصل کریں
   - page evaluate کے لیے `Runtime.evaluate` چلائیں
   - element evaluate کے لیے `DOM.resolveNode` + `Runtime.callFunctionOn` چلائیں
   - timeout/abort پر: بہترین کوشش کے طور پر `Runtime.terminateExecution` پھر socket بند کریں
4. ذخیرہ شدہ role ref میٹاڈیٹا کو بڑھا کر اختیاری طور پر `backendDOMNodeId` شامل کریں:
   - Playwright ایکشنز کے لیے موجودہ `{ role, name, nth }` رویہ برقرار رکھیں
   - CDP element targeting کے لیے `backendDOMNodeId?: number` شامل کریں
5. snapshot تخلیق کے دوران `backendDOMNodeId` کو پُر کریں (بہترین کوشش):
   - CDP کے ذریعے AX درخت حاصل کریں (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` کا حساب لگائیں اور اسے محفوظ شدہ ref نقشے میں ضم کریں
   - اگر میپنگ مبہم ہو یا موجود نہ ہو تو id کو غیر متعین چھوڑ دیں
6. `act:evaluate` روٹنگ کو اپ ڈیٹ کریں:
   - اگر `ref` موجود نہ ہو: ہمیشہ CDP evaluate استعمال کریں
   - اگر `ref` کسی `backendDOMNodeId` میں حل ہو جائے: CDP element evaluate استعمال کریں
   - بصورتِ دیگر: Playwright evaluate پر واپس جائیں (جو اب بھی محدود اور منسوخ کے قابل ہو)
7. موجودہ "آخری حل" ریکوری راستے کو بطور fallback رکھیں، نہ کہ بطور ڈیفالٹ راستہ۔
8. ٹیسٹس شامل کریں:
   - اٹکا ہوا evaluate مقررہ وقت کے اندر ٹائم آؤٹ ہو اور اگلا click/type کامیاب ہو
   - abort evaluate کو منسوخ کرے (کلائنٹ ڈسکنیکٹ یا ٹائم آؤٹ) اور بعد کی کارروائیوں کو ان بلاک کرے
   - میپنگ کی ناکامیاں صاف طور پر Playwright پر fallback کریں
9. آبزرویبلٹی شامل کریں:
   - evaluate کے دورانیے اور ٹائم آؤٹ کاؤنٹرز
   - terminateExecution کا استعمال
   - fallback کی شرح (CDP -> Playwright) اور اسباب

### قبولیت کے معیار

- جان بوجھ کر ہینگ کیا گیا `act:evaluate` کالر کے مقررہ بجٹ کے اندر واپس آ جائے اور
  بعد کی کارروائیوں کے لیے tab کو جام نہ کرے۔
- `timeoutMs` CLI، agent tool، node proxy، اور in-process کالز میں یکساں برتاؤ کرے۔
- اگر `ref` کو `backendDOMNodeId` سے میپ کیا جا سکے تو element evaluate CDP استعمال کرے؛ بصورتِ دیگر
  fallback راستہ پھر بھی محدود اور قابلِ بحالی ہو۔

## ٹیسٹنگ پلان

- یونٹ ٹیسٹس:
  - `(role, name, nth)` کی میچنگ لاجک role refs اور AX tree نوڈز کے درمیان۔
  - بجٹ ہیلپر کا رویہ (headroom، باقی وقت کا حساب).
- انٹیگریشن ٹیسٹس:
  - CDP evaluate ٹائم آؤٹ مقررہ بجٹ کے اندر واپس آئے اور اگلی کارروائی کو بلاک نہ کرے۔
  - Abort evaluate کو منسوخ کرے اور بہترین کوشش کے طور پر termination کو متحرک کرے۔
- کنٹریکٹ ٹیسٹس:
  - یقینی بنائیں کہ `BrowserActRequest` اور `BrowserActResponse` ہم آہنگ رہیں۔

## خطرات اور تدارک

- میپنگ مکمل طور پر درست نہیں:
  - تدارک: بہترین کوشش پر مبنی میپنگ، Playwright evaluate پر fallback، اور debug ٹولنگ شامل کریں۔
- `Runtime.terminateExecution` کے سائیڈ ایفیکٹس ہیں:
  - تدارک: اسے صرف ٹائم آؤٹ/abort پر استعمال کریں اور اس رویے کو errors میں دستاویز کریں۔
- اضافی اوورہیڈ:
  - تدارک: AX درخت صرف اس وقت حاصل کریں جب snapshots درکار ہوں، فی ہدف کیش کریں، اور
    CDP سیشن کو مختصر المدت رکھیں۔
- Extension relay کی حدود:
  - تدارک: جب فی صفحہ ساکٹس دستیاب نہ ہوں تو browser لیول attach APIs استعمال کریں، اور
    موجودہ Playwright راستے کو fallback کے طور پر برقرار رکھیں۔

## کھلے سوالات

- کیا نیا انجن `playwright`، `cdp`، یا `auto` کے طور پر قابلِ ترتیب ہونا چاہیے؟
- کیا ہم ایڈوانس صارفین کے لیے ایک نیا "nodeRef" فارمیٹ ظاہر کرنا چاہتے ہیں، یا صرف `ref` ہی برقرار رکھیں؟
- فریم اسنیپ شاٹس اور سلیکٹر-اسکوپڈ اسنیپ شاٹس کو AX میپنگ میں کیسے شامل ہونا چاہیے؟
