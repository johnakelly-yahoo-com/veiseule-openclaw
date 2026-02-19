---
summary: "အစီအစဉ်: Playwright queue မှ browser act:evaluate ကို CDP အသုံးပြုပြီး ခွဲထုတ်ခြင်း၊ end-to-end deadline များနှင့် ပိုမိုလုံခြုံသော ref resolution ဖြင့်"
owner: "openclaw"
status: "မူကြမ်း"
last_updated: "2026-02-10"
title: "Browser Evaluate CDP Refactor"
---

# Browser Evaluate CDP Refactor အစီအစဉ်

## အခြေအနေ

`act:evaluate` သည် အသုံးပြုသူ ပေးထားသော JavaScript ကို စာမျက်နှာအတွင်း လုပ်ဆောင်သည်။ လက်ရှိတွင် ၎င်းကို Playwright (`page.evaluate` သို့မဟုတ် `locator.evaluate`) မှတစ်ဆင့် လုပ်ဆောင်နေသည်။ Playwright သည် စာမျက်နှာတစ်ခုလျှင် CDP command များကို အစဉ်လိုက် စီမံဆောင်ရွက်သောကြောင့် မပြီးဆုံးဘဲ ကြာရှည်နေသော evaluate တစ်ခုက စာမျက်နှာ၏ command queue ကို ပိတ်ဆို့နိုင်ပြီး ထို tab ပေါ်ရှိ နောက်လာမည့် လုပ်ဆောင်ချက်အားလုံးကို "ပိတ်ဆို့နေသည်" ဟု ထင်ရစေနိုင်သည်။

PR #13498 သည် လက်တွေ့အသုံးဝင်သော လုံခြုံရေးအကာအကွယ်တစ်ရပ် (ကန့်သတ်ထားသော evaluate၊ abort propagation နှင့် အကောင်းဆုံးဖြေရှင်းနိုင်မှု) ကို ထည့်သွင်းထားသည်။ ဤစာတမ်းသည် `act:evaluate` ကို Playwright မှ သဘောတရားအရ သီးခြားခွဲထုတ်ပေးနိုင်သည့် ပိုမိုကြီးမားသော ပြန်လည်ဖွဲ့စည်းမှုကို ဖော်ပြထားပြီး stuck ဖြစ်သော evaluate တစ်ခုကြောင့် ပုံမှန် Playwright လုပ်ဆောင်ချက်များ မပိတ်ဆို့နိုင်စေရန် ရည်ရွယ်သည်။

## ရည်မှန်းချက်များ

- `act:evaluate` သည် တူညီသော tab ပေါ်ရှိ နောက်လာမည့် browser လုပ်ဆောင်ချက်များကို အမြဲတမ်း မပိတ်ဆို့နိုင်ရ။
- Timeout များကို end-to-end တစ်ခုတည်းသော အမှန်တရားအဖြစ် သတ်မှတ်ထားပြီး ခေါ်ယူသူက သတ်မှတ်ထားသော အချိန်ဘတ်ဂျက်ကို ယုံကြည်စိတ်ချနိုင်ရ။
- Abort နှင့် timeout ကို HTTP နှင့် in-process dispatch နှစ်မျိုးစလုံးတွင် တူညီသော ပုံစံဖြင့် ကိုင်တွယ်ရ။
- Playwright အားလုံးကို ပိတ်မထားဘဲ evaluate အတွက် element targeting ကို ပံ့ပိုးနိုင်ရ။
- လက်ရှိ ခေါ်ယူသူများနှင့် payload များအတွက် နောက်ပြန်ကိုက်ညီမှုကို ထိန်းသိမ်းရ။

## မရည်ရွယ်သည့်အချက်များ

