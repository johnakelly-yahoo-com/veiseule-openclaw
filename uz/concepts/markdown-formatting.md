---
title: "Markdown formatlash"
---

# Markdown formatlash

OpenClaw chiquvchi Markdown’ni avval umumiy oraliq
tasvir (IR) ga aylantirib, so‘ngra kanalga xos chiqishga render qilish orqali formatlaydi. IR
manba matnini o‘zgartirmagan holda saqlaydi va uslub/havola diapazonlarini olib yuradi, shunda
bo‘laklash va render qilish kanallar bo‘ylab izchil bo‘ladi.

## Maqsadlar

- **Izchillik:** bitta parse bosqichi, bir nechta renderer.
- **Xavfsiz bo‘laklash:** matnni render qilishdan oldin bo‘lish, shunda inline formatlash
  bo‘laklar orasida buzilmaydi.
- **Kanalga moslik:** bir xil IR’ni Slack mrkdwn, Telegram HTML va Signal
  uslub diapazonlariga Markdown’ni qayta parse qilmasdan moslash.

## Quvur

1. **Markdown -> IR parse qilish**
   - IR oddiy matn hamda uslub diapazonlari (bold/italic/strike/code/spoiler) va havola diapazonlaridan iborat.
   - Ofsetlar UTF-16 kod birliklarida beriladi, shunda Signal uslub diapazonlari uning API’siga mos keladi.
   - Jadvallar faqat kanal jadval konvertatsiyasini yoqqan bo‘lsa parse qilinadi.
2. **IR’ni bo‘laklash (avval format)**
   - Bo‘laklash render qilishdan oldin IR matnida amalga oshiriladi.
   - Inline formatlash bo‘laklar orasida bo‘linmaydi; diapazonlar har bir bo‘lak uchun kesib olinadi.
3. **Har bir kanal uchun render**
   - **Slack:** mrkdwn tokenlari (bold/italic/strike/code), havolalar `<url|label>` ko‘rinishida.
   - **Telegram:** HTML teglari (`<b>`, `<i>`, `<s>`, `<code>`, `<pre><code>`, `<a href>`).
   - **Signal:** oddiy matn + `text-style` diapazonlari; agar label URL’dan farq qilsa, havolalar `label (url)` ko‘rinishiga o‘tadi.

## IR misoli

Kirish Markdown:

```markdown
Hello **world** — see [docs](https://docs.openclaw.ai).
```

IR (sxematik):

```json
{
  "text": "Hello world — see docs.",
  "styles": [{ "start": 6, "end": 11, "style": "bold" }],
  "links": [{ "start": 19, "end": 23, "href": "https://docs.openclaw.ai" }]
}
```

## Qayerda ishlatiladi

- Slack, Telegram va Signal chiquvchi adapterlari IR’dan render qiladi.
- Boshqa kanallar (WhatsApp, iMessage, MS Teams, Discord) hanuz oddiy matn yoki
  o‘z formatlash qoidalaridan foydalanadi; yoqilgan bo‘lsa, Markdown jadval konvertatsiyasi
  bo‘laklashdan oldin qo‘llanadi.

## Jadval bilan ishlash

Markdown jadvallari chat mijozlari orasida bir xil qo‘llab-quvvatlanmaydi. Har bir kanal (va akkaunt) uchun
konvertatsiyani boshqarish uchun `markdown.tables` dan foydalaning.

- `code`: jadvallarni code blok sifatida render qilish (ko‘pchilik kanallar uchun standart).
- `bullets`: har bir qatordan punktli ro‘yxat qilish (Signal + WhatsApp uchun standart).
- `off`: jadval parse va konvertatsiyasini o‘chirish; jadvalning xom matni o‘zgarmasdan o‘tadi.

Config kalitlari:

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## Bo‘laklash qoidalari

- Bo‘lak cheklovlari kanal adapterlari/config’dan olinadi va IR matniga qo‘llanadi.
- Code fence’lar yakunida yangi qator bilan yagona blok sifatida saqlanadi, shunda kanallar
  ularni to‘g‘ri render qiladi.
- Ro‘yxat prefikslari va blockquote prefikslari IR matnining bir qismi, shuning uchun
  bo‘laklash prefiks o‘rtasida bo‘lib yubormaydi.
- Inline uslublar (bold/italic/strike/inline-code/spoiler) hech qachon bo‘laklar orasida
  bo‘linmaydi; renderer har bir bo‘lak ichida uslublarni qayta ochadi.

Kanallar bo‘ylab bo‘laklash xatti-harakati haqida batafsil ma’lumot uchun qarang:
[Streaming + chunking](/concepts/streaming).

## Havola siyosati

- **Slack:** `[label](url)` -> `<url|label>`; oddiy URL’lar o‘z holicha qoladi. Ikki marta havolaga aylantirishni oldini olish uchun
  parse vaqtida autolink o‘chiriladi.
- **Telegram:** `[label](url)` -> `<a href="url">label</a>` (HTML parse rejimi).
- **Signal:** `[label](url)` -> agar label URL bilan bir xil bo‘lmasa, `label (url)`.

## Spoilerlar

Spoiler belgilar (`||spoiler||`) faqat Signal uchun parse qilinadi va u yerda
SPOILER uslub diapazonlariga moslanadi. Boshqa kanallar ularni oddiy matn sifatida qabul qiladi.

## Kanal formatterni qo‘shish yoki yangilash

1. **Bir marta parse qiling:** kanalga mos
   opsiyalar (autolink, heading uslubi, blockquote prefiksi) bilan umumiy `markdownToIR(...)` yordamchisidan foydalaning.
2. **Render qiling:** `renderMarkdownWithMarkers(...)` va
   uslub marker xaritasi (yoki Signal uslub diapazonlari) bilan renderer yozing.
3. **Bo‘laklang:** render qilishdan oldin `chunkMarkdownIR(...)` ni chaqiring; har bir bo‘lakni alohida render qiling.
4. **Adapterni ulang:** kanal chiquvchi adapterini yangi bo‘laklagich
   va renderer’dan foydalanadigan qilib yangilang.
5. **Test qiling:** format testlarini qo‘shing yoki yangilang va agar kanal
   bo‘laklashdan foydalansa, chiquvchi yetkazib berish testini ham qo‘shing.

## Keng tarqalgan xatolar

- Slack burchak-qavs tokenlari (`<@U123>`, `<#C123>`, `<https://...>`) saqlanishi kerak;
  xom HTML’ni xavfsiz tarzda escape qiling.
- Telegram HTML’da teglardan tashqaridagi matnni escape qilish kerak, aks holda markup buziladi.
- Signal uslub diapazonlari UTF-16 ofsetlariga bog‘liq; code point ofsetlaridan foydalanmang.
- Fenced code bloklar uchun oxirgi yangi qatorni saqlang, shunda yopuvchi belgilar
  alohida qatorda joylashadi.