- browser လုပ်ဆောင်ချက်အားလုံး (click, type, wait စသည်) ကို CDP implementation များဖြင့် အစားထိုးခြင်း။
- PR #13498 တွင် မိတ်ဆက်ထားသော လက်ရှိ safety net ကို ဖယ်ရှားခြင်း (၎င်းသည် အသုံးဝင်သော fallback အဖြစ် ဆက်လက်ရှိနေမည်)။
- ရှိပြီးသား `browser.evaluateEnabled` gate ကို ကျော်လွန်သည့် မလုံခြုံသော စွမ်းရည်အသစ်များကို မိတ်ဆက်ခြင်း။
- evaluate အတွက် process isolation (worker process/thread) ကို ထည့်သွင်းခြင်း။ ဤ refactor ပြီးနောက်တွင်လည်း ပြန်လည်ပြုပြင်ရန် ခက်ခဲသော stuck state များကို တွေ့ရှိနေပါက၊ ထိုအရာသည် နောက်ဆက်တွဲ စဉ်းစားရမည့် အကြံအစည်တစ်ခုဖြစ်သည်။

## လက်ရှိ Architecture (အဘယ်ကြောင့် Stuck ဖြစ်နိုင်သနည်း)

အမြင့်မားဆုံးအဆင့်တွင်:

- ခေါ်ယူသူများသည် browser control service သို့ `act:evaluate` ကို ပို့သည်။
- Route handler သည် JavaScript ကို လုပ်ဆောင်ရန် Playwright ကို ခေါ်ယူသည်။
- Playwright သည် page command များကို အစဉ်လိုက် စီမံသောကြောင့် မပြီးဆုံးသော evaluate တစ်ခုက queue ကို ပိတ်ဆို့နိုင်သည်။
- Queue ပိတ်ဆို့နေပါက ထို tab ပေါ်ရှိ နောက်လာမည့် click/type/wait လုပ်ဆောင်ချက်များသည် တုန့်ပြန်မှုမရှိသကဲ့သို့ ဖြစ်နိုင်သည်။

## အဆိုပြုထားသော Architecture

### 1. Deadline Propagation

တစ်ခုတည်းသော အချိန်ဘတ်ဂျက် အယူအဆကို မိတ်ဆက်ပြီး အရာအားလုံးကို ထိုအရာမှ ဆင်းသက်သတ်မှတ်သည်:

- ခေါ်ဆိုသူသည် `timeoutMs` (သို့မဟုတ် အနာဂတ်ရှိ deadline တစ်ခု) ကို သတ်မှတ်သည်။
- အပြင်ဘက် request timeout၊ route handler logic နှင့် page အတွင်းရှိ execution budget
  တို့အားလုံးသည် တူညီသော budget ကို အသုံးပြုကြပြီး၊ serialization overhead အတွက် လိုအပ်သည့်နေရာများတွင် အနည်းငယ် headroom ထားရှိသည်။
- Abort ကို နေရာအားလုံးတွင် `AbortSignal` အဖြစ် ဖြန့်ဝေထားသဖြင့် cancellation သည် တသမတ်တည်း ဖြစ်သည်။

အကောင်အထည်ဖော်ရန် လမ်းညွှန်ချက်:

- အသေးစား helper တစ်ခု (ဥပမာ `createBudget({ timeoutMs, signal })`) ထည့်သွင်းပါ၊ ၎င်းသည် အောက်ပါတို့ကို ပြန်လည်ပေးမည်:
  - `signal`: ချိတ်ဆက်ထားသော AbortSignal
  - `deadlineAtMs`: အတိအကျ deadline အချိန်
  - `remainingMs()`: child operations များအတွက် ကျန်ရှိသော budget
- ဤ helper ကို အောက်ပါနေရာများတွင် အသုံးပြုပါ:
  - `src/browser/client-fetch.ts` (HTTP နှင့် in-process dispatch)
  - `src/node-host/runner.ts` (proxy path)
  - browser action implementations (Playwright နှင့် CDP)

### 2. သီးခြား Evaluate Engine (CDP Path)

Playwright ၏ per page command queue ကို မမျှဝေသည့် CDP အခြေပြု evaluate implementation တစ်ခု ထည့်သွင်းပါ။ အဓိက လက္ခဏာမှာ evaluate transport သည် သီးခြား WebSocket connection တစ်ခုဖြစ်ပြီး target သို့ ချိတ်ဆက်ထားသော သီးခြား CDP session တစ်ခု ဖြစ်ခြင်းဖြစ်သည်။

အကောင်အထည်ဖော်ရန် လမ်းညွှန်ချက်:

- ဥပမာ `src/browser/cdp-evaluate.ts` ကဲ့သို့ module အသစ်တစ်ခု၊ ၎င်းသည်:
  - သတ်မှတ်ထားသော CDP endpoint (browser level socket) သို့ ချိတ်ဆက်သည်။
  - `Target.attachToTarget({ targetId, flatten: true })` ကို အသုံးပြု၍ `sessionId` ကို ရယူသည်။
  - အောက်ပါတို့အနက် တစ်ခုကို လုပ်ဆောင်သည်:
    - page level evaluate အတွက် `Runtime.evaluate`၊ သို့မဟုတ်
    - element evaluate အတွက် `DOM.resolveNode` နှင့် `Runtime.callFunctionOn` ကို ပေါင်းစပ်အသုံးပြုသည်။
  - timeout သို့မဟုတ် abort ဖြစ်ပါက:
    - session အတွက် `Runtime.terminateExecution` ကို best-effort ဖြင့် ပို့သည်။
    - WebSocket ကို ပိတ်ပြီး ရှင်းလင်းသော error တစ်ခု ပြန်လည်ပေးပို့သည်။

မှတ်ချက်များ:

- ၎င်းသည် page အတွင်း JavaScript ကို ဆက်လက် လုပ်ဆောင်နေဆဲဖြစ်သောကြောင့် termination သည် side effects များ ဖြစ်ပေါ်နိုင်သည်။ အကျိုးကျေးဇူးမှာ
  Playwright queue ကို မပိတ်ဆို့စေခြင်းဖြစ်ပြီး CDP session ကို ပိတ်ခြင်းဖြင့် transport
  layer တွင် cancel လုပ်နိုင်ခြင်း ဖြစ်သည်။

### 3. Ref Story (Element Targeting ကို Full Rewrite မပြုလုပ်ဘဲ)

ခက်ခဲသော အပိုင်းမှာ element targeting ဖြစ်သည်။ CDP သည် DOM handle သို့မဟုတ် `backendDOMNodeId` လိုအပ်သော်လည်း၊ ယနေ့တွင် browser actions အများစုသည် snapshot များမှ refs ကို အခြေခံထားသော Playwright locators များကို အသုံးပြုနေသည်။

အကြံပြုထားသော နည်းလမ်း: ရှိပြီးသား refs များကို ဆက်လက်ထိန်းသိမ်းထားပြီး၊ optional CDP resolvable id တစ်ခုကို ထည့်သွင်းချိတ်ဆက်ပါ။

#### 3.1 သိမ်းဆည်းထားသော Ref အချက်အလက်ကို တိုးချဲ့ခြင်း

သိမ်းဆည်းထားသော role ref metadata တွင် optional CDP id တစ်ခု ပါဝင်နိုင်ရန် တိုးချဲ့ပါ:

- ယနေ့: `{ role, name, nth }`
- အဆိုပြုချက်: `{ role, name, nth, backendDOMNodeId?: number }`

၎င်းသည် ရှိပြီးသား Playwright အခြေပြု actions အားလုံးကို ဆက်လက် လုပ်ဆောင်နိုင်စေပြီး `backendDOMNodeId` ရရှိနိုင်ပါက တူညီသော `ref` တန်ဖိုးကို CDP evaluate မှ လက်ခံနိုင်စေသည်။

#### 3.2 Snapshot အချိန်တွင် backendDOMNodeId ဖြည့်သွင်းခြင်း

role snapshot တစ်ခု ထုတ်လုပ်သောအခါ:

1. ယနေ့အတိုင်းရှိနေသော role ref map (role, name, nth) ကို ဖန်တီးပါ။
2. CDP မှတစ်ဆင့် AX tree (`Accessibility.getFullAXTree`) ကို ရယူပြီး တူညီသော duplicate handling စည်းမျဉ်းများကို အသုံးပြုကာ `(role, name, nth) -> backendDOMNodeId` ဟုသော parallel map တစ်ခုကို တွက်ချက်ပါ။
3. လက်ရှိ tab အတွက် သိမ်းဆည်းထားသော ref အချက်အလက်ထဲသို့ id ကို ပြန်လည်ပေါင်းထည့်ပါ။

ref တစ်ခုအတွက် mapping မအောင်မြင်ပါက `backendDOMNodeId` ကို undefined အဖြစ် ထားရှိပါ။ ဤနည်းလမ်းသည် feature ကို best-effort အဖြစ် လုံခြုံစွာ ဖြန့်ချိနိုင်စေသည်။

#### 3.3 Ref ဖြင့် Behavior ကို စစ်ဆေးခြင်း

`act:evaluate` တွင်:

- `ref` ရှိပြီး `backendDOMNodeId` ပါရှိပါက CDP မှတစ်ဆင့် element evaluate ကို လုပ်ဆောင်ပါ။
- `ref` ရှိသော်လည်း `backendDOMNodeId` မရှိပါက Playwright လမ်းကြောင်းကို fallback အဖြစ် အသုံးပြုပါ (safety net နှင့်အတူ)။

ရွေးချယ်နိုင်သော escape hatch:

- အဆင့်မြင့် ခေါ်ဆိုသူများ (နှင့် debugging အတွက်) အတွက် `backendDOMNodeId` ကို တိုက်ရိုက် လက်ခံနိုင်ရန် request ပုံစံကို တိုးချဲ့ပါ၊ သို့သော် `ref` ကို အဓိက interface အဖြစ် ဆက်လက်ထားရှိပါ။

### 4. နောက်ဆုံး အရေးပေါ် ပြန်လည်ပြုပြင်ရေး လမ်းကြောင်းတစ်ခုကို ထားရှိပါ

CDP evaluate ဖြင့်ပင် tab သို့မဟုတ် connection ကို ပိတ်ဆို့သွားစေနိုင်သော အခြားနည်းလမ်းများ ရှိနိုင်သည်။ အောက်ပါအခြေအနေများအတွက် နောက်ဆုံးအဖြစ် အသုံးပြုရန် ရှိပြီးသား recovery mechanisms (execution ကို terminate လုပ်ခြင်း + Playwright ကို disconnect လုပ်ခြင်း) ကို ဆက်လက်ထားရှိပါ:

- legacy callers
- CDP attach ကို ပိတ်ပင်ထားသော environments များ
- မမျှော်လင့်ထားသော Playwright edge cases များ

## Implementation Plan (Single Iteration)

### ပေးပို့ရမည့်အရာများ

- Playwright ၏ per-page command queue ပြင်ပတွင် လုပ်ဆောင်သည့် CDP အခြေပြု evaluate engine တစ်ခု။
- callers နှင့် handlers များက တူညီစွာ အသုံးပြုသော end-to-end timeout/abort budget တစ်ခု။
- element evaluate အတွက် `backendDOMNodeId` ကို ရွေးချယ်အသုံးပြုနိုင်စေရန် သယ်ဆောင်နိုင်သော Ref metadata။
- ဖြစ်နိုင်ပါက `act:evaluate` သည် CDP engine ကို ဦးစားပေးအသုံးပြုပြီး မဖြစ်နိုင်ပါက Playwright သို့ fallback လုပ်သည်။
- stuck ဖြစ်နေသော evaluate တစ်ခုကြောင့် နောက်ဆက်တွဲ action များ မပိတ်ဆို့သွားကြောင်း သက်သေပြသော စမ်းသပ်မှုများ။
- failure များနှင့် fallback များကို မြင်သာစေသော logs/metrics များ။

### Implementation Checklist

1. `timeoutMs` + upstream `AbortSignal` ကို အောက်ပါအရာများအဖြစ် ချိတ်ဆက်ပေးသော shared "budget" helper တစ်ခု ထည့်သွင်းပါ:
   - တစ်ခုတည်းသော `AbortSignal`
   - အပြည့်အစုံ သတ်မှတ်ထားသော deadline တစ်ခု
   - downstream operations များအတွက် `remainingMs()` helper တစ်ခု
2. `timeoutMs` သည် နေရာတိုင်းတွင် အဓိပ္ပါယ်တူညီစေရန် caller လမ်းကြောင်းများအားလုံးကို ထို helper ကို အသုံးပြုရန် update လုပ်ပါ:
   - `src/browser/client-fetch.ts` (HTTP နှင့် in-process dispatch)
   - `src/node-host/runner.ts` (node proxy လမ်းကြောင်း)
   - `/act` ကို ခေါ်သော CLI wrappers များ (`browser evaluate` တွင် `--timeout-ms` ထည့်သွင်းပါ)
3. `src/browser/cdp-evaluate.ts` ကို အကောင်အထည်ဖော်ပါ:
   - browser-level CDP socket သို့ ချိတ်ဆက်ပါ
   - `Target.attachToTarget` ကို အသုံးပြုပြီး `sessionId` ရယူပါ
   - page evaluate အတွက် `Runtime.evaluate` ကို လုပ်ဆောင်ပါ
   - element evaluate အတွက် `DOM.resolveNode` + `Runtime.callFunctionOn` ကို run လုပ်ပါ
   - timeout/abort ဖြစ်ပါက: best-effort `Runtime.terminateExecution` ကို အရင်လုပ်ပြီးနောက် socket ကို ပိတ်ပါ
4. သိမ်းဆည်းထားသော role ref metadata ကို လိုအပ်ပါက `backendDOMNodeId` ပါဝင်နိုင်하도록 တိုးချဲ့ပါ:
   - Playwright actions များအတွက် ရှိပြီးသား `{ role, name, nth }` အပြုအမူကို ဆက်လက်ထိန်းသိမ်းပါ
   - CDP element targeting အတွက် `backendDOMNodeId?: number` ကို ထည့်သွင်းပါ
5. snapshot ဖန်တီးစဉ် `backendDOMNodeId` ကို (best-effort) ဖြင့် ဖြည့်သွင်းပါ:
   - CDP မှတစ်ဆင့် AX tree ကို ရယူပါ (`Accessibility.getFullAXTree`)
   - `(role, name, nth) -> backendDOMNodeId` ကို တွက်ချက်ပြီး သိမ်းဆည်းထားသော ref map ထဲသို့ ပေါင်းစည်းပါ
   - mapping မရှင်းလင်းပါက သို့မဟုတ် မရှိပါက id ကို undefined အဖြစ်ထားပါ
6. `act:evaluate` routing ကို အပ်ဒိတ်လုပ်ပါ:
   - `ref` မရှိပါက: CDP evaluate ကို အမြဲအသုံးပြုပါ
   - `ref` သည် `backendDOMNodeId` သို့ resolve ဖြစ်ပါက: CDP element evaluate ကို အသုံးပြုပါ
   - မဟုတ်ပါက: Playwright evaluate သို့ fallback လုပ်ပါ (အချိန်ကန့်သတ်ထားပြီး abort လုပ်နိုင်ဆဲ)
7. ရှိပြီးသား "last resort" recovery path ကို default မဟုတ်ဘဲ fallback အဖြစ်သာ ထားရှိပါ။
8. tests များ ထည့်သွင်းပါ:
   - stuck evaluate သည် သတ်မှတ်ထားသော budget အတွင်း timeout ဖြစ်ပြီး နောက်တစ်ကြိမ် click/type သည် အောင်မြင်ရမည်
   - abort လုပ်ပါက evaluate ကို ပယ်ဖျက်ပြီး (client disconnect သို့မဟုတ် timeout) နောက်ဆက်တွဲ actions များကို ပြန်လည်လုပ်ဆောင်နိုင်စေရန် unblock လုပ်ရမည်
   - mapping မအောင်မြင်မှုများသည် Playwright သို့ သန့်ရှင်းစွာ fallback လုပ်ရမည်
9. observability ကို ထည့်သွင်းပါ:
   - evaluate ကြာချိန်နှင့် timeout counters
   - `terminateExecution` အသုံးပြုမှု
   - fallback rate (CDP -> Playwright) နှင့် အကြောင်းရင်းများ

### လက်ခံမှု စံနှုန်းများ (Acceptance Criteria)

- ရည်ရွယ်ချက်ဖြင့် hung ဖြစ်စေထားသော `act:evaluate` သည် ခေါ်ယူသူ၏ budget အတွင်း ပြန်လည်တုံ့ပြန်ရမည်ဖြစ်ပြီး နောက်ပိုင်း actions များအတွက်
  tab ကို မပိတ်ဆို့စေရ။
- `timeoutMs` သည် CLI, agent tool, node proxy နှင့် in-process calls တို့အကြား တ一致တည်း လုပ်ဆောင်ရမည်။
- `ref` ကို `backendDOMNodeId` သို့ map လုပ်နိုင်ပါက element evaluate သည် CDP ကို အသုံးပြုရမည်။ မဟုတ်ပါက fallback path သည်လည်း အချိန်ကန့်သတ်ထားပြီး ပြန်လည်ပြုပြင်နိုင်ရမည်။

## စမ်းသပ်မှု အစီအစဉ် (Testing Plan)

- Unit tests:
  - role refs နှင့် AX tree nodes များအကြား `(role, name, nth)` ကို ကိုက်ညီစေသော logic။
  - Budget helper ၏ လုပ်ဆောင်ပုံ (headroom, ကျန်ရှိသော အချိန်တွက်ချက်မှု)။
- Integration tests:
  - CDP evaluate timeout သည် သတ်မှတ်ထားသော budget အတွင်း ပြန်လာပြီး နောက်တစ်ကြိမ် action ကို မပိတ်ဆို့စေရ။
  - Abort လုပ်ခြင်းသည် evaluate ကို ပယ်ဖျက်ပြီး termination ကို best-effort ဖြင့် လုပ်ဆောင်စေရမည်။
- Contract tests:
  - `BrowserActRequest` နှင့် `BrowserActResponse` တို့သည် ဆက်လက်ကိုက်ညီမှုရှိနေကြောင်း သေချာစေရန်။

## အန္တရာယ်များ နှင့် လျော့ချရေးများ (Risks And Mitigations)

- Mapping သည် ပြည့်စုံမှု မရှိနိုင်ပါ:
  - လျော့ချရေး: best-effort mapping ကို အသုံးပြု၍ Playwright evaluate သို့ fallback လုပ်ပြီး debug tooling ကို ထည့်သွင်းပါ။
- `Runtime.terminateExecution` တွင် ဘေးထွက်သက်ရောက်မှုများ ရှိနိုင်ပါသည်:
  - လျော့ချရေး: timeout/abort ဖြစ်သည့်အခါတွင်သာ အသုံးပြုပြီး အမှားများတွင် အပြုအမူကို မှတ်တမ်းတင်ဖော်ပြပါ။
- အပို ထပ်ဆောင်း ဝန်ထုပ်ဝန်ပိုး:
  - လျှော့ချရေး: snapshots များကို တောင်းဆိုသည့်အခါမှသာ AX tree ကို ရယူပြီး၊ target တစ်ခုချင်းစီအလိုက် cache လုပ်ထားကာ CDP session ကို အချိန်တိုအတွင်းသာ အသုံးပြုပါ။
- Extension relay ကန့်သတ်ချက်များ:
  - လျှော့ချရေး: page တစ်ခုချင်းစီအတွက် socket မရရှိနိုင်သည့်အခါ browser အဆင့် attach APIs များကို အသုံးပြုပြီး၊ လက်ရှိ Playwright လမ်းကြောင်းကို fallback အဖြစ် ထားရှိပါ။

## ဖွင့်မေးခွန်းများ

- engine အသစ်ကို `playwright`, `cdp`, သို့မဟုတ် `auto` ဟု သတ်မှတ်ချိန်ညှိနိုင်အောင် ပြုလုပ်သင့်ပါသလား?
- အဆင့်မြင့်အသုံးပြုသူများအတွက် "nodeRef" format အသစ်ကို ဖော်ပြပေးသင့်ပါသလား၊ သို့မဟုတ် `ref` တစ်ခုတည်းသာ ထားရှိသင့်ပါသလား?
- frame snapshots နှင့် selector scoped snapshots များသည် AX mapping တွင် မည်သို့ ပါဝင်သင့်သနည်း?
